---
title: Scala基础教程
date: 2022-04-01 11:35:00 +0800
categories: [Scala]
tags: [scala]
---
## 简介
Scala是一种结合了面向对象和函数式编程的、静态类型的高级编程语言。

Scala代码被编译成.class文件，运行在Java虚拟机(JVM)上，可以调用Java类库。

官方网站：<https://www.scala-lang.org/>

官方文档：<https://docs.scala-lang.org/>
* [Scala语言规范](https://www.scala-lang.org/files/archive/spec/2.13/)
* [Scala标准库](https://www.scala-lang.org/api/current/index.html)
* [代码风格指南](https://docs.scala-lang.org/style/index.html)

官方教程：
* [Tour of Scala](https://docs.scala-lang.org/tour/tour-of-scala.html)
* [Scala Book](https://docs.scala-lang.org/overviews/scala-book/introduction.html)

在线运行环境：
* [Scastie](https://scastie.scala-lang.org/)
* [Scala在线工具](https://c.runoob.com/compile/15/)

sbt构建工具：<https://www.scala-sbt.org/index.html>

## 安装
### 第1步：安装Java
由于Scala运行在JVM上，因此首先要安装JDK并配置JAVA_HOME环境变量。可从[Oracle Java](https://www.oracle.com/java/technologies/downloads/)或[AdoptOpenJDK](https://adoptopenjdk.net/)下载。

### 第2步：安装Scala
下载地址：<https://www.scala-lang.org/download/all.html>

要使用IDE编辑Scala，可安装IntelliJ IDEA的Scala插件或VSCode的Metals插件

### 第3步：配置环境变量
将SCALA_HOME环境变量设置为Scala安装/解压目录，并将$SCALA_HOME/bin目录添加到PATH环境变量。例如，在~/.bashrc文件末尾增加：

```bash
export SCALA_HOME=/usr/local/scala-2.13.8
export PATH=$PATH:$SCALA_HOME/bin
```

之后执行`source ~/.bashrc`或重启终端，即可使用scala和scalac命令：

```bash
$ scala -version
Scala code runner version 2.13.8 -- Copyright 2002-2021, LAMP/EPFL and Lightbend, Inc.
$ scalac -version
Scala compiler version 2.13.8 -- Copyright 2002-2021, LAMP/EPFL and Lightbend, Inc.
```

## Hello, world
Scala的 "Hello, world" 程序如下：

```scala
object Hello {
  def main(args: Array[String]) = {
    println("Hello, world!")
  }
}
```

另一种写法：

```scala
object Hello extends App {
  println("Hello, world!")
}
```

将代码保存到Hello.scala文件，使用scalac命令编译：

```bash
$ scalac Hello.scala
```

该命令将生成两个文件：Hello.class和Hello$.class。这些和使用javac创建的字节码文件相同，可以在JVM上运行。使用scala命令运行Hello程序：

```bash
$ scala Hello
Hello, world!
```

## Scala REPL
Scala REPL ("Read-Evaluate-Print-Loop")是一个命令行解释器，可以将其用作playground来测试Scala代码（相当于Python的IDLE）。

在命令行输入scala即可启动REPL会话：

```bash
$ scala
Welcome to Scala 2.13.8 (OpenJDK 64-Bit Server VM, Java 1.8.0_312).
Type in expressions for evaluation. Or try :help.

scala> 
```

输入`:help`查看帮助，输入`:quit`退出。

在REPL中可以输入Scala表达式：

```scala
scala> val x = 1
val x: Int = 1

scala> val y = x + 1
val y: Int = 2
```

## 两种类型的变量
Scala有两种类型的变量：
* `val`是不可变变量，类似于Java中的`final`，应该是首选
* `var`创建一个可变变量，只有在有特定原因时才应该使用它

例如：

```scala
val x = 1   // 不可变
var y = 0   // 可变
```

## 声明变量类型
在Scala中创建变量时通常不声明变量类型，Scala会自动推断类型：

```scala
val x = 1
val s = "a string"
val p = new Person("Regina")
```

这一特性称为类型推断，有助于保持代码简洁。也可以显式声明变量类型，但通常没有必要：

```scala
val x: Int = 1
val s: String = "a string"
val p: Person = new Person("Regina")
```

## 控制结构
### if/else
Scala的`if/else`控制结构与其他语言类似：

```scala
if (test1) {
  doA()
} else if (test2) {
  doB()
} else if (test3) {
  doC()
} else {
  doD()
}
```

但是，与Java和其他语言不同的是，**if/else结构返回一个值**，因此可以将其用作三元运算符：

```scala
val x = if (a < b) a else b
```

### match表达式
Scala的`match`表达式类似于Java的`switch`语句：

```scala
val result = i match {
  case 1 => "one"
  case 2 => "two"
  case _ => "not 1 or 2"
}
```

`match`表达式不仅限于整数，它可以与任何数据类型一起使用：

```scala
val booleanAsString = b match {
  case true => "true"
  case false => "false"
}
```

下面是一个将`match`用作方法体的示例，与多种不同的类型进行匹配：

```scala
def getClassAsString(x: Any): String = x match {
  case s: String => s + " is a String"
  case i: Int => "Int"
  case f: Float => "Float"
  case l: List[_] => "List"
  case p: Person => "Person"
  case _ => "Unknown"
}
```

（这里是判断x的类型而不是具体值，相当于Java 14的`instanceof`模式匹配和Java 17的`switch`模式匹配）

### try/catch
Scala的`try/catch`控制结构用于捕获异常，类似于Java，但其语法与`match`表达式一致：

```scala
try {
  writeToFile(text)
} catch {
  case fnfe: FileNotFoundException => println(fnfe)
  case ioe: IOException => println(ioe)
}
```

### for循环和表达式
Scala的`for`循环有三种形式：

```scala
// for-in
for (arg <- args) println(arg)

// "x to y" syntax
for (i <- 0 to 5) println(i)

// "x to y by" syntax
for (i <- 0 to 10 by 2) println(i)
```

注意：`to`后面的上界是包含的！

可以在`for`循环中添加`yield`关键字来创建`for`表达式（相当于Python的列表推导式）。下面是一个将序列1~5中的每个值加倍的`for`表达式：

```scala
scala> val x = for (i <- 1 to 5) yield i * 2
val x: IndexedSeq[Int] = Vector(2, 4, 6, 8, 10)
```

下面是一个迭代字符串列表的`for`表达式：

```scala
val fruits = List("apple", "banana", "lime", "orange")

val fruitLengths = for {
  f <- fruits
  if f.length > 4
} yield f.length
// val fruitLengths: List[Int] = List(5, 6, 6)
```

从字面上即可理解代码的含义。

### while和do/while
Scala的`while`和`do/while`循环语法如下：

```scala
// while loop
while (condition) {
  statement
}

// do-while
do {
  statement
} while (condition)
```

## 类
下面是一个Scala类的例子：

```scala
class Person(var firstName: String, var lastName: String) {
  def printFullName() = println(s"$firstName $lastName")
}
```

这是使用该类的方式：

```scala
val p = new Person("Julia", "Kern")
println(p.firstName)
p.lastName = "Manes"
p.printFullName()
```

注意，不需要创建 "get" 和 "set" 方法来访问类中的字段。

## 方法
像其他OOP语言一样，Scala类也有方法。Scala方法的语法如下：

```scala
def sum(a: Int, b: Int): Int = a + b
def concatenate(s1: String, s2: String): String = s1 + s2
```

不必声明方法的返回类型，因此这两个方法等价于

```scala
def sum(a: Int, b: Int) = a + b
def concatenate(s1: String, s2: String) = s1 + s2
```

这是调用方法的方式：

```scala
val x = sum(1, 2)
val y = concatenate("foo", "bar")
```

## 特质(Traits)
Scala的特质(traits)类似于Java的接口，可以将代码分解为小的模块化单元。例如，给定三个trait：

```scala
trait Speaker {
  def speak(): String // has no body, so it’s abstract
}

trait TailWagger {
  def startTail(): Unit = println("tail is wagging")
  def stopTail(): Unit = println("tail is stopped")
}

trait Runner {
  def startRunning(): Unit = println("I’m running")
  def stopRunning(): Unit = println("Stopped running")
}
```

可以创建一个扩展(extend)所有这些trait的`Dog`类，同时实现`speak`方法：

```scala
class Dog(name: String) extends Speaker with TailWagger with Runner {
  def speak(): String = "Woof!"
}
```

类似地，`Cat`类覆盖了trait方法：

```scala
class Cat extends Speaker with TailWagger with Runner {
  def speak(): String = "Meow"
  override def startRunning(): Unit = println("Yeah ... I don’t run")
  override def stopRunning(): Unit = println("No need to stop")
}
```

## 集合类
虽然可以在Scala中使用Java集合类，但强烈建议学习基本的Scala集合类：`List`、`ListBuffer`、`Vector`、`ArrayBuffer`、`Map`和`Set`。Scala 集合类的一大好处是它们提供了许多强大的方法，可以使用这些方法来简化代码。

### 创建列表
Scala提供了很多创建列表的方法，例如：

```scala
scala> val nums = List.range(0, 10)
val nums: List[Int] = List(0, 1, 2, 3, 4, 5, 6, 7, 8, 9)

scala> val nums = (1 to 10 by 2).toList
val nums: List[Int] = List(1, 3, 5, 7, 9)

scala> val letters = ('a' to 'f').toList
val letters: List[Char] = List(a, b, c, d, e, f)

scala> val letters = ('a' to 'f' by 2).toList
val letters: List[Char] = List(a, c, e)
```

### 序列方法
给定两个列表：

```scala
val nums = (1 to 10).toList
val names = List("joel", "ed", "chris", "maurice")
```

`foreach`方法：

```scala
scala> names.foreach(println)
joel
ed
chris
maurice
```

`filter`方法：

```scala
scala> nums.filter(_ < 4).foreach(println)
1
2
3
```

`map`方法：

```scala
scala> val doubles = nums.map(_ * 2)
val doubles: List[Int] = List(2, 4, 6, 8, 10, 12, 14, 16, 18, 20)

scala> val capNames = names.map(_.capitalize)
val capNames: List[String] = List(Joel, Ed, Chris, Maurice)

scala> val lessThanFive = nums.map(_ < 5)
val lessThanFive: List[Boolean] = List(true, true, true, true, false, false, false, false, false, false)
```

`foldLeft`方法（相当于Python的`functools.reduce()`函数）：

```scala
scala> nums.foldLeft(0)(_ + _)
val res18: Int = 55

scala> nums.foldLeft(1)(_ * _)
val res19: Int = 3628800
```

第一个例子求`nums`中所有数字的和，第二个例子求`nums`中所有数字的积。

详细介绍见[Scala Book - Scala Collections](https://docs.scala-lang.org/overviews/scala-book/collections-101.html)和[Overview - Scala Collections](https://docs.scala-lang.org/overviews/collections-2.13/introduction.html)。

## 元组
元组可以将不同类型的元素放在一个小容器中。一个元组可以包含2~22个值，并且所有值都可以具有不同的类型。例如，下面是一个包含`Int`、`Double`和`String`三种类型的元组：

```scala
(11, 11.0, "Eleven")
```

其类型是`Tuple3`。

元组在很多地方都很方便。例如，可以从方法中返回一个元组：

```scala
def getAaplInfo(): (String, BigDecimal, Long) = {
  // get the stock symbol, price, and volume
  ("AAPL", BigDecimal(123.45), 101202303L)
}
```

之后可以将方法的结果赋值给一个变量：

```scala
val t = getAaplInfo()
```

通过数字前面加下划线来访问元组的值：

```scala
scala> t._1
val res23: String = AAPL

scala> t._2
val res24: BigDecimal = 123.45

scala> t._3
val res25: Long = 101202303
```

下面的例子将元组中的字段赋值给变量（类似于Python的元组赋值）：

```scala
scala> val (symbol, price, volume) = t
val symbol: String = AAPL
val price: BigDecimal = 123.45
val volume: Long = 101202303
```

元组非常适合快速、临时地将一些东西组合在一起。如果发现多次使用相同的元组，则声明一个专用的case类可能很有用。例如：

```scala
case class StockInfo(symbol: String, price: BigDecimal, volume: Long)
```
