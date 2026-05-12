---
title: Scala基础教程 第3节 类和方法
date: 2026-04-01 20:55:26 +0800
categories: [Scala]
tags: [scala, class, constructor, instance field, method, function, block, default argument, keyword argument, operator overloading, variadic argument, implicit parameter, package, import statement, singleton, companion, case class, enumeration, inner class, annotation]
---
<https://docs.scala-lang.org/overviews/scala-book/classes.html>

<https://docs.scala-lang.org/tour/classes.html>

为了支持面向对象编程(OOP)，Scala提供了**类**(class)。其语法比Java简洁得多，但仍然易于使用和阅读。

## 3.1 定义类
下面是一个Scala类，其**构造器**(constructor)有两个参数`firstName`和`lastName`：

```scala
class Person(var firstName: String, var lastName: String)
```

这个类定义大致等价于以下Java代码：

```java
public class Person {
    private String firstName;
    private String lastName;

    public Person(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }

    public String getFirstName() {
        return this.firstName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

    public String getLastName() {
        return this.lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }
}
```

可以像这样创建`Person`实例：

```scala
val p = new Person("Bill", "Panner")
```

定义构造器参数会自动在类中创建**字段**(field)。在这个例子中，可以像这样访问`firstName`和`lastName`字段：

```scala
println(p.firstName + " " + p.lastName) // Bill Panner
```

由于这两个字段都定义为`var`，因此它们是可变的：

```scala
p.firstName = "William"
p.lastName = "Bernheim"
```

也可以将字段定义为`val`，使其不可变：

```scala
class Person(val firstName: String, val lastName: String)
```

现在如果尝试修改`Person`实例的字段，会看到一个错误（与1.4节介绍的`val`变量一样）：

```scala
scala> p.firstName = "Fred"
                   ^
       error: reassignment to val
```

带有`val`或`var`的构造器参数是公有的，没有`val`或`var`的参数是私有的，仅在类内可见。

```scala
class Point(x: Int, y: Int)

val point = new Point(1, 2)
point.x  // error: value x is not a member of Point
```

提示：如果用Scala编写面向对象编程(OOP)代码，通常创建`var`字段，以便对其进行修改。如果用Scala编写函数式编程(FP)代码，通常会使用case类，而不是像这样的类。

## 3.2 构造器
### 3.2.1 主构造器
在Scala中，类的**主构造器**(primary constructor)是以下各项的组合：
* 构造器参数
* 在类体中调用的方法
* 在类体中执行的语句和表达式

Scala类体中声明的字段处理方式类似于Java，在类首次实例化时赋值。例如：

```scala
class Person(var firstName: String, var lastName: String) {
  println("the constructor begins")

  var age = 0

  def fullName: String = s"$firstName $lastName"

  println(fullName)
  println("the end of the constructor")
}
```

类体中可以包含方法、字段和类型，统称为**成员**(member)。上面的`Person`类有4个成员：字段`firstName`、`lastName`和`age`以及方法`fullName`。

构造`Person`对象时，将依次执行类体中的`println()`调用和`age`字段初始化：

```scala
scala> val p = new Person("Kim", "Carnes")
the constructor begins
Kim Carnes
the end of the constructor
val p: Person = Person@27a90ce5
```

这大致等价于以下Java代码：

```java
public class Person {
    private String firstName;
    private String lastName;
    private int age;

    public Person(final String firstName, final String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
        System.out.println("the constructor begins");
        this.age = 0;
        System.out.println(this.fullName());
        System.out.println("the end of the constructor");
    }

    public String fullName() {
        return firstName + " " + lastName();
    }

    // getters and setters
}
```

### 3.2.2 辅助构造器
<https://docs.scala-lang.org/overviews/scala-book/classes-aux-constructors.html>

可以通过定义名为`this`的方法来定义**辅助构造器**(auxiliary constructor)。需要遵循以下规则：
* 每个辅助构造器必须具有不同的签名（参数列表）。
* 每个构造器都必须调用一个之前定义的构造器。

下面是一个定义了多个构造器的类的示例：

```scala
// the primary constructor
class Person(var firstName: String, var lastName: String) {
  // one-arg auxiliary constructor
  def this(firstName: String) = this(firstName, "")

  // zero-arg auxiliary constructor
  def this() = this("", "")
}
```

可以通过几种不同的方式创建`Person`实例：

```scala
val p1 = new Person("Kim", "Carnes")
val p2 = new Person("Kim") // same as new Person("Kim", "")
val p3 = new Person        // same as new Person("", "")
```

注：也可以为构造器参数指定默认值，这样就不必使用辅助构造器了：

```scala
class Person(var firstName: String = "", var lastName: String = "")
```

## 3.3 方法
<https://docs.scala-lang.org/overviews/scala-book/methods-first-look.html>

### 3.3.1 定义方法
在Scala中，使用关键字`def`定义**方法**(method)。例如：

```scala
def double(a: Int) = a * 2
```

定义了一个名为`double`的方法，接受一个名为`a`的整数参数，`=`后是方法体，返回该整数的两倍（不需要`return`关键字）。

在Scala中，方法是在类内定义的（就像Java一样）。但出于测试目的，也可以在REPL中定义方法。

像这样调用方法：

```scala
val x = double(2)
```

前面例子省略了方法的返回类型，编译器会根据类体自动推断。也可以显式声明方法的返回类型：

```scala
def double(a: Int): Int = a * 2
```

当方法体有多行时，可以将其放在花括号内，称为**块**(block)。**块中最后一个表达式的值作为其结果。** 例如：

```scala
def addThenDouble(a: Int, b: Int): Int = {
  val sum = a + b
  val doubled = sum * 2
  doubled
}
```

注：在Scala中调用方法时可以直接用`{}`代替`()`，例如：

```scala
val x = double {
  val a = 1 + 1
  a * 2
}
```

#### 匿名函数
**匿名函数**(anonymous function)即没有名字的方法（类似于Java的Lambda表达式），使用`=>`分隔参数列表和函数体。例如：

```scala
val plusOne = (x: Int) => x + 1
val less = (x: Int, y: Int) => x < y
val greet = () => println("Hello!")
```

可以像方法一样调用函数，例如`plusOne(42)`, `less(3, 4)`, `greet()`。

在Scala中，函数是对象。可以将其赋给变量、传递给方法或从方法中返回。上面三个函数的类型分别是`Int => Int`、`(Int, Int) => Boolean`和`() => Unit`。

注：Scala标准库定义了函数类型`Function0`~`Function22`。`(A1, A2, ..., AN) => R`是`FunctionN[A1, A2, ..., AN, R]`的简写形式。

把函数传递给方法时，如果编译器可以推断出参数类型，就可以将其省略。例如：

```scala
val nums = Seq(1, 2, 3, 4, 5)
val doubledNums = nums.map(x => x * 2)
```

`nums`的元素类型为`Int`，其`map()`方法的参数是函数类型`Int => B`（`B`是`map()`的类型参数），因此`x`的类型为`Int`。在这种情况下，函数可以进一步简写为`_ * 2`。

注：接受函数参数的方法也有`()`和`{}`两种调用方式：

```scala
val a = nums.map(x => (x + 1) * 2)

val b = nums.map(x => {
  val y = x + 1
  y * 2
})

val c = nums.map { x =>
  val y = x + 1
  y * 2
}
```

### 3.3.2 默认参数
<https://docs.scala-lang.org/tour/default-parameter-values.html>

Scala允许为方法参数提供默认值，在调用方法时可以省略这些参数。

```scala
def log(message: String, level: String = "INFO") = {
  println(s"$level: $message")
}

log("System starting")  // prints INFO: System starting
log("User not found", "WARNING")  // prints WARNING: User not found
```

方法`log()`的参数`level`具有默认值，因此是可选的。在第二个调用中，参数`"WARNING"`覆盖了默认值`"INFO"`。使用默认参数可以达到与Java中的重载方法相同的效果。

也可以为构造器参数提供默认值：

```scala
class Point(val x: Double = 0, val y: Double = 0)
```

注意，如果调用者省略了一个参数，则后续参数必须命名。例如：

```scala
val point1 = new Point(y = 1) // same as new Point(0, 1)
val point2 = new Point(1)     // same as new Point(1, 0)
```

为了避免引起歧义，Scala不允许有两个重载方法都具有默认参数。考虑下面的两个方法：

```scala
def func(x: Int = 34): Unit = println(s"func($x)")
def func(y: String = "abc"): Unit = println(s"func($y)")
```

如果调用`func()`，编译器无法知道应该调用哪个方法。即使显式提供参数（如`func(12)`），仍然会编译失败。

注：下面的两个重载方法是合法的，`func(12)`会调用第一个方法。

```scala
def func(x: Int): Unit = println(s"func($x)")
def func(x: Int, y: String = "abc"): Unit = println(s"func($x, $y)")
```

### 3.3.3 命名参数
<https://docs.scala-lang.org/tour/named-arguments.html>

在调用方法时，可以指定参数名，如下所示（类似于Python的关键字参数）：

```scala
def printName(first: String, last: String): Unit =
  println(s"$first $last")

printName("John", "Public")  // Prints "John Public"
printName(first = "John", last = "Public")  // Prints "John Public"
printName(last = "Public", first = "John")  // Prints "John Public"
printName("Elton", last = "John")  // Prints "Elton John"
```

命名参数可以按任何顺序写。但是，从左到右一旦有参数不按形参顺序，其余的参数必须都命名。例如：

```scala
def printFullName(first: String, middle: String = "Q.", last: String): Unit =
  println(s"$first $middle $last")

printFullName("John", "Quincy", "Public")  // Prints "John Quincy Public"
printFullName("John", middle = "Quincy", "Public")  // Prints "John Quincy Public"

printFullName(first = "John", last = "Public")  // Prints "John Q. Public"
printFullName("John", last = "Public")  // Prints "John Q. Public"
printFullName("John", "Public")  // error: not enough arguments

printFullName(last = "Public", first = "John")  // Prints "John Q. Public"
printFullName(last = "Public", "John")  // error: positional after named argument
```

在最后一个调用中，第一个参数是乱序的，因此第二个参数必须命名。

### 3.3.4 getter/setter语法
默认情况下成员是公有的。使用`private`访问修饰符将其声明为私有，只能在类内访问。

```scala
class Point {
  private var _x = 0
  private var _y = 0
  private val bound = 100

  def x: Int = _x

  def x_=(newValue: Int): Unit = {
    if (newValue < bound)
      _x = newValue
    else
      printWarning()
  }

  def y: Int = _y

  def y_=(newValue: Int): Unit = {
    if (newValue < bound)
      _y = newValue
    else
      printWarning()
  }

  private def printWarning(): Unit =
    println("WARNING: Out of bounds")
}
```

在这个`Point`类中，数据存储在私有字段`_x`和`_y`中，并为其定义了访问方法：
* getter方法`x`和`y`返回`_x`和`_y`的值。
* setter方法`x_=`和`y_=`验证并设置`_x`和`_y`的值。注意setter的特殊语法：方法名在getter后面添加`_=`。

```scala
val point1 = new Point
point1.x = 99
point1.y = 101 // prints the warning
```

### 3.3.5 运算符
<https://docs.scala-lang.org/tour/operators.html>

在Scala中，**运算符是方法**。任何具有单个参数的方法都可以用作**中缀运算符**(infix operator)。例如，`+`运算符可以使用`.`语法调用：`10.+(1)`，但中缀运算符更易读：`10 + 1`。

#### 定义运算符
可以使用任何合法标识符作为运算符，包括名字（如`add`）和符号（如`+`）。

```scala
case class Vec(x: Double, y: Double) {
  def +(that: Vec) = Vec(this.x + that.x, this.y + that.y)
}

val vector1 = Vec(1.0, 2.0)
val vector2 = Vec(3.0, 4.0)

val vector3 = vector1 + vector2 // Vec(4.0, 6.0)
```

类`Vec`有一个方法`+`，用于将两个向量相加。

使用括号可以构建复杂的表达式。以下是`MyBool`类的定义，其中包括方法`and`、`or`和`negate`：

```scala
case class MyBool(x: Boolean) {
  def and(that: MyBool): MyBool = if (x) that else this
  def or(that: MyBool): MyBool = if (x) this else that
  def negate: MyBool = MyBool(!x)
}
```

现在可以使用`and`和`or`作为中缀运算符：

```scala
def not(x: MyBool) = x.negate
def xor(x: MyBool, y: MyBool) = (x or y) and not(x and y)
```

这使得`xor`的定义更容易阅读（使用`.`语法要写为`x.or(y).and(not(x.and(y)))`）。

注：`not`和`xor`方法并没有访问`MyBool`的实例字段，因此适合放在类`MyBool`的伴生对象中（详见3.5节）。

#### 运算符优先级
当表达式使用多个运算符时，将根据运算符**第一个字符**的优先级进行求值。下面按优先级递减的顺序列出，同一行的字符具有相同的优先级：

```
(characters not shown below)
* / %
+ -
:
< >
= !
&
^
|
(all letters, $, _)
```

例如，表达式

```scala
a + b ^? c ?^ d less a ==> b | c
```

等价于

```scala
((a + b) ^? (c ?^ d)) less ((a ==> b) | c)
```

其中出现的运算符按优先级从高到低依次是`?^`, `+`, `==>`, `^?`, `|`和`less`。

### 3.3.6 嵌套方法
<https://docs.scala-lang.org/tour/nested-functions.html>

在Scala中，可以嵌套方法定义。下面的`factorial()`方法用于计算给定数字的阶乘：

```scala
def factorial(x: Int): Int = {
  def fact(x: Int, accumulator: Int): Int = {
    if (x <= 1) accumulator else fact(x - 1, x * accumulator)
  }  
  fact(x, 1)
}

println("Factorial of 3: " + factorial(3))
```

### 3.3.7 零参数和无参数方法
**零参数**(zero-parameter)方法是指参数列表为空的方法。在定义时必须使用空括号，而在调用时括号是可选的。例如：

```scala
def sayHello(): Unit = println("Hello")

sayHello()
sayHello    // also OK
```

**无参数**(parameterless)方法类似于零参数方法，但在定义和调用时都不使用空括号（就像访问字段一样）。例如：

```scala
def getAnswer: Int = 42

val a = getAnswer
val a = getAnswer()  // error
```

另外，在Scala代码中调用零参数的Java方法时，空括号也是可选的。

最佳做法是：
* 定义不需要参数的Scala方法时，如果返回`Unit`则使用零参数，否则（getter方法）使用无参数。
* 调用零参数的Scala方法时添加空括号，调用无参数的Scala方法时不添加空括号。
* 调用零参数的Java方法时，如果返回`void`则添加空括号，否则（getter方法）不添加空括号。

| 语言 | 返回类型 | 参数定义 | 调用括号 |
| --- | --- | --- | --- |
| Scala | `Unit` | 零 | 可选（建议） |
| Scala | `Unit` | 无（不建议） | 不能 |
| Scala | 非`Unit` | 零（不建议） | 可选 |
| Scala | 非`Unit` | 无 | 不能 |
| Java | `void` | 零 | 可选（建议） |
| Java | 非`void` | 零 | 可选（不建议） |

例如，对于下面的Scala类`Foo`：

```scala
// Scala
class Foo {
  def doX(): Unit = {}
  def doY: Unit = {}   // not recommended: mutator method is parameterless
  def getX(): Int = 0  // not recommended: accessor method has empty parameter
  def getY: Int = 0
}
```

以及下面的Java类`Bar`：

```java
// Java
public class Bar {
    public void doX() {}
    public int getX() { return 0; }
}
```

下面的例子展示了如何在Scala代码中调用这些方法：

```scala
// Scala
var a = 0
val foo = new Foo
foo.doX()  // or foo.doX, but not recommended
foo.doY    // foo.doY() is an error
a = foo.getX()  // or foo.getX, but not recommended
a = foo.getY    // foo.getY() is an error

val bar = new Bar
bar.doX()     // or bar.doX, but not recommended
a = bar.getX  // or bar.getX(), but not recommended
```

### 3.3.8 多个参数列表
<https://docs.scala-lang.org/tour/multiple-parameter-lists.html>

方法可以有多个参数列表。例如，Scala集合API中`Iterable`特质的`foldLeft()`方法定义如下：

```scala
trait Iterable[A] {
  ...
  def foldLeft[B](z: B)(op: (B, A) => B): B = ...
}
```

`foldLeft()`将双参数函数`op`应用于初始值`z`和该集合的所有元素，从左向右（即`op(...op(op(z, e1), e2)..., eN)`）。下面是使用示例：

```scala
val numbers = List(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
val res = numbers.foldLeft(0)((m, n) => m + n)
println(res) // 55
```

`res`是0与`numbers`中所有元素相加的总和。

#### 用例
下面是多参数列表的一些建议用法。

（1）辅助类型参数推断

在Scala 2中，泛型方法的类型推断一次处理一个参数列表。假设有以下方法：

```scala
def foldLeft1[A, B](as: List[A], z: B, op: (B, A) => B): B = ???
```

以下调用方式会编译失败：

```scala
val notPossible = foldLeft1(numbers, 0, _ + _)
```

必须显式指定类型参数`A`、`B`，或者显式指定函数`op`的参数类型，像这样：

```scala
val firstWay = foldLeft1[Int, Int](numbers, 0, _ + _)
val secondWay = foldLeft1(numbers, 0, (a: Int, b: Int) => a + b)
```

否则编译器无法推断函数`_ + _`的类型，因为它仍然在推断`A`和`B`。

通过将参数`op`移动到单独的参数列表中，`A`和`B`就能在第一个参数列表中被推断出来。然后将其用于第二个参数列表，从而可以推断`_ + _`的类型为`(Int, Int) => Int`。

```scala
def foldLeft2[A, B](as: List[A], z: B)(op: (B, A) => B): B = ???
def possible = foldLeft2(numbers, 0)(_ + _)
```

这个定义不需要任何类型提示就可以推断出所有类型参数。

（2）隐式参数

为了将某些参数指定为`implicit`，必须将它们放在单独的隐式参数列表中（详见3.3.10节）。例如：

```scala
def execute(arg: Int)(implicit ec: ExecutionContext) = ???
```

（3）部分应用

当使用比声明少的参数列表调用方法时，将产生一个接受缺少的参数列表作为其参数的函数，这称为[部分应用](https://en.wikipedia.org/wiki/Partial_application)(partial application)。

例如：

```scala
val numbers = List(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
val numberFunc = numbers.foldLeft(List[Int]()) _ // ((List[Int], Int) => List[Int]) => List[Int]

val squares = numberFunc((xs, x) => xs :+ x*x)
println(squares) // List(1, 4, 9, 16, 25, 36, 49, 64, 81, 100)

val cubes = numberFunc((xs, x) => xs :+ x*x*x)
println(cubes)  // List(1, 8, 27, 64, 125, 216, 343, 512, 729, 1000)
```

#### 与柯里化的比较
有时会将具有多个参数列表的方法称为[柯里化](https://en.wikipedia.org/wiki/Currying)(currying)：将接受多个参数的函数转换为一系列接受单个参数的函数。

不建议使用 "curry" 一词指代Scala的多参数列表。原因有两个：
* Scala的多参数列表是作为语言的一部分直接实现的，而不是从单参数函数中派生出来的。
* 容易与Scala标准库的`curried`和`uncurried`方法混淆，它们根本不涉及多参数列表。

但是，多参数列表和柯里化之间有一定的相似之处。尽管定义语法不同，但调用语法看起来一样：

```scala
// version with multiple parameter lists
def addMultiple(n1: Int)(n2: Int) = n1 + n2

// two different ways of arriving at a curried version instead
def add(n1: Int, n2: Int) = n1 + n2
val addCurried1 = (add _).curried
val addCurried2 = (n1: Int) => (n2: Int) => n1 + n2

// regardless, all three call sites are identical
addMultiple(3)(4)  // 7
addCurried1(3)(4)  // 7
addCurried2(3)(4)  // 7
```

### 3.3.9 重复参数
<https://scala-lang.org/files/archive/spec/2.13/04-basic-declarations-and-definitions.html#repeated-parameters>

参数列表的最后一个参数可以添加后缀`*`（例如`x: T*`），称为**重复参数**(repeated parameter)。具有重复参数`T*`的方法可以接受任意数量的`T`类型参数。在方法内部，重复参数的类型是`Seq[T]`。

具有重复参数的方法不允许有默认参数。

例如，下面的方法计算可变数量的整数参数的和。

```scala
def sum(args: Int*) = {
  var result = 0
  for (arg <- args) result += arg
  result
}
```

```scala
println(sum())  // 0
println(sum(1))  // 1
println(sum(1, 2, 3))  // 6
```

如果要将一个数组或序列传递给重复参数，需要使用语法`: _*`。

```scala
val xs = List(1, 2, 3)
println(sum(xs: _*))  // 6
```

### 3.3.10 隐式参数
<https://docs.scala-lang.org/tour/implicit-parameters.html>

方法可以有**上下文参数**(contextual parameter)，也称为**隐式参数**(implicit parameter)。以关键字`implicit`开头的参数列表表示隐式参数。如果调用者没有显式提供这些参数，编译器将查找正确类型的`implicit`值。如果能找到合适的值，则自动传递。

考虑下面的例子。我们定义了一个特质`Comparator[A]`，用于比较`A`类型的值，并提供了`Int`和`String`两种实现。然后定义一个方法`max()`，返回两个参数的最大值。

```scala
trait Comparator[A] {
  def compare(x: A, y: A): Int
}

object Comparator {
  implicit object IntComparator extends Comparator[Int] {
    override def compare(x: Int, y: Int): Int = Integer.compare(x, y)
  }

  implicit object StringComparator extends Comparator[String] {
    override def compare(x: String, y: String): Int = x.compareTo(y)
  }
}

def max[A](x: A, y: A)(implicit comparator: Comparator[A]): A =
  if (comparator.compare(x, y) >= 0) x else y

println(max(10, 6))             // 10
println(max("hello", "world"))  // world
println(max(false, true))       // error: could not find implicit value for parameter comparator: Comparator[Boolean]
```

调用`max(10, 6)`和`max("hello", "world")`的`comparator`参数会分别自动填充为`Comparator.IntComparator`和`Comparator.StringComparator`。由于无法找到隐式的`Comparator[Boolean]`，因此调用`max(false, true)`编译失败。

Scala会在两个地方查找可用的隐式值：
* 首先在调用位置查找可直接访问（没有前缀）的隐式值（例如`max()`方法所在类的成员）。
* 然后在隐式候选类型的伴生对象中查找标记为`implicit`的成员（例如，候选类型`Comparator[Int]`的伴生对象`Comparator`）。

详见[FAQ - Where does Scala look for implicits?](https://docs.scala-lang.org/tutorials/FAQ/index.html#where-does-scala-look-for-implicits)

### 3.3.11 隐式转换
<https://docs.scala-lang.org/tour/implicit-conversions.html>

**隐式转换**(implicit conversion)是Scala的一个强大特性，有两个常见用法：
* 允许将一种类型的值传递给另一种类型的参数。
* 在Scala 2中为类提供额外的成员。

在Scala 2中，从类型`S`到类型`T`的隐式转换有3种定义方式：
* 具有单个`S`类型参数的[隐式类](https://docs.scala-lang.org/overviews/core/implicit-classes.html) `T`（必须定义在另一个`class`/`object`/`trait`内部）。

```scala
object Helpers {
  implicit class T(s: S)
}
```

* 具有函数类型`S => T`的隐式值。

```scala
implicit val s2t: S => T = ???
```

* 可转换为该类型值的隐式方法。

```scala
implicit def s2t(s: S): T = ???
```

隐式转换应用于两种情况：
* 如果表达式`e`的类型为`S`，而`S`不符合表达式的期望类型`T`。
* 在表达式`e.m`中，`e`的类型为`S`，而`m`不是`S`的成员。

在第一种情况下，会搜索可用于`e`且结果类型符合`T`的转换。一个例子是将一个`Int`传递给接受`Long`参数的方法，这种情况下会插入隐式转换`Int.int2long(x)`。

在第二种情况下，会搜索可用于`e`且结果类型包含名为`m`的成员的转换。一个例子是比较两个字符串`"foo" < "bar"`。`String`没有成员`<`，在这种情况下会插入隐式转换`Predef.augmentString("foo")`，其结果类型为`StringOps`，该类定义了`<`方法。（`scala.Predef`会自动导入到所有Scala程序中）

另一个例子是可以对数组调用`map()`等方法，这是因为隐式转换`Predef.xxxArrayOps()`能够将`Array[T]`转换为`ArrayOps[T]`，而后者定义了`map()`方法。

### 3.3.12 传名参数
<https://docs.scala-lang.org/tour/by-name-parameters.html>

**传名参数**(by-name parameter)在每次使用时都会求值，如果没有被使用则不会求值（这类似于**将传名参数替换为传递的表达式**）。相反，传值参数(by-value parameter)即使没有使用也会求值，但只求值一次。

要使一个参数传名调用，只需在其类型前添加`=>`。例如：

```scala
def byName(x: => Int): Unit = println(s"$x $x")

byName(Random.nextInt(10)) // possible output: 4 7
```

这个调用等价于`println(s"${Random.nextInt(10)} ${Random.nextInt(10)}")`，两个`x`生成了两个不同的随机数（在这里是4和7）。

作为对比，传值参数只会求值一次，两个`x`对应同一个随机数（在这里是8）：

```scala
def byValue(x: Int): Unit = println(s"$x $x")

byValue(Random.nextInt(10)) // possible output: 8 8
```

注：可以将传名参数理解为无参数的匿名函数，只是省略了空括号。上面的`byName()`方法等价于

```scala
def byName(x: () => Int): Unit = println(s"${x()} ${x()}")

byName(() => Random.nextInt(10))
```

下面的示例使用传名参数实现了`while`循环：

```scala
def whileLoop(condition: => Boolean)(body: => Unit): Unit = {
  if (condition) {
    body
    whileLoop(condition)(body)
  }
}
```

```scala
var i = 5

whileLoop (i > 0) {
  println(i)
  i -= 1
}  // prints 5 4 3 2 1
```

如果`condition`是`false`，则`body`永远不会被求值。如果`body`是计算密集型或运行时间较长的代码（如获取URL），这有助于提高性能。

## 3.4 包和导入
<https://docs.scala-lang.org/tour/packages-and-imports.html>

Scala使用**包**(package)来创建名称空间，使你能够模块化程序。

### 3.4.1 创建包
包是通过在Scala文件的顶部用关键字`package`声明一个或多个包名来创建的。

```scala
package users

class User
```

按照惯例，包名与Scala文件所在的目录名相同（但不是强制的）。然而，Scala对文件布局是不可知的。一个sbt项目的目录结构可能如下：

```
ExampleProject/
  build.sbt
  project/
  src/
    main/
      scala/
        users
          User.scala
          UserProfile.scala
          UserPreferences.scala
    test/
      scala/
```

users目录中的每个Scala文件都具有相同的包声明。

注：Java要求每个源文件只能有一个公有类，且类名必须与文件名相同，Scala没有这个要求。

声明包的另一种方式是将其嵌套在一起（类似于C++的命名空间）：

```scala
package users {
  package administrators {
    class NormalUser
  }
  package normalusers {
    class NormalUser
  }
}
```

包名应全部小写。如果开发代码的组织有网站，则包名应该遵循以下格式约定：`<top-level-domain>.<domain-name>.<project-name>`（即网站域名的逆序+项目名称，与Java相同）。例如，如果Google有一个名为SelfDrivingCar的项目，包名可能为像这样：

```scala
package com.google.selfdrivingcar.camera

class Lens
```

### 3.4.2 导入
`import`语句用于访问其他包中的成员（类、特质、函数等）。访问同一个包的成员不需要导入。

```scala
import users._  // import everything from the users package
import users.User  // import the class User
import users.{User, UserPreferences}  // Only imports selected members
import users.{UserPreferences => UPrefs}  // import and rename for convenience
```

Scala与Java的一个不同之处在于，导入可以在任何地方使用：

```scala
def sqrtplus1(x: Int) = {
  import scala.math.sqrt
  sqrt(x) + 1.0
}
```

如果发生命名冲突，你希望从项目的根包导入某些内容，则在包名前加上`_root_`：

```scala
import _root_.users._
```

注意：`scala`和`java.lang`包以及`object Predef`都是默认导入的。

## 3.5 单例对象
<https://docs.scala-lang.org/tour/singleton-objects.html>

**对象**(object)是只有一个实例的类。当它被引用时才会惰性创建。

作为顶级值，对象是一个单例类。作为类成员或局部值，其行为与`lazy val`完全一样。

### 3.5.1 定义单例对象
对象的定义类似于类，但使用关键字`object`：

```scala
package logging

object Logger {
  def info(message: String): Unit = println(s"INFO: $message")
}
```

方法`info()`可以从程序中的任何位置导入。创建像这样的辅助方法是单例对象的一种常见用法。

可以像这样在另一个包中使用`info()`：

```scala
import logging.Logger.info

class Project(name: String, daysToComplete: Int)

class Test {
  val project1 = new Project("TPS Reports", 1)
  val project2 = new Project("Website redesign", 5)
  info("Created projects")  // Prints "INFO: Created projects"
}
```

注
* 上面的`import`语句相当于Java的静态导入。
* 也可以导入`logging.Logger`然后调用`Logger.info()`。
* 如果一个对象不是顶级的，而是嵌套在另一个类中，那么该对象就“依赖于”外层类的实例。例如，假设`class A`具有成员`object B`，那么对于类`A`的两个实例`a1`和`a2`，`a1.B`和`a2.B`不是同一个对象。
* `object`不能被继承，但`object`可以继承`class`。

### 3.5.2 伴生对象
<https://docs.scala-lang.org/overviews/scala-book/companion-objects.html>

与类同名的对象称为类的**伴生对象**(companion object)。反过来，这个类是对象的**伴生类**(companion class)。

伴生类和对象可以互相访问私有成员。伴生对象用于非特定于类实例的方法和值。

注意：如果一个类或对象有伴生，则它们必须定义在同一个文件中。要在REPL中定义伴生，需要进入`:paste`模式。

```scala
import scala.math.{Pi, pow}

class Circle(val radius: Double) {
  def area: Double = Circle.calculateArea(radius)
}

object Circle {
  private def calculateArea(radius: Double): Double = Pi * pow(radius, 2.0)
}

val circle1 = new Circle(5.0)
val area = circle1.area
```

`class Circle`有一个特定于实例的成员`area`，单例`object Circle`有一个可用于所有实例的`calculateArea()`方法。

伴生对象还可以包含工厂方法：

```scala
class Email(val username: String, val domainName: String)

object Email {
  def fromString(emailString: String): Option[Email] = {
    emailString.split('@') match {
      case Array(a, b) => Some(new Email(a, b))
      case _ => None
    }
  }
}

val scalaCenterEmail = Email.fromString("scala.center@epfl.ch")
scalaCenterEmail match {
  case Some(email) => println(
    s"""Registered an email
       |Username: ${email.username}
       |Domain name: ${email.domainName}
    """.stripMargin)
  case None => println("Error: could not parse email")
}
```

`object Email`包含工厂方法`fromString()`，从字符串创建`Email`实例，返回一个`Option[Email]`以防解析错误。

Java程序员注释：
* Java中的静态成员相当于Scala中伴生对象的普通成员。
* 在Java代码中使用伴生对象时，其成员将被定义在伴生类中，并带有`static`修饰符（即使你自己没有定义伴生类）。这叫做静态转发(static forwarding)。
* Scala编译器会为`Email`生成两个类文件：Email.class和Email$.class，分别对应`class Email`和`object Email`。等价的Java代码如下所示：

```java
// Email.class
public class Email {
    private final String username;
    private final String domainName;

    public static Option<Email> fromString(final String emailString) {
        return Email$.MODULE$.fromString(emailString);
    }

    public String username() { return this.username; }
    public String domainName() { return this.domainName; }

    public Email(final String username, final String domainName) {
        this.username = username;
        this.domainName = domainName;
    }
}

// Email$.class
public final class Email$ {
    public static final Email$ MODULE$ = new Email$();

    public Option<Email> fromString(final String emailString) { ... }

    private Email$() {}
}
```

## 3.6 Case类
<https://docs.scala-lang.org/overviews/scala-book/case-classes.html>

<https://docs.scala-lang.org/tour/case-classes.html>

另一个支持函数式编程的Scala特性是**case类**。通过在`class`前添加关键字`case`来定义case类。

```scala
case class Person(name: String, relation: String)
```

注：Scala的case类类似于Java 16的`record`。

### 3.6.1 优点
Case类具有常规类的所有功能，但有更多优点。编译器会自动为case类生成以下代码：
* 构造函数参数默认是公有`val`字段，因此会为每个字段生成访问器方法。
* 在伴生对象中生成`apply()`方法，因此不需要使用`new`关键字来创建新实例。
* 在伴生对象中生成`unapply()`方法，使你可以在`match`表达式中使用case类。
* 生成`copy()`方法。你可能不会在OOP代码中使用这个特性，但它一直在FP中使用。
* 生成`equals()`和`hashCode()`方法，使你可以用`==`比较对象并将其用作映射的键。
* 生成默认的`toString()`方法，有助于调试。

（1）不需要`new`

定义`case`类时，不需要使用`new`关键字来创建新实例：

```scala
// "new" not needed before Person
scala> val christina = Person("Christina", "niece")
val christina: Person = Person(Christina,niece)
```

这是因为在`Person`的伴生对象中生成了`apply()`方法，等价的代码如下：

```scala
class Person(val name: String, val relation: String)

object Person {
  def apply(name: String, relation: String): Person = new Person(name, relation)
}
```

（2）没有修改器方法

Case类的构造函数参数默认是公有`val`字段，因此会为每个字段生成**访问器**(accessor)方法：

```scala
scala> christina.name
val res0: String = Christina
```

但不会生成**修改器**(mutator)方法：

```scala
// can't mutate the `name` field
scala> christina.name = "Fred"
                      ^
       error: reassignment to val
```

这意味着case类的实例是**不可变**的，因此是并发安全的。

注：也可以在case类中使用`var`字段，但不建议这样做，因为这违背了case类的初衷。

（3）`unapply()`方法

2.4.8介绍了如何编写`unapply()`方法。Case类的一个优点是它会自动生成`unapply()`方法，因此支持模式匹配并提取字段。这是case类最大的优点。

为了说明这一点，假设有以下特质：

```scala
trait Person {
  def name: String
}
```

然后创建两个扩展该特质的case类：

```scala
case class Student(name: String, year: Int) extends Person
case class Teacher(name: String, specialty: String) extends Person
```

可以像这样编写`match`表达式：

```scala
def getPrintableString(p: Person): String = p match {
  case Student(name, year) =>
    s"$name is a student in Year $year."
  case Teacher(name, specialty) =>
    s"$name teaches $specialty."
}
```

为了展示这段代码能够工作，首先创建一个`Student`和`Teacher`的实例：

```scala
val alice = Student("Alice", 1)
val bob = Teacher("Bob Donnan", "Mathematics")
```

接下来在REPL中使用这两个实例调用`getPrintableString()`：

```scala
scala> getPrintableString(alice)
val res0: String = Alice is a student in Year 1.

scala> getPrintableString(bob)
val res1: String = Bob Donnan teaches Mathematics.
```

模式`case Student(name, year) =>`能够工作，是因为`Student`被定义为case类，其`unapply()`方法的签名符合特定的标准：接受一个`Student`参数，返回包装在`Option`元组中的构造器字段。等价的代码如下：

```scala
object Student {
  def unapply(x: Student): Option[(String, Int)] = {
    if (x == null) None else Some((x.name, x.year))
  }
}
```

这种模式匹配称为**构造器模式**(constructor pattern)。

（4）`copy()`方法

Case类具有自动生成的`copy()`方法，可用于克隆整个对象，或者在克隆过程中更新一个或多个字段。例如：

```scala
scala> val alice = Student("Alice", 1)
val alice: Student = Student(Alice,1)

scala> val alice2 = alice.copy(year = 2)
val alice2: Student = Student(Alice,2)
```

如上所示，使用`copy()`方法时只需提供想要在克隆过程中修改的字段的名称，如果不提供参数则拷贝整个对象。这个过程称为“拷贝时更新”(update as you copy)。

注：`Student`类的`copy()`方法等价的代码如下：

```scala
def copy(name: String = this.name, year: Int = this.year) = Student(name, year)
```

（5）`equals()`和`hashCode()`方法

Case类自动生成了`equals()`和`hashCode()`方法，因此可以对实例进行相等比较：

```scala
scala> val bob = Student("Bob", 1)
val bob: Student = Student(Bob,1)

scala> val bob2 = alice.copy(name = "Bob")
val bob2: Student = Student(Bob,1)

scala> bob == bob2
val res0: Boolean = true

scala> bob eq bob2
val res1: Boolean = false
```

注：Scala的`==`和`eq`运算符分别等价于Java的`equals()`和`==`。

这些方法使你可以在集和映射中使用case类。

（6）`toString()`方法

最后，case类有一个默认的`toString()`方法实现，这在调试代码时很有帮助：

```scala
scala> s"alice is $alice"
val 3: String = alice is Student(Alice,1)
```

### 3.6.2 Case对象
<https://docs.scala-lang.org/overviews/scala-book/case-objects.html>

`case object`类似于`object`。但就像case类比常规类具有更多特性一样，case对象也比常规对象具有更多特性：
* 是可序列化的
* 具有默认的`hashCode()`实现
* 具有改进的`toString()`实现

Case对象主要用于两个地方：
* 创建枚举
* 创建在其他对象之间传递的“消息”容器（例如使用[Akka](https://akka.io/) actors库）

（1）用case对象创建枚举

在Scala中可以像这样创建枚举：

```scala
sealed trait Size
case object Small extends Size
case object Medium extends Size
case object Large extends Size
```

创建枚举的另一种方式见3.7节。

（2）将case对象用作消息

Case对象的另一种用途是表示“消息”的概念。例如，假设你正在编写一个像Amazon Alexa这样的智能语音助理应用，你希望能够传递“说话”消息，比如“说出指定的文本”、“停止说话”、“暂停”和“继续”。你可以为这些消息创建case对象，如下所示：

```scala
case class StartSpeakingMessage(text: String)
case object StopSpeakingMessage
case object PauseSpeakingMessage
case object ResumeSpeakingMessage
```

如果使用Akka库的[Actor类](https://doc.akka.io/api/akka-core/2.10/akka/actor/Actor.html)，就可以编写这样的代码：

```scala
class Speak extends Actor {
  def receive = {
    case StartSpeakingMessage(text) =>
      // code to speak the text
    case StopSpeakingMessage =>
      // code to stop speaking
    case PauseSpeakingMessage =>
      // code to pause speaking
    case ResumeSpeakingMessage =>
      // code to resume speaking
  }
}
```

## 3.7 枚举
Scala 2没有内置的枚举类型，必须通过其他方式来定义枚举。

（1）case对象

可以通过定义一个特质和多个实现该特质的case对象来创建枚举，这种方式已经在3.6.2节介绍过。

（2）`Enumeration`类

<https://www.scala-lang.org/api/2.13.18/scala/Enumeration.html>

创建枚举的另一种方式是扩展`Enumeration`类。例如：

```scala
object WeekDay extends Enumeration {
  type WeekDay = Value
  val Mon, Tue, Wed, Thu, Fri, Sat, Sun = Value
}
```

其中，第一行中的`Value`是`Enumeration`的表示枚举值的成员类型。在这里将其定义为别名`WeekDay`，在外部需要通过`WeekDay.WeekDay`来表示这个枚举类型。

第二行中的`Value`是`Enumeration`的无参数方法，返回下一个枚举值。在这里同时为多个变量赋值，等价于以下代码：

```scala
val Mon = Value
val Tue = Value
...
val Sun = Value
```

可以像这样使用`WeekDay`枚举：

```scala
import WeekDay._

def isWorkingDay(d: WeekDay): Boolean = !(d == Sat || d == Sun)

WeekDay.values.filter(isWorkingDay).foreach(println)
```

注意，`isWorkingDay()`的参数类型是`WeekDay.WeekDay`，而最后一行访问的是`object WeekDay`。

可以通过扩展`Enumeration.Val`类为枚举添加属性：

```scala
object Color extends Enumeration {
  case class Color(rgb: Int) extends super.Val {
    def red: Byte = ((rgb >> 16) & 0xFF).toByte
    def green: Byte = ((rgb >> 8) & 0xFF).toByte
    def blue: Byte = (rgb & 0xFF).toByte
  }

  val Black = Color(0x000000)
  val Red = Color(0xFF0000)
  val Green = Color(0x00FF00)
  val Blue = Color(0x0000FF)
  val Yellow = Color(0xFFFF00)
  val Magenta = Color(0xFF00FF)
  val Cyan = Color(0x00FFFF)
  val White = Color(0xFFFFFF)
}
```

注：Scala 3支持使用关键字`enum`创建枚举，详见[Scala 3 Book - Enums](https://docs.scala-lang.org/scala3/book/domain-modeling-tools.html#Enums_Scala_3_only)。

## 3.8 内部类
<https://docs.scala-lang.org/tour/inner-classes.html>

在Scala中，可以让类作为其他类的成员，称为**内部类**(inner class)。与Java不同的是，在Scala中内部类是绑定到外部类**实例**的。

假设类`Graph`和`Node`分别表示图和节点。如果我们希望编译器在编译时防止将属于不同图的节点连接起来，可以将`Node`作为`Graph`的内部类：

```scala
class Graph {
  class Node {
    var connectedNodes: List[Node] = Nil

    def connectTo(node: Node): Unit = {
      if (!connectedNodes.exists(node.equals)) {
        connectedNodes = node :: connectedNodes
      }
    }
  }

  var nodes: List[Node] = Nil

  def newNode: Node = {
    val res = new Node
    nodes = res :: nodes
    res
  }
}
```

在这里将图表示为节点的列表(`List[Node]`)。每个节点都有一个它连接到的其他节点的列表(`connectedNodes`)。类`Node`是一个**路径相关的类型**(path-dependent type)，因为它嵌套在类`Graph`中。因此，`connectedNodes`中的所有节点都必须使用同一个`Graph`实例中的`newNode`创建。

```scala
val graph1: Graph = new Graph
val node1: graph1.Node = graph1.newNode
val node2: graph1.Node = graph1.newNode
val node3: graph1.Node = graph1.newNode
node1.connectTo(node2)
node3.connectTo(node1)
```

其中`node1`, `node2`和`node3`的类型是`graph1.Node`，即绑定到实例`graph1`的`Node`类。

如果现在有两个图，Scala的类型系统不允许将一个图的节点连接到另一个图的节点，因为它们具有不同的类型：

```scala
val graph1: Graph = new Graph
val node1: graph1.Node = graph1.newNode
val node2: graph1.Node = graph1.newNode
node1.connectTo(node2)  // legal
val graph2: Graph = new Graph
val node3: graph2.Node = graph2.newNode
node1.connectTo(node3)  // illegal!
```

这是因为类型`graph1.Node`不同于类型`graph2.Node`。

在Java中，前面示例中的最后一行是正确的，因为两个图的节点具有相同的类型`Graph.Node`。在Scala中，也可以表达这样的类型，写作`Graph#Node`。如果我们希望能够连接不同图的节点，需要这样修改类的定义：

```scala
class Graph {
  class Node {
    var connectedNodes: List[Graph#Node] = Nil

    def connectTo(node: Graph#Node): Unit = {
      if (!connectedNodes.exists(node.equals)) {
        connectedNodes = node :: connectedNodes
      }
    }
  }

  var nodes: List[Node] = Nil

  def newNode: Node = {
    val res = new Node
    nodes = res :: nodes
    res
  }
}
```

## 3.9 注解
<https://docs.scala-lang.org/tour/annotations.html>

**注解**(annotation)用于将元信息与定义相关联。

注解应用于其后的第一个定义或声明。定义和声明之前可以有多个注解，注解的顺序不重要。

### @deprecated
方法前的注解`@deprecated`会导致编译器在调用该方法时打印警告。

```scala
object DeprecationDemo extends App {
  @deprecated("deprecation message", "1.2.3")
  def hello = "hola"

  hello
}
```

编译器会打印警告 "warning: 1 deprecation" ，添加选项`-deprecation`查看详细信息。

```shell
$ scalac DeprecationDemo.scala
warning: 1 deprecation (since 1.2.3); re-run with -deprecation for details
1 warning

$ scalac -deprecation DeprecationDemo.scala
DeprecationDemo.scala:5: warning: method hello in object DeprecationDemo is deprecated (since 1.2.3): deprecation message
  hello
  ^
1 warning
```

### @tailrec
有些注解在不满足条件时会导致编译失败。例如，注解`@tailrec`确保方法是[尾递归](https://en.wikipedia.org/wiki/Tail_call)(tail-recursive)的。尾递归是递归的一种特殊形式，即一个方法的所有递归调用都出现在方法的末尾。尾递归可以保持常数内存复杂度，这使得编译器可以进行优化。

下面是尾递归的计算阶乘方法：

```scala
import scala.annotation.tailrec

def factorial(x: Int): Int = {
  @tailrec
  def factorialHelper(x: Int, accumulator: Int): Int = {
    if (x == 1) accumulator else factorialHelper(x - 1, accumulator * x)
  }
  factorialHelper(x, 1)
}
```

注解`@tailrec`可以确保`factorialHelper()`方法确实是尾递归的。如果将该方法的实现改为下面这样，编译会失败：

```scala
def factorial(x: Int): Int = {
  @tailrec
  def factorialHelper(x: Int): Int = {
    if (x == 1) 1 else x * factorialHelper(x - 1)
  }
  factorialHelper(x)
}
```

会看到错误消息

```
error: could not optimize @tailrec annotated method factorialHelper: it contains a recursive call not in tail position
```

### Java注解
可以在Scala代码中使用Java注解。注意：确保使用`-target:jvm-1.8`选项。

例如，假设有一个用于跟踪类的源代码的注解

```java
@interface Source {
    public String url();
    public String mail();
}
```

在Scala中可以使用命名参数实例化注解：

```scala
@Source(url = "https://coders.com/", mail = "support@coders.com")
class MyScalaClass ...
```

如果注解只包含一个（没有默认值的）元素，按照惯例，如果将其命名为`value`，就可以使用类似构造器的语法：

```java
@interface SourceURL {
    public String value();
    public String mail() default "";
}
```

```scala
@SourceURL("https://coders.com/")
class MyScalaClass ...
```

也可以只命名`email`参数：

```scala
@SourceURL("https://coders.com/", mail = "support@coders.com")
class MyScalaClass ...
```

还有一些预定义注解。例如，字段注解`@transient`和`@volatile`分别等价于Java的`transient`和`volatile`修饰符。详见[Scala语言规范-Annotations](https://scala-lang.org/files/archive/spec/2.13/11-annotations.html)。
