---
title: CH04-开发指南
---

# CH04-开发指南

## 单处理器

如果能够保证要生成的代码仅会运行在单个处理器上，那就可以跳过本节的其余部分。因为单处理器保持着明确的顺序一致性，除非对象内存以某种方式与可异步访问的 IO 内存共享，否则永远都不需要插入屏障指令。采用了特殊映射的 java.nio buffers 可能会出现这种情况，但也许只会影响内部的 JVM 支持代码，而不会影响 Java 代码。而且，可以想象，如果上下文切换时不要求充分的同步，那就需要使用一些特殊的屏障了。

## 插入屏障

当程序执行时遇到了不同类型的存取，那就需要屏障指令。几乎无法找到一个“最理想”的位置，能将屏障执行总次数讲到最小。编译器不知道指定的 load 或 store 指令是先于还是后于需要一个屏障指令的另一个 load 或 store 指令，如：当 volatile store 后面是一个 return 时。最简单保守的策略是为任何一个给定的 load、store、lock 或 unlock 生成代码时，都假设该类型的存取需要“最重量级”的屏障：

- 在每条 volatile store 指令之前插入一个 StoreStore 屏障。
- 如果一个类包含 final 字段，在该类每个构造器的全部 store 指令之后、return 指令之前插入一个 StoreStore 屏障。
- 在每条 volatile store 指令之后插入一条 StoreStore 屏障。注意，虽然也可以在每条 volatile load 指令之前插入一个 StoreStore 屏障，但对于使用 volatile 的典型程序来说则会更慢，因为读操作会大大超过写操作。或者如果可以的话，将 volatile store 实现成一条原子指令，就可以省略掉屏障操作。如果原子指令比 StoreLoad 屏障成本低，这种方式就会更加高效。
- 在每条 volatile load 指令之后插入 LoadLoad 和 LoadStore 屏障。在持有数据依赖的处理器上，如果下一条存取指令依赖于 volatile load 出来的值，就不需要插入屏障。特别是，在 load 一个 volatile 引用之后，如果后续指令是 null 检查或 load 此引用所指对象中的某个字段，此时就无需屏障。
- 在每条 MonitorEnter 指令之前或在每条 MonitorExit 指令之后插入一个 ExitEnter 屏障。(根据上面的讨论，如果 MonitorExit 或 MonitorEnter 使用了相当于 StoreLoad 屏障的原子指令，ExitEnter 可以是个空操作(no-op)。其余步骤中，其他涉及 Enter 和 Eixt 的屏障也是如此。)
- 在每条 MonitorEnter 指令之后插入 EnterLoad 和 EnterStore 屏障。
- 在每条 MonitorExit 指令之后插入 StoreExit 和 LoadExit 屏障。
- 如果在未内置直接间接 load 顺序的处理器上，可以在 final 字段的每条 load 指令之前插入一个 LoadLoad 屏障。

这些屏障中的有一些通常会简化成空操作。实际上，大部分都会简化成空操作，只不过是在不同处理器的锁模式下使用了不同的方式。最简单的例子，在 x86 或 sparc-TSO 平台上使用 CAS 实现锁，仅相当于在 volatile store 后面放了一个 StoreLoad 屏障。

## 移除屏障

上面的保守策略对有些程序来说也许还可以接受。volatile 的主要性能问题出在 store 指令相关的 StoreLoad 屏障上。这些应当是相对罕见的——将 volatile 主要用于避免并发程序里读操作中锁的使用，仅当读操作大大超过写操作才会有问题。但是至少能在以下几个方面改进这种策略：

1. 移除冗余的屏障。可以根据前面章节的表格来消除屏障：

|            |      Original       |            |  =>  |           |     Transformed     |            |
| ---------- | :-----------------: | ---------- | :--: | --------- | :-----------------: | ---------- |
| 1st        |         ops         | 2nd        |  =>  | 1st       |         ops         | 2nd        |
| LoadLoad   |     [no loads]      | LoadLoad   |  =>  |           |     [no loads]      | LoadLoad   |
| LoadLoad   |     [no loads]      | StoreLoad  |  =>  |           |     [no loads]      | StoreLoad  |
| StoreStore |     [no stores]     | StoreStore |  =>  |           |     [no stores]     | StoreStore |
| StoreStore |     [no stores]     | StoreLoad  |  =>  |           |     [no stores]     | StoreLoad  |
| StoreLoad  |     [no loads]      | LoadLoad   |  =>  | StoreLoad |     [no loads]      |            |
| StoreLoad  |     [no stores]     | StoreStore |  =>  | StoreLoad |     [no loads]      |            |
| StoreLoad  | [no volatile loads] | StoreLoad  |  =>  |           | [no volatile loads] | StoreLoad  |

类似的屏障消除也可以用于锁的交互，但要依赖于锁的实现方式。使用循环、调用及分支来实现这一切的工作就作为读者练习吧。

2. 重排代码(在允许的范围内)以进一步移除 LoadLoad 和 LoadStore 屏障，这些屏障因处理器维持着数据依赖顺序而不再需要。
3. 移动指令流中屏障的位置以提高调度效率，只要在该屏障被需要的时间内最终仍会在某处执行即可。
4. 移除那些没有多线程依赖因此不再需要的屏障，例如，某个 volatile 变量被证实只会对单个线程可见。而且，如果能证明线程仅能对某些特定字段执行 store 指令或仅能执行 load 指令，则可以移除这里面使用的屏障。但是所有这些通常都需要进行大量的分析。

## 杂记

JSR 133 也讨论了在更为特殊的情况下可能需要屏障的其他几个问题：

- Thread.start 需要屏障来确保该已启动的线程能够看到在调用时刻对调用者可见的所有 store 的内容。相反，Thread.join 需要屏障来确保调用者能看到正在终止的线程所 store 的内容。实现 Thread.start 和 Thread.join 时需要同步，这些屏障通常是通过这些同步来产生的。
- static final 初始化需要 StoreStore 屏障，遵守 Java 类加载和初始化规则的那些机制需要这些屏障。
- 确保默认的 0/null 初始字段值时通常需要屏障、同步或/和垃圾收集器里的底层缓存控制。
- 在构造器之外或静态初始化器之外设置 System.in、System.out、System.err 的 JVM 私有例程需要特别注意，因为它们是 JMM final 字段规则的遗留例外情况。
- 类似的，JVM 内部反序列化设置 final 字段的代码通常需要一个 StoreStore 屏障。
- 终结方法可能需要屏障(垃圾收集器里)来确保 Object.finalize 中的代码能够看到某个对象不再被引用之前 store 到该对象所有字段的值。这通常是通过同步来确保的，这些同步用于在 reference 队列中添加和删除 reference。
- 调用 JNI 例程以及从 JNI 例程中返回可能需要屏障，尽管看起来是实现方面的问题。
- 大多数处理器都设计有其他专用于 IO 或 OS 操作的同步指令。他们不会直接影响 JMM 的这些问题，但是有可能与 IO、类加载及动态代码生成紧密相关。

