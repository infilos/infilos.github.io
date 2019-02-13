---
title: CH07-Java APIs
---

# CH07-Java APIs

Parboiled Java 的应用步骤：

1. 安装依赖。
2. 确定要解析器值栈中要参数化的类型，继承 BaseParser 实现自定义解析类。
3. 在该解析类中添加返回类型为 Rule 的规则方法。
4. 通过 `Parboiled.createParser` 创建解析器实例。
5. 调用解析器的根规则方法来创建规则树。
6. 选择 ParseRunner 的特定实现，调用其 run 方法并传入根规则和输入文本。
7. 查看 ParsingResult 对象的属性。