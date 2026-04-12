---
title: Scala基础教程 第4节 特质和抽象类
date: 2026-04-09 22:47:23 +0800
categories: [Scala]
tags: [scala, interface, abstract class, inheritance, mix-in, self-type]
---
<https://docs.scala-lang.org/overviews/scala-book/traits-intro.html>

Scala **特质**(trait)是该语言的一大特点。可以像Java接口一样使用，也可以像抽象类一样使用。Scala类还可以扩展和“混入”(mix in)多个特质。

Scala也有抽象类的概念，我们将展示何时应该使用抽象类而不是特质。

## 4.1 特质用作接口
<https://docs.scala-lang.org/overviews/scala-book/traits-interfaces.html>

使用Scala特质的一种方式是类似于Java接口，即为某些功能定义需要的接口，但不实现任何行为。

### 4.1.1 定义特质
在Scala中使用关键字`trait`定义特质。例如：

```scala
trait Speaker {
  def speak(): String
}
```

这段代码声明了一个名为`Speaker`的特质，包含一个没有参数、返回`String`的方法`speak()`。该方法没有方法体，因此是抽象方法。任何扩展该特质的类都必须实现`speak()`方法。这段代码等价于以下Java接口：

```java
public interface Speaker {
    String speak();
}
```

注：在Scala 2中，特质不能有构造器参数，但可以有成员变量。

### 4.1.2 扩展特质
使用关键字`extends`创建扩展单个特质的类。可以像这样编写两个类`Cat`和`Dog`，扩展特质`Speaker`并实现其中的方法：

```scala
class Cat extends Speaker {
  override def speak(): String = "Meow~"
}

class Dog extends Speaker {
  override def speak(): String = "Woof!"
}
```

关键字`override`等价于Java的`@Override`注解。不是强制的，但能使编译器检查方法签名与特质/超类方法的一致性，因此建议加上。

可以在REPL中测试代码：

```scala
def speak(s: Speaker): Unit = println(s.speak())

val c = new Cat
val d = new Dog
speak(c) // prints "Meow~"
speak(d) // prints "Woof!"
```

### 4.1.3 扩展多个特质
Scala特质使你能够创建非常模块化的代码。例如，可以将动物的属性分解为小的模块化单元：

```scala
trait Speaker {
  def speak(): String
}

trait TailWagger {
  def startTail(): Unit
  def stopTail(): Unit
}

trait Runner {
  def startRunning(): Unit
  def stopRunning(): Unit
}
```

现在可以创建一个`Dog`类扩展所有特质并实现必要的方法：

```scala
class Dog extends Speaker with TailWagger with Runner {
  // Speaker
  override def speak(): String = "Woof!"

  // TailWagger
  override def startTail(): Unit = println("tail is wagging")
  override def stopTail(): Unit = println("tail is stopped")

  // Runner
  override def startRunning(): Unit = println("I'm running")
  override def stopRunning(): Unit = println("Stopped running")
}
```

注意，对于第一个特质使用`extends`，对于后续特质使用`with`。

## 4.2 特质用作抽象类
<https://docs.scala-lang.org/overviews/scala-book/traits-abstract-mixins.html>

还可以向特质中添加具体（非抽象）方法，并将其用作抽象类（更准确地说是**混入**(mix-in)）。

### 4.2.1 示例
下面的特质`Pet`有一个具体方法`speak()`和一个抽象方法`comeToMaster()`：

```scala
trait Pet {
  def speak(): Unit = println("Yo")  // concrete method
  def comeToMaster(): Unit           // abstract method
}
```

当一个类扩展一个特质时，必须实现所有抽象方法。但不必重新定义具体方法，除非你想覆盖它。下面的类`Dog`扩展了`Pet`：

```scala
class Dog(name: String) extends Pet {
  override def comeToMaster(): Unit = println("Woo-hoo, I'm coming!")
}
```

```scala
val d = new Dog("Zeus")
d.speak() // prints "Yo"
d.comeToMaster() // prints "Woo-hoo, I'm coming!"
```

### 4.2.2 覆盖方法
类也可以覆盖特质中定义的方法。例如，类`Cat`覆盖了`speak()`方法：

```scala
class Cat extends Pet {
  // override 'speak'
  override def speak(): Unit = println("Meow")
  override def comeToMaster(): Unit = println("That's not gonna happen.")
}
```

```scala
val c = new Cat
c.speak() // prints "Meow"
c.comeToMaster() // prints "That's not gonna happen."
```

### 4.2.3 混入多个特质
Scala特质的一个优点是可以将多个具有行为（即具体方法）的特质混入到类中。例如，下面有一组特质，其中一个定义了抽象方法，其他定义了具体方法实现：

```scala
trait Speaker {
  def speak(): String   // abstract
}

trait TailWagger {
  def startTail(): Unit = println("tail is wagging")
  def stopTail(): Unit = println("tail is stopped")
}

trait Runner {
  def startRunning(): Unit = println("I'm running")
  def stopRunning(): Unit = println("Stopped running")
}
```

可以像这样创建`Dog`和`Cat`类并扩展所有这些特质：

```scala
class Dog(name: String) extends Speaker with TailWagger with Runner {
  override def speak(): String = "Woof!"
}

class Cat extends Speaker with TailWagger with Runner {
  override def speak(): String = "Meow~"
  override def startRunning(): Unit = println("Yeah ... I don't run")
  override def stopRunning(): Unit = println("No need to stop")
}
```

注：当一个Scala类的超类和特质具有相同的方法时，解决冲突的方式与Java一样，详见[《Java核心技术》笔记 卷I 第6章]({% post_url 2024-09-19-java-note-v1ch06-interfaces-lambda-expressions-and-inner-classes %}) 6.1.6节。

### 4.2.4 创建实例时混入特质
对于具有具体方法的特质，可以在创建实例时将其混入到类中。例如，对于上一节中的`TailWagger`、`Runner`特质和以下`Dog`类：

```scala
class Dog(name: String)
```

可以像这样创建`Dog`实例：

```scala
val d = new Dog("Fido") with TailWagger with Runner
d.startTail()  // prints "tail is wagging"
d.startRunning()  // prints "I'm running"
```

注：`d`的实际类型是一个匿名类，扩展了`Dog`类和`TailWagger`、`Runner`特质。

不能用这种方式混入包含抽象方法的特质，除非提供实现。例如：

```scala
val d = new Dog("Fido") with Speaker {
  override def speak(): String = "Woof!"
}
```

## 4.3 抽象类
<https://docs.scala-lang.org/overviews/scala-book/abstract-classes.html>

Scala还有**抽象类**(abstract class)的概念，类似于Java的抽象类。但由于特质非常强大，只有在以下情况下才需要使用抽象类：
* 你希望创建一个有构造器参数的超类。
* 你的Scala代码会被Java代码调用。

### 4.3.1 定义抽象类
抽象类语法类似于普通类，只是多了关键字`abstract`。例如，下面是一个名为`Pet`的抽象类，类似于4.2.1节定义的`Pet`特质：

```scala
abstract class Pet(name: String) {
  def speak(): Unit = println("Yo")  // concrete method
  def comeToMaster(): Unit           // abstract method
}
```

可以像这样定义扩展该抽象类的`Dog`类：

```scala
class Dog(name: String) extends Pet(name) {
  override def speak(): Unit = println("Woof!")
  override def comeToMaster(): Unit = println("Here I come!")
}
```

需要注意的一点是构造器参数`name`是如何从`Dog`构造器传递到`Pet`构造器的。

一个类只能扩展一个抽象类，但可以扩展多个特质。

### 4.3.2 在Java代码中调用
Java类可以扩展Scala抽象类。例如，对于上一节中的抽象类`Pet`：

```java
public class Dog extends Pet {
    public Dog(String name) {
        super(name);
    }

    @Override
    public void speak() {
        System.out.println("Woof!");
    }

    @Override
    public void comeToMaster() {
        System.out.println("Here I come!");
    }
}
```

Java类也可以通过`implements`关键字实现Scala特质（就像Java接口一样）。例如，对于4.2.1节中的特质`Pet`：

```java
public class Dog implements Pet {
    private String name;

    public Dog(String name) {
        this.name = name;
    }

    @Override
    public void speak() {
        System.out.println("Woof!");
    }

    @Override
    public void comeToMaster() {
        System.out.println("Here I come!");
    }
}
```

## 4.4 抽象类型成员
<https://docs.scala-lang.org/tour/abstract-type-members.html>

抽象类型（如特质和抽象类）可以有**抽象类型成员**(abstract type member)，由实现类定义实际的类型。例如：

```scala
trait Buffer {
  type T
  val element: T
}
```

特质`Buffer`有一个抽象类型成员`T`，用于描述`element`的类型。可以在抽象类中扩展这个特质，并为`T`添加类型上界，使其更具体。

```scala
abstract class SeqBuffer extends Buffer {
  type U
  type T <: Seq[U]
  def length: Int = element.length
}

abstract class IntSeqBuffer extends SeqBuffer {
    type U = Int
}
```

类`SeqBuffer`声明`T`的类型上界(`<:`)是另一个抽象类型`U`的序列，即`T`必须是`Seq[U]`的子类型。这意味着只能在`element`中存储序列，因此可以调用`element.length`。

类`IntSeqBuffer`定义了成员类型`U = Int`，但成员类型`T`和成员变量`element`仍然是抽象的。

具有抽象类型成员的特质或类通常与匿名类实例化结合使用。例如，下面的程序创建了引用整数列表的`IntSeqBuffer`：

```scala
def newIntSeqBuf(elems: Int*): IntSeqBuffer = new IntSeqBuffer {
  type T = List[U]
  val element = List(elems: _*)
}

val buf = newIntSeqBuf(7, 8)
println("length = " + buf.length)   // length = 2
println("content = " + buf.element) // content = List(7, 8)
```

其中，工厂方法`newIntSeqBuf()`使用`IntSeqBuffer`的匿名子类，将抽象类型`T`设置为具体类型`List[Int]`。

也可以将抽象类型成员转换为泛型类的类型参数，反之亦然。以下是上面的代码只使用类型参数的版本：

```scala
abstract class Buffer[+T] {
  val element: T
}

abstract class SeqBuffer[U, +T <: Seq[U]] extends Buffer[T] {
  def length: Int = element.length
}
```

```scala
def newIntSeqBuf(elems: Int*): SeqBuffer[Int, Seq[Int]] = new SeqBuffer[Int, List[Int]] {
  val element = List(elems: _*)
}

val buf = newIntSeqBuf(7, 8)
println("length = " + buf.length)
println("content = " + buf.element)
```

注意，在这里使用了协变类型(`+T <: Seq[U]`)来隐藏方法`newIntSeqBuf()`返回的对象的具体类型。

## 4.5 self类型
<https://docs.scala-lang.org/tour/self-types.html>

**Self类型**(self-type)是一种声明特质`A`必须混入到另一个特质`B`中的方式，即使`B`没有直接扩展`A`（换句话说，扩展特质`B`的类必须同时扩展特质`A`）。这使得`B`可以直接访问`A`的成员而无需导入。

Self类型是一种缩窄`this`（或其别名标识符）的类型的方式。要在特质中使用self类型，使用语法`someIdentifier: SomeOtherTrait =>`，其中`someIdentifier`是`this`的别名标识符，`SomeOtherTrait`是要混入的特质。例如：

```scala
trait User {
  def username: String
}

trait Tweeter {
  this: User =>  // self-type

  def tweet(tweetText: String): Unit = println(s"$username: $tweetText")
}

class VerifiedTweeter(val username_ : String) extends Tweeter with User {  // We mixin User because Tweeter required it
  override def username = s"real $username_"
}

val realBeyoncé = new VerifiedTweeter("Beyoncé")
realBeyoncé.tweet("Just spilled my glass of lemonade")  // prints "real Beyoncé: Just spilled my glass of lemonade"
```

因为特质`Tweeter`声明了`this: User =>`，因此`tweet()`方法可以直接访问`User`的方法`username`。这也意味着由于`VerifiedTweeter`扩展了`Tweeter`，它必须同时扩展`User`。
