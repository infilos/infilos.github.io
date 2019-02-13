---
title: SE02-Parser Action Expressions
---

# SE02-Parser Action Expressions

Parboiled Java 的解析器可以在规则定义的任意位置包含解析器动作。这些动作可以分为 4 类。

## “Regular” objects implementing the Action interface

如果你的动作代码较多，则可以将其封装到一个实现了 Action 接口的自定义类中，然后再在规则定义方法中使用该自定义类的实例。

```java
class MyParser extends BaseParser<Object> {
    Action myAction = new MyActionClass();

    Rule MyRule() {
        return Sequence(
            ...,
            myAction,
            ...
        );
    }
}
```

如果使用这种方式，你的自定义动作类也可以实现 SkippableAction 接口以告诉解析器引擎在执行内部的语法判定时是否跳过这些动作。

## Anonymous inner classes implementing the Action interface

更多时候动作仅会包含少量的代码，这时可以直接使用匿名类：

```java
class MyParser extends BaseParser<Object> {

    Rule MyRule() {
        return Sequence(
            ...,
            new Action() {
                public boolean run(Context context) {
                    ...; // arbitrary action code
                    return true; // could also return false to stop matching the Sequence and continue looking for other matching alternatives
                }
            },
            ...
        );
    }
}
```

## Explicit action expressions

虽然匿名类要比独立的动作类定义简单一点，但仍然显得冗余。可以继续简化为一个布尔表达式：

```java
class MyParser extends BaseParser<Object> {

    Rule MyRule() {
        return Sequence(
            ...,
            ACTION(...), // the argument being the boolean expression to wrap
            ...
        );
    }
}
```

`BaseParser.ACTION` 是一个特殊的标记方法，其告诉 Parboiled 将参数表达式封装到一个单独的、自动创建的动作类中，类似上面匿名类的例子。这样的动作表达式中可以包含对本地变量的访问代码或方法参数、读写非私有的解析器字段、调用非私有的解析器方法。

此外，如果动作表达式中调用实现了 ContextAware 接口的类对象方法，将自动在调用方法之前插入 setContext 方法。比如你想将所有的动作代码移出到解析器类之外以简化实现：

```java
class MyParser extends BaseParser<Object> {
    MyActions actions = new MyActions();

    Rule MyRule() {
        return Sequence(
            ...,
            ACTION(actions.someAction()),
            ...
        );
    }
}
```

如果 MyActions 实现了 ContextAware 接口，Parboiled 将会自动在内部转换为类似下列清单的代码：

```java
class MyParser extends BaseParser<Object> {
    MyActions actions = new MyActions();

    Rule MyRule() {
        return Sequence(
            ...,
            new Action() {
                public boolean run(Context context) {
                    actions.setContext(context);
                    return actions.someAction();
                }
            },
            ...
        );
    }
}
```

注意 BaseParser 已经继承了 BaseActions，其实现了 ContextAware 接口，所以解析器类中的所有动作方法可以通过 getContext 方法获得当前的上下文。

## Implicit action expressions

大多数情况下，Parboiled 可以自动识别你的规则定义中哪些是动作表达式。比如下面的规则定义中包含了一个隐式的动作表达式：

```java
Rule Number() {
    return Sequence(
        OneOrMore(CharRange('0', '9')),
        Math.random() < 0.5 ? extractIntegerValue(match()) : someObj.doSomething()
    );
}
```

Parboiled 的检测逻辑如下：

BaseParser 中所有的默认规则创建器方法都拥有通用的 Java Object 参数，Java 编译器会自动将原始布尔表达式的结果作为一个 Boolean 对象传递。这是通过在布尔动作表达式的代码之后隐式地插入对 Boolean.valueOf 的调用来实现的。Parboiled 会在你的规则方法字节码中查找这些调用然后将其当做隐式动作表达式来处理，如果其结果直接被用作规则创建方法的参数的话。也可以通过 `@ExplicitActionsOnly` 注解(定义在解析类或规则方法上)来关闭该功能。

## Return Values

动作表达式均为布尔表达式。其返回值将影响对当前值的解析进度。如果动作表达式的结果为 false，解析将继续，就像替换动作表达式的假设解析规则失败一样。因此，你可以将动作表达式视作可以(匹配)成功或(匹配)失败的“规则”，具体则取决于其返回值。