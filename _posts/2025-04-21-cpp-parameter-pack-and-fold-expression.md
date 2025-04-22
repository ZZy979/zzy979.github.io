---
title: 【C++】参数包与折叠表达式
date: 2025-04-21 22:25:35 +0800
categories: [C/C++]
tags: [cpp, template, parameter pack, fold expression]
---
## 1.参数包
**参数包**(parameter pack)是C++11引入的模板特性，允许模板接受可变数量的参数，用语法`...`表示。参数包有两种形式：
* **模板参数包**(template parameter pack)是接受零个或多个模板实参的模板形参，可以出现在类模板和函数模板的形参列表中。
* **函数参数包**(function parameter pack)是接受零个或多个函数实参的函数形参，只能出现在函数模板的形参列表中。

带有参数包的模板称为**变参模板**(variadic template)。

变参类模板可以用任意数量的模板实参实例化。例如：

```cpp
template<class... Types>  // template parameter pack
struct Tuple {};

Tuple<> t0;           // Types contains no arguments
Tuple<int> t1;        // Types contains one argument: int
Tuple<int, float> t2; // Types contains two arguments: int and float
Tuple<0> t3;          // error: 0 is not a type
```

变参函数模板可以用任意数量的函数实参调用（模板参数可以自动推导）。例如：

```cpp
template<class... Args>  // template parameter pack
void f(Args... args);    // function parameter pack

f();       // OK: args contains no arguments
f(1);      // OK: args contains one argument: int
f(2, 1.0); // OK: args contains two arguments: int and double
```

注：变参函数模板类似于[变长参数](https://en.cppreference.com/w/cpp/language/variadic_arguments)，不过前者是基于模板实现的，后者是基于头文件\<cstdarg\>。

### 1.1 包展开
如果模式`p`是一个带有包名字的表达式，则`p...`会被展开成零个或多个逗号分隔的模式实例，其中包名字依次被替换为包中的各个元素。例如：

```cpp
template<class... Us>
void f(Us... pargs) {}
 
template<class... Ts>
void g(Ts... args) {
    f(&args...); // "&args..." is a pack expansion, "&args" is its pattern
}
 
g(1, 0.2, "a"); // Ts... args expand to int E1, double E2, const char* E3
                // &args... expands to &E1, &E2, &E3
                // Us... pargs expand to int* E1, double* E2, const char** E3
```

上面的代码等价于

```cpp
void f(int* parg1, double* parg2, const char** parg3) {}

void g(int arg1, double arg2, const char* arg3) {
    f(&arg1, &arg2, &arg3);
}

g(1, 0.2, "a");
```

### 1.2 sizeof...运算符
`sizeof...(pack)`返回包`pack`的元素个数。例如：

```cpp
#include <array>
#include <iostream>
#include <type_traits>

template<class... Ts>
auto make_array(Ts... ts) {
    using CT = std::common_type_t<Ts...>;
    return std::array<CT, sizeof...(Ts)>{ts...};
}

int main() {
    std::array<double, 4> arr = make_array(1, 2.71f, 3.14, '*');
    for (double elem : arr)
        std::cout << elem << ' ';
    return 0;
}
```

程序输出如下：

```
1 2.71 3.14 42 
```

### 1.3 示例
（1）计算任意数量参数的和

例如`sum(1, 2, 3, 4)`应该返回10。可以使用参数包递归展开的方式实现：

```cpp
#include <iostream>
#include <string>

template<class T>
T sum(T t) {
    return t;
}

template<class T, class... Ts>
T sum(T t0, Ts... ts) {
    return t0 + sum(ts...);
}

int main() {
    using namespace std::string_literals;
    std::cout << sum(1, 2, 3, 4) << '\n';  // prints "10"
    std::cout << sum("Hello"s, ", "s, "world"s, "!"s) << '\n';  // prints "Hello, world!"
    return 0;
}
```

（2）std::make_unique

标准库头文件\<memory\>中的函数`std::make_unique<T>()`使用给定的参数构造一个`T`类型的对象，并返回拥有该对象的`std::unique_ptr<T>`。该函数就是使用参数包实现的，其中一个重载的定义如下：

```cpp
template<class T, class... Args>
std::unique_ptr<T> make_unique(Args&&... args) {
    return std::unique_ptr<T>(new T(std::forward<Args>(args)...));
}
```

其中`Args&&`是转发引用，包展开模式`std::forward<Args>(args)`利用了完美转发（详见[【C++】右值引用、移动语义和完美转发]({% post_url 2023-06-02-cpp-rvalue-reference-move-semantics-and-perfect-forwarding %})）。

下面是一个测试程序：

```cpp
#include <iostream>
#include <memory>

class C {
public:
    C() { std::cout << "C()\n"; }
    C(const C& c) { std::cout << "C(const C&)\n"; }
    C(C&& c) noexcept { std::cout << "C(C&&)\n"; }
    C(int, double) { std::cout << "C(int, double)\n"; }
};

template<class T, class... Args>
std::unique_ptr<T> make_unique(Args&&... args) {
    return std::unique_ptr<T>(new T(std::forward<Args>(args)...));
}

int main() {
    C c;
    auto p1 = make_unique<C>();             // (1) calls C()
    auto p2 = make_unique<C>(c);            // (2) calls C(const C&)
    auto p3 = make_unique<C>(std::move(c)); // (3) calls C(C&&)
    auto p4 = make_unique<C>(1, 2.5);       // (4) calls C(int, double)
    return 0;
}
```

使用[C++ Insights](https://cppinsights.io/s/2c3d8372)工具可以看到，这四个调用对应的模板实例化如下：

```cpp
// (1) T = C, Args = {}
template<>
std::unique_ptr<C> make_unique<C>() {
    return std::unique_ptr<C>(new C());
}

// (2) T = C, Args = {C&}
template<>
std::unique_ptr<C> make_unique<C, C&>(C& arg0) {
    return std::unique_ptr<C>(new C(std::forward<C&>(arg0)));
}

// (3) T = C, Args = {C}
template<>
std::unique_ptr<C> make_unique<C, C>(C&& arg0) {
    return std::unique_ptr<C>(new C(std::forward<C>(arg0)));
}

// (4) T = C, Args = {int, double}
template<>
std::unique_ptr<C> make_unique<C, int, double>(int&& arg0, double&& arg1) {
    return std::unique_ptr<C>(new C(std::forward<int>(arg0), std::forward<double>(arg1)));
}
```

## 2.折叠表达式
C++17引入了**折叠表达式**(fold expression)，使用二元运算符对包进行归约(reduce)/折叠(fold)。

折叠表达式有四种形式：

| 形式 | 语法 | 含义 |
| --- | --- | --- |
| 一元右折叠 | `(E op ...)` | <code>(E<sub>1</sub> op (... op (E<sub>N-1</sub> op E<sub>N</sub>)))</code> |
| 一元左折叠 | `(... op E)` | <code>(((E<sub>1</sub> op E<sub>2</sub>) op ...) op E<sub>N</sub>)</code> |
| 二元右折叠 | `(E op ... op I)` | <code>(E<sub>1</sub> op (... op (E<sub>N−1</sub> op (E<sub>N</sub> op I))))</code> |
| 二元左折叠 | `(I op ... op E)` | <code>((((I op E<sub>1</sub>) op E<sub>2</sub>) op ...) op E<sub>N</sub>)</code> |

其中，`op`是二元运算符，`E`是包含未展开包的表达式，`I`是不包含未展开包的表达式。注意，两边的圆括号是必需的。

### 2.1 示例
（1）计算任意数量参数的和

再次考虑1.3节中计算参数和的示例，使用折叠表达式可以更加简洁地实现：

```cpp
template<class... Ts>
auto sum(Ts... ts) {
    return (... + ts);
}
```

调用`sum(1, 2, 3, 4)`等价于`((1 + 2) + 3) + 4`。

（2）打印任意数量的参数

为了打印所有参数，可以使用`<<`运算符的二元左折叠：

```cpp
#include <iostream>

template<class... Args>
void printer(Args&&... args) {
    (std::cout << ... << args) << '\n';
}

int main() {
    printer(1, 2.5, "abc");  // prints "12.5abc"
    return 0;
}
```

调用`printer(1, 2.5, "abc")`等价于`((std::cout << 1) << 2.5) << "abc"`。

如果希望打印分隔符，则可以使用`,`运算符的一元折叠（`,`运算符是从左到右求值的）：

```cpp
#include <iostream>

template<class... Args>
void printer(Args&&... args) {
    ((std::cout << args << ' '), ...) << '\n';
}

int main() {
    printer(1, 2.5, "abc");  // prints "1 2.5 abc "
    return 0;
}
```

调用`printer(1, 2.5, "abc")`等价于`(std::cout << 1 << ' '), ((std::cout << 2.5 << ' '), (std::cout << "abc" << ' '))`。

（3）打印元组

下面考虑另一个例子：打印一个元组的所有元素，例如：

```cpp
auto t = std::make_tuple(42, "Foo", 3.14);
print_tuple(t);  // prints "42 Foo 3.14"
```

C++的`std::tuple`不是可迭代的，只能通过`std::get<I>()`获取第`I`个元素。但模板实参`I`必须是编译时常量，因此无法直接用`for`循环遍历：

```cpp
template<class Tuple>
void print_tuple(const Tuple& t) {
    for (std::size_t i = 0; i < std::tuple_size_v<Tuple>; ++i)
        std::cout << std::get<i>(t) << ' ';  // error: 'i' is not constant expression
}
```

为此，需要借助`std::integer_sequence`。这个类模板的模板参数包含一个整数序列，其本身没有任何成员：

```cpp
template<class T, T... Ints>
class integer_sequence;

template<std::size_t... Ints>
using index_sequence = std::integer_sequence<std::size_t, Ints...>;
```

辅助别名模板`std::index_sequence_for`用于创建与参数包长度相同的索引序列：

```cpp
template<class... T>
using index_sequence_for = std::make_index_sequence<sizeof...(T)>;
```

例如，`std::index_sequence_for<int, const char*, double>`等价于`std::index_sequence<0, 1, 2>`。

有了这些，就可以根据元组的模板参数列表构造一个索引序列，然后使用其模板参数中的整数和折叠表达式实现打印元组的所有元素。

```cpp
#include <iostream>
#include <string>
#include <tuple>

template<class Tuple, std::size_t... I>
void print_tuple_impl(const Tuple& t, std::index_sequence<I...>) {
    ((std::cout << std::get<I>(t) << ' '), ...);
}

template<class... Args>
void print_tuple(const std::tuple<Args...>& t) {
    print_tuple_impl(t, std::index_sequence_for<Args...>{});
}

int main() {
    auto t = std::make_tuple(42, "Foo", 3.14);
    print_tuple(t);  // prints "42 Foo 3.14"
    return 0;
}
```

使用[C++ Insights](https://cppinsights.io/s/436028dd)工具可以看到，这个示例中的模板实例化如下：

```cpp
template<>
void print_tuple_impl<std::tuple<int, const char *, double>, 0, 1, 2>(
        const std::tuple<int, const char *, double>& t, std::integer_sequence<unsigned long, 0, 1, 2>) {
    (std::cout << std::get<0>(t) << ' '), ((std::cout << std::get<1>(t) << ' '), (std::cout << std::get<2>(t) << ' '));
}

template<>
void print_tuple<int, const char *, double>(const std::tuple<int, const char *, double>& t) {
    print_tuple_impl(t, std::integer_sequence<unsigned long, 0, 1, 2>{});
}
```

## 参考
* [Pack - cppreference](https://en.cppreference.com/w/cpp/language/pack)
* [Fold expressions - cppreference](https://en.cppreference.com/w/cpp/language/fold)
