---
title: CH03-Java Example
---

# CH03-Java Example

首先是一个计算器的文法定义：

```
Expression <- Term (('+' / '-') Term)*
Term <- Factor (('*' / '/') Factor)*
Factor <- Number / '(' Expression ')'
Number <- [0-9]+
```

然后是基于 Java 代码的解析器定义：

```java
@BuildParseTree
class CalculatorParser extends BaseParser<Object> {
  
  Rule Expression() {
    return Sequence(
      Term(),
      ZeroOrMore(AnyOf("+-"), Term())
    );
  }
  
  Rule Term() {
    return Sequence(
      Factor(),
      ZeroOrMore(AnyOf("*/"), Factor())
    );
  }
  
  Rule Factor() {
    return FirstOf(
      Number(),
      Sequence('(', Expression(), ')')
    );
  }
  
  Rule Number() {
    return OneOrMore(CharRange('0', '9'));
  }
}
```

最后是对该解析器的应用：

```java
String input = "1+2";
CalculatorParser parser = Parboiled.createParser(CalculatorParser.class);
ParsingResult<?> result = new ReportingParseRunner(parser.Expression()).run(input);
String parseTreePrintOut = ParseTreeUtils.printNodeTree(result);
System.out.println(parseTreePrintOut);
```

第 2 行创建了一个解释器实例，其方法可以被用于各种不同的 ParserRunner 来执行实际的解析过程并创建 ParsingResult。ParsingResult 对象除了包含输入是否匹配的信息之外，还包含表达式的解析树的根(如果启用了解析书构建)、结果值、解析错误列表。理解解析器是如何处理输入的方式是使用 ParseTreeUtils.printNodeTree，如上面示例中第 4~5 行那样。

通常来说，Parboiled 会在 Java 语法的约束下使你的规则规范尽可能的可读。