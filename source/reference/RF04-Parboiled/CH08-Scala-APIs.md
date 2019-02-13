---
title: CH08-Scala APIs
---

# CH08-Scala APIs

与 Java 的区别在于规则的构造过程，在 Scala 中使用了特殊的 Scala DSL 构造。相比 Java API，Scala API 更具优势：

- 更加简明的规则构建 DSL(Scala 语言的丰富表现力)。
- 通过对值栈的进一步抽象隐藏了值栈，增加了类型安全性(Scala Type Inference)。
- 高阶规则构造。
- 更快的初次规则构建(不再有昂贵的解析器扩展步骤)。