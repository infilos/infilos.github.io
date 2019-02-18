---
title: CH02-内存屏障
---

# CH02-内存屏障

编译器和处理器必须同时遵守重排规则。由于单核处理器能确保与“顺序执行”相同的一致性，所以在单核处理器上并不需要做什么处理就可以保证正确的执行顺序。但是在多核处理器上通常需要使用内存屏障指令来确保这种一致性。即使编译器优化掉了一个字段访问(比如因为一个读入的值未被使用)，这种情况下还是需要产生内存屏障，就好像该访问仍然需要被保护一样。

内存屏障仅仅与内存模型中“获取”、“释放”这些高层次概念有间接的关系。内存屏障并非“同步屏障”，内存屏障也与在一些垃圾回收机制中的“写屏障(write barriers)”概念无关。内存屏障指令仅仅直接控制 CPU 与缓存之间，CPU 与其准备将数据写入主存或写入等待读取、预测指令执行的缓冲中的写缓冲之间的相互操作。这些操作可能导致缓冲、主存和其他处理器执行进一步的交互。但在 Java 内存模型规范中，没有强制处理之间的交互方式，只要数据最终变为全局可用，即在所有处理器中均可见，并当这些数据可见时可以获得它们。

## 内存屏障的种类

几乎所有处理器都至少支持一种粗粒度的屏障指令，通常被称为“栅栏(fence)”，它保证栅栏前初始化的 load 和 store 指令，能够严格有序的在栅栏后的 load 和 store 指令之前执行。无论在何种处理器上，这几乎是最耗时的操作之一(与原子指令差不多、甚至更加消耗资源)，所以大部分处理器支持更细粒度的屏障指令。

内存屏障的一个特性是将它们运用于内存之间的访问。尽管在一些处理器上有一些名为屏障的指令，但是正确的、最好的屏障使用取决于内存访问的类型。下面是一些屏障指令的通用分类，它们正好能够对应上常用处理器上的特定指令(有时这些指令会导致空操作)。

### LoadLoad 屏障

```
Load1, LoadLoad, Load2
```

确保 Load1 所要读入的数据能够在被 Load2 和后续的 load 指令访问前读入。通常执行预加载指令或/和支持乱序处理的处理器中需要显式声明该 LoadLoad 屏障，因为在这些处理器中正在等待的加载指令能够绕过正在等待存储的指令。而对于总是能保证处理顺序的处理器，设置该屏障相当于空操作。

### StoreStore 屏障

```
Store1, StoreStore, Store2
```

确保 Store1 的数据在 Store2 及后续 store 指令操作相关数据之前对其处理器可见(如向主存刷新数据)。通常情况下，如果处理器不能保证从写缓冲或/和缓存向其他处理器和主存中按顺序刷新数据，那么就需要使用 StoreStore 屏障。

### LoadStore 屏障

```
Load1, LoadStore, Store2
```

确保 Load1 的数据在 Store2 和后续 store 指令被刷新之前读取。在等待 store 指令可以越过 load 指令的乱序处理器上需要使用 LoadStore 屏障。

### StoreLoad 屏障

```
Store1, LoadStore, Load2
```

确保 Store1 的数据在被 Load2 及后续的 load 指令读取之前对其他处理器可见。StoreLoad 屏障可以放置一个后续的 load 指令不正确的使用 Store1 的数据，而不是另一个处理器在相同内存位置写入一个新数据。真因为如此，所以下面所讨论的处理器为了在屏障前读取同样内存位置存过的数据时，必须使用一个 StoreLoad 屏障将存储指令和后续的加载指令分开。StoreLoad 屏障在几乎所有的现代处理器中都需要使用，但通常它的开销也是最昂贵的。它们昂贵的部分原因是它们必须关闭通常的“略过缓存直接从写缓冲区读取数据”的机制。这可能通过让一个缓冲区进行充分刷新，以及它的延迟的方式来实现。

在下面讨论的所有处理器中，执行 StoreLoad 指令也会同时获得其他三种屏障效果。所以 StoreLoad 可以作为最通用(但通常也是最耗性能)的一种 fence。(这是基于经验得出的结论，并非必然)。反之则不成立，为了达到 StoreLoad 的效果而组合使用其他屏障的情况并不多见。

### 排序规则

下表显示了这些屏障如何符合 JSR 133 的排序规则：

| 需要的屏障                  | 下个操作   | 下个操作     | 下个操作                   | 下个操作                    |
| ---------- | ---------- | ---------- | ---------- | ---------- |
| 第一个操作                  | Nomal Load | Nornal Store | Volatile Load Moitor Enter | Volatile Store Monitor Exit |
| Nomal Load                  |            |              |                            | LoadStore                   |
| Nornal Store                |            |              |                            | StoreStore                  |
| Volatile Load Moitor Enter  | LoadLoad   | LoadStore    | LoadLoad                   | LoadStore                   |
| Volatile Store Monitor Exit |            |              | StoreLoad                  | StoreStore                  |

另外，特殊的 final 字段规则在下列代码中需要一个 StoreStore 屏障：

```
x.finalField = v;
StoreStore;
sharedRef = x;
```

下面的例子解释了如何放置屏障：

```
Class X {
  int a, b;
  volatil int v, u;
  
  void f() {
    int i, j;
    i = a;// load a
    j = b;// load b
    i = v;// load v
    // LoadLoad
    j = u;// load u
    // LoadStore
    a = i;// store a
        b = j;// store b
        // StoreStore
        v = i;// store v
        // StoreStore
        u = j;// store u
        // StoreLoad
        i = u;// load u
        // LoadLoad
        // LoadStore
        j = b;// load b
        a = i;// store a
  }
}
```

## 数据依赖与屏障

一些处理器为了保证依赖指令的交互次序需要使用 LoadLoad 和 LoadStore 屏障。在一些(大部分)处理器中，一个 load 指令或者一个依赖于之前加载值的 store 指令被处理器排序，并不需要一个显式的屏障。这通常发生于两种情况：

- 间接取值(indirection)：`Load x; Load x.field`
- 条件控制(control)： `Load x; if(predicate(x)) Load or Store y;`

但特别的是不遵循间接排序的处理器，需要为 final 字段设置屏障，使它能通过共享引用来访问最初的引用。

```
x = sharedRef;
...;
LoadLoad;
i = x.finalField;
```

相反的，如下讨论，确定遵循数据依赖的处理器，提供了几种优化掉 LoadLoad 和 LoadStore 屏障指令的机会。(尽管如此，在任何处理器上，对于 StoreLoad 屏障不会自动清除依赖关系)

### 与原子指令交互

屏障在不同处理器上还需要与 MonitorEnter 和 MonitorExit 实现交互。加解锁通常必须使用原子条件更新操作 CampareAndSwap(CAS) 指令或 LoadLinked/StoreConditional(LL/SC)，就如执行一个 volatile store 之后紧跟 volatile load 的语义一样。CAS 或者 LL/SC 能够满足最小功能，一些处理器还需要提供其他的原子操作(如，一个无条件交换)，这在某些时候它可以替代或者与原子条件更新操作结合使用。

在所有处理器中，原子操作可以避免在正被读取/更新的内存位置进行“写后读(read-after-write)”。(否则标准的循环直到成功的结构体(loop-until-success)无法正常工作)。但处理器是否在为原子操作提供比隐式的 StoreLoad 更一般的屏障特性上表现不同。一些处理器上这些指令可以为 MonitorEnter/Exit 原生的生成屏障；其他处理器中一部分或全部屏障必须显式的指定。

为了分清这些影响，我们必须把 volatile 和 monitor 分开：

| 需要的屏障     | 下个操作   | 下个操作     | 下个操作      | 下个操作       | 下个操作     | 下个操作     |
| ---------- | ---------- | ---------- | ---------- | ---------- | ---------- | ---------- |
| 第一个操作     | Nomal Load | Nornal Store | Volatile Load | Volatile Store | Moitor Enter | Monitor Exit |
| Nomal Load     |            |              |               | LoadStore      |              | LoadStore    |
| Nornal Store   |            |              |               | StoreStore     |              | StoreExit    |
| Volatile Load  | LoadLoad   | LoadStore    | LoadLoad      | LoadStore      | LoadEnter    | LoadExit     |
| Volatile Store |            |              | StoreLoad     | StoreStore     | StoreEnter   | StoreExit    |
| Moitor Enter   | EnterLoad  | EnterStore   | EnterLoad     | EnterStore     | EnterEnter   | EnterExit    |
| Monitor Exit   |            |              | ExitLoad      | ExitStore      | ExitEnter    | ExitExit     |

同样，特殊的 final 字段规则需要一个 StoreLoad 屏障：

```
x.finalField = v;
StoreStore;
sharedRef = x;
```

在该表中，“Enter” 与 “Load” 相同，“Exit” 与 “Store” 相同，除非被原子性指令的使用和特性覆盖。特别是：

- EnterLoad 在进入任何需要执行 Load 指令的同步块/方法时都需要。这与 LoadLoad 相同，除非在 MonitorEnter 时候使用了原子指令并且它本身提供一个至少有 LoadLoad 属性的屏障。如果是这种情况，相当于空操作。
- StoreExit 在退出任何执行 Store 指令的同步方法块时都需要。这与 StoreStore 一致，除非 MonitorExit 使用原子操作，并且提供了一个至少拥有 StoreStore 属性的屏障，如果是这种情况，相当于空操作。
- ExitEnter 和 StoreLoad 一样，除非 MonitorExit 使用了原子指令，并且/或者 MonitorEnter 至少提供一种屏障，该屏障具有 StoreLoad 的属性，如果是这种情况，相当于没有操作。

在编译时不起作用或者导致处理器上不产生操作的指令比较特殊。例如，当没有交替的 laod 和 store 指令时，EnterEnter 用于分离嵌套的 MonitorEnter。下面的例子说明了如何使用这些指令类型：

```
class X {
  int a;
  volatile int v;

  void f() {
    int i;
    synchronized (this) { // enter EnterLoad EnterStore
      i = a;// load a
      a = i;// store a
    }// LoadExit StoreExit exit ExitEnter

    synchronized (this) {// enter ExitEnter
      synchronized (this) {// enter
      }// EnterExit exit
    }// ExitExit exit ExitEnter ExitLoad

    i = v;// load v

    synchronized (this) {// LoadEnter enter
    } // exit ExitEnter ExitStore

    v = i; // store v
    synchronized (this) { // StoreEnter enter
    } // EnterExit exit
  }

}
```

Java 层次的对原子条件更新的操作将在 JDK 1.5 中发布(JSR 166)，因此编译器需要发布相应的代码，综合使用上表中对 MonitorEnter 和 MonitorExist 的方式，从语义上说，有时在实践中，这些 Java 中的原子更新操作，就如同他们被锁所包围一样。