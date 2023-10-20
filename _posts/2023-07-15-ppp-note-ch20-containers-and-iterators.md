---
title: 《C++程序设计原理与实践》笔记 第20章 容器和迭代器
date: 2023-07-15 17:19:56 +0800
categories: [C/C++, PPP]
tags: [cpp, stl, container, sequence, iterator, linked list]
---
本章和下一章将介绍STL，即C++标准库的容器和算法部分。关键概念序列和迭代器用于将容器（数据）和算法（处理）联系在一起。

## 20.1 存储和处理数据
首先考虑一个简单的例子：Jack和Jill各自在测量车速，并记录为浮点值。Jack将测量值存储在数组中，而Jill存储在`vector`中。现在我们想在程序中使用他们的数据，应该怎么做？

我们可以让Jack和Jill的程序将结果写到文件中，然后在我们的程序中读取。这样我们将与他们所选择的数据结构完全隔离。如果决定这样做，我们可以使用第10~11章中的技术读取输入，使用`vector<double>`进行计算（如下图所示）。

![从文件读取数据](/assets/images/ppp-note-ch20-containers-and-iterators/从文件读取数据.png)

但是，如果我们的任务不适合使用文件呢？假设我们每秒调用一次Jack和Jill的函数来获得要处理的数据：

```cpp
// Jack puts doubles into an array and returns the number of elements in *count
double* get_from_jack(int* count);

// Jill fills the vector
vector<double>* get_from_jill();

void fct() {
    int jack_count = 0;
    double* jack_data = get_from_jack(&jack_count);
    vector<double>* jill_data = get_from_jill();
    // ... process ...
    delete[] jack_data;
    delete jill_data;
}
```

![调用函数获取数据](/assets/images/ppp-note-ch20-containers-and-iterators/调用函数获取数据.png)

这里假设数据保存在自由存储中，我们使用完后应该删除。另一个假设是我们不能重写Jack和Jill的代码。

### 20.1.1 使用数据
显然，这是一个简化的例子，但它与很多现实世界中的问题没有什么不同。问题的关键在于，我们不能控制“数据提供者”存储数据的方式。

我们需要如何处理这些数据？存在无限种可能，但首先考虑一个非常简单的任务：找出每组数据中的最大值。我们可以将以下代码插入`fct()`函数中的 "... process ..." 注释部分：

[Jack-and-Jill v1](https://github.com/ZZy979/PPP-code/blob/main/ch20/jack_and_jill_v1.cpp)

注意这种丑陋的写法：`(*jill_data)[i]`，其中的圆括号是必须的，因为`[]`的优先级比`*`高。

### 20.1.2 一般化代码
我们希望使用统一的方式来访问和操作数据。下面以Jack和Jill的代码为例，讨论如何让我们的代码更加抽象和统一。

显然，我们对Jack和Jill的数据的处理方法非常相似。但是，有一些令人讨厌的差异：`jack_count`和`jill_data->size()`以及`jack_data[i]`和`(*jill_data)[i]`。

如何编写一个可以同时处理Jack和Jill的数据的函数？出于通用性的原因（这将在接下来的两章中变得清楚），我们选择了基于指针的解决方案：

[Jack-and-Jill v2](https://github.com/ZZy979/PPP-code/blob/main/ch20/jack_and_jill_v2.cpp)

于是，我们可以这样写：

```cpp
double* jack_high = high(jack_data, jack_data + jack_count);
vector<double>& v = *jill_data;
double* jill_high = high(&v[0], &v[0] + v.size());
cout << "Jill's max: " << *jill_high
     << "; Jack's max: " << *jack_high;
```

这段代码就简洁多了，我们没有引入那么多变量，也没有重复编写循环。

注：函数`high()`的思想与18.7节中回文的例子是类似的，使用一对指针/索引来表示元素序列/区间`[first, last)`。

【试一试】这段程序中有两个潜在的严重错误：
* `high()`默认所有数据都非负（将`h`初始化为-1），如果存在负值则返回结果是错误的。
* 指针`high`未初始化，当序列为空（即`first == last`）或数据全部小于-1时，`high()`返回的是一个野指针。另外，`high()`默认`last >= first`，否则循环永远不会结束。

函数`high()`的局限性在于它只能解决一个特定的问题：
* 只能处理数组（能处理`vector`是依赖于其元素也存储在数组中），不能处理`list`和`map`等。
* 只能处理`double`类型的元素。

下面我们探讨如何在更一般化的数据集合上进行计算。

## 20.2 STL思想
C++标准库提供了一个将数据作为元素序列处理的框架，叫做**标准模板库**(standard template library, STL)。STL是ISO C++标准库的一部分，提供了容器（例如`vector`、`list`和`map`）和通用算法（例如`sort()`、`find()`和`accumulate()`）。其他标准库特性（例如`ostream`和C风格字符串函数）并不属于STL。

计算包括两个主要方面：算法和数据。

我们希望编写的代码能够简单、高效地处理计算任务。而对于程序员来说，问题在于：
* **数据类型**有无限种变化。
* **存储数据元素集合的方式**多得令人眼花缭乱。
* 我们希望对数据集合执行的**计算任务**各种各样。

因此，我们希望**一般化/通用化**(generalize)代码来应对这些变化。我们的目标是无需考虑各种容器之间的差别、各种访问元素的方法之间的差别以及各种数据类型之间的差别（注：前两点通过迭代器实现，第三点通过模板实现）。

## 20.3 序列和迭代器
STL的核心概念是**序列**(sequence)。从STL的角度来看，数据集合就是一个序列。序列有一个**开始**(beginning)和一个**结尾**(end)，由一对迭代器指定，如下图所示。**迭代器**(iterator)是标识序列中元素位置的对象。

![序列](/assets/images/ppp-note-ch20-containers-and-iterators/序列.png)

其中，`begin`和`end`是迭代器，它们标识了序列的开始和结尾。**STL序列是“半开”的**，即`begin`迭代器标识的元素是序列的一部分，而`end`迭代器指向序列结尾的后面一个位置。在数学上，这种序列通常表示为左闭右开区间`[begin, end)`。从一个元素到下一个元素的箭头表示：如果我们有指向一个元素的迭代器，那么可以得到指向下一个元素的迭代器。

迭代器究竟是什么？迭代器是一个非常抽象的概念：
* 迭代器指向序列中的一个元素（或最后一个元素的下一个位置）。
* 可以使用`==`和`!=`来比较两个迭代器。
* 可以使用`*`来引用迭代器所指向的元素的值。
* 可以使用`++`得到下一个元素的迭代器。

注：
* “迭代器”本质上是一种抽象接口/概念，以上就是在描述迭代器支持的操作。
* 序列只是对数据集合的一种抽象，并不关心元素是如何存储的。例如，`vector`的元素是连续存储的，而`list`的元素不是连续的，但二者都可以通过迭代器抽象为序列。

例如，如果`p`和`q`是指向同一个序列中元素的迭代器：

| 操作 | 含义 |
| --- | --- |
| `p == q` | `p`和`q`指向同一个元素 |
| `p != q` | `p`和`q`指向不同元素 |
| `*p` | `p`指向的元素 |
| `++p` | 使`p`指向下一个元素 |

显然，迭代器的思想与指针是相关的（见17.4节）。实际上，**数组元素的指针就是数组的迭代器**。事实证明，将迭代器作为一种抽象概念而不是特定类型可以带来极大的灵活性和通用性。

注：STL容器都定义了各自的迭代器类型，并且提供了成员函数`begin()`和`end()`分别返回开始和结束迭代器。例如，`vector<T>`的迭代器是`vector<T>::iterator`，4.6.3节就使用过`sort(temps.begin(), temps.end())`对向量`temp`进行排序。

[试一试](https://github.com/ZZy979/PPP-code/blob/main/ch20/copy.cpp)

迭代器可以将算法与数据分离——算法代码的作者只知道迭代器，而不知道迭代器是如何访问数据的；数据提供者（容器代码的作者）只需提供迭代器，而不是将数据存储的细节暴露给用户。正如STL的作者Alex Stepanov所说：“STL算法和容器之所以能很好地协同工作，是因为它们对彼此一无所知。”但是，它们都知道由一对迭代器所定义的序列。

![算法-迭代器-容器](/assets/images/ppp-note-ch20-containers-and-iterators/算法-迭代器-容器.png)

STL可能是目前最著名、使用最广泛的泛型编程的例子。

注：
* 通过使用迭代器和模板，就可以解决20.2节提到的三个问题。
* STL容器库：<https://en.cppreference.com/w/cpp/container>
* STL算法库：<https://en.cppreference.com/w/cpp/algorithm>

### 20.3.1 回到示例
下面使用STL的序列概念来表达“查找最大值”问题：

[Jack-and-Jill v3](https://github.com/ZZy979/PPP-code/blob/main/ch20/jack_and_jill_v3.cpp)

注意，我们消除了函数`high()`中表示当前遇到的最大值的局部变量`h`。当我们不知道序列元素的实际类型时，使用-1来初始化太随意了（可能与元素类型不兼容，导致编译错误）。另外，虽然在这个例子中不会存在负的速度，但是像-1这样的“魔数”是不利于代码维护的。

这个“一般化”的`high()`可用于任何可以使用`<`比较的元素类型，并且要求模板参数`Iterator`必须可拷贝并支持`!=`、`++`和`*`运算符（注：这就是19.3.2节所说的“满足特定的语法和语义要求”）。

模板函数`high()`可用于任何由一对迭代器定义的序列。在上面的代码中，第一次调用的模板参数是`double*`（指针作为数组的迭代器），第二次是`vector<double>::iterator`（使用`vector`自定义的迭代器，本质上仍然是指针）。

第二次调用也可以使用20.1.2节中的方式`high(&v[0], &v[0] + v.size())`，即使用指针作为迭代器，模板参数为`double*`，这依赖于`vector`将元素存储在数组中。如果使用这种方法，则`main()`函数的代码与上一个版本没有任何区别，但模板版本的`high()`具有更好的通用性。

【试一试】这段程序仍然遗留了一个严重错误：当序列为空（即`first == last`）时，`high()`返回的迭代器等于`last`（最后一个元素之后的位置），对其进行解引用会引起问题。

## 20.4 链表
在20.3节的序列概念示意图中，元素之间的箭头仅仅表示其逻辑位置是相邻的，与实际内存位置无关。

对于`vector`，其元素在内存中是连续排列的，可视化示意图如下：

![vector内存示意图](/assets/images/ppp-note-ch20-containers-and-iterators/vector内存示意图.png)

本质上，下标0与迭代器`v.begin()`都标识第一个元素，下标`size()`与迭代器`v.end()`都标识最后一个元素的下一个位置。因此，`vector`的迭代器可以简单地使用下标或指针实现（见20.5节）。

对于链表（17.9.3节），其元素在内存中不是连续的，而是通过指针连接起来。双向链表的示意图如下（与STL序列示意图更接近）：

![双向链表示意图](/assets/images/ppp-note-ch20-containers-and-iterators/双向链表示意图.png)

图中元素之间的箭头通常实现为指针（即[Link结构体](https://github.com/ZZy979/PPP-code/blob/main/ch17/doubly_linked_list.h)中的`prev`和`succ`）。链表的**节点**(node)/**链接**(link)由元素和一个或两个指针组成。本节将实现一个双向链表，即C++标准库提供的`list`（17.9.3节的实现本质上只是链表节点，并不是完整的链表类）。

上述示意图可以用代码表达为：

```cpp
// doubly-linked list node
template<class Elem>
struct Link {
    Link* prev;     // previous link
    Link* succ;     // successor (next) link
    Elem val;       // the value
};

// doubly-linked list
template<class Elem>
class list {
public:
    // ...
private:
    Link<Elem>* first;
    Link<Elem>* last;   // one beyond the last link
};
```

链表的关键性质是：**可以在不影响已有元素的前提下插入和删除元素**（只需要调整相邻节点的指针，因此插入和删除操作的时间复杂度为O(1)；相反，`vector`的插入和删除操作需要移动元素，时间复杂度为O(n)）。

当你尝试思考链表时，我们强烈建议你画一些图示来可视化你正在考虑的操作（如17.9.3节）。

### 20.4.1 链表操作
对于链表我们需要什么操作？
* 构造函数、析构函数、赋值、`size()`
* 插入和删除
* 迭代器：用于引用元素和遍历链表

```cpp
// doubly-linked list
template<class Elem>
class list {
public:
    class iterator;     // member type: iterator

    iterator begin();   // iterator to first element
    iterator end();     // iterator to one beyond last element

    iterator insert(iterator p, const Elem& v); // insert v into list after p
    iterator erase(iterator p);                 // remove p from the list

    void push_back(const Elem& v);      // insert v at end
    void push_front(const Elem& v);     // insert v at front
    void pop_front();   // remove the first element
    void pop_back();    // remove the last element

    Elem& front();  // the first element
    Elem& back();   // the last element

    // ...

private:
    Link<Elem>* first;
    Link<Elem>* last;   // one beyond the last link
};
```

就像我们的`vector`不是完整的标准库`vector`一样，这里的`list`也不是标准库`list`的完整定义。其目的是帮助你理解链表是什么，如何实现，以及如何使用其关键特性。

迭代器是STL `list`定义的核心，用于访问元素、标识插入/删除元素的位置以及遍历链表。STL容器将迭代器作为成员类型，并命名为`iterator`，例如`list<T>::iterator`、`vector<T>::iterator`、`map<K,V>::iterator`等。

`list`没有下标操作（因为它无法像`vector`一样通过下标访问元素，即**随机访问**(random access)）。如果需要，可以使用迭代器来实现。

### 20.4.2 迭代
`list`迭代器必须提供`*`、`++`、`==`和`!=`操作。由于`list`是双向链表，还提供了`--`操作用于反向迭代。

```cpp
template<class Elem>
class list<Elem>::iterator {
public:
    explicit iterator(Link<Elem>* p) :curr(p) {}
    iterator& operator++() { curr = curr->succ; return *this; }  // forward
    iterator& operator--() { curr = curr->prev; return *this; }  // backward
    Elem& operator*() { return curr->val; }  // get value (dereference)
    bool operator==(const iterator& b) const { return curr == b.curr; }
    bool operator!=(const iterator& b) const { return curr != b.curr; }

private:
    Link<Elem>* curr;  // current link
};
```

这些函数十分简明且高效：没有循环和复杂的表达式。这里的`list`迭代器就是指向节点的指针，`*`、`++`和`--`运算符分别对应其指向节点的`val`、`succ`和`prev`成员。

再次回顾20.3.1节中的`high()`，我们可以将其用于`list`：

[查找链表中的最大值](https://github.com/ZZy979/PPP-code/blob/main/ch20/exec20-12.cpp)

其中，`high()`的模板参数是`list<int>::iterator`。

现在，是时候回答20.3.1节中“试一试”提出的问题：对于函数`high()`，如果序列为空（即`first == last`），则其返回的迭代器等于`last`（最后一个元素的下一个位置），对其进行解引用是一个灾难性错误（结果取决于元素类型和迭代器实现，对于`vector`将导致越界访问，对于`list`可能导致空指针访问）。

解决方法是：**通过比较开始和结尾来判断序列是否为空**，如下图所示。

![空序列](/assets/images/ppp-note-ch20-containers-and-iterators/空序列.png)

这是使`end`指向最后一个元素的下一个位置而不是最后一个元素的深层次原因：**空序列不是特例**。

在我们的例子中可以这样使用：

```cpp
auto p = high(lst.begin(), lst.end());
if (p == lst.end())     // did we reach the end?
    cout << "The list is empty";
else
    cout << "the highest value is " << *p << '\n';
```

STL查找类算法普遍采用了这种方式：**返回结尾迭代器表示“未找到”**。

因为标准库提供了链表，我们在这里不再深入讨论其实现（见习题12）。

## 20.5 再次一般化vector
标准库`vector`有`iterator`成员类型以及`begin()`和`end()`成员函数（就像`list`一样）。然而，在第19章中我们并没有为我们的`vector`提供这些。

[简单向量v3 - 增加迭代器](https://github.com/ZZy979/PPP-code/blob/b52d27d494931937ba42d3bedc60d969ce9bb6f6/ch19/simple_vector.h)

其中的`using`声明为类型创建了一个别名（详见[Type alias](https://en.cppreference.com/w/cpp/language/type_alias)）。也就是说，对于我们的`vector`，我们选择使用指针作为迭代器，`iterator`就是`T*`的别名/同义词（因此迭代器的`*`、`++`、`==`等操作就是普通指针的运算符）。现在，对于`vector`对象`v`，可以这样写：

```cpp
vector<int>::iterator p = find(v.begin(), v.end(), 32);
for (vector<int>::size_type i = 0; i < v.size(); ++i) cout << v[i] << '\n';
```

这里的关键点在于，在编写这段代码时，我们实际上不需要知道`iterator`和`size_type`的实际类型。在大多数C++实现中，标准库`vector`的迭代器并不是普通指针，但上面的代码仍然能够正常运行。

使用`using`定义类型别名是C++11引入的语法，在C和C++11之前使用`typedef`。

注意：将`size()`改为无符号类型后要小心倒序遍历时溢出导致的无限循环：

```cpp
for (auto i = v.size() - 1; i >= 0; --i)  // infinite loop!
    cout << v[i] << '\n';
```

因为无符号的0减1等于2<sup>32</sup>-1（即4294967295）。

### 20.5.1 容器遍历
使用`size()`和下标，我们可以从头到尾遍历`vector`元素。例如：

```cpp
void print1(const vector<double>& v) {
    for (int i = 0; i < v.size(); ++i)
        cout << v[i] << '\n';
}
```

这不能用于`list`，因为`list`没有提供下标操作。然而，我们可以使用更简单的range-`for`循环来遍历`vector`和`list`。例如：

```cpp
void print2(const vector<double>& v, const list<double>& lst) {
    for (double x : v)
        cout << x << '\n';

    for (double x : lst)
        cout << x << '\n';
}
```

这段代码对于标准库容器和我们自己的`vector`和`list`都能正常运行。实际上，range-`for`循环仅仅是使用迭代器遍历序列的“语法糖”。对于容器`v`：

```cpp
for (T x : v)
    // ...
```

大致等价于

```cpp
for (auto begin = v.begin(), end = v.end(); begin != end; ++begin) {
    T x = *begin;
    // ...
}
```

由于我们为`vector`和`list`定义了迭代器以及`begin()`和`end()`，因此“意外地”支持了使用range-`for`遍历。

详见[range-for](https://en.cppreference.com/w/cpp/language/range-for)。

### 20.5.2 auto
当我们使用迭代器时，类型名称写起来很繁琐。例如：

```cpp
template<class T>
void user(vector<T>& v, list<T>& lst) {
    for (vector<T>::iterator p = v.begin(); p != v.end(); ++p) cout << *p << '\n';
    list<T>::iterator q = find(lst.begin(), lst.end(), T{42});
}
```

幸运的是，我们不必这样写：我们可以将变量类型声明为`auto`，表示其类型由初始值推导出来（类似于模板参数推导）。例如：

```cpp
template<class T>
void user(vector<T>& v, list<T>& lst) {
    for (auto p = v.begin(); p != v.end(); ++p) cout << *p << '\n';
    auto q = find(lst.begin(), lst.end(), T{42});
}
```

其中，`p`是`vector<T>::iterator`，`q`是`list<T>::iterator`。

我们可以在任何包含初始值的声明中使用`auto`。例如：

```cpp
auto x = 123;  // x is an int
auto c = 'y';  // c is a char
auto& r = x;   // r is an int&
auto y = r;    // y is an int (references are implicitly dereferenced)
```

注意，字符串常量的类型是`const char*`：

```cpp
auto s1 = "San Antonio";       // s1 is a const char* (Surprise!?)
string s2 = "Fredericksburg";  // s2 is a string
```

`auto`的一个常见用法是指定range-`for`循环中的循环变量。例如：

```cpp
template<class C>
void print3(const C& cont) {
    for (const auto& x : cont)
        cout << x << '\n';
}
```

## 20.6 示例：简单文本编辑器
链表最重要的性质是可以在不移动其他元素的情况下添加或删除元素（只需几次指针赋值操作，例如17.9.3节`insert()`操作的示意图）。下面我们通过一个简单的例子来说明这一点。考虑如何在文本编辑器中表示文本文件的字符，表示方式应当使得对文档的操作简单且高效。我们所选择的表示方式必须支持5种操作：
* 从输入的字节流创建
* 插入一个或多个字符
* 删除一个或多个字符
* 查找字符串
* 生成字节流从而输出到文件或屏幕

注：本节要实现的“文本编辑器”不是下面这样，而是一个能够对多行文本进行操作的**类**。这个类可以作为真实文本编辑器应用的后端代码，只是没有GUI界面。

![记事本](/assets/images/ppp-note-ch20-containers-and-iterators/记事本.png)

最简单的表示方式是`vector<char>`。然而，要插入或删除一个字符必须移动后面的所有字符，这对于超大的文档是不可接受的。我们考虑文档“分解”成多个“块”，使得改变一部分不影响其他部分（这恰好符合链表的性质）。因此，我们将文档表示为文本行的链表`list<Line>`，其中`Line`是`vector<char>`。例如，文档

```
This is the start of a very long document.
There are lots of ...
```

在内存中的表示如下：

![文档在内存中的表示](/assets/images/ppp-note-ch20-containers-and-iterators/文档在内存中的表示.png)

现在，插入字符只需要移动相应行中的字符。如果需要插入新行，则不需要移动任何字符。例如，在第一行之后插入 "This is a new line." ，得到

```
This is the start of a very long document.
This is a new line.
There are lots of ...
```

我们只需要在链表中插入一个“节点”：

![插入新行](/assets/images/ppp-note-ch20-containers-and-iterators/插入新行.png)

### 20.6.1 行
我们依靠换行符(`'\n'`)来划分“行”，并将文档表示为一个`Document`类的对象：

[简单文本编辑器](https://github.com/ZZy979/PPP-code/blob/main/ch20/simple_text_editor.h)

每个`Document`开始都只有一个空行：`Document`的构造函数会创建一个空行并添加到链表中（注：这是为了避免链表为空，方便迭代器的实现）。

`>>`运算符读取输入并划分成行。

`vector`和`list`都有一个成员函数`back()`，返回最后一个元素的引用（即`v.back()`等价于`v[v.size() - 1]`）。使用`back()`时一定要确保容器不为空，这也是我们为`Document`添加空行的原因。注意对输入的每个字符都会存储，包括换行符，这会极大地简化输出。

注：
* `Document`类的不变式：
    * 始终以一个空行结尾——（1）构造函数会自动添加一个空行；（2）`>>`运算符在读取完输入后会自动添加一个空行。这是为了标识文档结尾，用于实现`end()`迭代器（类似于C风格字符串始终以空字符结尾）。
    * 除最后一行和结尾空行外，每行都以换行符结尾——`>>`运算符读取到`'\n'`时插入一个新行。
* 以空行结尾会存在一个问题：如果两次调用`>>`运算符，第一次的输入结尾不是换行符，那么最后一行和第二次输入的第一行会被拆分到两个不同的“行”（链表元素）中。这不影响输出，但可能导致`erase_line()`删除不完整的行（见下一节）。

### 20.6.2 迭代
显然，我们可以使用`list<Line>::iterator`来迭代行的链表。但是，如果我们想要逐个访问字符而不考虑换行符，就需要专门为`Document`提供一个迭代器`Text_iterator`：

[Text_iterator](https://github.com/ZZy979/PPP-code/blob/main/ch20/simple_text_editor.h)

`Text_iterator`的思想是：用行号和列号标识一个字符，迭代时将整个文档视为所有行拼接起来的一个字符序列（实际上就是划分行前的原始字符序列），如下图所示（其中数字表示字符的迭代顺序）。

![Text_iterator](/assets/images/ppp-note-ch20-containers-and-iterators/Text_iterator.png)

为了使用`Text_iterator`，我们需要为`Document`定义`begin()`和`end()`函数。

现在，我们可以这样迭代文档的字符：

```cpp
void print(Document& d) {
    for (auto c : d) cout << c;
}
```

将文档表示为行的链表（内存角度）/字符序列（迭代器角度）对于很多操作都是有用的。例如，函数`erase_line()`删除文档的第n行：

[erase_line()](https://github.com/ZZy979/PPP-code/blob/main/ch20/simple_text_editor.cpp)

标准库函数`advance(p, n)`将迭代器`p`向前（或向后）移动n个元素；`next(p, n)`与`advance(p, n)`类似，但返回移动后的迭代器。

对于双向迭代器（可以向前和向后移动），比如`list<T>::iterator`，`advance()`的参数为正将向前移动，参数为负将向后移动。对于随机访问迭代器（支持与整数加减），比如`vector<T>::iterator`，`advance()`将直接移动到指定位置（等价于`p += n`），而不是使用`++`移动n次。

对于用户来说，查找可能是最直观的一种迭代。下面实现在`Document`中查找一个字符串。我们使用一种简单的、但不是最优的算法：
* 在文档中查找字符串的第一个字符。
* 判断后续字符是都与查找的字符串匹配。
* 如果是，则结束；否则，继续查找字符串的第一个字符。

为了通用性，我们采用STL约定，把待搜索的文本定义为一对迭代器表示的序列。如果在文档中找到了字符串，则返回其第一个字符的迭代器；否则返回序列结尾的迭代器。

[find_txt()](https://github.com/ZZy979/PPP-code/blob/main/ch20/simple_text_editor.cpp)

**返回序列结尾迭代器来表示“未找到”是一个重要的STL约定。** 我们可以这样使用`find_txt()`：

```cpp
auto p = find_txt(my_doc.begin(), my_doc.end(), "secret\nhomestead");
if (p == my_doc.end())
    cout << "not found";
else {
    // do something
}
```

注意：自定义的迭代器类型可以用于range-`for`循环，但不能直接用于STL算法，还需要指定**迭代器类别**(iterator category)，否则函数`find_txt()`中调用STL算法`find()`会报错 "In template: no matching function for call to '__iterator_category'" 。迭代器类别用于标准库内部选择最优的算法（例如`advance()`对于`list`和`vector`迭代器的不同实现），见20.10.1节。为自定义迭代器指定类别有两种方法：

（1）（使用`using`或`typedef`）定义以下5个成员类型（缺一不可）：

| 成员类型 | 含义 |
| --- | --- |
| `iterator_category` | 迭代器类别 |
| `value_type` | 元素类型 |
| `difference_type` | 表示迭代器之间距离的类型（通常为`ptrdiff_t`） |
| `pointer` | 元素指针类型（通常为`value_type*`） |
| `reference` | 元素引用类型（通常为`value_type&`） |

其中，迭代器类别从标准库头文件\<iterator\>中定义的[迭代器标签](https://en.cppreference.com/w/cpp/iterator/iterator_tags)中选择。例如：

```cpp
#include <iterator>

class Text_iterator {
public:
    using iterator_category = std::forward_iterator_tag;
    using value_type = char;
    using difference_type = ptrdiff_t;
    using pointer = char*;
    using reference = char&;
    // ...
};
```

由于`Text_iterator`只提供了`*`和`++`操作，因此属于[前向迭代器](https://en.cppreference.com/w/cpp/named_req/ForwardIterator)(forward iterator)。

（2）继承`std::iterator`，该类型只是提供了以上5个成员类型的定义，通过模板参数指定（后3个类型可省略）。例如：

```cpp
#include <iterator>

class Text_iterator : public std::iterator<std::forward_iterator_tag, char> {
public:
    // ...
};
```

这种方法更简单（毕竟谁会需要自定义元素指针和引用类型呢），然而这个类在C++17中弃用了，因为当自定义迭代器是类模板时无法使用`std::iterator`提供的`value_type`等类型（参考[Why is std::iterator deprecated?](https://stackoverflow.com/questions/43268146/why-is-stditerator-deprecated)、[Deprecate std::iterator](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0174r1.html#2.1)）。

## 20.7 vector、list和string
为什么对行使用`list`而对字符使用`vector`呢？为什么不使用`string`来存储一行？

现在我们已经知道4种存储字符序列的方式：
* `char[]`（字符数组）
* `vector<char>`
* `string`
* `list<char>`

这些类型的逻辑特性如下：

| 容器 | `T[]` | `vector<T>` | `string` | `list<T>` |
| --- | --- | --- | --- | --- |
| 特性 | 可转换为指针，传递给C风格函数 | 基本可以做任何事 | 提供文本处理操作（例如`+`和`+=`） | 提供所有常用操作（除了下标），支持高效插入删除 |
| 大小（`size()`） | × | √ | √ | √ |
| 迭代器（`begin()`和`end()`） | ×（可用指针实现） | √ | √ | √ |
| 下标（`[]`） | √ | √ | √ | × |
| 边界检查（`at()`） | × | √ | √ | × |
| 元素连续存储 | √ | √ | √ | × |
| 可扩展（`push_back()`等） | × | √ | √ | √（双端） |
| 增删元素（`insert()`和`erase()`） | × | √（低效） | √（低效） | √（高效） |
| 比较运算符（`==`、`!=`等） | 比较指针 | 比较元素 | 比较元素 | 比较元素 |

正如我们所看到的（17.2节、18.6节），当我们处理底层内存或者和C程序交互时，数组是有用且必需的。除此之外，**vector是首选**，因为它更容易使用、更灵活、更安全。

`list`占用的内存大约是另外三种容器的3倍（因为多了两个指针成员）。在一台个人计算机上，`list<char>`的每个元素占12字节，而`vector<char>`只占1字节。

与`vector`相比，`string`提供了字符串操作（例如拼接、读取空白符分隔的单词等），与C风格字符串交互等功能，但元素类型只能是字符。总而言之，只有需要字符串操作时才考虑使用`string`，否则使用`vector`。因此在文本编辑器的例子中，我们选择`vector`而不是`string`来表示行。

### 20.7.1 insert和erase
如20.4节指出，`vector`的`insert()`和`erase()`操作需要移动元素，而`push_back()`可能导致扩容（见19.2.4节）。因此在执行这些操作时，不要保存元素的迭代器或指针：如果元素移动了，迭代器或指针将指向错误的元素，或根本不指向任何元素。这叫做**迭代器失效**(iterator invalidation)，详见[Iterator invalidation](https://en.cppreference.com/w/cpp/container#Iterator_invalidation)（失效的迭代器相当于野指针，对其进行解引用是严重的错误）。

下面比较`vector`和`list`的`insert()`和`erase()`操作：

```cpp
vector<int> v = {0, 1, 2, 3, 4, 5};
auto p = v.begin();  // take a vector
++p; ++p; ++p;       // points to its 4th element
auto q = p;
++q;                 // points to its 5th element
```

![vector迭代器-初始](/assets/images/ppp-note-ch20-containers-and-iterators/vector迭代器-初始.png)

```cpp
p = v.insert(p, 99);  // p points at the inserted element
```

![vector迭代器-insert()之后](/assets/images/ppp-note-ch20-containers-and-iterators/vector迭代器-insert之后.png)

现在`q`是无效的，因为向量大小增长，元素可能已经被重新分配。如果没有重新分配，则`q`应该指向元素3而不是4，但不要试图利用这一点。

```cpp
p = v.erase(p);  // p points at the element after the erased one
```

![vector迭代器-erase()之后](/assets/images/ppp-note-ch20-containers-and-iterators/vector迭代器-erase之后.png)

也就是说，先`insert()`再`erase()`同一个元素，`vector`会回到开始的状态，但`q`变成无效的了。然而，在这两次操作之间，我们移动了插入点之后的所有元素，并且`v`的所有元素可能都被重新分配了。

作为对比，我们使用`list`完成相同的操作：

```cpp
list<int> l = {0, 1, 2, 3, 4, 5};
auto p = l.begin();  // take a list
++p; ++p; ++p;       // points to its 4th element
auto q = p;
++q;                 // points to its 5th element
```

![list迭代器-初始](/assets/images/ppp-note-ch20-containers-and-iterators/list迭代器-初始.png)

```cpp
p = l.insert(p, 99);  // p points at the inserted element
```

![list迭代器-insert()之后](/assets/images/ppp-note-ch20-containers-and-iterators/list迭代器-insert之后.png)

注意，`p`仍然指向元素4。

```cpp
p = v.erase(p);  // p points at the element after the erased one
```

![list迭代器-erase()之后](/assets/images/ppp-note-ch20-containers-and-iterators/list迭代器-初始.png)

同样，我们又回到的开始状态。然而，`list`与`vector`的不同之处在于，我们没有移动任何元素，并且`q`始终是有效的。

注意：**不要使用迭代器在遍历容器的同时删除元素。** 例如，以下删除`vector`中所有偶数的程序是错误的：

```cpp
vector<int> v = {2, 8, 9, 6, 5, 1, 0, 4, 3, 7};
for (auto it = v.begin(); it != v.end(); ++it)
    if (*it % 2 == 0)
        v.erase(it);

for (int x : v)
    cout << x << ' ';
cout << endl;
```

程序将输出 "8 9 5 1 4 3 7" ，而不是期望的 "9 5 1 3 7" ，这是因为在删除偶数元素时又移动了迭代器，跳过了被删除元素的下一个元素，因此无法删除相邻的偶数。

另外，调用`erase()`之后`it`将失效，继续使用`it`是不正确的。不过由于`vector`的元素是连续存储的，删除的元素被后面的元素代替，因此上面的程序并不会崩溃，只是输出错误的结果。如果换成`list`，则调用`erase()`之后`it`将指向被释放的内存，上面的程序将会崩溃。

正确方法1：使用`erase()`的返回值更新`it`，并且在删除元素时不移动迭代器

```cpp
for (auto it = v.begin(); it != v.end();)
    if (*it % 2 == 0)
        it = v.erase(it);
    else
        ++it;
```

正确方法2：`remove_if()` + `erase()`

```cpp
auto it = remove_if(v.begin(), v.end(), [](int x) { return x % 2 == 0; });
v.erase(it, v.end());
```

正确方法3：C++20引入了`erase_if()`（非成员函数）

```cpp
erase_if(v, [](int x) { return x % 2 == 0; });
```

## 20.8 使我们的vector适配STL
我们在20.5节为`vector`添加迭代器之后，现在只差`insert()`和`erase()`就基本达到`std::vector`的近似了：

[简单向量v3 - 增加insert()和erase()](https://github.com/ZZy979/PPP-code/blob/5846224c91a6a8edcc9612c6e9421fca2936bab8/ch19/simple_vector.h)

注意：
* 在`insert()`中，使用`reserve()`可能造成重新分配，元素会被移动到新的内存区域，原来的迭代器将会失效（可以理解为指向旧的内存），因此必须记录要插入元素位置的索引，而不是迭代器。
* 书中给出的`insert()`代码存在一个bug：无法正确处理`p == end()`的情况，因此需要单独处理，此时等价于`push_back()`。

## 20.9 使内置数组适配STL
如18.6.2节和19.3.5节指出，内置数组的不足之处在于：不知道自己的大小，一不小心就会转换成指针，不能使用赋值拷贝等。其优点是：它们几乎完美地建模了物理内存。

为了两全其美，我们创建了具有数组的优点而没有其不足的`array`容器：

[简单数组 - 增加迭代器](https://github.com/ZZy979/PPP-code/blob/2479b08e7fbdad80a33bdba4530889f683b87c43/ch19/simple_array.h)

例如，`array`可以和20.3.1节中的`high()`一起使用：

```cpp
void f() {
    array<double,6> a = {0.0, 1.1, 2.2, 3.3, 4.4, 5.5};
    auto p = high(a.begin(), a.end());
    cout << "the highest value was " << *p << '\n';
}
```

注意，我们在编写`high()`时并没有考虑`array`。之所以能够对`array`使用`high()`，是因为二者都是按照标准的习惯定义的。

## 20.10 容器概览
STL提供了很多容器：

| 容器 | 描述 | 底层实现 |
| --- | --- | --- |
| `vector` | 向量，动态连续数组 | 动态数组 |
| `array` | 静态连续数组 | 内置数组 |
| `list` | 双向链表 | 节点+指针 |
| `forward_list` | 单向链表 | 节点+指针 |
| `deque` | 双端队列 | 多段连续空间+指针数组 |
| `map` | 映射（键值对集合），按key排序，key是唯一的 | 红黑树 |
| `multimap` | 映射（键值对集合），按key排序，key不是唯一的 | 红黑树 |
| `unordered_map` | 映射（键值对集合），key无序，key是唯一的 | 散列表 |
| `unordered_multimap` | 映射（键值对集合），key无序，key不是唯一的 | 散列表 |
| `set` | 集合，按key排序，key是唯一的 | 红黑树 |
| `multiset` | 集合，按key排序，key不是唯一的 | 红黑树 |
| `unordered_set` | 集合，key无序，key是唯一的 | 散列表 |
| `unordered_multiset` | 集合，key无序，key不是唯一的 | 散列表 |

大部分容器都定义在同名的头文件中，详见附录B以及[Containers library](https://en.cppreference.com/w/cpp/container)。

**容器**(container)是容纳其他对象的对象或类型，即元素的**集合**(collection)。STL将容器建模为序列（见20.3节），这里给出其非正式的定义。STL容器
* 是元素序列`[begin(), end())`。
* 提供了拷贝元素的拷贝操作，拷贝可以通过赋值或拷贝构造函数来完成。
* 将元素类型（通常是模板参数）命名为`value_type`。
* 具有迭代器类型`iterator`和`const_iterator`，并提供`begin()`和`end()`来获取序列首尾迭代器。迭代器提供`*`、`++`（前缀和后缀）、`==`和`!=`操作。双向迭代器还支持`--`，随机访问迭代器还支持`[]`、`+`和`-`（见20.10.1节）。
* 提供`[]`、`size()`、`push_back()`、`pop_back()`、`insert()`、`erase()`、`front()`和`back()`等操作。
* 提供比较元素的比较运算符（`==`、`!=`、`<`、`<=`、`>`和`>=`）。使用字典序实现`<`、`<=`、`>`和`>=`，即从第一个元素开始依次比较。

有些数据类型提供了标准容器所要求的大部分特性，但不是全部。有时称之为“**近似容器**”(almost containers)：

| 近似容器 | 描述 |
| --- | --- |
| `T[]` | 内置数组，没有`size()`等成员函数 |
| `string` | 字符串，只存储字符，提供文本处理操作（例如`+`和`+=`） |
| `valarray` | 数学向量，提供高性能向量运算（例如按元素加减乘除） |

注：
* C++容器与Java和Python容器的大致对应关系如下：

| C++容器 | Java容器 | Python容器 |
| --- | --- | --- |
| `vector` | `ArrayList` | `list` |
| `list` | `LinkedList` | - |
| `deque` | `ArrayDeque` | `collections.deque` |
| `map` | `TreeMap` | - |
| `unordered_map` | `HashMap` | `dict` |
| `set` | `TreeSet` | - |
| `unordered_set` | `HashSet` | `set` |

* 除了容器，STL还提供了一些**容器适配器**(container adaptor)，通过对已有容器进行封装来实现特定的数据结构，可以通过模板参数指定要使用的底层容器：

| 容器适配器 | 描述 | 默认底层容器 |
| --- | --- | --- |
| `stack` | 栈，后进先出(LIFO)数据结构 | `deque` |
| `queue` | 队列，先进先出(FIFO)数据结构 | `deque` |
| `priority_queue` | 优先队列/堆 | `vector` |

### 20.10.1 迭代器类别
20.6.2节和20.10节提到，不同类型的迭代器支持不同的操作。因此C++标准库定义了**迭代器类别**(iterator category)：

| 迭代器类别 | 支持的操作 | 示例 |
| --- | --- | --- |
| 输入迭代器(input iterator) | `*`（可读）、`++`、`==`、`!=` | `istream_iterator` |
| 输出迭代器(output iterator) | `*`（可写）、`++`、`==`、`!=` | `ostream_iterator`、`front_insert_iterator`、`back_insert_iterator` |
| 前向迭代器(forward iterator) | 满足输入迭代器和输出迭代器（除非元素类型是`const`），并且可多次遍历 | `forward_list<T>::iterator`、`unordered_map<K,V>::iterator` |
| 双向迭代器(bidirectional iterator) | 满足前向迭代器，并支持`--` | `list<T>::iterator`、`map<K,V>::iterator` |
| 随机访问迭代器(random-access iterator) | 满足双向迭代器，并支持`+`、`+=`、`-`、`-=`、`[]` | `vector<T>::iterator`、`deque<T>::iterator` |

注：
* 和普通指针一样，迭代器通常也提供`->`运算符，`it->m`等价于`(*it).m`。
* 迭代器通常是对指针的封装，`operator*`和`operator->`分别返回被指向的对象和指针本身。例如，假设迭代器`it`的底层指针是`p`，则这两个运算符分别返回`*p`和`p`。`(*it).m`用函数调用语法表示为`it.operator*().m`（即`(*p).m`），`it->m`则等价于`it.operator->()->m`（即`p->m`）。参考第19章习题[19-10](https://github.com/ZZy979/PPP-code/blob/main/ch19/unique_ptr.h)和[19-11](https://github.com/ZZy979/PPP-code/blob/main/ch19/counted_ptr.h)。

迭代器类别可以用下图表示：

![迭代器类别](/assets/images/ppp-note-ch20-containers-and-iterators/迭代器类别.png)

注意，迭代器类别并不是类，因此上图不是用派生实现的类层次结构（图中箭头仅表示“支持的操作集合的包含关系”）。

参考：
* [Iterator categories](https://en.cppreference.com/w/cpp/iterator#Iterator_categories)
* [Iterator tags](https://en.cppreference.com/w/cpp/iterator/iterator_tags)

## 简单练习
[drill20](https://github.com/ZZy979/PPP-code/blob/main/ch20/drill20.cpp)

## 习题
* [20-2](https://github.com/ZZy979/PPP-code/blob/main/ch20/jack_and_jill_v2.cpp)
* [20-4](https://github.com/ZZy979/PPP-code/blob/main/ch20/jack_and_jill_v4.cpp)
* [20-5](https://github.com/ZZy979/PPP-code/blob/main/ch19/drill19.cpp)
* [20-6](https://github.com/ZZy979/PPP-code/blob/main/ch20/simple_text_editor.cpp)
* [20-7](https://github.com/ZZy979/PPP-code/blob/main/ch20/high.h)
* [20-8~20-10](https://github.com/ZZy979/PPP-code/blob/main/ch20/simple_text_editor.cpp)
* [20-12](https://github.com/ZZy979/PPP-code/blob/main/ch20/exec20-12.cpp)
* [20-13](https://github.com/ZZy979/PPP-code/blob/main/ch20/doubly_linked_list_v4.h)
* [20-14](https://github.com/ZZy979/PPP-code/blob/main/ch20/singly_linked_list.h) （简化的`std::forward_list`）
* [20-15](https://github.com/ZZy979/PPP-code/blob/main/ch20/pvector.h)
* [20-16](https://github.com/ZZy979/PPP-code/blob/main/ch20/ovector.h)
* [20-17](https://github.com/ZZy979/PPP-code/blob/main/ch20/ownership_vector.h)
* [20-18~20-19](https://github.com/ZZy979/PPP-code/blob/main/ch20/range_checked_iterator.h)
* [20-20](https://github.com/ZZy979/PPP-code/blob/main/ch20/exec20-20.cpp)
