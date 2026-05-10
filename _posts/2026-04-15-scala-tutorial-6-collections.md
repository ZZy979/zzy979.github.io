---
title: Scala基础教程 第6节 集合类
date: 2026-04-15 22:16:05 +0800
categories: [Scala]
tags: [scala, collection, sequence, array list, linked list, vector, map, set, array, generic array, string, function, view]
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

注意：不可变集合类可以安全地用于函数式编程(FP)风格。这些类的方法不会修改集合，而是返回一个新集合。

更详细的Scala集合文档：
* [Scala Collections (2.13)](https://docs.scala-lang.org/overviews/collections-2.13/introduction.html)
* [Scala Collections (2.8-2.12)](https://docs.scala-lang.org/overviews/collections/introduction.html)
* [The Architecture of Scala Collections](https://docs.scala-lang.org/overviews/core/architecture-of-scala-collections.html)

### 6.1.1 ArrayBuffer类
<https://docs.scala-lang.org/overviews/scala-book/arraybuffer-examples.html>

[ArrayBuffer[T]](https://www.scala-lang.org/api/2.13.18/scala/collection/mutable/ArrayBuffer.html)类是可变的序列，因此可以使用它的方法来修改其内容。`ArrayBuffer`类似于Java的`ArrayList`。

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

`isEmpty`和`nonEmpty`方法分别返回集合是否为空、是否非空。

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

[List[T]](https://www.scala-lang.org/api/2.13.18/scala/collection/immutable/List.html)类是不可变的链表(linked list)。如果想要添加或删除元素，需要从现有的`List`创建一个新的`List`。

#### 创建列表
像这样创建列表：

```scala
val nums = List(1, 2, 3)
val names = List("Joel", "Chris", "Ed")
```

注：这实际上调用了`List`类的伴生对象的`apply()`方法。也可以使用抽象基类来创建序列，这会使用其默认实现。例如：

```scala
scala> Seq(1, 2, 3)
val res0: Seq[Int] = List(1, 2, 3)

scala> Iterable(1, 2, 3)
val res1: Iterable[Int] = List(1, 2, 3)
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
for (name <- names) println(name) // prnums: Joel Chris Ed
names.foreach(println)  // prnums: Joel Chris Ed
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

在末尾追加元素：

```scala
val a = Vector(1, 2, 3)
val b = a :+ 4  // Vector(1, 2, 3, 4)
val c = a :++ Vector(4, 5)  // Vector(1, 2, 3, 4, 5)
```

注：在这里`:++`/`appendedAll()`等价于`++`/`concat()`。

在开头追加元素：

```scala
val b = 0 +: a  // Vector(0, 1, 2, 3)
val c = Vector(-1, 0) ++: a  // Vector(-1, 0, 1, 2, 3)
```

遍历方式同`List`：

```scala
for (name <- names) println(name)
names.foreach(println)
```

### 6.1.4 Map类
<https://docs.scala-lang.org/overviews/scala-book/map-class.html>

[Map[K, V]](https://www.scala-lang.org/api/2.13.18/scala/collection/Map.html)类描述了**映射**(map)——由键值对构成的集合。

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
使用`()`访问与给定的键关联的值（等价于`apply()`方法），如果不存在则抛出`NoSuchElementException`。

```scala
val ak = states("AK")  // "Alaska"
val ca = states("CA")  // NoSuchElementException: key not found
```

`get()`方法类似于`()`，但返回一个`Option[V]`，当键不存在时返回`None`。

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

[Set[T]](https://www.scala-lang.org/api/2.13.18/scala/collection/Set.html)类是没有重复元素的集合。

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
使用`-=`和`--=`删除元素，不存在的元素将被忽略：

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

### 6.1.6 数组
<https://docs.scala-lang.org/overviews/collections-2.13/arrays.html>

#### 数组与序列兼容
在Scala中，数组[Array[T]](https://www.scala-lang.org/api/2.13.18/scala/Array.html)是一种特殊的集合。一方面，Scala数组与Java数组一一对应。例如，Scala的`Array[Int]`表示为Java的`int[]`，`Array[String]`表示为Java的`String[]`。但同时，Scala数组所提供的功能远比Java更多。首先，Scala数组是**泛型的**。也就是说，可以创建`Array[T]`，其中`T`是类型参数（在Java中不能创建`T[]`）。其次，Scala数组与序列兼容，可以将`Array[T]`赋值给`Seq[T]`变量。最后，Scala数组还支持所有序列操作。下面是一个示例：

```scala
val a1 = Array(1, 2, 3)
val a2 = a1.map(_ * 3)  // Array(3, 6, 9)
val a3 = a2.filter(_ % 2 != 0)  // Array(3, 9)
val a4 = a3.reverse  // Array(9, 3)
```

`Array`并不是`Seq`的子类型，而是有一个从`Array`到`scala.collection.mutable.ArraySeq`的隐式转换（参见3.3.11节），而后者是`Seq`的子类。可以使用`toArray`将序列转换回数组。例如：

```scala
val s: collection.Seq[Int] = a1  // ArraySeq(1, 2, 3)
val a5: Array[Int] = s.toArray  // Array(1, 2, 3)
a1 eq a5  // false
```

最后一行说明将数组包装为序列再转换回数组会生成原始数组的副本。

还有一个从`Array`到`scala.collection.ArrayOps`的隐式转换。该转换只将序列方法“添加”到数组中，但不会将数组转换为序列（`ArrayOps`也不是`Seq`的子类）。例如：

```scala
val seq: collection.Seq[Int] = a1  // ArraySeq(1, 2, 3)
val a2 = seq.reverse  // ArraySeq(3, 2, 1)

val ops: collection.ArrayOps[Int] = a1
val a3 = ops.reverse  // Array(3, 2, 1)
```

在`ArraySeq`上调用`reverse`会得到一个`ArraySeq`，而在`ArrayOps`上调用`reverse`会得到一个`Array`。

通常，你永远不会定义`ArrayOps`类型的值，而是直接在数组上调用序列方法：

```scala
val a2 = a1.reverse  // Array(3, 2, 1)
```

这会自动插入隐式转换`intArrayOps(a1)`，创建一个`ArrayOps[Int]`对象。

这两种隐式转换都能将数组转换为支持`reverse`方法的类型，那么在上面这行代码中，编译器为什么选择转换为`ArrayOps`而不是`ArraySeq`呢？这是因为`ArrayOps`转换的优先级高于`ArraySeq`转换。前者定义在`Predef`中，而后者定义在`LowPriorityImplicits`（`Predef`的超类）中。**子类中的隐式转换优先于超类。** 因此，如果两种转换都适用，则选择`Predef`中的转换。字符串也有类似的机制。

```scala
object Predef extends LowPriorityImplicits {
  // ...
  @inline implicit def intArrayOps(xs: Array[Int]): ArrayOps[Int] = new ArrayOps(xs)
}

abstract class LowPriorityImplicits {
  // ...
  implicit def wrapIntArray(xs: Array[Int]): ArraySeq.ofInt = if (xs ne null) new ArraySeq.ofInt(xs) else null
}
```

#### 泛型数组
在Java中不能创建泛型数组`T[]`，其中`T`是类型参数（参见[《Java核心技术》笔记 卷I 第8章]({% post_url 2024-10-29-java-note-v1ch08-generic-programming %}) 8.6.6节）。那么Scala的`Array[T]`是如何表示的呢？事实上，泛型数组`Array[T]`在运行时可能是Java的基本类型数组（如`int[]`），也可能是对象数组（如`String[]`）。例如，考虑下面的泛型方法`count()`：

```scala
def count[T](arr: Array[T], value: T): Int = {
  var n = 0
  for (x <- arr) {
    if (x == value) n += 1
  }
  n
}

println(count(Array(1, 2, 1, 4, 3, 2, 1), 1))  // T = Int
println(count(Array("a", "c", "a", "d", "c", "b"), "c"))  // T = String
```

`int[]`和`String[]`的唯一公共类型是`java.lang.Object`，因此Scala编译器会将参数`arr`的类型映射为`Object`。另外，在访问泛型数组的元素时，有一系列类型测试来确定实际的数组类型，这在一定程度上会降低效率。Scala编译器会将泛型方法`count()`翻译为与如下Java代码等价的字节码：

```java
public <T> int count(Object arr, T value) {
    IntRef n = IntRef.create(0);
    genericArrayOps(arr).foreach(x -> {
        if (BoxesRunTime.equals(x, value)) {
            ++n.elem;
        }
    });
    return n.elem;
}
```

可以看到，`Array[T]`被使用`genericArrayOps()`包装为`ArrayOps`，而`x == value`被翻译为`BoxesRunTime.equals()`（其中包含大量的`instanceof`检查）。

作为对比，下面的`countInt()`方法使用具体数组类型`Array[Int]`：

```scala
def countInt(arr: Array[Int], value: Int): Int = {
  var n = 0
  for (x <- arr) {
    if (x == value) n += 1
  }
  n
}
```

等价的Java代码如下：

```java
public int countInt(int[] arr, int value) {
    IntRef n = IntRef.create(0);
    intArrayOps(arr).foreach(x -> {
        if (x == value) {
            ++n.elem;
        }
    });
    return n.elem;
}
```

与使用泛型数组的方法相比，参数`arr`的类型被直接映射为`int[]`，而`x == value`被翻译为Java的`==`运算符，没有任何额外开销。

访问泛型数组比具体类型数组慢3~4倍。因此，如果需要最佳性能，应该优先使用具体类型数组而不是泛型数组。

创建泛型数组是一个更难的问题。考虑下面的例子：

```scala
// this is wrong!
def evenElems[T](xs: Vector[T]): Array[T] = {
  val arr = new Array[T]((xs.length + 1) / 2)
  for (i <- 0 until xs.length by 2)
    arr(i / 2) = xs(i)
  arr
}
```

`evenElems()`方法返回向量`xs`位于偶数位置的元素构成的新数组。由于类型参数`T`对应的实际类型在运行时被擦除，编译上面的代码会得到以下错误消息：

```
error: cannot find class tag for element type T
    val arr = new Array[T]((xs.length + 1) / 2)
              ^
```

你需要提供一些提示来帮助编译器了解`T`的实际类型是什么。这个提示是`scala.reflect.ClassTag`类型的对象，称为**类清单**(class manifest)。类清单是一种类型描述符对象，它描述了类型的顶级类是什么。

需要给`evenElems()`方法添加一个类清单作为隐式参数：

```scala
def evenElems[T](xs: Vector[T])(implicit m: ClassTag[T]): Array[T] = ...
```

也可以使用较短的**上下文边界**(context bound)语法`T: ClassTag`，如下所示：

```scala
def evenElems[T: ClassTag](xs: Vector[T]): Array[T] = ...
```

这两种形式的含义完全相同，意味着要求存在`ClassTag[T]`类型的隐式值。如果找到了，则使用它的`newArray()`方法构造正确类型的数组；否则报告上面的错误消息。

`ClassTag`的伴生对象中定义了所有基本类型的类清单，一般的对象类型则使用`GenericClassTag`。例如：

```scala
scala> evenElems(Vector(1, 2, 3, 4, 5))  // scala.reflect.ManifestFactory$IntManifest
val res0: Array[Int] = Array(1, 3, 5)

scala> evenElems(Vector("this", "is", "a", "test", "run"))  // scala.reflect.ClassTag$GenericClassTag
val res1: Array[String] = Array(this, a, run)
```

但是，如果类型参数本身是另一个没有类清单的类型参数，编译器仍然会报错：

```scala
def wrap[U](xs: Vector[U]) = evenElems(xs)
// error: No ClassTag available for U
```

解决方法是要求存在`U`的类清单：

```scala
def wrap[U: ClassTag](xs: Vector[U]) = evenElems(xs)
```

总之，创建泛型数组需要类清单。每当需要创建类型参数`T`的数组时，还需要提供`T`的隐式类清单。

### 6.1.7 字符串
<https://docs.scala-lang.org/overviews/collections-2.13/strings.html>

与数组一样，字符串本身不是序列，但可以被隐式转换为序列，并且支持所有序列操作。例如：

```scala
val str = "hello"
str.reverse  // "olleh"
str.map(_.toUpper)  // "HELLO"
str.drop(3)  // "lo"
str.slice(1, 4)  // "ell"
val s: Seq[Char] = str  // WrappedString
```

有两个隐式转换：
* 一个将`String`转换为`StringOps`，将不可变序列的所有方法添加到字符串中，优先级较高。上面的前四个方法调用中自动插入了该转换。
* 另一个将`String`转换为`WrappedString`（`IndexedSeq[Char]`的子类），优先级较低。上面最后一行应用了该转换。

## 6.2 可变和不可变集合
<https://docs.scala-lang.org/overviews/collections-2.13/overview.html>

Scala系统地区分了可变和不可变集合。**可变**(mutable)集合可以添加、删除或更新元素。**不可变**(immutable)集合在创建后永远不会改变，其添加、删除或更新操作会返回一个新的集合，旧集合保持不变。

大多数集合类都有三种变体：
* `scala.collection.immutable`包中的集合是不可变的。
* `scala.collection.mutable`包中的集合是可变的。
* `scala.collection`包中的集合可能是可变的或不可变的，是另外两个的超类，称为根集合。

例如，`scala.collection.Seq[T]`是`scala.collection.immutable.Seq[T]`和`scala.collection.mutable.Seq[T]`的超类。

一般来说，`scala.collection`包中的根集合提供转换操作，`scala.collection.immutable`包中的不可变集合通常增加用于添加或删除单个值的操作，`scala.collection.mutable`包中的可变集合通常增加有副作用的修改操作。

根集合和不可变集合的另一个区别是：不可变集合一定不会被修改，而根集合的运行时类型仍然可能是可变集合，可能在其他地方被修改。例如：

```scala
val s1: scala.collection.immutable.Seq[Int] = ...  // guaranteed to be immutable
val s2: scala.collection.mutable.Seq[Int] = ...
val s: scala.collection.Seq[Int] = s2  // can be mutated through s2
```

默认情况下，Scala总是选择不可变集合。例如，如果直接使用没有任何前缀、也没有从其他地方导入的`Set`，将会得到不可变集。因为这是[scala.Predef](https://github.com/scala/scala/blob/v2.13.18/src/library/scala/Predef.scala)类中定义的别名：

```scala
type Set[A] = scala.collection.immutable.Set[A]
val Set = scala.collection.immutable.Set
```

要获得可变集，需要显式地写`collection.mutable.Set`（`scala`包是默认导入的）。

如果想同时使用集合的可变和不可变版本，一种约定是只导入`collection.mutable`包。这样没有前缀的`Set`仍然是不可变集，`mutable.Set`是可变集：

```scala
import scala.collection.mutable

val s1 = Set(1, 2, 3)  // immutable
val s2 = mutable.Set(1, 2, 3) // mutable
```

为了方便，一些重要类型在`scala`包中定义了别名，因此可以直接使用而无需导入。例如，`List`类型可以通过以下三种方式访问：
* `scala.collection.immutable.List`
* `scala.List`
* `List`

注：这些别名定义在[scala/package.scala](https://github.com/scala/scala/blob/v2.13.18/src/library/scala/package.scala)中：

```scala
type List[+A] = scala.collection.immutable.List[A]
val List = scala.collection.immutable.List
```

其他类型别名包括`Iterable`, `Iterator`, `Seq`, `IndexedSeq`, `LazyList`, `Vector`, `StringBuilder`和`Range`。

### 6.2.1 集合类层次结构
下图显示了`scala.collection`包中的所有集合。这些都是抽象类或特质，通常有可变和不可变的实现。

![集合类图](/assets/images/scala-tutorial-6-collections/collections-diagram.svg)

下图显示了`scala.collection.immutable`包中的所有集合。

![不可变集合类图](/assets/images/scala-tutorial-6-collections/collections-immutable-diagram.svg)

下图显示了`scala.collection.mutable`包中的所有集合。

![可变集合类图](/assets/images/scala-tutorial-6-collections/collections-mutable-diagram.svg)

图例：

![图例](/assets/images/scala-tutorial-6-collections/collections-legend-diagram.svg)

集合类层次结构中的大多数类都有三种变体：根、可变和不可变。唯一的例外是`Buffer`，它只有可变集合。

所有集合都可以通过统一的语法创建——类名后跟其元素。特质和具体实现类都适用。

```scala
Iterable("x", "y", "z")
Seq(1, 2, 3)
List(4, 5, 6)
Vector(7, 8, 9)
Buffer(x, y, z)
IndexedSeq(1.0, 2.0)
LinearSeq(a, b, c)
Set(Color.red, Color.green, Color.blue)
SortedSet("hello", "world")
Map("a" -> 1, "b" -> 2, "c" -> 3)
HashMap("x" -> 24, "y" -> 25, "z" -> 26)
```

所有集合都实现了`toString()`方法，按照与上面相同的方式显示。

所有集合都支持`Iterable`提供的API，但返回类型可能更具体。例如，`Iterable.map()`方法返回`Iterable`，但`List.map()`返回`List`，`Set.map()`返回`Set`，等等。这种行为称为**统一返回类型原则**(uniform return type principle)。

```scala
scala> List(1, 2, 3).map(_ + 1)
val res0: List[Int] = List(2, 3, 4)

scala> Set(1, 2, 3).map(_ * 2)
val res1: scala.collection.immutable.Set[Int] = Set(2, 4, 6)
```

## 6.3 匿名函数
<https://docs.scala-lang.org/overviews/scala-book/anonymous-functions.html>

本节将介绍函数式编程的一个特性，称为**匿名函数**(anonymous functions)（类似于Java的Lambda表达式）。

### 6.3.1 单参数函数
给定一个列表：

```scala
val nums = List(1, 2, 3)
```

`List[A]`的`map()`方法接受一个`A => B`类型的函数`f`，将每个元素`x`映射为`f(x)`，返回一个新列表`List[B]`。简化的声明如下：

```scala
def map[B](f: A => B): List[B]
```

可以通过`map()`方法将`nums`中的每个元素加倍来创建一个新列表，如下所示：

```scala
val doubledNums = nums.map(_ * 2)  // List(2, 4, 6)
```

在这个示例中，`_ * 2`是一个匿名函数（类型为`Int => Int`），表示“将每个元素乘以2”。

如果愿意，也可以用`=>`语法编写匿名函数。上面的代码等价于

```scala
val doubledNums = nums.map(i => i * 2)
val doubledNums = nums.map((i: Int) => i * 2)
```

这三种写法的含义相同。

这个`map()`示例等价于以下Java代码：

```java
List<Integer> nums = new ArrayList<>(Arrays.asList(1, 2, 3));

// the `map` process
List<Integer> doubledNums = nums.stream()
    .map(i -> i * 2)
    .collect(Collectors.toList());
```

还等价于Scala的`for`表达式：

```scala
val doubledNums = for (i <- nums) yield i * 2
```

使用匿名函数的另一个例子是`List`类的`filter()`方法。该方法接受一个`A => Boolean`类型的谓词`p`，返回一个新列表`List[A]`，其中仅包含使`p(x)`为`true`的元素`x`。声明如下：

```scala
def filter(p: A => Boolean): List[A]
```

给定以下列表：

```scala
val nums = List.range(1, 10)  // List(1, 2, 3, 4, 5, 6, 7, 8, 9)
```

可以像这样创建一个包含`nums`中所有大于5的整数的新列表：

```scala
val a = nums.filter(_ > 5)  // List(6, 7, 8, 9)
```

这行代码等价于

```scala
def lessThanFive(i: Int): Boolean = i < 5
val a = nums.filter(lessThanFive)
```

像这样创建只包含偶数的列表：

```scala
val b = nums.filter(_ % 2 == 0)  // List(2, 4, 6, 8)
```

### 6.3.2 多参数函数
`List[A]`的`reduce()`方法接受一个`(A, A) => A`类型的二元运算符`op`，将`op`依次应用于每个元素，返回最终结果<code>op(...op(op(a<sub>1</sub>, a<sub>2</sub>), a<sub>3</sub>)..., a<sub>n</sub>)</code>。简化的声明如下：

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

### 6.3.3 高阶函数
<https://docs.scala-lang.org/tour/higher-order-functions.html>

**高阶函数**(higher order function)是接受函数作为参数或者返回一个函数的函数或方法。

最常见的例子之一是Scala集合的`map()`方法。

```scala
val salaries = Seq(20000, 70000, 40000)
val doubleSalary = (x: Int) => x * 2
val newSalaries = salaries.map(doubleSalary) // List(40000, 140000, 80000)
```

也可以将方法作为参数传递给高阶函数。

```scala
case class WeeklyWeatherForecast(temperatures: Seq[Double]) {
  private def convertCtoF(temp: Double) = temp * 1.8 + 32

  def forecastInFahrenheit: Seq[Double] = temperatures.map(convertCtoF)
}
```

其中，方法`convertCtoF`被传递给高阶函数`map()`。这是因为编译器会将方法`convertCtoF`转换为函数`x => convertCtoF(x)`。

#### 接受函数参数的函数
使用高阶函数的一个原因是减少冗余代码。例如：

```scala
object SalaryRaiser {
  def smallPromotion(salaries: List[Double]): List[Double] =
    salaries.map(salary => salary * 1.1)

  def greatPromotion(salaries: List[Double]): List[Double] =
    salaries.map(salary => salary * math.log(salary))

  def hugePromotion(salaries: List[Double]): List[Double] =
    salaries.map(salary => salary * salary)
}
```

这三个方法的唯一区别是乘法系数。为了简化，可以将重复代码提取为高阶函数，如下所示：

```scala
object SalaryRaiser {
  private def promotion(salaries: List[Double], promotionFunction: Double => Double): List[Double] =
    salaries.map(promotionFunction)

  def smallPromotion(salaries: List[Double]): List[Double] =
    promotion(salaries, salary => salary * 1.1)

  def greatPromotion(salaries: List[Double]): List[Double] =
    promotion(salaries, salary => salary * math.log(salary))

  def hugePromotion(salaries: List[Double]): List[Double] =
    promotion(salaries, salary => salary * salary)
}
```

新方法`promotion()`接受一个列表和一个`Double => Double`类型的函数。

#### 返回函数的函数
在某些情况下，需要生成一个函数。例如：

```scala
def urlBuilder(ssl: Boolean, domainName: String): (String, String) => String = {
  val schema = if (ssl) "https://" else "http://"
  (endpoint, query) => s"$schema$domainName/$endpoint?$query"
}

val domainName = "www.example.com"
def getURL = urlBuilder(true, domainName)
val endpoint = "users"
val query = "id=1"
val url = getURL(endpoint, query) // "https://www.example.com/users?id=1"
```

`urlBuilder()`的返回类型是`(String, String) => String`。在上面的例子中，返回的匿名函数是`(endpoint: String, query: String) => s"https://www.example.com/$endpoint?$query"`。

### 6.3.4 Java函数式接口
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
* [Scala语言规范 - SAM conversion](https://www.scala-lang.org/files/archive/spec/2.13/06-expressions.html#sam-conversion)

## 6.4 常用集合方法
Scala集合类的一大优势是它内置了大量有用的方法。本节展示一些最常用的序列方法和映射方法。

### 6.4.1 常用序列方法
<https://docs.scala-lang.org/overviews/scala-book/collections-methods.html>

本节将展示一些最常用的序列方法。这些方法适用于所有序列类，包括`ArrayBuffer`、`List`、`Vector`、`Array`等。另外，这些方法都不会修改当前集合，而是返回一个新集合。

以下面的列表为例：

```scala
val a = List(1, 2, 3, 4, 5)
```

| 方法 | 描述 | 示例 |
| --- | --- | --- |
| `map(f)` | 将每个元素`x`映射为`f(x)` | `a.map(_ * 2)`返回`List(2, 4, 6, 8, 10)` |
| `filter(p)` | 只保留满足`p`的元素 | `a.filter(_ > 3)`返回`List(4, 5)` |
| `foreach(f)` | 将`f`应用于每个元素 | `a.foreach(println)`打印所有元素 |
 | `head`, `tail` | 第一个元素、剩余的元素 | `a.head`返回`1`<br>`a.tail`返回`List(2, 3, 4, 5)` |
| `init`, `last` | 之前的元素、最后一个元素 | `a.init`返回`List(1, 2, 3, 4)`<br>`a.last`返回`5` |
| `take(n)` | 选择前`n`个元素 | `a.take(3)`返回`List(1, 2, 3)` |
| `drop(n)` | 丢弃前`n`个元素 | `a.drop(3)`返回`List(4, 5)` |
| `takeWhile(p)` | 选择满足`p`的最长前缀 | `a.takeWhile(_ < 3)`返回`List(1, 2)` |
| `dropWhile(p)` | 丢弃满足`p`的最长前缀 | `a.dropWhile(_ < 3)`返回`List(3, 4, 5)` |
| `reduce(op)`,<br>`reduceLeft(op)`,<br>`reduceRight(op)` | 将`op`依次应用于每个元素<br>从左到右：<code>op(...op(op(a<sub>1</sub>, a<sub>2</sub>), a<sub>3</sub>)..., a<sub>n</sub>)</code><br>从右到左：<code>op(a<sub>1</sub>, op(a<sub>2</sub>, ...op(a<sub>n-1</sub>, a<sub>n</sub>)...))</code> | `a.reduce(_ + _)`返回`15` |
| `fold(z)(op)`,<br>`foldLeft(z)(op)`,<br>`foldRight(z)(op)` | 类似于`reduce`，但从零元素`z`开始<br>从左到右：<code>op(...op(op(z, a<sub>1</sub>), a<sub>2</sub>)..., a<sub>n</sub>)</code><br>从右到左：<code>op(a<sub>1</sub>, op(a<sub>2</sub>, ...op(a<sub>n</sub>, z)...))</code> | `a.fold(0)(_ + _)`返回`15` |
| `scan(z)(op)`,<br>`scanLeft(z)(op)`,<br>`scanRight(z)(op)` | 返回前缀序列<br>从左到右：<code>z, op(z, a<sub>1</sub>), op(op(z, a<sub>1</sub>), a<sub>2</sub>), ...</code><br>从右到左：<code>..., op(a<sub>n-1</sub>, op(a<sub>n</sub>, z)), op(a<sub>n</sub>, z), z</code> | `a.scanLeft(0)(_ + _)`返回`List(0, 1, 3, 6, 10, 15)`<br>`a.scanRight(0)(_ + _)`返回`List(15, 14, 12, 9, 5, 0)` |

完整列表参见[Seq类](https://docs.scala-lang.org/overviews/collections-2.13/seqs.html)和[API文档](https://www.scala-lang.org/api/2.13.18/scala/collection/Seq.html)。

### 6.4.2 常用映射方法
<https://docs.scala-lang.org/overviews/scala-book/collections-maps.html>

本节将展示一些最常用的映射方法。

以下面的映射为例：

```scala
val m = Map(1 -> "a", 2 -> "b", 3 -> "c", 4 -> "d")
```

| 方法 | 描述 | 示例 |
| --- | --- | --- |
| `keys` | 键集合 | `m.keys`返回`Iterable(1, 2, 3, 4)` |
| `values` | 值集合 | `m.values`返回`Iterable(a, b, c, d)` |
| `contains(k)` | 测试映射是否包含键`k` | `m.contains(3)`返回`true` |
| `transform(f)` | 将每个键值对`(k, v)`的值映射为`f(k, v)` | `m.transform((k, v) => v.toUpperCase)`返回<br>`Map(1 -> A, 2 -> B, 3 -> C, 4 -> D)` |

完整列表参见[Map类](https://docs.scala-lang.org/overviews/collections-2.13/maps.html)和[API文档](https://www.scala-lang.org/api/2.13.18/scala/collection/Map.html)。

### 6.4.3 工厂方法
<https://docs.scala-lang.org/overviews/collections-2.13/creating-collections-from-scratch.html>

下表总结了集合伴生对象提供的工厂方法：

| 方法 | 描述 |
| --- | --- |
| `C.empty` | 空集合 |
| `C(x, y, ...)` | 由元素`x, y, ...`构成的集合 |
| `C.concat(xs, ys, ...)` | 拼接`xs, ys, ...`的所有元素得到的集合 |
| `C.fill(n)(e)` | 长度为`n`、元素全部为`e`的集合（`e`是传名参数） |
| `C.fill(m, n)(e)` | 大小为`m × n`、元素全部为`e`的集合 |
| `C.tabulate(n)(f)` | 长度为`n`、索引`i`处的元素为`f(i)`的集合 |
| `C.tabulate(m, n)(f)` | 大小为`m × n`、索引`(i, j)`处的元素为`f(i, j)`的集合 |
| `C.range(start, end)` | 整数集合`start, start+1, ..., end-1` |
| `C.range(start, end, step)` | 从`start`到`end`（不包含）、步长为`step`的整数集合 |
| `C.iterate(x, n)(f)` | 长度为`n`的集合，元素为`x, f(x), f(f(x)), ...` |

完整列表参见[API文档](https://www.scala-lang.org/api/2.13.18/scala/collection/Seq$.html)。

## 6.5 与Java集合相互转换
<https://docs.scala-lang.org/overviews/collections-2.13/conversions-between-java-and-scala-collections.html>

与Scala一样，Java也有丰富的集合库。两者有很多相似之处，有时可能需要相互转换。例如，对现有的Java集合调用Scala集合方法，或者将Scala集合传递给接受Java集合的Java方法。这很容易做到，[JavaConverters](https://www.scala-lang.org/api/2.12.21/scala/collection/JavaConverters$.html)对象提供了主要集合类型之间的相互转换。

注：从Scala 2.13起应该使用`scala.jdk.CollectionConverters`。

通过`asScala`和`asJava`方法支持以下双向转换：

```
scala.collection.Iterable       <=> java.lang.Iterable
scala.collection.Iterator       <=> java.util.Iterator
scala.collection.mutable.Buffer <=> java.util.List
scala.collection.mutable.Set    <=> java.util.Set
scala.collection.mutable.Map    <=> java.util.Map
scala.collection.concurrent.Map <=> java.util.concurrent.ConcurrentMap
```

通过`asJava`方法支持以下单向转换：

```
scala.collection.Seq         => java.util.List
scala.collection.mutable.Seq => java.util.List
scala.collection.Set         => java.util.Set
scala.collection.Map         => java.util.Map
```

要使用这些转换，只需从`JavaConverters`对象导入即可：

```scala
scala> import scala.collection.JavaConverters._
import scala.collection.JavaConverters._

scala> import scala.collection.mutable
import scala.collection.mutable

scala> val jul: java.util.List[Int] = mutable.ArrayBuffer(1, 2, 3).asJava
val jul: java.util.List[Int] = [1, 2, 3]

scala> val buf: mutable.Seq[Int] = jul.asScala
val buf: scala.collection.mutable.Seq[Int] = ArrayBuffer(1, 2, 3)

scala> val jum: java.util.Map[String, Int] = mutable.HashMap("abc" -> 1, "hello" -> 2).asJava
val jum: java.util.Map[String,Int] = {abc=1, hello=2}
```

在内部，这些转换会创建一个“包装器”对象，将所有操作转发给底层集合对象。因此在转换时，集合不会被拷贝。如果转换后再转换回原始类型，最终会得到与开始时同一个集合对象。

由于Java在类型上不区分可变和不可变集合，`scala.collection.immutable.List`会转换为一个`java.util.List`，但其中所有修改操作都抛出`UnsupportedOperationException`。例如：

```scala
val jul = List(1, 2, 3).asJava
jul.add(7)  // throws UnsupportedOperationException
```

## 6.6 视图
<https://docs.scala-lang.org/overviews/collections-2.13/views.html>

集合有很多构造新集合的方法，例如`map()`、`filter()`和`++`。这些方法称为**变换**(transformer)。

实现变换操作主要有两种方式。一种是**严格的**(strict)，即立刻构造一个新集合作为结果；另一种是**惰性的**(lazy)，即构造结果集合的一个代理，其元素仅在需要时计算。除了`LazyList`，Scala集合的所有转换操作默认都是严格的。

考虑下面的例子，假设对一个整数向量连续做两次映射：

```scala
val v = Vector(1, 2, 3, 4, 5)
val w = v.map(_ + 1).map(_ * 2)  // Vector(4, 6, 8, 10, 12)
```

表达式`v.map(_ + 1)`构造了一个新向量，然后通过调用`map(_ * 2)`将其转换为第三个向量。在许多情况下，构造中间结果是浪费的。在上面的例子中，使用两个函数的组合`(_ + 1) * 2`进行单次映射会更快。但通常，连续的变换是在程序的不同模块中完成的，组合这些变换会破坏模块化。

避免中间结果的一种通用方式是：首先将集合转换为视图，然后对视图应用所有变换，最后将视图转换回集合。**视图**(view)是一种特殊的集合，是表示底层集合的轻量级对象，所有变换操作都是惰性的。

要从集合得到其视图，调用`view`方法（如`c.view`）。要从视图转换回集合，调用`toXxx`或`to(Xxx)`方法（如`v.toList`或`v.to(List)`）。

前面的例子可以像这样使用视图：

```scala
val w = v.view.map(_ + 1).map(_ * 2).to(Vector)  // or .toVector
```

下面逐步查看每个操作：

```scala
scala> val vv = v.view
val vv: scala.collection.IndexedSeqView[Int] = IndexedSeqView(<not computed>)

scala> vv.map(_ + 1)
val res0: scala.collection.IndexedSeqView[Int] = IndexedSeqView(<not computed>)

scala> res0.map(_ * 2)
val res1: scala.collection.IndexedSeqView[Int] = IndexedSeqView(<not computed>)

scala> res1.to(Vector)
val res2: scala.collection.immutable.Vector[Int] = Vector(4, 6, 8, 10, 12)
```

* `v.view`生成一个视图`IndexedSeqView[Int]`，即惰性求值的`IndexedSeq[Int]`。视图的`toString()`方法不会计算元素，因此其内容显示为`<not computed>`。
* `vv.map(_ + 1)`的结果是另一个视图。这本质上是一个包装器，记录了需要对向量`v`应用`map(_ + 1)`操作，但不会真正计算。
* `res0.map(_ * 2)`同上，仅记录操作。
* 最后调用`to(Vector)`强制计算结果，这样就不需要中间结果。

下面是另一个例子。假设有一个很长的单词序列，你想从前100万个单词中找到第一个回文。

```scala
def isPalindrome(x: String) = x == x.reverse
def findPalindrome(s: Seq[String]): Option[String] = s.find(isPalindrome)

val firstPalindrome = findPalindrome(words.take(1000000))  // bad
val firstPalindrome = findPalindrome(words.view.take(1000000))  // good
```

`words.take(1000000)`总是会构造包含100万个单词的中间序列，而`words.view.take(1000000)`只会构造一个轻量级的视图对象。

一般来说，视图具有以下属性：
1. 转换操作的复杂度为`O(1)`。
2. 元素访问操作的复杂度与底层数据结构相同（例如，`IndexedSeqView`的索引访问是`O(1)`的）。

不过也有一些例外。例如，`sorted`操作无法同时满足这两个属性。实际上，视图的`sorted`操作不是惰性的，而是始终强制计算，因此违反了属性1，但仍满足属性2。

既然视图这么好用，为什么还要有严格的集合？一个原因是惰性集合的性能并不总是更好。对于较小的集合，在视图中构造和应用闭包的额外开销通常大于避免中间结果的收益。

另一个可能更重要的原因是，如果操作有副作用，惰性求值可能会令人困惑。例如，在Scala 2.8以前，`Range`类型是惰性的。用户期望下面的代码会创建10个actor：

```scala
val actors = for (i <- 1 to 10) yield actor { ... }
```

然而实际上没有创建任何actor。这是因为在早期版本的Scala中，`1 to 10`产生的对象行为类似于视图。上面的`for`表达式等价于`(1 to 10).map(i => actor { ... })`，其结果也是一个视图，因此不会计算任何元素。为了避免此类问题，当前的Scala集合库要求除`LazyList`和视图外，所有集合都是严格的。

总之，视图是调和效率和模块化的强大工具。不过，应该将视图限制在纯函数式代码中。最好避免将视图与创建新集合同时有副作用的操作混合。

## 6.7 并行集合
<https://docs.scala-lang.org/overviews/parallel-collections/overview.html>

Scala标准库包含了**并行集合**(parallel collection)，旨在提供可靠并行编程的高层抽象，同时使用户免受底层细节的困扰。

考虑下面的例子，在大型集合上执行`map()`操作：

```scala
val a = (1 to 1000000).toList
val b = a.map(_ + 42)
```

要并行执行相同的操作，只需在串行集合上调用`par`方法，之后就可以像串行集合一样使用并行集合。例如：

```scala
val b = a.par.map(_ + 42)
```

Scala并行集合库为许多重要的集合类提供了对应的并行版本（在`scala.collection.parallel`包中），包括`ParArray`, `ParSeq`, `ParVector`, `ParMap`, `ParSet`, `ParRange`, `ParTrieMap`等

注意：如果使用Scala 2.13+，使用并行集合必须依赖一个单独的模块[scala-parallel-collections](https://github.com/scala/scala-parallel-collections)（[API文档](https://javadoc.io/doc/org.scala-lang.modules/scala-parallel-collections_2.13/latest/index.html)）：

```xml
<dependency>
    <groupId>org.scala-lang.modules</groupId>
    <artifactId>scala-parallel-collections_2.13</artifactId>
    <version>1.2.0</version>
</dependency>
```

并在代码中添加导入：

```scala
import scala.collection.parallel.CollectionConverters._
```

### 6.7.1 创建并行集合
有两种创建并行集合的方式。第一种是使用`new`关键字：

```scala
import scala.collection.parallel.immutable.ParVector
val pv = new ParVector[Int]
```

第二种是使用串行集合的`par`方法：

```scala
val pv = Vector(1, 2, 3, 4, 5).par
```

也可以通过调用并行集合的`seq`方法转换为串行集合。

### 6.7.2 并行操作示例
下面是一些简单的用法示例。注意：这些示例都是对小集合进行操作，仅作为演示。只有当集合很大时，速度提升才会显著。

```scala
val lastNames = List("Smith", "Jones", "Frankenstein", "Bach", "Jackson", "Rodin").par
lastNames.map(_.toUpperCase)  // ParVector(SMITH, JONES, FRANKENSTEIN, BACH, JACKSON, RODIN)
lastNames.filter(_.head >= 'J')  // ParVector(Smith, Jones, Jackson, Rodin)

val parArray = (1 to 10000).toArray.par
parArray.fold(0)(_ + _)  // 50005000
```

### 6.7.3 语义
虽然并行集合的用法看起来与串行集合非常相似，但要注意它的语义不同，特别是对于副作用和非结合性操作。

从概念上讲，Scala并行集合通过递归地“分割”(split)给定的集合，并行地对每个分区应用操作，最后重新“组合”(combine)所有结果来完成并行化操作。

并行集合的并发和“无序”(out-of-order)语义意味着：
* 副作用操作可能导致不确定性。
* 非结合性操作会导致不确定性。

（1）副作用操作

由于并行集合的并发执行语义，为了保持确定性，应该避免在集合上执行会产生副作用的操作。一个简单的例子是在`foreach()`方法中修改外部声明的变量。

```scala
val list = (1 to 1000).toList.par
var sum = 0
list.foreach(sum += _)
println(sum)  // 467766

sum = 0
list.foreach(sum += _)
println(sum)  // 457073

sum = 0
list.foreach(sum += _)
println(sum)  // 468520
```

多次执行会得到不同的`sum`值。这种不确定性的来源是**数据竞争**——对同一变量的并发读写。

（2）非结合性操作

由于并行集合的“无序”语义，只能执行结合性(associative)操作（即`(a op b) op c) == a op (b op c)`），以避免不确定性。例如，减法是一种非结合性操作：

```scala
val list = (1 to 1000).toList.par
list.reduce(_ - _)  // -228888
list.reduce(_ - _)  // -61000
list.reduce(_ - _)  // -331818
```

注意：非交换性操作并不会导致不确定性。一个简单的例子是字符串拼接——一种结合性但非交换性操作：

```scala
val strings = List("abc", "def", "ghi", "jk", "lmnop", "qrs", "tuv", "wx", "yz").par
val alphabet = strings.reduce(_ ++ _)  // abcdefghijklmnopqrstuvwxyz
```

并行集合的“无序”语义只意味着操作的执行（在时间意义上）无序，并不意味着结果的组合（在空间意义上）无序。结果会按划分分区的顺序重新组合。
