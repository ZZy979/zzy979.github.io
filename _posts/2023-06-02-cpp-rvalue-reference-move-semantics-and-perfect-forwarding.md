---
title: 【C++】右值引用、移动语义和完美转发
date: 2023-06-02 01:50:58 +0800
categories: [C/C++]
tags: [cpp, value category, reference, rvalue reference, move semantics, move constructor, move assignment, copy elision, forwarding reference, perfect forwarding]
---
在C++中，如果一个类获取了资源（例如动态内存、文件、锁、线程、套接字等），则需要定义拷贝构造函数、拷贝赋值运算符和析构函数以确保资源被正确地拷贝和释放。然而，在某些情况下会存在不必要的拷贝，影响程序性能。为了解决这一问题，C++11引入了移动语义。本文首先介绍C++的值类别和左/右值引用，之后介绍移动语义及其实现，最后介绍完美转发。

## 1.值类别
在C++中，每个表达式除了具有类型，还有**值类别**(value category)。

传统C++ (C++98/03)继承了C语言的值类别，只有两种：lvalue和rvalue。

| 值类别 | 性质 | 示例 |
| --- | --- | --- |
| **左值**(lvalue) | 可以出现在赋值运算符左侧，可以取地址 | 变量名`a`、数据成员`a.m`、数组下标`a[i]`、指针解引用`*p`、<br>返回左值引用的函数调用等 |
| **右值**(rvalue) | 只能出现在赋值运算符右侧，不能取地址 | 整数常量`42`、算术表达式`a + b`、临时对象`Point{3, 4}`、<br>返回非引用类型的函数调用等 |

详见[C Value categories](https://en.cppreference.com/c/language/value_category)。

例如：

```cpp
int a;
int* p = &a;  // OK, a is lvalue
*p = 42;      // OK, *p is lvalue

p = &42;      // error, 42 is rvalue
a + 1 = *p;   // error, a + 1 is rvalue
```

然而，由于C++11引入了右值引用和移动语义，值类别的定义发生了很大变化。因此C++标准做了扩充和修正，定义了三种基本值类别：lvalue、xvalue和prvalue，以及两种混合类别：glvalue和rvalue。

| 值类别 | 性质 | 示例 |
| --- | --- | --- |
| 左值(lvalue) | 具有标识(identity)，不可移动 | `a`、`a.m`、`a[i]`、`*p`、<br>返回左值引用的函数调用等 |
| 将亡值(xvalue, "eXpiring" value) | 具有标识，可移动 | `f().m`、<br>返回右值引用的函数调用（如`std::move(x)`）等 |
| 纯右值(prvalue, "pure" rvalue) | 没有标识，可移动 | `42`、`this`、`a + b`、`Point{3, 4}`、<br>返回非引用类型的函数调用等 |
| 泛左值(glvalue, "generalized" lvalue)：lvalue或xvalue | 具有标识 | |
| 右值(rvalue)：prvalue或xvalue | 可移动 | |

![值类别](/assets/images/cpp-rvalue-reference-move-semantics-and-perfect-forwarding/值类别.png)

详见[C++ Value categories](https://en.cppreference.com/w/cpp/language/value_category)。

## 2.左值引用和右值引用
C++的**引用**(reference)是一种类型，可以看作对象的别名。主流编译器底层用指针实现引用，本质上都是对象的地址（示例见[Compiler Explorer](https://godbolt.org/z/xx4vz4YWM)，指针和引用的区别详见[《C++程序设计原理与实践》笔记 第17章]({% post_url 2023-04-22-ppp-note-ch17-vector-and-free-store %}) 17.9节）。

C++提供了两种类型的引用：
* **左值引用**(lvalue reference)：使用`&`表示，`T&`是`T`类型的左值引用。左值引用是最常用的引用类型，可用于在函数调用中实现**传引用**(pass-by-reference)语义。
* **右值引用**(rvalue reference)：使用`&&`表示，`T&&`是`T`类型的右值引用。右值引用是C++11引入的，用于实现**移动语义**（见第3节）。

左值/右值是表达式的**值类别**，而左值引用/右值引用是表达式的**类型**。二者是完全不同的概念，但是存在一定的联系：
* 非`const`左值引用(`T&`)只能绑定到左值，`const`左值引用(`const T&`)可以绑定到左值或右值（会延长右值的生存期），右值引用(`T&&`)只能绑定到右值。
* 返回类型是左值引用的函数调用表达式（如`vec[i]`）是左值；返回类型是右值引用的函数调用表达式（如`std::move(x)`）是将亡值；返回类型不是引用的函数调用表达式（如`vec.size()`）是纯右值。
* 左值引用类型的表达式一定是左值；右值引用类型的表达式可能是左值或右值：**有名字的右值引用（例如变量和形参）是左值**，没有名字的右值引用（例如`std::move()`和`std::forward()`调用表达式）是右值。详见[lvalue - cppreference](https://en.cppreference.com/w/cpp/language/value_category#lvalue)：

> Even if the variable's type is rvalue reference, the expression consisting of its name is an lvalue expression.

### 2.1 示例
下面是一个使用左值引用和右值引用的示例：

```cpp
int a = 8;
int& lr = a;   // lr is lvalue reference
int* p = &lr;  // OK, lr is lvalue
lr = 42;       // OK, lr is lvalue

int& lr2 = 42;         // error, lvalue reference can't bind to rvalue
const int& clr = 42;   // OK, const lvalue reference can bind to rvalue
const int* cp = &clr;  // OK, clr is lvalue

int&& rr = a + 1;  // rr is rvalue reference
p = &rr;  // OK, rr is lvalue
rr += 1;  // OK, rr is lvalue

int&& rr2 = a;             // error, rvalue reference can't bind to lvalue
int&& rr3 = rr;            // error, rvalue reference can't bind to lvalue
int&& rr4 = std::move(a);  // OK, std::move(a) is rvalue

int f(int);
int& g(int);
f(a) = 1;   // error, f(a) is rvalue
g(a) = 1;   // OK, g(a) is lvalue
```

在这个示例中，各表达式的类型和值类别如下表所示：

| 表达式 | 类型 | 值类别 |
| --- | --- | --- |
| `42` | `int` | 纯右值 |
| `a` | `int` | 左值 |
| `a+1` | `int` | 纯右值 |
| `lr` | `int&` | 左值 |
| `rr` | `int&&` | 左值 |
| `lr+1`和`rr+1` | `int` | 纯右值 |
| `f(a)` | `int` | 纯右值 |
| `g(a)` | `int&` | 左值 |
| `std::move(a)` | `int&&` | 将亡值 |

其中，`std::move()`函数将在3.3节介绍。

### 2.2 重载解析
如果函数实参是右值，那么重载解析规则会优先选择右值引用版本的重载。例如：

```cpp
#include <iostream>
#include <utility>

void f(int& x) {
    std::cout << "lvalue reference overload f(" << x << ")\n";
}

void f(const int& x) {
    std::cout << "lvalue reference to const overload f(" << x << ")\n";
}

void f(int&& x) {
    std::cout << "rvalue reference overload f(" << x << ")\n";
}

int main() {
    int i = 1;
    const int ci = 2;

    f(i);  // calls f(int&)
    f(ci); // calls f(const int&)
    f(3);  // calls f(int&&)
           // would call f(const int&) if f(int&&) overload wasn't provided
    f(std::move(i)); // calls f(int&&)

    // rvalue reference variables are lvalues when used in expressions
    int&& x = 1;
    f(x);            // calls f(int& x)
    f(std::move(x)); // calls f(int&& x)
}
```

注意：
* `f(3)`调用`f(int&&)`，因为`3`是右值。如果没有定义`f(int&&)`，则会调用`f(const int&)`，因为`const`左值引用可以绑定到右值。
* `f(x)`调用`f(int&)`，因为`x`是左值（尽管其类型是`int&&`）。

### 2.3 decltype
运算符`decltype`可用于检查表达式的类型和值类别。假设表达式`expr`的类型为`T`，则根据其值类别的不同，`decltype(expr)`会产生不同的类型：

| 值类别 | decltype |
| --- | --- |
| xvalue | `T&&` |
| lvalue | `T&` |
| prvalue | `T` |

注意：对于一个变量名`x`，`decltype(x)`产生变量的声明类型，而`decltype((x))`产生左值表达式在上表中对应的类型；对于其他表达式，双括号等同于单括号。详见[decltype specifier](https://en.cppreference.com/cpp/language/decltype)。

例如，对于以下代码

```cpp
int a = 42;
int& r = a;
int&& rr = std::move(a);
```

| x | decltype(x) | decltype((x)) |
| --- | --- | --- |
| `42` | `int` | `int` |
| `a` | `int` | `int&` |
| `r` | `int&` | `int&` |
| `rr` | `int&&` | `int&` |
| `std::move(a)` | `int&&` | `int&&` |

可以像这样利用类模板和`decltype`来判断表达式的值类别：

```cpp
#include <type_traits>
#include <utility>

template <class T> struct is_prvalue : std::true_type {};
template <class T> struct is_prvalue<T&> : std::false_type {};
template <class T> struct is_prvalue<T&&> : std::false_type {};

template <class T> struct is_lvalue : std::false_type {};
template <class T> struct is_lvalue<T&> : std::true_type {};
template <class T> struct is_lvalue<T&&> : std::false_type {};

template <class T> struct is_xvalue : std::false_type {};
template <class T> struct is_xvalue<T&> : std::false_type {};
template <class T> struct is_xvalue<T&&> : std::true_type {};

int main() {
    int a = 42;
    int& r = a;
    int&& rr = std::move(a);

    // Expression `42` is prvalue
    static_assert(is_prvalue<decltype((42))>::value);

    // Expression `a` is lvalue
    static_assert(is_lvalue<decltype((a))>::value);

    // Expression `r` is lvalue
    static_assert(is_lvalue<decltype((r))>::value);

    // Expression `std::move(a)` is xvalue
    static_assert(is_xvalue<decltype((std::move(a)))>::value);

    // Expression `rr` is lvalue
    static_assert(is_lvalue<decltype((rr))>::value);
}
```

## 3.移动语义
为了在特定情况下避免不必要的拷贝，C++11引入了移动语义。下面通过一个`vector`的例子说明什么情况下存在不必要的拷贝，之后介绍如何实现移动语义。

### 3.1 简化的vector
一个类可能会获取资源（例如动态内存），这样的类通常具有指向资源的指针成员。标准库`vector`是一个典型的例子。

简化的`std::vector<double>`实现如下：

```cpp
#include <algorithm>

class vector {
public:
    explicit vector(int s) :sz(s), elem(new double[s]{0}) {}

    vector(const vector& v) :sz(v.sz), elem(new double[sz]) {
        std::copy(v.elem, v.elem + sz, elem);
    }

    vector& operator=(const vector& v) {
        double* p = new double[v.sz];
        std::copy(v.elem, v.elem + v.sz, p);
        delete[] elem;
        elem = p;
        sz = v.sz;
        return *this;
    }

    ~vector() { delete[] elem; }

    int size() const { return sz; }

    double& operator[](int i) { return elem[i]; }
    double operator[](int i) const { return elem[i]; }

private:
    int sz;
    double* elem;
};
```

然而，在某些情况下会存在**不必要的拷贝**。考虑下面的例子（来自[《C++程序设计原理与实践》第18章]({% post_url 2023-05-28-ppp-note-ch18-vectors-and-arrays %}) 18.3.4节）：

```cpp
vector fill(istream& is) {
    vector res;
    for (double x; is >> x;) res.push_back(x);
    return res;
}

void use() {
    vector vec = fill(cin);
    // ... use vec ...
}
```

在函数`fill()`返回的过程中，会发生拷贝操作(`res`→`vec`)。假设`res`有10万个元素，则将其拷贝到`vec`的代价是很高的。

![vector拷贝](/assets/images/cpp-rvalue-reference-move-semantics-and-perfect-forwarding/vector拷贝.gif)

但实际上，`use()`永远不会使用`res`，因为`res`在函数`fill()`返回后就会被销毁，因此从`res`到`vec`的拷贝就是不必要的——可以设法让`vec`直接复用`res`的资源。

为了解决这一问题，C++11引入了**移动语义**(move semantics)：通过“窃取”资源，直接将`res`的资源**移动**(move)到`vec`，如下图所示：

![vector移动](/assets/images/cpp-rvalue-reference-move-semantics-and-perfect-forwarding/vector移动.gif)

移动之后，`vec`将引用`res`的元素，而`res`将被置空。从而以仅仅拷贝一个`int`和一个指针的代价将10万个元素从`res`“移交”给`vec`。

换句话说，**移动即转移资源所有权**，是一种安全的浅拷贝，目的是解决由**即将被销毁的对象**初始化或赋值给其他对象时发生不必要的拷贝。

### 3.2 移动构造函数和移动赋值
为了在C++中表达移动语义，需要定义**移动构造函数**(move constructor)和**移动赋值**(move assignment)运算符：

```cpp
T(T&& v);             // move constructor
T& operator=(T&& v);  // move assignment
```

移动构造函数和移动赋值运算符的形参都是右值引用，而实参通常是将亡值表达式——这正是前面提到的“即将被销毁的对象”。

`vector`的移动构造函数和移动赋值运算符定义如下：

```cpp
// move constructor
vector::vector(vector&& v) noexcept
        :sz(v.sz), elem(v.elem) {
    v.sz = 0;
    v.elem = nullptr;
}

// move assignment
vector& vector::operator=(vector&& v) noexcept {
    delete[] elem;
    elem = v.elem;
    sz = v.sz;
    v.elem = nullptr;
    v.sz = 0;
    return *this;
}
```

注：虽然C++标准没有强制要求，但最好将自定义类型的移动操作标记为`noexcept`，这可以使标准库容器在重新分配内存时优先使用移动而非拷贝，从而提升性能（例如`std::vector`要求移动操作是`noexcept`才会使用移动）。

移动构造函数和移动赋值运算符的调用时机如下：
* 当使用一个右值初始化一个相同类型的对象时，移动构造函数将被调用。
* 当对象出现在赋值表达式左侧，并且右侧是一个相同类型的右值时，移动赋值运算符将被调用。

注：如果初始值是纯右值（例如`T a = T();`、`f(T())`、`return T();`），则移动构造函数调用可能会被拷贝消除优化掉，详见3.4节。

再次考虑前面的例子。为`vector`实现了移动语义后，在`fill()`返回时，`vector`的移动构造函数将被隐式调用，因为表达式`fill(cin)`是右值（`fill()`和`use()`的代码均不需要修改）。

注意：
* 移动语义只对拥有资源的类（如STL容器）有意义，对于[基本类型](https://en.cppreference.com/w/cpp/named_req/ScalarType)和[POD类型](https://en.cppreference.com/w/cpp/named_req/PODType)无意义。
* 对于拥有资源的类，移动语义只能省略资源的拷贝，而类本身的数据成员仍然需要拷贝。例如，`vector`的移动构造函数省略了数组元素的拷贝，但仍然需要拷贝`sz`和`elem`两个成员。
* 如果一个类没有定义移动操作，但所有基类和成员都是可移动的，并且没有定义拷贝操作和析构函数，则编译器会自动生成移动操作（逐个成员移动）。详见[Implicitly-declared move constructor](https://en.cppreference.com/w/cpp/language/move_constructor#Implicitly-declared_move_constructor)和[Implicitly-declared move assignment operator](https://en.cppreference.com/w/cpp/language/move_assignment#Implicitly-declared_move_assignment_operator)。
* 拥有资源的类应当提供适当的析构函数、拷贝操作和移动操作（因为移动语义的“正确含义”只有类作者自己知道，而编译器自动生成的移动操作很有可能是错误的）。这称为[三/五/零原则](https://cppreference.com/cpp/language/rule_of_three)。

### 3.3 std::move
前面提到，左值不可移动。但是，标准库头文件\<utility\>提供了`std::move()`函数，作用是**将参数强制转换为右值引用**，使其变成可移动的。

实际上，`std::move()`仅仅是一个强制类型转换：

```cpp
template<class T>
std::remove_reference_t<T>&& move(T&& t) noexcept {
    return static_cast<std::remove_reference_t<T>&&>(t);
}
```

其中`std::remove_reference_t<T>`的作用是移除参数类型中的引用，再加上`&&`就变成了右值引用。

详见[std::move - cppreference](https://en.cppreference.com/w/cpp/utility/move)。

以`int`类型为例，当实参是左值时，函数`std::move()`的实例化如下：

```cpp
int&& move(int& t) {
    return static_cast<int&&>(t);
}
```

当实参是右值时，函数`std::move()`的实例化如下：

```cpp
int&& move(int&& t) {
    return static_cast<int&&>(t);
}
```

C++标准规定：**`std::move()`的调用表达式是将亡值**。这意味着，如果`x`是左值，虽然`x`本身不可移动，但`std::move(x)`是可移动的。使用该表达式初始化或赋值给其他对象后，就会将`x`的资源移动到该对象，因此`x` **不能再使用了**。

注意：**`std::move()`函数本身并不执行任何移动操作**。只有将函数调用结果用于初始化或赋值给其他对象时才会执行移动操作，否则没有任何作用—— "std::move doesn't move anything." （`move`这个名字相当具有误导性，或许叫`make_movable`更合适）

考虑下面的例子：

```cpp
std::vector<int> a = {1, 2, 3};
std::vector<int> b = std::move(a);  // (1) calls move constructor
std::cout << a.size() << ' ' << b.size() << std::endl;  // prints "0 3"

std::vector<int>&& r = std::move(b);  // (2) no move
std::cout << b.size() << ' ' << r.size() << std::endl;  // prints "3 3"
```

其中，(1)处的移动操作并不是发生在`std::move(a)`，而是`b`的移动构造函数，(2)处没有执行任何移动操作。

### 3.4 拷贝消除
C++标准支持**拷贝消除**(copy elision)，允许编译器在某些情况下省略拷贝构造函数和移动构造函数的调用，从而提高程序的性能。拷贝消除的规则也随着C++标准版本的更新而不断扩展。

从C++17开始，在下列情况下编译器会强制进行拷贝消除：
* 在对象初始化中，初始值是相同类型的纯右值。例如，`T x = T();`
* 在`return`语句中，操作数是与返回类型相同的纯右值。例如，`T f() { return T(); }`

在下列情况下，编译器允许但不强制进行拷贝消除：
* 在对象初始化中，源对象是一个相同类型的无名的临时对象。当这个临时对象来自`return`语句时，这一规则称为**返回值优化**(return value optimization, RVO)。例如，`T x = f();`
* 在`return`语句中，操作数是与返回类型相同的变量的名字，但不能是函数参数。这一规则称为**命名返回值优化**(named return value optimization, NRVO)。例如，`T f() { T x; return x; }`

上面仅列出了常见情况，完整规则详见[Copy elision - cppreference](https://en.cppreference.com/w/cpp/language/copy_elision)。

当拷贝消除发生时，源对象和目标对象将变成同一个对象，见3.5节示例。

### 3.5 示例
小结：拷贝构造函数、拷贝赋值、移动构造函数和移动赋值这四个特殊成员函数被调用的时机如下。

| 上下文 | 源表达式 | 调用 | 示例 |
| --- | --- | --- | --- |
| 初始化 | 左值 | 拷贝构造函数 | `T a = b;`，其中`b`是`T`类型<br>`f(a);`，其中`a`和函数参数都是`T`类型<br>`return a;`，其中函数返回值是`T`类型，且`T`没有移动构造函数 |
| 赋值 | 左值 | 拷贝赋值 | `a = b;` |
| 初始化 | 右值 | 移动构造函数 | `T a = std::move(b);`，其中`b`是`T`类型<br>`f(std::move(a));`，其中`a`和函数参数都是`T`类型<br>`return a;`，其中函数返回值是`T`类型，且`T`有移动构造函数 |
| 赋值 | 右值 | 移动赋值 | `a = std::move(b);` |

下面是一个测试示例：

```cpp
#include <iostream>

class C {
public:
    C() {}
    C(const C& c) { std::cout << "copy constructor\n"; }
    C(C&& c) { std::cout << "move constructor\n"; }
    C& operator=(const C& c) { std::cout << "copy assignment\n"; return *this; }
    C& operator=(C&& c) { std::cout << "move assignment\n"; return *this; }
};

C f() {
    C c;
    return c;
}

void g(C c) {}

int main() {
    std::cout << "C a = f();\n";
    C a = f();

    std::cout << "\nC b = std::move(a);\n";
    C b = std::move(a);

    std::cout << "\na = C();\n";
    a = C();

    std::cout << "\nb = a;\n";
    b = a;

    C&& r = std::move(b);
    std::cout << "\na = r;\n";
    a = r;

    std::cout << "\nb = std::move(a);\n";
    b = std::move(a);

    const C c;
    std::cout << "\nb = std::move(c);\n";
    b = std::move(c);

    std::cout << "\ng(a);\n";
    g(a);

    std::cout << "\ng(C());\n";
    g(C());

    std::cout << "\ng(std::move(a));\n";
    g(std::move(a));
    return 0;
}
```

输出如下：

```
C a = f();
move constructor
move constructor

C b = std::move(a);
move constructor

a = C();
move assignment

b = a;
copy assignment

a = r;
copy assignment

b = std::move(a);
move assignment

b = std::move(c);
copy assignment

g(a);
copy constructor

g(C());
move constructor

g(std::move(a));
move constructor
```

* `return c;`调用移动构造函数（将局部变量`c`移动到返回值临时对象），因为函数`f()`的返回类型`C`不是引用类型，且`C`有移动构造函数
* `C a = f();`调用移动构造函数（将返回值临时对象移动到`a`），因为`f()`是一个右值
* `C b = std::move(a);`调用移动构造函数，因为`std::move(a)`是一个右值
* `a = C();`调用移动赋值，因为`C()`是一个右值，且`C`有移动赋值
* `b = a;`调用拷贝赋值，因为`a`是一个左值
* `a = r;`调用拷贝赋值，因为`r`是一个左值（之前的`std::move(b)`对`b`没有任何影响）
* `b = std::move(a);`调用移动赋值，因为`std::move(a)`是一个右值，且`C`有移动赋值
* `b = std::move(c);`调用拷贝赋值，因为`c`是`const`
* `g(a);`调用拷贝构造，因为`a`是一个左值
* `g(C());`调用移动构造，因为`C()`是一个右值，函数`g()`的参数类型是`C`，且`C`有移动构造函数
* `g(std::move(a));`调用移动构造，原因同上

注：`C a = f();`涉及的两次移动构造函数调用可能会被编译器的拷贝消除特性优化掉，从而`c`和`a`的地址是一样的，整个语句只有一次默认构造函数调用。使用不同的C++标准版本和编译选项的情况下，该语句调用移动构造函数的次数如下表所示（使用的编译器是GCC 13）：

| C++标准版本 | 编译选项 | 移动构造次数 |
| --- | --- | --- |
| C++11 | `-fno-elide-constructors` | 2 (`c`→返回值临时对象→`a`) |
| C++17 | `-fno-elide-constructors` | 1 (`c`→`a`) |
| C++11 | 无 | 0 (`&c == &a`) |
| C++17 | 无 | 0 (`&c == &a`) |

其中，选项`-fno-elide-constructors`禁用拷贝消除。

如果类`C`没有定义移动操作，则所有移动操作会退化为拷贝操作。然而，在这种情况下，编译器会自动生成移动构造函数，因此类`C`仍然是可移动构造的（可用`std::is_move_constructible`验证）。使用[C++ Insights](https://cppinsights.io/)工具可以看到，编译器会将`C b = std::move(a);`解释为`C b = C(static_cast<const C &&>(std::move(a)));`，因此调用的是拷贝构造函数而不是移动构造函数。

如果类`C`是不可移动构造的，即移动构造函数被（显式或隐式）定义为已删除，则函数`f()`会编译失败。
* 显式删除：`C(C&&) = delete;`
* 隐式删除：包含不可移动构造的成员或基类

### 3.6 最佳实践
不要过度使用`std::move()`！

经验法则：当需要将左值传递给右值引用类型的参数、通过转移资源所有权的方式避免拷贝时，应该使用`std::move()`。

使用`std::move()`之前考虑三个问题：**可移动吗？移动了吗？移动比拷贝更快吗？**

（1）使用`std::move()`的前提：类型是**可移动的**，否则移动操作会退化为拷贝操作。
* 可移动的类型：基本类型（如`int`）、定义了移动操作的类（如STL容器、protobuf消息类）和所有成员都可移动的类。
* 不可移动的类型：禁止了移动操作的类（如`std::mutex`）和包含不可移动成员的类。这种类型只能通过传指针或引用的方式来避免拷贝。

（2）**std::move ≠ 移动。** 只有当参数是非`const`左值，并且将`std::move()`的结果用于**初始化或赋值给其他对象**时才会执行移动操作。
* 对右值使用`std::move()`没有意义，因为结果仍然是右值。
* 不要对`const`对象使用`std::move()`，因为移动操作通常会修改源对象（置空），而`const`对象禁止修改，重载解析会优先匹配拷贝操作。
* 将`std::move()`的结果赋给右值引用变量没有意义，因为并没有执行任何移动操作（见3.3节结尾的示例）。

（3）不要对仍然需要的对象使用`std::move()`。已被移动的标准库对象处于一种“合法但未指定的状态”，因此不应该再被使用。

（4）不要在`return`语句中使用`std::move()`，因为会影响NRVO。见[Move-eligible expressions](https://en.cppreference.com/w/cpp/language/value_category#Move-eligible_expressions)和[Automatic move from local variables and parameters](https://en.cppreference.com/w/cpp/language/return#Automatic_move_from_local_variables_and_parameters)。

（5）**移动 ≠ 避免拷贝。** 只有对拥有资源的类型，移动才有显著收益。
* 对于基本类型，移动等价于拷贝（示例见[Compiler Explorer](https://godbolt.org/z/Efz9n9hcc)）。
* 对于拥有资源的类，移动语义只能省略资源的拷贝，而类本身的数据成员仍然需要拷贝，且移动的字节数一般是拷贝的2倍（拷贝+置空）。
* 对于一般的类，要想知道移动是否比拷贝更快，需要比较二者拷贝的字节数。考虑下面的几个例子。

①只包含基本类型成员的类（POD类型）

```cpp
struct Point {
    int x, y;
};
```

假设`int`为4字节
* 拷贝的字节数 = `2 * sizeof(int)` = 8
* 移动的字节数 = `2 * sizeof(int)` = 8

对于这种类型，移动和拷贝的性能完全相同。

②容器类

对于3.1节中的`vector`，假设指针为4字节，元素个数为n
* 拷贝的字节数 = `sizeof(vector) + n * sizeof(double)` = 8 + 8n
* 移动的字节数 = `2 * sizeof(vector)` = 16

拷贝的字节数随着n线性增长，而移动的字节数是常数。如果向量只包含几个元素或者为空，则移动和拷贝的性能几乎相同；如果包含几百万个元素，则移动的性能远高于拷贝。

③同时包含基本类型和容器类型成员的类

```cpp
struct C {
    vector v;
    int k[10000];
};
```

`vector`的定义同上，假设`v`的元素个数为n
* 拷贝的字节数 = `sizeof(vector) + n * sizeof(double) + 10000 * sizeof(int)` = 40008 + 8n
* 移动的字节数 = `2 * sizeof(vector) + 10000 * sizeof(int)` = 40016

则`v`的元素个数越多，移动比拷贝快得越多。数组`k`的40000字节无法避免拷贝。实际性能测试结果如下图所示（测试代码 <https://godbolt.org/z/74Mb7jM3r>）。

![拷贝和移动操作的性能对比](/assets/images/cpp-rvalue-reference-move-semantics-and-perfect-forwarding/copy_vs_move.png)

注：protobuf消息类都是可移动的，具体性能可采用类似的方法分析。`optional`字段相当于基本类型，`string`类型和`repeated`字段相当于容器类型（见[Protocol Buffers入门教程]({% post_url 2022-04-26-protocol-buffers-tutorial %}) 3.1.5节“注”）。

最后给出几个适合使用`std::move()`的示例。

（1）为自定义类型实现移动构造函数和移动赋值时，调用成员的移动操作。例如：

```cpp
struct C {
    std::vector<int> v;
    int k;

    C(const std::vector<int>& v, int k) :v(v), k(k) {}

    C(C&& c) noexcept
            :v(std::move(c.v)),  // calls move constructor of std::vector
            k(std::exchange(c.k, 0)) {}

    C& operator=(C&& c) noexcept {
        v = std::move(c.v);  // calls move assignment of std::vector
        k = std::exchange(c.k, 0);
        return *this;
    }
};
```

注：对于这个类，即使没有提供移动构造函数和移动赋值，编译器也会自动生成（但成员`k`是直接拷贝而不是交换）。

（2）将已初始化的局部变量添加到容器。例如：

```cpp
bool addX(std::vector<X>& v) {
    X x(...);
    if (!x.init())
        return false;
    v.push_back(std::move(x));  // calls push_back(X&&) overload
    return true;
}
```

这几乎是真实业务代码中唯一需要使用`std::move()`的情况。当然，仍然需要满足前提：类`X`可移动，且移动比拷贝更快。如果类`X`不可移动，可用智能指针来包装：`vector<shared_ptr<X>>`。另外，如果不需要额外的初始化操作，可用`v.push_back(X(...))`或`v.emplace_back(...)`来代替（前者也会调用移动构造函数，即使类`X`不可拷贝；而后者直接原地构造，省去拷贝或移动操作）。

（3）对于不再需要的对象，将其资源转移给其他对象。例如：

```cpp
void process(Data data);

void f() {
    Data foo_data(...);
    process(std::move(foo_data));  // calls move constructor of Data
    // foo_data can no longer be used
}
```

调用`process()`函数时，将`foo_data`的资源转移给了形参`data`，因此在调用之后不能再使用`foo_data`（在实际中，一般会将`process()`的参数定义为`const Data&`，这样就不需要使用`std::move()`）。

## 4.完美转发
除了移动语义，右值引用还有一个重要的用途——实现**完美转发**(perfect forwarding)。完美转发是指在模板函数中将参数“无损地”传递给另一个函数，并保持参数的原始值类别。

考虑实现简化的`std::make_unique`函数。该函数只接受一个参数，（拷贝或移动）构造一个`T`类型的对象，并返回拥有该对象的`std::unique_ptr<T>`。

```cpp
#include <iostream>
#include <memory>
#include <utility>

class C {
public:
    C() {}
    C(const C& c) { std::cout << "copy constructor\n"; }
    C(C&& c) noexcept { std::cout << "move constructor\n"; }
};

template<class T, class Arg>
std::unique_ptr<T> make_unique(??? x) {
    return std::unique_ptr<T>(new T(???));
}

int main() {
    C c;
    auto p = make_unique<C>(c);             // should call C's copy constructor
    auto q = make_unique<C>(std::move(c));  // should call C's move constructor
    return 0;
}
```

我们希望当实参是左值时应该调用拷贝构造函数，当实参是右值时应该调用移动构造函数。那么`make_unique()`的形参类型应该如何定义？传递给构造函数的参数又应该是什么？为了解决这两个问题，需要用到完美转发。

下面首先介绍引用折叠和转发引用的概念，之后介绍`std::forward()`函数以及如何实现完美转发。

### 4.1 引用折叠
在C++中不存在引用的引用，但是允许通过模板或`typedef`间接形成引用的引用。在这种情况下，适用**引用折叠**(reference collapsing)规则：右值引用的右值引用折叠为右值引用，其他组合都形成左值引用。例如：

```cpp
typedef int&  lref;
typedef int&& rref;
int n;

lref&  r1 = n;  // type of r1 is int&
lref&& r2 = n;  // type of r2 is int&
rref&  r3 = n;  // type of r3 is int&
rref&& r4 = 1;  // type of r4 is int&&
```

详见[Reference collapsing - cppreference](https://en.cppreference.com/w/cpp/language/reference#Reference_collapsing)。

### 4.2 转发引用
前面提到，`&&`表示右值引用，但是有一个例外——以下两种形式的引用称为**转发引用**(forwarding reference)（也叫万能引用(universal reference)）：
* 函数模板的参数`T&&`，其中`T`是该函数模板的模板参数（无`const`限定）
* `auto&&`

详见[Forwarding references - cppreference](https://en.cppreference.com/w/cpp/language/reference#Forwarding_references)。

例如：

```cpp
template<class T>
void f(T&& x);  // x is a forwarding reference

template<class T>
void g(const T&& x);  // x is not a forwarding reference

template<class T>
struct A {
    template<class U>
    void f(T&& x, U&& y);  // x is not a forwarding reference, but y is
};

auto&& r = foo();  // r is a forwarding reference

for (auto&& x : bar()) {  // x is a forwarding reference
    // ...
}
```

转发引用可以绑定到左值或右值，其实际类型取决于实参：
* 如果实参是左值，`T`被推导为`X&`，`T&&`折叠为`X&`，即左值引用。
* 如果实参是右值，`T`被推导为`X`，`T&&`就是`X&&`，即右值引用。

这依赖于特殊的模板参数推导规则，详见[Deduction from a function call](https://en.cppreference.com/w/cpp/language/template_argument_deduction#Deduction_from_a_function_call)。转发引用是实现完美转发的基础。

回到前面的例子，转发引用正是`make_unique()`函数需要的形参类型：

```cpp
template<class T, class Arg>
std::unique_ptr<T> make_unique(Arg&& x) {
    return std::unique_ptr<T>(new T(???));
}
```

分别用左值和右值调用该函数：

```cpp
C c;
auto p = make_unique<C>(c);             // argument is lvalue, calls make_unique<C, C&>(C&)
auto q = make_unique<C>(std::move(c));  // argument is rvalue, calls make_unique<C, C>(C&&)
```

现在已经解决了形参类型的问题，但是又遇到另一个问题：如何将参数正确地“转发”给构造函数？无论实参是左值还是右值，形参`x`都是左值，即**右值引用在传递时会失去右值属性**。如果写成`new T(x)`则调用的都是拷贝构造函数，而`new T(std::move(x))`调用的都是移动构造函数，都无法满足要求。

因此还需要一种方式能够实现：当形参是左值引用时得到左值表达式，当形参是右值引用时得到右值表达式——这就是`std::forward()`函数。

### 4.3 std::forward
函数`std::forward()`定义在标准库头文件\<utility\>中，作用是**保持参数的值类别**。与`std::move()`类似，`std::forward()`仅仅是一个强制类型转换，但是有两个重载版本：

```cpp
// (1)
template<class T>
T&& forward(std::remove_reference_t<T>& t) noexcept {
    return static_cast<T&&>(t);
}

// (2)
template<class T>
T&& forward(std::remove_reference_t<T>&& t) noexcept {
    return static_cast<T&&>(t);
}
```

详见[std::forward - cppreference](https://en.cppreference.com/w/cpp/utility/forward)。

`std::forward()`的返回类型和调用表达式的值类别取决于它的实参和模板参数。假设`c`是一个类`C`的对象，`rr`是`C&&`类型的变量，则

| 情况 | 调用表达式 | 匹配的重载 | 返回类型 | 值类别 |
| --- | --- | --- | --- | --- |
| ① | `std::forward<C&>(c)` | (1) | `C&` | 左值 |
| ② | `std::forward<C>(c)` | (1) | `C&&` | 将亡值 |
| ③ | `std::forward<C&>(rr)` | (1) | `C&` | 左值 |
| ④ | `std::forward<C>(rr)` | (1) | `C&&` | 将亡值 |
| ⑤ | `std::forward<C&>(C())` | (2) | `C&` | 编译错误：右值不能绑定到左值引用 |
| ⑥ | `std::forward<C>(C())` | (2) | `C&&` | 将亡值 |

注意：
* 上表中只有①④⑥是有实际意义的。
* 调用`std::forward()`时必须显式指定模板参数，不能自动推导。因为需要原始实参的引用类型（左值引用/右值引用），仅靠实参无法还原。
* 在实际中，`std::forward()`的实参通常是外层函数模板的转发引用参数，模板参数指定为转发引用对应的模板参数。

回到前面的例子，正确的构造函数调用是`new T(std::forward<Arg>(x))`。`make_unique()`函数的完整定义如下：

```cpp
template<class T, class Arg>
std::unique_ptr<T> make_unique(Arg&& x) {
    return std::unique_ptr<T>(new T(std::forward<Arg>(x)));
}
```

程序的输出如下(<https://godbolt.org/z/fGE6GWGzn>)：

```
copy constructor
move constructor
```

下面分析两次调用中的模板实例化过程。

对于第一次调用`make_unique<C>(c)`，函数`make_unique()`的实例化如下：

```cpp
// T = C, Arg = C&
std::unique_ptr<C> make_unique(C& x) {
    return std::unique_ptr<C>(new C(std::forward<C&>(x)));
}
```

匹配`std::forward()`的重载(1)，实例化如下：

```cpp
// T = C&
C& forward(C& t) noexcept {
    return static_cast<C&>(t);
}
```

此时表达式`std::forward<C&>(x)`是左值，因此`new`调用拷贝构造函数。

对于第二次调用`make_unique<C>(std::move(c))`，函数`make_unique()`的实例化如下：

```cpp
// T = C, Arg = C
std::unique_ptr<C> make_unique(C&& x) {
    return std::unique_ptr<C>(new C(std::forward<C>(x)));
}
```

匹配`std::forward()`的重载(1)，实例化如下：

```cpp
// T = C
C&& forward(C&& t) noexcept {
    return static_cast<C&&>(t);
}
```

此时表达式`std::forward<C>(x)`是右值，因此`new`调用移动构造函数。

标准库头文件\<memory\>中的`std::make_unique()`函数更复杂，支持变长参数、数组类型等。其中一个重载的定义如下：

```cpp
template<class T, class... Args>
std::unique_ptr<T> make_unique(Args&&... args) {
    return std::unique_ptr<T>(new T(std::forward<Args>(args)...));
}
```

类似的函数还有`vector<T>::emplace_back()`、`allocator<T>::construct()`、`std::invoke()`等。

### 4.4 小结
完美转发的实现步骤：
1. 在函数模板中使用转发接收参数。
2. 利用模板参数推导和引用折叠得到正确的引用类型。
3. 使用`std::forward()`传递参数并恢复原始值类别。

`std::forward()`与`std::move()`的对比如下表所示。

| 实参 | 形参`T&& x`（转发引用） | `std::forward<T>(x)` | `std::move(x)` |
| --- | --- | --- | --- |
| 左值 | 左值引用、左值 | 左值引用、左值 | 右值引用、右值 |
| 右值 | 右值引用、左值 | 右值引用、右值 | 右值引用、右值 |

## 5.总结

* 值语义会导致不必要的拷贝，C++11通过移动语义弥补性能缺陷。
* `std::move` ≠ 移动，移动 ≠ 避免拷贝
* 完美转发 = 引用折叠 + 转发引用 + `std::forward`

## 参考
* [Reference declaration - cppreference](https://en.cppreference.com/w/cpp/language/reference)
* [Move constructor - cppreference](https://en.cppreference.com/w/cpp/language/move_constructor)
* [Move assignment - cppreference](https://en.cppreference.com/w/cpp/language/move_assignment)
* [A Brief Introduction to Rvalue References](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2006/n2027.html)
* [【C++深陷】之“左值与右值”](https://blog.csdn.net/u014609638/article/details/106640142)
* [【C++深陷】之“对象移动”](https://blog.csdn.net/u014609638/article/details/107138552)
* [Understanding lvalues, rvalues and their references](https://www.fluentcpp.com/2018/02/06/understanding-lvalues-rvalues-and-their-references/)
* [Beware of using std::move on a const lvalue](https://www.nextptr.com/tutorial/ta1211389378/beware-of-using-stdmove-on-a-const-lvalue)
* [Understanding when not to std::move in C++](https://developers.redhat.com/blog/2019/04/12/understanding-when-not-to-stdmove-in-c)
* [On harmful overuse of std::move](https://devblogs.microsoft.com/oldnewthing/20231124-00/?p=109059)
* [聊聊C++中的完美转发](https://zhuanlan.zhihu.com/p/161039484)
* [谈谈完美转发(Perfect Forwarding)：完美转发 = 引用折叠 + 万能引用 + std::forward](https://zhuanlan.zhihu.com/p/369203981)
* [C++模板从入门到劝退(0)——左值与右值](https://r00tk1ts.github.io/2022/05/27/C++模板从入门到劝退(0)——左值与右值/)
