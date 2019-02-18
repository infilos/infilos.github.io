---
title: 泛型型变
---

# 泛型型变

## 里氏替换原则

> 所有引用父类型的地方必须能透明地使用其子类型的对象。

这里面暗含了一条重要的信息：所有你想由父类完成的操作，子类都能够完成，而且子类能比父类做的更多。

## 概念

- 协变(covariance)、逆变(contravariance)、不可变(invariant) 统称为型变(variance)。
- 型变表述了：具有父子类型关系的类型经过“类型转换”后，所构造出的更复杂的对应类型之间的关系是如何变化的。
- 协变：具有父子类型关系的类型，经过“类型转换”所构造的复杂类型之间仍然保持着相同的父子类型关系。
- 逆变：具有父子类型关系的类型，经过“类型转换”所构造的复杂类型之间保持着与原类型之间相反的父子关系。
- 不变：不支持协变或逆变。

型变描述的是一组类型转换规则。

## 公式

假设两种类型 X 和 Y，`<:` 表示子类型关系，`>:` 表示父类型关系，即 `X <: Y` 表示 X 是 Y 的子类。`f` 表示类型转换，即一个类型构造操作，通过传入一个类型来构造一个新类型。因此 X、Y 对应的构造类型分别为 `f(X)` 和 `f(Y)`。

因此，型变描述的是：基于具有父子关系的 X 和 Y，通过 `f` 构造而成的 `f(X)` 和 `f(Y)`，`f` 是如何影响 `f(X)` 和 `f(Y)` 之间的父子关系的。

规则如下：

- 如果 `X <: Y`, 经过 `f` 构造类型之后之后 `f(X) <: f(Y)`, `f` 具有协变性。
- 如果 `X <: Y`, 经过 `f` 构造类型之后之后 `f(X) >: f(Y)`, `f` 具有逆变性。
- 如果以上两种情况都不符合，则类型构造、或称类型转换 `f` 具有不可变性。

## 容器类型

这里的容器类型泛指集合类型，比如 `Array`。

### 协变

假设以下类型：`FujiApple <: Apple <: Fruit, Orange <: Fruit`。

根据协变规则的定义，子类型数组可以赋值给一个类型为父类型数组的变量，这里使用的 `f` 类型构造即为 Java 中的 `Array` 类型，这是一个协变类型：

```java
Fruit[] fruits = new Apple[0];	// 1
fruits[0] = new Apple();		// 2
fruits[1] = new FujiApple();	// 3

fruits[2] = new Fruit();		// 4
fruits[3] = new Orange();		// 5
```

首先我们可以确认两点：

- 从 `fruits` 中取出的元素一定是 `Fruit` 类型；
- 在编译期，编译器绝对允许我们将 `Fruit` 及其子类型的元素填充到 `fruits` 协变数组，因为 `fruits` 的类型为 `Fruit[]`，可以包含任意 `Fruit` 及其子类型的元素。

从协变数组 `fruits` 中读取元素是绝对安全的，无论是编译期还是运行时，都不会出现任何问题。上面的操作中，`fruits` 实际引用的是一个 `Apple` 数组，即便取出的是 `Apple` 元素，但仍然是 `Fruit` 类型。

但是将 `Fruit` 类型或其 `Fruit` 类型的子类型元素填充到协变数组 `fruits` 时，可能会在运行时出现问题。这时 `fruits` 实际上引用的是一个 `Apple[]` 类型数组，填充 `Apple` 或 `Apple` 子类型之外的任意类型的元素都是不合法的，最终会引起 `ArrayStoreException`。

上面的代码中，如果 `fruits` 是其他接口返回的数组，我们仅知道它是一个 `Fruit[]` 类型的数组，因此可以按照里氏替换原则向其填充 `Fruit` 及其子类型元素，编译器也不会抛出错误。但如果像上面的代码中的应用一样实际上引用的是一个子类型数组，则最终会引发运行时异常。

**因此，在实际使用或定义协变容器类型时，总是将其当做一个只读容器是绝对安全的。**

### 逆变

Java 实际上是不支持逆变数组的，但这里我们可以假设一下支持逆变的情况：

```java
Fruit[] fruits = new Fruit[10]{ new Apple(), new Orange(), new FujiApple() }
Orange[] oranges = fruits;	// 应用逆变特性
oranges[0] = new Orange();	// 1
Orange orange = oranges(2);	// 2
```

这里我们将父类型数组 `fruits` 赋值给类型为子类型数组的变量 `oranges`，如果这样的操作理论上成立，即支持逆变，也同样会出现问题。

这里假如我们从 `oranges` 数组读取一个元素，因为该数组为 `Orange[]`，则其返回的元素一定会是 `Orange` 类型，但实际上并非如此，它实际上引用的是一个 `Fruit[]` 数组，元素类型可能是 `Fruit` 的任何其他子类型。

**因此，在实际使用或定义逆变容器类型时，总是将其当做一个只写容器是绝对安全的。**

### Scala：声明点型变(declaration-site variance)

```scala
trait List[+T]

trait A
trait B extends A
```

这时，`List[A]` 即为 `List[B]` 的父类型。

### Java：使用点型变(use-site variance)

在 Java 中，实际上参数化类型是不可型变的，比如 `List<String>` 并非 `List<Object>` 的子类型，即便 `String` 实际是 `Object` 的子类型。但有些时候需要一些比类型型变更多的灵活性。

> Java 中的泛型并没有对协变的内建支持，因此 Java 泛型具有不可变型性。但是，为了兼容遗留代码而被保留下来的原生类型确实可以做出这样的行为，但同时我们也非常清楚使用它就像使用数组的协变性一样意味着代码变得不再安全。所以，制定Java标准的那群人想出了个法子使得泛型像数组一样具有这些特性(Java 中的数组具有协变性)，同时这种代替原生类型的方案必须是绝对安全的：他们通过给泛型增加通配符特性使得泛型在参数化后具有协变性或逆变性。

因此提供了一种方式来实现型变：

```java
List<? extends Object> list = new ArrayList<String>();
```

这里，将 `List<String>` 赋值给 `List<Object>`，实际上表示了他们之间的父子类型关系。

Java 中提供的这种特性称为“类型界限通配符”：

- `? extends T`：T 表示类型通配符的上界。即由 `?` 表示的未知类型参数必须是 `T` 的子类，或 `T` 本身。
- `? super T`：T 表示类型通配符的下界。即由 `?` 表示的未知类型参数必须是 `T` 的父类，或 `T` 本身。

但是何时使用这两种通配符呢，在 Effective Java 中(3rd, Item-31)详细解释了 PECS 原则：

> **P**roducer use "**E**xtends" and **C**onsumer uses "**S**uper".

这里的 Extends 和 Super 对应类型边界声明符 `extends` 和 `super`。为了更大的灵活性，在方法的输入参数中使用通配符类型来表示生产者或消费者。即从一个容器(集合)类型的视角出发：

- 如果仅需要从集合中读取元素，这时集合作为一个生产者，应该在定义集合时使用 `extends` 来声明类型上界；
- 如果仅需要向集合中填充元素，这时集合作为一个消费者，应该在定义集合时使用 `super` 来声明类型下界。
- 如果同时需要读取和填充操作，则需要制定精确的类型，不能使用上下界来修饰类型参数。

```java
public class Stack<E> { 
  public Stack(); 
    public void push(E e); 
    public E pop(); 
    public boolean isEmpty(); 
    
    public void pushAll(Iterable<? extends E> src) {	// 1
    for (E e : src)
      push(e); 
  }
    
    public void popAll(Collection<? super E> dst) {		// 2
    while (!isEmpty())
      dst.add(pop()); 
        }
  }
}
```

1. `src` 作为生产者集合，所包含的元素均为 E 的子类或 E，这时可以对一个 `Stack<Number>` 调用 `pushAll` 传入一个 `Iterable<Int>`。
2. `dst` 作为消费者集合，所包含的元素均为 E 的父类或 E，这时可以对一个 `Stack<Number>` 调用 `popAll` 传入一个 `Iterable<Object>`。

## 返回值与参数中的型变

这种情况的典型应用便是 Scala 中的 `Function1` 接口：

```scala
trait Function1[-T1, +R] extends AnyRef
```

在 Scala 中，`-T` 表示逆变，`+T` 表示协变，`T` 表示不变。

假定我们拥有以下类型：`Garfield <: Cat <: Animal`, `Husky <: Dog <: Animal`。对于 `Function1[Cat, Dog]` 来说，其子类必须保证两个功能：

1. 你可以向其传入任意类型的 `Cat`，即 `Cat` 的的所有子类；
2. 你可以使用其返回值调用 `Dog` 的任意方法。

### 逆变参数

假如参数是协变的，即 `Function1[Garfield, _] <: Function1[Cat, _]`，根据里氏替换原则，任何需要父类的地方都可以使用一个子类来替换，这里作为父类的 `Function1[Cat, _]` 可以接收任意类型的 `Cat`，但作为子类的 `Function1[Garfield, _]` 仅能接收 `Garfield`。因此没有意义。

假如参数是逆变的，即 `Function[Animal, _] <: Function[Cat, _]`，根据里氏替换原则，这里作为父类的 `Function1[Cat, _]` 可以接收任意类型的 `Cat`，同样作为子类的 `Function[Animal, _]` 也可以接收更多类型的 `Cat`，而且不仅仅是 `Cat`，甚至是 `Dog` (子类能做的更多)。

因此对于 `A <: B` 来说，要求 `Function1[B, _] <: Function1[A, _]`。

### 协变返回值

假如返回值是协变的，即 `Function[_, Husky] <: Function[_, Dog]`，根据里氏替换原则，这里作为父类的 `Function[_, Dog]` 可以在其返回值上调用任意 `Dog` 的方法，而作为子类的 `Function[_, Husky]` 同样可以在其返回值上调用任意 `Dog` 的方法，甚至还能调用 Husky 特有的方法 `destoryHouse`(子类能做的更多)。

假如返回值是逆变的，即 `Function[_, Animal] <: Function[_, Dog]`，根据里氏替换原则，这里作为父类的 `Function[_, Dog]` 可以在其返回值上调用任意 `Dog` 的方法，而作为子类的 `Function[_, Animal]` 则可能无法执行 `Dog` 特有的方法，因此没有意义。

### 反向推导

因此，对于一个 `Function1[Animal, Husky]`，它可以完成 `Function1[Cat, Dog]` 所能完成的所有工作：可以传入任意类型的 `Cat`, 甚至是其他 `Animal`，可以调用 `Dog` 的任意方法，甚至是 `Husky` 特有的方法。总的来说，`Function1[Animal, Husky] <: Function1[Cat, Dog]`。因此对于 `Function1[Cat, Dog]` 本身来说，其参数类型可以替换为父类(逆变)，返回值类型可以替换为子类(协变)。
