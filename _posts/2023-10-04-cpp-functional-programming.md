---
title: C++函数式编程
date: 2023-10-04 00:55 +0800
categories: [C/C++]
tags: [cpp, functional programming, function object, pointer to function, lambda expression]
math: true
---
## 1.引言
**函数式编程**(functional programming)是一种编程范式，通过函数的求值和组合来构造程序。在函数式编程中，函数被视为一等公民，可以像其他数据类型一样作为参数传递和从函数中返回。接受函数作为参数、或者返回函数的函数叫做**高阶函数**(high-order function)，是函数式编程的核心概念之一。函数式编程强调使用**纯函数**(pure function)，即没有状态和副作用、返回值只依赖于参数的函数。

例如，在Scala中计算一个数组中所有偶数的平方和，使用传统的命令式编程(imperative programming)：

```scala
var result = 0
for (x <- arr) {
  if (x % 2 == 0) {
    result += x * x
  }
}
```

使用函数式编程：

```scala
val result = arr.filter(x => x % 2 == 0)
  .map(x => x * x)
  .reduce((x, y) => x + y)
```

其中，`filter()`、`map()`和`reduce()`是高阶函数，`x => x % 2 == 0`、`x => x * x`和`(x, y) => x + y`是纯函数。

函数式编程提供了一种清晰的、模块化和可扩展的编程方式，在并发和大规模数据处理等领域具有优势。C++11、Java 8、Python、Scala等编程语言都提供了支持函数式编程的特性。

本文将介绍C++中支持函数式编程的语言特性，包括函数对象、函数指针、Lambda表达式等。

## 2.函数对象
**函数对象**(function object)是指重载了**函数调用运算符**(function call operator) `operator()`的对象，从而能够像函数一样被调用。例如函数指针、Lambda表达式、`std::function`对象，以及任何重载了`operator()`的自定义类型的对象。函数调用运算符的参数列表和返回值可以是任意的。

例如，`Linear`对象表示一次函数 $y = ax + b$：

```cpp
// An object of this type represents a linear function y = a * x + b.
struct Linear {
    double a, b;
    double operator()(double x) const { return a * x + b; }
};

int main() {
    Linear f{2, 1};     // y = 2x + 1
    Linear g{-1, 0};    // y = -x
    // f and g are objects that can be used like a function.

    double f_0 = f(0);  // = 1
    double f_1 = f(1);  // = 3
    double g_2 = g(2);  // = -2
}
```

很多STL算法都接受函数对象以实现自定义行为。例如，`find_if()`自定义搜索条件，`sort()`自定义排序规则，`accumulate()`自定义“加法”操作，`for_each()`指定对每个元素执行的操作等等。

例如，下面的代码在一个`int`向量中分别查找大于42、大于n的元素：

```cpp
struct Larger_than {
    int v;
    bool operator()(int x) const { return x > v; }
};

void f(std::vector<int>& v, int n) {
    auto p = std::find_if(v.begin(), v.end(), Larger_than{42});
    if (p != v.end()) { /* ... */ }

    auto q = std::find_if(v.begin(), v.end(), Larger_than{n});
    if (q != v.end()) { /* ... */ }
}
```

其中，`Larger_than{42}`是一个函数对象，`Larger_than{42}(x)`等价于`x > 42`。

下面的代码对一个`Person`向量分别按姓名升序和年龄降序排序：

```cpp
struct Person {
    std::string name;
    int age;
};

struct Less_by_name {
    bool operator()(const Person& a, const Person& b) const { return a.name < b.name; }
};

struct Greater_by_age {
    bool operator()(const Person& a, const Person& b) const { return a.age > b.age; }
};

void f(std::vector<Person>& v) {
    // sort by name in ascending order
    std::sort(v.begin(), v.end(), Less_by_name());

    // sort by age in descending order
    std::sort(v.begin(), v.end(), Greater_by_age());
}
```

其中，`Less_by_name()`和`Greater_by_age()`是指定自定义排序规则的函数对象。

## 3.函数指针
**函数指针**(pointer to function)即存储函数地址的指针，可以使用普通函数或静态成员函数的地址初始化。例如：

```cpp
int f(int x) { return x + 1; }
int (*pf)(int) = &f;
int (*pf2)(int) = f;  // same as &f
```

由于**函数名可以隐式转换为函数指针**，因此取地址运算符`&`可以省略，`pf2`和`pf`的定义是等价的。

通过函数指针调用函数时，解引用运算符`*`也是可选的：

```cpp
int y = (*pf)(0);
int y2 = pf(0);  // same as (*pf)(0)
```

使用类型别名可以简化函数指针的声明：

```cpp
using F = int(int);
using PF = int(*)(int);   // same as F*
typedef int (*PF2)(int);  // same as PF
PF pf = &f;
```

其中`F`是接受一个`int`参数、返回`int`的函数类型，`PF`是指向这种函数的指针类型。

与普通指针一样，函数指针可以被赋值、传递给函数参数、作为函数返回值等等。例如，下面的代码在一个`int`向量中查找奇数元素：

```cpp
bool odd(int x) { return x % 2 == 1; }

void f(std::vector<int>& v) {
    auto p = std::find_if(v.begin(), v.end(), odd);
    if (p != v.end()) { /* ... */ }
}
```

下面的代码将向量中的每个元素加1并打印出来：

```cpp
void increase(int& n) { ++n; }

void print(int n) { std::cout << n << ' '; }

void f(std::vector<int>& v) {
    std::for_each(v.begin(), v.end(), increase);
    std::for_each(v.begin(), v.end(), print);
}
```

上一节中排序的例子也可以使用函数指针实现：

```cpp
bool less_by_name(const Person& a, const Person& b) { return a.name < b.name; }

bool greater_by_age(const Person& a, const Person& b) { return a.age > b.age; }

void f(std::vector<Person>& v) {
    // sort by name in ascending order
    std::sort(v.begin(), v.end(), less_by_name);

    // sort by age in descending order
    std::sort(v.begin(), v.end(), greater_by_age);
}
```

## 4.成员指针
### 4.1 数据成员指针
**数据成员指针**(pointer to data member)是指向类的数据成员的指针，用于访问和操作类的非静态数据成员。指向类`C`的非静态数据成员`m`的指针定义如下：

```cpp
T C::* pm = &C::m;
```

可以使用**成员指针访问运算符**(pointer-to-member access operator) `.*`和`->*`通过对象/对象指针和数据成员指针来访问数据成员的值。对于`C`类对象`c`及其指针`pc`，以下表达式是等价的，都表示访问对象`c`的数据成员`m`：

```cpp
c.m
pc->m
c.*pm
pc->*pm
```

在`C`的成员函数内部，`&(C::m)`和`&m`不是数据成员指针，而只是普通指针（前者仅用于静态数据成员）。

注：虽然`::`运算符的优先级高于`$`，但C++标准规定对于非静态数据成员，`&C::m`和`&(C::m)`是不同的：前者在类外是数据成员指针，在类内是普通指针，后者无论在类内还是类外都是非法的；而对于静态数据成员，二者无论在类内还是类外都是等价的。

> A pointer to non-static member object `m` which is a member of class `C` can be initialized with the expression `&C::m` exactly. Expressions such as `&(C::m)` or `&m` inside `C`'s member function do not form pointers to members.

例如：

```cpp
#include <iostream>

struct C { int m; };

int main() {
    int C::* p = &C::m;          // pointer to data member m of class C
    C c{7};
    std::cout << c.*p << '\n';   // prints 7
    C* cp = &c;
    cp->m = 10;
    std::cout << cp->*p << '\n'; // prints 10
    return 0;
}
```

数据成员指针也可以传递给函数，例如：

```cpp
#include <iostream>

struct Point { int x; int y; };

int get(const Point& p, int Point::*pm) {
    return p.*pm;
}

int main() {
    Point p = {3, 4};
    int x = get(p, &Point::x);  // same as p.x
    int y = get(p, &Point::y);  // same as p.y
    std::cout << x << ' ' << y << '\n';  // prints "3 4"
    return 0;
}
```

注意：**数据成员指针并未绑定到具体对象**，存储的仅仅是数据成员的偏移量（并不一定是这样，取决于编译器具体实现，但有助于理解），其值在编译时就是已知的。通过对象和数据成员指针访问数据成员时，实际上就是将对象的地址加上数据成员指针的值（偏移量）得到数据成员的地址。使用[Compiler Explorer](https://godbolt.org/)可以验证gcc编译器的实现方式：

```cpp
struct Point { int x; int y; };
 
int main() {
    int Point::* px = &Point::x;
    int Point::* py = &Point::y;
    Point p = {3, 4};
    int x = p.*px;
    return 0;
}
```

![数据成员指针汇编代码](/assets/images/cpp-functional-programming/数据成员指针汇编代码.png)

### 4.2 成员函数指针
**成员函数指针**(pointer to member function)是指向类的成员函数的指针，用于调用类的非静态成员函数。指向类`C`的非静态成员函数`f`的指针定义如下：

```cpp
R (C::* pf)(Args...) = &C::f;
```

可以使用`.*`和`->*`运算符通过对象/对象指针和成员函数指针来调用成员函数。对于`C`类对象`c`及其指针`pc`，以下表达式是等价的，都表示调用对象`c`的成员函数`f`：

```cpp
c.f(args)
pc->f(args)
(c.*pf)(args)
(pc->*pf)(args)
```

在`C`的成员函数内部，`&(C::f)`和`&f`不是成员函数指针，而只是普通函数指针（前者仅用于静态成员函数）。

例如：

```cpp
#include <iostream>

struct C {
    void f(int n) { std::cout << n << '\n'; }
};

int main() {
    void (C::* p)(int) = &C::f; // pointer to member function f of class C
    C c;
    (c.*p)(1);                  // prints 1
    C* cp = &c;
    (cp->*p)(2);                // prints 2
    return 0;
}
```

成员函数指针可以通过`std::mem_fn`或`std::bind`包装为函数对象，详见6.3和6.4节。

## 5.Lambda表达式
**Lambda表达式**(lambda expression)是（可以捕获当前作用域中变量的）匿名函数，也叫做**闭包**(closure)，在C++11中引入。它提供了一种方便的方式来编写函数，是函数式编程的一种重要特性。

Lambda表达式的语法如下：

```cpp
[captures](params) -> return_type { body }
```

其中，
* `captures`是捕获列表，用于捕获当前作用域中的变量（详见5.1节）
* `params`是参数列表，如果为空则可省略
* `return_type`是返回类型，可省略，编译器将会从函数体的`return`语句中自动推断
* `body`是函数体

Lambda表达式是一种函数对象，可以存储在变量中、直接调用或者传递给函数。例如：

```cpp
auto sum = [](int a, int b) { return a + b; };
int result = sum(3, 4);
```

对于简短的、一次性使用的函数，使用Lambda表达式可以在需要的地方直接内联定义，避免了显式定义命名函数的繁琐过程。前面几节中使用函数对象和函数指针的例子都可以使用Lambda表达式实现。例如，第3节中查找奇数元素的例子：

```cpp
void f(std::vector<int>& v) {
    auto p = std::find_if(v.begin(), v.end(), [](int x) { return x % 2 == 1; });
    if (p != v.end()) { /* ... */ }
}
```

自增并打印向量元素的例子：

```cpp
void f(std::vector<int>& v) {
    std::for_each(v.begin(), v.end(), [](int &n) { ++n; });
    std::for_each(v.begin(), v.end(), [](int n) { std::cout << n << ' '; });
}
```

排序的例子：

```cpp
void f(std::vector<Person>& v) {
    // sort by name in ascending order
    std::sort(v.begin(), v.end(), [](const Person& a, const Person& b) { return a.name < b.name; });

    // sort by age in descending order
    std::sort(v.begin(), v.end(), [](const Person& a, const Person& b) { return a.age > b.age; });
}
```

查找大于n的元素的例子：

```cpp
void f(std::vector<int>& v, int n) {
    auto p = find_if(v.begin(), v.end(), [](int x) { return x > 42; });
    if (p != v.end()) { /* ... */ }

    auto q = find_if(v.begin(), v.end(), [n](int x) { return x > n; });
    if (q != v.end()) { /* ... */ }
}
```

### 5.1 捕获列表
上面例子中的最后一个Lambda表达式`[n](int x) { return x > n; }`捕获了变量`n`，从而在函数体中可以访问它。

Lambda表达式的**捕获**(captures)是逗号分隔的列表，可以包含零个或多个捕获说明符，用于指定可以在函数体中访问的（除形参以外的）变量。 访问全局变量不需要捕获。常用捕获说明符的语法如下：

| 捕获说明符 | 含义 |
| --- | --- |
| `a` | 以拷贝方式捕获变量`a` |
| `&a` | 以引用方式捕获变量`a` |
| `this` | 以引用方式捕获当前对象 |
| `=` | 以拷贝方式隐式捕获所有用到的变量 |
| `&` | 以引用方式隐式捕获所有用到的变量 |

完整列表见[Lambda capture](https://en.cppreference.com/w/cpp/language/lambda#Lambda_capture)。

以上捕获说明符可以组合，`=`和`&`必须出现在捕获列表的开头。例如：

| 捕获列表 | 含义 |
| --- | --- |
| `[a, &b]` | 以拷贝方式捕获变量`a`，以引用方式捕获变量`b` |
| `[=, &a]` | 以引用方式捕获变量`a`，以拷贝方式捕获其他变量 |
| `[&, a]` | 以拷贝方式捕获变量`a`，以引用方式捕获其他变量 |

### 5.2 实现原理
**不带捕获的Lambda表达式可以被转换为函数指针，但带捕获的Lambda表达式不可以。** 例如：

```cpp
void f(int (*pf)(int));

void g(int n) {
    f([](int x) { return x + 1; });   // OK
    f([n](int x) { return x + n; });  // error: no matching function for call to 'f'
}
```

然而，从前面`find_if()`的例子中可以看出，带捕获和不带捕获的Lambda表达式都可以作为`find_if()`的第三个参数。

针对以上两个问题，需要理解Lambda表达式的实现原理。实际上，**编译器会为每个Lambda表达式生成一个匿名类**，也叫做闭包类型(closure type)，而Lambda表达式本身是这个类的对象。闭包类型具有以下成员：
* 捕获列表中的变量作为数据成员，以引用方式捕获的变量对应的数据成员是引用类型
* `ret operator()(params) { body }` 重载了`()`运算符，执行Lambda表达式的函数体，从而可以作为函数对象
* `using F = ret (*)(params); operator F() const noexcept;` 只有当捕获列表为空时才会定义函数指针类型转换运算符

下面使用[C++ Insights](https://cppinsights.io/)工具查看编译器对Lambda表达式的实际处理方式。

对于不带捕获的Lambda表达式`[](int x) { return x > 42; }`：

![Lambda表达式匿名类1](/assets/images/cpp-functional-programming/Lambda表达式匿名类1.png)

可以看到，编译器生成了一个闭包类型`__lambda_7_9`，并为其生成了`operator()`以及`bool (*)(int)`函数指针类型转换运算符，而Lambda表达式被替换为该类型对象`__lambda_7_9{}`。

对于带捕获的Lambda表达式`[n](int x) { return x > n; }`：

![Lambda表达式匿名类2](/assets/images/cpp-functional-programming/Lambda表达式匿名类2.png)

可以看到，编译器对于这个Lambda表达式生成的闭包类型定义与第2节中的`Larger_than`完全相同：具有数据成员`n`（对应捕获变量）和`operator()`，而并没有函数指针类型转换运算符，Lambda表达式被替换为`__lambda_7_9{n}`。

由此可见，Lambda表达式是否带捕获的区别仅仅是对应的闭包类型是否有数据成员、是否可转换为函数指针。

回到前面的问题，由于`find_if()`的第三个参数的类型是模板参数，将Lambda表达式传递给`find_if()`时，模板参数将被自动推导为对应的闭包类型，因此Lambda表达式是否带捕获对STL算法没有影响。

## 6.标准库函数对象
C++标准库提供了很多内置函数对象以及创建和操作函数对象的工具，定义在头文件[\<functional\>](https://en.cppreference.com/w/cpp/header/functional)中。

### 6.1 运算符函数对象
C++定义了一些表示常用运算符的函数对象（类似于Python的[`operator`模块](https://docs.python.org/3/library/operator.html)）。

算术运算符：

| 函数对象 | 运算符 |
| --- | --- |
| `plus` | `x + y` |
| `minus` | `x - y` |
| `multiplies` | `x * y` |
| `divides` | `x / y` |
| `modulus` | `x % y` |
| `negate` | `-x` |

比较运算符：

| 函数对象 | 运算符 |
| --- | --- |
| `equal_to` | `x == y` |
| `not_equal_to` | `x != y` |
| `greater` | `x > y` |
| `less` | `x < y` |
| `greater_equal` | `x >= y` |
| `less_equal` | `x <= y` |

逻辑运算符：

| 函数对象 | 运算符 |
| --- | --- |
| `logical_and` | `x && y` |
| `logical_or` | `x || y` |
| `logical_not` | `!x` |

按位运算符：

| 函数对象 | 运算符 |
| --- | --- |
| `bit_and` | `x & y` |
| `bit_or` | `x | y` |
| `bit_xor` | `x ^ y` |
| `bit_not` | `~x` |

这些都是类模板，模板参数是操作数类型。例如，`std::plus`的定义如下：

```cpp
template<class T>
struct plus {
    T operator()(const T& x, const T& y) const { return x + y; }
};
```

例如，计算一个向量所有元素的乘积：

```cpp
std::vector<double> v = {1.1, 2.2, 3.3, 4.4};
double product = std::accumulate(v.begin(), v.end(), 1.0, std::multiplies<double>());
```

其中，`multiplies<double>()`是表示乘法`*`的函数对象，`multiplies<double>()(x, y)`等价于`x * y`。

对一个向量进行倒序排序：

```cpp
std::vector<int> v = {1, 2, 3, 4};
std::sort(v.begin(), v.end(), std::greater<int>());
```

### 6.2 std::function
类模板`std::function`是一个通用的函数包装器，可以存储、拷贝和调用任何函数对象，包括函数指针、Lambda表达式、成员指针或其他函数对象。其声明如下：

```cpp
template<class R, class... Args>
class function<R(Args...)>;
```

模板参数是函数类型`R(Args...)`，其中`R`是返回值类型（可以是`void`），`Args`是逗号分隔的参数类型列表（可以为空）。

例如：

```cpp
#include <functional>
#include <iostream>

void print_num(int i) {
    std::cout << i << '\n';
}

struct Foo {
    int num_;
    void print_add(int i) const { std::cout << num_ + i << '\n'; }
};

struct PrintNum {
    void operator()(int i) const { std::cout << i << '\n'; }
};

int main() {
    // store a free function
    std::function<void(int)> f_print = print_num;
    f_print(-9);

    // store a lambda
    std::function<void()> f_print_42 = []() { print_num(42); };
    f_print_42();

    // store the result of a call to std::bind
    std::function<void()> f_print_31337 = std::bind(print_num, 31337);
    f_print_31337();

    // store a call to a member function
    std::function<void(const Foo&, int)> f_add_display = &Foo::print_add;
    const Foo foo{314159};
    f_add_display(foo, 1);

    // store a call to a data member accessor
    std::function<int(const Foo&)> f_num = &Foo::num_;
    std::cout << "num_: " << f_num(foo) << '\n';

    // store a call to a member function and object
    using std::placeholders::_1;
    std::function<void(int)> f_add_display2 = std::bind(&Foo::print_add, foo, _1);
    f_add_display2(2);

    // store a call to a member function and object ptr
    std::function<void(int)> f_add_display3 = std::bind(&Foo::print_add, &foo, _1);
    f_add_display3(3);

    // store a call to a function object
    std::function<void(int)> f_display_obj = PrintNum();
    f_display_obj(18);
    return 0;
}
```

程序输出：

```
-9
42
31337
314160
num_: 314159
314161
314162
18
```

#### 6.2.1 函数调用
从上面的例子中可以看出，对于不同类型的函数对象，`std::function`对调用参数的解释是不同的：对于普通函数，所有调用参数直接传递给函数；对于成员函数，第一个调用参数是被调用对象，其他参数传递给成员函数。

为了将各种情况统一起来，引入一个仅用于说明的操作`INVOKE(f, t1, t2, ..., tN)`，其定义如下：
* 如果`f`是类`T`的成员函数指针，则`INVOKE(f, t1, t2, ..., tN)`等价于
  * `(t1.*f)(t2, ..., tN)` 如果`t1`是`T`或其派生类的对象或引用
  * `(t1->*f)(t2, ..., tN)` 如果`t1`是`T`或其派生类的指针
* 否则，如果`N == 1`且`f`是类`T`的数据成员指针，则`INVOKE(f, t1)`等价于
  * `t1.*f` 如果`t1`是`T`或其派生类的对象或引用
  * `t1->*f` 如果`t1`是`T`或其派生类的指针
* 否则，如果`f`是函数对象，则`INVOKE(f, t1, t2, ..., tN)`等价于`f(t1, t2, ..., tN)`

`INVOKE<R>(f, t1, t2, ..., tN)`表示将`INVOKE(f, t1, t2, ..., tN)`的结果隐式转换为`R`。

详见[Function invocation](https://en.cppreference.com/w/cpp/utility/functional#Function_invocation)。

利用上述操作，`std::function<R(Args...)>`的`operator()`定义如下：

```cpp
R operator()(Args... args) const {
    return INVOKE<R>(f, std::forward<Args>(args)...);
}
```

其中`f`是`std::function`保存的函数对象。

#### 6.2.2 std::invoke
C++17引入了`std::invoke()`函数，等价于上述`INVOKE`操作，其定义如下：

```cpp
template<class F, class... Args>
std::invoke_result_t<F, Args...> invoke(F&& f, Args&&... args) {
    return INVOKE(std::forward<F>(f), std::forward<Args>(args)...);
}
```

C++23引入了`std::invoke_r()`函数，等价于上述`INVOKE<R>`操作：

```cpp
template<class R, class F, class... Args>
R invoke_r(F&& f, Args&&... args) {
    return INVOKE<R>(std::forward<F>(f), std::forward<Args>(args)...);
}
```

继续6.2节中的例子：

```cpp
int main() {
    // invoke a free function
    std::invoke(print_num, -9);

    // invoke a lambda
    std::invoke([]() { print_num(42); });

    // invoke a member function
    const Foo foo{314159};
    std::invoke(&Foo::print_add, foo, 1);

    // invoke (access) a data member
    std::cout << "num_: " << std::invoke(&Foo::num_, foo) << '\n';

    // invoke a function object
    std::invoke(PrintNum(), 18);

    auto add = [](int x, int y) { return x + y; };
    int ret = std::invoke(add, 11, 22);
    std::cout << ret << '\n';
    return 0;
}
```

程序输出：

```
-9
42
314160
num_: 314159
18
33
```

### 6.3 std::mem_fn
函数模板`std::mem_fn`返回一个成员指针包装器，可以存储、拷贝和调用成员指针。调用时可以使用对象的引用或指针（包括智能指针）。其声明如下：

```cpp
template<class T, class C>
/* unspecified */ mem_fn(T C::* pm) noexcept;
```

参数`pm`是类`C`的成员指针，`T`是数据成员类型或成员函数返回类型。返回值是一个函数对象，其`()`运算符的定义如下：

```cpp
template<class... Args>
std::invoke_result_t<decltype(pm), Args&&...> operator()(Args&&... args) {
    return INVOKE(pm, std::forward<Args>(args)...);
}
```

其中`args`的第一个参数是被调用对象的引用或指针。

注：这类似于Python的[实例方法](https://docs.python.org/3/reference/datamodel.html#instance-methods)：`c.f(args)`等价于`C.f(c, args)`，以及Java的[方法引用]({% post_url 2019-08-24-java-8-method-reference %})：`C::f`等价于`(c, args) -> c.f(args)`。

例如：

```cpp
#include <functional>
#include <iostream>

struct Foo {
    void display_greeting() {
        std::cout << "Hello, world.\n";
    }

    void display_number(int i) {
        std::cout << "number: " << i << '\n';
    }

    int add_xy(int x, int y) {
        return data + x + y;
    }

    int data = 7;
};

int main() {
    Foo f;

    auto greet = std::mem_fn(&Foo::display_greeting);
    greet(f);

    auto print_num = std::mem_fn(&Foo::display_number);
    print_num(f, 42);

    auto access_data = std::mem_fn(&Foo::data);
    std::cout << "data: " << access_data(f) << '\n';

    auto add_xy = std::mem_fn(&Foo::add_xy);
    std::cout << "add_xy: " << add_xy(f, 1, 2) << '\n';

    auto u = std::make_unique<Foo>();
    std::cout << "access_data(u): " << access_data(u) << '\n';
    std::cout << "add_xy(u, 1, 2): " << add_xy(u, 1, 2) << '\n';
    return 0;
}
```

程序输出：

```
Hello, world.
number: 42
data: 7
add_xy: 10
access_data(u): 7
add_xy(u, 1, 2): 10
```

其中，`print_num`大致等价于Lambda表达式`[](Foo& f, int i) { f.display_number(i); }`，调用`print_num(f, 42)`等价于`f.display_number(42)`，也可以写成`print_num(&f, 42)`。

### 6.4 std::bind
函数模板`std::bind`返回一个函数包装器，通过固定（绑定）函数的部分参数得到一个新的函数，即[部分应用](https://en.wikipedia.org/wiki/Partial_application)(partial application)（类似于Python的[`functools.partial()`](https://docs.python.org/3/library/functools.html#functools.partial)）。例如，设 $f(x, y) = x + y$，固定 $y = 1$，得到 $g(x) = f(x, 1) = x + 1$。

其声明如下：

```cpp
template<class F, class... Args>
/* unspecified */ bind(F&& f, Args&&... args);
```

其中，参数列表`args`的长度应该和`f`的形参个数相同，绑定的参数将被传递给`f`，未绑定的参数使用命名空间`std::placeholders`中的占位符`_1`、`_2`、`_3`等表示，对应新函数的形参。

例如：

```cpp
#include <functional>
#include <iostream>

int sub(int x, int y) { return x - y; }

struct Foo {
    int num_;
    void print_add(int i) const { std::cout << num_ + i << '\n'; }
};

void f(int n1, int n2, int n3, const int& n4, int n5) {
    std::cout << n1 << ' ' << n2 << ' ' << n3 << ' ' << n4 << ' ' << n5 << '\n';
}

int main() {
    using namespace std::placeholders;

    // same as [](int x) { return sub(x, 5); }
    auto sub_five = std::bind(sub, _1, 5);
    std::cout << sub_five(8) << '\n';

    // same as [](int x) { return sub(5, x); }
    auto five_sub = std::bind(sub, 5, _1);
    std::cout << five_sub(8) << '\n';

    Foo foo{6};
    // same as [&foo](int i) { return foo.print_add(i); }
    auto foo_print_add = std::bind(&Foo::print_add, foo, _1);
    foo_print_add(10);

    int n = 7;
    // same as [&r = n, n](int a, int b) { f(b, 42, a, r, n); }
    auto f1 = std::bind(f, _2, 42, _1, std::cref(n), n);
    f1(1, 2);
    return 0;
}
```

程序输出：

```
3
-3
16
2 42 1 7 7
```

注：4.1节中提到，成员指针并未绑定到具体对象，而上面示例中的`std::bind(&Foo::print_add, foo, _1)`将成员函数指针`&Foo::print_add`绑定到对象`foo`，这类似于Java的方法引用：`c::f`等价于`(args) -> c.f(args)`。

### 6.5 std::reference_wrapper
类模板`std::reference_wrapper`将引用包装为一个可拷贝、可赋值的对象，通常用于将引用存储在STL容器中，或者将对象按引用方式传递给`std::bind`、`std::thread`、`std::make_pair`等。

注：普通引用本身无法拷贝和赋值，必须借助`std::reference_wrapper`实现，而指针本身就支持这些操作。实际上，`std::reference_wrapper`的底层实现就是保存了一个指针。

`std::reference_wrapper<T>`对象可隐式转换为`T&`。如果保存的引用是可调用的（即函数对象），则`std::reference_wrapper`也可以使用相同的参数调用。

通常不直接使用这个类模板，而是使用辅助函数`std::ref`和`std::cref`来创建`std::reference_wrapper`对象。

例如：

```cpp
#include <functional>
#include <iostream>

void f(int& n1, int& n2, const int& n3) {
    std::cout << "In function: " << n1 << ' ' << n2 << ' ' << n3 << '\n';
    ++n1;  // increments the copy of n1 stored in the function object
    ++n2;  // increments the main()'s n2
}

int main() {
    int n1 = 1, n2 = 2, n3 = 3;
    std::function<void()> bound_f = std::bind(f, n1, std::ref(n2), std::cref(n3));
    n1 = 10;
    n2 = 11;
    n3 = 12;
    std::cout << "Before function: " << n1 << ' ' << n2 << ' ' << n3 << '\n';
    bound_f();
    std::cout << "After function: " << n1 << ' ' << n2 << ' ' << n3 << '\n';
    return 0;
}
```

程序输出：

```
Before function: 10 11 12
In function: 1 11 12
After function: 10 12 12
```

## 7.Ranges库
C++20引入的ranges库是对STL算法库的扩展，使得算法和迭代器可以通过组合变得更加强大。Ranges库的核心概念是**视图**(view)，是间接表示可迭代序列（**范围**(range)）的轻量级对象。头文件[\<ranges\>](https://en.cppreference.com/w/cpp/header/ranges)提供了创建和操作视图的功能。

在C++20之前无法用Scala的方式实现第1节中计算偶数平方和的例子，而利用ranges库可以做到：

```cpp
int arr[] = {0, 1, 2, 3, 4, 5, 6};
auto even = arr | std::views::filter([](int i) { return i % 2 == 0; })
        | std::views::transform([](int i) { return i * i; });
int result = std::accumulate(even.begin(), even.end(), 0);
```

## 参考
* [Functional programming - Wikipedia](https://en.wikipedia.org/wiki/Functional_programming)
* [named requirement FunctionObject - cppreference](https://en.cppreference.com/w/cpp/named_req/FunctionObject)
* [Function call operator - cppreference](https://en.cppreference.com/w/cpp/language/operators#Function_call_operator)
* [Pointers to functions - cppreference](https://en.cppreference.com/w/cpp/language/pointer#Pointers_to_functions)
* [Pointers to members - cppreference](https://en.cppreference.com/w/cpp/language/pointer#Pointers_to_members)
* [Lambda expressions - cppreference](https://en.cppreference.com/w/cpp/language/lambda)
* [Function objects - cppreference](https://en.cppreference.com/w/cpp/utility/functional)
* [Ranges library - cppreference](https://en.cppreference.com/w/cpp/ranges)
