---
title: 函数式与类型类
---

# 函数式与类型类

[翻译自原文：On Scala, Functional Programming and Type-Classes](https://alexn.org/blog/2012/11/02/scala-functional-programming-type-classes.html)

我曾经在 Coursera 上追随一个名为“ [Functional Programming Principles in Scala](https://www.coursera.org/course/progfun)”的精彩课程，该课程由 Martin Odersky(Scala 作者) 执教。这并不是我第一次遇到 Scala，因为我已经把它用在了日常工作当中。与此同时，我感觉需要找一个 Javascript 语言的替代者，因为优秀的 ClojureScript，我也开始了对 Clojure 的学习。

我对这两种语言都非常喜欢，真的说不上来更喜欢哪一个。这篇文档代表了我使用 Scala 的(菜鸟)经验，完全的瞎扯，或者你可以称它为“一个傻瓜的精神自慰”。

## 1. 函数式编程的双赢

它并非银弹，但整体来说非常棒。你真的有必要经历一次，同时撇开那些通过多年的必要技能而建立起的成见和偏见。学生学习起函数式编程会相对容易，他们并无任何经验，否则学习的过程将会很痛苦。

但在过去的 20 万年里我们进化的并不多，所以我们的大脑总是能在那些吸引我们内在兽性(inner-animal)的地方找到乐趣，对繁衍、吃饭、睡觉和躲避野兽感兴趣。学习是一种乐趣，但对于陌生的领域并非如此，因此如果你已经开始，那就要坚持下去。

首先我们需要一些对于函数式编程的定义：

- 通过“引用透明”对函数求值来处理计算；(引用透明：函数的行为类似数学函数，相同的输入总会得到相同的结果)
- 一个计算的最终输出是对输入的多次转换结果的组合，而非通过那些构建可变状态的方式；

一个函数式编程语言：

- 将函数当做“一类(first-class)对象”，这表示处理高阶函数不但是可能的，而且是很以很舒服的方式；
- 为你提供用于组合函数与类型的工具。

根据定义，像 Ruby、Javascript 这些也可以被认为是像样的函数式语言。然而我还要加几条：

- 拥有丰富的不可变、持久化数据结构；
- 提供有效的处理“[expression problem](http://en.wikipedia.org/wiki/Expression_problem)”的类型系统。Rich Hickey 称之为“*polymorphism a la carte*”

你也可以指定所有的副作用(side-effect)必须通过一元(monadic)类型来建模，不过这有点太清规戒律的意思(IMHO)，因为只有一种符合主流的语言 - Haskell。

## 2. Scala 是一个函数式语言吗

当然是。你只需要追随上面我提到的 Coursera 上的精彩[课程](https://www.coursera.org/course/progfun)、完成作业，你就会意识到 Scala 真正是一个非常函数式的语言。该课程虽短，不过有后续计划。因此现在就行动吧....

## 3. Polymorphism À la Carte

这是我从 Rich Hickey 那听来的名词，当他谈论到开放式类型系统(open type-system)，主要引用了 Clojure 的 Protocol 和 Haskell 的 Type-Class。

这些多态机制能够很好的解决表达式问题，这与我们已知的 Java、C++ 这些面向对象语言形成鲜明对比。

OOP 通常是一个封闭的类型系统(closed type-system)，特别是在静态语言中使用时。将一个新类添加到层级结构、添加新函数来操作整个层级结构、给接口添加新的抽象成员、使内置类型以某种方式运转，所有这些都难以处理。

Haskell 通过 [Type Classes](http://en.wikipedia.org/wiki/Type_class) 来处理。Clojure 通过  [Multi-Methods](http://en.wikipedia.org/wiki/Multiple_dispatch) 和 Protocol 来处理，Protocol 是动态的，相当于 动态类型系统(dynamic type-system)中的 type-class。

## 4. Yes Virginia，Scala 拥有 Type-Class

那什么又是 type-class？类似于 Java 中的接口，除了你可以使任何现有类型遵循它而不用修改该类型的实现。

比如，我们想要一个泛型函数能够将事物加起来....比如一个`foldLeft()`或`sum()`，但是相较于如何 fold，你想要环境知道如何处理每个特殊的类型。

在 Java 或 C# 中这样做有很多问题：

- 对于那些支持相加操作的类型，并没有为`+`定义接口，比如：Integer/BigInteger/BigDecimal/Float/String...
- 我们需要从一些"0"开始(你想要折叠的列表可能为空)

或许你可以定义一个这样的类型类：

```scala
trait CanFold[-T, R]{
  def sum(acc:R, elem:T): R
  def zero: R
}
```

但是等等，这不就是一个类 Java 的接口吗？对，他就是。这就是 Scala 最棒的地方，Scala 中任何实例都是对象，任何类型(type)都是一个类(class)。

那又是什么让这个接口成为了一个 type-class？当然是因为“伴生对象中带有隐式参数的对象”([Objects in combination with implicit parameters](http://ropas.snu.ac.kr/~bruno/papers/TypeClasses.pdf))。我们看一下如何使用这些来实现`sum`函数：

```scala
def sum[A, B](list: Traversable[A])(implicit adder: CanFold[A, B]): B = 
  list.foldLeft(addr.zero)((acc,e) => adder.sum(acc,e))
```

因此，如果 Scala 编译器能够在作用域中找到一个为 A 定义的 隐式`CanFold`，就会使用它生产一个 B。它的出色表现在多个级别：

- 类型 A 的隐式定义建立在返回类型 B 之上
- 可以为任何你需要的类型定义一个`CanFold`，整数、字符串、列表等等等

隐式定义是有范围的，因此需要导入。如果你需要一些类型的默认隐式定义(全局可见)，可以在`CanFold`特质的伴生对象中定义：

```scala
object CanFold{
  // default implementation for integers
  implicit object CanFoldInts extends CanFold[Int, Long] {
    def sum(acc:Long, e:Int) = acc + e
    def zero = 0
  }
}
```

使用时则和预期一样：

```scala
// notice how the result of summing Integers is a Long
sum(1 :: 2 :: 3 :: Nil)
//=> Long = 6
```

我不会骗你这些方式有多难学或者如何学，你最终会拉起头发，期盼这些都不再是问题的动态类型。然而你要分清 hard 和 complex 的区别，前者是相对的、主观上的，后者是绝对的、可观上的。

我们实现中的一个难题是如何为一个基本类型提供默认实现。这也是为什么在`CanFold[-T,R]`的定义中我们将类型参数 T 设为逆变(*contravariant*)。逆变性代表的意思是：

```scala
if B inherits from A (B <: A), then
CanFold[A, _] inherits from CanFold[B, _] (CanFold[A,_] <: CanFold[B,_])
```

这允许我们为任何 Traversable 定义一个 CanFold，该 CanFold 可以支持任何 Seq/Vector/List 等等。

```scala
implicit object CanFoldSeqs extends CanFold[Traversable[_], Traversable[_]] {
  def sum(x:Travrsable[_], y:Travsesable[_]) = x ++ y
  def zero = Traversable()
}
```

这可以将任何类型的`Traversable`相加。问题是会在过程中丢失类型参数：

```scala
sum(List(1,3,4) :: List(4,5) :: Nil)
//=> Traversable[Any] = List(1,2,3,4,6)
```

为什么我会说它难的原因是在我把头发拉出来之后，不得不去[StackOverFlow](http://stackoverflow.com/questions/13176697/problems-with-contravariance-in-scala)请教怎么才能够返回一个`Traversable[Int]`。因此，你可以使用一个隐式的`def`替换之前的具体隐式对象，来帮助编译器识别容器中嵌入的类型：

```scala
implicit def CanFoldSeqs[A] = new CanFold[Traversable[A], Traversable[A]] {
  def sum(x: Traversable[A], y:Traversable[A]) = x ++ y
  def zero = Traversable()
}

sum(List(1,2,3) :: List(4,5) :: Nil)
//=> Traversable[Int] = List(1,2,3,4,5)
```

Implicit 比眼见的要灵活。显然编译器同样能够使用返回你需要的实例的函数，而不是具体的实例。作为一个旁注，我上面做的是很难的，甚至在 Haskell 中，因为子类化(sub-typing)是复杂的，但是 Clojure 中也很简单，因为你无需关注返回类型。

**NOTE：上面的实现并不严谨，可能会发生冲突。**

未完...

