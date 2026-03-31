---
title: Scala基础教程 第1节 基础
date: 2026-03-22 11:44:59 +0800
categories: [Scala]
tags: [scala, hello world, command-line argument, variable, declaration, data type, type conversion, string, string interpolation, formatting, raw string, tuple, io]
---
本文将对Scala语言的常用特性进行详细介绍。

参考：
* [Scala Book](https://docs.scala-lang.org/overviews/scala-book/introduction.html)
* [Tour of Scala](https://docs.scala-lang.org/tour/tour-of-scala.html)

<https://docs.scala-lang.org/overviews/scala-book/preliminaries.html>

Scala的安装和环境配置参见[《Scala快速入门教程》]({% post_url 2022-04-01-scala-quickstart %})。本文使用的Scala版本是2.13。

## 1.1 Hello, World
<https://docs.scala-lang.org/overviews/scala-book/hello-world-1.html>

下面是Scala "Hello, world" 示例的源代码：

```scala
object Hello {
  def main(args: Array[String]): Unit = {
    println("Hello, world")
  }
}
```

* 这段代码在一个名为`Hello`的`object`中定义了一个名为`main`的方法。
* `object`类似于`class`，但只有单个实例。这意味着`main()`类似于Java中的静态方法。
* `main()`接受一个名为`args`的字符串数组参数。
* `Array`是包装Java数组的类。

将源代码保存到名为Hello.scala的文件中，之后在命令行运行`scalac`命令进行编译（类似于`javac`）：

```shell
$ scalac Hello.scala
```

该命令会生成两个文件：Hello.class和Hello$.class。这些与使用`javac`命令创建的.class字节码文件相同，可以使用JVM运行。

使用`scala`命令运行`Hello`程序：

```shell
$ scala Hello
Hello, world
```

可以使用`javap`命令查看Hello.class文件：

```shell
$ javap Hello.class
Compiled from "Hello.scala"
public final class Hello {
  public static void main(java.lang.String[]);
}
```

可以看到，Scala创建的.class文件就像是从Java源代码创建的一样。Scala代码可以在JVM上运行，也可以使用现有的Java库，这对Scala程序员来说都是极大的优势。

## 1.2 Hello, World - 版本2
<https://docs.scala-lang.org/overviews/scala-book/hello-world-2.html>

Scala提供了一种更方便地编写应用程序的方式。可以让`object`扩展`App`特质（特质类似于Java的接口，将在第4节详细介绍），而不是定义`main()`方法，如下所示：

```scala
object Hello extends App {
  println("Hello, world")
}
```

运行结果与上一节的程序相同。`App`特质有自己的`main()`方法，该方法执行子类的初始化代码（即类体中的代码，在这里是`println()`调用）。

### 命令行参数
可以通过`main()`方法的`args`参数访问命令行参数。如果扩展`App`，也可以直接访问`args`变量。

```scala
object HelloYou extends App {
  if (args.size == 0)
    println("Hello, you")
  else
    println("Hello, " + args(0))
}
```

编译这段代码，并分别带和不带命令行参数运行：

```shell
$ scalac HelloYou.scala

$ scala HelloYou
Hello, you

$ scala HelloYou Al
Hello, Al
```

`args`是一个数组，可以通过`args.size`（或`args.length`）获得元素个数，通过`args(i)`（不是`[]`）访问元素。（注：与Java一样，在Scala中命令行参数不包括程序名）

## 1.3 Scala REPL
<https://docs.scala-lang.org/overviews/scala-book/scala-repl.html>

Scala REPL ("Read-Evaluate-Print-Loop")是一个命令行解释器，可以用作playground来测试Scala代码。在命令行输入`scala`即可启动REPL会话：

```shell
$ scala
Welcome to Scala 2.13.14 (Java HotSpot(TM) 64-Bit Server VM, Java 17.0.12).
Type in expressions for evaluation. Or try :help.

scala> 
```

可以在REPL中输入Scala表达式：

```scala
scala> val x = 1
val x: Int = 1

scala> val y = x + 1
val y: Int = 2
```

如果不把表达式的结果赋给变量，REPL会自动创建以`res`开头的变量：

```scala
scala> 2 + 3
val res0: Int = 5

scala> 8 / 4
val res1: Int = 2

scala> val z = res0 * res1
val z: Int = 10
```

按Tab键自动补全：

```scala
scala> "hello".ta
tail         tails        take(        takeRight(   takeWhile(   tapEach(
```

输入`:help`查看帮助，输入`:quit`或`:q`退出。

详见[Scala REPL overview](https://docs.scala-lang.org/overviews/repl/overview.html)。

## 1.4 两种类型的变量
<https://docs.scala-lang.org/overviews/scala-book/two-types-variables.html>

Scala有两种类型的变量：
* `val`创建**不可变**变量
* `var`创建**可变**变量

在Scala中像这样声明变量：

```scala
val s = "hello"   // immutable
var i = 42        // mutable

val p = new Person("Joel Fleischman")
```

编译器能够从`=`右侧的表达式推断出变量的类型。如果愿意，也可以显式声明变量类型：

```scala
val s: String = "hello"
var i: Int = 42
```

大多数情况下不需要显式类型，但如果能使代码更容易阅读也可以添加。另见[类型推断](https://docs.scala-lang.org/tour/type-inference.html)。

`val`和`var`的区别在于：`val`变量不可变，初始化后不能重新赋值（类似于Java中的`final`）；而`var`变量可变，可以多次赋值。因此`val`变量也称为 "value" 而不是 "variable" 。

```scala
scala> val a = 'a'
val a: Char = a

scala> a = 'b'
         ^
       error: reassignment to val
```

```scala
scala> var a = 'a'
var a: Char = a

scala> a = 'b'
// mutated a
```

注：`val`变量不同于C++中的`const`，只是不能重新赋值，但可以调用修改类成员的方法。

一般规则是：除非有充分的理由，否则应该始终使用`val`。

注意，在REPL中可以重新定义`val`变量，而在真实代码中不能这样做。

```scala
scala> val age = 18
val age: Int = 18

scala> val age = 19
val age: Int = 19
```

## 1.5 内置类型
<https://docs.scala-lang.org/overviews/scala-book/built-in-types.html>

### 1.5.1 数值类型
Scala具有标准数值类型。与Java不同，在Scala中这些数据类型都是对象（不是基本类型），例如整数类型是`scala.Int`。

像这样声明数值类型的变量：

```scala
val b: Byte = 1
val s: Short = 2
val x: Int = 3
val l: Long = 4
val f: Float = 5.0
val d: Double = 6.0
```

如果不显式声明类型，整数字面值（如`1`）默认为`Int`，浮点数字面值（如`2.0`）默认为`Double`。后缀`L`表示`Long`字面值（如`1L`），后缀`f`或`F`表示`Float`字面值（如`2.0f`），后缀`d`或`D`表示`Double`字面值（可省略）。

```scala
val i = 123   // defaults to Int
val j = 123L  // Long
val x = 1.0   // defaults to Double
val y = 1.0f  // Float
```

数值数据类型及其范围如下表所示：

| 类型 | 描述 | 范围 |
| --- | --- | --- |
| `Boolean` | 布尔值 | `true`或`false` |
| `Byte` | 8位有符号整数 | -128~127 (-2<sup>7</sup>~2<sup>7</sup>-1) |
| `Short` | 16位有符号整数 | -32768~32767 (-2<sup>15</sup>~2<sup>15</sup>-1) |
| `Int` | 32位有符号整数 | -2147483648~2147483647 (-2<sup>31</sup>~2<sup>31</sup>-1) |
| `Long` | 64位有符号整数 | -9223372036854775808~9223372036854775807 (-2<sup>63</sup>~2<sup>63</sup>-1) |
| `Float` | 32位IEEE 754单精度浮点数 | 1.40129846432481707×10<sup>-45</sup> ~ 3.40282346638528860×10<sup>38</sup> |
| `Double` | 64位IEEE 754双精度浮点数 | 4.94065645841246544×10<sup>-324</sup> ~ 1.79769313486231570×10<sup>308</sup> |
| `Char` | 16位无符号Unicode字符 | 0~65535 (0~2<sup>16</sup>-1) |

每种类型的常量`MinValue`和`MaxValue`表示最小值和最大值。

数值类型可以按以下方式转换：

![type-casting-diagram](/assets/images/scala-tutorial-1-preliminaries/type-casting-diagram.svg)

例如：

```scala
val x: Long = 987654321
val y: Float = x  // 9.8765434E8 (note that some precision is lost in this case)

val face: Char = '☺'
val number: Int = face  // 9786
```

转换是单向的。不能将一个`Double`赋给`Int`类型的变量，除非显式调用`toInt`。

注：
* Scala编译器会将这些数值类型翻译为对应的Java基本类型。例如，对于下面的Scala类

```scala
class Foo(val i: Int)
```

编译器生成的字节码等价于以下Java类：

```java
public class Foo {
    private final int i;

    public int i() { return this.i; }

    public Foo(final int i) { this.i = i; }
}
```

* Scala数值类型与对应的Java基本类型可以互相隐式转换，这些转换定义在`scala.Predef`类中。例如，`scala.Int`和`java.lang.Integer`的隐式转换定义如下：

```scala
implicit def int2Integer(x: Int): java.lang.Integer = x.asInstanceOf[java.lang.Integer]
implicit def Integer2int(x: java.lang.Integer): Int = x.asInstanceOf[Int]
```

* 数值类型可以隐式转换为对应的Rich类（例如`Int`可转换为`scala.runtime.RichInt`），因此可以直接对数字调用方法：

```scala
scala> 1.to(5).toList
val res0: List[Int] = List(1, 2, 3, 4, 5)
```

### 1.5.2 BigInt和BigDecimal
对于大数字，Scala提供了`BigInt`和`BigDecimal`，分别表示任意大的整数和任意精度的浮点数（注：底层分别使用`java.math`包中的`BigInteger`和`BigDecimal`实现）。

```scala
scala> val a = 12345678987654321
               ^
       error: integer number too large

scala> val b = BigInt("12345678987654321")
val b: scala.math.BigInt = 12345678987654321

scala> val c = 3.141592653589793238462643383279
val c: Double = 3.141592653589793

scala> val d = BigDecimal("3.141592653589793238462643383279")
val d: scala.math.BigDecimal = 3.141592653589793238462643383279
```

`BigInt`和`BigDecimal`的一个优点是它们支持习惯的数值运算符（而不必像Java的大数值类一样只能使用方法调用）：

```scala
scala> b + 1
val res0: scala.math.BigInt = 12345678987654322

scala> b * b
val res1: scala.math.BigInt = 152415789666209420210333789971041
```

### 1.5.3 String和Char
Scala也有`String`和`Char`类型，字面值分别用双引号和单引号括起来：

```scala
val name = "Bill"
val c = 'a'
```

## 1.6 字符串
<https://docs.scala-lang.org/overviews/scala-book/two-notes-about-strings.html>

Scala的字符串类就是`java.lang.String`，可以直接调用所有Java字符串方法。另外，Scala字符串可以隐式转换为`StringOps`，这个类提供了许多额外的辅助方法。例如：

```scala
scala> "hello, world".substring(7) // Java string method
val res0: String = world

scala> "hello, world".map(_.toUpper) // StringOps method
val res1: String = HELLO, WORLD

scala> "42".toInt // StringOps method
val res2: Int = 42

scala> "hello" * 3 // StringOps method
val res3: String = hellohellohello
```

### 1.6.1 字符串插值
Scala字符串有一种很好的特性，叫做**字符串插值**(string interpolation)。

字符串插值提供了一种在字符串中使用变量的方式。只需在字符串前添加前缀`s`，并在变量名前添加`$`。例如：

```scala
val name = "James"
val age = 30
println(s"$name is $age years old") // "James is 30 years old"
```

可以将变量名用花括号括起来：

```scala
println(s"${name} is ${age} years old")
```

也可以将表达式放在花括号中，例如：

```scala
scala> println(s"1+1 = ${1+1}")
1+1 = 2
```

带前缀`f`的字符串可以使用printf风格的格式化，变量名后面跟着像`%d`这样的格式字符串。例如：

```scala
val name = "James"
val height = 1.9
println(f"$name%s is $height%2.2f meters tall") // "James is 1.90 meters tall"
```

前缀`raw`类似于`s`，但它不执行转义字符。例如：

```scala
scala> val foo = 42
scala> println(raw"a\n$foo")
a\n42
```

另外，还可以自定义插值符，以及在模式匹配中使用字符串插值。详见文档[String Interpolation](https://docs.scala-lang.org/scala3/book/string-interpolation.html)。

### 1.6.2 多行字符串
可以通过使用三个双引号来创建多行字符串：

```scala
val speech = """Four score and
               seven years ago
               our fathers ..."""
```

这种方式的一个缺点是第一行之后的行会被缩进：

```scala
scala> print(speech)
Four score and
               seven years ago
               our fathers ...
```

解决这个问题的一种简单方法是在第一行后之的所有行前添加一个`|`符号，并在字符串后调用`stripMargin()`方法：

```scala
val speech = """Four score and
               |seven years ago
               |our fathers ...""".stripMargin
```

这样所有行都是左对齐的：

```scala
scala> print(speech)
Four score and
seven years ago
our fathers ...
```

## 1.7 Scala类型层次结构
<https://docs.scala-lang.org/tour/unified-types.html>

在Scala中，所有值都有类型，包括数值和函数。下图展示了类型层次结构的一个子集。

![Scala Type Hierarchy](/assets/images/scala-tutorial-1-preliminaries/unified-types-diagram.svg)

* `Any`是所有类型的超类型，也称为顶级类型(top type)。它定义了一些通用方法，例如`equals()`、`hashCode()`和`toString()`。`Any`有两个直接子类：`AnyVal`和`AnyRef`。
* `AnyVal`表示值类型(value type)。有9种预定义的值类型，不可为null：`Boolean`, `Byte`, `Short`, `Int`, `Long`, `Float`, `Double`, `Char`和`Unit`。`Unit`是不包含有意义信息的值类型，只有单一实例，用字面值`()`表示。`Unit`可以用作无返回值的函数的返回类型（类似于Java的`void`和Python的`None`）。
* `AnyRef`表示引用类型(reference type)。所有非值类型都被定义为引用类型。Scala中所有用户定义类型都是`AnyRef`的子类型。在Java运行时环境的上下文中，`AnyRef`对应于`Java.lang.Object`。
* `Nothing`是所有类型的子类型，也称为底部类型(bottom type)。没有类型为`Nothing`的值。通常用于“永远不会返回正常结果”的场景。例如，空列表的类型为`List[Nothing]`；如果一个函数会抛出异常或进入无限循环，其返回类型可以定义为`Nothing`。 
* `Null`是所有引用类型的子类型。它只有一个值，用关键字`null`表示。`Null`主要是为了与其他JVM语言互操作，不应该在Scala代码中使用。

## 1.8 元组
<https://docs.scala-lang.org/overviews/scala-book/tuples.html>

<https://docs.scala-lang.org/tour/tuples.html>

**元组**(tuple)可以将不同类型的元素放在同一个容器中。通过将元素放在圆括号中来创建元组。例如，

```scala
val ingredient = ("Sugar", 25)
```

创建了包含一个`String`元素和一个`Int`元素的2元组，其类型为`(String, Int)`（或者`Tuple2[String, Int]`）。

元组可以包含2~22个元素，并且是不可变的。

当你只需要将一些东西组合在一起但不想定义一个类时，元组很有用。元组在从方法返回多个值时特别方便。例如：

```scala
def getStockInfo: (String, Double, Double) = {
  // other code here ...
  ("NFLX", 100.00, 101.00)  // this is a Tuple3
}
```

可以通过编号访问元组元素，如`_1`、`_2`等。

```scala
println(ingredient._1) // Sugar
println(ingredient._2) // 25
```

也可以使用模式匹配将元组元素赋值给变量：

```scala
val (name, quantity) = ingredient
```

下面是另一个元组模式匹配的例子：

```scala
val planets = List(
  ("Mercury", 57.9), ("Venus", 108.2), ("Earth", 149.6),
  ("Mars", 227.9), ("Jupiter", 778.3))
planets.foreach {
  case ("Earth", distance) =>
    println(s"Our planet is $distance million kilometers from the sun")
  case _ =>
}
```

或者在`for`循环中：

```scala
val numPairs = List((2, 5), (3, -7), (20, 56))
for ((a, b) <- numPairs) {
  println(a * b)
}
```

## 1.9 命令行I/O
<https://docs.scala-lang.org/overviews/scala-book/command-line-io.html>

### 1.9.1 写输出
可以使用`println()`写到标准输出(stdout)，并在末尾添加换行符：

```scala
println("Hello, world")
```

如果不希望添加换行符，则使用`print()`：

```scala
print("Hello without newline")
```

使用`printf()`打印格式化输出：

```scala
printf("%s is %2.2f meters tall", "James", 1.9)
```

像这样写到标准错误(stderr)：

```scala
System.err.println("yikes, an error happened")
```

### 1.9.2 读取输入
读取命令行输入最简单的方式是使用`scala.io.StdIn`类的`readLine()`方法。该方法从标准输入(stdin)读取一整行，并删除末尾的换行符，如果到达输入结尾则返回`null`。

下面是一个简单的例子：

```scala
import scala.io.StdIn.readLine

object HelloInteractive {
  def main(args: Array[String]): Unit = {
    print("Enter your first name: ")
    val firstName = readLine()

    print("Enter your last name: ")
    val lastName = readLine()

    println(s"Your name is $firstName $lastName")
  }
}
```

```shell
$ scala HelloInteractive  
Enter your first name: Alvin
Enter your last name: Alexander
Your name is Alvin Alexander
```
