---
title: SE01-Rule Construction
---

# SE01-Rule Construction

一个 PEG 由任意数量的规则组成，规则又可以由其他规则、终止符、或下表中的原语规则组成：

| Name           | Common Notation | Primitive     |
| :------------- | :-------------: | :------------ |
| Sequence       |       a b       | a ~ b         |
| Ordered Choice |     a &#124; b     | a &#124; b       |
| Zero-Or-More   |       a *       | zeroOrMore(a) |
| One-Or-More    |       a +       | oneOrMore(a)  |
| Optional       |       a ?       | optional(a)   |
| And-Predicate  |       & a       | &(a)          |
| Non-Predicate  |       ! a       | !a            |

除了以上原语方法，还有以下原语可供使用：

| Method/Field        | Description                   |
| :------------------ | :---------------------------- |
| ANY                 | 匹配任何除了 EOI 的单个字符   |
| NOTHING             | 不匹配任何，总是失败          |
| EMPTY               | 不匹配任何，总是成功          |
| EOI                 | 匹配特殊的 EOI 字符           |
| ch(Char)            | 创建一个匹配单个字符的规则    |
| {String} ~ {String} | 匹配给定的字符范围            |
| anyOf(String)       | 匹配给定字符串中的任意字符    |
| ignoreCase(Char)    | 匹配单个字符且忽略大小写      |
| ignoreCase(String)  | 匹配整个字符串且胡烈大小写    |
| str(String)         | 创建一个匹配整个字符串的      |
| nTimes(Int, Rule)   | 创建一个匹配子规则 N 次的规则 |

