---
title: Endix-C-虚拟机参数
---

# Endix-C-虚拟机参数

使用 `-XX:+PrintFlagsFinal` 可以输出所有参数的名称和默认值。应用参数的方式有以下 3 种：

1. `-XX:+＜option＞开启option参数`
2. `-XX:-＜option＞关闭option参数`
3. `-XX:＜option＞=＜value＞将option参数的值设置为value`

以下是 JDK6 中常用的参数。

## 内存管理参数

| 参数                           | 默认值                                                       | 使用介绍                                                     |
| ------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| DisableExplicitGC              | 默认关闭                                                     | 忽略来自 System.gc() 方法触发的垃圾回收                      |
| ExplicitGCInvokesConcurrent    | 默认关闭                                                     | 当收到 System.gc() 方法提交的垃圾回收申请时，使用 CMS 收集器进行收集 |
| UseSerialGC                    | Client 模式 的虚拟机默认开启，其他模式默认关闭               | 虚拟机运行在 Client 模式下的默认值，打开此开关后，使用 Serial+Serial Old 的收集器组合进行内存回收 |
| UseParNewGC                    | 默认关闭                                                     | 打开此开关后，使用 ParNew+Serial Old 的收集器组合进行内存回收 |
| UserConcMarkSweepGC            | 默认关闭                                                     | 打开此开关后，使用 ParNew+CMS+Serial Old 的收集器组合进行内存回收。如果 CMS 收集器出现 Concurrent Mode Failure，则 Serial Old 收集器将作为后备收集器 |
| UserParallelGC                 | Server 模式的虚拟机默认开启，其他模式默认关闭                | 虚拟机运行在 Server 模式下的默认值，打开此开关后，使用 Parallel Scavenge+Serial Old 的收集器组合进行内存回收 |
| UseParallelOldGC               | 默认关闭                                                     | 打开此开关后，使用 Parallel Scavenge+Parallel Old 的收集器组合进行内存回收 |
| ServivorRatio                  | 默认为 8                                                     | 新生代中 Eden 区域与 Survivor 区域的容量比值                 |
| PretenureSizeThreshold         | 无默认值                                                     | 直接晋升到老年代的对象大小，设置该参数后，大于这个参数的对象将直接在老年代分配 |
| MaxTenuringThreshold           | 默认为 15                                                    | 晋升到老年代的对象年龄。每个对象在坚持过一次 Mirror GC 之后，年龄就加 1，当超过该参数值时进入老年代 |
| UseAdaptiveSizePolicy          | 默认开启                                                     | 动态调整 Java 堆中各个区域的大小及进入老年代的年龄           |
| HandlePromotionFailure         | JDK 1.5 及之前的版本默认关闭，JDK 1.6 之后默认开启           | 是否允许分配担保失败，即老年代的剩余空间不足以应付新生代的整个 Eden 和 Survivor 区的所有对象都存活的极端情况 |
| ParallelGCThreads              | 少于或等于 8 个 CPU 时默认为 CPU 的数量值，多余 8 个 CPU 时比 CPU 的数量之小 | 设置并行 GC 时进行内存回收的线程数                           |
| GCTimeRatio                    | 默认为 99                                                    | GC 时间占总时间的比例，默认允许 1% 的 GC 时间。仅在使用 Parallel Scavenge 收集器时生效 |
| MaxGCPauseMillis               | 无默认值                                                     | 设置 GC 的最大停顿时间。仅在使用 Parallel Scavenge 收集器时生效 |
| CMSInitiatingOccupancyFraction | 默认 68                                                      | 设置 CMS 收集器在老年代空间被使用多少后触发垃圾回收，仅在使用 CMS 收集器时生效 |
| UseCMSCompactAtFullCollection  | 默认开启                                                     | 设置 CMS 收集器在完成垃圾收集后是否需要进行一次内存碎片整理。仅在使用 CMS 收集器时生效 |
| CMSFullGCsBeforeCompaction     | 无默认值                                                     | 设置 CMS 收集器在进行若干次垃圾收集后在启动一次内存碎片整理。仅在使用 CMS 收集器时生效 |
| ScavengeBeforeFullGC           | 默认开启                                                     | 在 Full GC 发生之前触发一次 Mirror GC                        |
| UseGCOverheadLimit             | 默认开启                                                     | 禁止 GC 过程无限制的执行，如果过于频繁，就直接发生 OutOfMemory 异常 |
| UseTLAB                        | Server 模式默认开启                                          | 优先在本地线程缓冲区中分配对象，避免分配内存时的锁定过程     |
| MaxHeapFreeRatio               | 默认为 70                                                    | 当 Xmx 比 Xms 值大时，堆可以动态收缩和扩展，该参数控制当堆空闲大于指定比例时自动收缩 |
| MinHeapFreeRatio               | 默认为 70                                                    | 当 Xmx 比 Xms 值小时，对可以动态收缩或扩展，该参数控制当对空闲小于指定比率时自动扩展 |
| MaxPermSize                    | 大部分情况下默认为 64MB                                      | 永久代的最大值                                               |

## 即时编译参数

| 参数                     | 默认值                                         | 使用介绍                                                     |
| ------------------------ | ---------------------------------------------- | ------------------------------------------------------------ |
| CompileThreshold         | Client 模式下默认为 1500，Server 模式下为 1000 | 触发方法即时编译的阈值                                       |
| OnStackReplaceRercentage | Client 模式下为 933，Server 模式下为 140       | OSR 比率，它是 OSR 即时编译阈值计算公式的一个参数，用于代替 BckEdgeThreshold 参数控制回边计数器的实际溢出阈值 |
| ReversedCodeCacheSize    | 大部分情况下是 32MB                            | 即时编译器编译的代码缓存的最大值                             |

## 类型加载参数

| 参数                    | 默认值   | 使用介绍                                                     |
| ----------------------- | -------- | ------------------------------------------------------------ |
| UseSplitVerifer         | 默认开启 | 使用依赖 StackMapTable 信息的类型检查代替数据流分析，以加快字节码校验速度 |
| FailOverToOldVerifier   | 默认开启 | 当类型校验失败时，是否允许回到老的类型推导校验方式重新校验，如果开启则允许 |
| RelaxAccessControlCheck | 默认关闭 | 在检验阶段放松对类型访问性的限制                             |

## 多线程相关参数

| 参数                   | 默认值                             | 使用介绍                                                 |
| ---------------------- | ---------------------------------- | -------------------------------------------------------- |
| UseSpinning            | JDK 1.6 默认开启，JDK 1.5 默认关闭 | 开启自旋锁以避免线程频繁挂起和唤醒                       |
| PreBlockSpin           | 默认为 10                          | 使用自旋锁时默认的自旋次数                               |
| UseThreadPriorities    | 默认开启                           | 使用本地线程优先级                                       |
| UseBiasedLocking       | 默认开启                           | 使用使用偏向锁                                           |
| UseFastAccessorMethods | 默认开启                           | 当频繁反射执行某个方法时，生成字节码来加快反射的执行速度 |

## 性能参数

| 参数                 | 默认值                             | 使用介绍                                                     |
| -------------------- | ---------------------------------- | ------------------------------------------------------------ |
| AggressiveOpts       | JDK 1.6 默认开启，JDK 1.5 默认关闭 | 使用激进的优化特性，这些特性一般是具备正面和负面双重影响的，需要根据具体应用特点来分析才能判定是否对性能有益 |
| UseLargePages        | 默认开启                           | 如果可能，使用大内存分页，该特性需要操作系统的支持           |
| LargePageSizeInBytes | 默认为 4MB                         | 使用指定大小的内存分页，该特性需要操作系统的支持             |
| StringCache          | 默认开启                           | 使用使用字符串缓存                                           |

## 调试参数

| 参数                       | 默认值   | 使用介绍                                                     |
| -------------------------- | -------- | ------------------------------------------------------------ |
| HeapDumpOnOutOfMemoryError | 默认关闭 | 在发生内存溢出时是否生成堆转储快照                           |
| OnOutOfMemoryError         | 无默认值 | 当虚拟机抛出内存溢出异常时，执行指定的命令                   |
| OnError                    | 无默认值 | 当虚拟机抛出 ERROR 异常时，执行指定的命令                    |
| PrintClassHistogram        | 默认关闭 | 使用 `[ctrl]-[break]` 快捷键输出类统计状态，相当于 jmap-histo 的功能 |
| PrintConcurrentLocks       | 默认关闭 | 打印 JUC 中锁的状态                                          |
| PrintCommandLineFlags      | 默认关闭 | 打印启动虚拟机时输入的非稳定参数                             |
| PrintCompilation           | 默认关闭 | 打印方法的即时编译信息                                       |
| PrintGC                    | 默认关闭 | 打印 GC 信息                                                 |
| PrintGCDetails             | 默认关闭 | 打印 GC 详细信息                                             |
| PrintGCtimeStamps          | 默认关闭 | 打印 GC 停顿耗时                                             |
| PrintTenuringDistribution  | 默认关闭 | 打印 GC 后新生代各个年龄对象的大小                           |
| TracClassLoading           | 默认关闭 | 打印类加载信息                                               |
| TraceClassUnloading        | 默认关闭 | 打印类卸载信息                                               |
| PringInlining              | 默认关闭 | 打印方法的内联信息                                           |
| PrintCFGToFile             | 默认关闭 | 将 CFG 图信息输出到文件，只有 DEBUG 版虚拟机才支持该参数     |
| PrintIdealGraphFile        | 默认关闭 | 将 Ideal 图信息输出到文件，只有 DEBUG 版虚拟机才支持该参数   |
| UnlockDiagnosticVM Options | 默认关闭 | 让虚拟机进入诊断模式，一些参数(如 PrintAssembly)需要在诊断模式下才能使用 |
| PrintAssembly              | 默认关闭 | 打印即时编译后的二进制信息                                   |



