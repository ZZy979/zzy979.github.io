---
title: Scala基础教程 第2节 控制结构
date: 2026-03-28 09:46:59 +0800
categories: [Scala]
tags: [scala, if statement, for statement, yield statement, match expression, pattern matching, sealed class, regular expression, partial function, try statement, error handling]
---
<https://docs.scala-lang.org/overviews/scala-book/control-structures.html>

## 2.1 if/else
<https://docs.scala-lang.org/overviews/scala-book/if-then-else-construct.html>

Scala的基本`if`语句如下：

```scala
if (a == b) {
  doSomething()
}
```

对于单条语句可以省略花括号：

```scala
if (a == b) doSomething()
```

`if/else`结构如下：

```scala
if (a == b) {
  doSomething()
} else {
  doSomethingElse()
}
```

`if/else-if/else`结构如下（实际上是`else`后面跟着另一个`if/else`）：

```scala
if (test1) {
  doX()
} else if (test2) {
  doY()
} else {
  doZ()
}
```

Scala的`if`语句**总是返回一个结果**。可以像前面的例子一样忽略结果，但一种更常见用法（尤其是在函数式编程中）是将结果赋给一个变量：

```scala
val minValue = if (a < b) a else b
```

这样Scala就不需要特殊的“三元运算符”。

## 2.2 for循环
<https://docs.scala-lang.org/overviews/scala-book/for-loops.html>

Scala的`for`循环可用于迭代集合中的元素。形式如下：

```scala
for (elem <- collection) doSomething()
```

例如：

```scala
val nums = Seq(1, 2, 3)
for (n <- nums) println(n)
```

可以像这样遍历整数范围：

```scala
scala> for (i <- 0 to 5) print(s"$i ")
0 1 2 3 4 5 
scala> for (i <- 0 until 5) print(s"$i ")
0 1 2 3 4 
scala> for (i <- 0 to 10 by 2) print(s"$i ")
0 2 4 6 8 10 
```

注：这实际上是用中缀语法调用了`RichInt`类的`to()`和`until()`方法。`0 until 5`等价于`0.until(5)`，返回一个可迭代对象`Range(0, 5)`。

Scala也有`while`和`do/while`循环：

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

### 2.2.1 多个生成器
`for`循环可以有多个生成器（相当于嵌套循环）。例如：

```scala
for {
  i <- 1 to 2
  j <- 'a' to 'c'
} {
  println(s"i = $i, j = $j")
}
```

输出结果如下：

```
i = 1, j = a
i = 1, j = b
i = 1, j = c
i = 2, j = a
i = 2, j = b
i = 2, j = c
```

### 2.2.2 守卫
`for`循环还可以包含`if`语句，称为**守卫**(guard)。例如：

```scala
for {
  i <- 1 to 5
  if i % 2 == 0
} {
  println(i)
}
```

输出结果为

```
2
4
```

`for`循环可以有任意数量的守卫（相当于所有的条件取and）。下面的例子只打印 "4" ：

```scala
for {
  i <- 1 to 10
  if i > 3
  if i < 6
  if i % 2 == 0
} {
  println(i)
}
```

### 2.2.3 迭代映射
可以使用`for`循环迭代Scala `Map`（类似于Java `HashMap`）的键值对。`Map[K, V]`是键值对元组`(K, V)`的集合。

例如，给定电影名称和评分的映射：

```scala
val ratings = Map(
  "Lady in the Water"  -> 3.0, 
  "Snakes on a Plane"  -> 4.0, 
  "You, Me and Dupree" -> 3.5
)
```

可以像这样使用`for`循环打印电影评分：

```scala
for ((name, rating) <- ratings)
  println(s"Movie: $name, Rating: $rating")
```

### 2.2.4 foreach方法
为了迭代集合元素，还可以使用Scala集合类的`foreach()`方法，其参数是接受一个元素参数的函数。

例如，可以像这样打印整数列表：

```scala
val nums = Seq(1, 2, 3)
nums.foreach(println)
```

也可以像这样使用`foreach()`打印前面的映射：

```scala
ratings.foreach {
  case (name, rating) => println(s"Movie: $name, Rating: $rating")
}
```

注：上面的代码利用了模式匹配的匿名函数语法，等价于以下写法：

```scala
ratings.foreach(t => println(s"Movie: ${t._1}, Rating: ${t._2}"))
```

## 2.3 for表达式
<https://docs.scala-lang.org/overviews/scala-book/for-expressions.html>

<https://docs.scala-lang.org/tour/for-comprehensions.html>

`for`循环用于副作用（如打印输出），而`for`表达式用于从现有集合创建新集合（类似于Python的列表推导式）。`for`表达式的语法类似于`for`循环，但语句体是`yield`语句。

例如，给定一个整数列表

```scala
val nums = Seq(1, 2, 3, 4, 5)
```

可以像这样创建一个新的整数列表，其中每个值都加倍：

```scala
scala> val doubledNums = for (n <- nums) yield n * 2
val doubledNums: Seq[Int] = List(2, 4, 6, 8, 10)
```

这等价于下面的`map()`方法调用：

```scala
val doubledNums = nums.map(_ * 2)
```

`for`表达式也可以有多个生成器和守卫。例如：

```scala
val fruits = List("apple", "banana", "lime", "orange")
val fruitLengths = for (f <- fruits if f.length > 4) yield f.length // List(5, 6, 6)
```

这等价于

```scala
val fruitLengths = fruits.filter(_.length > 4).map(_.length)
```

## 2.4 match表达式和模式匹配
<https://docs.scala-lang.org/overviews/scala-book/match-expressions.html>

<https://docs.scala-lang.org/tour/pattern-matching.html>

**模式匹配**(pattern matching)是一种根据模式检查值的机制。Scala通过`match`表达式实现模式匹配，它是Java `switch`语句的一个更强大的版本。

Scala模式匹配的完整语法参见[Scala语言规范-Pattern Matching](https://scala-lang.org/files/archive/spec/2.13/08-pattern-matching.html)。

### 2.4.1 基本用法
`match`表达式由一个值、`match`关键字和若干个`case`子句（也称为**备选项**(alternative)）组成。例如：

```scala
import scala.util.Random

val i: Int = Random.nextInt(10)

i match {
  case 1 => println("one")
  case 2 => println("two")
  case _ => println("other")
}
```

其中`i`是0~9之间的随机整数，后面的`match`表达式有3个case，最后一个`_`是默认case，可以匹配其他任何值(catch-all)。如果不匹配任何case，也没有默认case，则会抛出`MatchError`异常。

`match`表达式也会返回值（类似于Java 14引入的`switch`表达式），因此可以将其赋给变量：

```scala
val s = i match {
  case 1 => "one"
  case 2 => "two"
  case _ => "other"
}
```

将`match`表达式作为方法体也是一种常见的用法：

```scala
def matchTest(i: Int): String = i match {
  case 1 => "one"
  case 2 => "two"
  case _ => "other"
}
```

`match`表达式允许在单个`case`语句中匹配多个值。下面的例子计算“布尔相等”：0或空字符串的结果为`false`，其他任何值的结果为`true`。

```scala
def isTrue(a: Any) = a match {
  case 0 | "" => false
  case _ => true
}
```

由于参数类型为`Any`（Scala的顶级类型），这个方法适用于任何数据类型：

```scala
scala> isTrue(0)
val res0: Boolean = false

scala> isTrue("")
val res1: Boolean = false

scala> isTrue(1.1F)
val res2: Boolean = true

scala> isTrue(List())
val res3: Boolean = true
```

下面是另一个在一个`case`语句中处理多个值的例子：

```scala
val evenOrOdd = i match {
  case 1 | 3 | 5 | 7 | 9 => "odd"
  case 2 | 4 | 6 | 8 | 10 => "even"
  case _ => "some other number"
}
```

```scala
cmd match {
  case "start" | "go" => println("starting")
  case "stop" | "quit" | "exit" => println("stopping")
  case _ => println("doing nothing")
}
```

### 2.4.2 模式守卫
可以在`case`语句中使用`if`表达式，例如：

```scala
count match {
  case 1 =>
    println("one, a lonely number")
  case x if x == 2 || x == 3 =>
    println("two's company, three's a crowd")
  case x if x > 3 =>
    println("4+, that's a party")
  case _ =>
    println("i'm guessing your number is zero or less")
}
```

`case`后的变量`x`会匹配任何满足条件的值，并将值绑定到该变量，可以在`=>`后的语句中访问该变量。

### 2.4.3 匹配元组
可以像这样匹配元组并提取元素：

```scala
def matchTuple(t: Product): String = t match {
  case (a, b) => s"Pair: ($a, $b)"
  case (a, b, c) => s"Triple: ($a, $b, $c)"
  case _ => "Other tuple"
}
```

```scala
scala> matchTuple(("hello", 42))
val res0: String = Pair: (hello, 42)

scala> matchTuple(("hello", 42, true))
val res1: String = Triple: (hello, 42, true)

scala> matchTuple(("hello", 42, true, 3.14))
val res2: String = Other tuple
```

其中`Product`特质是`Product1`~`Product22`的公共超类，分别表示具有1~22个固定元素的类型。元组`Tuple1`~`Tuple22`分别扩展了这些特质。

### 2.4.4 匹配数组和集合
匹配数组：

```scala
def matchArray(arr: Array[Int]): String = arr match {
  case Array() => "Empty array"
  case Array(x) => s"Single element: $x"
  case Array(x, y) => s"Two elements: $x, $y"
  case Array(x, _*) => s"At lease one element, head: $x"
}
```

匹配列表：

```scala
def matchList(list: List[Int]): String = list match {
  case Nil => "Empty list"
  case head :: tail => s"Head: $head, tail: $tail"
}
```

### 2.4.5 匹配类型
还可以匹配值的类型（类似于Java 16引入的`instanceof`模式匹配）。例如：

```scala
def getClassAsString(x: Any): String = x match {
  case s: String => s"'$s' is a String"
  case i: Int => "Int"
  case d: Double => "Double"
  case l: List[_] => "List"
  case _ => "Unknown"
}
```

```scala
scala> getClassAsString(1)
val res0: String = Int

scala> getClassAsString("hello")
val res1: String = 'hello' is a String

scala> getClassAsString(List(1, 2, 3))
val res2: String = List
```

注意：由于类型擦除，`match`表达式无法匹配泛型类的类型参数。即使将第4个case写为`case l: List[Int]`，传递一个`List[String]`仍然会匹配这个case。

### 2.4.6 匹配case类
Case类对于模式匹配特别有用（将在第3节详细介绍）。

```scala
sealed trait Notification

case class Email(sender: String, title: String, body: String) extends Notification

case class SMS(caller: String, message: String) extends Notification

case class VoiceRecording(contactName: String, link: String) extends Notification
```

`Notification`是一个密封特质（只能被同一个文件中的类扩展），有3个实现case类：`Email`、`SMS`和`VoiceRecording`。现在可以对这些case类进行模式匹配：

```scala
def showNotification(notification: Notification): String = {
  notification match {
    case Email(sender, title, _) =>
      s"You got an email from $sender with title: $title"
    case SMS(number, message) =>
      s"You got an SMS from $number! Message: $message"
    case VoiceRecording(name, link) =>
      s"You received a Voice Recording from $name! Click the link to hear it: $link"
  }
}
```

```scala
val someEmail = Email("jenny@gmail.com", "Drinks tonight?", "I'm free after 5!")
val someSms = SMS("12345", "Are you there?")
val someVoiceRecording = VoiceRecording("Tom", "voicerecording.org/id/123")

println(showNotification(someEmail))
  // You got an email from jenny@gmail.com with title: Drinks tonight?
println(showNotification(someSms))
  // You got an SMS from 12345! Message: Are you there?
println(showNotification(someVoiceRecording))
  // You received a Voice Recording from Tom! Click the link to hear it: voicerecording.org/id/123
```

当基类型是`sealed`时，编译器会检查`match`表达式的case是否穷尽。例如，在上面的`showNotification()`方法中，如果遗漏一个case（比如`VoiceRecording`），编译器会发出警告：

```
match may not be exhaustive.
It would fail on the following input: VoiceRecording(_, _)
```

此时如果输入`VoiceRecording`对象，会抛出`MatchError`异常。

### 2.4.7 字符串匹配
`s`插值符也可以用于模式匹配。例如：

```scala
val input = "Alice is 25 years old"

val result = input match {
  case s"$name is $age years old" => s"$name's age is $age"
  case _ => "No match"
}
// result: "Alice's age is 25"
```

在这个例子中，根据模式提取了字符串的`name`和`age`部分，这有助于解析结构化文本。

还可以将提取器对象用于字符串模式匹配。

```scala
object Age {
  def unapply(s: String): Option[Int] = s.toIntOption
}

val input: String = "Alice is 25 years old"

val (name, age) = input match {
  case s"$name is ${Age(age)} years old" => (name, age)
}
// name: String = Alice
// age: Int = 25
```

### 2.4.8 自定义提取器
<https://docs.scala-lang.org/tour/extractor-objects.html>

**提取器对象**(extractor object)是具有`unapply()`方法的对象。`apply()`方法类似于构造器，接受参数并创建一个对象；`unapply()`方法相反，接受一个对象并尝试还原出构造参数。这通常用于模式匹配和偏函数。

```scala
object Address {
  def apply(host: String, port: Int): String = s"$host:$port"

  def unapply(address: String): Option[(String, Int)] = {
    address.split(":", 2) match {
      case Array(host, portStr) if host.nonEmpty => portStr.toIntOption match {
        case Some(port) if port >= 1 && port <= 65535 => Some((host, port))
        case _ => None
      }
      case _ => None
    }
  }
}
```

`Address(host, port)`等价于`Address.apply(host, port)`，由主机名和端口号构造一个地址字符串。例如：

```scala
println(Address("localhost", 8080))  // "localhost:8080"
println(Address("192.168.1.1", 80))  // "192.168.1.1:80"
println(Address("example.com", 443)) // "example.com:443"
```

`unapply()`方法执行相反的操作，尝试将地址字符串解析为主机名和端口号。返回类型`Option`表示可选的值，只有两个子类：`Some`和`None`。对一个字符串`s`进行模式匹配时，`case Address(host, port) => ...`会调用`Address.unapply(s)`。如果返回`Some((host, port))`则匹配该case，并将元组的两个元素分别赋给变量`host`和`port`；否则不匹配该case。例如：

```scala
def matchAddress(address: String): String = address match {
  case Address(host, port) => s"host is $host, port is $port"
  case _ => "Invalid address"
}
```

```scala
println(matchAddress("localhost:8080"))  // "host is localhost, port is 8080"
println(matchAddress("192.168.1.1:80"))  // "host is 192.168.1.1, port is 80"
println(matchAddress("example.com:443")) // "host is example.com, port is 443"
println(matchAddress("host:99999"))      // "Invalid address"
println(matchAddress("host:abc"))        // "Invalid address"
println(matchAddress(":8080"))           // "Invalid address"
println(matchAddress("invalid"))         // "Invalid address"
```

提取器也可用于初始化变量：

```scala
val addr = "localhost:8080"
val Address(host, port) = addr
// host: String = localhost
// port: Int = 8080
```

这等价于`val (host, port) = Address.unapply(addr).get`。如果匹配失败，会抛出`MatchError`。

Scala会自动为case类生成`unapply()`方法，因此case类可以直接用于模式匹配。

`unapply()`方法的返回类型应该按如下方式选择：
* 如果仅测试是否匹配，则返回`Boolean`。
* 如果结果为单个`T`类型的值，则返回`Option[T]`。
* 如果结果为多个值`T1,...,Tn`，则返回可选的元组`Option[(T1,...,Tn)]`。

有时，要提取的值的数量不是固定的。在这种情况下，可以定义具有`unapplySeq()`方法的提取器，返回`Option[Seq[T]]`。例如匹配列表和正则表达式。

### 2.4.9 正则表达式
<https://docs.scala-lang.org/tour/regular-expression-patterns.html>

**正则表达式**(regular expression)是可用于查找模式的字符串。可以使用`.r`方法将字符串转换为正则表达式。

```scala
val numberPattern = "[0-9]".r

numberPattern.findFirstMatchIn("awesomepassword") match {
  case Some(_) => println("Password OK")
  case None => println("Password must contain a number")
}
```

在上面的示例中，`numberPattern`是一个`scala.util.matching.Regex`，用于匹配单个0~9的数字。

可以使用圆括号搜索正则表达式的**分组**(group)。

```scala
val keyValPattern = "([0-9a-zA-Z- ]+): ([0-9a-zA-Z-#()/. ]+)".r

val input: String =
  """background-color: #A03300;
    |background-image: url(img/header100.png);
    |background-position: top center;
    |background-repeat: repeat-x;
    |background-size: 2160px 108px;
    |margin: 0;
    |height: 108px;
    |width: 100%;""".stripMargin

for (patternMatch <- keyValPattern.findAllMatchIn(input))
  println(s"key: ${patternMatch.group(1)} value: ${patternMatch.group(2)}")
```

输出结果如下：

```
key: background-color value: #A03300
key: background-image value: url(img/header100.png)
key: background-position value: top center
key: background-repeat value: repeat-x
key: background-size value: 2160px 108px
key: margin value: 0
key: height value: 108px
key: width value: 100
```

另外，`Regex`类定义了`unapplySeq()`方法，因此正则表达式可以用于模式匹配，从而方便地提取匹配的分组。模式中的变量个数必须与分组个数完全一致，但可以用`_*`捕获结尾的所有变量。

```scala
def saveContactInformation(contact: String): Unit = {
    val emailPattern = """^(\w+)@(\w+(.\w+)+)$""".r
    val phonePattern = """^(\d{3}-\d{3}-\d{4})$""".r

  contact match {
      case emailPattern(localPart, _*) =>
      println(s"Hi $localPart, we have saved your email address.")
    case phonePattern(phoneNumber) => 
      println(s"Hi, we have saved your phone number $phoneNumber.")
    case _ => 
      println("Invalid contact information, neither an email address nor phone number.")
  }
}

saveContactInformation("123-456-7890")
saveContactInformation("JohnSmith@sample.domain.com")
saveContactInformation("2 Franklin St, Mars, Milky Way")
```

输出结果如下：

```
Hi, we have saved your phone number 123-456-7890.
Hi JohnSmith, we have saved your email address.
Invalid contact information, neither an email address nor phone number.
```

### 2.4.10 偏函数
<https://docs.scala-lang.org/scala3/book/fun-partial-functions.html>

**偏函数**（部分函数，partial function）是一种并非对所有可能的参数值都有定义的函数。在Scala中，偏函数是实现了`PartialFunction[A, B]`特质的一元函数，其中`A`是参数类型，`B`是结果类型。

要定义偏函数，使用与`match`表达式中相同的`case`语句。例如，偏函数`doubledOdds`将参数值乘以2，但仅对奇数有定义：

```scala
val doubledOdds: PartialFunction[Int, Int] = {
  case i if i % 2 == 1 => i * 2
}
```

要检查偏函数是否对某个参数有定义，使用`isDefinedAt()`方法：

```scala
doubledOdds.isDefinedAt(3)  // true
doubledOdds.isDefinedAt(4)  // false
```

用不属于定义域的参数调用偏函数会导致`MatchError`：

```scala
doubledOdds(3)  // 6
doubledOdds(4)  // scala.MatchError: 4
```

可以使用`applyOrElse()`方法为不在定义域中的参数提供默认值，`pf.applyOrElse(x, f)`等价于`if (pf.isDefinedAt(x)) pf(x) else f(x)`。例如：

```scala
doubledOdds.applyOrElse(4, _ + 1)  // 5
```

可以用`orElse()`组合两个偏函数，对于第一个偏函数未定义的参数应用第二个函数：

```scala
val incrementedEvens: PartialFunction[Int, Int] = {
  case i if i % 2 == 0 => i + 1
}
val pf = doubledOdds.orElse(incrementedEvens)
pf(3)  // 6
pf(4)  // 5
```

Scala集合的`collect()`方法将偏函数应用于有定义的元素，返回新集合。`c.collect(pf)`等价于`c.filter(pf.isDefinedAt).map(pf)`。例如：

```scala
val res = List(1, 2, 3, 4).collect(doubledOdds)  // List(2, 6)
```

注：`Seq`和`Map`本身也是一种偏函数
* `Seq[T]`是`PartialFunction[Int, T]`的子类型，将索引映射为元素，定义域为`0 <= i < length`。
* `Map[K, V]`是`PartialFunction[K, V]`的子类型，将键映射为值，定义域为键集。

可选函数（即返回`Option`）、偏函数和提取器对象可以相互转换，如下表所示：

| | 到可选函数 | 到偏函数 | 到提取器对象 |
| --- | --- | --- | --- |
| 从可选函数 | `Predef.identity` | `Function1.UnliftOps.unlift`<br>或`Function.unlift` | `Function1.UnliftOps.unlift` |
| 从偏函数 | `lift` | `Predef.identity` | `Predef.identity` |
| 从提取器对象 | `extractor.unapply(_)` | `{ case extractor(x) => x }` | `Predef.identity` |

## 2.5 try/catch
<https://docs.scala-lang.org/overviews/scala-book/try-catch-finally.html>

与Java一样，Scala的`try/catch`语句可以捕获异常。主要区别在于，Scala使用与`match`匹配相同的语法——`case`语句来匹配可能发生的异常。

在下面的例子中，`openAndReadAFile()`方法打开一个文件并读取其中的文本，将结果赋给变量`text`。该方法可能会抛出`FileNotFoundException`和`IOException`，在`catch`块中捕获这两个异常。

```scala
var text = ""
try {
  text = openAndReadAFile(filename)
} catch {
  case e: FileNotFoundException =>
    println("Couldn't find that file.")
  case e: IOException =>
    println("Had an IOException trying to read that file")
}
```

Scala的`try/catch`语法还允许使用`finally`子句，通常用于关闭资源。以下是一个示例：

```scala
val in = new FileInputStream(...);
try {
  // your scala code here
} catch {
  case e: IOException =>
    println("Got an IOException")
  case _: Throwable =>
    println("Got some other kind of Throwable exception")
} finally {
  in.close();
}
```
