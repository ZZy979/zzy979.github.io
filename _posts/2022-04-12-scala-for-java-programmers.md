---
title: 面向Java程序员的Scala教程
date: 2022-04-12 00:35:37 +0800
categories: [Scala]
tags: [scala]
---
官方教程：[Scala for Java Programmers](https://docs.scala-lang.org/tutorials/scala-for-java-programmers.html)

## 第一个示例
第一个示例是标准的Hello world程序：

```scala
object HelloWorld {
  def main(args: Array[String]): Unit = {
    println("Hello, world!")
  }
}
```

Java程序员应该熟悉该程序的结构：它由一个`main`方法组成，该方法以命令行参数（字符串数组）作为参数，方法的主体包括对预定义方法`println`的一次调用。`main`方法不返回值，因此其返回类型声明为`Unit`（相当于Java的`void`）。

Java程序员不太熟悉的是包含`main`方法的`object`声明。该声明引入了一个**单例对象**(singleton object)，即具有单个实例的类。因此，上面的声明同时声明了一个名为`HelloWorld`的类和该类的一个实例，也称为`HelloWorld`。该实例是在第一次使用时按需创建的。

注意在这里`main`方法并没有声明为`static`，这是因为Scala中不存在静态成员（方法或字段）。Scala程序员在单例对象中声明这些成员，而不是定义静态成员。

### 编译该示例
使用Scala编译器`scalac`编译该示例。`scalac`将源文件作为参数，生成一个或多个标准Java类文件。

假设将上面的程序保存在HelloWorld.scala文件中，使用以下命令来编译：

```bash
$ scalac HelloWorld.scala
```

这将在当前目录下生成几个.class文件，其中一个名为HelloWorld.class。

### 运行该示例
编译完成后，可以使用`scala`命令运行Scala程序。其用法与`java`命令类似，并接受相同的选项（例如`-classpath`）。使用以下命令来运行该示例：

```bash
$ scala HelloWorld
Hello, world!
```

## 与Java交互
Scala的优势之一是与Java代码交互非常容易。`java.lang`包中的所有类都默认导入，而其他类则需要显式导入。

让我们看一个证明这一点的例子。我们希望获取当前日期并根据特定国家（例如法国）使用的约定来格式化。Java类库定义了强大的实用程序类，例如`Date`和`DateFormat`。由于Scala与Java无缝互操作，因此无需在Scala类库中实现等价类——可以简单地导入相应Java包中的类：

```scala
import java.util.{Date, Locale}
import java.text.DateFormat._

object FrenchDate {
  def main(args: Array[String]): Unit = {
    val now = new Date
    val df = getDateInstance(LONG, Locale.FRANCE)
    println(df format now)
  }
}
```

Scala的`import`语句与Java类似，但更加强大。可以从同一个包中导入多个类，方法是将它们括在花括号中（如第一行所示）。另一个区别是，在导入一个包或类的所有名称时，使用下划线(_)而不是星号(*)，这是因为星号是一个有效的Scala标识符。第二行的`import`语句导入了`DateFormat`类的所有成员，因此静态方法`getDateInstance`和静态成员`LONG`直接可见。

最后一行展示了Scala语法的一个有趣的属性：接收一个参数的方法可以使用中缀语法。即`df format now`等价于`df.format(now)`。

值得注意的是，可以直接在Scala中继承Java类和实现Java接口。

## 一切皆对象
Scala是一种纯面向对象的语言，因为**一切**都是对象，包括数字或函数。在这方面与Java不同，因为Java区分原始类型（例如`boolean`和`int`）和引用类型。

### 数字是对象
由于数字是对象，它们也有方法。实际上，像`1 + 2 * 3 / x`这样的算术表达式完全由方法调用组成，因为它等价于以下表达式：`1.+(2.*(3)./(x))`（如上一节所述）。这也意味着`+`、`*`等是Scala中的有效标识符。

### 函数是对象
函数也是`Scala`中的对象，因此可以将函数作为参数传递、存储在变量中、从其他函数返回。这种将函数作为值进行操作的能力是一种称为**函数式编程**的非常有趣的编程范式的基石之一。

作为一个非常简单的例子，说明为什么使用函数作为值很有用，考虑一个计时器函数，其目的是每秒执行一些操作。如何将要执行的操作传递给它？很合乎逻辑：作为一个函数。许多程序员应该熟悉这种非常简单的函数传递：它通常用于用户界面代码中，用于注册回调函数，这些函数在某些事件发生时被调用。

在下面的程序中，计时器函数`oncePerSecond`接收一个回调函数作为参数，其类型写作`() => Unit`，表示无参数无返回值的函数类型。该程序无限地每秒打印一句 "time flies like an arrow"。

```scala
object Timer {
  def oncePerSecond(callback: () => Unit): Unit = {
    while (true) { callback(); Thread sleep 1000 }
  }
  def timeFlies(): Unit = {
    println("time flies like an arrow...")
  }
  def main(args: Array[String]): Unit = {
    oncePerSecond(timeFlies)
  }
}
```

#### 匿名函数
在Scala中可以使用**匿名函数**，即没有名字的函数。下面的程序使用匿名函数代替`timeFlies`函数：

```scala
object TimerAnonymous {
  def oncePerSecond(callback: () => Unit): Unit = {
    while (true) { callback(); Thread sleep 1000 }
  }
  def main(args: Array[String]): Unit = {
    oncePerSecond(() => println("time flies like an arrow..."))
  }
}
```

匿名函数使用右箭头`=>`分隔参数表和主体。在该示例中，参数表为空，函数体与上面的`timeFlies`相同。

## 类
Scala是一种面向对象的语言，因此具有类的概念。Scala中的类声明语法与Java接近。一个重要的区别是Scala中的类可以有参数（相当于构造函数参数）。下面的复数定义说明了这一点：

```scala
class Complex(real: Double, imaginary: Double) {
  def re() = real
  def im() = imaginary
}
```

`Complex`类有两个参数，分别是复数的实部和虚部。创建`Complex`类的实例时必须传递这些参数，例如：`new Complex(1.5, 2.3)`。该类包含两个方法，称为`re`和`im`，分别用于访问这两个部分。

需要注意的是，这两个方法的返回类型没有明确给出。它将由编译器自动推断，编译器查看这些方法的右侧并推断它们都返回`Double`类型的值。

编译器并不总是能够像这样推断类型，但没有一个简单的规则可以准确地知道何时能够推断。实际上，这通常不是问题，因为编译器在无法推断出未明确给出的类型时会报错。一个简单的规则：Scala初学者应该尝试省略看起来很容易从上下文中推断出来的类型声明，看编译器是否同意。一段时间后，程序员应该对何时省略类型以及何时显式指定有一个很好的感觉。

### 不带参数的方法
`re`和`im`方法的一个小问题是，调用时必须使用空括号，如下例所示：

```scala
object ComplexNumbers {
  def main(args: Array[String]): Unit = {
    val c = new Complex(1.2, 3.4)
    println("imaginary part: " + c.im())
  }
}
```

如果能够不使用空括号、像字段一样访问会更好。这在Scala中是完全可行的，只需将其定义为**不带参数的方法**即可。这种方法与**零参数方法**的区别是名字后面没有括号，无论是在定义中还是在使用中。`Complex`类可以重写如下：

```scala
class Complex(real: Double, imaginary: Double) {
  def re = real
  def im = imaginary
}
```

### 继承和覆盖
Scala中的所有类都继承自超类。当没有指定超类时，隐式使用`scala.AnyRef`。

在Scala中可以覆盖从超类继承的方法。然而，必须使用`override`修饰符明确指定一个方法覆盖另一个方法，以避免意外覆盖。例如，`Complex`类可以重新定义继承自`Object`的`toString`方法：

```scala
class Complex(real: Double, imaginary: Double) {
  def re = real
  def im = imaginary
  override def toString() =
    "" + re + (if (im >= 0) "+" else "") + im + "i"
}
```

可以调用覆盖的`toString`方法，如下所示：

```scala
object ComplexNumbers {
  def main(args: Array[String]): Unit = {
    val c = new Complex(1.2, 3.4)
    println("Overridden toString(): " + c.toString)
  }
}
```

## Case类和模式匹配
程序中经常出现的一种数据结构是树(tree)。例如，编译器在内部将程序表示为树；XML文档是树；有几种容器是基于树的（比如红黑树）。

下面通过一个小型计算器程序来研究如何在Scala中表示和操作树。该程序的目的是处理由整数常量、变量和加法组成的简单算术表达式。例如`1+2`、`(x+x)+(7+y)`。

表示这种表达式最自然的一种方式是树，其中非叶节点是操作（这里是加法），叶节点是值（这里是常数或变量）。

在Java中，可以使用一个抽象超类表示树、每种节点一个具体子类来表示。在函数式编程语言中，可以使用代数数据类型。Scala提供了介于两者之间的**case类**的概念。使用case类定义该示例中树的类型的方式如下：

```scala
abstract class Tree
case class Sum(l: Tree, r: Tree) extends Tree
case class Var(n: String) extends Tree
case class Const(v: Int) extends Tree
```

case类与普通类的区别有以下几个方面：
* 创建case类的实例时`new`关键字不是必需的（例如，可以写`Const(5)`而不是`new Const(5)`）
* 自动定义了构造函数参数的getter函数（即可以使用`c.v`来获取`Const`类的实例`c`的构造函数参数`v`的值）
* 提供了`equals`和`hashCode`方法的默认定义，它们基于实例的结构而不是同一性(identity)（例如，`Const(5) == Const(5)`返回true）
* 提供了`toString`方法的默认定义（例如，表达式`x+1`的树打印为`Sum(Var(x),Const(1))`）
* case类的实例可以通过模式匹配来分解，如下所示

有了表示算术表达式的数据类型后，下面定义其操作。首先定义一个在给定的“环境”中对表达式求值的函数，“环境”的目的是提供变量的值。例如，表达式`x+1`在环境`{x=5}`中求值，结果为6。

“环境”可以使用哈希表来表示，也可以使用函数。例如：

```scala
val f: String => Int = { case "x" => 5 }
```

定义了一个函数`f`，当参数为`"x"`时返回5，否则抛出异常`scala.MatchError`。

为了简化程序，首先给“环境”类型`String => Int`定义一个别名（相当于C++的`typedef`）：

```scala
type Environment = String => Int
```

下面给出求值函数的定义。使用递归定义很简单：两个表达式和的值就是值的和；变量的值直接从环境中获取；常数的值就是它本身。

```scala
def eval(t: Tree, env: Environment): Int = t match {
  case Sum(l, r) => eval(l, env) + eval(r, env)
  case Var(n) => env(n)
  case Const(v) => v
}
```

该求值函数对树`t`进行**模式匹配**，其含义很直观：
1. 首先检查`t`是否为`Sum`，如果是，则将左子树绑定到一个名为`l`的新变量，将右子树绑定到一个名为`r`的变量，之后计算箭头后面表达式的值，其中可以使用变量`l`和`r`；
2. 如果`t`不是`Sum`，则检查`t`是否为`Var`，如果是，则将`Var`节点中包含的名称绑定到变量`n`并计算右边的表达式；
3. 如果`t`既不是`Sum`也不是`Var`，则检查`t`是否为`Const`，如果是，则将`Const`节点中包含的值绑定到变量`v`并计算右边的表达式；
4. 如果所有的检查都失败，则产生异常以表示模式匹配表达式失败（在此处只有当定义了`Tree`的其他子类才可能发生这种情况）

可以看出，模式匹配的基本思想是将一个值与一系列模式进行匹配。一旦匹配成功，就提取并命名该值的各个部分，并执行一些操作（通常会使用这些命名的部分）。

这里并没有将`eval`定义为`Tree`及其子类的方法。实际上Scala允许在case类中定义方法，就像普通类一样。因此，决定使用模式匹配还是方法是个人喜好问题。两种方式在可扩展性上各有优劣：
* 使用方法时，添加一种新的节点很容易，只需定义一个`Tree`的子类；添加一种新的操作（例如下面的求导）则比较困难，因为`Tree`和所有子类都需要添加这个方法
* 使用模式匹配时情况正好相反：添加一种新的节点需要修改所有的操作函数；添加一种新的操作很容易，只需将其定义为独立的函数即可

为了进一步探索模式匹配，定义算术表达式的另一个操作：符号求导。规则如下：
* 和的导数是导数的和
* 如果对`v`求导，则变量`v`的导数为1，否则为0
* 常数的导数为0

这些规则可以直接翻译成Scala代码：

```scala
def derive(t: Tree, v: String): Tree = t match {
  case Sum(l, r) => Sum(derive(l, v), derive(r, v))
  case Var(n) if (v == n) => Const(1)
  case _ => Const(0)
}
```

（注意`eval`返回的是表达式的值(`Int`)，而`derive`返回的是求导后的表达式(`Tree`)）

该函数引入了两个与模式匹配相关的新概念。首先，变量的`case`表达式有一个条件，即`if`关键字后面的表达式，只有当其值为真时模式匹配才成功。在这里用于确保被求导变量`n`与求导变量`v`相同。这里使用的第二个新特性是通配符(_)，可以匹配任何值。

下面写一个简单的`main`函数对表达式`(x+x)+(7+y)`执行几个操作：首先在环境`{x=5, y=7}`中计算它的值，之后计算它关于`x`和`y`的导数。`Environment`类型以及`eval`、`derive`和`main`方法需要放在`Calc`对象中：

```scala
abstract class Tree
case class Sum(l: Tree, r: Tree) extends Tree
case class Var(n: String) extends Tree
case class Const(v: Int) extends Tree

object Calc {
  type Environment = String => Int

  def eval(t: Tree, env: Environment): Int = t match {
    case Sum(l, r) => eval(l, env) + eval(r, env)
    case Var(n) => env(n)
    case Const(v) => v
  }

  def derive(t: Tree, v: String): Tree = t match {
    case Sum(l, r) => Sum(derive(l, v), derive(r, v))
    case Var(n) if (v == n) => Const(1)
    case _ => Const(0)
  }

  def main(args: Array[String]): Unit = {
      val exp: Tree = Sum(Sum(Var("x"), Var("x")), Sum(Const(7), Var("y")))
      val env: Environment = { case "x" => 5 case "y" => 7 }
      println("Expression: " + exp)
      println("Evaluation with x=5, y=7: " + eval(exp, env))
      println("Derivative relative to x:\n " + derive(exp, "x"))
      println("Derivative relative to y:\n " + derive(exp, "y"))
    }
}
```

程序的输出如下：
```
Expression: Sum(Sum(Var(x),Var(x)),Sum(Const(7),Var(y)))
Evaluation with x=5, y=7: 24
Derivative relative to x:
 Sum(Sum(Const(1),Const(1)),Sum(Const(0),Const(0)))
Derivative relative to y:
 Sum(Sum(Const(0),Const(0)),Sum(Const(0),Const(1)))
```
这里并没有对结果进行化简。

## 特质
Scala类除了从超类继承代码外，还可以继承一个或多个**特质**(traits)。

对于Java程序员来说，理解特质最简单的方式是将其视为**接口**。在Scala中，当一个类继承一个特质时，会实现该特质的方法，并继承该特质包含的所有代码（从Java 8开始，Java接口可以通过`default`关键字提供方法的默认实现）。

为了了解特质的用处，下面看一个经典的例子：有序对象。能够比较给定类的对象通常很有用，例如排序。在Java中，可比较的对象实现`Comparable`接口。在Scala中，可以定义一个等价的`Ord`特质。

在比较对象时，通常使用六个不同的谓词：小于、小于等于、等于、不等于、大于、大于等于。但是不需要定义全部六个，给定等于和小于就可以表达其他谓词。

```scala
trait Ord {
  def <(that: Any): Boolean
  def <=(that: Any): Boolean = this < that || this == that
  def >(that: Any): Boolean = !(this <= that)
  def >=(that: Any): Boolean = !(this < that)
}
```

这个定义创建了一个名为`Ord`的特质，其作用与Java的`Comparable`接口相同。该特质提供了三个方法的默认实现，只有`<`是抽象方法。等于和不等于没有出现在这里，因为它们默认存在于所有对象中。

`Any`类型是Scala中所有其他类型的超类。它可以看作是Java的`Object`类的更通用版本，因为它也是`Int`、`Float`等基本类型的超类。

注：`Any`和`Object`的关系参考 [scala 中 Any、AnyRef、Object、AnyVal 关系以及主要特点分析](https://www.cnblogs.com/wudeyun/p/12866340.html)

为了使一个对象可比较，只需继承`Ord`特质并定义`equals`和`<`方法即可。例如，定义一个表示日期的`Date`类：

```scala
class Date(y: Int, m: Int, d: Int) extends Ord {
  def year = y
  def month = m
  def day = d
  override def toString: String = s"$year-$month-$day"

  override def equals(that: Any): Boolean =
    that.isInstanceOf[Date] && {
      val o = that.asInstanceOf[Date]
      o.year == year && o.month == month && o.day == day
    }

  override def <(that: Any): Boolean = {
    if (!that.isInstanceOf[Date])
      sys.error("cannot compare " + that + " and a Date")

    val o = that.asInstanceOf[Date]
    year < o.year || (year == o.year && (month < o.month || (month == o.month && day < o.day)))
  }
}
```

其中，`extends Ord`声明该类继承自`Ord`特质。

之后重新定义了继承自`Object`类的`equals`方法，使用各字段来比较日期，来覆盖Java的默认实现（比较是否是同一个对象）。该方法使用了预定义的方法`isInstanceOf`和`asInstanceOf`。`isInstanceOf`对应Java的`instanceof`运算符，当且仅当指定对象是指定类型的实例时返回true。`asInstanceOf`对应Java的强制类型转换：如果对象是指定类型的实例则将其视为该类型，否则抛出`ClassCastException`。

## 泛型
泛型(genericity)是编写按类型参数化的代码的能力。例如，要编写一个链表类，元素的类型是不确定的。

Java 1.5支持了泛型，在Scala中也可以定义泛型类（和方法）。下面的示例是一个最简单的容器类：引用，可以为空，也可以指向某种类型的一个对象。

```scala
class Reference[T] {
  private var content: T = _
  def set(value: T): Unit = { content = value }
  def get: T = content
}
```

`Reference`类有一个名为`T`的类型参数，表示其元素类型。此类型在类的主体中用作`content`变量的类型、`set`方法的参数类型以及`get`方法的返回类型。

在上面的代码中，变量`content`的初始值是`_`，它表示默认值。默认值对于数值类型为0，对于`Boolean`类型为`false`，对于`Unit`类型为`()`，对于所有对象类型为`null`。

要使用`Reference`类，需要指定类型参数`T`使用哪种类型。例如，以下代码创建了持有一个整数的引用：

```scala
object IntegerReference {
  def main(args: Array[String]): Unit = {
    val cell = new Reference[Int]
    cell.set(13)
    println("Reference contains the half of " + (cell.get * 2))
  }
}
```
