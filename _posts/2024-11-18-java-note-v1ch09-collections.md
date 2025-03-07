---
title: 《Java核心技术》笔记 卷I 第9章 集合
date: 2024-11-18 22:08:07 +0800
categories: [Java, Core Java]
tags: [java, collection, iterator, linked list, array list, hash table, set, hash set, tree set, queue, priority queue, map, hash map, tree map, algorithm, sort, binary search, configuration]
math: true
---
本章将介绍如何利用Java标准库帮助我们实现程序设计所需的传统**数据结构**(data structure)。

## 9.1 Java集合框架
这一节将介绍**Java集合框架**(Java collections framework)的基本设计，展示如何使用，并解释一些颇具争议的特性背后的考虑。

注：
* **集合**(collection)即一组元素，相当于C++中的容器。
* 官方文档：
  * [Collections Framework Documentation](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/doc-files/coll-index.html)
  * [Collections Framework Tutorial](https://docs.oracle.com/javase/tutorial/collections/index.html)

### 9.1.1 集合接口与实现分离
Java集合库将**接口**与**实现**分离。下面利用熟悉的数据结构——**队列**(queue)来说明这种分离。

**队列接口**指出可以在队尾添加元素，在队头删除元素，并且可以查出队列中的元素个数。当需要收集对象并按照“先进先出”(first in, first out)的方式获取对象时，就应该使用队列（见下图）。

![队列](/assets/images/java-note-v1ch09-collections/队列.png)

队列接口的最简形式如下：

```java
// a simplified form of the interface in the standard library
public interface Queue<E> {
    void add(E element);
    E remove();
    int size();
}
```

这个接口并没有说明队列是如何实现的。队列通常有两种实现方式：一种是使用循环数组，另一种是使用链表（见下图）。

![队列的实现](/assets/images/java-note-v1ch09-collections/队列的实现.png)

每种实现都可以用一个实现了`Queue`接口的类表示。Java标准库中的`ArrayDeque`和`LinkedList`类分别实现了循环数组队列和链表队列，简化的定义如下所示：

```java
public class ArrayDeque<E> implements Queue<E> {
    private int head;
    private int tail;
    private E[] elements;

    ArrayDeque(int capacity) { ... }
    public void add(E element) { ... }
    public E remove() { ... }
    public int size() { ... }
}

public class LinkedList<E> implements Queue<E> {
    private Link head;
    private Link tail;

    LinkedList() { ... }
    public void add(E element) { ... }
    public E remove() { ... }
    public int size() { ... }
}
```

一旦已经构造了集合对象，就不需要知道究竟使用了哪种实现。因此，只有在构造集合对象时，使用具体的类才有意义；而使用接口类型存放集合引用。

```java
Queue<Customer> expressLane = new ArrayDeque<>(100);
expressLane.add(new Customer("Harry"));
```

采用这种方法，可以很容易地使用另一种不同的实现。只需修改程序中的一个地方——构造器调用。如果觉得链表是更好的选择，就将代码修改为：

```java
Queue<Customer> expressLane = new LinkedList<>();
expressLane.add(new Customer("Harry"));
```

在研究API文档时，会发现另一组名字以`Abstract`开头的类，例如`AbstractQueue`。这些类是为库实现者而设计的。如果想要实现自己的队列类，会发现扩展`AbstractQueue`类要比实现`Queue`接口中的所有方法更容易。

注：`AbstractQueue`扩展了`AbstractCollection`，为一些与实现无关的方法提供了默认实现。例如，`isEmpty()`返回`size() == 0`（另见6.1.5节）。而`size()`与具体实现有关，因此是抽象方法。

[circularArrayQueue/CircularArrayQueue.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch09/circularArrayQueue/CircularArrayQueue.java)

### 9.1.2 Collection接口
在Java标准库中，集合类的基本接口是`Collection`。这个接口有两个基本方法：

```java
public interface Collection<E> {
    boolean add(E element);
    Iterator<E> iterator();
    ...
}
```

`add()`方法用于向集合中添加元素。如果确实改变了集合则返回`true`，否则返回`false`。

`iterator()`方法用于返回一个迭代器（将在下一节讨论）。

### 9.1.3 迭代器
**迭代器**(iterator)用于依次访问集合中的元素。`Iterator`接口包含4个方法：

```java
public interface Iterator<E> {
    E next();
    boolean hasNext();
    void remove();
    default void forEachRemaining(Consumer<? super E> action);
}
```

`next()`方法返回集合中的下一个元素，如果到达了集合的末尾则抛出`NoSuchElementException`。因此，在调用`next()`之前需要调用`hasNext()`方法：如果迭代器还有更多可以访问的元素，这个方法就返回`true`。如果要访问集合中的所有元素，就获取一个迭代器，当`hasNext()`返回`true`时反复地调用`next()`方法。例如：

```java
Collection<String> c = ...;
Iterator<String> iter = c.iterator();
while (iter.hasNext()) {
    String element = iter.next();
    // do something with element
}
```

可以将这个循环写成更加简洁的for each循环：

```java
for (String element : c) {
    // do something with element
}
```

编译器直接把for each循环翻译为使用迭代器的循环。

for each循环可用于任何实现了`Iterable`接口的对象，这个接口只有一个抽象方法：

```java
public interface Iterable<E> {
    Iterator<E> iterator();
    ...
}
```

`Collection`接口扩展了`Iterable`接口。因此，对于标准库中的任何集合都可以使用for each循环（即都是“可迭代的”）。

也可以不写循环，而是调用`forEachRemaining()`方法并提供一个lambda表达式，将对迭代器的每个元素调用这个lambda表达式：

```java
iterator.forEachRemaining(element -> /* do something with element */);
```

访问元素的顺序取决于集合类型。`ArrayList`是按索引顺序访问，而`HashSet`是基本随机的顺序，但可以确保遍历到集合中的所有元素。

Java集合库中的迭代器与C++ STL中的迭代器在概念上有着重要的区别。在C++中，迭代器是按照数组索引建模的：如果有数组索引`i`，可以通过`a[i]`访问数组元素，也可以通过`i++`向前移动索引（注：详见[《C++程序设计原理与实践》笔记 第20章]({% post_url 2023-07-15-ppp-note-ch20-containers-and-iterators %}) 20.3节）。

但是，Java迭代器并不是这样工作的。访问元素与位置变化是紧密耦合的。访问元素的唯一方式是调用`next()`，在访问元素的同时向前移动迭代器。因此，应该将Java迭代器看作是**位于两个元素之间**。当调用`next()`时，迭代器就**越过**下一个元素，并返回刚刚越过的那个元素的引用（见下图）。

![向前移动迭代器](/assets/images/java-note-v1ch09-collections/向前移动迭代器.png)

注：
* 可以将迭代器想象为光标，见9.3.1节注释。
* Java中的`iter.next()`相当于C++中的`*iter++`，而没有与`*iter`或`++iter`等价的操作。

注释：还有一个有用的类比。可以认为`Iterator.next()`等价于`InputStream.read()`：从输入流读取一个字节会自动“消耗掉”这个字节。

`Iterator`接口的`remove()`方法删除上次调用`next()`方法返回的元素。例如，可以如下删除一个字符串集合中的第一个元素：

```java
Iterator<String> it = c.iterator();
it.next(); // skip over the first element
it.remove(); // now remove it
```

如果在调用`remove()`之前没有调用`next()`，将会抛出`IllegalStateException`。如果想删除两个相邻的元素，不能连续两次调用`remove()`，而是要依次调用`remove()`、`next()`和`remove()`。

例如，假设一个`ArrayList`初始为[1, 2, 3, 4, 5]，`it`是其迭代器：

```java
it.next();     // 1
it.next();     // 2
it.next();     // 3
it.remove();   // [1, 2, 4, 5]
it.next();     // 4
it.remove();   // [1, 2, 5]
it.next();     // 5
it.hasNext();  // false
```

注：很少会像这样通过迭代器手动访问和删除元素，更常用的做法是通过`removeIf()`等方法操作整个集合（见下一节）。

### 9.1.4 泛型实用方法
`Collection`和`Iterator`都是泛型接口，这意味着可以编写操作任何类型集合的实用方法(utility method)。例如，下面是一个检测任意集合是否包含指定元素的泛型方法：

```java
public static <E> boolean contains(Collection<E> c, Object obj) {
    for (E element : c)
        if (element.equals(obj))
            return true;
    return false;
}
```

`Collection`接口声明了很多有用的方法，这样库的使用者就不必重新造轮子了。下面列举了其中的一部分：

```java
int size()
boolean isEmpty()
boolean contains(Object obj)
boolean containsAll(Collection<?> c)
boolean equals(Object other)
boolean addAll(Collection<? extends E> from)
boolean remove(Object obj)
boolean removeAll(Collection<?> c)
boolean removeIf(Predicate<? super E> filter)
boolean retainAll(Collection<?> c)
void clear()
Object[] toArray()
```

注：`addAll()`、`retainAll()`和`removeAll()`可理解为集合(set)操作（见9.6.5节）：
* `a.addAll(b)` 并集($A = A \cup B$)
* `a.retainAll(b)` 交集($A = A \cap B$)
* `a.removeAll(b)` 差集($A = A - B$)

为了让实现者更轻松，Java库提供了`AbstractCollection`类，其中基础方法`size()`和`iterator()`仍为抽象方法，根据它们实现了其他实用方法（`add()`默认抛出`UnsupportedOperationException`）。`toString()`方法会调用所有元素的`toString()`，并生成一个`[A, B, C]`格式的字符串，这对于调试很方便。

具体集合类可以扩展`AbstractCollection`类，并提供抽象方法。不过，如果子类有更加高效的方式实现`contains()`，完全可以这么做。

注：`Collection`接口的伴随类`Collections`包含操作和创建集合的静态方法，见9.6节。

## 9.2 集合框架中的接口
Java集合框架为不同类型的集合定义了许多接口，如下图所示。

![集合框架中的接口](/assets/images/java-note-v1ch09-collections/集合框架中的接口.png)

有两个基本的集合接口：`Collection`和`Map`。已经看到，可以用`add(e)`方法在集合中添加元素，使用迭代器访问元素。而映射包含键值对，使用`put(k, v)`方法插入键值对，使用`get(k)`方法读取键对应的值。映射将在9.4节介绍。

注意：`Map`并未扩展`Collection`接口。

`List`是**有序集合**(ordered collection)（也叫做**序列**(sequence)）（注：这里的“有序”是指元素的位置顺序，不是大小顺序）。可以用两种方式访问元素：迭代器或整数**索引**(index)。后者称为**随机访问**(random access)，因为可以按任意顺序访问元素。相反，使用迭代器时必须顺序地访问元素。`List`接口定义了几个用于随机访问的方法：

```java
void add(int index, E element)
E remove(int index)
E get(int index)
E set(int index, E element)
```

`ListIterator`接口是`Iterator`的子接口，支持双向迭代，`add()`方法用于在迭代器位置前面插入元素。

实际上有两种列表，其性能有很大差异。数组列表(`ArrayList`)可以快速地随机访问，使用索引是有意义的。相反，链表(`LinkedList`)随机访问很慢，最好使用迭代器来遍历。

注释：为了避免对链表执行随机访问操作，Java 1.4引入了标记接口`RandomAccess`。这个接口没有方法，但可以用来测试一个集合是否支持高效的随机访问。

`Set`接口表示**集**(set)（数学上叫做“集合”，这里为了与 "collection" 区分），其`add()`方法不允许添加重复的元素。`equals()`方法定义为：只要两个集包含相同的元素就认为是相等的，而不要求元素有相同的顺序。`hashCode()`方法的定义要保证包含相同元素的两个集会得到相同的散列码。集将在9.3.3和9.3.4节介绍。

## 9.3 具体集合
下表展示了Java标准库中的集合（这里省略了线程安全集合，将在第12章介绍）。其中，以`Map`结尾的类实现了`Map`接口，其他类都实现了`Collection`接口。

| 集合类 | 描述 |
| --- | --- |
| `ArrayList` | 可以动态增长和缩减的索引序列 |
| `LinkedList` | 可以在任意位置高效插入和删除的有序序列 |
| `ArrayDeque` | 循环数组实现的双端队列 |
| `PriorityQueue` | 可以高效删除最小元素的集合 |
| `HashSet` | 没有重复元素的无序集 |
| `TreeSet` | 有序集 |
| `EnumSet` | 值为枚举类型的集 |
| `LinkedHashSet` | 可以记住元素插入顺序的集 |
| `HashMap` | 存储键值对的数据结构 |
| `TreeMap` | 键有序的映射 |
| `EnumMap` | 键为枚举类型的映射 |
| `LinkedHashMap` | 可以记住插入顺序的映射 |
| `WeakHashMap` | 如果值不在别处使用就会被垃圾收集的映射 |
| `IdentityHashMap` | 用`==`而不是`equals()`比较键的映射 |

下图显示了这些类之间的关系。

![集合框架中的类](/assets/images/java-note-v1ch09-collections/集合框架中的类.png)

注：
* Java集合类与C++ STL容器的对应关系见[《C++程序设计原理与实践》笔记 第20章]({% post_url 2023-07-15-ppp-note-ch20-containers-and-iterators %}) 20.10节。
* 所有的Java集合类都是可变的，而Scala集合类还分为可变和不可变，详见[Mutable and Immutable Collections](https://docs.scala-lang.org/overviews/collections-2.13/overview.html)。

### 9.3.1 链表
数组和数组列表有一个主要缺点：从数组中间插入或删除一个元素代价很大，因为后面的所有元素都要移动。

**链表**(linked list)解决了这个问题。数组将对象引用存储在连续的内存位置（见5.3.1节图1），而链表将每个对象存储在单独的**链接**(link)（节点）中，每个链接还存储序列中下一个链接的引用。在Java中，`LinkedList`实际上是**双向链表**(doubly linked list)，即每个链接还存储其前驱的引用，如下图所示。

![双向链表](/assets/images/java-note-v1ch09-collections/双向链表.png)

从链表中间删除元素是一个很廉价的操作——只需更新被删除元素相邻的链接（见下图）。

![从链表中删除元素](/assets/images/java-note-v1ch09-collections/从链表中删除元素.png)

下面的代码示例向链表中添加三个元素，然后删除第二个元素：

```java
var staff = new LinkedList<String>();
staff.add("Amy");
staff.add("Bob");
staff.add("Carl");
ListIterator<String> iter = staff.iterator();
String first = iter.next(); // visit first element
String second = iter.next(); // visit second element
iter.remove(); // remove last visited element
```

`LinkedList.add()`方法将对象添加到链表末尾。要在链表的中间添加对象，可以使用`ListIterator.add()`方法，该方法在迭代器位置**之前**添加新元素。链表的`listIterator()`方法返回一个实现了`ListIterator`接口的对象。

例如，下面的代码在第二个元素之前添加 "Juliet" （见下图）。

```java
var staff = new LinkedList<String>();
staff.add("Amy");
staff.add("Bob");
staff.add("Carl");
ListIterator<String> iter = staff.listIterator();
iter.next(); // skip past first element
iter.add("Juliet");
```

![向链表中插入元素](/assets/images/java-note-v1ch09-collections/向链表中插入元素.png)

注：`LinkedList`也提供了`add(index, element)`方法，但该方法会从头开始遍历到第`index`个元素，是效率很低的随机访问方法。

另外，`ListIterator`接口有两个方法可以用来反向遍历链表：`previous()`和`hasPrevious()`。

如果多次调用`add()`方法，元素将被依次添加到当前迭代器位置之前。在上面的示例中，调用`iter.add("Juliet")`之后，迭代器位于 "Juliet" 和 "Bob" 之间。如果再调用`iter.add("Doug")`，则`staff`将变为`[Amy, Juliet, Doug, Bob, Carl]`。

如果链表有n个元素，则有n+1个位置可以添加新元素（对应迭代器可能的位置）。例如，如果链表包含3个元素[A, B, C]，就有4个位置（标记为`|`）可以插入新元素：

```
|ABC
A|BC
AB|C
ABC|
```

注释：在用“光标”做类比时要当心。`ListIterator.remove()`操作与退格(Backspace)键并不完全一样。在调用`next()`之后立即调用`remove()`确实会删除迭代器左侧的元素，这与退格键一样。但是，如果刚刚调用了`previous()`，则会删除迭代器右侧的元素。而且不能连续调用两次`remove()`。

注：
* 链表迭代器的`add()`方法只依赖于迭代器的位置，而`remove()`方法还依赖于迭代器的状态（方向）。
* `remove()`方法删除“上次返回的元素”，由`LinkedList.ListItr`类的`lastReturned`字段表示。`next()`和`previous()`方法会更新该字段，而`add()`和`remove()`会将其置为`null`。

列表迭代器的`set()`方法将上次（调用`next()`或`previous()`）返回的元素替换为新元素。例如，下面的代码将列表的第一个元素替换为新值：

```java
ListIterator<String> iter = list.listIterator();
String oldValue = iter.next(); // returns first element
iter.set(newValue); // sets first element to newValue
```

如果一个迭代器修改集合时，另一个迭代器在遍历这个集合，可能就会出现混乱。如果迭代器发现它的集合被另一个迭代器或集合本身的方法修改了，就会抛出`ConcurrentModificationException`。例如：

```java
List<String> list = ...;
ListIterator<String> iter1 = list.listIterator();
ListIterator<String> iter2 = list.listIterator();
iter1.next();
iter1.remove();
iter2.next(); // throws ConcurrentModificationException
```

为了避免这一问题，可以遵循这个简单的规则：为一个集合关联任意多个只读的迭代器，或者只关联一个可以读写的迭代器。

检测并发修改的做法很简单。集合会跟踪修改操作的次数，每个迭代器都会单独计数它所负责的修改操作次数。在每个迭代器方法的开始处，迭代器会检查自己的修改计数与集合的计数是否相等。如果不相等就抛出异常。（注：参见`AbstractList.modCount`和`LinkedList.ListItr.expectedModCount`字段）

链表不支持快速随机访问。如果要访问链表的第n个元素，就必须从头开始越过前n-1个元素，没有捷径。因此，需要按索引访问元素时，通常不使用链表。

然而，`LinkedList`类还是提供了用来访问特定元素的`get()`和`set()`方法。绝对不要使用这个虚假的随机访问方法来遍历链表。下面的代码效率极低，每次访问一个元素都要从头开始搜索。

```java
for (int i = 0; i < list.size(); i++) {
    var elem = list.get(i);
    // do something with elem
}
```

`ListIterator`接口还有两个获取当前位置索引的方法：`nextIndex()`方法返回下次调用`next()`所返回元素的索引，`previousIndex()`方法返回下次调用`previous()`所返回元素的索引（见下图）。

![列表迭代器的索引](/assets/images/java-note-v1ch09-collections/列表迭代器的索引.png)

最后，如果有一个整数索引n，则`list.listIterator(n)`返回一个指向索引为n的元素前面位置的迭代器，但链表获得这个迭代器的效率比较低（需要从头开始遍历）。

使用链表的唯一理由是尽可能减少在列表中间插入或删除元素的开销。如果需要对集合进行随机访问，就使用数组或`ArrayList`。

程序清单9-1中的程序使用了链表。创建了两个链表，将它们合并，然后从第二个链表中每隔一个元素删除一个，最后测试`removeAll()`方法。

[程序清单9-1 linkedList/LinkedListTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch09/linkedList/LinkedListTest.java)

建议跟踪一下程序流程，特别要注意迭代器。你会发现画出迭代器位置示意图很有帮助，如下所示：

```
|ACE  |BDFG
A|CE  |BDFG
AB|CE B|DFG
...
```

### 9.3.2 数组列表
`ArrayList`类也实现了`List`接口。`ArrayList`类封装了一个动态分配的对象数组，支持快速的随机访问（具体用法已在5.3节介绍过）。

注释：Java标准库中还有一个表示动态数组的遗留类`Vector`。`Vector`类的所有方法都是同步的（线程安全的），而`ArrayList`不是线程安全的。建议在不需要同步（只从一个线程访问）时使用`ArrayList`而不是`Vector`。

### 9.3.3 散列集
数组和链表允许指定元素的次序。但是，如果要查找特定的元素，就需要访问所有元素，直到找到匹配的元素为止。

**散列表**(hash table)可用于快速查找对象。在Java中，散列表实现为**链表数组**，每个链表叫做**桶**(bucket)（见下图）。要查找一个对象，首先计算它的**散列码**(hash code)（见5.2.4节），然后对桶的总数取余，得到的结果就是保存这个元素的桶的索引。例如，某个对象的散列码是76268，总共有128个桶，那么这个对象所在的桶索引是76268 % 128 = 108。如果这个桶中没有其他元素，直接将元素插入桶中即可。如果桶中已经有元素，这叫做**散列冲突**(hash collision)，此时需要将新元素与这个桶中的所有元素进行比较，查看这个元素是否已经存在。如果散列码合理地随机分布，而且桶的数目足够大，需要比较的次数就会很少。

![散列表](/assets/images/java-note-v1ch09-collections/散列表.png)

注释：从Java 8起，桶中元素过多(>8)时会从链表变为平衡二叉树，这能够在产生大量冲突时提高性能。

提示：散列表的键要尽可能属于实现了`Comparable`接口的类，这样就能保证不会由于散列码分布不均匀而导致性能低下。

如果大致知道散列表中最终会有多少个元素，就可以设置初始桶数。通常，应该将桶数设置为预计元素个数的75%~150%。标准库使用的桶数是2的幂，默认为16（指定的桶数将自动调整为2的下一个幂）。

如果散列表太满，就需要**再散列**(rehash)。为此，需要创建一个桶数是原来的两倍的表，并将所有元素插入新表，然后丢弃原来的表。**装填因子**(load factor)决定何时进行再散列：如果键值对数量大于桶数×装填因子，就会自动再散列。例如，如果装填因子为0.75（默认值），桶数为16，则键值对数量大于9时就会再散列，新表的桶数为32。

散列表可用于实现几个重要的数据结构，其中最简单的是**集**(set)。集是没有重复的元素集合。Java标准库提供了`HashSet`类，基于散列表实现的集。其`add()`方法只有在元素不存在时才会添加，`contains()`方法可以快速查找一个元素是否存在（因为只查看一个桶中的元素）。

注：
* 散列表的另一个应用是实现映射，见9.4节。实际上`HashSet`就是利用`HashMap`实现的，只使用key不使用value。
* 上面介绍的就是Java标准库`HashMap`的实现方式，详见[HashMap源代码解读]({% post_url 2021-03-17-java-hash-map-source-code %})。

散列集迭代器会依次访问所有的桶。因为散列将元素分散在表中，所以会以一种看似随机的顺序访问元素。只有不关心集合中的元素顺序时才应该使用`HashSet`。

警告：在修改集中的元素时要格外小心。如果元素的散列码发生了变化，这个元素在散列表中将不再处于正确的位置（因此集的元素最好是不可变类型）。

程序清单9-2中的示例程序从`System.in`读取单词，将其添加到集中，最后打印前20个单词。例如，可以通过以下命令将《爱丽丝漫游仙境》(Alice in Wonderland)的文本（可以从 <https://www.gutenberg.org/> 获取）输入到这个程序：

```shell
java set.SetTest < alice30.txt
```

[程序清单9-2 set/SetTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch09/set/SetTest.java)

### 9.3.4 树集
树集`TreeSet`是一个**有序集合**(sorted collection)。可以以任意顺序插入元素，遍历集合时元素将自动地按照排序后的顺序出现。例如：

```java
var sorter = new TreeSet<String>();
sorter.add("Bob");
sorter.add("Amy");
sorter.add("Carl");
for (String s : sorter) System.out.println(s);
```

这将打印 "Amy Bob Carl" 。正如类名所示，排序是利用**树**(tree)结构完成的（当前实现使用的是**红黑树**(red-black tree)）。

将一个元素添加到树中要比添加到散列表慢，但比在数组或链表中检查重复要快得多。如果树包含n个元素，找到新元素的正确位置平均需要 $\log_2 n$ 次比较。

注释：要使用树集，必须能够比较元素。因此元素必须实现`Comparable`接口，或者构造集时提供一个`Comparator`（见第6章）。

应该使用树集还是散列集？答案取决于所要收集的数据。如果不需要数据有序，就没有必要付出排序的开销。更重要的是，对于某些数据排序要比散列更加困难（例如表示矩形的类`Rectangle`）。

注释：从Java 6起，`TreeSet`类实现了`NavigableSet`接口。这个接口增加了几个便利的方法，用于查找元素和反向遍历。

程序清单9-3中的程序创建了两个`Item`对象的树集。第一个按照编号排序（`Item`对象的默认排序顺序），第二个按照描述信息排序（使用自定义比较器）。

[程序清单9-3 treeSet/TreeSetTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch09/treeSet/TreeSetTest.java)

[程序清单9-4 treeSet/Item.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch09/treeSet/Item.java)

### 9.3.5 队列和双端队列
前面已经讨论过，队列允许高效地在队尾添加元素、在队头删除元素。**双端队列**(deque)在队头和队尾都能高效地添加或删除元素。不支持在队列中间添加元素。Java 6引入了`Deque`接口，`ArrayDeque`和`LinkedList`类实现了这个接口。

注：`Queue`接口对于插入、删除和查看操作都提供了两种形式，区别在于队列满或空时的处理方式：

| 操作 | 抛出异常 | 返回特殊值 |
| --- | --- | --- |
| 插入 | `add(e)` | `offer(e)` |
| 删除 | `remove()` | `poll()` |
| 查看 | `element()` | `peek()` |

类似地，`Deque`接口对于在队头和队尾的三种操作分别提供了两种形式，详见API文档。

### 9.3.6 优先队列
**优先队列**(priority queue)可以按任意顺序插入元素，但是按排序后的顺序删除元素。也就是说，调用`remove()`方法时总会获得当前最小的元素。不过，优先队列并没有对所有元素进行排序。如果遍历元素，它们不一定是有序的。优先队列使用了一种精巧且高效的数据结构，称为**堆**(heap)。堆是一个自组织的二叉树，其`add()`和`remove()`操作会让最小的元素移动到根，而不必花费时间对所有元素进行排序。

与`TreeSet`一样，`PriorityQueue`的元素可以实现`Comparable`，或者在构造器中提供一个`Comparator`对象。

注：默认构造的是小根堆。要构造大根堆，可在构造器中指定比较器`(a, b) -> -a.compareTo(b)`、`(a, b) -> b.compareTo(a)`或`Comparator.reverseOrder()`。

优先队列的典型用法是作业调度。每个作业有一个优先级，作业以随机顺序添加。每当可以启动一个新的作业时，将从队列中取出优先级最高的作业（习惯上将1作为最高优先级）。

程序清单9-5具体使用了优先队列。与`TreeSet`不同，这里的迭代并不是按照排序的顺序访问元素，但删除操作总是返回最小的元素。

[程序清单9-5 priorityQueue/PriorityQueueTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch09/priorityQueue/PriorityQueueTest.java)

## 9.4 映射
**映射**(map)存储键值对（也叫做**条目**(entry)）。提供一个键，可以查找对应的值。例如，可以存储一张员工记录表，其中键是员工ID，值是`Employee`对象。在下面的小节中，将学习如何使用映射。

### 9.4.1 基本映射操作
Java标准库为映射提供了两个通用的实现：`HashMap`和`TreeMap`。这两个类都实现了`Map`接口。

注：`Map`接口有两个类型参数`K`和`V`，分别表示键类型和值类型。

散列映射对键进行散列，树映射根据键的顺序将其组织成搜索树。散列或比较函数**只应用于键**。与键关联的值不会被散列或比较。

应该选择散列映射还是树映射？与集一样，散列通常稍快一些。如果不需要按照排序的顺序访问键，最好选择散列映射。

可以如下创建一个散列映射来存储员工信息：

```java
var staff = new HashMap<String, Employee>(); // HashMap implements Map
var harry = new Employee("Harry Hacker");
staff.put("987-98-9996", harry);
...
```

每当向映射中添加一个对象时，必须同时提供一个键。

要获取对象，必须使用键。

```java
var id = "987-98-9996";
Employee e = staff.get(id); // gets harry
```

如果映射中没有指定的键，则返回`null`。

有时对于不存在的键有一个合适的默认值，那么可以使用`getOrDefault()`方法。

```java
Map<String, Integer> scores = ...;
int score = scores.getOrDefault(id, 0); // gets 0 if the id is not present
```

**键必须是唯一的**。不能对同一个键存储两个值。如果用同一个键调用两次`put()`方法，第二个值就会取代第一个值。`put()`会返回之前与键关联的值（如果键不存在则返回`null`）。

`remove()`方法从映射中删除给定的键对应的元素。`size()`方法返回映射中的键值对数量。

迭代映射的键和值最简单的方法是`forEach()`方法，提供一个接收键和值的lambda表达式。

```java
scores.forEach((k, v) -> System.out.println("key=" + k + ", value=" + v));
```

注：`Map`没有扩展`Iterable`接口，因此映射本身不能使用for each循环，但映射视图可以（见9.4.3节）。

程序清单9-6展示了映射的具体使用。

[程序清单9-6 map/MapTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch09/map/MapTest.java)

小结：下面列出了常用的映射操作，完整列表参见API文档。
* `get(k)` 返回键`k`关联的值，如果不存在则返回`null`
* `getOrDefault(k, d)` 返回键`k`关联的值，如果不存在则返回默认值`d`
* `put(k, v)` 将值`v`关联到键`k`，返回旧的值
* `putIfAbsent(k, v)` 如果键`k`不存在则将其关联到值`v`并返回`null`，否则返回当前值
* `replace(k, v)` 如果键`k`存在则将其关联到值`v`并返回旧的值，否则返回`null`
* `replace(k, oldValue, newValue)` 如果键`k`关联的值等于`oldValue`则将其替换为`newValue`并返回`true`，否则返回`false`
* `remove(k)` 如果键`k`存在则将其删除，返回之前关联的值
* `containsKey(k)` 如果映射中存在键`k`则返回`true`
* `containsValue(v)` 如果映射中存在值`v`则返回`true`
* `size()` 返回映射中的键值对数量

### 9.4.2 更新映射条目
处理映射的一个难点是更新映射条目。正常情况下，可以获取与键关联的旧值，更新它，再放回更新后的值。但是必须考虑键第一次出现的特殊情况。考虑使用映射来统计一个文件中的单词出现频次。看到一个单词时，将计数器加1：

```java
counts.put(word, counts.get(word) + 1);
```

这是可以的，但有一种情况除外：即第一次遇到`word`时。此时`get()`会返回`null`，因此会出现`NullPointerException`。

一种简单的方法是使用`getOrDefault()`方法：

```java
counts.put(word, counts.getOrDefault(word, 0) + 1);
```

另一种方法是先调用`putIfAbsent()`：

```java
counts.putIfAbsent(word, 0);
counts.put(word, counts.get(word) + 1); // now we know that get will succeed
```

不过还可以做得更好。`merge()`方法可以简化这个常见的操作。以下调用

```java
counts.merge(word, 1, Integer::sum);
```

如果原先键不存在则把`word`与1关联，否则使用`Integer::sum`函数组合原值和1。

小结：下面列出了一些更新映射条目的方法，完整列表参见API文档。
* `merge(k, v, f)` 大致等价于`put(k, get(k) == null ? v : f(get(k), v))`
* `compute(k, f)` 大致等价于`put(k, f(k, get(k)))`
* `computeIfAbsent(k, f)` 大致等价于`putIfAbsent(k, f(k))`
* `computeIfPresent(k, f)` 大致等价于`if (get(k) != null) put(k, f(k, get(k)))`
* `replaceAll(f)` 大致等价于`for ((k, v) : entrySet()) put(k, f(k, v))`

### 9.4.3 映射视图
Java集合框架不认为映射是一种集合（`Map`没有扩展`Collection`）（相反，C++ STL认为映射是键值对的集合）。不过，可以得到映射的**视图**(view)——实现了`Collection`或其子接口的对象。

有三种视图：键集、值集合（不是集）以及键值对集（条目集）。以下方法

```java
Set<K> keySet()
Collection<V> values()
Set<Map.Entry<K, V>> entrySet()
```

分别返回这三个视图。（条目集的元素是实现了`Map.Entry`接口的类的对象。）

注意，`keySet()`返回的不是`HashSet`或`TreeSet`，而是实现了`Set`接口的另外某个类的对象（`HashMap`和`TreeMap`各有一个名为`KeySet`的内部类）。

例如，可以遍历一个映射的所有键：

```java
for (String key : staff.keySet()) {
    // do something with key
}
```

如果想同时查看键和值，可以通过遍历条目来避免查找值：

```java
for (Map.Entry<String, Employee> entry : staff.entrySet()) {
    String k = entry.getKey();
    Employee v = entry.getValue();
    // do something with k, v
}
```

注释：通过使用`var`声明可以避免冗长的变量类型：

```java
for (var entry : map.entrySet()) {
    // do something with entry.getKey(), entry.getValue()
}
```

或者直接使用`forEach()`方法：

```java
map.forEach((k, v) -> {
    // do something with k, v
});
```

如果调用视图迭代器的`remove()`方法，实际上会从映射中删除这个键和关联的值。不过，不能向视图中添加元素，其`add()`方法会抛出`UnsupportedOperationException`。

### 9.4.4 弱散列映射
在集合类库中有几个用于特殊需求的映射类，本节和后面几节将做简要介绍。

`WeakHashMap`类旨在解决一个有趣的问题。假设对某个键的最后一个引用已经消失，就再也没有任何方法可以访问关联的值对象了（注：实际上`IdentityHashMap`才会有这个问题，`HashMap`只要使用散列码相同的键仍然可以访问）。但是，垃圾收集器不能回收这个键值对，因为映射的桶还引用它（只有当整个映射对象被回收时其中的键值对才会被回收）。此时可以使用`WeakHashMap`，这个类将与垃圾收集器合作，删除那些键的唯一引用来自映射条目的键值对。

考虑下面的例子

```java
Map<String, String> map = new WeakHashMap<>();
String key1 = new String("key1");
String key2 = new String("key2");

map.put(key1, "value1");
map.put(key2, "value2");
System.out.println("Before: " + map);

key1 = null; // the last reference to key1 has gone away
System.gc();
Thread.sleep(1000);
System.out.println("After: " + map);
```

当变量`key1`置为`null`后，映射条目对键`"key1"`（下图中红色的字符串对象）的引用就成了唯一的引用。如果此时发生垃圾收集，这个键值对就会被删除。程序输出如下：

```
Before: {key1=value1, key2=value2}
After: {key2=value2}
```

![弱散列映射](/assets/images/java-note-v1ch09-collections/弱散列映射.png)

注：
* `WeakHashMap`使用`equals()`而不是`==`判断键相等，因此在将`key1`置为`null`后立即查询`get("key1")`仍然能访问到`value1`（即使字符串常量`"key1"`和映射中的键`"key1"`并非同一个对象）。
* 在实际中，`WeakHashMap`通常用于实现缓存，避免内存泄露。

下面是这种机制的内部工作原理。`WeakHashMap`使用**弱引用**(weak reference)存储键值对。正常情况下，如果垃圾收集器发现某个对象已经没有引用了，就会将其回收。不过，如果这个对象只能由`WeakReference`访问，垃圾收集器也会将其回收，但会将这个弱引用放入一个队列。`WeakHashMap`的操作会定期检查这个队列，查找新加入的弱引用，并删除关联的条目。

### 9.4.5 链接散列集和映射
`LinkedHashSet`和`LinkedHashMap`类会记住插入元素的顺序。插入条目时，会将其加入一个双向链表（见下图）。

![链接散列表](/assets/images/java-note-v1ch09-collections/链接散列表.png)

例如，考虑程序清单9-6中的映射插入操作：

```java
var staff = new LinkedHashMap<String, Employee>();
staff.put("144-25-5464", new Employee("Amy Lee"));
staff.put("567-24-2546", new Employee("Harry Hacker"));
staff.put("157-62-7935", new Employee("Gary Cooper"));
staff.put("456-62-5527", new Employee("Francesca Cruz"));
```

则`staff`会按以下顺序迭代条目：

```
144-25-5464=Amy Lee
567-24-2546=Harry Hacker
157-62-7935=Gary Cooper
456-62-5527=Francesca Cruz
```

链接散列映射也可以使用**访问顺序**(access order)而不是插入顺序来迭代条目。每次调用`get()`或`put()`时，受影响的条目将被移至链表末尾（不影响散列表的桶）。为此，需要提供构造器的第三个参数：

```java
new LinkedHashMap<>(initialCapacity, loadFactor, true)
```

注：使用访问顺序的`LinkedHashMap`类似于Python的`collections.OrderedDict`。

例如，对于上图中的映射，当前顺序为①②③④。如果访问了条目②，则顺序将变为①③④②，如下图所示。

![更新后的链接散列表](/assets/images/java-note-v1ch09-collections/更新后的链接散列表.png)

访问顺序对于实现“最近最少使用”(least recently used, LRU)缓存十分有用。例如，你希望将频繁访问的元素放在内存中（使用`LinkedHashMap`作为缓存），而从数据库读取访问频率低的元素。如果要向缓存中插入数据，而映射已经很满，就可以迭代并删除前几个元素（这些是最近最少使用的元素）。

甚至可以自动完成这一过程。构造一个`LinkedHashMap`的子类，然后覆盖`removeEldestEntry()`方法。每次调用`put()`时就会自动调用该方法，如果返回`true`则删除最旧的条目。例如，下面的缓存最多可以存放100个元素：

```java
var cache = new LinkedHashMap<K, V>(128, 0.75f, true) {
    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > 100;
    }
};
```

另外，还可以根据参数`eldest`（最旧的条目）来决定是否将它删除。例如，检查条目中存储的时间戳。

### 9.4.6 枚举集和映射
`EnumSet`是一种高效的集实现，其元素属于枚举类型。因为枚举类型只有有限个实例，`EnumSet`内部实现为位(bit)序列。如果一个值在集中，则对应的位被置为1。

`EnumSet`类没有公共构造器，需要使用静态工厂方法来构造：

```java
enum Weekday { MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY }
EnumSet<Weekday> always = EnumSet.allOf(Weekday.class); // 1111111
EnumSet<Weekday> never = EnumSet.noneOf(Weekday.class); // 0000000
EnumSet<Weekday> workday = EnumSet.range(Weekday.MONDAY, Weekday.FRIDAY);             // 0011111
EnumSet<Weekday> mwf = EnumSet.of(Weekday.MONDAY, Weekday.WEDNESDAY, Weekday.FRIDAY); // 0010101
```

可以使用`Set`接口的方法来修改`EnumSet`。

`EnumMap`是一种键属于枚举类型的映射。它简单高效地实现为一个值数组（使用键的序数`ordinal()`作为索引）。需要在构造器中指定键类型：

```java
var personInCharge = new EnumMap<Weekday, Employee>(Weekday.class);
```

### 9.4.7 标识散列映射
`IdentityHashMap`有一个很特殊的用途。在这里，键的散列码不是使用`hashCode()`方法计算的，而是使用`System.identityHashCode()`方法（`Object.hashCode()`根据对象的内存地址计算散列码时就使用了这个方法）。另外，`IdentityHashMap`使用`==`而不是`equals()`比较键对象。在实现对象遍历算法（如对象序列化）时，你想跟踪哪些对象已经遍历过，这个类就很有用。

## 9.5 副本和视图
通过使用**视图**(view)可以得到（除了标准库提供的类以外）其他实现了`Collection`或`Map`接口的对象。映射类的`keySet()`方法就是这样一个例子，这个方法返回的集可以操作原映射。这种集合称为视图。

视图技术在集合框架中有许多非常有用的应用。下面几节将讨论这些应用。

### 9.5.1 小集合
Java 9引入了一些静态方法，可以生成具有给定元素的列表、集或映射（Java终于有集合“字面值”了）。例如：

```java
List<String> names = List.of("Peter", "Paul", "Mary");
Set<Integer> numbers = Set.of(2, 3, 5);
Map<String, Integer> scores = Map.of("Peter", 2, "Paul", 3, "Mary", 5);
```

元素、键或值不能为`null`。集和映射键不能重复：

```java
numbers = Set.of(13, null); // Error - null element
scores = Map.of("Peter", 4, "Peter", 2); // Error - duplicate key
```

警告：对于这些集和映射中的元素顺序并没有任何保证。实际上，会故意用每次虚拟机启动时随机得到的一个种子打乱这个顺序。编写程序时不要对元素顺序做任何假设。

`List`和`Set`接口有11个`of()`方法，分别有0到10个参数，另外还有一个参数个数可变的`of()`方法。提供这种特化是为了效率（参数较少时无需构造数组）。

`Map`接口有一个静态方法`ofEntries()`，能接受任意多个`Map.Entry<K, V>`对象（可以用静态方法`entry()`创建）。例如：

```java
import static java.util.Map.*;

Map<String, Integer> scores = ofEntries(
    entry("Peter", 2),
    entry("Paul", 3),
    entry("Mary", 5));
```

这些集合对象是**不可修改的**(unmodifiable)。如果试图改变它们的内容，会导致`UnsupportedOperationException`。

如果需要一个可变的集合，可以把不可修改的集合传递给构造器：

```java
var names = new ArrayList<>(List.of("Peter", "Paul", "Mary")); // A mutable list of names
```

方法调用`Collections.nCopies(n, obj)`返回一个包含`n`个`obj`的不可变列表（实际上存储代价很小，对象只存储一次）。例如，下面创建了一个包含100个字符串`"DEFAULT"`的列表：

```java
List<String> settings = Collections.nCopies(100, "DEFAULT");
```

注释：`of()`方法是Java 9引入的。之前有静态方法`Arrays.asList()`、`Collections.emptySet()`和`Collections.singleton()`等。

注释：`Collections`类包含很多实用方法，这些方法的参数或返回值是集合。不要将它与`Collection`接口混淆。

提示：Java没有`Pair`类，有些程序员会使用`Map.Entry`。在Java 9之前，这会很麻烦——你必须构造`new AbstractMap.SimpleImmutableEntry<>(first, second)`。现在可以调用`Map.entry(first, second)`。

### 9.5.2 不可修改副本和视图
要创建集合的**不可修改副本**(unmodifiable copy)，可以使用集合类型的`copyOf()`方法：

```java
ArrayList<String> names = ...;
Set<String> nameSet = Set.copyOf(names); // The names as an unmodifiable set
List<String> nameList = List.copyOf(names); // The names as an unmodifiable list
```

`copyOf()`方法会创建集合的一个副本。如果修改了原集合，这个副本不受影响。

如果原集合恰好是不可修改的，而且类型正确，则`copyOf()`会直接返回它：

```java
Set<String> names = Set.of("Peter", "Paul", "Mary");
Set<String> nameSet = Set.copyOf(names); // No need to make a copy: names == nameSet
```

`Collections`类还有一些方法可以产生集合的**不可修改视图**(unmodifiable view)。这些视图对现有集合增加了一个运行时检查，如果试图修改集合就抛出一个异常。不过，如果原集合改变，视图会反映这些变化。这正是视图与副本的区别。

可以使用`Collections`类的`unmodifiableList()`、`unmodifiableSet()`和`unmodifiableMap()`等方法获得不可修改视图，每个方法用于一种接口。

例如，假设你想让某部分代码查看（但不能修改）一个集合的内容，可以如下实现：

```java
var staff = new LinkedList<String>();
...
lookAt(Collections.unmodifiableList(staff));
```

`unmodifiableList()`方法返回的对象的访问器方法会调用底层集合的方法，但所有的修改器方法被重新定义为抛出`UnsupportedOperationException`。

不可修改视图并不会让集合本身不可变。仍然可以通过原始引用（在这里是`staff`）修改这个集合。

由于视图只是包装了接口而不是具体的集合类，所以只能访问接口中定义的方法。例如，`LinkedList`类的`addFirst()`和`addLast()`方法不是`List`接口的一部分，因此不能通过视图调用。

警告：`unmodifiableCollection()`返回的集合的`equals()`方法并不调用底层集合的`equals()`方法，而是继承自`Object`，`hashCode()`方法也是一样。不过，`unmodifiableSet()`和`unmodifiableList()`会使用底层集合的`equals()`和`hashCode()`方法。

### 9.5.3 子范围
可以为很多集合创建**子范围视图**(subrange view)。例如，假设有一个列表`staff`，想从中取出第10~19个元素。可以使用`subList()`方法获得这个列表子范围的视图：

```java
List<Employee> group2 = staff.subList(10, 20);
```

第一个索引包含、第二个不包含，类似于`String.substring()`的参数。

可以对子范围应用任何操作，它们会自动反映到整个列表。例如，可以删除整个子范围：

```java
group2.clear(); // staff reduction
```

这些元素会自动地从`staff`列表中清除，而`group2`变为空。

对于有序集和映射，使用排序顺序（而不是元素位置）创建子范围。`SortedSet`接口声明了三个方法：

```java
SortedSet<E> subSet(E from, E to)
SortedSet<E> headSet(E to)
SortedSet<E> tailSet(E from)
```

这些方法返回大于等于`from`且小于`to`的所有元素构成的子集。有序映射也有类似的方法：

```java
SortedMap<K, V> subMap(K from, K to)
SortedMap<K, V> headMap(K to)
SortedMap<K, V> tailMap(K from)
```

这些方法返回**键**落在指定范围内的所有条目构成的映射视图。

Java 6引入的`NavigableSet`接口允许对这些子范围操作有更多控制。可以指定是否包括边界：

```java
NavigableSet<E> subSet(E from, boolean fromInclusive, E to, boolean toInclusive)
NavigableSet<E> headSet(E to, boolean toInclusive)
NavigableSet<E> tailSet(E from, boolean fromInclusive)
```

### 9.5.4 检查型视图
**检查型视图**(checked view)用来对泛型类型可能出现的问题提供调试支持。如8.5.4节所述，有可能将错误类型的元素混入泛型集合中。例如：

```java
var strings = new ArrayList<String>();
ArrayList rawList = strings; // warning only, not an error, for compatibility with legacy code
rawList.add(new Date()); // now strings contains a Date object!
```

这个错误的`add()`命令在运行时检测不到。相反，只有当其他代码调用`get()`并将结果强制转换为`String`时，才会发生`ClassCastException`。

检查型视图可以检测这类问题。如下定义一个安全列表：

```java
List<String> safeStrings = Collections.checkedList(strings, String.class);
```

这个视图的`add()`方法将检查插入的对象是否属于给定的类，如果不是就立即抛出`ClassCastException`。这样做的好处是可以在正确的位置报告错误：

```java
ArrayList rawList = safeStrings;
rawList.add(new Date()); // checked list throws a ClassCastException
```

警告：检查型视图受限于虚拟机可以完成的运行时检查。例如，对于`ArrayList<Pair<String>>`，就无法阻止插入`Pair<Date>`，因为虚拟机只有一个“原始”`Pair`类。（让标准库来弥补语言本身的缺陷，但又没完全弥补……）

### 9.5.5 同步视图
如果从多个线程访问一个集合，就必须确保集合不会被意外破坏。例如，如果一个线程试图向散列表添加元素，同时另一个线程正在对元素再散列，其结果将是灾难性的。

库设计者使用视图机制使常规集合线程安全（而不是直接将其实现为线程安全的）。例如，静态方法`Collections.synchronizedMap()`可以将任意映射转换成有同步访问方法的`Map`：

```java
var staff = Collections.synchronizedMap(new HashMap<String, Employee>());
```

现在就可以从多个线程访问`staff`对象了。第12章将会详细讨论同步访问的问题。

注：`java.util.concurrent`包已经提供了一些线程安全的集合，如`ConcurrentHashMap`。

### 9.5.6 关于可选操作的说明
通常，视图有一些限制——可能只读，可能无法改变大小，或者可能只支持删除而不支持插入（如映射的键视图）。如果试图进行不恰当的操作，就会抛出`UnsupportedOperationException`。

在集合和迭代器接口的API文档中，许多方法描述为“可选操作”(optional operation)。这看起来与接口的概念有冲突。毕竟，接口的目的就是说明一个类**必须**实现的方法。确实，从理论的角度看，这种安排并不令人满意。一种更好的解决方案是为只读视图和不能改变大小的视图设计单独的接口。不过，这将会使接口的数量增至原来的三倍，让库设计者无法接受。

不应该将“可选”方法这一技术用于自己的设计中。尽管集合被频繁地使用，但实现集合的编码风格未必适用于其他问题领域。集合类库的设计者必须解决一组严格且相互冲突的需求——易于学习、使用方便、彻底泛型化，同时又与手写算法一样高效。要同时达到（甚至接近）这些目标显然是不可能的。但是，在你自己的编程问题中，很少遇到这种极端的约束。你应该能够找到合适的解决方案，而不必依赖可选接口操作这种极端做法。

## 9.6 算法
除了实现集合类，Java集合框架还提供了许多有用的算法。本节将介绍如何使用这些算法，以及如何编写自己的算法。

### 9.6.1 为什么使用泛型算法
泛型集合接口有一个很大的优点，即算法只需要实现一次。例如，考虑找出集合中最大元素的简单算法。可以用循环实现这个算法。下面是找出数组中最大元素的算法：

```java
public static <T extends Comparable> T max(T[] a) {
    if (a.length == 0) throw new NoSuchElementException();
    T largest = a[0];
    for (int i = 1; i < a.length; i++)
        if (largest.compareTo(a[i]) < 0)
            largest = a[i];
    return largest;
}
```

要找出数组列表中的最大元素，编写的代码会稍有差别：

```java
public static <T extends Comparable> T max(ArrayList<T> v) {
    if (v.size() == 0) throw new NoSuchElementException();
    T largest = v.get(0);
    for (int i = 1; i < v.size(); i++)
        if (largest.compareTo(v.get(i)) < 0)
            largest = v.get(i);
    return largest;
}
```

链表没有高效的随机访问，但可以使用迭代器：

```java
public static <T extends Comparable> T max(LinkedList<T> l) {
    if (l.isEmpty()) throw new NoSuchElementException();
    Iterator<T> iter = l.iterator();
    T largest = iter.next();
    while (iter.hasNext()) {
        T next = iter.next();
        if (largest.compareTo(next) < 0)
            largest = next;
    }
    return largest;
}
```

编写这些循环很繁琐，而且容易出错。是否存在“差1”错误？对于空集合能正常工作吗？对于只有一个元素的集合呢？我们不希望每次都测试和调试这些代码，也不想实现一系列方法。

这正是集合接口的用武之地。考虑执行这个算法所需要的**最小**集合接口。这项任务并不需要随机访问，直接迭代元素即可。因此，可以将`max()`方法实现为接受**任何**实现了`Collection`接口的对象（只用到`isEmpty()`和`iterator()`两个方法）：

```java
public static <T extends Comparable> T max(Collection<T> c) {
    if (c.isEmpty()) throw new NoSuchElementException();
    Iterator<T> iter = c.iterator();
    T largest = iter.next();
    while (iter.hasNext()) {
        T next = iter.next();
        if (largest.compareTo(next) < 0)
            largest = next;
    }
    return largest;
}
```

现在就可以用一个方法来计算任意集合中的最大值了。

这是一个很强大的概念。Java标准库包含了一些基本的算法（定义在`Collections`类中）：排序、二分查找和一些实用算法。

注：数组没有实现`Collection`接口，因此不能直接把数组传递给`max()`方法，但可以借助视图`List.of()`或`Arrays.asList()`（见9.5.1节）。注意，基本类型数组应改为包装器类型。

### 9.6.2 排序和洗牌
`Collections`类的`sort()`方法对一个`List`进行排序。

```java
var staff = new LinkedList<String>();
// fill collection
Collections.sort(staff);
```

这个方法假定列表元素实现了`Comparable`接口。如果想采用其他方式对列表进行排序，可以传入一个`Comparator`对象。可以如下对员工列表按工资排序：

```java
Collections.sort(staff, Comparator.comparingDouble(Employee::getSalary));
```

注：实际上`Collections.sort()`方法直接调用`List.sort()`。因此`Collections.sort(list)`等价于`list.sort()`，`Collections.sort(list, comp)`等价于`list.sort(comp)`。

如果想按照**降序**对列表进行排序，可以使用静态方法`Comparator.reverseOrder()`或实例方法`reversed()`（见6.2.8节）。例如，

```java
staff.sort(Comparator.reverseOrder());
```

按照`compareTo()`方法给定顺序的逆序对列表`staff`进行排序。类似地，

```java
staff.sort(Comparator.comparingDouble(Employee::getSalary).reversed());
```

按工资降序排序。

通常，算法书籍介绍的排序算法都是有关数组的，而且使用的是随机访问方式。但是，链表的随机访问效率很低（注：因此C++标准库的`sort()`算法不能用于链表，而要用链表类的`sort()`函数）。有一种归并排序可以高效地对链表排序。不过Java并不是这样实现的，而是将所有元素放在一个数组中，对这个数组排序，再将排序后的数组复制回链表。

集合库使用的排序算法比快速排序(quick sort)要慢一些，不过有一个主要的有点：**稳定**(stable)。也就是说，不会改变相等元素的顺序。例如，假设有一个已经按姓名排序的员工列表，现在再按工资排序。如果采用稳定的排序算法，对于工资相等的员工，将会保持按名字排序的顺序。换句话说，结果是一个先按工资排序、再按姓名排序的列表。

集合不需要实现所有的“可选”方法，因此所有接受集合参数的方法必须描述什么时候可以安全地将集合传递给它。例如，显然不能将`unmodifiableList()`返回的列表传递给`sort()`算法。根据文档说明，列表必须是可修改的，但不必是可改变大小的。有关的术语定义如下：
* 如果列表支持`set()`方法，则是**可修改的**(modifiable)。
* 如果列表支持`add()`和`remove()`方法，则是**可改变大小的**(resizable)。

`Collections`类的`shuffle()`算法与排序刚好相反，它会随机地打乱列表中元素的顺序（洗牌）。例如：

```java
ArrayList<Card> cards = ...;
Collections.shuffle(cards);
```

如果提供的列表没有实现`RandomAccess`接口，`shuffle()`方法就会将元素复制到一个数组中，打乱这个数组，最后再将打乱后的元素复制回列表。

程序清单9-7中的程序用1~49的整数填充数组列表。然后打乱这个列表，并从中选择前6个值。最后对选择的值进行排序并打印。

[程序清单9-7 shuffle/ShuffleTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch09/shuffle/ShuffleTest.java)

### 9.6.3 二分查找
要在数组中查找一个对象，通常要依次访问每个元素，直到找到匹配的元素为止（线性查找）。然而，如果数组是有序的，就可以使用**二分查找**(binary search)：查看中间的元素是否大于要查找的元素。如果是，就在数组的前半部分继续查找；否则，在后半部分继续查找。这样就可以将问题规模缩减一半，并以同样的方式继续下去。例如，如果数组有1000个元素，二分查找最多需要10步，而线性查找平均需要500步。

`Collections`类的`binarySearch()`方法实现了这个算法。注意，集合必须是有序的，否则算法会返回错误的答案。如果返回一个非负值，则表示匹配对象的索引。如果返回负值，则表示没有匹配的元素。不过，可以利用返回值来计算应该插入元素的位置（如果返回`i`，则插入位置是`-i - 1`）。例如：

```java
int i = Collections.binarySearch(c, element);
if (i < 0)
    c.add(-i - 1, element);
```

将把元素插入正确的位置。

只有采用随机访问，二分查找才有意义。如果提供一个链表，`binarySearch()`会退化为线性查找。

### 9.6.4 简单算法
`Collections`类包含几个简单但很有用的算法，例如前面介绍的`max()`。
* `min(c)` 返回集合中最小的元素
* `max(c)` 返回集合中最大的元素
* `copy(to, from)` 将列表`from`的所有元素复制到列表`to`，目标列表必须至少与原列表一样长
* `fill(l, value)` 将列表的所有位置填充为`value`
* `addAll(c, values...)` 将所有值添加到集合中，如果集合改变了则返回`true`
* `replaceAll(l, oldValue, newValue)` 将列表中所有`oldValue`替换为`newValue`
* `indexOfSubList(l, s)`, `lastIndexOfSubList(l, s)` 返回`l`中第一个或最后一个等于`s`的子列表的索引，如果不存在则返回-1
* `swap(l, i, j)` 交换列表中的两个元素
* `reverse(l)` 反转列表中元素的顺序
* `rotate(l, d)` 旋转列表中的元素，将索引`i`的元素移动到位置`(i + d) % l.size()`
* `frequency(c, value)` 返回集合中`value`的出现次数
* `disjoint(c1, c2)` 如果两个集合没有公共元素则返回`true`

与简单的循环相比，这些算法可以让代码更易读。例如：

```java
for (int i = 0; i < words.size(); i++)
    if (words.get(i).equals("C++")) words.set(i, "Java");
```

将这个循环与以下调用比较

```java
Collections.replaceAll(words, "C++", "Java");
```

看到这个方法调用时，马上就能知道代码要做什么。

默认方法`Collection.removeIf()`和`List.replaceAll()`需要提供一个lambda表达式来测试或转换元素。例如，下面的代码删除所有短单词，并将其余单词改为小写：

```java
words.removeIf(w -> w.length() <= 3);
words.replaceAll(String::toLowerCase);
```

### 9.6.5 批操作
`Collection`接口中有几个“成批”(in bulk)操作元素的方法：
* `c1.removeAll(c2)` 从`c1`中删除所有在`c2`中出现的元素（差集）
* `c1.retainAll(c2)` 从`c1`中删除所有未在`c2`中出现的元素（交集）
* `c1.addAll(c2)` 将`c2`中的所有元素添加到`c1`（并集）

也可以对视图应用批操作。例如，假设有一个ID到员工的映射，另外有一个包含要终止聘用关系的所有员工ID的集合：

```java
Map<String, Employee> staffMap = ...;
Set<String> terminatedIDs = ...;
```

从键集视图中删除这些ID，键和相关联的员工会自动从映射中删除：

```java
staffMap.keySet().removeAll(terminatedIDs);
```

通过使用子范围视图，可以将批操作限制在子列表或子集（类似于Python的列表切片）。例如，将一个列表的前10个元素添加到另一个容器：

```java
relocated.addAll(staff.subList(0, 10));
```

也可以对子范围执行修改操作：

```java
staff.subList(0, 10).clear();
```

### 9.6.6 集合与数组的转换
有时需要在数组和集合之间进行转换。

要把数组转换为集合，可以使用`List.of()`方法：

```java
String[] names = ...;
List<String> staff = List.of(names);
```

要把集合转换为数组，可以使用`toArray()`方法：

```java
String[] values = staff.toArray(String[]::new);
```

注释：在Java 11之前，必须使用另一种形式的`toArray()`方法，传递一个正确类型的数组（见8.6.6节结尾）：

```java
String[] values = staff.toArray(new String[0]);
```

或者

```java
String[] values = staff.toArray(new String[staff.size()]);
```

### 9.6.7 编写自己的算法
如果编写自己的算法（或者任何接受集合参数的方法），应该尽可能使用接口而不是具体的实现。例如，假设你想处理元素。当然，可以像这样实现一个方法：

```java
public void processItems(ArrayList<Item> items) {
    for (Item item : items)
        // do something with item
}
```

但是，这样会限制方法的调用者必须提供一个`ArrayList`。最好接受一个更加通用的集合。

要问问自己：完成这项工作的最通用的集合接口是什么？如果关心顺序，就应该接受`List`。如果顺序不重要，就可以接受`Collection`：

```java
public void processItems(Collection<Item> items) {
    for (Item item : items)
        // do something with item
}
```

现在，可以用`ArrayList`或`LinkedList`，甚至`List.of()`方法包装的数组来调用这个方法。

提示：在这种情况下甚至可以做得更好，可以接受一个`Iterable<Item>`。

反过来，如果你的方法返回一个集合，也应该使用接口。例如：

```java
public ArrayList<Item> lookupItems(...) {
    var result = new ArrayList<Item>();
    ...
    return result;
}
```

这个方法承诺返回一个`ArrayList`，但调用者并不关心它是什么类型的列表。如果返回类型是`List`，就可以随时增加一个分支，返回其他类型的列表。

## 9.7 遗留集合
自Java首次发布以来，在集合框架出现之前，已经存在许多“遗留”集合类。这些类已经集成到集合框架中，如下图所示。下面几节将简要介绍这些类。

![遗留集合类](/assets/images/java-note-v1ch09-collections/遗留集合类.png)

### 9.7.1 Hashtable类
`Hashtable`类与`HashMap`类的作用一样，接口也基本相同。和`Vector`一样，`Hashtable`的方法也是同步的。如果不需要与遗留代码的兼容性，就应该使用`HashMap`。如果需要并发访问，则使用`ConcurrentHashMap`，参见第12章。

### 9.7.2 枚举
遗留集合使用`Enumeration`接口遍历元素。`Enumeration`接口有两个方法：`hasMoreElements()`和`nextElement()`。这两个方法完全类似于`Iterator`接口的`hasNext()`和`next()`方法。

如果发现遗留的类使用了这个接口，可以使用`Collections.list()`将元素收集到一个`ArrayList`中。例如：

```java
ArrayList<String> loggerNames = Collections.list(LogManager.getLoggerNames());
```

从Java 9起，可以把枚举转换为迭代器：

```java
LogManager.getLoggerNames().asIterator().forEachRemaining(n -> { ... });
```

有时还会遇到接受枚举参数的遗留方法。静态方法`Collections.enumeration()`生成一个枚举集合中元素的枚举对象。例如：

```java
List<InputStream> streams = ...;
var in = new SequenceInputStream(Collections.enumeration(streams));
    // the SequenceInputStream constructor expects an enumeration
```

### 9.7.3 属性映射
**属性映射**(property map)是一种特殊类型的映射结构。它有下面3个特性：
* 键和值都是字符串。
* 可以很容易地保存到文件以及从文件加载。
* 有一个二级表（同类型的字段`defaults`）存放默认值。

实现属性映射的类是`Properties`。属性映射对于指定程序的配置选项很有用。例如：

```java
var settings = new Properties();
settings.setProperty("width", "200");
settings.setProperty("title", "Hello World");
```

使用`store()`方法将属性映射保存到文件。第二个参数是包含在文件中的注释。

```java
var out = new FileWriter("program.properties", StandardCharsets.UTF_8);
settings.store(out, "Program Properties");
```

文件program.properties的内容如下：

```properties
#Program Properties
#Sun Dec 08 22:42:06 CST 2024
width=200
title=Hello World
```

使用`load()`方法从文件加载属性映射：

```java
var in = new FileReader("program.properties", StandardCharsets.UTF_8);
settings.load(in);
```

`System.getProperties()`方法会生成一个`Properties`对象来描述系统信息，可以使用`getProperty()`方法读取信息。例如，用户主目录的键为`"user.home"`：

```java
String userDir = System.getProperty("user.home");
```

警告：由于历史原因，`Properties`类实现了`Map<Object, Object>`，因此可以使用`Map`接口的`get()`和`put()`方法。但是这两个方法处理的是`Object`。最好使用处理字符串而不是对象的`getProperty()`和`setProperty()`方法。

要得到虚拟机的Java版本，可以查询`"java.version"`属性，会得到一个诸如 "17.0.1" 的字符串（但是Java 8之前的形式为 "1.8.0" ）。

注释：在Java 9中版本编号发生了变化。如果要解析版本字符串，一定要阅读[JEP 322](https://openjdk.org/jeps/322)。

注：系统属性的完整列表参见`System.getProperties()`方法的API文档。

`Properties`类有两种提供默认值的机制。第一种方法是在`getProperty()`调用中指定默认值：

```java
String title = settings.getProperty("title", "Default title");
```

也可以把所有默认值都放在一个二级映射中，并在主映射的构造器中提供：

```java
var defaultSettings = new Properties();
defaultSettings.setProperty("width", "300");
defaultSettings.setProperty("height", "200");
defaultSettings.setProperty("title", "Default title");
...
var settings = new Properties(defaultSettings);
```

下面的示例程序展示了如何使用属性映射存储和加载程序状态。这个程序使用了第2章的`ImageViewer`程序，可以记住窗口位置、大小和上次加载的文件。也可以手动编辑主目录中的.corejava/ImageViewer.properties文件。

[properties/ImageViewer.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch09/properties/ImageViewer.java)

属性映射是没有层次结构的简单表。通常会用`window.main.color`、`window.main.title`等键名引入一个伪层次结构。但是`Properties`类没有方法来组织这种层次结构。如果要存储复杂的配置信息，就应当改用`Preferences`类，参见第10章。

### 9.7.4 栈
`Stack`类包含`push()`和`pop()`方法。但是，`Stack`类扩展了`Vector`类，甚至可以使用并非栈操作的`insert()`和`remove()`方法。

注：现在可以使用数组列表或双端队列当作栈。

### 9.7.5 位集
`BitSet`类存储一个位序列（它不是数学意义上的集，叫做位向量或位数组可能更合适）。如果需要高效地存储位序列（例如标志(flag)），就可以使用位集。位集将位包装在字节中，因此比`ArrayList<Boolean>`高效得多。

`BitSet`提供了一个方便的接口用于读取、设置或清除各个位。使用这个接口可以避免各种麻烦的位运算（如果将位存储在`int`或`long`变量中这就是必需的）。

对于一个位集`b`：
* `b.get(i)` 获取第i位（等价于`(n & (1 << i)) != 0`）
* `b.set(i)` 设置第i位（等价于`n = n | (1 << i)`）
* `b.clear(i)` 清除第i位（等价于`n = n & ~(1 << i)`）

作为位集应用的一个示例，这里给出一个“埃拉托色尼筛法”(sieve of Eratosthenes)算法的实现，这个算法用于查找素数。这个程序计算2~2000000之间所有素数的个数（共有148933个素数）。

思想是遍历一个包含200万位的位集。首先将所有位打开，然后将已知素数的倍数所对应的位都关闭，最后保留下来的位对应的就是素数。程序清单9-8是用Java实现的这个程序，程序清单9-9是用C++实现的。

[程序清单9-8 sieve/Sieve.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch09/sieve/Sieve.java)

[程序清单9-9 sieve/sieve.cpp](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch09/sieve/sieve.cpp)
