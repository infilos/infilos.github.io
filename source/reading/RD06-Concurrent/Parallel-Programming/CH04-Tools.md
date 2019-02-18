---
title: CH04-并行工具
---

# CH04-并行工具

本章主要介绍一些并行编程领域的基本工具，主要是类 Linux 系统上可以供应用程序使用的工具。

## 脚本语言

Shell 脚本提供了一种简单有效的并行化：

```bash
1 compute_it 1 > compute_it.1.out & 
2 compute_it 2 > compute_it.2.out & 
3 wait 
4 cat compute_it.1.out
5 cat compute_it.2.out
```

第 1、2 行分别启动了两个实例，通过 `&` 符号使这两个程序在后台运行，并分别将程序的输出重定向到一个文件。第 3 行等待两个实例执行完毕，第 4、5 行显示程序的输出。

<div  align="center">
<img src="https://infi-img.oss-cn-hangzhou.aliyuncs.com/img/20180922142852.png" style="display:block;width:50%;" alt="NAME" align=center />
</div>

另外，例如 `make` 脚本语言提供了一个 `-j` 选项来指定编译过程中同时执行多少个并行任务，`make -j4` 则表示同时执行 4 个并行编译过程。

既然基于脚本的并行编程这么简单，为什么还需要其他工具呢？

## POSIX 多进程

### POSIX 进程的创建与销毁

进程通过 `fork()` 原语创建，通过 `kill()` 原语销毁，也可以通过 `exit()` 原语实现自我销毁。执行 `fork()` 原语的进程被称为新创建进程的父进程，父进程可以功能通过 `wait()` 原语等待子进程执行完毕。

```c
1 pid = fork(); 
2 if (pid == 0) { 
3 	/ * child * / 
4 } else if (pid < 0) { 
5 	/ * parent, upon error * / 
6 	perror("fork"); 
7 	exit(-1); 
8 } else { 
9 	/ * parent, pid == child ID * / 
10 }
```

`fork()` 的返回值表示了其执行状态，即上面片段中的 pid。

```c
1 void waitall(void) 
2 { 
3 	int pid; 
4 	int status; 
5 
6 	for (;;) { 
7 		pid = wait(&status); 
8 		if (pid == -1) { 
9 			if (errno == ECHILD) 
10 			break; 
11 			perror("wait"); 
12 			exit(-1); 
13 		} 
14 	} 
15 }
```

父进程使用 `wait()` 原语来等待子进程时，`wait()` 只能等待一个子进程。我们将 `wait()` 原语封装成一个 `waitall()` 函数，该函数与 shell 中的 `wait` 意义一样：`for(;;)` 将会一直循环，每次循环等待一个子进程，阻塞直到该子进程退出，并返回子进程的进程 ID 号，如果该进程号为 -1，则表示 `wait()` 无法等待子进程执行完毕。如果检查错误码为 `ECHILD` 则表示没有其他子进程了，这时退出循环。

`wait()` 原语的复杂性在于，父进程与子进程之间不共享内存，而最细粒度的并行化需要共享内存，这时则要比不共享内存式的并行化复杂很多。

这种 `fork-waitall` 的形式被称为 **fork-join**。

### POSIX 线程创建与销毁

在一个已有的进程中创建线程，需要调用 `pthread_create()` 原语，它的第一个参数指向 `pthread_t` 类型的指针，第二个 NULL 参数是一个可选的指向 `pthread_attr_t` 结构的指针，第三个参数是新线程要调用的函数(下面的例子中是 `mythread()`)，最后一个 NULL 参数是传递给 `mythread()` 的参数。

```c
1 int x = 0; 
2 
3 void * mythread(void * arg) 
4 { 
5   x = 1; 
6   printf("Child process set x=1\n"); 
7   return NULL; 
8 } 
9 
10 int main(int argc, char * argv[]) 
11 { 
12   pthread_t tid; 
13   void * vp; 
14 
15   if (pthread_create(&tid, NULL, 
16 				        mythread, NULL) != 0) { 
17     perror("pthread_create"); 
18     exit(-1); 
19   } 
20   if (pthread_join(tid, &vp) != 0) { 
21     perror("pthread_join"); 
22     exit(-1); 
23   } 
24   printf("Parent process sees x=%d\n", x); 
25   return 0; 
26 }
```

- 第 7 行中，`mythread` 直接选择了返回，也可以使用 `pthread_exist()` 结束。
- 第 20 行的 `pthread_join()` 原语是对 fork-join 中 `wait()` 的模仿，它一直阻塞到 tid 变量指向的线程返回。线程的返回值要么是传给 `pthread_exit()` 的返回值，要么是线程调用函数的返回值，这取决于线程退出的方式。

上面代码的执行结果为：

```
Child process set x=1 
Parent process sees x=1
```

上面的代码中小心构造了一次只有一个线程为变量赋值的场景。任何一个线程为某变量赋值而另一线程读取变量值的场景，都会产生数据竞争(data-race)。因此我们需要一些手段来安全的并发读取数据，即下面的加锁原语。

### POSIX 锁

POSIX 规范支持开发者使用 POSIX 锁来避免数据竞争。POSIX 锁包括几个原语，其中最基础的是 `pthread_mutex_lock()` 和 `pthread_mutex_unlock()`。这些原语将会操作 `pthread_mutex_t` 类型的锁。该锁的静态声明和初始化由 `PTHREAD_MUTEX_INITIALIZER` 完成，或者由 `pthread_mutex_init()` 来动态分配并初始化。

因为这些加锁、解锁原语是互相排斥的，所以一次只能有一个线程在一个特定的时刻持有一把特定的锁。比如，如果两个线程尝试同时获取一把锁，那么其中一个线程会首先获准持有该锁，另一线程只能等待第一个线程释放该锁。

### POSIX 读写锁

POSIX API 提供了一种读写锁，用 `pthread_rwlock_t` 类型来表示，与 `pthread_mutex_t` 的初始化方式类似。`pthread_rwlock_wrlock()` 获取写锁，`pthread_rwlock_rdlock()` 获取读锁，`pthread_rwlock_unlock()` 用于释放锁。

读写锁是专门为大多数读的情况设计的。该锁能够提供比互斥锁更多的扩展性，因为互斥锁从定义上已经限制了任意时刻只能有一个线程持有锁，而读写锁运行任意多的线程同时持有读锁。

<div  align="center">
<img src="https://infi-img.oss-cn-hangzhou.aliyuncs.com/img/20180922143317.png" style="display:block;width:50%;" alt="读写锁的扩展性" align=center />
</div>

读写锁的可扩展性不甚理想，尤其是临界区较小时。为什么读锁的获取这么慢呢？应该是由于所有想获取读锁的线程都要更新 `pthread_rwlock_t` 的数据结构，因此一旦 128 个线程同时尝试获取读写锁的读锁时，那么这些线程必须逐个更新读锁中的 `pthread_rwlock_t` 结构。最幸运的线程几乎立即就获得了读锁，而最倒霉的线程则必须在前 127 的线程完成对该结构的更新后再能获得读锁。而增加 CPU 则会让性能变得更糟。

但是在临界区较大，比如开发者进行高延迟的文件或网络 IO 操纵时，读写锁仍然值得使用。

### 原子操作(GCC)

读写锁在临界区最小时开销最大，因此需要其他手段来保护极其短小的临界区。GCC 编译器提供了许多附加的原子操作：

- 返回参数原值
  - `__sync_fetch_and_sub()`
  - `__sync_fetch_and_or()`
  - `__sync_fetch_and_and()`
  - `__sync_fetch_and_xor()`
  - `__sync_fetch_and_nand()`
- 返回变量新值
  - `__sync_add_and_fetch()`
  - `__sync_sub_and_fetch()`
  - `__sync_or_ and_fetch()`
  - `__sync_and_and_fetch()`
  - `__sync_xor_and_fetch()`
  - `__sync_nand_ and_fetch()`

经典的比较并交换(CAS)是由一对原语 `__sync_bool_compare_and_swap()` 和 `__sync_val_compare_and_swap()` 提供的，当变量的原值与指定的参数值相等时，这两个原语会自动将新值写到指定变量。第一个原语在操作成功时返回 1，或在变量原值不等于指定值时返回 0；第二个原语在变量值等于指定的参数值时返回变量的原值，表示操作成功。任何对单一变量进行的原子操作都可以用 CAS 方式实现，上述两个原语是通用的，虽然第一个原语在适用的场景中效率更高。CAS 操作通常作为其他原子操作的基础。

`__sync_synchronize()` 原语是一个内存屏障，它限制编译器和 CPU 对指令乱序执行的优化。有些时候只限制编译器对指令的优化就够了，CPU 的优化可以保留，这时则需要 `barrier()` 原语。有时只需要让编译器不优化某个内存访问就够了，这时可以使用 `ACCESS_ONCE()` 原语。后两个原语并非由 GCC 直接提供，可以按如下方式实现：

```c
#define ACCESS_ONCE(x) (*(volatile typeof(x) *)&(x))
#define barrier() __asm____volatile__("": : :"memory")
```

## POSIX 的操作的替代选择

线程操作、加解锁原语、原子操作的出现早于各种标准委员会，因此这些操作存在多种变体。直接使用汇编实现这些操作也十分常见，不仅因为历史原因，还可以在某些特定场景下获得更好的性能。

## 如何选择

基于经验法则，应该在能够胜任工作的工具中选择最简单的一个。

1. 尽量保持串行。
2. Shell 脚本。
3. C 中的 fork-join。
4. POSIX 线程库原语。
5. 第几章将要介绍的原语。

除此之外，不要忘记除了共享内存多线程执行之外，还可以选择进程间通信和消息传递。
