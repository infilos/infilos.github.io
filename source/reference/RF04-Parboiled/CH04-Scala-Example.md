---
title: CH04-Scala Example
---

# CH04-Scala Example

首先是一个计算器的文法定义：

```
Expression <- Term (('+' / '-') Term)*
Term <- Factor (('*' / '/') Factor)*
Factor <- Number / '(' Expression ')'
Number <- [0-9]+
```

然后是基于 Scala 代码的解析器定义：

```scala
import org.parboiled.scala._

class SimpleCalculator extends Parser {
  
  def Expression:Rule0 = rule { Term ~ zeroOrMore(anyOf("+-") ~ Term) }
  
  def Term = rule { Factor ~ zeroOrMore(anyOf("*/") ~ Factor) }
  
  def Factor = rule { Number | "(" ~ Expression ~ ")" }
  
  def Number = rule { oneOrMore("0" - "9") }
}
```

最后是对该解析器的应用：

```scala
val input = "1+2"
val parser = new SimpleCalculator { override val buildParseTree = true }
val result = ReportingParseRunner(parser.Expression).run(input)
val parseTreePrintOut = org.parboiled.support.ParseTreeUtils.printNodeTree(result)
println(parseTreePrintOut)
```


