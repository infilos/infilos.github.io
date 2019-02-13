---
title: SE01-Rule Construction
---

# SE01-Rule Construction

一个 PEG 由任意数量的规则组成，规则又可以由其他规则、终止符、或下表中的原语规则组成：

| Name           | Common Notation | Primitive      |
| :------------- | :-------------: | :------------- |
| Sequence       |       a b       | Sequence(a, b) |
| Ordered Choice |      a &#124; b       | FisrtOf(a, b)  |
| Zero-Or-More   |       a *       | ZeroOrMore(a)  |
| One-Or-More    |       a +       | OneOrMore(a)   |
| Optional       |       a ?       | Optional(a)    |
| And-Predicate  |       & a       | Test(a)        |
| Non-Predicate  |       ! a       | TestNot(a)     |

这些原语实际是 BaseParser 类的实例方法，即所有自定义解析器的必要父类。这些原语规则创建方法可以接收一个或多个 Object 参数，这些参数的类型可以是：

- 一个 Rule 实例
- 一个字符字面量
- 一个字符串字面量
- 一个字符数组
- 一个动作表达式
- 实现了 Action 接口的类的实例

除了以上原语方法，还有以下原语可供使用：

| Method/Field          | Description                        |
| :-------------------- | :--------------------------------- |
| ANY                   | 匹配任何除了 EOI 的单个字符        |
| NOTHING               | 不匹配任何，总是失败               |
| EMPTY                 | 不匹配任何，总是成功               |
| EOI                   | 匹配特殊的 EOI 字符                |
| Ch(char)              | 创建一个匹配单个字符的规则         |
| CharRange(char, char) | 匹配给定的字符范围                 |
| AnyOf(string)         | 匹配给定字符串中的任意字符         |
| NoneOf(string)        | 不匹配给定字符串中的任意字符和 EOI |
| IgnoreCase(char)      | 匹配单个字符且忽略大小写           |
| IgnoreCase(String)    | 匹配整个字符串且胡烈大小写         |
| String(string)        | 创建一个匹配整个字符串的           |


