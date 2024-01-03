---
title: 《C++程序设计原理与实践》笔记 第18章 向量和数组
date: 2023-05-28 11:54:48 +0800
categories: [C/C++, PPP]
tags: [cpp, vector, copy constructor, copy assignment, rvalue reference, move semantics, move constructor, move assignment, array]
---
本章将介绍如何拷贝以及通过下标访问向量。为此，我们讨论一般的拷贝技术，并考虑向量与底层数组表示之间的关系。我们将展示数组与指针的关系及其使用引发的问题。我们还将讨论对于每种类型必须考虑的五种基本操作：构造、默认构造、拷贝构造、拷贝赋值和析构。另外，容器还需要移动构造函数和移动赋值。

## 18.1 引言
在本章中，我们将学习编程语言特性和技术，从而摆脱来自底层计算机内存的限制和困难。我们希望能够使用根据逻辑需求提供我们恰好想要的属性的类型进行编程。

`vector`类型能够控制所有对它元素的访问，并提供了从用户角度而不是硬件角度看起来“很自然”的操作。

本章重点介绍**拷贝**(copying)的概念。这是一个重要但相当技术性的问题：拷贝一个非平凡的（nontrivial，即比`int`等更加复杂的类型，例如`string`、`vector`等）对象意味着什么？拷贝操作后副本之间在多大程度上是独立的？有几种拷贝操作？如何指定使用哪一种？拷贝和其他基本操作（例如初始化和析构）之间有什么关系？

当没有高层次的类型（例如`vector`和`string`）时，我们不可避免地需要讨论如何操作内存。我们将研究数组和指针，以及它们的关系、用法和陷阱。这对于任何需要编写低层次C/C++代码的人而言是十分重要的。

注意，虽然`vector`的实现细节是特定于C++的，但每种语言中的“高层次”类型（`string`、`vector`、`list`、`map`等）的构建都反应了本章所介绍的一些基本问题的解决方案。

## 18.2 初始化
第17章中的`vector`只能初始化为默认值，之后再赋值：

```cpp
vector v(2); // tedious and error-prone
v.set(0, 1.2);
v.set(1, 7.89);
v.set(2, 12.34);
```

这种方式不仅繁琐而且容易出错（上面的代码搞错了元素个数）。我们希望直接用一组值来初始化向量，例如：

```cpp
vector v = {1.2, 7.89, 12.34};
```

为此，我们需要写一个接受初始化列表作为参数的构造函数。从C++11开始，**`{}`括起来的`T`类型的元素列表是一个标准库类型`initializer_list<T>`的对象**，定义在头文件\<initializer_list\>中。因此可以这样编写：

```cpp
// initializer-list constructor
vector(initializer_list<double> lst) :sz(lst.size()), elem(new double[sz]) {
    copy(lst.begin(), lst.end(), elem);    // initialize (using std::copy())
}
```

这里使用了标准库`copy()`算法将初始化列表中的元素拷贝到`elem`数组。

[简单向量v2 - 初始化列表构造函数](https://github.com/ZZy979/PPP-code/blob/1457d5b3074149705609f3711df265229605cedd/ch18/simple_vector.h)

现在有两种方式初始化向量：

```cpp
vector v1 = {1,2,3};  // three elements 1.0, 2.0, 3.0
vector v2(3);         // three elements each with the (default) value 0.0
```

注意，`()`用于指定元素个数（调用构造函数`vector(int)`），`{}`用于指定元素列表（调用构造函数`vector(initializer_list<double>)`）。例如：

```cpp
vector v1{3};  // one element with the value 3.0
vector v2(3);  // three elements each with the (default) value 0.0
```

注：如果没有初始化列表构造函数，则`vector v{3};`等价于`vector v(3);`，详见9.4.2节“C++初始化语法”。

在大多数情况下，`{}`前的`=`可以省略，因此`vector v = {1, 2, 3};`等价于`vector v{1, 2, 3};`。

注意，`initializer_list<double>`是传值的，这是语言规则要求的：`initializer_list`只是引用了分配在“其他地方”的元素（底层就是一个数组元素的指针，类似于`vector`，但拷贝`initializer_list`对象并不会拷贝这些元素）。

## 18.3 拷贝
再次考虑我们的`vector`，尝试拷贝一个向量：

```cpp
void f(int n) {
    vector v(3);    // define a vector of 3 elements
    v.set(2,2.2);   // set v[2] to 2.2
    vector v2 = v;  // what happens here?
    // ...
}
```

理想情况下，`v2`将成为`v`的拷贝（副本），即`v2.size() == v.size()`，并且对于[0, `v.size()`)范围内的所有`i`满足`v2[i] == v[i]`（虽然现在还没有定义`[]`运算符）。另外，当`f()`退出时，所有内存都被返回给自由存储。标准库`vector`是这样做的，但我们的（仍然过于简单的）`vector`却不是这样。

对于一个类而言，**拷贝的默认含义是“拷贝所有数据成员”**。对于只有内置类型成员的简单类（例如`Point`），这通常就足够了。但是对于指针成员，仅仅拷贝成员（指针的值）会产生问题。例如对于`vector`，这意味着拷贝之后`v2.sz == v.sz`且`v2.elem == v.elem`，如下图所示：

![错误的拷贝构造](/assets/images/ppp-note-ch18-vectors-and-arrays/错误的拷贝构造.png)

也就是说，`v2`并没有拷贝`v`的元素，只是共享了`v`的元素（只拷贝了指针），这会导致两个问题：
* 给`v2`的元素赋值会影响`v`的元素，反之亦然，这不是我们想要的结果。
* 当`f()`返回时，`v`和`v2`的析构函数被隐式调用，导致`elem`指向的内存被释放了两次，造成灾难性的结果（见17.4.6节）。

### 18.3.1 拷贝构造函数
我们应该提供一个拷贝操作来拷贝元素，并保证当我们使用一个`vector`初始化另一个`vector`时，这个拷贝操作将被调用。

为此，我们需要一个**拷贝构造函数**(copy constructor)，其参数是被拷贝对象的引用。对于`vector`：

```cpp
vector(const vector&);
```

**当使用一个对象初始化另一个相同类型的对象时，拷贝构造函数将被调用。** 包括：
* 初始化：`T a = b;`或`T a(b);`，其中`b`是`T`类型
* 函数参数传递：`f(a)`，其中`a`和函数参数都是`T`类型
* 函数返回值：`return a;`，其中函数返回值是`T`类型，且`T`没有移动构造函数

改进后的`vector`如下：

[简单向量v2 - 拷贝构造函数](https://github.com/ZZy979/PPP-code/blob/18dc9ecd4f29b321db69ceebccf11e453780d7c9/ch18/simple_vector.h)

给定拷贝构造函数，再次考虑例子`vector v2 = v;`，现在的结果如下图所示：

![正确的拷贝构造](/assets/images/ppp-note-ch18-vectors-and-arrays/正确的拷贝构造.png)

显然，现在两个向量相互独立，改变`v`中元素的值不会影响`v2`，反之亦然。析构函数也能够正常工作。

`vector v2 = v;`等价于`vector v2{v};`或者`vector v2(v);`。当`v`和`v2`是相同的类型，且该类型定义了拷贝构造函数时，这几种写法的含义完全相同。

### 18.3.2 拷贝赋值
除了拷贝构造，还可以通过赋值来拷贝对象。和拷贝构造一样，**拷贝赋值的默认含义是“拷贝所有成员”**，目前的`vector`就是这样。例如：

```cpp
void f2(int n) {
    vector v(3);  // define a vector
    v.set(2,2.2);
    vector v2(4);
    v2 = v;       // assignment: what happens here?
    // ...
}
```

我们希望`v2`称为`v`的副本，但默认拷贝赋值的结果是使`v2.sz == v.sz`且`v2.elem == v.elem`，如下图所示：

![错误的拷贝赋值](/assets/images/ppp-note-ch18-vectors-and-arrays/错误的拷贝赋值.png)

这会导致两个问题：
* 重复删除：和默认拷贝构造函数一样，当`f2()`返回时，`v`和`v2`共同指向的元素被释放了两次。
* 内存泄露：`v2`原来指向的4个元素的内存没有被释放。

对于拷贝赋值的改进本质上与拷贝构造相同，需要定义**拷贝赋值**(copy assignment)运算符：

```cpp
vector& operator=(const vector& v);
```

**当对象出现在赋值表达式左侧时，拷贝赋值运算符将被调用。**

[简单向量v2 - 拷贝赋值](https://github.com/ZZy979/PPP-code/blob/4c570d2714eb8905c5fbc7216b4196fe95dce18f/ch18/simple_vector.h)

赋值比构造更复杂一些，因为必须处理旧的元素。这里先拷贝新元素，之后释放旧元素，最后让`elem`指向新元素，如下图所示：

![正确的拷贝赋值](/assets/images/ppp-note-ch18-vectors-and-arrays/正确的拷贝赋值.png)

现在的`vector`不会泄露内存和重复释放内存了。

注意，实现赋值操作时，先释放旧元素的内存再拷贝可能会简化代码，但这样无法处理将一个`vector`赋值给自身的情况：

```cpp
vector v(10);
v = v;  // self-assignment
```

除非先检查被拷贝的对象和当前对象是否是同一个对象，即`if (&v == this) return;`。书中的实现方式虽然性能不是最优的（对于自身赋值也拷贝了一次元素），但正确处理了这种情况。

### 18.3.3 拷贝术语
拷贝的基本问题是拷贝指针（或引用）还是拷贝被指向（或引用）的数据：
* **浅拷贝**(shallow copy)：只拷贝指针，从而两个指针指向同一个对象。
* **深拷贝**(deep copy)：拷贝指针指向的数据，从而两个指针指向不同的对象。标准库的`vector`、`string`等都是这样。当我们想要对自己的类的对象进行深拷贝时，就需要定义拷贝构造函数和拷贝赋值。

下面是一个浅拷贝的例子：

```cpp
int* p = new int(77);
int* q = p;  // copy the pointer p
*p = 88;     // change the value of the int pointed to by p and q
```

![浅拷贝](/assets/images/ppp-note-ch18-vectors-and-arrays/浅拷贝.png)

相反，下面是一个深拷贝的例子：

```cpp
int* p = new int(77);
int* q = new int(*p);  // allocate a new int, then copy the value pointed to by p
*p = 88;               // change the value of the int pointed to by p
```

![深拷贝](/assets/images/ppp-note-ch18-vectors-and-arrays/深拷贝.png)

使用这一术语，原来的`vector`问题在于只做了浅拷贝，而改进后的`vector`（和标准库`vector`）做了深拷贝。

提供浅拷贝的类型称为具有**指针语义**(pointer semantics)或**引用语义**(reference semantics)（只拷贝地址）；提供深拷贝的类型称为具有**值语义**(value semantics)（拷贝指向的值）。

### 18.3.4 移动
如果一个`vector`有很多元素，那么拷贝它的代价会很高。考虑一个例子：

```cpp
vector fill(istream& is) {
    vector res;
    for (double x; is>>x; ) res.push_back(x);
    return res;
}

void use() {
    vector vec = fill(cin);
    // ... use vec ...
}
```

假设`res`有10万个元素，则将其拷贝到`vec`的代价是很高的。但是我们并不希望拷贝，毕竟我们永远不会使用`res`。

一种替代方法是传引用参数：

```cpp
void fill(istream& is, vector& v) {
    for (double x; is>>x; ) v.push_back(x);
}

void use() {
    vector vec;
    fill(cin, vec);
    // ... use vec ...
}
```

缺点是不能使用返回值语法，必须先声明变量。

另一种方法是返回`new`创建的指针：

```cpp
vector* fill(istream& is) {
    vector* res = new vector;
    for (double x; is>>x; ) res->push_back(x);
    return res;

}
void use() {
    vector* vec = fill(cin);
    // ... use vec ...
    delete vec;
}
```

缺点是必须记得`delete`这个向量（如17.4.6节所述）。

我们希望**使用返回值语法，同时避免拷贝**。为此，C++11引入了**移动语义**(move semantics)：通过“窃取”资源，直接将`res`的资源**移动**(move)到`vec`，如下图所示：

![移动前](/assets/images/ppp-note-ch18-vectors-and-arrays/移动前.png)

![移动后](/assets/images/ppp-note-ch18-vectors-and-arrays/移动后.png)

移动之后，`vec`将引用`res`的元素，而`res`将被置空。从而以仅仅拷贝一个`int`和一个指针的代价将10万个元素从`res`移动到`vec`。换句话说，移动 = “窃取”资源 = 浅拷贝+置空原指针。

为了在C++中表达移动语义，需要定义**移动构造函数**(move constructor)和**移动赋值**(move assignment)运算符：

```cpp
vector(vector&& v);             // move constructor
vector& operator=(vector&& v);  // move assignment
```

其中`&&`符号称为**右值引用**(rvalue reference)。注意移动操作的参数不是`const`，因为移动操作的目的之一是将源对象置空。

**当使用一个右值初始化一个相同类型的对象时，移动构造函数将被调用。** 包括：
* 初始化：`T a = std::move(b);`或`T a(std::move(b));`，其中`b`是`T`类型
* 函数参数传递：`f(std::move(a))`，其中`a`和函数参数都是`T`类型
* 函数返回值：`return a;`，其中函数返回值是`T`类型，且`T`有移动构造函数

注：
* 有名字的变量是左值，字面值、算术表达式、临时对象、返回值是值类型的函数调用等都是右值，详见[Value categories](https://en.cppreference.com/w/cpp/language/value_category)。
* 左值不能赋给右值引用（因此不能被移动），除非使用`std::move()`函数转换为右值引用，这意味着该对象可能被窃取资源，因此不能再使用。
* 从C++17开始，编译器会强制进行[拷贝消除](https://en.cppreference.com/w/cpp/language/copy_elision)(copy elision)，如果初始值是纯右值(prvalue)，则移动构造函数调用会被优化掉。

**当对象出现在赋值表达式左侧，并且右侧是一个相同类型的右值时，移动赋值运算符将被调用。**

[简单向量v2 - 移动构造函数和移动赋值](https://github.com/ZZy979/PPP-code/blob/b1a9d9b1a02947665630cbc2b476bb155efa7bc1/ch18/simple_vector.h)

再次考虑前面的例子。为`vector`实现了移动语义后，在`fill()`返回时，`vector`的移动构造函数将被隐式调用（`fill()`和`use()`的代码均不需要修改）。

注：另见[【C++】右值引用、移动语义和完美转发]({% post_url 2023-06-02-cpp-rvalue-reference-move-semantics-and-perfect-forwarding %})

## 18.4 基本操作
现在可以讨论如何决定一个类应该有哪些构造函数，是否应该有析构函数，以及是否应该提供拷贝和移动操作。有7种需要考虑的基本操作：
* 有参构造函数`T(A1 a1, A2 a2, ...)`
* 默认构造函数`T()`
* 拷贝构造函数`T(const T&)`
* 拷贝赋值`T& operator=(const T&)`
* 移动构造函数`T(T&&)`
* 移动赋值`T& operator=(T&&)`
* 析构函数`~T()`

通常，我们需要一个或多个带参数的构造函数来初始化对象，初始值（参数）的含义和用途完全取决于构造函数。通常我们使用构造函数来建立不变式（9.4.3节）。

如果我们希望在不指定初始值的情况下创建类的对象，则需要默认构造函数。最常见的例子是将对象放在标准库`vector`中（例如`vector vs(10);`）。当我们可以用一个有意义的、明显的默认值为类建立不变式时，默认构造函数是有意义的。例如，对于数值类型默认值为0，对于`string`为空字符串，对于`vector`为空向量。对于类型`T`，如果存在默认构造函数，则`T{}`或`T()`是默认值。

如果一个类获取了资源，则需要析构函数。资源包括自由存储、文件、锁、线程句柄、套接字等。一个类需要析构函数的另一个特征是它具有指针或引用成员。

一个需要析构函数的类几乎总是需要拷贝操作和移动操作。原因很简单：如果一个对象获取了资源（具有指向它的指针成员），那么拷贝的默认含义（浅拷贝）几乎肯定是错误的。`vector`是一个典型的例子。

另外，如果派生类具有析构函数，则基类需要虚析构函数（17.5.2节）。

### 18.4.1 显式构造函数
接受单个参数的构造函数定义了一个从参数类型到该类型的转换。例如：

```cpp
class complex {
public:
    complex(double);  // defines double-to-complex conversion
    complex(double,double);
    // ...
};

complex z1 = 3.14; // OK: convert 3.14 to (3.14,0)
complex z2 = complex(1.2, 3.4);
```

然而，应该谨慎地使用这种隐式转换，因为它可能造成意想不到的结果。例如，`vector`有一个接受`int`的构造函数，因此可以这样使用：

```cpp
vector v = 10;  // odd: makes a vector of 10 doubles
v = 20;         // eh? Assigns a new vector of 20 doubles to v

void f(const vector&);
f(10);          // eh? Calls f with a new vector of 10 doubles
```

我们可以通过**显式构造函数**(explicit constructor)禁止将构造函数用于隐式转换。显式构造函数使用关键字`explicit`定义：

```cpp
class vector {
    // ...
    explicit vector(int);
    // ...
};

vector v = 10;  // error: no int-to-vector conversion
v = 20;         // error: no int-to-vector conversion
vector v0(10);  // OK

void f(const vector&);
f(10);          // error: no int-to-vector<double> conversion
f(vector(10));  // OK
```

[简单向量v2 - 显式构造函数](https://github.com/ZZy979/PPP-code/blob/63deb7eeac30c2b0a25abd4e17ac2c4e0a53c00a/ch18/simple_vector.h)

### 18.4.2 调试构造函数和析构函数
在程序的执行过程中，构造函数和析构函数都在明确定义的、可预测的点被调用：
* 每当类型`X`的一个对象被创建时，`X`的一个构造函数将被调用。例如，变量初始化、传值参数、使用`new`创建对象、拷贝对象等。
* 每当类型`X`的一个对象被销毁时，`X`的析构函数将被调用。例如，变量离开作用域、`delete`一个指向对象的指针或者程序结束。

为了体会这个问题，一种好方法是在构造函数、赋值操作和析构函数中加入打印语句。例如：

[试一试](https://github.com/ZZy979/PPP-code/blob/main/ch18/debug_ctor_dtor.cpp)

★一定要运行这个例子，并确保你理解了输出结果，这样你就会明白对象的构造和析构的大致过程。

程序的每一行语句对应的输出如下图所示：

![示例程序输出](/assets/images/ppp-note-ch18-vectors-and-arrays/示例程序输出.png)


其中几个比较复杂语句的拷贝过程如下图所示（其中箭头上的 "c" 表示拷贝构造， "=" 表示拷贝赋值）：

![复杂语句的拷贝过程](/assets/images/ppp-note-ch18-vectors-and-arrays/复杂语句的拷贝过程.png)

* `loc = X(5)`：①创建临时对象`X(5)`（地址为0xae19bffa90）；②将临时对象拷贝赋值到`loc`（地址为0xae19bffa8c）；③销毁临时对象。
* `loc2 = copy(loc)`：①由`loc`拷贝构造`copy()`函数的形参`x`（地址为0xae19bffa98）；②由形参`x`拷贝构造返回值临时对象（地址为0xae19bffa94）；③将临时对象拷贝赋值到`loc2`（地址为0xae19bffa88）；④销毁临时对象；⑤销毁形参`x`。
* `loc2 = copy2(loc)`：①由`loc`拷贝构造`copy2()`函数的形参`x`（地址为0xae19bffaa0）；②由形参`x`拷贝构造局部变量`xx`（地址为0xae19bffa9c）；③由`xx`拷贝构造返回值临时对象；④将临时对象拷贝赋值到`loc2`（地址为0xae19bffa88）；⑤销毁临时对象；⑥销毁局部变量`xx`；⑦销毁形参`x`。注意，这里的返回值临时对象可能会被编译器优化掉，即直接将`xx`拷贝赋值到`loc2`，上面的输出结果就是这样。
* `X& r = ref_to(loc)`：参数和返回值都是引用，因此没有任何构造函数调用。
* `vector<X> v(4)`：每个元素都调用默认构造函数（这也说明`vector`的这种用法要求元素类型必须是可默认构造的）。
* 程序结束之前，所有对象的析构函数被调用，并且销毁顺序与创建顺序相反。

有些编译器足够智能，能够消除不必要的拷贝。但是，如果想要做到可移植（即在不同平台、使用不同编译器都能得到相同的效果），考虑移动操作。

## 18.5 访问vector元素
到目前为止，我们一直使用`set()`和`get()`成员函数访问元素。为了使用通常的下标语法`v[i]`，需要重载`[]`运算符，即定义一个名为`operator[]`的成员函数：

```cpp
double operator[](int i) { return elem[i]; }
```

这种实现看起来很好，但过于简单了——这个下标运算符返回的值只可读不可写（因为返回值`double`是一个右值而不是左值）：

```cpp
vector v(10);
double x = v[2];  // fine
v[3] = x;         // error: v[3] is not an lvalue
```

其中`v[i]`被解释为`v.operator[](i)`。

返回元素的引用可以解决这一问题：

```cpp
double& operator[](int n) { return elem[n]; }
```

现在上面的代码可以正常工作，因为`double&`是一个左值。

注：`[]`运算符可以代替`set()`和`get()`。

### 18.5.1 const重载
目前的`operator[]`还有一个问题：不能对`const vector`调用。例如：

```cpp
void f(const vector& cv) {
    double d = cv[1];  // error, but should be fine
    cv[1] = 2.0;       // error (as it should be)
}
```

其中第一个`cv[1]`只读取了元素的值，并没有修改`vector`，但编译器仍然会报错，因为`operator[]`的返回类型是`double&`，本身就是可修改的。解决方法是提供一个`const`成员函数版本：

```cpp
double& operator[](int n);       // for non-const vectors
double operator[](int n) const;  // for const vectors
```

`const`版本的`operator[]`也可以返回`const double&`，但`double`是一个很小的对象，因此没有必要返回引用（见8.5.6节）。现在可以这样使用：

```cpp
void ff(const vector& cv, vector& v) {
    double d = cv[1];  // fine (uses the const [])
    cv[1] = 2.0;       // error (uses the const [])
    d = v[1];          // fine (uses the non-const [])
    v[1] = 2.0;        // fine (uses the non-const [])
}
```

[简单向量v2 - 下标运算符](https://github.com/ZZy979/PPP-code/blob/39a3492c878637d53dc38fc7049ba7d6d138d009/ch18/simple_vector.h)

## 18.6 数组
**数组**(array)是一个固定大小的、连续的元素序列。
* `T a[N];`声明了一个由`N`个`T`类型的元素组成的数组，其中`N`必须是整型常量表达式。
* `T* p = new T[n];`在自由存储上分配了一个由`N`个`T`类型的元素组成的数组，其中`n`可以是任意整型表达式。

数组元素编号为0到N-1，可以使用下标运算符`[]`访问，即`a[0]`、...、`a[N-1]`。

数组有很大的局限性（例如**数组不知道自身的大小**），因此尽可能优先使用`vector`。然而，数组早在`vector`之前就存在了，并且与其他语言（尤其是C语言）的数组大致等价，因此你必须了解数组，以便能够应付那些很久以前的代码，或者由不能使用`vector`的人编写的代码。

### 18.6.1 指向数组元素的指针
指针可以指向数组元素。例如：

```cpp
double ad[10];
double* p = &ad[5];  // point to ad[5]
```

现在指针`p`指向`ad[5]`：

![指向数组元素的指针](/assets/images/ppp-note-ch18-vectors-and-arrays/指向数组元素的指针.png)

可以对指针使用下标和解引用，从而访问数组元素：

```cpp
*p = 7;  // equivalent to p[0] = 7
p[2] = 6;
p[-3] = 9;
```

得到

![通过指针访问数组元素](/assets/images/ppp-note-ch18-vectors-and-arrays/通过指针访问数组元素.png)

下标可以是正数或负数，只要对应的元素在数组的范围[0, N)内。然而，访问数组范围之外的数据是非法的（见17.4.3节）。

当指针指向一个数组时，加减操作可以让指针指向其他元素。例如：

```cpp
p += 2;  // move p 2 elements to the right
```

![指针加操作](/assets/images/ppp-note-ch18-vectors-and-arrays/指针加操作.png)

```cpp
p -= 5;  // move p 5 elements to the left
```

![指针减操作](/assets/images/ppp-note-ch18-vectors-and-arrays/指针减操作.png)

使用`+`、`-`、`+=`和`-=`移动指针的操作称为**指针运算**(pointer arithmetic)。进行这种操作时，我们必须保证结果不会超过数组的范围：

```cpp
p += 1000;      // insane: p points into an array with just 10 elements
double d = *p;  // illegal: probably a bad value (definitely an unpredictable value)
*p = 12.34;     // illegal: probably scrambles some unknown data
```

不幸的是，并非所有涉及指针运算的bug都很容易发现。通常最好避免指针运算。

指针运算最常见的用法是使用`++`指向下一个元素以及使用`--`指向上一个元素。例如，可以这样打印`ad`的元素值：

```cpp
for (double* p = &ad[0]; p < &ad[10]; ++p) cout << *p << '\n';
for (double* p = &ad[9]; p >= &ad[0]; --p) cout << *p << '\n';
```

注：
* `p+i`等价于`&p[i]`，`*(p+i)`等价于`p[i]`
* 指针与整数加减时，会自动放缩对象大小的倍数。例如，`p+i`与`p`实际的地址值之差为`i * sizeof(*p)`，因此刚好指向`p`之后的第`i`个对象。

指针运算的大多数实际用法是将指针作为函数参数传递。在这种情况下，编译器并不知道指针指向的数组有多少个元素，你必须主动提供（见18.7.2节）。我们应当尽量避免这种情况。

### 18.6.2 指针和数组
数组名代表了数组的所有元素。例如：

```cpp
char ch[100];
```

则`sizeof(ch)`是100。

然而，**数组名是首元素的地址**，因此可以转换（“退化”）为指针。例如：

```cpp
char* p = ch;
```

其中`p`被初始化为`&ch[0]`，但`sizeof(p)`是4（或8）而不是100。

这一点是非常有用的。一个原因是将数组传递给指针参数时可以避免拷贝。例如，函数`strlen()`统计以0结尾的字符数组中的字符数（即C风格字符串的长度）：

```cpp
// similar to the standard library strlen()
int strlen(const char* p) {
    int count = 0;
    while (*p) { ++count; ++p; }
    return count;
}
```

可以通过`strlen(ch)`调用该函数，这等价于`strlen(&ch[0])`。和`vector`不同，**通过参数传递数组并不会拷贝数组元素**（只拷贝了一个指针）。参数声明`char* p`等价于`char p[]`。

注意，**数组名不能被赋值**，例如：

```cpp
char ac[10];
ac = new char[20];      // error: no assignment to array name
&ac[0] = new char[20];  // error: no assignment to pointer value
```

**也不能使用赋值拷贝数组**：

```cpp
int x[100];
int y[100];
x = y;           // error
int z[100] = y;  // error
```

如果需要拷贝数组，则必须编写复杂的代码。例如：

```cpp
for (int i=0; i<100; ++i) x[i]=y[i];  // copy 100 ints
memcpy(x,y,100*sizeof(int));          // copy 100*sizeof(int) bytes
copy(y,y+100, x);                     // copy 100 ints
```

注：关于指针与数组的关系另见
* [《C程序设计语言》笔记 第5章 指针与数组]({% post_url 2022-04-26-tcpl-note-ch5-pointers-and-arrays %})
* [二维数组的行指针和列指针]({% post_url 2022-05-11-cpp-two-dimensional-array-and-pointer %})

### 18.6.3 数组初始化
**字符数组可以使用字符串常量初始化。** 例如：

```cpp
char ac[] = "Beorn";  // array of 6 chars
```

字符串只有5个字符，但`ac`的长度为6，因为编译器会自动在字符串常量的末尾添加0，如下图所示：

![字符数组](/assets/images/ppp-note-ch18-vectors-and-arrays/字符数组.png)

这种以0结尾的字符数组称为**C风格字符串**(C-style string)。所有的字符串常量都是C风格字符串。例如：

```cpp
char* pc = "Howdy";  // pc points to an array of 6 chars
```

![字符串常量](/assets/images/ppp-note-ch18-vectors-and-arrays/字符串常量.png)

注意，结尾是字符`'\0'`（值为0）而不是`'0'`（值为48）。结尾0的目的在于让函数找到字符串的结尾（例如`strlen()`），因为数组不知道自身的大小。

标准库头文件\<cstring\>定义了`strlen()`函数。该函数统计字符数，不包括结尾的0。因此，n个字符的C风格字符串需要长度为n+1的字符数组来存储。

注：`string`提供了`c_str()`函数，返回字符串底层字符数组的指针，可以将`string`转换为C风格字符串。

只有字符数组可以使用字符串常量初始化，但所有数组都可以使用对应元素类型的初始值列表进行初始化。例如：

```cpp
int ai[] = {1, 2, 3, 4, 5, 6};         // array of 6 ints
int ai2[100] = {0,1,2,3,4,5,6,7,8,9};  // the last 90 elements are initialized to 0
double ad[100] = {};                   // all elements initialized to 0.0
char chars[] = {'a', 'b', 'c'};        // no terminating 0!
```

注意，`chars`的长度是3（而不是4）——“在末尾添加0”的规则只适用于字符串。**如果没有指定数组大小，则从初始化列表中推断出来。** 如果初始值的个数小于数组大小（例如`ai2`和`ad`），则剩余元素将被初始化为元素类型的默认值。

### 18.6.4 指针问题
指针和数组经常被过度使用和误用，本节对经常出现的问题进行总结。所有与指针相关的严重问题都涉及**试图访问非预期类型对象的数据**，而很多这种问题涉及**访问数组边界之外的数据**（越界访问）。在本节中我们主要考虑：
* 通过空指针访问
* 通过未初始化的指针（野指针）访问
* 访问数组结尾之后的数据
* 访问已释放的对象
* 访问已离开作用域的对象

在所有情况下，对于程序员来说实际的问题在于代码看起来没有问题。更糟糕的是（当通过指针写数据时），问题可能在某些表面上不相关的对象被损坏很长时间之后才显现出来。下面考虑一些例子。

**（1）不要通过空指针访问数据**

```cpp
int* p = nullptr;
*p = 7;  // ouch!
```

显然，在真实世界的程序中，初始化和使用之间通常还存在其他代码，将`p`传递给函数或者接受函数返回值是十分常见的例子。尽量不要传递空指针，如果必须这么做，则**在使用前检查空指针**：

```cpp
int* p = fct_that_can_return_a_nullptr();

if (p == nullptr) {
    // do something
}
else {
    // use p
    *p = 7;
}
```

```cpp
void fct_that_can_receive_a_nullptr(int* p) {
    if (p == nullptr) {
        // do something
    }
    else {
        // use p
        *p = 7;
    }
}
```

使用引用和使用异常捕获错误是避免空指针的主要工具。

注：引用无法像空指针一样表达“不存在”的含义（相当于将检查空指针的工作留给了调用者）。C++17引入了`optional`，表示一个“可能存在的值”，同时有助于避免空指针问题。

**（2）对指针进行初始化**

```cpp
int* p;
*p = 9;  // ouch!
```

特别的，不要忘记初始化类成员指针。

**（3）不要访问不存在的数组元素**

```cpp
int a[10];
int* p = &a[10];
*p = 11;     // ouch!
a[10] = 12;  // ouch!
```

小心循环的第一个和最后一个元素（例如18.6.1节最后打印数组元素的例子）。尽量不要将数组作为第一个元素的指针进行传递，取而代之的是使用`vector`。如果必须这么做，那么应该极其小心，并且将数组大小一起传递（见18.7.2节）。

**（4）不要通过已删除的指针访问数据**

```cpp
int* p = new int(7);
// ...
delete p;
// ...
*p = 13;  // ouch!
```

`delete p`或此后的代码可能已经改写了`*p`或将其用于其他对象（注：有的编译器可能在`delete`时将`p`指向的内存区域填充为一些无意义的字节，之后这块内存可能被内存管理器分配给其他对象）。在所有问题中，这一问题是最难系统地避免的。防止这一问题的最有效方法是避免出现“裸露的”`new`和`delete`——只在构造函数和析构函数中使用`new`和`delete`，或者使用容器来处理`delete`。

**（5）不要返回局部变量的指针**

```cpp
int* f() {
    int x = 7;
    // ...
    return &x;
}

// ...

int* p = f();
// ...
*p = 15;  // ouch!
```

在函数退出时局部变量将被释放。几乎没有编译器会捕获与返回局部变量的指针相关的问题。

考虑一个逻辑上等价的例子：

```cpp
vector& ff() {
    vector x(7);  // 7 elements
    // ...
    return x;
}  // the vector x is destroyed here

// ...

vector& p = ff();
// ...
p[4] = 15;  // ouch!
```

很多编译器都能捕获这种返回问题。

程序员通常会低估这些问题。然而，很多有经验的程序员都曾经被这些简单的数组和指针问题不计其数的变体和组合所打败。解决方法是不要使用指针、数组、`new`和`delete`污染你的代码。如果你这么做了，那么在真实规模的程序中，“小心谨慎”是根本不够的。相反，应该依赖向量、RAII（ "Resource Acquisition Is Initialization" ，见19.5节），以及其他的系统途径来管理内存和其他资源。

## 18.7 实例：回文
**回文**(palindrome)是从两端拼写相同的单词。例如，anna、petep和malayalam是回文，而ida和homesick不是回文。有两种基本方法判断一个单词是否是回文：
* 构造一个逆序的副本，与原单词比较。
* 判断第一个和最后一个字母是否相同，然后判断第二个和倒数第二个字母是否相同，直到到达单词中间。

这里我们将采用第二种方法。根据单词的表示方式和跟踪比较位置的方式，有很多种方法在代码中表达这一思想。

### 18.7.1 使用string实现
首先使用标准库`string`以及`int`索引来跟踪比较位置：

[is_palindrome函数](https://github.com/ZZy979/PPP-code/blob/main/ch18/palindrome.cpp)（重载1）

如果到达中间且未发现不同的字符则返回`true`。建议使用空字符串、只有一个字符的字符串、以及具有偶数个和奇数个字符的字符串进行测试。

### 18.7.2 使用数组实现
如果没有`string`，则不得不使用数组存储字符：

[is_palindrome函数](https://github.com/ZZy979/PPP-code/blob/main/ch18/palindrome.cpp)（重载2）

为了测试该函数，可以使用`c_str()`函数将`string`转换为0结尾的字符数组，也可以使用`istream`的`>>`运算符（该运算符提供了接受`const char*`参数的版本）。例如：

```cpp
// read at most max-1 characters from is into buffer
istream& read_word(istream& is, char* buffer, int max) {
    is.width(max);  // read at most max-1 characters in the next >>
    is >> buffer;   // read whitespace-terminated word,
                    // add zero after the last character read into buffer
    return is;
}
```

数组版本的代码比`string`版本复杂得多，并且不能处理任意长度的字符串（必须指定最大长度）。

### 18.7.3 使用指针实现
除了索引，还可以使用指针来标识字符位置：

[is_palindrome函数](https://github.com/ZZy979/PPP-code/blob/main/ch18/palindrome.cpp)（重载3）

我们还可以这样重写`is_palindrome()`函数（只是为了好玩）：

```cpp
// first points to the first letter, last to the last letter
bool is_palindrome(const char* first, const char* last) {
    if (first<last) {
        if (*first!=*last) return false;
        return is_palindrome(first+1,last-1);
    }
    return true;
}
```

这是递归实现版本。当我们重新描述回文的定义（递归定义）时，这段代码就变得很明显了：如果一个单词的第一个和最后一个字符相同，并且删除首尾字符后的子串是回文，则这个单词是回文。

## 简单练习
[drill18](https://github.com/ZZy979/PPP-code/blob/main/ch18/drill18.cpp)

## 习题
* [18-1~18-7](https://github.com/ZZy979/PPP-code/blob/main/ch18/string_algorithm_exec18.cpp)
* [18-8](https://github.com/ZZy979/PPP-code/blob/main/ch18/palindrome.cpp)
* [18-9](https://github.com/ZZy979/PPP-code/blob/main/ch17/exec17-9.cpp)
