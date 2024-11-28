---
title: 【C++】右值引用、移动语义和完美转发
date: 2023-06-02 01:50:58 +0800
categories: [C/C++]
tags: [cpp, rvalue reference, move semantics, move constructor, move assignment, perfect forwarding]
---
在C++中，如果一个类获取了资源（例如动态内存、文件、锁、线程、套接字等），则需要定义拷贝构造函数、拷贝赋值运算符和析构函数以确保资源被正确地拷贝和释放。然而，在某些情况下会存在不必要的拷贝，影响程序性能。为了解决这一问题，C++11引入了移动语义。本文首先介绍C++的左值和右值及其引用，之后介绍移动语义和完美转发及其实现。

## 1.左值和右值
在C++中，每个表达式除了具有类型，还有**值类别**(value category)：
* **左值**(lvalue)：可以出现在赋值表达式左侧的值，例如变量名`a`、数据成员`a.m`、下标表达式`a[n]`、解引用表达式`*p`、返回类型是左值引用的函数调用等。**左值可以被赋值和取地址。**
* **右值**(rvalue)：只能出现在赋值表达式右侧的值，例如字面值`42`、算术表达式`a+b`、临时对象`Point(3,4)`、返回类型不是引用类型的函数调用等。**右值不能被赋值和取地址。**

例如：

```cpp
int a;
int* p = &a;  // OK, a is lvalue
*p = 42;      // OK, *p is lvalue

p = &42;      // error, 42 is rvalue
a + 1 = *p;   // error, a + 1 is rvalue
```

注：实际上C++标准定义了三个值类别——左值(lvalue)、纯右值(prvalue)和将亡值(xvalue)，纯右值和将亡值统称为右值(rvalue)，详见[Value categories - cppreference](https://en.cppreference.com/w/cpp/language/value_category)。“右值”只是沿用了C语言中的叫法。

## 2.左值引用和右值引用
C++的**引用**(reference)是一种类型，可以看作对象的别名。引用在本质上和指针一样，都是对象的地址（示例见[Compiler Explorer](https://godbolt.org/z/xx4vz4YWM)，指针和引用的区别详见[《C++程序设计原理与实践》笔记 第17章]({% post_url 2023-04-22-ppp-note-ch17-vector-and-free-store %}) 17.9节）。

C++提供了两种类型的引用：
* **左值引用**(lvalue reference)：使用`&`表示，`T&`是`T`类型的左值引用。左值引用是最常用的引用类型，可用于在函数调用中实现**传引用**(pass-by-reference)语义。
* **右值引用**(rvalue reference)：使用`&&`表示，`T&&`是`T`类型的右值引用。右值引用是C++11引入的，用于实现**移动语义**（见第3节）。

注：左值/右值是表达式的**值类别**，而左值引用/右值引用是表达式的**类型**。二者是完全不同的概念，但是存在一定的联系：
* **左值引用必须使用左值初始化**（即左值引用只能绑定到左值），一个例外是`const`左值引用可以使用右值初始化；**右值引用必须使用右值初始化**。
* 如果函数的返回类型是左值引用（例如`vector::operator[]`），则函数调用表达式是左值；如果函数的返回类型是右值引用（例如`std::move()`）或者不是引用（例如`vector::size()`），则函数调用表达式是右值。
* 左值引用类型的表达式是左值；右值引用类型的表达式可能是左值或右值：**有名字的右值引用（例如变量和形参）是左值**，没有名字的右值引用（例如`std::move()`和`std::forward()`调用表达式）是右值。详见[lvalue - cppreference](https://en.cppreference.com/w/cpp/language/value_category#lvalue)：

> Even if the variable's type is rvalue reference, the expression consisting of its name is an lvalue expression.

### 2.1 示例
下面是一个使用左值引用和右值引用的示例：

```cpp
int a;
int& lr = a;   // lr is lvalue reference
int* p = &lr;  // OK, lr is lvalue
lr = 42;       // OK, lr is lvalue

int& lr2 = 42;         // error, lvalue reference can't bind to rvalue
const int& clr = 42;   // OK, const lvalue reference can bind to rvalue
const int* cp = &clr;  // OK, clr is lvalue

int&& rr = a + 1;  // rr is rvalue reference
p = &rr;  // OK, rr is lvalue
++rr;     // OK, rr is lvalue

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
| `42` | `int` | 右值 |
| `a` | `int` | 左值 |
| `a+1` | `int` | 右值 |
| `lr` | `int&` | 左值 |
| `rr` | `int&&` | 左值 |
| `lr+1`和`rr+1` | `int` | 右值 |
| `f(a)` | `int` | 右值 |
| `g(a)` | `int&` | 左值 |
| `std::move(a)` | `int&&` | 右值 |

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

其中
* `f(3)`调用`f(int&&)`，因为3是右值。如果没有定义`f(int&&)`，则调用`f(const int&)`，因为`const`左值引用可以绑定到右值。
* `f(x)`调用`f(int&)`，因为`x`是左值（尽管其类型是`int&&`）。

## 3.移动语义
为了在特定情况下避免不必要的拷贝，C++11引入了移动语义。下面通过一个`vector`的例子说明什么情况下存在不必要的拷贝，之后介绍如何实现移动语义。

### 3.1 简化的vector
一个类可能会获取资源（例如动态内存），这样的类通常具有指向资源的指针成员。标准库`vector`是一个典型的例子。

简化的`vector<double>`实现如下：

```cpp
class vector {
public:
    explicit vector(int s) :sz(s), elem(new double[s]{0}) {}

    vector(const vector& v) { ... }
    vector& operator=(const vector& v) { ... }

    ~vector() { delete[] elem; }

    int size() const { return sz; }

    double& operator[](int i) { return elem[i]; }
    double operator[](int i) const { return elem[i]; }

private:
    int sz;
    double* elem;
};
```

然而，在某些情况下会存在**不必要的拷贝**。考虑下面的例子（来自《C++程序设计原理与实践》第18章）：

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

在函数`fill()`返回的过程中，会发生拷贝操作(`res`→`vec`)。假设`res`有10万个元素，则将其拷贝到`vec`的代价是很高的。但实际上，`use()`永远不会使用`res`，因为`res`在函数`fill()`返回后就会被销毁，因此从`res`到`vec`的拷贝就是不必要的——可以设法让`vec`直接复用`res`的资源。

为了解决这一问题，C++11引入了**移动语义**(move semantics)：通过“窃取”资源，直接将`res`的资源**移动**(move)到`vec`，如下图所示：

![移动前](/assets/images/ppp-note-ch18-vectors-and-arrays/移动前.png)

![移动后](/assets/images/ppp-note-ch18-vectors-and-arrays/移动后.png)

移动之后，`vec`将引用`res`的元素，而`res`将被置空。从而以仅仅拷贝一个`int`和一个指针的代价将10万个元素从`res`“移交”给`vec`。

换句话说，**移动即“窃取”资源（转移资源所有权）**，是一种安全的浅拷贝，目的是解决由**即将被销毁的对象**初始化或赋给其他对象时发生不必要的拷贝。

### 3.2 移动构造函数和移动赋值
为了在C++中表达移动语义，需要定义**移动构造函数**(move constructor)和**移动赋值**(move assignment)运算符：

```cpp
T(T&& v);             // move constructor
T& operator=(T&& v);  // move assignment
```

移动构造函数和移动赋值运算符的参数都是右值引用，因为右值正是前面提到的“即将被销毁的对象”。

`vector`的移动构造函数和移动赋值运算符定义如下：

```cpp
// move constructor
vector::vector(vector&& v) :sz(v.sz), elem(v.elem) {
    v.sz = 0;
    v.elem = nullptr;
}

// move assignment
vector& vector::operator=(vector&& v) {
    delete[] elem;
    elem = v.elem;
    sz = v.sz;
    v.elem = nullptr;
    v.sz = 0;
    return *this;
}
```

* 当使用一个右值初始化一个相同类型的对象时，移动构造函数将被调用。
* 当对象出现在赋值表达式左侧，并且右侧是一个相同类型的右值时，移动赋值运算符将被调用。

注：如果初始值是纯右值（例如`T a = T();`、`f(T())`、`return T();`），则移动构造函数调用可能会被拷贝消除优化掉，详见3.4节。

再次考虑前面的例子。为`vector`实现了移动语义后，在`fill()`返回时，`vector`的移动构造函数将被隐式调用，因为表达式`fill(cin)`是右值（`fill()`和`use()`的代码均不需要修改）。

注意：
* 移动语义只对拥有资源的类（如STL容器）有意义，对于[基本类型](https://en.cppreference.com/w/cpp/named_req/ScalarType)（如`int`）和[POD类型](https://en.cppreference.com/w/cpp/named_req/PODType)（如`struct Point { int x, y; };`）无意义。
* 对于拥有资源的类，移动语义只能省略资源的拷贝，而类本身的数据成员仍然需要拷贝。例如，`vector`的移动构造函数省略了数组元素的拷贝，但仍然需要拷贝`sz`和`elem`两个成员。
* 如果一个类没有定义移动操作，但所有成员都是可移动的，编译器会自动生成移动构造函数和（或）移动赋值，即逐个成员移动（见[Implicitly-declared move constructor](https://en.cppreference.com/w/cpp/language/move_constructor#Implicitly-declared_move_constructor)和[Implicitly-declared move assignment operator](https://en.cppreference.com/w/cpp/language/move_assignment#Implicitly-declared_move_assignment_operator)）。
* 如果定义了一个拥有资源的类，则应当提供适当的移动操作，因为移动语义的“正确含义”只有类作者自己知道（编译器自动生成的移动操作很有可能是错误的）。

### 3.3 std::move
前面提到，右值引用不能绑定到左值，因此**左值不能被移动**。但是，标准库头文件\<utility\>提供了`std::move()`函数，作用是**将参数转换为右值引用**，即将参数“当作”右值，使其变成“可移动的”。

实际上，`std::move()`仅仅是一个强制类型转换：

```cpp
template<class T>
typename std::remove_reference<T>::type&& move(T&& t) noexcept {
    return static_cast<typename std::remove_reference<T>::type&&>(t);
}
```

其中`std::remove_reference<T>::type`的作用是移除参数类型中的引用，再加上`&&`就变成了右值引用。

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

C++标准规定：**`std::move()`的调用表达式是右值**（准确来说是将亡值）。如果`a`是一个左值，则`std::move(a)`是一个右值，这意味着该对象被认为是“可移动的”（可能被窃取资源），因此**不能再使用**。

注意：**`std::move()`本身并不执行任何移动操作**。只有将`std::move()`的结果用于初始化或赋值给另一个对象时才会执行移动操作，否则没有任何作用—— "std::move doesn't move anything." （`move`这个名字相当具有误导性，或许叫`make_movable`更合适）

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
* 在`return`语句中，操作数是与返回类型相同的纯右值。例如，`T f() { return T(); }`
* 在对象初始化中，初始值是相同类型的纯右值。例如，`T x = T();`

在下列情况下，编译器允许但不强制进行拷贝消除：
* 在`return`语句中，操作数是与返回类型相同的变量的名字，但不能是函数参数。这一规则称为**命名返回值优化**(named return value optimization, NRVO)。例如，`T f() { T x; return x; }`
* 在对象初始化中，源对象是一个相同类型的无名的临时对象。当这个临时对象来自`return`语句时，这一规则称为**返回值优化**(return value optimization, RVO)。例如，`T x = f();`

注：上面仅列出了常见情况，完整规则详见[Copy elision - cppreference](https://en.cppreference.com/w/cpp/language/copy_elision)。

当拷贝消除发生时，被省略的拷贝/移动构造函数的源对象（参数）和目标对象(`this`)将变成同一个对象，见3.5节示例。

### 3.5 示例
拷贝构造函数、拷贝赋值、移动构造函数和移动赋值这四个特殊成员函数被调用的时机如下：

| 上下文 | 源表达式 | 调用 | 示例 |
| --- | --- | --- | --- |
| 初始化 | 左值 | 拷贝构造函数 | `T a = b;` |
| 赋值 | 左值 | 拷贝赋值 | `a = b;` |
| 初始化 | 右值 | 移动构造函数 | `T a = std::move(b);` |
| 赋值 | 右值 | 移动赋值 | `a = std::move(b);` |

如果没有定义移动构造函数和移动赋值，则移动操作会退化为拷贝操作。

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

int main() {
    std::cout << "C a = f();\n";
    C a = f();

    std::cout << "\nC b = a;\n";
    C b = a;

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
    return 0;
}
```

输出如下：

```
C a = f();
move constructor
move constructor

C b = a;
copy constructor

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
```

* `return c;`调用移动构造函数（将局部变量`c`移动到返回值临时对象），因为函数`f()`的返回类型`C`不是引用类型，且`C`有移动构造函数
* `C a = f();`调用移动构造函数（将返回值临时对象移动到`a`），因为`f()`是一个右值
* `C b = a;`调用拷贝构造函数，因为`a`是一个左值
* `a = C();`调用移动赋值，因为`C()`是一个右值，且`C`有移动赋值
* `b = a;`调用拷贝赋值，因为`a`是一个左值
* `a = r;`调用拷贝赋值，因为`r`是一个左值（之前的`std::move(b)`对`b`没有任何影响）
* `b = std::move(a);`调用移动赋值，因为`std::move(a)`是一个右值，且`C`有移动赋值
* `b = std::move(c);`调用拷贝赋值，因为`c`是`const`

注：`C a = f();`涉及的两次移动构造函数调用可能会被编译器的拷贝消除特性优化掉，从而`c`和`a`的地址是一样的，整个语句只有一次默认构造函数调用。使用不同的C++标准版本和编译选项的情况下，该语句调用移动构造函数的次数如下表所示（使用的编译器是GCC 13）：

| C++标准版本 | 编译选项 | 移动构造次数 |
| --- | --- | --- |
| C++11 | `-fno-elide-constructors` | 2 (`c`→返回值临时对象→`a`) |
| C++17 | `-fno-elide-constructors` | 1 (`c`→`a`) |
| C++11 | 无 | 0 (`&c == &a`) |
| C++17 | 无 | 0 (`&c == &a`) |

其中，选项`-fno-elide-constructors`禁用拷贝消除。

如果类`C`没有定义移动操作，则输出会变为：

```
C a = f();
copy constructor
copy constructor

C b = a;
copy constructor

a = C();
copy assignment

b = a;
copy assignment

b = std::move(a);
copy assignment

b = std::move(c);
copy assignment
```

注意：在这种情况下，编译器会自动生成移动构造函数，因此类`C`仍然是可移动构造的（可用`std::is_move_constructible`验证）。

类似地，`C a = f();`涉及的两次拷贝构造函数调用可能会被编译器的拷贝消除特性优化掉：

| C++标准版本 | 编译选项 | 拷贝构造次数 |
| --- | --- | --- |
| C++11 | `-fno-elide-constructors` | 2 (`c`→返回值临时对象→`a`) |
| C++17 | `-fno-elide-constructors` | 1 (`c`→`a`) |
| C++11 | 无 | 0 (`&c == &a`) |
| C++17 | 无 | 0 (`&c == &a`) |

如果类`C`是不可移动构造的，即移动构造函数被（显式或隐式）定义为已删除，则函数`f()`会编译失败。
* 显式删除：`C(C&&) = delete;`
* 隐式删除：包含不可移动构造的成员或基类

### 3.6 最佳实践
不要过度使用`std::move()`！

经验法则：当需要将左值传递给右值引用类型的参数、通过转移资源所有权的方式避免拷贝时，应该使用`std::move()`。

使用`std::move()`之前考虑三个问题：**可移动吗？移动了吗？移动比拷贝更快吗？**

（1）使用`std::move()`的前提：**类型是可移动的**，否则移动操作会退化为拷贝操作。
* 可移动的类型：基本类型（如`int`）、定义了移动操作的类（如STL容器）和所有成员都可移动的类。
* 不可移动的类型：禁止了移动操作的类（如`std::mutex`）和包含不可移动成员的类。这种类型只能通过传指针或引用的方式来避免拷贝。

（2）只有当参数是非`const`左值，并且将`std::move()`的结果用于初始化或赋值给另一个对象时，使用`std::move()`才有意义。赋给右值引用变量没有任何意义，因为并没有执行任何移动操作（见3.3节结尾的示例）。

（3）不要在`return`语句中使用`std::move()`，因为会影响NRVO。见[Move-eligible expressions](https://en.cppreference.com/w/cpp/language/value_category#Move-eligible_expressions)和[Automatic move from local variables and parameters](https://en.cppreference.com/w/cpp/language/return#Automatic_move_from_local_variables_and_parameters)。

（4）**移动≠避免拷贝。** 即使类型是可移动的，也要考虑移动是否确实比拷贝更快。
* 对于基本类型，移动等价于拷贝（示例见[Compiler Explorer](https://godbolt.org/z/Efz9n9hcc)）。
* 对于拥有资源的类，移动语义只能省略资源的拷贝，而类本身的数据成员仍然需要拷贝，且移动的字节数一般是拷贝的2倍（拷贝+置空）。

对于一个具体的类，要想知道移动是否比拷贝更快，需要比较二者拷贝的字节数。例如：

```cpp
struct Point {
    int x, y;
};
```

* 拷贝的字节数 = `2 * sizeof(int)` = 8
* 移动的字节数 = `2 * sizeof(int)` = 8

因此，对于只包含基本类型成员的类（POD类型），移动和拷贝的性能相同，移动语义没有任何意义。

对于同时包含基本类型和容器类型成员的类：

```cpp
struct C {
    vector v;
    int k1, k2, ..., k100;
};
```

假设`vector`的定义如3.1节所示，`sizeof(vector)` = 8，向量包含的元素个数为n
* 拷贝的字节数 = `sizeof(vector) + n * sizeof(double) + 100 * sizeof(int)` = 408 + 8n
* 移动的字节数 = `2 * sizeof(vector) + 100 * sizeof(int)` = 416

对于类`C`，如果`v`只包含几个元素（或者为空），则移动和拷贝的性能几乎相同；如果`v`包含几百万个元素，则移动的性能远高于拷贝。

注：protobuf消息类都是可移动的，具体性能可采用类似的方法分析，`optional`字段相当于基本类型，`repeated`字段相当于容器类型（见[Protocol Buffers入门教程]({% post_url 2022-04-26-protocol-buffers-tutorial %}) 3.1.5节“注”）。

最后给出几个适合使用`std::move()`的示例。

（1）为自定义类型实现移动构造函数和移动赋值。例如：

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

注：对于类`C`，即使没有提供移动构造函数和移动赋值，编译器也会自动生成（但成员`k`是直接拷贝而不是使用`std::exchange()`）。

（2）将局部对象传递给接受右值引用参数的函数。例如：

```cpp
bool f(std::vector<X>& v) {
    X x(args...);
    if (!x.init())
        return false;
    v.push_back(std::move(x));  // calls push_back(X&&) overload
    return true;
}
```

这几乎是真实业务代码中唯一需要使用`std::move()`的情况。当然，仍然需要满足前提：类`X`可移动，且移动比拷贝更快。如果类`X`不可移动，可用智能指针来包装：`vector<shared_ptr<X>>`。另外，如果不需要额外的初始化操作，可用`v.push_back(X(args...))`或`v.emplace_back(args...)`来代替（前者也会调用移动构造函数，即使类`X`不可拷贝，而后者直接原地构造）。

## 4.完美转发
除了移动语义，右值引用还有一个重要的用途——实现**完美转发**(perfect forwarding)。下面首先介绍引用折叠和转发引用的概念，之后介绍`std::forward()`函数，并通过一个示例说明如何实现完美转发。

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

引用折叠是实现转发引用的基础。

详见[Reference collapsing - cppreference](https://en.cppreference.com/w/cpp/language/reference#Reference_collapsing)。

### 4.2 转发引用
前面提到，`&&`表示右值引用，但是有一个例外——**转发引用**(forwarding reference)，也叫万能引用(universal reference)。转发引用有两种形式：
* 函数模板中的`T&&`（不能有`const`限定），其中`T`是模板参数
* `auto&&`

详见[Forwarding references - cppreference](https://en.cppreference.com/w/cpp/language/reference#Forwarding_references)。

例如：

```cpp
template<class T>
void f(T&& x) {  // x is a forwarding reference
    // ...
}

auto&& r = foo();  // r is a forwarding reference
```

转发引用的作用是**根据实参是左值或右值自动匹配为左值引用或右值引用**。例如：

```cpp
int i;
f(i);  // argument is lvalue, calls f<int&>(int&)
f(0);  // argument is rvalue, calls f<int>(int&&)
```

* 对于`f(i)`，实参是左值，则`T = int&`，形参类型为`int& &&`，折叠为`int&`。
* 对于`f(0)`，实参是右值，则`T = int`，形参类型为`int&&`。

其中，模板参数`T`的推导依赖于特殊的模板参数推导规则，详见[Deduction from a function call - cppreference](https://en.cppreference.com/w/cpp/language/template_argument_deduction#Deduction_from_a_function_call)。

### 4.3 std::forward
下面介绍`std::forward()`函数，该函数与转发引用一起实现完美转发。

首先考虑一个例子：

```cpp
#include <iostream>
#include <utility>

void g(int& x) {
    std::cout << "lvalue overload, x = " << x << std::endl;
}

void g(int&& x) {
    std::cout << "rvalue overload, x = " << x << std::endl;
}

template<class T>
void f(T&& x) {
    g(x);
}

int main() {
    int x = 42;
    f(x);
    f(123);
    return 0;
}
```

这段程序的输出如下：

```
lvalue overload, x = 42
lvalue overload, x = 123
```

其中，函数`f()`的形参`x`是一个转发引用。虽然`x`能够根据实参自动匹配为左值引用或右值引用，但它**始终是一个左值**（见第2节的小结）。因此无论`f()`的实参是左值还是右值，`g(x)`调用的都是`g(int&)`重载，这与我们的本意不符。我们希望当`f()`的实参是左值时调用`g(int&)`，实参是右值时调用`g(int&&)`，这就要用到`std::forward()`函数。

函数`std::forward()`定义在标准库头文件\<utility\>中，作用是**保持参数的值类别**——如果实参是左值，则调用表达式是左值；如果实参是右值，则调用表达式是右值。

和`std::move()`类似，`std::forward()`实际上也仅仅是一个强制类型转换：

```cpp
template<class T>
T&& forward(std::remove_reference_t<T>& t) noexcept {
    return static_cast<T&&>(t);
}

template<class T>
T&& forward(std::remove_reference_t<T>&& t) noexcept {
    return static_cast<T&&>(t);
}
```

注意：调用`std::forward()`时必须显式指定模板参数，不能依赖模板参数推导。

详见[std::forward - cppreference](https://en.cppreference.com/w/cpp/utility/forward)。

回到前面的例子，将函数`f()`中的调用`g(x)`改为`g(std::forward<T>(x))`，程序的输出如下：

```
lvalue overload, x = 42
rvalue overload, x = 123
```

下面分析两次调用函数`f()`中的模板实例化过程。

对于第一次调用`f(x)`，函数`f()`的实例化如下：

```cpp
// T = int&
void f(int& && x) {
    g(std::forward<int&>(x));
}
```

根据引用折叠规则，`int& &&`等价于`int&`。函数`std::forward()`的实例化如下：

```cpp
// T = int&
int& && forward(std::remove_reference_t<int&>& t) noexcept {
    return static_cast<int& &&>(t);
}
```

根据引用折叠规则，上面的代码等价于

```cpp
int& forward(int& t) noexcept {
    return static_cast<int&>(t);
}
```

可以看出，当实参为左值时，`std::forward()`将`int&`（左值）转换为`int&`（左值），相当于什么都不做，因此`g(std::forward<T>(x))`调用`g(int&)`。

对于第二次调用`f(123)`，函数`f()`的实例化如下：

```cpp
// T = int
void f(int&& x) {
    g(std::forward<int>(x));
}
```

函数`std::forward()`的实例化如下：

```cpp
// T = int
int&& forward(int&& t) noexcept {
    return static_cast<int&&>(t);
}
```

可以看出，当实参为右值时，`std::forward()`将`int&&`（左值）转换为`int&&`（右值），作用相当于`std::move()`，因此`g(std::forward<T>(x))`调用`g(int&&)`。

总结：通过使用转发引用和`std::forward()`，函数`f()`在调用`g()`时保持了参数的值类别，从而实现了完美转发。

与`std::forward()`不同，`std::move()`无论实参是左值还是右值，都转换为右值引用，调用表达式始终是右值。在上面的例子中，如果将调用`g(x)`改为`g(std::move(x))`，则两次调用的都是`g(int&&)`重载，程序的输出如下：

```
rvalue overload, x = 42
rvalue overload, x = 123
```

小结：

| 实参 | 形参`T&& x`（转发引用） | `std::forward<T>(x)` | `std::move(x)` |
| --- | --- | --- | --- |
| 左值 | 左值引用、左值 | 左值引用、左值 | 右值引用、右值 |
| 右值 | 右值引用、左值 | 右值引用、右值 | 右值引用、右值 |

### 4.4 示例
标准库头文件\<memory\>提供了`make_unique()`函数，用于构造一个`T`类型的对象，并返回指向它的`unique_ptr`。其中一个重载的定义如下：

```cpp
template<class T, class... Args>
std::unique_ptr<T> make_unique(Args&&... args) {
    return std::unique_ptr<T>(new T(std::forward<Args>(args)...));
}
```

其中，`args`是转发引用，通过`std::forward()`传递给`T`的构造函数。在这里，`T`的构造函数接收到的参数的值类别将与`make_unique()`函数的实参相同。例如，假设`t`是`T`类对象，则`make_unique<T>(t)`将调用`T`的拷贝构造函数，而`make_unique<T>(std::move(t))`将调用`T`的移动构造函数。

类似的函数还有`vector<T>::emplace_back()`、`allocator<T>::construct()`、`std::invoke()`、`std::function`构造函数等等。

注：使用Python实现类似的功能则简单得多，因为根本不存在值类别的概念（虽然Python根本不需要`unique_ptr`）：

```python
def make_unique(T, *args):
    return unique_ptr(T(*args))
```

## 5.总结
C++的值语义是万恶之源。

## 参考
* [Reference declaration - cppreference](https://en.cppreference.com/w/cpp/language/reference)
* [Move constructor - cppreference](https://en.cppreference.com/w/cpp/language/move_constructor)
* [Move assignment - cppreference](https://en.cppreference.com/w/cpp/language/move_assignment)
* [A Brief Introduction to Rvalue References](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2006/n2027.html)
* [【C++深陷】之“左值与右值”](https://blog.csdn.net/u014609638/article/details/106640142)
* [【C++深陷】之“对象移动”](https://blog.csdn.net/u014609638/article/details/107138552)
* [Understanding lvalues, rvalues and their references](https://www.fluentcpp.com/2018/02/06/understanding-lvalues-rvalues-and-their-references/)
* [《C++程序设计原理与实践》笔记 第18章]({% post_url 2023-05-28-ppp-note-ch18-vectors-and-arrays %})
* [Beware of using std::move on a const lvalue](https://www.nextptr.com/tutorial/ta1211389378/beware-of-using-stdmove-on-a-const-lvalue)
* [Understanding when not to std::move in C++](https://developers.redhat.com/blog/2019/04/12/understanding-when-not-to-stdmove-in-c)
* [On harmful overuse of std::move](https://devblogs.microsoft.com/oldnewthing/20231124-00/?p=109059)
* [聊聊C++中的完美转发](https://zhuanlan.zhihu.com/p/161039484)
* [谈谈完美转发(Perfect Forwarding)：完美转发 = 引用折叠 + 万能引用 + std::forward](https://zhuanlan.zhihu.com/p/369203981)
