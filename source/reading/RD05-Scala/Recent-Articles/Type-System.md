---
title: 类型系统
---

# 类型系统

**什么是类型系统(type-system)？**

*A type system is a tractable syntactic method for proving the absence of certain program behaviors by classifying phrases according to the kinds of values the compute.*  — Benjamin Pierce

一个类型系统是一种易于理解的语法方法，根据所计算的值的类别对短语进行分类，以证明某些程序行为的缺失。

**什么是类型(type)？**

- 一个类型定义了一个变量可以拥有的一组值，以及一组能够应用于这些值的函数；
- 一组值可以被定义为：
  - 笛卡尔产品类型(Cartesian product Types)，比如 `case` 类或元组；
  - 总和类型(Sum Types)，比如 `Either`。
- 类型可以是抽象的和(或)多态的。

**为什么需要类型？**

- *Make illegal states unrepresentable.* — Yaron Minsky
  - 使非法状态无法表示
- *Where static typing fits, do it every time because it has just fantastic maintenance benefits.* — Simon Peyton
  - 在合适的场景总是使用静态类型，因为它具有非常好的维护优势
- *Compiler can use Type informations to optimize compiled code.* 
  - 编译器可以使用类型信息来优化被编译的代码

**Scala 的类型系统？**

- Scala 是一种同时支持面向对象和函数式的多范式语言
- Scala 拥有**强大**的**静态**类型系统
  - 会在编译期检查类型
  - 编译器能够通过推断类型
  - 函数也可以作为类型：`A => B`

**类型的用途？**

类型可以用于定义：

- (抽象)类
- 对象
- 特质

**[Scala 中类型相关的特性？](http://stackoverflow.com/questions/7432146/what-is-scalas-powerful-type-system)**

## 设计目的

> 本部分整理至 Martin Odersky 的访谈，原文地址：[The Purpose of Scala's Type System](http://www.artima.com/scalazine/articles/scalas_type_system.html)，中文地址：[Scala类型系统的目的](http://www.infoq.com/cn/articles/scala-type-system)。

### 可伸缩性(scalability)

Scala 可以让你不必混用多种专用语言，无论是小型程序还是大型程序，无论是通用用途还是特定应用领域，Scala 都可以胜任。这样，可以避免你在多种语言环境中传递数据。

比如你想跨越数据边界传递数据，像 JDBC，从 Java 像数据库发起一次 SQL 查询，那么发出的查询最终将会是个字符串。这就意味着如果程序中只要有小小的拼写错误，在最终运行时，就会表现为一次非法查询，从而导致一系列相关错误。整个过程中编译器和类型系统不会告诉你那里写错了。

同时，如果仅使用一种语言，则只需要面对一套环境和工具。

### 可扩展性(extensibility)

“可伸缩性“的维度是”从小到大“。而可扩展性表示从”从通用用途到特定需求“。你可以强化 Scala 语言，使之涵盖你特别关注的领域。

比如数字类型，不同领域有很多不同的数字类型，密码学中的大数、商务人士用的十进制大数、科学家用的复数等等。但如果在语言中加入所有这些可能的类型，则会变得非常笨重。

因此我们可以留给外部库实现，但是又想像使用内置类型的代码一样干净、优雅。为此，你需要语言提供某些扩展机制，是你能够编写用起来不像库的库。

### 小规模编程中的类型

小规模编程中类型会显得不那么重要。类型的价值分布在一条长长的光谱上，一端表示超级有用，一端表示极度麻烦。通常说它麻烦是因为类型定义可能会太过冗余，要求你手动指定大量类型。有用则是因为，类型能避免运行时错误，能够提供 API 签名文档，能够为代码重构提供安全底线。

Scala 的类型推断，试图尽可能减少类型的麻烦之处。这意味着你编写脚本时并不需要涉及类型，系统会自动进行推断，同时编译器内部仍然会考虑类型。因此仍然能够享受类型检查带来的便利。

### 单元测试与随心所欲的表达式

仍然需要单元测试来测试你的程序逻辑。但相比动态类型语言，不需要额外对类型进行琐碎的单元测试，因此 Scala 中的单元测试也会精练很多。

另一条针对类型系统的反对意见是：静态类型对表达式限制太严。比如”我想更自由的表达自己，不想要类型系统的条条框框“。我(Martin)认为这种意见不靠谱，第一，Scala 的类型系统实际上很灵活，所以通常它可以让你用非常灵活的模式排列组合，而像 Java 这种类型系统偏弱的语言则难以实现；第二，通过模式匹配，你可以通过非常灵活的方式提取类型信息，甚至根本感觉不到类型信息的损失。

### 鸭子类型，Java 中的缺失

只要它具备我需要的功能，那么就可以把它当真。比如我需要某种支持关闭操作的资源，我可以以这种方式声明”它必须有 close 方法“，因此我不用关心它是 File、Channel 等等。

要想在 Java 中实现的话，需要定义一个包含该方法的通用接口，然后大家都需要实现这个接口。首先为了实现这一切就会带来大量接口和样板代码。其次，如果有了即成事实之后，你才想到要提个接口，那么几乎不可能做到。如果事先就写了一个类，鉴于这些类早已存在，那么你不改代码就无法加入该接口，除非修改所有客户端代码。所以，这一切都是类型系统强加给你的限制。

而 Scala 中则能表达鸭子类型。你完全可以用 Scala 把某一类型定义为：”任何拥有 close 方法并且该方法返回 Unit 的类型“。同时还可以把类型定义与其他约束结合起来。则可以这样定义类型：任何继承自某个类型，且拥有某某方法签名的类型。还可以这样定义，任何继承自某某类，且拥有某某类型的内部类的类型。从本质上讲，你可以通过定义类型汇总有哪些东西将为你所用，来描绘类型的结构。

### 既存类型(existential types)

既存类型已有 20 多年历史，它表示，给定某种类型，比如 List，但其内部元素是未知类型。你只知道它是由某种特定类型元素构成的 List，然而并不知道这个特定类型具体是哪种类型。这个概念在 Scala 中可以用既存类型来表达。比如：`List[T] forSome {type T}`。稍微有点笨重，这也算有意为之。因为事实证明，既存类型往往不大好处理。Scala 有更好的选择，因此不那么需要既存类型，因为 Scala 的类型可以包含其他类型作为内部成员。

归根结底，Scala 只有三种情况需要既存类型。

- Scala 需要能表示 Java 通配符的语义。
- Scala 需要能表示 Java 的 raw 类型，因为有许多库使用类非泛型类型，会出现 raw 类型。如果有一个 Java raw 类型 `java.util.List`，那么它其实是位置元素类型的 List。
- 既存类型可以把虚拟机中的实现细节反映到上层。类似 Java ，Scala 使用的泛型模型时”擦除类型“。所以在程序运行起来以后，我们就再也找不到类型参数了。之所以要进行擦除，是为了与 Java 的对象模型进行互操作。可是，如果我们需要反射，或者需要表达虚拟机的实现细节时该怎么办呢？我们需要有能力用 Scala 中的某些类型表示虚拟机的行为。

有了既存类型，即使某一类型中的某些方面尚不可知，我们仍然可以操作该类型。

比如，Scala 中的 List，我希望能够描述 head 方法的返回类型。在虚拟机级别，List 类型是 `List[T] forSome {type T}`。我们不知道 T 是什么。只知道 head 返回 T。既存类型理论告诉我们，该类型是”某些类型 T 中的 T“。也就相当于根类型 Object。那么我们从 head 方法中取回的就是 Object。因此 Scala 中我们要是知道更多信息，就可以直接指定具体类型而不用既存类型的限定规则。但是如果没有更多信息，我们就留着既存类型，让既存类型理论帮我们推断出返回类型。

如果 Java 采用的是”具现化“的泛型类型系统，不支持 raw 类型或通配符类型，那么 Scala 中的既存类型在意义不大，恐怕不会实现既存类型。

## Java 与 Scala 中的型变

Java 的型变是定义在使用通配符的代码之处，而 Scala 则是在类型定义之处，当然 Scala 的既存类型一样支持通配符，支持使用 Java 的写法，但并不推荐这么做。

首先，什么是”类型定义之处的型变“？当你定义某个类的类型参数时，例如 `List[T]`，会有一个问题，如果你给一个苹果列表，那这个列表算不算水果列表呢？显然算，只要苹果是水果的子类型，`List[Appke]`就是`List[Fruit]`的子类型，这称为协变。

但某些情况下种种关系不成立，比如我有一个变量，只能用来保存 Apple，那么这个变量就是对类型 Apple 的引用。这个变量并不能当做 Fruit 类型的引用，因为我不能把任意 Fruit 赋值给这个变量，它只能是 Apple。因此可以发现，上述子类型关系在有些情况下适用，有些则不适用。

Scala 中的解决方案是给类型参数添加一个标志。如果 List 中的 T 支持协变，则写作`List[+T]`。这将意味着任意 List 之间的关系都可以随着其 T 的关系而协变。要想支持协变，必须遵守协变的附加条件。举例来说，只有 List 内容不可变式，List 才能支持协变，否则将会遇到刚才引用变量中类似的问题，而导致无法协变。

Scala 中的机制是这样的：程序员可以声明”我觉得 List 应用支持协变“，即，要求 List 必须遵守子类型关系，然后给 List 的类型参数加上加号标志，仅需一次，List 的所有用户都可以使用。然后编译器会去找出 List 内的所有定义实际上是否兼容协变，确保 List 中不存在导致冲突的成员签名。

相比之下，以 Java 的方式使用通配符，这就意味着库的作者对协变爱莫能助，只能草草定义成 `List<T>`了事。但接下来如果用户需要协变 List，却不能写作`List<Fruit>`，而是`List<? extends Fruit>`。通配符就是这样用的。问题在于这是用户代码啊，用户总不能人人都像设计库的人那么专业吧。此外，这些标注之间，只要有一处不匹配，就会导致类型错误。

Scala 中可以在库中处理型变，让用户感觉不到型变存在，不必手动处理型变。

### 抽象类型成员

抽象类型与泛型参数看似类似，但拥有更多好处。对于抽象，业界有两套不同的机制：参数化和抽象类型成员。Java 也一样支持两套抽象，只不过 Java 的两套抽象取决于对什么进行抽象。Java 支持抽象方法，但不支持把方法作为参数；Java 不支持抽象字段，但支持把值作为参数；Java 不支持抽象类型成员，但支持把类型作为参数。所以在 Java 中三者均可抽象，但是之间的原理有所区别。

在 Scala 中试图把这些抽象支持的更完备、更正交，即对上述三类成员都采用相同的构造原理。因此，你可以使用抽象字段，也可以使用值参数；可以把方法(函数)作为参数，也可以声明抽象方法；即可以指定类型参数也可以声明抽象类型。至少在原则上，我们可以用同一种面向对象抽象成员的形式，表达全部三类参数。

抽象类型能够更好的处理先前谈到的协变问题。比如，一个 Ainmal 类，其中一个 eat 方法。问题是，如果从 Animal 派生一个类，比如 Cow，那么就能吃某一种食物，比如 Grass。Cow 不可以吃 Fish 之类的其他食物。你希望有办法可以声明 Cow 拥有一个 eat 方法，且该方法只能吃 Grass。实际上，这个需求在 Java 中实现不了，因为你一定会构造出有矛盾的情形，类似之前把 Fruit 复制给 Apple 一样。

Scala 中则可以增加一个类型成员，比如声明一个 SuitableFood 类型，它不定义具体是什么，这就是抽象类型，直接让 Animal 的 eat 方法吃下 SuitableFood 即可。然后在 Cow 中指定其为 Grass 即可。

所以，抽象类型提供了一种机制：先在父类中声明未知类型，稍后再在子类中填上某种已知类型。

你可能会说用参数也可以实现同样功能。确实可以。你可以给 Animal 增加参数，表示能吃的食物。但实践中，当你需要支持许多不同的功能是就会导致参数爆炸。而且通常更要命的问题是参数的边界。

## 相关概念

### 类型推断

Scala 拥有类型推断，这表示我们可以省略掉类型注解。

```scala
trait Thing
def getThing = new Thing { }

// without Type Ascription, the type is infered to be `Thing`
val infered = getThing

// with Type Ascription
val thing: Thing = getThing
```

在这些情况下都是可以编译通过的。所以都有哪些地方不能省略类型注解呢：

- 参数
- 公共方法返回值
- 递归或重载方法
- 需要包含类型签名来加快编译速度
- 需要类型签名来使代码更易读

### 统一的类型系统 — Any、AnyRef、AnyVal

我们之所以称 Scala 的类型系统是统一的，是因为它拥有一个顶层类型 Any。这与 Java 不同，它有一些原始类型(int/long/float/double/byte/char/short/boolean)并非扩展自 Java 中看起来像是顶层类型的 Object。

Any 作为顶层类型，下面有 AnyVal 和 AnyRef 两种子类型。

AnyRef 相当于 Java 中 对象的世界，对应于 Object，作为所有对象的超类。而 AnyVal 表示了 Java 中的 值世界，像 int 或其他 JVM 原始类型。

因此我们可以定义一个接收 Any 类型的方法，同时能够处理 Int 或 String。这对类型系统来说是透明的虽然在虚拟机层 Int 实例会被打包成一个对象。通过查看字节码能够发现：

```scala
Method scala/runtime/BoxesRunTime.boxToInteger:(I)Ljava/lang/Integer;
```

- Conventional types
- Value type classes
- Nonnullable type
- Monad types
- Trait types
- Singlton object types
- Compound types
- Functional Types
- Case classes
- Path-dependent types
- Anonymous types
- Self types
- Type aliases
- Package object
- Generic types
  - 类型擦除问题
- Pattern Match
- Bounded generic types
- Type Variance
- Higher kinded types
- Abstract types
- Existential types
- Implicit types
- View bounded types
- Structural types
- Dotty?

TODO: http://ktoso.github.io/scala-types-of-types/#type-of-an-code-object-code

##  社区评论

[Scala 的语言设计有哪些缺陷？](https://www.zhihu.com/question/28573046)

Scala 中提供了相当多的便利，但这与**类型系统**的强大无关，仅仅是因为与**类型**有关。而类型系统的强大之处在于其强大的表现力，能够表示在其它语言中无法表示的事情。

事实上，类型推断是类型系统效率的直接影响因素—它并非那么强大，有些语言拥有完整的类型推断，比如 Haskell。

Scala 中采用局部的基于流的推导，而不是全局式的 HM 推导，导致有些情况不如 Haskell 的推导那么强大。

[维基百科：类型推断](https://en.wikipedia.org/wiki/Type_inference)

[为什么有的语言不支持类型推断？](https://www.quora.com/Why-doesnt-every-programming-language-have-type-inference)

图灵完备：http://www.tuicool.com/articles/rqUjAb