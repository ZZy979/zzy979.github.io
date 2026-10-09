---
title: Scala基础教程 第5节 泛型
date: 2026-04-12 20:13:02 +0800
categories: [Scala]
tags: [scala, generic programming, covariance, type bound]
---
## 5.1 泛型类
<https://docs.scala-lang.org/tour/generic-classes.html>

**泛型类**(generic class)是接受类型参数的类。它们对于集合类特别有用。

### 5.1.1 定义泛型类
泛型类在类名后的`[]`内声明类型参数。

```scala
class Stack[A] {
  private var elements: List[A] = Nil

  def push(x: A): Unit = {
    elements = x :: elements
  }

  def peek: A = elements.head

  def pop(): A = {
    val currentTop = peek
    elements = elements.tail
    currentTop
  }
}
```

这段代码实现了一个接受类型参数`A`的泛型类`Stack`。这意味着底层列表`elements`只能存储`A`类型的元素。`push()`方法只接受`A`类型的对象。

注：
* `x :: elements`创建了一个新列表，将`x`附加到当前的`elements`之前。
* `Nil`表示空列表，不要与`null`混淆。

### 5.1.2 使用泛型类
要使用泛型类，将类型放在`[]`中以替换`A`。

```scala
val stack = new Stack[Int]
stack.push(1)
stack.push(2)
println(stack.pop())  // prints 2
println(stack.pop())  // prints 1
```

如果类型参数有子类型，也可以向`push()`方法传递子类型的对象：

```scala
class Fruit
class Apple extends Fruit
class Banana extends Fruit

val stack = new Stack[Fruit]
val apple = new Apple
val banana = new Banana

stack.push(apple)
stack.push(banana)
```

类`Apple`和`Banana`都扩展了`Fruit`，因此可以将实例`apple`和`banana`压入`Fruit`栈中。

注意：泛型类的继承是**不变的**(invariant)，即`Stack[A]`是`Stack[B]`的子类型当且仅当`A = B`。这意味着不能将`Stack[Apple]`类型的对象赋给`Stack[Fruit]`类型的变量。但是Scala提供了一种类型参数注释机制来控制泛型类型的继承行为，详见5.3节。

## 5.2 泛型方法
<https://docs.scala-lang.org/tour/polymorphic-methods.html>

在Scala中，方法也可以接受类型参数。语法与泛型类相似，类型参数用方括号括起来，放在方法名和参数列表之间。

下面是一个示例：

```scala
def listOfDuplicates[A](x: A, length: Int): List[A] = {
  if (length < 1)
    Nil
  else
    x :: listOfDuplicates(x, length - 1)
}

println(listOfDuplicates[Int](3, 4))  // List(3, 3, 3, 3)
println(listOfDuplicates("La", 8))  // List(La, La, La, La, La, La, La, La)
```

方法`listOfDuplicates()`接受类型参数`A`以及值参数`x`和`length`，`x`的类型为`A`。该方法返回长度为`length`、元素全部是`x`的列表。

在第一个调用中，我们显式提供了类型参数`Int`。因此第一个参数必须是`Int`，返回类型是`List[Int]`。

第二个调用省略了类型参数，编译器通常可以根据上下文或值参数的类型来推断它。在这个例子中，`"La"`是一个`String`，因此编译器知道`A = String`。

## 5.3 协变和逆变
<https://docs.scala-lang.org/tour/variances.html>

变型(variance)让你能够控制类型参数在继承关系中的行为。Scala支持对泛型类的类型参数进行标注，使其可以是**协变**(covariant)、**逆变**(contravariant)或**不变**(invariant)（如果没有任何标注）。在类型系统中使用变型使我们能够在复杂类型之间建立直观的联系。

```scala
class Foo[+A] // A covariant class
class Bar[-A] // A contravariant class
class Baz[A]  // An invariant class
```

注：Java的泛型类都是不变的。Java的通配符类型可以实现类似于Scala变型的功能。详见[《Java核心技术》笔记 卷I 第8章]({% post_url 2024-10-29-java-note-v1ch08-generic-programming %}) 8.7和8.8节。

### 5.3.1 不变
默认情况下，Scala的类型参数是**不变的**(invariant)：类型参数之间的继承关系不会反映在泛型类中。考虑下面的泛型类`Box`：

```scala
class Box[A](var content: A)
```

我们将在其中存放`Animal`类型的对象，其定义如下：

```scala
abstract class Animal {
  def name: String
}

case class Cat(name: String) extends Animal
case class Dog(name: String) extends Animal
```

`Cat`和`Dog`都是`Animal`的子类型，这意味着以下赋值是合法的：

```scala
val myAnimal: Animal = Cat("Felix")
```

但是，`Box[Cat]`并不是`Box[Animal]`的子类型。如果试图将一个`Box[Cat]`对象赋给`Box[Animal]`变量，编译器将会报错：

```scala
val myCatBox: Box[Cat] = new Box[Cat](Cat("Felix"))
val myAnimalBox: Box[Animal] = myCatBox // this doesn't compile
val myAnimal: Animal = myAnimalBox.content
```

假设这样做是合法的，由于`Box`的`content`字段是可变的，就可以将`myAnimalBox`的内容替换为一个`Dog`：

```scala
myAnimalBox.content = Dog("Fido") // OK, Dog is an Animal
```

但是，变量`myAnimalBox`实际引用了一个`Box[Cat]`对象。此时如果获取`myCatBox`的内容，期望是一个`Cat`，但实际是一个`Dog`，这就破坏了类型可靠性！

```scala
val myCat: Cat = myCatBox.content // myCat would be a Dog!
```

由此可以得出结论：即使`Cat`是`Animal`的子类型，`Box[Cat]`和`Box[Animal]`也不能有子类型关系。

### 5.3.2 协变
上面遇到的问题是：因为可以把`Dog`放入`Box[Animal]`，所以`Box[Cat]`不能是`Box[Animal]`的子类型。

但是，如果不能把`Dog`放入`Box[Animal]`（即`Box`是不可变的），那么就可以保证从`Box[Cat]`获取到的一定是`Cat`。这样就可以让`Box[Cat]`是`Box[Animal]`的子类型：

```scala
class ImmutableBox[+A](val content: A)

val myCatBox: ImmutableBox[Cat] = new ImmutableBox[Cat](Cat("Felix"))
val myAnimalBox: ImmutableBox[Animal] = myCatBox // now this compiles
```

我们说`ImmutableBox`对`A`是**协变的**(covariant)，由`A`前面的`+`表示。

一般地，给定`class Cov[+T]`，如果`A`是`B`的子类型，则`Cov[A]`是`Cov[B]`的子类型。

在下面的例子中，`printAnimalNames()`方法接受一个`List[Animal]`参数，并打印它们的名字。如果`List[A]`不是协变的，最后两个方法调用将编译失败，这将严重限制`printAnimalNames()`方法的有用性。

```scala
def printAnimalNames(animals: List[Animal]): Unit = {
  animals.foreach(animal => println(animal.name))
}

val cats: List[Cat] = List(Cat("Whiskers"), Cat("Tom"))
val dogs: List[Dog] = List(Dog("Fido"), Dog("Rex"))
printAnimalNames(cats)  // prints: Whiskers, Tom
printAnimalNames(dogs)  // prints: Fido, Rex
```

### 5.3.3 逆变
在上一节已经看到，如果确保只读、不写，就可以定义协变类型。反过来，如果只写、不读呢？假设有一个序列化器，接受`A`类型的值并将其转换为序列化格式，就会出现这种情况。

```scala
abstract class Serializer[-A] {
  def serialize(a: A): String
}

class AnimalSerializer extends Serializer[Animal] {
  override def serialize(animal: Animal): String = s"""{"name": "${animal.name}"}"""
}

val animalSerializer: Serializer[Animal] = new AnimalSerializer

val catSerializer: Serializer[Cat] = animalSerializer
println(catSerializer.serialize(Cat("Felix"))) // prints: {"name": "Felix"}
```

我们说`Serializer`对`A`是**逆变的**(contravariant)，由`A`前面的`-`表示。`Serializer[Animal]`是`Serializer[Cat]`的子类型。

一般地，给定`class Contra[-T]`，如果`A`是`B`的子类型，则`Contra[B]`是`Contra[A]`的子类型。

### 5.3.4 不可变性与变型
不可变性(immutability)是变型背后的设计决策的重要组成部分。例如，Scala的集合系统地区分了[可变和不可变集合](https://docs.scala-lang.org/overviews/collections-2.13/overview.html)。协变的可变集合会破坏类型安全，因此集合`scala.collection.immutable.List`是协变的，而`scala.collection.mutable.ListBuffer`是不变的。

假设`ListBuffer`是协变的，以下有问题的代码将能够编译通过：

```scala
import scala.collection.mutable.ListBuffer

val bufInt: ListBuffer[Int] = ListBuffer(1, 2, 3)
val bufAny: ListBuffer[Any] = bufInt
bufAny(0) = "Hello"
val firstElem: Int = bufInt(0)
```

由于`bufInt(0)`包含一个`String`而不是`Int`，将其赋给`firstElem`将抛出`ClassCastException`。

### 小结

| 变型 | 语法 | 含义 | 适用场景 |
| --- | --- | --- | --- |
| 不变 | `class C[T]` | 如果`A`是`B`的子类型，`C[A]`与`C[B]`无关 | 可变类 |
| 协变 | `class C[+T]` | 如果`A`是`B`的子类型，则`C[A]`是`C[B]`的子类型 | 不可变类，只读不写 |
| 逆变 | `class C[-T]` | 如果`A`是`B`的子类型，则`C[B]`是`C[A]`的子类型 | 不可变类，只写不读 |

* 协变(`+T`)类型参数只能出现在输出位置（不可变字段、返回类型），不能出现在输入位置（方法参数）。
* 逆变(`-T`)类型参数只能出现在输入位置，不能出现在输出位置。

例如，Scala标准库特质`Function1`简化的定义如下：

```scala
trait Function1[-A, +R] {
  def apply(arg: A): R
}
```

## 5.4 类型边界
在Scala中，类型参数和抽象类型成员可以受**类型边界**(type bound)的约束。

注：Java也有类似的类型边界概念，详见[《Java核心技术》笔记 卷I 第8章]({% post_url 2024-10-29-java-note-v1ch08-generic-programming %}) 8.4节。

### 5.4.1 类型上界
<https://docs.scala-lang.org/tour/upper-type-bounds.html>

**类型上界**(upper type bound)用`<:`表示。`B <: A`声明`B`必须是`A`的子类型。

例如：

```scala
abstract class Animal
abstract class Pet extends Animal
class Cat extends Pet
class Dog extends Pet
class Lion extends Animal

class PetContainer[P <: Pet](val p: P)

val dogContainer = new PetContainer[Dog](new Dog)
val catContainer = new PetContainer[Cat](new Cat)
val lionContainer = new PetContainer[Lion](new Lion) // this would not compile
```

类`PetContainer`接受一个类型参数`P`，该参数必须是`Pet`的子类型。`Dog`和`Cat`是`Pet`的子类型，因此可以创建`PetContainer[Dog]`和`PetContainer[Cat]`。但是，`Lion`不是`Pet`的子类型，如果试图创建`PetContainer[Lion]`会得到以下错误：

```
type arguments [Lion] do not conform to class PetContainer's type parameter bounds [P <: Pet]
```

### 5.4.2 类型下界
<https://docs.scala-lang.org/tour/lower-type-bounds.html>

**类型下界**(lower type bound)用`>:`表示。`B >: A`声明`B`必须是`A`的超类型。在大多数情况下，`A`是类的类型参数，`B`是方法的类型参数。

下面是一个示例：

```scala
trait List[+A] {
  def prepend(elem: A): NonEmptyList[A] = NonEmptyList(elem, this)
}

case class NonEmptyList[+A](head: A, tail: List[A]) extends List[A]

case object Nil extends List[Nothing]
```

这个程序实现了一个单向链表。`Nil`表示空链表。类`NonEmptyList`表示一个节点，包含一个`A`类型的元素(`head`)和列表其余部分的引用(`tail`)。特质`List`及其子类型是协变的，因为声明了`+A`。

然而，这个程序无法编译，因为`prepend()`方法的参数`elem`是协变类型`A`。**方法的参数类型是逆变的，返回类型是协变的。**

为了解决这个问题，需要翻转`elem`类型的变型。可以通过引入一个以`A`为类型下界的新的类型参数`B`来实现：

```scala
trait List[+A] {
  def prepend[B >: A](elem: B): NonEmptyList[B] = NonEmptyList(elem, this)
}
```

现在可以这样使用`List`：

```scala
abstract class Animal
class Cat extends Animal
class Dog extends Animal

val dogs: List[Dog] = Nil.prepend(new Dog)
val catsFromAntarctica: List[Animal] = Nil
val someAnimal: Animal = new Cat

// OK, List is covariant in A
val animals: List[Animal] = dogs

// OK, A = Dog, B = Animal
val someAnimals = dogs.prepend(someAnimal) // NonEmptyList[Animal]

// OK, A = Animal, B = Animal
val moreAnimals = animals.prepend(new Cat) // NonEmptyList[Animal]

// OK, A = Dog, B = Animal (because that is the supertype common to Cat and Dog)
val allAnimals = dogs.prepend(new Cat) // NonEmptyList[Animal]

// mistake: A = Animal, B = Object (adding a list widens the type arg too much, -Xlint will warn)
val error = moreAnimals.prepend(catsFromAntarctica) // NonEmptyList[Object]
```

注意：
* `dogs.prepend(new Cat)`向`List[Dog]`添加了一个`Cat`，看起来是有问题的。实际上，编译器会将`prepend()`方法的类型参数`B`推导为`Animal`（即`Cat`和`Dog`的公共超类），因此返回类型为`NonEmptyList[Animal]`而不是`NonEmptyList[Dog]`。
* 由于`List[A]`是协变的，一个`List[Animal]`变量实际可能引用`List[Animal]`、`List[Cat]`或`List[Dog]`对象。换句话说，类型参数`A`代表了以`A`为上界的类层次结构（即“`A`或其子类型”）。在类型边界中，`A`只能作为下界（因为`Animal`、`Cat`和`Dog`没有公共子类型，但有公共超类型）。同理，逆变类型参数只能作为类型上界。

### 小结

| 类型边界 | 语法 | 含义 |
| --- | --- | --- |
| 类型上界 | `B <: A` | `B`必须是`A`的子类型 |
| 类型下界 | `B >: A` | `B`必须是`A`的超类型 |

变型与类型边界的关系：

| 变型 | 上界`B <: A` | 下界`B >: A` |
| --- | --- | --- |
| 不变`A` | √ | √ |
| 协变`+A` | × | √ |
| 逆变`-A` | √ | × |

### 5.4.3 上下文边界
<https://docs.scala-lang.org/scala3/book/ca-context-bounds.html>

在许多情况下，不必显式定义隐式参数的名字，只需提供参数类型即可。

例如，假设`maxElement()`方法返回一个集合中的最大值：

```scala
def maxElement[A](as: List[A])(implicit ord: Ord[A]): A = {
  as.reduceLeft(max(_, _)(ord))
}

val nums = List(1, 2, 3, 4, 5)
implicit val ord: Ord[Int] = (x, y) => x > y
println(maxElement(nums))
```

该方法只接受一个`Ord[A]`类型的隐式参数，只是为了将其传递给`max()`方法。`Ord`和`max()`的定义如下：

```scala
/** Defines how to compare values of type `A` */
trait Ord[A] {
  def greaterThan(a1: A, a2: A): Boolean
}

/** Returns the maximum of two values */
def max[A](a1: A, a2: A)(implicit ord: Ord[A]): A = {
  if (ord.greaterThan(a1, a2)) a1 else a2
}
```

注：Scala集合框架使用特质`scala.math.Ordering`比较元素，它扩展了Java的`Comparator`接口。例如，`List[T]`的`max`和`sorted`方法接受一个`Ordering[T]`类型的隐式参数。

在`maxElement()`中调用`max()`方法时，可以省略隐式参数`ord`：

```scala
def maxElement[A](as: List[A])(implicit ord: Ord[A]): A = {
  as.reduceLeft(max(_, _))
}
```

方法声明还可以进一步简化。**上下文边界**(context bound)是一种语法糖，用于表达“依赖于类型参数的隐式参数”模式。

使用上下文边界，`maxElement()`方法可以这样写：

```scala
def maxElement[A: Ord](as: List[A]): A = {
  as.reduceLeft(max(_, _))
}
```

类型参数`A`的绑定`: Ord`表示有`Ord[A]`类型的隐式参数。编译器会将这种语法转换为前面的隐式参数写法（这种隐式参数称为**证据参数**(evidence parameter)）。

注：真正使用了隐式参数的方法（例如这里的`max()`）不能使用上下文边界。

## 5.5 广义类型约束
在Scala中，`=:=`和`<:<`是定义在`scala.Predef`中的**广义类型约束**(generalized type constraint)，用于在编译时对类型参数进行约束。

```scala
sealed abstract class =:=[From, To] extends (From => To)
sealed abstract class <:<[-From, +To] extends (From => To)
```

### 含义

| 约束 | 含义 |
| --- | --- |
| `A =:= B` | 要求`A`和`B`是同一类型 |
| `A <:< B` | 要求`A`是`B`的子类型 |

注：在Scala中，泛型类型`op[A, B]`可以写成`A op B`（类似于表达式的中缀语法）。因此`A =:= B`是一种类型`=:=[A, B]`，而不是一个表达式。

### 用法
广义类型约束通常用作泛型方法的隐式证据参数。编译器只在满足约束时才会提供隐式实例。

`=:=`示例：

```scala
def update[A, B](set: Set[A])(f: A => B)(implicit ev: A =:= B): Set[B] = {
  set.map(f)
}

val s: Set[String] = ...
update[String, String](s)(_.toUpperCase) // OK
update[String, Int](s)(_.length) // error: Cannot prove that String =:= Int
```

`<:<`示例：

```scala
def upcast[A, B](set: Set[A])(implicit ev: A <:< B): Set[B] = {
  set.map(ev(_))
}

// assume that Dog is a subclass of Animal
val s: Set[Dog] = ...
val s2: Set[Int] = ...
upcast[Dog, Animal](s) // OK
upcast[Int, Animal](s2) // error: Cannot prove that Int <:< Animal
```

其中，`ev`充当函数`A => B`（因为`<:<[From, To]`继承了`From => To`）。

对于这两个简单的例子，直接省略第二个类型参数或者使用类型边界会更简单：

```scala
def update[A](set: Set[A])(f: A => A): Set[A] = {
  set.map(f)
}

def upcast[A, B >: A](set: Set[A]): Set[B] = {
  set.map(identity)
}
```

但是有些约束无法用类型边界来表达。例如，在`scala.Option`类的`flatten()`方法中，类型边界只能约束`B`本身，而无法约束`Option[B]`。

```scala
abstract class Option[+A] {
  def get: A

  def flatten[B](implicit ev: A <:< Option[B]): Option[B] =
    if (isEmpty) None else ev(this.get)
}
```

标准库中的另一个例子是集合类的`toMap()`方法，类型边界无法表达“类型参数`K`和`V`构成的键值对是`A`的超类型”这一约束。

```scala
trait TraversableOnce[+A] {
  def toMap[K, V](implicit ev: A <:< (K, V)): Map[K, V]
}
```

参考
* [Scala School - Advanced types](https://twitter.github.io/scala_school/advanced-types.html)
* [<:< operator in scala - Stack Overflow](https://stackoverflow.com/questions/2603003/operator-in-scala)
* [Kinds of types in Scala, part 2: take type, return type or type parameters](https://kubuszok.com/2018/kinds-of-types-in-scala-part-2/)
* [Scala Generics: Generalized Type Constraints (Part 3)](https://dzone.com/articles/scala-generics-generalized-type-constraints-part-3)
