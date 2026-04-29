---
title: Scala基础教程 第6节 集合类
date: 2026-04-15 22:16:05 +0800
categories: [Scala]
tags: [scala, collection, sequence, array list, linked list, vector, map, set, function]
---
<https://docs.scala-lang.org/overviews/scala-book/collections-101.html>

如果你从Java转向Scala，最好忘记Java集合类并开始使用Scala集合类。

## 6.1 主要集合类
常用的主要Scala集合类如下表所示：

| 类 | 描述 |
| --- | --- |
| `Seq` | 序列基类 |
| `ArrayBuffer` | 有索引的可变序列 |
| `List` | 链表，不可变序列 |
| `Vector` | 有索引的不可变序列 |
| `Map` | 映射（键值对）基类 |
| `Set` | 集基类 |

`Seq`、`Map`和`Set`都有可变和不可变两种版本。

本节将介绍这些类的基本用法。

注意：**不可变**(immutable)集合类可以安全地用于函数式编程(FP)风格。这些类的方法不会修改集合，而是返回一个新集合。

更详细的Scala集合文档：
* [Scala Collections (2.13)](https://docs.scala-lang.org/overviews/collections-2.13/introduction.html)
* [Scala Collections (2.8-2.12)](https://docs.scala-lang.org/overviews/collections/introduction.html)

### 6.1.1 ArrayBuffer类
<https://docs.scala-lang.org/overviews/scala-book/arraybuffer-examples.html>

[ArrayBuffer[T]](https://www.scala-lang.org/api/2.13.18/scala/collection/mutable/ArrayBuffer.html)类是**可变的**(mutable)序列，因此可以使用它的方法来修改其内容。`ArrayBuffer`类似于Java的`ArrayList`。

为了使用`ArrayBuffer`，必须先导入它：

```scala
import scala.collection.mutable.ArrayBuffer
```

像这样创建空的`ArrayBuffer`：

```scala
val nums = ArrayBuffer[Int]()
val names = ArrayBuffer.empty[String]
```

也可以像这样指定初始元素：

```scala
val nums = ArrayBuffer(1, 2, 3)
```

使用`()`访问具有给定索引的元素：

```scala
val x = nums(1)  // 2
nums(2) = 333
```

`length`或`size`方法返回元素个数：

```scala
val n = nums.size  // 3, same as nums.length
```

使用`+=`添加元素（等价于`addOne()`方法）：

```scala
// add one element
nums += 4  // ArrayBuffer(1, 2, 3, 4)
```

`+=`运算符返回当前对象，因此可以像这样添加多个元素：

```scala
// add multiple elements
nums += 5 += 6  // ArrayBuffer(1, 2, 3, 4, 5, 6)
```

使用`++=`添加另一个集合的所有元素（等价于`addAll()`方法）：

```scala
// add multiple elements from another collection
nums ++= List(7, 8, 9)  // ArrayBuffer(1, 2, 3, 4, 5, 6, 7, 8, 9)
```

使用`-=`删除元素的第一次出现，如果不存在则不变（等价于`subtractOne()`方法）：

```scala
// remove one element
nums -= 9  // ArrayBuffer(1, 2, 3, 4, 5, 6, 7, 8)

// remove multiple elements
nums -= 7 -= 8  // ArrayBuffer(1, 2, 3, 4, 5, 6)
```

使用`--=`删除另一个集合的所有元素（等价于`subtractAll()`方法）：

```scala
// remove multiple elements using another collection
nums --= Array(5, 6)  // ArrayBuffer(1, 2, 3, 4)
```

下面展示了`ArrayBuffer`的更多方法：

```scala
val a = ArrayBuffer(1, 2, 3)         // ArrayBuffer(1, 2, 3)
a.append(4)                          // ArrayBuffer(1, 2, 3, 4)
a.appendAll(Seq(5, 6))               // ArrayBuffer(1, 2, 3, 4, 5, 6)
a.clear()                            // ArrayBuffer()

val a = ArrayBuffer(9, 10)           // ArrayBuffer(9, 10)
a.insert(0, 8)                       // ArrayBuffer(8, 9, 10)
a.insertAll(0, Vector(4, 5, 6, 7))   // ArrayBuffer(4, 5, 6, 7, 8, 9, 10)
a.prepend(3)                         // ArrayBuffer(3, 4, 5, 6, 7, 8, 9, 10)
a.prependAll(Array(0, 1, 2))         // ArrayBuffer(0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10)

val a = ArrayBuffer.range('a', 'h')  // ArrayBuffer(a, b, c, d, e, f, g)
a.remove(0)                          // ArrayBuffer(b, c, d, e, f, g)
a.remove(2, 3)                       // ArrayBuffer(b, c, g)

val a = ArrayBuffer.range('a', 'h')  // ArrayBuffer(a, b, c, d, e, f, g)
a.dropInPlace(2)                     // ArrayBuffer(c, d, e, f, g)
a.dropRightInPlace(2)                // ArrayBuffer(c, d, e)
```

`ArrayBuffer`类有数百个方法，这里仅仅展示了冰山一角。完整列表参见API文档。

### 6.1.2 List类
<https://docs.scala-lang.org/overviews/scala-book/list-class.html>

[List[T]](https://www.scala-lang.org/api/2.13.18/scala/collection/immutable/List.html)类是不可变链表(linked list)。如果想要添加或删除元素，需要从现有的`List`创建一个新的`List`。

#### 创建列表
像这样创建列表：

```scala
val nums = List(1, 2, 3)
val names = List("Joel", "Chris", "Ed")
```

#### 添加元素
`List`是不可变的，因此不能直接向其中添加新元素，而是通过在现有列表的开头或末尾追加元素来创建新列表。例如，给定列表

```scala
val a = List(1, 2, 3)
```

使用`+:`在开头追加元素，`x +: a`等价于`a.prepended(x)`（以`:`结尾的方法是右结合的）。

```scala
val b = 0 +: a  //  List(0, 1, 2, 3)
```

使用`++:`在开头追加另一个集合的所有元素，`b ++: a`等价于`a.prependedAll(b)`。

```scala
val b = List(-1, 0) ++: a  // List(-1, 0, 1, 2, 3)
```

还可以使用`a :+ x`（等价于`a.appended(x)`）和`a :++ b`（等价于`a.appendedAll(b)`）向列表末尾追加元素。但由于`List`是单向链表，应该只在开头追加元素，在末尾追加元素是相对较慢的操作（尤其是当列表很大时）。

提示：如果想在不可变序列的开头和末尾追加元素，应使用`Vector`。

由于`List`是链表，不应该试图通过索引访问元素（例如，`myList(999999)`需要很长时间）。如果想按索引访问，应使用`Vector`或`ArrayBuffer`。

#### 方法名助记
`+:` vs `:+` 冒号(COLon)位于集合(COLlection)侧

```scala
1 +: List(2, 3)  //  List(1, 2, 3)
List(1, 2) :+ 3  //  List(1, 2, 3)
```

`++:` vs `:++` 冒号位于新集合类型侧

```scala
Array(1, 2) ++: List(3, 4)  // List(1, 2, 3, 4)
Array(1, 2) :++ List(3, 4)  // Array(1, 2, 3, 4)
Array(1, 2) ++ List(3, 4)   // Array(1, 2, 3, 4)
```

这些方法名对于其他不可变序列类（如`Seq`和`Vector`）同样适用。

#### 遍历列表
2.2节已经介绍过，可以使用`for`循环或`foreach()`方法来遍历列表。

```scala
val names = List("Joel", "Chris", "Ed")
for (name <- names) println(name) // prints: Joel Chris Ed
names.foreach(println)  // prints: Joel Chris Ed
```

这种方式适用于所有序列类。

#### ::和Nil
Scala的`List`类似于Lisp语言的`List`类。还可以像这样创建列表：

```scala
val list = 1 :: 2 :: 3 :: Nil  // same as List(1, 2, 3)
```

实际上，`List`类有两个子类：
* `case class ::(head, next)`表示非空列表。
* `case object Nil`表示空列表。

另外，`List`类有两个冒号方法：
* `x :: a`在开头追加元素，返回`new ::(x, this)`，等价于`+:`/`prepended()`。
* `b ::: a`在开头追加另一个列表的所有元素，等价于`++:`/`prependedAll()`。

### 6.1.3 Vector类
<https://docs.scala-lang.org/overviews/scala-book/vector-class.html>

[Vector[T]](https://www.scala-lang.org/api/2.13.18/scala/collection/immutable/Vector.html)类是有索引的不可变序列。“有索引的”(indexed)意味着支持快速随机访问（例如`listOfPeople(999999)`）。除此之外，`Vector`的用法与`List`相同。

创建`Vector`：

```scala
val nums = Vector.range(1, 6)  // Vector(1, 2, 3, 4, 5)
val names = Vector("Joel", "Chris", "Ed")
```

在开头追加元素：

```scala
val a = Vector(1, 2, 3)
val b = a :+ 4  // Vector(1, 2, 3, 4)
val c = a :++ Vector(4, 5)  // Vector(1, 2, 3, 4, 5)
```

注：在这里`:++`/`appendedAll()`等价于`++`/`concat()`。

在末尾追加元素：

```scala
val b = 0 +: a  // Vector(0, 1, 2, 3)
val c = Vector(-1, 0) ++: a  // Vector(-1, 0, 1, 2, 3)
```

遍历方式同`List`：

```scala
for (name <- names) println(name)
```

### 6.1.4 Map类
<https://docs.scala-lang.org/overviews/scala-book/map-class.html>

[Map[K, V]](https://www.scala-lang.org/api/2.13.18/scala/collection/Map.html)类描述了**映射**(map)——由键值对构成的可迭代集合。

#### 创建映射
像这样通过指定的键值对创建映射：

```scala
val states = Map(
  "AK" -> "Alaska",
  "IL" -> "Illinois",
  "KY" -> "Kentucky"
) // scala.collection.immutable.Map[String, String]
```

注：`->`是隐式类`scala.Predef.ArrowAssoc`中定义的方法。对于任何对象，`x -> y`返回二元组`(x, y)`。因此上面的映射等价于`Map(("AK", "Alaska"), ...)`。

`Map`默认是不可变的。要创建可变映射，需要使用`scala.collection.mutable.Map`：

```scala
import scala.collection.mutable
val states = mutable.Map("AK" -> "Alaska")
```

#### 访问元素
使用`()`访问给定的键关联的值（等价于`apply()`方法），如果不存在则抛出`NoSuchElementException`。

```scala
val ak = states("AK")  // "Alaska"
val ca = states("CA")  // NoSuchElementException: key not found
```

`get()`方法类似于`()`，但返回一个`Option[V]`，当键不存在时返回`None`而不是抛出异常。

```scala
val ak = states.get("AK")  // Some(Alaska)
val ca = states.get("CA")  // None
```

#### 添加元素
对于可变映射，使用`+=`添加单个元素（键值对）：

```scala
states += ("AL" -> "Alabama")
states += ("AR" -> "Arkansas", "AZ" -> "Arizona")
```

使用`++=`添加另一个映射的所有元素：

```scala
states ++= Map("CA" -> "California", "CO" -> "Colorado")
```

#### 删除元素
使用`-=`和`--=`删除指定的键，如果不存在则不变：

```scala
states -= "AR"
states -= ("AL", "AZ")
states --= List("AL", "AZ")
```

#### 更新元素
通过将键关联到新值来更新映射元素。如果键已存在则覆盖旧值，否则插入新键值对。

```scala
states("AK") = "Alaska, A Really Big State"
```

`m(k) = v`等价于`m.update(k, v)`以及`m += (k -> v)`。

#### 遍历映射
2.2.3节已经介绍过遍历映射元素的几种方式。对于映射

```scala
val ratings = Map(
  "Lady in the Water" -> 3.0, 
  "Snakes on a Plane" -> 4.0,
  "You, Me and Dupree" -> 3.5
)
```

可以使用`for`循环遍历键值对：

```scala
for ((name, rating) <- ratings)
  println(s"Movie: $name, Rating: $rating")
```

也可以使用`foreach()`方法：

```scala
ratings.foreach { case (name, rating) =>
  println(s"Movie: $name, Rating: $rating")
}
```

注：特质`Map`的默认实现类是`HashMap`，键是无序的。子特质`SortedMap`及其实现类`TreeMap`是有序的。

更多用法参见[Map类](https://docs.scala-lang.org/overviews/collections-2.13/maps.html)和API文档。

### 6.1.5 Set类
<https://docs.scala-lang.org/overviews/scala-book/set-class.html>

[Set[T]](https://www.scala-lang.org/api/2.13.18/scala/collection/Set.html)类是没有重复元素的可迭代集合。

#### 创建集
与`Map`类似，`Set`类也有可变和不可变版本。

创建不可变集：

```scala
val s = Set(1, 2, 3)  // scala.collection.immutable.Set[Int]
```

创建可变集：

```scala
import scala.collection.mutable
val s = mutable.Set(1, 2, 3)
```

#### 成员检查
使用`()`或`contains()`方法检查一个元素是否属于集：

```scala
s(3)  // true
s.contains(8)  // false
```

#### 添加元素
对于可变集，使用`+=`和`++=`添加元素：

```scala
val s = mutable.Set.empty[Int]
s += 1  // Set(1)
s += 2 += 3  // Set(1, 2, 3)
s ++= Vector(4, 5)  // Set(1, 2, 3, 4, 5)
```

注意，尝试添加已存在的元素将会被忽略。

`Set`还有一个`add()`方法，如果确实添加了元素则返回`true`，否则返回`false`：

```scala
s.add(6)  // true
s.add(5)  // false
```

#### 删除元素
使用`-=`和`--=`从集删除元素，不存在的元素将被忽略：

```scala
val s = mutable.Set(1, 2, 3, 4, 5)
s -= 1  // Set(2, 3, 4, 5)
s --= Array(2, 3)  // Set(4, 5)
```

`remove()`方法删除元素并返回布尔值，`clear()`方法清空集。

```scala
val s = mutable.Set(1, 2, 3, 4, 5)
s.remove(2)  // true, s = Set(1, 3, 4, 5)
s.remove(40)  // false, s = Set(1, 3, 4, 5)
s.clear()  // Set()
```

#### 集合操作
* 交集：`a & b`（等价于`intersect()`）
* 并集：`a | b`（等价于`union()`）
* 差集：`a &~ b`（等价于`diff()`）

`Set`的默认实现类是`HashSet`，另外还有`TreeSet`、`LinkedHashSet`等。

更多用法参见[Set类](https://docs.scala-lang.org/overviews/collections-2.13/sets.html)和API文档。

## 6.2 匿名函数
<https://docs.scala-lang.org/overviews/scala-book/anonymous-functions.html>

本节将介绍函数式编程的一个特性，称为**匿名函数**(anonymous functions)。

### 6.2.1 单参数函数
给定一个列表：

```scala
val ints = List(1, 2, 3)
```

`List[A]`的`map()`方法接受一个`A => B`类型的函数`f`，将每个元素`x`映射为`f(x)`，返回一个新列表`List[B]`：

```scala
def map[B](f: A => B): List[B]
```

可以通过`map()`方法将`int`中的每个元素加倍来创建一个新列表，如下所示：

```scala
val doubledInts = ints.map(_ * 2)  // List(2, 4, 6)
```

在这个示例中，`_ * 2`是一个匿名函数（类型为`Int => Int`），表示“将每个元素乘以2”。

如果愿意，也可以用`=>`语法编写匿名函数。上面的代码等价于

```scala
val doubledInts = ints.map(i => i * 2)
val doubledInts = ints.map((i: Int) => i * 2)
```

这三种写法的含义相同。

这个`map()`示例等价于以下Java代码：

```java
List<Integer> ints = new ArrayList<>(Arrays.asList(1, 2, 3));

// the `map` process
List<Integer> doubledInts = ints.stream()
    .map(i -> i * 2)
    .collect(Collectors.toList());
```

还等价于Scala的`for`表达式：

```scala
val doubledInts = for (i <- ints) yield i * 2
```

使用匿名函数的另一个例子是`List`类的`filter()`方法。该方法接受一个`A => Boolean`类型的谓词函数`p`，返回一个新列表`List[A]`，其中仅包含使`p(x)`为`true`的元素`x`：

```scala
def filter(p: A => Boolean): List[A]
```

给定以下列表：

```scala
val ints = List.range(1, 10)  // List(1, 2, 3, 4, 5, 6, 7, 8, 9)
```

可以像这样创建一个包含`ints`中所有大于5的整数的新列表：

```scala
val a = ints.filter(_ > 5)  // List(6, 7, 8, 9)
```

这行代码等价于

```scala
def lessThanFive(i: Int): Boolean = i < 5
val a = ints.filter(lessThanFive)
```

像这样创建只包含偶数的列表：

```scala
val b = ints.filter(_ % 2 == 0)  // List(2, 4, 6, 8)
```

### 6.2.2 多参数函数
`List[A]`的`reduce()`方法接受一个`(A, A) => A`类型的二元运算符`op`，将`op`依次应用于每个元素，返回最终结果`op(... op(op(x0, x1), x2)..., x(N-1))`。

```scala
def reduce(op: (A, A) => A): A
```

例如，可以像这样计算列表中所有元素的和：

```scala
val a = List.range(1, 101)  // List(1, 2, ..., 100)
val s = a.reduce(_ + _)  // 5050
```

其中，匿名函数`_ + _`等价于`(x, y) => x + y`。

另外也可以直接调用`sum`方法：

```scala
val s = a.sum
```

### 6.2.3 Java函数式接口
从Scala 2.12起，匿名函数可以直接充当Java函数式接口，即单一抽象方法类型(single abstract method, SAM)。这使得在Scala中使用Java库变得更容易。例如：

```scala
scala> val r: Runnable = () => println("Run!")
val r: Runnable = $Lambda$1292/0x000001f44c5b6a08@248227db

scala> r.run()
Run!
```

注意，只有匿名函数可以转换为SAM类型的实例，而`FunctionN`类型的任意表达式不可以：

```scala
scala> val f = () => println("Faster!")
val f: () => Unit = $Lambda$1294/0x000001f44c588800@73032867

scala> val fasterRunnable: Runnable = f
                                      ^
       error: type mismatch;
        found   : () => Unit
        required: Runnable
```

参见
* [Lambda syntax for SAM types](https://www.scala-lang.org/news/2.12.0/#lambda-syntax-for-sam-types)
* [Scala语言规范 - SAM conversion](https://www.scala-lang.org/files/archive/spec/2.12/06-expressions.html#sam-conversion)

## 6.3 常用集合方法
### 6.3.1 常用序列方法
<https://docs.scala-lang.org/overviews/scala-book/collections-methods.html>

<https://docs.scala-lang.org/overviews/collections-2.13/seqs.html>

### 6.3.2 常用映射方法
<https://docs.scala-lang.org/overviews/scala-book/collections-maps.html>

<https://docs.scala-lang.org/overviews/collections-2.13/maps.html>

## 6.4 可变和不可变集合
<https://docs.scala-lang.org/overviews/collections-2.13/overview.html>
scala.Seq, scala.collections.Seq, scala.collections.mutable.Seq, scala.collections.immutable.Seq

## 数组
<https://docs.scala-lang.org/overviews/collections-2.13/arrays.html>

## 字符串
<https://docs.scala-lang.org/overviews/collections-2.13/strings.html>

## 工厂方法
<https://docs.scala-lang.org/overviews/collections-2.13/creating-collections-from-scratch.html>

## 视图
<https://docs.scala-lang.org/overviews/collections-2.13/views.html>
.view

## 并行集合
<https://docs.scala-lang.org/overviews/parallel-collections/overview.html>
<https://github.com/scala/scala-parallel-collections>
.par（Scala 2.13+需要依赖库）

## 与Java集合互相转换
<https://docs.scala-lang.org/overviews/collections-2.13/conversions-between-java-and-scala-collections.html>
asScala, asJava

## CanBuildFrom
<https://docs.scala-lang.org/overviews/core/architecture-of-scala-collections.html>

## 偏函数

## 高阶函数
<https://docs.scala-lang.org/tour/higher-order-functions.html>
