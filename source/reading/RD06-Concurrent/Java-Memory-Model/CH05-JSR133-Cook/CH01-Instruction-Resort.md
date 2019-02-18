---
title: CH01-指令重排
---

# CH01-指令重排

对于编译器的编写者来说，JMM 主要是由“禁止指令重排的规则”所组成的，其中包括了字段(包括数组中的元素)的存取指令和监视器(锁)的控制指令。

## Volatile 与监视器

JMM 中关于 volatile 和监视器的主要规则可以被看做是一个矩阵。这个矩阵中的单元格表示当存在一些特定的后续关联指令的情况下，指令不能被重排。下面的表格并非 JMM 所包含的内容，而是一个用来观察 JMM 对编译器和运行系统所造成的影响的工具。

| 能否重排                    | 下个操作                | 下个操作                   | 下个操作                    |
| ---------- | ---------- | ---------- | ---------- |
| 第一个操作                  | Nomal Load/Nornal Store | Volatile Load/Moitor Enter | Volatile Store/Monitor Exit |
| Nomal Load/Nornal Store     |                         |                            | NO                          |
| Volatile Load/Moitor Enter  | NO                      | NO                         | NO                          |
| Volatile Store/Monitor Exit |                         | NO                         | NO                          |

术语说明：

- Normal Load 指令包括：对非 volatile 字段的读取，getField/getStatic/arrayLoad。
- Normal Store 指令包括：对非 volatile 字段的存储，putFiled/putStatic/arrayStore。
- Volatile Load 指令包括：对多线程环境的 volatile 变量的读取，getField/getStatic。
- Volatile Store 指令包括：对多线程环境的 volatile 变量的存储，putField/putStatic。
- Monitor Enter(包括进入同步块 synchronized 方法)是用于多线程环境的锁对象。
- Monitor Exist(包括离开同步块 synchronized 方法)是用于多线程环境的锁对象。

在 JMM 中，Normal Load 指令与 Nornal Store 指令的规则是一致的，类似的还有 Volatile Load 指令与 Monitor Enter 指令，以及 Volatile Store 指令和 Monitor Exit 指令，因此这几对指令的单元格在上面的表格中都被合并在了一起(但是在后续的表格中，会在必要的时候将其展开)。在这个小节中，我们仅仅考虑那些被当做原子单元的可读写的变量，也就是说那些没有位域(bit fields)、非对齐访问(unaligned acesses)、或者超过平台最大字长(word size)的访问。

任意数量的指令操作都可被表示成这个表格中的“第一个操作”或“下一个操作”。例如在单元格 [Normal Store, Volatile Store] 中，有一个 NO，就表示任何非 volatile 字段的 store 指令操作不能与后面任何一个 vaolatile store 指令重排，如果出现任何这样的重排就会使得多线程程序的行为发生变化。

JSR 133 规范规定上述关于 volatile 和监视器的规则仅仅适用于可能会被多线程访问的变量或对象。因此，如果一个编译器可以最终证明(这往往需要很大的努力)一个锁仅被单线程访问，那么这个锁就可以被移除。与之类似，一个 volatile 变量只被单线程访问也可以被当做是普通的变量。还有进一步耕细粒度的分析与优化，例如：那些被证明在一段时间内对多线程不可访问的字段。

在上表中，空白的单元格代表在不违反 Java 的基本语义下重排是允许的(详细可参考 [JLS](https://docs.oracle.com/javase/specs/) 中的说明)。例如，即使上表中没有说明，但是也不能对同一内存地址上的 load 指令和之后紧跟着的 store 指令进行重排。但是你可以对两个不同内存地址上的 load 和 store 指令进行重排，而且往往还在很多编译器转换和优化中会这么做。这往往就包括了一些不被认为是指令重排的例子，如：重用一个基于已加载的字段的计算后的值，而不是像第一次指令重排那样去重新加载并重新计算。然而，JMM 规范允许编译器经过一些转换后消除这些可以避免的依赖，使其可以支持指令重排。

在任何情况下，即使是开发者错误的使用了同步读取，指令重排的结果也必须达到最基本的 Java 安全要求，所有的显式字段都必须要么被设定成 0 或 null 这样的与构造值，要么被其他线程设置值。这通常必须把所有存储在堆内存里的对象在其被构造函数使用前进行归零操作，并且从来不对归零 store 指令进行重排。一种比较好的方式是在垃圾回收中对回收的内存进行归零操作。可以参考 JSR 133 规范中其他情况下的一些关于安全保证的规则。

这里描述的规则和属性都是适用于读取 Java 环境的字段。在实际的应用中，这些都可能会另外与读取内部的一些记账字段和数据交互，例如对象头，GC 表和动态生成的代码。

## final 字段

Final 字段的 load 和 store 指令相对于有锁的或者 volatile 字段来说，就跟 Normal load 和 Normal store 的存取是一样的，但是需要加入两条附加的指令重排规则：

1. 如果在构造函数中有一条 fianl 字段的 store 指令，同时这个字段是一个引用，那么它将不能与构造函数外后续可以让持有这个 final 字段的对象被其他线程访问的指令重排。例如，你不能重排下列语句：

```
x.finalField = v;
...;
sharedRef = x;
```

这条规则会在下列情况下生效，例如当你内联一个构造函数时，正如“...”的部分表示的该构造函数的逻辑边界那样。你不能把这个构造函数中的对这个 final 字段的 store 紫菱移动到构造函数外的一条 store 指令之后，因为这可能会使这个对象对其他线程可见。(正如你将在下面看到的，这样的操作还需要声明一个内存屏障)。类似的，你不能把下面的前两条指令与第三条指令进行重排：

```
x.afield = 1;
x.finalField = v;
...;
sharedRef = x;
```

2. 一个 final 字段的初始化 load 指令不能与包含该字段的对象的初始化 load 指令进行重排。在下面的情况中，这条规则就会生效：`x = shareRef;...;x=x.finalField`。由于这两条指令是依赖的，编译器不能对这样的指令进行重排。但是，这条规则会对某些处理器有影响。

上述规则，要求对于带有 fianl 字段的对象的 load 本身是 synchronized、volatile、final 或来自类似的 load 指令，从而确保 Java 开发者对于 fianl 字段的正确使用，并最终使构造函数中初始化的 store 指令和构造函数外的 store 指令排序。

