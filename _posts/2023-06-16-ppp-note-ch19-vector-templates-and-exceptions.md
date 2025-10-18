---
title: 《C++程序设计原理与实践》笔记 第19章 向量、模板和异常
date: 2023-06-16 00:30:58 +0800
categories: [C/C++, PPP]
tags: [cpp, vector, template, generic programming, concept, memory error, memory leak, raii]
---
本章将完成最常见、最有用的STL容器`vector`的设计与实现。我们将展示如何实现元素数量可变的容器，如何以参数形式指定容器的元素类型，以及如何处理越界错误。本章使用的技术依赖模板和异常，因此我们将介绍如何定义模板，并给出资源管理的基本技术，这些技术是正确使用异常的关键。

## 19.1 问题
在第18章结束时，我们的`vector`已经支持创建指定数量元素的`vector`、拷贝赋值和初始化、正确地释放内存，以及使用传统的下标语法访问元素（但是到目前为止仍然等价于固定大小的`double`数组）。

但是要达到我们期望的水平（基于使用标准库`vector`的经验），我们还需要解决三个问题：
* 如何改变`vector`的大小（改变元素数量）？
* 如何捕获和报告越界元素访问？
* 如何以参数形式指定元素类型？

即如何定义`vector`使得下面的操作是合法的：

```cpp
vector<double> vd;     // elements of type double
for (double d; cin>>d; )
    vd.push_back(d);   // grow vd to hold all the elements

vector<char> vc(100);  // elements of type char
int n;
cin>>n;
vc.resize(n);          // make vc have n elements
```

## 19.2 改变大小
标准库`vector`提供了三种改变大小的操作。对于

```cpp
vector<double> v(n);  // v.size()==n
```

有三种方式可以改变其大小：

```cpp
v.resize(10);    // v now has 10 elements
v.push_back(7);  // add an element with the value 7 to the end of v, v.size() increases by 1
v = v2;          // assign another vector; v is now a copy of v2, v.size() now equals v2.size()
```

标准库`vector`提供了更多改变大小的操作，例如`erase()`和`insert()`，但本章只考虑这三种。

### 19.2.1 表示
在实际中，我们通常会多次改变`vector`的大小。例如，我们很少见到只做一次`push_back()`。因此，`vector`会同时记录元素数量以及为“后续扩展”预留的“空闲空间”的数量：

```cpp
class vector {
    int sz;        // number of elements
    double* elem;  // address of first element
    int space;     // number of elements plus "free space"/"slots" for new elements ("the current allocation")
public:
    // ...
};
```

如下图所示：

![向量示意图](/assets/images/ppp-note-ch19-vector-templates-and-exceptions/向量示意图.png)

其中，[0, space)是`elem`的范围，[0, sz)是已有元素，[sz, space)是空闲位置。

默认构造函数（创建空向量）将整数成员置为0，指针成员置为`nullptr`：

```cpp
vector() :sz(0), elem(nullptr), space(0) {}
```

### 19.2.2 reserve和capacity
改变大小的最基本操作是`reserve()`，该操作用于增加向量的容量（不改变元素个数）：

```cpp
void vector::reserve(int new_space) {
    if (new_space <= space) return;     // never decrease allocation
    double* p = new double[new_space];  // allocate new space
    std::copy(elem, elem + sz, p);      // copy old elements
    delete[] elem;                      // deallocate old space
    elem = p;
    space = new_space;
}
```

注意，我们并不初始化空闲空间中的元素，因为这是`push_back()`和`resize()`的工作。

和标准库一样，我们提供了成员函数`capacity()`来获取向量的容量`space`：

```cpp
int vector::capacity() const { return space; }
```

也就是说，对于一个向量`v`，`v.capacity() - v.size()`（即`space - sz`）是在不重新分配空间的前提下可以通过`push_back()`添加的元素个数。

### 19.2.3 resize
有了`reserve()`，实现`resize()`是十分简单的。我们需要处理几种情况：
* new_size > capacity()
* size() < new_size ≤ capacity()
* new_size = size()
* new_size < size()

```cpp
void vector::resize(int new_size) {
    reserve(new_size);
    for (int i = sz; i < new_size; ++i) elem[i] = 0;    // initialize new elements
    sz = new_size;
}
```

我们让`reserve()`来完成处理内存的复杂工作，循环将初始化新元素（如果有）。

虽然`resize()`没有显式地处理每一种情况，但可以验证所有情况均被正确地处理了：

| 旧的`size()` | 旧的`capacity()` | `new_size` | 新的`size()` | 新的`capacity()` |
| --- | --- | --- | --- | --- |
| 10 | 16 | 20 | 20 | 20 |
| 10 | 16 | 15 | 15 | 16 |
| 10 | 16 | 10 | 10 | 16 |
| 10 | 16 | 5 | 5 | 16 |

注：如果new_size < 0，则`resize()`会使`size()` < 0。标准库`vector::resize()`的参数是无符号整数类型`size_t`，因此不存在这一问题。

### 19.2.4 push_back
有了`reserve()`之后，`push_back()`是非常简单的：

```cpp
void vector::push_back(double d) {
    if (space == 0) reserve(8);                // start with space for 8 elements
    else if (sz == space) reserve(2 * space);  // get more space
    elem[sz] = d;   // add d at end
    ++sz;           // increase the size (sz is the number of elements)
}
```

**当没有空闲空间时，将分配的空间增加一倍**。在实际中，这种策略被证明是一种非常好的选择，并且被标准库`vector`的大多数实现所采用。

注：这种策略使得向量的容量始终是2的幂。不发生扩容时`push_back()`的时间复杂度是O(1)，发生扩容时是O(n) = O(2<sup>k</sup>)，平摊到接下来2<sup>k</sup>-1次`push_back()`操作，使得每次操作的平均时间复杂度仍然是O(1)。

### 19.2.5 赋值
在向量赋值时，我们需要拷贝元素，但不需要考虑末尾的空闲空间。例如：

![向量赋值前](/assets/images/ppp-note-ch19-vector-templates-and-exceptions/向量赋值前.png)

赋值`v1=v2`之后：

![向量赋值后](/assets/images/ppp-note-ch19-vector-templates-and-exceptions/向量赋值后.png)

最简单的实现只需在原有拷贝赋值的基础上增加`space = sz;`即可。然而，如果`v1`的容量足以容纳`v2`的所有元素，就没有必要重新分配存储空间，只需要拷贝元素：

```cpp
vector& vector::operator=(const vector& v) {
    if (this == &v) return *this;   // self-assignment, no work needed

    if (v.sz <= space) {    // enough space, no need for new allocation
        std::copy(v.elem, v.elem + v.sz, elem);     // copy elements
        sz = v.sz;
        return *this;
    }

    double* p = new double[v.sz];           // allocate new space
    std::copy(v.elem, v.elem + v.sz, p);    // copy elements
    delete[] elem;      // deallocate old space
    elem = p;           // set new elements
    space = sz = v.sz;  // set new size
    return *this;       // return a self-reference
}
```

这里我们首先判断了自身赋值（例如`v=v`），这种情况下什么都不做。注意，判断`this==&v`和`v.sz<=space`只是为了优化，即使删除，代码仍然能够正确工作。

注意：拷贝构造函数需要相应修改，`elem`数组的大小必须和`space`保持一致，否则`push_back()`可能导致越界访问。

### 19.2.6 到目前为止我们的vector类
现在已经有了一个接近真实的`double`向量：

[简单向量v3 - 改变大小](https://github.com/ZZy979/PPP-code/blob/fcd8c5e87a415babb571cb412ba853c6f9925f8d/ch19/simple_vector.h)

## 19.3 模板
我们不仅仅需要`double`向量，而是希望能够自由指定元素类型。例如：

```cpp
vector<double>
vector<int>
vector<Month>
vector<Window*>         // vector of pointers to Windows
vector<vector<Record>>  // vector of vectors of Records
vector<char>
```

为此，我们必须知道如何定义模板。我们在4.6节就使用过模板，但到现在还没有定义过模板。

**模板**(template)是一种允许程序员使用类型或值参数化类或函数的机制，这些参数叫做**模板参数**(template parameter)，具有模板参数的类叫做**类模板**(class template)，具有模板参数的函数叫做**函数模板**(function template)。当我们提供具体类型或值作为参数时，编译器将生成一个特定的类或函数。

### 19.3.1 类型作为模板参数
我们希望将元素类型作为`vector`的参数，因此我们在`vector`类中将`double`替换为`T`，其中`T`是一个（类型）参数，可以被赋予`double`、`int`、`string`、`vector<Record>`和`Window*`之类的“值”。

在C++中，引入类型参数的语法是`template<class T>`或`template<typename T>`，其含义是“对于所有的类型`T`”。例如：

[简单向量v3 - 改为模板](https://github.com/ZZy979/PPP-code/blob/c996d5f164baa22b26cdf722e9c6323695e93c3b/ch19/simple_vector.h)

这里只是将19.2.6节的`vector`中的`double`替换为模板参数`T`（但对于实现类模板是不够的，见19.3.7节）。我们可以这样使用类模板`vector`：

```cpp
vector<double> vd;        // T is double
vector<int> vi;           // T is int
vector<double*> vpd;      // T is double*
vector<vector<int>> vvi;  // T is vector<int>, in which T is int
```

注：
* **类模板的成员函数也被认为是模板**，因此在类外定义时，必须使用`template`声明模板参数，并与类的模板参数一致。
* 在类模板内声明/定义成员函数时，参数类型或返回类型出现自身类型时可以省略模板参数；如果在类外定义，参数类型可以省略模板参数，而返回类型和`::`前不可省略。例如：

```cpp
template<class T>
class A {
public:
    A<T>& operator=(const A<T>& a);
};

template<class T>
A<T>& A<T>::operator=(const A<T>& a) {
    return *this;
}
```

可简化为：

```cpp
template<class T>
class A {
public:
    A& operator=(const A& a);
};

template<class T>
A<T>& A<T>::operator=(const A& a) {
    return *this;
}
```

当我们使用模板时，可以认为编译器用**模板实参**(template argument)（实际类型或值）替换模板参数生成了一个类或函数。对于`vector<double>`，编译器将会（大致）生成19.2.6节中的`vector`。

我们有时称类模板为**类型生成器**(type generator)。给定模板实参从模板生成类或函数的过程叫做**特化**(specialization)或**模板实例化**(template instantiation)。例如，`vector<double>`叫做`vector`的一个特化。模板实例化发生在编译时或链接时，而不是运行时。

自然地，我们可以使用类模板的成员函数。例如：

```cpp
void fct(vector<string>& v) {
    int n = v.size();
    v.push_back("Norah");
    // ...
}
```

编译器会根据模板定义

```cpp
template<class T> void vector<T>::push_back(const T& x) { /* ... */ };
```

生成函数

```cpp
void vector<string>::push_back(const string& x) { /* ... */ }
```

### 19.3.2 泛型编程
在C++中，模板是泛型编程的基础。

**泛型编程**(generic programming)：编写能够处理各种类型（作为参数）的代码，只要这些类型满足特定的语法和语义要求。

例如，`vector`的元素类型必须是可拷贝的（因为`push_back()`中的`elem[sz] = x;`是拷贝操作）。在第20和21章，我们将会看到要求参数支持算术运算的模板。

注：C++11之前，`std::vector`要求元素类型必须是可拷贝的。从C++11起，不同操作对元素类型有不同的要求。
* 构造函数（如`vector<T> v(n)`）：要求`T`满足[DefaultInsertable](https://en.cppreference.com/w/cpp/named_req/DefaultInsertable.html)（≈可默认构造）。
* 拷贝操作（如`vector<T> v = v2`或`v.push_back(x)`）：要求`T`满足[CopyInsertable](https://en.cppreference.com/w/cpp/named_req/CopyInsertable)（≈可拷贝构造）。
* 移动操作（如`vector<T> v = std::move(v2)`或`v.push_back(std::move(x))`）：要求`T`满足[MoveInsertable](https://en.cppreference.com/w/cpp/named_req/MoveInsertable)（≈可移动构造）。
* `emplace_back(args...)`：要求`T`满足[EmplaceConstructible](https://en.cppreference.com/w/cpp/named_req/EmplaceConstructible)和MoveInsertable，这是因为插入元素可能导致扩容，需要拷贝/移动已有元素。
* 扩容操作：C++11之前没有移动操作，因此总是使用拷贝操作；从C++11起，如果移动操作可用（定义了且为`noexcept`）则优先使用移动操作，否则退回到拷贝操作。
* 例如：
  * 如果类`C`没有默认构造函数，则可以定义`vector<C> v`，但不能`vector<C> v(5)`。
  * 如果类`C`可移动、不可拷贝，则不能`v.push_back(c)`，但可以`v.push_back(C())`或`v.emplace_back()`。
  * 如果类`C`可拷贝、不可移动，则可以`v.push_back(c)`，而`v.push_back(C())`仍然会调用拷贝构造函数。
  * 如果类`C`可默认构造，但不可拷贝、不可移动，则可以`vector<C> v(5)`，但不能`v.push_back(c)`或`v.emplace_back()`。
* 这些编译错误发生在模板实例化时，而不是编译模板本身的代码时，因为模板作者并不知道用户会提供什么样的元素类型。在这些情况下，可以用智能指针包装元素，例如`vector<shared_ptr<C>>`。

依赖于显式模板参数的这种泛型编程通常叫做**参数化多态**(parametric polymorphism)。相比之下，从类层次结构和虚函数中获得的多态叫做**专用多态**(ad hoc polymorphism)，而这种编程风格叫做面向对象编程。这两种编程风格都叫做“多态”的原因在于二者都依赖程序员通过一个单一的接口提供一个概念的多个版本。多态在希腊语中是“多种形状”(many shapes)的意思，是指你可以通过一个公共的接口操作很多不同的类型。在第16~19章`Shape`的例子中，我们可以通过`Shape`定义的接口访问多种形状（例如`Text`、`Circle`、`Polygon`等）。当我们使用`vector`时，我们通过`vector`模板定义的接口使用不同类型的`vector`（例如`vector<int>`、`vector<double>`和`vector<Shape*>`）。

面向对象编程和泛型编程最明显的差异是：使用泛型编程时，被调用函数的选择是在编译时决定的，而对于面向对象编程是在运行时决定的。例如：

```cpp
void f(vector<string>& v, Shape& s) {
    v.push_back(x);  // put x into the vector v
    s.draw();        // draw the shape s
}
```

对于`v.push_back(x)`，编译器将根据`v`的元素类型调用对应的`push_back()`；而对于`s.draw()`，编译器会间接地调用某个`draw()`函数（使用`s`的vtbl）。

总结：
* 泛型编程：由模板支持，依赖编译时解析。
* 面向对象编程：由类层次结构和虚函数支持，依赖运行时解析。

将二者结合是可能的也是有用的。例如：

```cpp
void draw_all(vector<Shape*>& v) {
    for (int i = 0; i < v.size(); ++i) v[i]->draw();
}
```

这里，调用虚函数`draw()`是面向对象编程，而使用模板`vector`是泛型编程。

模板具有极大的灵活性、近似最优的性能等优点。然而，模板的主要问题在于声明和定义不能很好地分离。当编译使用模板的代码时，编译器通常要求模板在被使用的位置完全定义，包括所有成员函数及其调用的模板函数。因此，**必须将模板的定义放在头文件中**（无论成员函数在类内还是类外定义）。

注：下面是来自[cppreference](https://en.cppreference.com/w/cpp/language/templates)的解释：

> The definition of a template must be visible at the point of implicit instantiation, which is why template libraries typically provide all template definitions in the headers.

### 19.3.3 概念
在编写模板时，有时需要对模板参数进行约束。例如：

```cpp
template<class T>  // requires T to be addable
T add(T a, T b) { return a + b; }
```

其中，函数模板`add()`要求模板参数`T`必须支持`+`运算，但只是通过注释以文字形式说明。如果使用模板`add()`时给定的模板参数不支持`+`运算，将导致编译错误：

```cpp
int i = add(1, 2);          // OK, returns 3
double d = add(1.2, 3.4);   // OK, returns 4.6
string s = add(string("abc"), string("def"));       // OK, returns "abcdef"
const char* c = add("abc", "def");                  // error: no operator+ for const char* and const char*
vector<int> v = add(vector<int>(), vector<int>());  // error: no operator+ for vector<int> and vector<int>
```

C++标准定义了一些常用的**具名要求**(named requirement)，例如“可默认构造”(DefaultConstructible)、“可调用”(Callable)等，完整列表见[Named Requirements - cppreference](https://en.cppreference.com/w/cpp/named_req)。但是具名要求仍然局限于自然语言描述。

C++20引入了一个新的语言特性——概念。**概念**(concept)是对模板参数的一组命名的约束/要求，是一种在编译时求值的类型断言，以编译器可理解的方式提供了一种模板参数检查机制。当模板参数不满足要求时，编译器将给出更加明确的错误信息。

例如，对于上面例子中“支持`+`运算”这一约束，可以定义概念`Addable`：

```cpp
template<class T>
concept Addable = requires(T x) { x + x; };
```

其中，`=`后面的部分叫做**requires表达式**，`{}`中的一个或多个语句用于断言这些表达式是合法的，即能够编译通过（并不真正求值）。如果模板参数满足所有的要求，则requires表达式结果为`true`。

在定义模板时，可以像这样使用概念：

```cpp
template<class T>
    requires Addable<T>
T add(T a, T b) { return a + b; }

template<Addable T>
T add(T a, T b) { return a + b; }
```

这两种形式是等价的，意味着“要求模板参数`T`必须满足概念`Addable`”。其中，第一种形式中的`requires Addable<T>`叫做**requires子句**。

标准库头文件[\<concepts\>](https://en.cppreference.com/w/cpp/header/concepts)定义了一组常用的概念。

注：
* C++11新增的标准库头文件[\<type_traits\>](https://en.cppreference.com/w/cpp/header/type_traits)也定义了一组用于断言模板参数是否满足特定要求的模板，与概念类似。这些模板可用于定义概念，例如：

```cpp
template<class T>
concept integral = std::is_integral<T>::value;

template<class T>
concept signed_integral = std::is_integral<T>::value && std::is_signed<T>::value;
```

* 可以通过逻辑运算符组合概念来定义新的概念，例如：

```cpp
template<class T>
concept unsigned_integral = std::integral<T> && !std::signed_integral<T>;
```

### 19.3.4 容器和继承
将派生类对象的容器赋给基类对象的容器是错误的。例如：

```cpp
vector<Shape> vs;
vector<Circle> vc;
vs = vc;  // error: vector<Shape> required

void f(vector<Shape>&);
f(vc);    // error: vector<Shape> required
```

因为将一个`Circle`对象`push_back()`到`vector<Shape>`中会发生截断（见14.2.4节）。

注：
* `Shape`类禁止拷贝，因此将一个`Shape`对象`push_back()`到`vector<Shape>`中也是错误的。另外，`Shape`是抽象类，根本不能创建`Shape`对象，因此不能使用`vector<Shape>`的构造函数和`resize()`等操作。总之，对于`vector<Shape>`，除了声明一个空向量以外什么都不能做，因此使用这个类型是没有意义的。
* 即使不存在抽象类和禁止拷贝问题，将`vector<Circle>`赋给`vector<Shape>`仍然是非法的，因为拷贝赋值在拷贝元素时也会发生截断；将`vector<Circle>`赋给`vector<Shape>&`也是非法的，因为`Circle`对象比`Shape`对象大，在`push_back()`时会发生截断，元素位置是完全错位的：

![Shape和Circle向量](/assets/images/ppp-note-ch19-vector-templates-and-exceptions/Shape和Circle向量.png)

使用指针再次尝试：

```cpp
vector<Shape*> vps;
vector<Circle*> vpc;
vps = vpc;  // error: vector<Shape*> required

void f(vector<Shape*>&);
f(vpc);    // error: vector<Shape*> required
```

将`vector<Circle*>`赋给`vector<Shape*>`仍然是非法的。考虑`f()`可能会做什么：

```cpp
void f(vector<Shape*>& v) {
    v.push_back(new Rectangle(Point(0,0),Point(100,100)));
}
```

显然，我们可以将一个`Rectangle*`放入`vector<Shape*>`。但是，如果这个`vector<Shape*>`在其他地方被认为是一个`vector<Circle*>`，将会得到糟糕的、意外的结果。假设上面的例子能够编译通过，使用者认为通过`vpc`访问的一定是`Circle`对象，而实际上可能是一个`Rectangle`：

![Circle*向量](/assets/images/ppp-note-ch19-vector-templates-and-exceptions/Circle指针向量.png)

总之：对于任意模板`C`，“`D`是`B`”并不意味着“`C<D>`是`C<B>`”或“`C<D*>`是`C<B*>`”。

### 19.3.5 整数作为模板参数
除了类型，整数也可以作为模板参数。

下面考虑一个整数作为模板参数最常见的例子，元素个数在编译时已知的容器（数组）：

[简单数组](https://github.com/ZZy979/PPP-code/blob/main/ch19/simple_array.h)

注：
* 标准库提供了具有相同功能的`std::array`，定义在头文件\<array\>中。
* 模板参数是类型的一部分，因此`array<int, 5>`和`array<int, 10>`是两种不同的类型。
* 由于`array`的大小`N`在编译时是已知的，因此不需要使用自由存储，而是直接将`elem`成员声明为数组；相反，`vector`需要使用自由存储，`elem`成员只是一个指针。因此，`array`的元素是其本身的一部分，而`vector`的元素不是（见17.3.1节）：
    * `sizeof(array<int, 100>) = 100 * sizeof(int) = 400`
    * `sizeof(vector<int>(100)) = sizeof(int) + sizeof(int*) + sizeof(int) = 4 + 4 + 4 = 12`（对于19.2.6节中的`vector`、32位系统）。
* `array`支持列表初始化，是因为它只有一个数组类型的公有成员，见[Aggregate initialization](https://en.cppreference.com/w/cpp/language/aggregate_initialization)。
* `array`的默认构造函数、拷贝构造函数、拷贝赋值和析构函数都是由编译器自动生成的。由于`elem`是数组而不是指针，因此默认拷贝操作就是深拷贝，析构函数什么都不用做。
* **内置数组本身不支持拷贝，但作为对象成员可以通过对象间接拷贝。** 例如：

```cpp
#include <iostream>

struct Foo {
    char s[8];
};

struct Bar {
    char* s;
};

int main() {
    Foo f1{"hello"}, f2 = f1;
    Bar b1{new char[]{"hello"}}, b2 = b1;
    f1.s[0] = b1.s[0] = '\0';
    std::cout << "f2.s = " << f2.s << '\n'
            << "b2.s = " << b2.s << '\n';
    delete[] b1.s;
    return 0;
}
```

程序输出

```
f2.s = hello
b2.s = 
```

![数组作为成员拷贝-修改前](/assets/images/ppp-note-ch19-vector-templates-and-exceptions/数组作为成员拷贝-修改前.png)

![数组作为成员拷贝-修改后](/assets/images/ppp-note-ch19-vector-templates-and-exceptions/数组作为成员拷贝-修改后.png)

我们可以这样使用`array`：

```cpp
array<double, 6> ad = {0.0, 1.1, 2.2, 3.3, 4.4, 5.5};  // initialize

void some_fct(int n) {
    array<int, 4> loc = {0, 1, 2, 3};
    array<char, n> oops;       // error: the value of n not known to compiler
    array<int, 4> loc2 = loc;  // copy constructor
    loc = loc2;      // copy assignment
    int x = loc[2];  // element access
    loc[3] = 42;
    // loc and loc2 automatically destroyed
}
```

显然，`array`非常简单，也不如`vector`强大。那么，为什么会有人希望使用`array`而不是`vector`呢？一种答案是“效率”。`array`的大小在编译时是已知的，因此编译器可以分配静态内存（对于全局对象）或栈内存（对于局部对象），而不是使用自由存储。

反过来，为什么不直接使用内置数组呢？如18.6节所述，内置数组很难用：不知道自身的大小，动不动就转换为指针，不能拷贝和赋值。而`array`不存在这些问题。例如：

```cpp
double* p = ad;         // error: no implicit conversion to pointer
double* q = ad.data();  // OK: explicit conversion

template<class C>
void printout(const C& c) {  // function template
    for (int i = 0; i < c.size(); ++i) cout << c[i] << '\n';
}
```

`printout()`对`array`和`vector`都可以调用，因为用到的接口（`size()`和`[]`）对于二者是相同的。这是一个将泛型编程用于数据访问的简单例子，第20和21章将会详细介绍这种编程风格。

### 19.3.6 模板参数推导
对于类模板，当你创建特定类型的对象时需要指定模板参数。例如：

```cpp
array<char, 1024> buf;  // for buf, T is char and N is 1024
array<double, 10> b2;   // for b2, T is double and N is 10
```

**对于函数模板，编译器通常能够从函数参数中推断出模板参数**（详见[Template argument deduction](https://en.cppreference.com/w/cpp/language/template_argument_deduction)）。例如：

```cpp
template<class T, int N>
void fill(array<T, N>& a, const T& val) {
    for (int i = 0; i < N; ++i) a[i] = val;
}

void f() {
    fill(buf, 'x');  // for fill(), T is char and N is 1024 because that's what buf has
    fill(b2, 0.0);   // for fill(), T is double and N is 10 because that's what b2 has
}
```

`fill(buf, 'x')`是`fill<char, 1024>(buf, 'x')`的简写，`fill(b2, 0.0)`是`fill<double, 10>(b2, 0.0)`的简写。幸运的是，我们通常不必写得这么具体，编译器会自动为我们推断出来。

### 19.3.7 一般化vector类
当我们将“`double`向量”一般化为模板“`T`向量”时，并没有重新考虑`push_back()`、`resize()`和`reserve()`的实现。在19.2.2和19.2.3节中定义这些函数时做了一些假设，这些假设对`double`成立，但并不是对所有`vector`元素类型都成立：
* **如果`X`没有默认值，如何处理`vector<X>`？**（构造函数和`resize()`涉及使用默认值初始化，而`double`有默认值0.0）
* 如何确保元素在用完后被销毁？（已经能够确保：拷贝赋值、移动赋值、析构函数和`reserve()`使用`delete[]`释放数组时会自动调用每个元素的析构函数。但是如果要实现`pop_back()`、`erase()`等只删除元素、不释放数组的操作，则必须显式调用元素的析构函数。）

对于没有默认值的类型，可以增加一个参数，允许用户指定“默认值”：

```cpp
void resize(int new_size, const T& v = T());
```

也就是说，使用`T()`作为默认值，除非用户指定了其他值（如果类型`T`没有默认值，则用户必须指定一个值，否则将编译失败）。例如：

```cpp
vector<double> v1;
v1.resize(100);       // add 100 copies of double(), that is, 0.0
v1.resize(200, 0.0);  // add 100 copies of 0.0 - mentioning 0.0 is redundant
v1.resize(300, 1.0);  // add 100 copies of 1.0

struct No_default {
    No_default(int);  // the only constructor for No_default
    // ...
};

vector<No_default> v2(10);      // error: tries to make 10 No_default()s
vector<No_default> v3;
v3.resize(100, No_default(2));  // add 100 copies of No_default(2)
v3.resize(200);                 // error: tries to add 100 No_default()s
```

但是，即使给构造函数和`resize()`增加了“默认值”参数，对于`No_default`仍然会报错。因为构造函数和`reserve()`中的 **`new[]`会强制对所有元素进行初始化，而这对于没有默认值的类型是不可行的**（见17.4.4节）。

另一方面，**空闲空间（即`elem[sz:space)`）中的元素不需要初始化**，因为它们不属于`vector`的有效范围，对其初始化会带来不必要的开销。我们需要一种同时包含已初始化数据和未初始化数据的数据结构（如下图所示）。作为`vector`的实现者，我们不得不面对这一问题。

![已初始化&未初始化](/assets/images/ppp-note-ch19-vector-templates-and-exceptions/已初始化和未初始化.png)

基于以上两个原因，我们需要一种能够分配和操作**未初始化内存**的方法。C++标准库头文件\<memory\>提供了`allocator`类（分配器），能够提供未初始化内存。下面是其简化版本：

```cpp
template<class T>
class allocator {
public:
    // ...

    // allocate uninitialized storage for n objects of type T
    T* allocate(int n) { return static_cast<T*>(operator new(n * sizeof(T))); }

    // deallocate n objects of type T starting at p
    void deallocate(T* p, int n) { operator delete(p, n * sizeof(T)); }

    // construct a T with the value v in p
    void construct(T* p, const T& v) { new(p) T(v); }

    // destroy the T in p
    void destroy(T* p) { p->~T(); }
};
```

这里列出了4个基本操作：
* 分配能够容纳n个`T`类型对象（即`n * sizeof(T)`字节）的未初始化内存空间（不调用构造函数）
* 在未初始化内存中构造一个`T`类型对象
* 销毁一个`T`类型对象，将其所在内存返回到未初始化状态
* 释放能够容纳n个`T`类型对象的内存空间（不调用析构函数）

例如：

```cpp
struct C {
    C(int x) {}
    ~C() {}
};

void f() {
    allocator<C> alloc;
    int n = 3;
    C* p = alloc.allocate(n);

    for (int i = 0; i < n; ++i)
        alloc.construct(p + i, i);  // calls C(int)

    for (int i = 0; i < n; ++i)
        alloc.destroy(p + i);       // calls ~C()

    alloc.deallocate(p, n);
}
```

注：
* C++20将`allocator`的`construct()`和`destroy()`函数移至`allocator_traits`类，后者分别调用`std::construct_at()`和`std::destroy_at()`。
* 实际上`allocator::allocate()`底层就是使用`malloc()`函数实现的，使用该函数代替`allocator`也能解决上述问题。使用`allocator`的另一个原因是：将内存分配/释放与对象构造/销毁分离，并抽象为接口，从而支持自定义内存管理策略，提供更好的灵活性。
* 除了`allocator`外，标准库头文件\<cstdlib\>提供了`malloc()`函数，\<new\>提供了一组`operator new`和`operator new[]`函数，都能够分配未初始化内存。这些函数与`new`/`new[]`运算符的区别和联系：
    * `new`和`new[]`叫做[new表达式](https://en.cppreference.com/w/cpp/language/new)(new expression)，是语言级别的运算符；`operator new`和`operator new[]`叫做[分配函数](https://en.cppreference.com/w/cpp/memory/new/operator_new)(allocation functions)，是标准库头文件\<new\>中定义的一组库函数。
    * `new`在分配内存后会调用构造函数来初始化对象，而`operator new`和`malloc()`只负责分配内存。
    * `new`通过调用`operator new`分配内存，而`operator new`就是通过调用`malloc()`函数实现的。
    * `new(p) T(args...)`这种特殊形式的`new`叫做[Placement new](https://en.cppreference.com/w/cpp/language/new#Placement_new)，并不分配内存，而是在已分配的未初始化内存空间中（原地）构造对象——在C++中这是在不分配内存的情况下调用构造函数的唯一方式。
* 下表总结了在自由存储上分配内存/初始化对象的各种方式：

| 表达式 | 分配内存（字节） | 初始化对象 |
| --- | --- | --- |
| `new T(args...)` | `sizeof(T)` | 是 |
| `new T[n]{...}` | `n * sizeof(T)` | 是 |
| `allocator<T>::allocate(n)` | `n * sizeof(T)` | 否 |
| `operator new(n)`或`malloc(n)` | `n` | 否 |
| `allocator<T>::construct(p, args...)` | 无 | 是 |
| `new(p) T(args...)` | 无 | 是 |

* 相应地，`delete`和`delete[]`会自动调用析构函数并释放内存；而`operator delete`和`free()`只负责释放内存，之前必须手动调用析构函数（`allocator<T>::destroy(p)`或`p->~T()`）。

（回到正题）`allocator`正是我们实现`vector<T>::reserve()`所需要的。首先给`vector`增加一个模板参数`A`（用于支持自定义分配器）：

```cpp
template<class T, class A = allocator<T>>
class vector {
    A alloc;  // use allocate to handle memory for elements
    // ...
};
```

注：与函数的默认参数一样，模板也支持默认参数，并且只需要在声明中写出。

`vector`中所有直接处理内存（使用`new`和`delete`）的成员函数都需要修改：

[简单向量v3 - 使用allocator](https://github.com/ZZy979/PPP-code/blob/e28699a7dfa06577062b89f00f99a2818f0ea089/ch19/simple_vector.h)

在`reserve()`中，通过`alloc.construct(&p[i], elem[i])`拷贝旧元素时，间接调用了元素的**拷贝构造**函数（这意味着`vector`的**元素类型必须是可拷贝构造的**）。这里**不能使用赋值** (`p[i] = elem[i]`)，因为对于像`vector`和`string`这样的类型，赋值假设目标对象是已初始化的（赋值操作会先销毁旧的元素并释放数组），而`p[i]`是未初始化的（相当于对野指针进行操作，这是十分危险的）。

在`resize()`中，将`vector`缩小时，还需要考虑多余元素的销毁。这里把析构函数看成将具有类型的对象变成“原始内存”。

注：
* 使用`allocator`时，必须将内存分配/释放与对象构造/析构分开考虑，要想清楚内存空间是已初始化的还是未初始化的，应该对元素赋值还是构造，元素是否需要销毁等。
* 书中并未给出修改后的构造函数、拷贝操作和移动操作，而使用`allocator`正确地实现这些操作并非易事。在纸上画出元素在内存中的示意图并写出伪代码会很有帮助。例如：

（1）`reserve()`

![reserve()](/assets/images/ppp-note-ch19-vector-templates-and-exceptions/reserve.png)

（2）`resize()`

![resize()](/assets/images/ppp-note-ch19-vector-templates-and-exceptions/resize.png)

（3）`push_back()`

![push_back()](/assets/images/ppp-note-ch19-vector-templates-and-exceptions/push_back.png)

（4）构造函数

![构造函数](/assets/images/ppp-note-ch19-vector-templates-and-exceptions/构造函数.png)

（5）拷贝构造

![拷贝构造](/assets/images/ppp-note-ch19-vector-templates-and-exceptions/拷贝构造.png)

（6）拷贝赋值

![拷贝赋值](/assets/images/ppp-note-ch19-vector-templates-and-exceptions/拷贝赋值.png)

### 小结

| 操作 | 错误实现 | 正确实现 | 原因 |
| --- | --- | --- | --- |
| 构造函数 | `elem = new T[n]` | `elem = malloc(n * sizeof(T))` | `new[]`会强制对所有元素进行初始化，对于不可默认构造的类型无法使用 |
| `push_back(v)` | `elem[sz++] = v` | `new(&elem[sz++]) T(v)` (placement new) | 赋值操作假设目标对象是已初始化的，而`elem[sz]`是未初始化的 |
| 拷贝构造函数 | `new[]` + `std::copy()` | `malloc()` + `std::uninitialized_copy()` | 同上 |
| `pop_back()` | `--sz` | `elem[--sz].~T()` | 删除元素时必须销毁对象 |
| 析构函数 | `delete[] elem` | 逐元素`~T()` + `free(elem)` | 同上 |

## 19.4 范围检查和异常
目前的`vector`没有对元素访问做范围检查。例如：

```cpp
vector<int> v(100);
v[-200] = v[200];  // oops!
int i;
cin>>i;
v[i] = 999;        // maul an arbitrary memory location
```

这段代码能够编译运行，并访问不属于`vector`的内存，这可能会造成严重的问题（见17.4.3节）（按照C++标准的说法，这属于未定义行为）。为了解决这一问题，最简单的方法是增加一个带范围检查的访问操作`at()`：

[简单向量v3 - 增加at()操作](https://github.com/ZZy979/PPP-code/blob/1998a8b06470c11fbdf2422ea9753fe3d6fe54e4/ch19/simple_vector.h)

如果下标越界则抛出`out_of_range`异常。于是可以编写以下代码：

```cpp
void print_some(vector<int>& v) {
    int i = -1;
    while(cin>>i && i!=-1)
    try {
        cout << "v[" << i << "]==" << v.at(i) << "\n";
    }
    catch(out_of_range) {
        cout << "bad index: " << i << "\n";
    }
}
```

通常做法是：如果确定索引是有效的则使用`[]`，否则使用`at()`。

注：
* 标准库`vector`也是`[]`不做范围检查，`at()`做范围检查。
* 可以通过自己对下标进行检查来避免处理异常，例如`if (i >= 0 && i < v.size()) {...} else {...}`
* 虽然`const`和非`const`版本的`at()`的实现看起来完全一样，但不能直接在一个中调用另一个：前者不能调用后者，因为不能直接将`const T&`转换为`T&`；后者不能调用前者，因为不能对`const`对象调用非`const`成员函数。如果想消除重复代码，唯一的方法是使用`const_cast`，即将非`const`版本实现为：`return const_cast<T&>(static_cast<const vector<T>&>(*this).at(i));`。参考：[How do I remove code duplication between similar const and non-const member functions?](https://stackoverflow.com/questions/123758/how-do-i-remove-code-duplication-between-similar-const-and-non-const-member-func)

### 19.4.1 题外话：设计上的考虑
那么，我们的`vector`（以及标准库`vector`）为什么不直接给`[]`添加范围检查呢？主要有4个方面的理由：
* 兼容性：早在C++有异常机制之前，人们就已经在使用不做检查的下标操作。
* 效率：不做检查的运算符性能更优，并且可以在其基础上实现范围检查，但反之不然。
* 约束：在某些环境中，异常是不可接受的。
* 检查的可选性：C++标准仅仅指出，越界访问`vector`是未定义行为。如果需要范围检查，可以自己实现。

### 19.4.2 忏悔：宏
（没看懂这一节在讲什么……）

## 19.5 资源和异常
编程的一个基本原则是，**如果我们获取了资源，则必须（直接或间接地）将其归还给管理这些资源的系统**。资源的例子包括：动态内存、锁、文件、线程、套接字等。

最简单的例子是自由存储内存，使用`new`获取、使用`delete`释放。例如：

```cpp
void suspicious(int s, int x) {
    int* p = new int[s];  // acquire memory
    // ...
    delete[] p;           // release memory
}
```

如17.4.6节所述，我们必须记得释放内存，但这并不总是容易做到。当我们使用异常时，资源泄露可能会变得更为普遍。特别地，我们要对像`suspicious()`这种显式使用`new`并将返回的指针赋给局部变量的代码保持怀疑。

我们将负责释放内存的对象（例如`vector`）称为资源的**拥有者**(owner)或**句柄**(handle)。

### 19.5.1 潜在的资源管理问题
虽然`suspicious()`有`delete[] p;`语句，但在某些情况下释放并不会发生。在`...`部分放入什么代码会导致内存泄露呢？

（1）**改变指向**，当程序运行到`delete`时，`p`可能不再指向之前的对象：

```cpp
void suspicious(int s, int x) {
    int* p = new int[s];  // acquire memory
    // ...
    if (x) p = q;         // make p point to another object
    // ...
    delete[] p;           // release memory
}
```

`if (x)`使你不能确定是否改变了`p`的值。

（2）**中途返回**，程序可能永远不会运行到`delete`：

```cpp
void suspicious(int s, int x) {
    int* p = new int[s];  // acquire memory
    // ...
    if (x) return;
    // ...
    delete[] p;           // release memory
}
```

（3）**抛出异常**，程序可能因为抛出异常而永远不会运行到`delete`：

```cpp
void suspicious(int s, int x) {
    int* p = new int[s];  // acquire memory
    vector<int> v;
    // ...
    if (x) p[x] = v.at(x);
    // ...
    delete[] p;           // release memory
}
```

捕获异常可以解决这一问题：

```cpp
void suspicious(int s, int x) {  // messy code
    int* p = new int[s];         // acquire memory
    vector<int> v;
    // ...
    try {
        if (x) p[x] = v.at(x);
        // ...
    } catch (. . .) {            // catch every exception
        delete[] p;              // release memory
        throw;                   // re-throw the exception
    }
    // ...
    delete[] p;                  // release memory
}
```

但这种解决方案的代价是造成资源释放代码的重复（写了两次`delete[] p;`）。更糟糕的是，这种方案不能通用。考虑获取更多资源的例子：

```cpp
void suspicious(vector<int>& v, int s) {
    int* p = new int[s];
    vector<int> v1;
    // ...
    int* q = new int[s];
    vector<double> v2;
    // ...
    delete[] p;
    delete[] q;
}
```

注意，**如果`new`分配内存失败，将抛出标准库异常`bad_alloc`**。对于这个例子，`try-catch`也可以解决问题，但需要添加多个`try`块，使代码变得重复且丑陋，难以阅读和维护。

试一试 在上面最后的例子中添加`try`块，以保证在可能抛出异常的所有情况下，所有资源都能被正确地释放。

```cpp
void suspicious(vector<int>& v, int s) {
    int* p = new int[s];
    vector<int> v1;
    try {
        // ...
    }
    catch (...) {
        delete[] p;
        throw;
    }
    int* q;
    try {
        q = new int[s];
    }
    catch (bad_alloc) {
        delete[] p;
        throw;
    }
    vector<double> v2;
    try {
        // ...
    }
    catch (...) {
        delete[] p;
        delete[] q;
        throw;
    }
    delete[] p;
    delete[] q;
}
```

注：如果C++的`try`语句有`finally`子句，则上面的代码会简单得多。

### 19.5.2 资源获取即初始化(RAII)
对于上面的例子，使用`vector`就不需要添加复杂的`try-catch`语句来处理潜在的资源泄露。例如：

```cpp
void f(vector<int>& v, int s) {
    vector<int> p(s);
    vector<int> q(s);
    // ...
}
```

**在对象的构造函数中获取资源，在对应的析构函数中释放资源**（换句话说，将资源的获取和释放与对象的生命周期绑定）——这一解决方法是通用的，适用于所有类型的资源。这一技术通常称为“**资源获取即初始化**”([Resource Acquisition Is Initialization](https://en.cppreference.com/w/cpp/language/raii), RAII)。

考虑上面的例子，无论以何种方式离开`f()`（即使`...`部分有`return`语句或抛出异常），`p`和`q`的析构函数都会被调用；如果有赋值语句，则由赋值操作释放内存。这一通用规则始终成立：**当程序执行离开作用域时，每个对象和子对象的析构函数将被调用。**

### 19.5.3 保证
有时我们不能将`vector`保持在单一作用域（及其子作用域）内。例如：

```cpp
vector<int>* make_vec() {              // make a filled vector
    vector<int>* p = new vector<int>;  // we allocate on free store
    // ... fill the vector with data; this may throw an exception . . .
    return p;
}
```

这个例子是一种常见的用法：**调用函数构造一个复杂的数据结构，并将其作为结果返回**（18.3.4节也给出了一个这种例子）。

问题在于，如果在填充`vector`时发生了异常，`make_vec()`将会泄露这个`vector`对象的内存。另一个问题是，如果函数成功了，调用者必须记得`delete`该函数返回的对象（见17.4.6节）。

我们可以添加`try`块来处理可能抛出的异常：

```cpp
vector<int>* make_vec() {              // make a filled vector
    vector<int>* p = new vector<int>;  // we allocate on free store
    try {
        // fill the vector with data; this may throw an exception
        return p;
    }
    catch (...) {
        delete p;  // do our local cleanup
        throw;     // re-throw to allow our caller to deal with the fact
                   // that make_vec() couldn't do what was required of it
    }
}
```

该函数展示了一种非常常见的错误处理方式：**函数试图完成它的工作，如果不能，则清理所有局部资源，并通过抛出异常来表示失败**。这是一种简单而有效的错误处理方式，并且能够被系统地使用：
* **基本保证**(basic guarantee)：函数要么成功，要么抛出异常而不泄露任何资源。
* **强保证**(strong guarantee)：除了提供基本保证外，函数还确保所有可观测值（非局部变量）在失败后与调用函数时的值相同。
* **无抛出保证**(no-throw guarantee)：基本上C++所有内置功能都提供了无抛出保证——根本不会抛出异常。为了避免抛出异常，应该避免使用`throw`、`new`和引用类型的`dynamic_cast`。

注：参见[Exception safety](https://en.cppreference.com/w/cpp/language/exceptions.html#Exception_safety)。

基本保证和强保证对于思考程序的正确性是十分有用的。RAII对于根据这些思想编写简单的、高性能的代码是必不可少的。

我们应该始终避免未定义行为（通常是灾难性的），例如对空指针解引用、除以0、对数组越界访问。捕获异常并不能保证你不违反基本的语言规则。

注：对于前面提到的“调用者必须记得`delete`由`make_vec()`返回的对象”这一问题，有三种解决方法：
* 使用移动语义（见18.3.4和19.5.5节）
* 依赖拷贝消除（见[【C++】右值引用、移动语义和完美转发]({% post_url 2023-06-02-cpp-rvalue-reference-move-semantics-and-perfect-forwarding %}) 3.3节）
* 使用`unique_ptr`或`shared_ptr`（见下一节）

### 19.5.4 unique_ptr
虽然`make_vec()`提供了基本保证，但`try-catch`代码仍然是丑陋的。解决方法很明显：我们必须使用RAII。也就是说，我们需要一个能够保存`vector<int>`的对象，当异常发生时它能够删除`vector`对象。标准库头文件\<memory\>提供了`unique_ptr`：

```cpp
unique_ptr<vector<int>> make_vec() {             // make a filled vector
    unique_ptr<vector<int>> p(new vector<int>);  // allocate on free store
    // ... fill the vector with data; this may throw an exception ...
    return p;
}
```

`unique_ptr`是一个能够保存指针的对象，使用`new`返回的指针初始化，并且可以像内置指针一样使用`->`和`*`（例如`p->at(2)`或`(*p).at(2)`），因此我们将`unique_ptr`认为是一种指针。然而，**`unique_ptr`拥有被指向对象的（唯一）所有权**：当`unique_ptr`被销毁时，它将`delete`被指向的对象（注：这与`vector`的析构函数释放底层数组一样，都是RAII的思想）。

`unique_ptr`的使用极大地简化了`make_vec()`，并且再次印证了19.5节所说的“对显式使用`new`和`try-catch`的代码保持怀疑”——大部分都可以被替换为RAII的某种变体。

另外，让`make_vec()`返回`unique_ptr<vector<int>>`而不是`vector<int>*`也省去了调用者`delete`指针的麻烦。

注意，`unique_ptr`是不可拷贝的，但是可移动，这是由其“唯一性”这一固有属性决定的（对于智能指针，拷贝=共享所有权，移动=转交所有权）。如果需要可拷贝的智能指针，则使用`shared_ptr`，它通过引用计数来保证：当指向同一个对象的最后一个`shared_ptr`被销毁时，删除被指向的对象。

与普通指针相比，`unique_ptr`并没有额外开销(`sizeof(unique_ptr<T>) == sizeof(T*)`)。

注：
* `unique_ptr`的默认构造函数创建一个空指针。
* C++14引入了`make_unique()`函数，用于创建`unique_ptr`，其参数将被转发给对象的构造函数，即`make_unique<T>(args...)`等价于`unique_ptr(new T(args...))`。

### 19.5.5 通过移动返回
幸运的是，`vector`的移动操作也能够解决该问题：

```cpp
vector<int> make_vec() {  // make a filled vector
    vector<int> res;
    // ... fill the vector with data; this may throw an exception ...
    return res;           // the move constructor efficiently transfers ownership
}
```

最后这个版本的`make_vec()`是最简单的，也是我们推荐的。使用移动的解决方法对于所有容器和所有类型的资源都是通用的。

### 19.5.6 vector的RAII
考虑19.3.7节中的`reserve()`，注意旧元素的拷贝操作`alloc.construct(&p[i], elem[i])`可能会抛出异常，此时`p`就是19.5.1节所述问题的例子。我们可以使用`unique_ptr`，但更好的解决方法是将“`vector`所占内存”认为是一种资源，定义一个`vector_base`类来表示这一基本概念：

```cpp
template<class T, class A>
struct vector_base {
    A alloc;    // allocator
    T* elem;    // start of allocation
    int sz;     // number of elements
    int space;  // amount of allocated space

    vector_base(const A& a, int n)
            :alloc(a), elem(alloc.allocate(n)), sz(n), space(n) {}
    ~vector_base() { alloc.deallocate(elem, space); }
};
```

基本上，`vector`就是`vector_base`的便捷接口：

```cpp
template<class T, class A = allocator<T>>
class vector : private vector_base<T, A> {
public:
    // ...
};
```

之后可以重写`reserve()`：

```cpp
template<class T, class A>
void vector<T, A>::reserve(int newalloc) {
    if (newalloc <= space) return;         // never decrease allocation
    vector_base<T, A> b(alloc, newalloc);  // allocate new space
    uninitialized_copy(b.elem, &b.elem[sz], elem);  // copy
    for (int i = 0; i < sz; ++i)
        alloc.destroy(&elem[i]);     // destroy old
    swap<vector_base<T, A>>(*this, b);           // swap representations
}
```

我们使用了标准库函数`uninitialized_copy()`来构造元素的拷贝，因为它能够正确处理来自元素拷贝构造函数的异常。这里使用`swap()`保证了：如果`reserve()`正常返回，则旧的内存空间将被`vector_base`的析构函数自动释放；如果退出是由异常造成的，则新分配的空间将被释放。

注：私有继承使得`vector_base`的公有成员成为`vector`的私有成员。

## 简单练习
[drill19](https://github.com/ZZy979/PPP-code/blob/main/ch19/drill19.cpp)

## 习题
* [19-1](https://github.com/ZZy979/PPP-code/blob/main/ch19/vector_add.h) （实现`std::valarray`的`operator+`）
* [19-2](https://github.com/ZZy979/PPP-code/blob/main/ch19/inner_product.h) （实现`std::inner_product()`）
* 19-3 （实现`std::pair`）
    * [Pair](https://github.com/ZZy979/PPP-code/blob/main/ch19/pair.h)
    * [Symbol_table](https://github.com/ZZy979/PPP-code/blob/main/ch19/symbol_table.h)
* 19-4
    * [双向链表v3](https://github.com/ZZy979/PPP-code/blob/main/ch19/doubly_linked_list_v3.h)
    * [链表的应用v3](https://github.com/ZZy979/PPP-code/blob/main/ch19/exec19-4.cpp)
* [19-5~19-7](https://github.com/ZZy979/PPP-code/blob/main/ch19/number.h)
* [19-8~19-9](https://github.com/ZZy979/PPP-code/blob/main/ch19/simple_allocator.h) （简化的`std::allocator`）
* [19-10](https://github.com/ZZy979/PPP-code/blob/main/ch19/unique_ptr.h) （简化的`std::unique_ptr`）
* [19-11](https://github.com/ZZy979/PPP-code/blob/main/ch19/counted_ptr.h) （简化的`std::shared_ptr`）
