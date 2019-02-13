---
title: SE05-AST Construction
---

# SE05-AST Construction

和解析树与语法之间的紧密关联关系相反，你想要构造的抽象语法树(AST)则完全取决于你的项目需要。这就是为什么 Parboiled 采用非常开放和灵活的方法来支持它。

事实上对 AST 节点的类型没有任何约束。Parboiled 没有提供任何不变或可变的对象来使用，因此也不会强制你使用什么。可以查看 `org.parboiled.trees` 包来开始。

通常你可以使用解析器值栈来作为构造 AST 的“工作台”。对于 Java API 可以参考 Calculators 示例，Scala API 可以参考 JSON Parser 示例。