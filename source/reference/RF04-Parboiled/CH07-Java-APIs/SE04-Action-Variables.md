---
title: SE04-Action Variables
---

# SE04-Action Variables

对值栈的操作需要一些设计素养，同时为了类型安全，值栈中仅能使用一个较为宽泛的通用类型，然后再在解析器动作使用使用强制类型转换，这会带来维护成本。为了提供更多的灵活性，提供了动作变量功能。

通常，一个规则方法会在规则的子结构中包含多个动作表达式，以协同的方式来构造出最终规则。在很多情况下如果能够通过一个通用的临时辅助变量来访问规则中所有的动作，则会大有帮助。考虑如下例子：

```java
Rule Verbatim() {
    StringVar text = new StringVar("");
    StringVar temp = new StringVar("");
        return Sequence(
            OneOrMore(
                ZeroOrMore(BlankLine(), temp.append("\n")),
                NonblankIndentedLine(), text.append(temp.getAndSet(""), pop().getText())
            ),
            push(new VerbatimNode(text.get()))
        );
}
```

该规则用于解析 Markdown 的逐行结构，其中包含一行或多行的缩进文本。这些缩进行可以通过完全的空行来拆分，如果其跟随的有最少一个缩进行则也可以被匹配。该规则的工作是创建一个 AST 节点并初始化为匹配到的文本(不带有行缩进)。

为了能够构建该 AST 节点的文本参数，如果能够访问一个字符串变量——作为构建字符串的临时容器，则会非常有帮助。在通常的 Java 方法中可以使用一个本地变量，然而，因为规则方法仅包含规则的构造代码而非规则实际运行的代码，因此本地变量起不了作用。因为本地变量仅能在规则的构造期间而非运行期间可见。

这就是为什么 Parboiled Java 提供了一个名为 Var 的类，它可以用作规则执行阶段的本地变量。Var 对象包装一个任意类型的值，可以拥有初始值，支持对值的读写，可以在嵌套规则方法之间传递。每轮规则调用(如规则匹配重试)都会接受到自己的 Var 域，因此递归规则中的动作也会像预期一样运行。此外，Var 类还定义了一系列简便的辅助方法来简化其在动作表达式中的应用过程。

如下所示：

```java
Rule A() {
    Var<Integer> i = new Var<Integer>();
    return Sequence(
        ...,
        i.set(42),
        B(i),
        action(i.get())
    );
}

Rule B(Var<Integer> i) {
    return Sequence(
        ...,
        i.set(26)
    );
}
```

规则方法 A 传递一个其域内定义的 Var 作为参数到规则方法 B，规则方法 B 内的动作向该 Var 写入一个新值，规则方法 A 中所有运行在 B 之后的动作都能看到该新写入的 Var 的值。上面的例子中，A 中的 action 读取 Var 值时会得到 26。

