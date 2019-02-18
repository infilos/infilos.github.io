---
title: CH05-JSR133 Cook
---

# CH05-JSR133 Cook

从最初的探索至今已经有十年了。在此期间，很多关于处理器和语言的内存模型的规范和问题变得更清楚，更容易理解，但还有一些没有研究清楚。本指南一直在修订、完善来保证它的准确性，然而本指南部分内容展开的细节还不是很完整。想更全面的了解, 可以特别关注下 Peter Sewell 和 [Cambridge Relaxed Memory Concurrency Group](http://www.cl.cam.ac.uk/~pes20/weakmemory/index.html) 的研究工作。

这是一篇用于说明在 [JSR 133](http://jcp.org/en/jsr/detail?id=133) 中制定的新 [Java 内存模型(JMM)](http://www.cs.umd.edu/%7Epugh/java/memoryModel/) 的非官方指南。这篇指南提供了在最简单的背景下各种规则存在的原因，而不是这些规则在指令重排、多核处理器屏障指令和原子操作等方面对编译器和 JVM 所造成的影响。它还包括了一系列如何遵守 JSR 133 的指南。本指南是“非官方”的文档，因为它还包括特定处理器性能和规范的解释，我们不能保证所有的解释都是正确的。此外，处理器的规范和实现也可能会随时改变。

## Reference

- [The JSR-133 Cookbook for Compiler Writers](http://gee.cs.oswego.edu/dl/jmm/cookbook.html)
- [Java 内存模型 Cookbook](http://ifeve.com/jmm-cookbook/)