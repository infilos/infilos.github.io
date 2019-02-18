---
title: Monoid
---

# Monoid

> 整理自《Functional Programming In Scala》第十章。

Monoid(幺半群) 是一个代数定义，是*纯代数(purely algebraic)*的一种，它简单、普遍存在且很实用。除了满足同样的代数法则外不同 Monoid 实例之间很少有关联，但这种代数结构定义了用于实现实用的多态函数所必须的所有法则。

操作列表、连接字符串或在循环中累加都可以被拆解成 Monoid。下面介绍它在两个方面的使用方式：将问题拆分成小部分然后并行计算；将简单的部分组装成复杂的计算。

## 1. 什么是 Monoid

比如在字符串拼接的代数表达中，“foo” + “bar” 得到“foobar”，空串是这个操作的*单位元(identity)*(或称“幺元”)元素，即 “”+s 与 s + “” 的值都是 s。进一步，如果将三个串相加，r + s + t，由于这个操作是可结合的(associative)，因此 (r +s) +t 与 r + (s+t) 的结果是一样的。

该规则同样适用于整数相加，它也是可结合的。(x+y)+z 与 x +(y+z) 的结果相同，而且由一个单位元素 0，它去其他整数相加时不会影响结果。同样乘法也是一样，它的单位元元素是 1。

布尔操作符 || 和 && 同样是可结合的，它们的单位元元素是 true 和 false。

像这样的代数便成为 Monoid，结合律(associativity)和同一律(identity)则一起被称为*monoid法则*。一个 Monoid 由如下几部分构成：

- 一个类型 A；
- 一个可结合的二元操作 OP，接收两个参数后返回相同类型的值，对于任何`x:A, y:A, z:A`来说，`OP(OP(x,y), z)`和`OP(x, OP(y,z))`是等价的；
- 一个值`zero:A`，它是一个单位元，对于任何`x:A`来说，`zero`与它的操作都等于 x 自身：`OP(x, zero) == x`或`OP(zero, x) == x`。

可以使用 Scala 表示：

```scala
trait Monoid[A] {
  def op(a1:A, a2:A): A
  def zero: A
}
```

然后是 String 实例：

```scala
val stringMonoid = new Monoid[String] {
  def op(a1:String, a2:String) = a1 + a2
  def zero:String = ""
}
```

或者是 List 的连接：

```scala
// 注意这里是 def，一个返回 Monoid 实例的函数，否则将丢失类型参数 A
def listMonoid[A] = new Monoid[List[A]] {
  def op(a1:List[A], a2:List[A]) = a1 ++ a2
  def zero: Nil
}
```

> “有”一个 Monoid，还是“是”一个 Monoid：
>
> 当程序员和数学家讨论：一个类型是 Monoid，或，有一个 Monoid实例，有两种不一致的表述方式。程序员易于认为一个`Monoid[A]`的实例是 Monoid，但这并不准确。Monoid 实际上是类型和定义法则的实例。更准确的说是类型 A 和 `Monoid[A]`实例定义的操作构成了一个 Monoid。

**一个类型、一个此类型的二元操作(满足结合律)、一个单位元元素，这三者构成一个 Monoid。**

## 2. 使用 Monoid 折叠列表

Monoid 和列表联系紧密，从 List 的`foldLeft`/`foldRight`签名中可以发现参数的类型很特别：

```scala
def foldRight[B](z: B)(f: (A, B) => B): B
def foldLeft[B](z: B)(f: (B, A) => B): B
```

当 A 和 B 类型一样时：

```scala
def foldRight(z: A)(f: (A, A) => A): A
def foldLeft(z: A)(f: (A, A) => A): A
```

如果一个字符串的列表，可以传递 StringMonoid 中的 OP 和 zero，用于将字符串进行拼接：

```scala
val words = List("Hic", "Est", "Index")
val s = words.foldRight(stringMonoid.zero)(stringMonoid.op)	// "HicEstIndex"
val t = words.foldLeft(stringMonoid.zero)(stringMonoid.op)	// "HicEstIndex"
```

会发现两个操作的结果一样，这正是因为结合律与同一律法则，无论左右结合效果都一样。

```scala
words.foldRight("")(_ + _) == (("" + "Hic") + "Est") + "Index"
words.foldLeft("")(_ + _) == "Hic" + ("Est" + ("Index" + ""))
```

可以编写一个通用的 concatenate 函数，使用 Monoid 去折叠列表：

```scala
def concatenate[A](as:List[A], m:Monoid[A]) :A = as.foldLeft(m.zero)(m.op)
```

但是假如列表中的元素类型不是 Monoid 实例该如何处理呢，总是可以将列表 map 成另外的类型：

```scala
def foldMap[A, B](as:List[A], m:Monoid[B])(f: A => B): B
```

## 3. 结合律与并行化

Monoid 操作的结合律意味着可以自由选择如何进行数据结构的折叠操作。前面展示了使用列表的 foldLeft 和 foldRight 去调用满足结合律的函数，对列表按照顺序向左或向右的 reduce。如果有个 Monoid 可以使用*平衡折叠法(balance fold)*对列表进行 reduce，这样一些操作可能更加高效或支持并行化。

假设一个有序集 a, b, c, d，三种不同的折叠方式：

```scala
op(a, op(b, op(c, d)))	// foldRight
op(op(op(a, b), c), d)	// foldLeft
op(op(a, b), op(c, d))	// balance fold
```

在平衡折叠中，因为两个 op 是独立的，因此支持同时运行。当每个 op 的时间花费与参数的长度成正比时平衡树的结构可以变得更加高效，比如下面的表达式：

```scala
List("lorem", "ipsum", "dolor", "sit").foldLeft("")(_ + _)
```

其求值轨迹为：

```scala
List("lorem", "ipsum", "dolor", "sit").foldLeft("")(_ + _)
List("ipsum", "dolor", "sit").foldLeft("lorem")(_ + _)
List("dolor", "sit").foldLeft("loremipsum")(_ + _)
List("sit").foldLeft("loremipsumdolor")(_ + _)
List().foldLeft("loremipsumdolorsit")(_ + _)
"loremipsumdolorsit"
```

每次折叠，分配一个临时的字符串(foldLeft 的第一个参数)然后丢弃，下次又要分配一个更大的字符串。字符串的值是不变的，当 a + b 时，需要分配一个字符数组然后将 a 和 b 的值复制到这个新数组。这个时间花费与 a、b 的总长度是成正比的。相比更高效的方式是对半组合顺序集，先构建“loremipsum”和“dolorsit”，然后将他们加在一起。

## 4. 例子：并行解析

如果需要统计字符串中的单词数，可以按顺序扫描字符串，寻找空格然后对连续的非空格字符计数。这样按顺序解析，解析器的状态可以表达成最后一个字符是否是空格。

但是如果要处理一个巨大的文件，达到单机内存装不下，需要对文件进行切分才能处理。策略是将文件拆分成多个可以管理的块(chunk)，并行处理这些块，最后将结果合并起来。这时，解析器的状态可能会复杂一些，需要可以合并中间结果，无论这个部分是文件的开头、中间或结尾。这意味这合并操作需要时可结合的。

把下面的字符串当做一个大文件：

```scala
"lorem ipsum dolor sit amet"
```

假如对半拆分字符串，可能会将一个单词拆分。当累加这些字符串的计算结果时需要避免重复计入同一个单词。所以这里仅仅将单词作为整体来计数是不严谨的。需要一个数据结构能处理部分结果，并能记录完整的单词。单词计数的结果则可以表示成一个代数数据结构：

```scala
sealed trait WC
case class Stub(chars: String) extends WC
case class Part(lStub:String, words:Int, rStum:String) extends WC
```

`Stub`表示没有看到任何完整的单词，`Part`保存看到的完整的单词的个数，以及左边的部分单词和右边的部分单词。

比如上面的字符串，拆分成“lorem ipsum do”和“lor sit amet”，对前者计数的结果为`Part("lorem", 1, do)`，对后者的计数结果为`Part("lor", 2, "")`。

> Monoid 同态
>
> 可能你会发现 Monoid 的函数之间有个法则。比如字符串的连接 Monoid 和整数累加 Monoid。假如取两个字符串的长度相加，等于连接两个字符串然后取其长度：
>
> ```scala
> "foo".length + "bar".length == ("foo" + "bar").length
> ```
>
> `length`是一个函数，它将 String 转化为 Int 并保存 Monoid 结构。这样的结构称为**Monoid 同态(homomorphism)**，一个 Monoid 同态 f 定义为在 Monoid M 和 N 之间对所有的值及 x、y 都遵守以下规则：
>
> ```scala
> M.op(f(x), f(y)) = f(N.op(x, y))
> ```
>
> 当设计自己的库时这个特性很有用，加入两个类型是 Monoid 并且他们之前存在函数，好的做法是考虑这个函数是否可以保持 Monoid 结构，并且测试其是否为 Monoid 同态。
>
> 某些时候两个 Monoid 之间是双向同态的，**同质(isomorphic)**是在 M 和 N 之间存在的两个同态的函数 f 和 g，而且`f andThen g`和`g andThen f`是等同的函数。
>
> 比如， String 和 List[Char] monoid 的连接操作是同质的。两个 Boolean monoid (false, ||) 和 (true, &&)通过取反(!)同样也是同质的。

## 5. 可折叠数据结构

现在需要为 IndexedSeq 实现一个折叠函数。一般处理这类数据结构中的额数据时，通常不在意具体结构是什么，也不在意是否延时或者提供有效的随机读写，等等。

比如有个结构中是整形，需要计算他们的总和，可以使用 foldRight：

```scala
ints.foldRight(0)(_ + _)
```

不需要关心 ints 的具体结构类型，他可以是 Vector、Stream 或其他列表，或者任何一个包含 foldRight 方法的类型。把这种通用性表达成下面的 trait：

```scala
trait Foldable[F[_]] {
  def foldRight[A,B](as:F[A])(z:B)(f: (A,B) => B): B
  def foldLeft[A,B](as:F[A])(z:B)(f: (B,A) => B): B
  def foldMap[A,B](as:F[A])(f: A => B)(mb:Monoid[B]): B
  def concatenate[A](as:F[A])(m:Monoid[A]): A = foldLeft(as)(m.zero)(m.op)
}
```

这里抽象出一个类型构造器 F，就像在之前章节中构建的 Parser 类型。表示为`F[_]`，这里的下划线表示 F 不是一个类型而是一个类型构造器，它接收一个类型参数。就像接收别的函数作为参数的函数被称为高阶函数，Foldable 是**高阶类型构造函数**或**高阶类型**。

## 6. Monoid 组合

Monoid 的真正强大之处在于组合。比如 类型 A、B 是 Monoid，那么 Tuple 类型 (A, B) 也是 Monoid (称 product)。

### 6.1. 组装更加复杂的 Monoid

只需要包含的元素是 monoid，某些数据结构就能构建成 Monoid。比如当 value 类型是 Monoid 时，合并 key-value 映射的操作就能够构建 monoid：

```scala
def mapMergeMonoid[K, V](V: Monoid[V]): Monoid[Map[K, V]] = {
  new Monoid[Map[K, V]] {
    def zero = Map[K, V]()
    def op(a:Map[K,V], b:Map[K,V]) = 
      (a.keySet ++ b.keySet).foldLeft(zero) { (acc, k) =>
        acc.updated(k, v.op(a.getOrElse(k, V.zero), b.getOrElse(k, V.zero)))
      }
  }
}
```

使用这个简单的组合子(combinator)既可以组装出复杂的 Monoid：

```scala
val m:Monoid[Map[String, Map[String, Int]]] = 
  mapMergeMonoid(mapMergeMonoid(intAddition))
```

### 6.2.使用组合的 Monoid 融合多个遍历

多个 Monoid 可以被组合在一起，则折叠数据时可以同时执行多个计算。比如同时获得一个列表的总和与长度，来计算平均值：

```scala
val m = prodocutMonoid(intAddition, intAddition)
val p = listFoldable.foldMap(List(1,2,3,4,5))(a => (1,a))(m)
val mean = p.1 / p.2.toDouble	//=> 2.5
```

## 7. 总结

Monoid 是第一个纯抽象代数，它定义为抽象的操作和对应的法则。可以在不知道参数是什么，仅知道其类型可以构建 monoid的情况下编写可用的函数。