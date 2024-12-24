---
title: C++20范围库
date: 2024-12-15 10:32:43 +0800
categories: [C/C++]
tags: [cpp, ranges library, functional programming]
math: true
---
## 1.引言
C++20引入的范围(Ranges)库是对STL算法库的扩展，使算法和迭代器可以组合，从而使其功能更加强大。范围库为函数式编程提供了更好的支持。

例如，下面的示例程序输出一个整数数组中所有偶数的平方。

```cpp
#include <iostream>
#include <ranges>

int main() {
    int arr[] = {0, 1, 2, 3, 4, 5};
    auto even = [](int i) { return i % 2 == 0; };
    auto square = [](int i) { return i * i; };

    // the "pipe" syntax of composing the views:
    for (int i : arr | std::views::filter(even) | std::views::transform(square))
        std::cout << i << ' ';
    std::cout << '\n';

    // a traditional "functional" composing syntax:
    for (int i : std::views::transform(std::views::filter(arr, even), square))
        std::cout << i << ' ';
    return 0;
}
```

程序输出如下：

```
0 4 16 
0 4 16 
```

本文将介绍范围、视图等基本概念，以及范围算法相比于传统STL算法的优势，最后将介绍如何自定义视图。

注：支持范围库的编译器最低版本是GCC 10和Clang 13，编译时需要添加选项`-std=c++20`。参见[C++ compiler support](https://en.cppreference.com/w/cpp/compiler_support)。

## 2.范围
**范围**(range)是对**可迭代序列**的抽象，即提供了`begin()`和`end()`迭代器、允许对其元素进行迭代的类型。内置数组和所有STL容器都是范围。

范围可用于foreach循环，或者传递给范围算法。例如，头文件\<algorithm\>中的算法`std::ranges::sort()`可以对给定的范围进行排序：

```cpp
std::vector<int> v = ...;
std::ranges::sort(v);  // with C++20
std::sort(v.begin(), v.end());  // before C++20
```

大部分传统的STL算法接受一对迭代器，而范围算法可以将容器作为单个参数传递。范围算法将在2.2节详细介绍。

为了简单起见，下面将命名空间`std::ranges`简称为`ranges`。

### 2.1 范围概念
头文件\<ranges\>中的概念[ranges::range](https://en.cppreference.com/w/cpp/ranges/range)给出了“范围”的定义：

```cpp
template<class T>
concept range = requires(T& t) {
    ranges::begin(t);
    ranges::end(t);
};
```

简单来说，范围就是可以传递给函数`ranges::begin()`和`ranges::end()`的类型。这两个函数的定义如下：
* 如果`t`是数组类型`T[N]`，则分别返回`t + 0`和`t + N`。
* 否则，返回`t.begin()`和`t.end()`，如果`T`定义了该成员函数，且返回类型是输入或输出迭代器（`std::input_or_output_iterator`，即支持`*`、`++`、`==`和`!=`操作）。例如STL容器。
* 否则，分别返回`begin(t)`和`end(t)`，如果该表达式合法且返回类型是输入或输出迭代器。
* 否则，该函数调用是非法的。

注：
* 范围的`end()`迭代器（也叫做**哨兵**(sentinel)）类型不必和`begin()`相同。例如，可以用`'\0'`结束一个C风格字符串，空字符串结束一个单词列表，空指针结束一个链表，或者-1结束一个非负数列表。
* 在C++20之前，标准库还有一对类似的函数`std::begin()`和`std::end()`，定义在各容器对应的头文件中。二者的区别详见[What is the difference between std::ranges::begin and std::begin?](https://stackoverflow.com/questions/62183286/what-is-the-difference-between-stdrangesbegin-and-stdbegin)
* 头文件\<ranges\>中还定义了其他的范围函数，如`ranges::size()`、`ranges::empty()`等。
* 概念`concept`也是C++20的新特性，详见[《C++20之概念》]({% post_url 2024-03-24-cpp20-concept %})。

根据其迭代器类别，范围也可以分为不同的类别，分类方式与迭代器类似：

| 范围概念 | 迭代器概念 |
| --- | --- |
| `ranges::input_range` | `std::input_iterator` |
| `ranges::output_range` | `std::output_iterator` |
| `ranges::forward_range` | `std::forward_iterator` |
| `ranges::bidirectional_range` | `std::bidirectional_iterator` |
| `ranges::random_access_range` | `std::random_access_iterator` |
| `ranges::contiguous_range` | `std::contiguous_iterator` |

可以使用上述概念检查一个类是否是（某种类别的）范围。例如：

```cpp
#include <forward_list>
#include <map>
#include <ranges>
#include <string>
#include <vector>

// A minimum range
struct SimpleRange {
    int* begin();
    int* end();
};

// Not a range: no begin/end
struct NotRange {
    int t;
};

// Not a range: begin does not return an input_or_output_iterator
struct NotRange2 {
    void* begin();
    int* end();
};

static_assert(std::ranges::random_access_range<int[10]>);
static_assert(std::ranges::random_access_range<std::vector<int>>);
static_assert(std::ranges::random_access_range<std::string>);
static_assert(std::ranges::bidirectional_range<std::map<int, int>>);
static_assert(std::ranges::forward_range<std::forward_list<double>>);
static_assert(std::ranges::range<SimpleRange>);

static_assert(!std::ranges::range<int[]>);
static_assert(!std::ranges::range<NotRange>);
static_assert(!std::ranges::range<NotRange2>);
```

### 2.2 范围算法
范围库为头文件\<algorithm\>中的大多数算法提供了基于范围的版本，定义在命名空间`std::ranges`中。例如`find`、`count`、`sort`、`for_each`等。完整列表参见[Constrained algorithms](https://en.cppreference.com/w/cpp/algorithm/ranges)。

与传统STL算法相比，范围算法有以下优点，从而使用起来更加灵活：
* 可以通过一对迭代器或单个参数指定范围。
* 支持元素映射和成员指针。
* 改变了大多数算法的返回类型，以返回可能有用的信息。

下面以几个常用的算法为例介绍范围算法的用法。

#### 2.2.1 sort
首先考虑`sort`算法。传统的`std::sort()`有以下两种常用形式：

```cpp
template<class RandomIt>
void sort(RandomIt first, RandomIt last);

template<class RandomIt, class Compare>
void sort(RandomIt first, RandomIt last, Compare comp);
```

第一种形式使用`<`对`[first, last)`中的元素进行排序，第二种形式使用自定义比较器`comp`。

范围库提供的`ranges::sort()`有以下两种形式：

```cpp
template<std::random_access_iterator I, std::sentinel_for<I> S,
    class Comp = ranges::less, class Proj = std::identity>
sort(I first, S last, Comp comp = {}, Proj proj = {});

template<ranges::random_access_range R,
    class Comp = ranges::less, class Proj = std::identity>
sort(R&& r, Comp comp = {}, Proj proj = {});
```

第一种形式等价于`std::sort(first, last, comp)`，但额外提供了`proj`参数，用于对元素进行映射操作，即对映射后的结果而不是元素本身进行排序（相当于Python中`sort()`函数的`key`参数），默认为恒等映射`std::identity`。第二种形式等价于`ranges::sort(ranges::begin(r), ranges::end(r), comp, proj)`。

下面的程序展示了`sort`算法的用法。

```cpp
#include <algorithm>
#include <iomanip>
#include <iostream>
#include <string>
#include <vector>

namespace ranges = std::ranges;

void print(const std::string& comment, const auto& seq, char sep = ' ') {
    for (std::cout << comment << '\n'; const auto& e : seq)
        std::cout << e << sep;
    std::cout << '\n';
}

struct Particle {
    std::string name;
    double mass; // MeV
};

std::ostream& operator<<(std::ostream& os, const Particle& p) {
    return os << std::left << std::setw(8) << p.name << " : " << p.mass;
}

int main() {
    std::vector<int> v = {5, 7, 4, 2, 8, 6, 1, 9, 0, 3};
    ranges::sort(v);
    print("Sort using operator<", v);

    ranges::sort(v, ranges::greater{});
    print("Sort using ranges::greater", v);

    Particle particles[] {
        {"Electron", 0.511}, {"Muon", 105.66}, {"Tau", 1776.86},
        {"Positron", 0.511}, {"Proton", 938.27}, {"Neutron", 939.57}
    };
    ranges::sort(particles, {}, &Particle::name);
    print("\nSort by name", particles, '\n');

    ranges::sort(particles, ranges::greater{}, &Particle::mass);
    print("Sort by mass in descending order", particles, '\n');
    return 0;
}
```

程序输出如下：

```
Sort using operator<
0 1 2 3 4 5 6 7 8 9 
Sort using ranges::greater
9 8 7 6 5 4 3 2 1 0 

Sort by name
Electron : 0.511
Muon     : 105.66
Neutron  : 939.57
Positron : 0.511
Proton   : 938.27
Tau      : 1776.86

Sort by mass in descending order
Tau      : 1776.86
Neutron  : 939.57
Proton   : 938.27
Muon     : 105.66
Electron : 0.511
Positron : 0.511
```

#### 2.2.2 find_if
`find_if`算法用于查找满足给定条件的元素。传统的`std::find_if()`实现如下：

```cpp
template<class InputIt, class Pred>
InputIt find_if(InputIt first, InputIt last, Pred pred) {
    for (; first != last; ++first)
        if (pred(*first))
            return first;
    return last;
}
```

`ranges::find_if()`有以下两种形式（忽略复杂的模板参数）：

```cpp
template<std::input_iterator I, std::sentinel_for<I> S, class Proj = std::identity,
         std::indirect_unary_predicate<std::projected<I, Proj>> Pred>
I find_if(I first, S last, Pred pred, Proj proj = {}) {
    for (; first != last; ++first)
        if (std::invoke(pred, std::invoke(proj, *first)))
            return first;
    return first;
}

template<ranges::input_range R, class Proj = std::identity,
         std::indirect_unary_predicate<std::projected<ranges::iterator_t<R>, Proj>> Pred>
ranges::borrowed_iterator_t<R> find_if(R&& r, Pred pred, Proj proj = {}) {
    return find_if(ranges::begin(r), ranges::end(r), std::ref(pred), std::ref(proj));
}
```

传统STL算法的谓词参数可以是普通函数、Lambda表达式或函数对象，但不能是成员指针（除非使用`std::mem_fn`包装）。这是由于成员指针的调用语法与其他函数对象不同（使用`.*`或`->*`而不是`()`）。而范围算法都支持。

从上面的实现中可以看出，`std::find_if()`直接使用`()`运算符来调用谓词，这不适用于成员指针；而`ranges::find_if()`使用了`std::invoke()`，该函数为成员指针和其他函数对象提供了统一的调用语法。参见[《C++函数式编程》]({% post_url 2023-10-04-cpp-functional-programming %}) 6.2.2节和6.3节“注”。

下面的程序展示了`find_if`的使用示例。

```cpp
#include <algorithm>
#include <iostream>
#include <string>
#include <vector>

namespace ranges = std::ranges;

struct User {
    std::string name;
    int age;
    bool is_adult() const { return age >= 18; }
};

std::ostream& operator<<(std::ostream& os, const User& u) {
    return os << '(' << u.name << ',' << u.age << ')';
}

class Larger_than {
public:
    explicit Larger_than(int v) :v(v) {}
    bool operator()(int x) const { return x > v; }

private:
    int v;
};

int main() {
    std::vector<int> v = {4, 1, 3, 2};
    for (int n : {3, 5}) {
        if (ranges::find(v, n) != v.end())
            std::cout << "v contains " << n << '\n';
        else
            std::cout << "v does not contain " << n << '\n';
    }

    auto is_even = [](int x) { return x % 2 == 0; };
    if (auto it = ranges::find_if(v, is_even); it != v.end())
        std::cout << "First even element in v: " << *it << '\n';
    else
        std::cout << "No odd element in v\n";

    if (auto it = ranges::find_if(v, Larger_than(42)); it != v.end())
        std::cout << "First element larger than 42 in v: " << *it << '\n';
    else
        std::cout << "No element larger than 42 in v\n";

    User users[] = { {"Alice", 17}, {"Bob", 25}, {"Cindy", 19}, {"Dale", 20}, {"Eric", 22} };
    if (auto it = ranges::find_if(users, &User::is_adult); it != ranges::end(users))
        std::cout << "First adult in users: " << *it << '\n';
    else
        std::cout << "No adult in users\n";

    std::string s = "Frank";
    if (auto it = ranges::find(users, s, &User::name); it != ranges::end(users))
        std::cout << "Found " << s << " in users: " << *it << '\n';
    else
        std::cout << "Not found " << s << " in users\n";
    return 0;
}
```

程序输出如下：

```
v contains 3
v does not contain 5
First even element in v: 4
No element larger than 42 in v
First adult in users: (Bob,25)
Not found Frank in users
```

其中，查找成年用户的例子使用了成员函数指针`&User::is_adult`作为谓词，而`std::find_if()`只能使用lambda表达式或`std::mem_fn`：

```cpp
auto p = std::find_if(users.begin(), users.end(), [](const User& u) { return u.is_adult(); });
auto q = std::find_if(users.begin(), users.end(), std::mem_fn(&User::is_adult));
```

#### 2.2.3 for_each
`std::for_each()`算法对`[first, last)`中的每个元素调用函数`f`：

```cpp
template<class InputIt, class Fun>
Fun for_each(InputIt first, InputIt last, Fun f) {
    for (; first != last; ++first)
        f(*first);
    return f;
}
```

算法最终返回`f`，这对于有状态的函数对象可能会有用（例如下面示例中的`Sum`）。

对应的`ranges::for_each()`实现如下：

```cpp
template<std::input_iterator I, std::sentinel_for<I> S, class Proj = std::identity,
         std::indirectly_unary_invocable<std::projected<I, Proj>> Fun>
ranges::for_each_result<I, Fun> for_each(I first, S last, Fun f, Proj proj = {}) {
    for (; first != last; ++first)
        std::invoke(f, std::invoke(proj, *first));
    return {std::move(first), std::move(f)};
}

template<ranges::input_range R, class Proj = std::identity,
         std::indirectly_unary_invocable<std::projected<ranges::iterator_t<R>, Proj>> Fun>
ranges::for_each_result<ranges::borrowed_iterator_t<R>, Fun> for_each(R&& r, Fun f, Proj proj = {}) {
    return for_each(ranges::begin(r), ranges::end(r), std::move(f), std::ref(proj));
}
```

除了增加映射参数和改变调用语法外，`ranges::for_each()`还返回了`first`迭代器的最终位置。

例如：

```cpp
#include <algorithm>
#include <cassert>
#include <iostream>
#include <string>
#include <utility>
#include <vector>

namespace ranges = std::ranges;

struct Sum {
    void operator()(int n) { sum += n; }
    int sum = 0;
};

int main() {
    std::vector<int> nums = {3, 4, 2, 8, 15, 267};
    auto print = [](const auto& n) { std::cout << n << ' '; };

    std::cout << "before: ";
    ranges::for_each(nums, print);

    ranges::for_each(nums, [](int& n) { ++n; });

    // calls Sum::operator() for each number
    auto [i, s] = ranges::for_each(nums, Sum{});
    assert(i == nums.end());

    std::cout << "\nafter: ";
    ranges::for_each(nums, print);
    std::cout << "\nsum: " << s.sum << '\n';

    using pair = std::pair<int, std::string>;
    std::vector<pair> pairs = { {1, "one"}, {2, "two"}, {3, "three"} };

    std::cout << "project the pair::first: ";
    ranges::for_each(pairs, print, [](const pair& p) { return p.first; });

    std::cout << "\nproject the pair::second: ";
    ranges::for_each(pairs, print, &pair::second);
    std::cout << '\n';
    return 0;
}
```

程序输出如下：

```
before: 3 4 2 8 15 267 
after: 4 5 3 9 16 268 
sum: 305
project the pair::first: 1 2 3 
project the pair::second: one two three 
```

## 3.视图
**视图**(view)是间接表示范围的轻量级对象。视图不拥有数据，而是在迭代时惰性计算或生成元素，其拷贝、移动和赋值等操作具有常数时间复杂度。

标准库提供的视图定义在头文件\<ranges\>中，主要包括两大类：
* 范围工厂：在迭代时生成元素，如`ranges::iota_view`。
* 范围适配器：对底层范围的包装器，在迭代时进行惰性计算，如`ranges::filter_view`。

视图是一种特殊的范围，因此也可用于foreach循环或者传递给范围算法。STL容器是范围，但不是视图。

注：C++中的范围和视图可分别类比为Python中的可迭代对象(iterable)和生成器(generator)。

### 3.1 范围工厂
**范围工厂**(range factory)是在迭代时生成元素的视图。

例如，视图`ranges::iota_view`用于生成连续的整数序列（类似于Python的`range()`函数）：
* `ranges::iota_view(m, n)`生成有限序列m, m+1, ..., n-1（注意，如果n < m则迭代不会终止）
* `ranges::iota_view(n)`生成无限序列n, n+1, ...（因此不能直接对其迭代）

大多数视图都有一个对应的函数，定义在命名空间`std::ranges::views`中，用于创建这种视图。例如，`ranges::views::iota(m, n)`等价于`ranges::iota_view(m, n)`，`ranges::views::iota(n)`等价于`ranges::iota_view(n)`。

例如：

```cpp
#include <algorithm>
#include <iostream>
#include <ranges>

int main() {
    for (int i : std::ranges::iota_view(1, 10))
        std::cout << i << ' ';
    std::cout << '\n';

    for (int i : std::views::iota(1, 10))
        std::cout << i << ' ';
    std::cout << '\n';

    for (int i : std::views::iota(1) | std::views::take(9))
        std::cout << i << ' ';
    std::cout << '\n';

    auto print = [](int i){ std::cout << i << ' '; };
    std::ranges::for_each(std::views::iota(1, 10), print);
    std::cout << '\n';
    return 0;
}
```

输出如下：

```
1 2 3 4 5 6 7 8 9 
1 2 3 4 5 6 7 8 9 
1 2 3 4 5 6 7 8 9 
1 2 3 4 5 6 7 8 9 
```

范围工厂的完整列表参见[Range factories](https://en.cppreference.com/w/cpp/ranges#Range_factories)。

命名空间别名`std::views`是`std::ranges::views`的简写。下面将其简称为`views`。

### 3.2 范围适配器
**范围适配器**(range adaptor)是对底层范围的包装器，在迭代时进行惰性计算（如过滤、映射等），并且支持通过管道运算符(`|`)将多个操作串联起来（就像UNIX Shell中的管道）。

范围算法是立即执行的，可能会修改范围中的元素；而范围适配器是惰性计算的，会返回一个新视图，不会修改原始数据。

例如，回看本文开头打印偶数平方的例子。表达式

```cpp
arr | views::filter(even) | views::transform(square)
```

返回一个新的视图，该视图是由`arr`中的元素经过谓词`even`过滤、再经过函数`square`转换构成的。另外，范围适配器还支持函数调用语法，上面的表达式等价于

```cpp
views::transform(views::filter(arr, even), square)
```

这种表达式的返回类型是某种视图，具体类型并不重要，只需知道可以直接对其迭代、传递给另一个视图或者传递给范围算法。如果需要将中间结果保存到变量中，可以使用`auto`声明。

对比上述两种形式会发现，范围适配器有两种调用方式：接受一个范围参数和额外参数（如`views::filter(arr, even)`），或者只有额外参数（如`views::filter(even)`）。要弄清底层实现原理，需要了解两个基本概念：范围适配器对象和范围适配器闭包对象。

**范围适配器闭包对象**(range adaptor closure object)是接受一个范围参数、返回一个视图的函数对象（详见具名要求[RangeAdaptorClosureObject](https://en.cppreference.com/w/cpp/named_req/RangeAdaptorClosureObject)和概念[ranges::range_adaptor_closure](https://en.cppreference.com/w/cpp/ranges/range_adaptor_closure)）。范围参数可以通过管道语法(`|`)或函数调用语法(`()`)传递。假设`c`是一个范围适配器闭包对象，`r`是一个范围，则以下两个表达式等价：

```cpp
c(r)
r | c
```

两个范围适配器闭包对象可以用管道运算符`|`连接起来，以产生另一个范围适配器闭包对象。假设`c`和`d`是范围适配器闭包对象，`c | d`产生范围适配器闭包对象`e`，`r`是一个范围，则以下表达式等价：

```cpp
d(c(r))
r | c | d
e(r)   // (c | d)(r)
r | e  // r | (c | d)
```

**范围适配器对象**(range adaptor object)是接受一个范围作为第一个参数（可能还有额外参数）、返回一个视图的函数对象（详见具名要求[RangeAdaptorObject](https://en.cppreference.com/w/cpp/named_req/RangeAdaptorObject)）。
* 如果一个范围适配器对象只接受一个参数，则它也是一个范围适配器闭包对象。
* 如果一个范围适配器对象接受多个参数，则它也支持[部分应用](https://en.wikipedia.org/wiki/Partial_application)(partial application)。假设`a`是一个范围适配器对象，`r`是一个范围，`args...`是额外参数，则表达式`a(args...)`是一个范围适配器闭包对象，以下表达式等价：

```cpp
a(r, args...)
a(args...)(r)
r | a(args...)
```

例如，范围适配器`views::filter`接受一个范围参数和一个谓词参数。在前面的例子中，`views::filter(arr, even)`是一个视图，而`views::filter(even)`是一个范围适配器闭包对象。以下三种形式是等价的：

```cpp
views::filter(arr, even)
views::filter(even)(arr)
arr | views::filter(even)
```

组合视图时甚至可以混合使用不同的形式，例如：

```cpp
views::transform(views::filter(arr, even), square)
arr | views::filter(even) | views::transform(square)
(views::filter(even) | views::transform(square))(arr)
views::transform(arr | views::filter(even), square)
...
```

范围适配器的完整列表参见[Range adaptors](https://en.cppreference.com/w/cpp/ranges#Range_adaptors)。

常用的视图及其等价的Python函数如下表所示。

| 视图 | 等价的Python函数 |
| --- | --- |
| `views::iota(m, n)` | `range(m, n)` |
| `views::iota(n)` | `itertools.count(n)` |
| `views::empty<T>` | |
| `views::single(e)` | |
| `views::repeat(e, n)` | `itertools.repeat(e, n)` |
| `views::repeat(e)` | `itertools.repeat(e)` |
| `views::filter(r, p)` | `filter(p, r)` |
| `views::transform(r, f)` | `map(f, r)` |
| `views::take(r, n)` | `itertools.islice(r, n)` |
| `views::take_while(r, p)` | `itertools.takewhile(p, r)` |
| `views::drop(r, n)` | `itertools.islice(r, n, None)` |
| `views::drop_while(r, p)` | `itertools.dropwhile(p, r)` |
| `views::concat(rs...)` | `itertools.chain(rs...)` |
| `views::join(r)` | `itertools.chain.from_iterable(r)` |
| `views::reverse(r)` | `reversed(r)` |
| `views::keys` | `map(operator.itemgetter(0), r)` |
| `views::values` | `map(operator.itemgetter(1), r)` |
| `views::elements<N>(r)` | `map(operator.itemgetter(N), r)` |
| `views::enumerate(r)` | `enumerate(r)` |
| `views::zip(rs...)` | `zip(rs...)` |
| `views::chunk(r, n)` | `itertools.batched(r, n)` |
| `views::stride(r, n)` | `itertools.islice(r, None, None, n)` |

另外，还有两个表示子范围的视图：
* `ranges::subrange(first, last)` 生成表示范围`[first, last)`的视图
* `views::counted(it, n)` 生成表示范围`[it, it + n)`的视图

### 3.3 基于范围的容器操作
C++23为STL容器增加了基于范围的操作（需要GCC 14+或Clang 17+）。

可以使用`ranges::to<C>`将范围转换为容器`C`。例如：

```cpp
auto vec = std::views::iota(1, 5)
        | std::views::transform([](int i) { return i * 2; })
        | std::ranges::to<std::vector>();
std::println("{}", vec);  // prints "[2, 4, 6, 8]"

auto list = vec | std::views::take(3) | std::ranges::to<std::list<double>>();
std::println("{}", list);  // prints "[2, 4, 6]"
```

容器还增加了基于范围的插入操作。例如，对于向量`v`：
* `v.insert_range(p, r)`将范围`r`中的元素插入`p`之前，等价于`v.insert(p, r.begin(), r.end())`
* `v.append_range(r)`将范围`r`中的元素添加到向量末尾，等价于`v.insert(v.end(), r.begin(), r.end())`

### 3.4 示例
本节给出一些视图的使用示例。

视图的元素类型取决于底层范围和执行的动作。如果直接对数组或容器使用`views::filter`，得到的视图的元素是底层范围元素的引用，因此可以通过视图修改原始数据。例如：

```cpp
int input[] = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
auto even = [](int i) { return i % 2 == 0; };
auto even_nums = input | std::views::filter(even);
std::ranges::fill(even_nums, 42);
for (int i : input)
    std::cout << i << ' ';
// prints "42 1 42 3 42 5 42 7 42 9 42"
```

`views::keys`和`views::values`分别提取元组式值的第一个和第二个元素。元组式(tuple-like)值包括`std::pair`、`std::tuple`、`std::array`等。这两个视图可用于映射或`std::pair`向量等。例如，可以如下从映射中提取键和值：

```cpp
std::map<char, int> m = { {'A', 1}, {'B', 2}, {'C', 3}, {'D', 4}, {'E', 5} };
for (char k : m | std::views::keys | std::views::transform(tolower))
    std::cout << k << ' ';
// prints "a b c d e"
std::cout << '\n';

auto odd = [](int i) { return i % 2 == 1; };
for (int v : m | std::views::values | std::views::filter(odd))
    std::cout << v << ' ';
// prints "1 3 5"
std::cout << '\n';
```

`views::split`使用给定的分隔符将范围分割成子范围。例如，可以如下使用`std::string_view`分割字符串：

```cpp
using std::operator""sv;
auto words = "Hello^_^C++^_^20^_^!"sv;
auto delim = "^_^"sv;
for (const auto word : std::views::split(words, delim))
    // with string_view's C++23 range constructor:
    std::cout << std::quoted(std::string_view(word)) << ' ';
// prints: "Hello" "C++" "20" "!" 
std::cout << '\n';
```

`views::join`打平(flatten)嵌套范围。例如，可以如下拼接字符串和向量：

```cpp
using namespace std::literals;

const auto bits = {"https:"sv, "//"sv, "cppreference"sv, "."sv, "com"sv};
for (char c : bits | std::views::join)
    std::cout << c;
// prints "https://cppreference.com"
std::cout << '\n';

std::vector<std::vector<int>> v = { {1, 2}, {3, 4, 5}, {6}, {7, 8, 9} };
auto jv = v | std::views::join | std::ranges::to<std::vector>();
for (int e : jv)
    std::cout << e << ' ';
// prints "1 2 3 4 5 6 7 8 9"
std::cout << '\n';
```

`views::enumerate`（C++23引入）将每个元素映射为(索引,值)对（类似于Python的`enumerate()`函数）。例如：

```cpp
std::string s = "ABCD";
for (auto [i, c] : std::views::enumerate(s))
    std::cout << '(' << i << ':' << c << ") ";
// prints "(0:A) (1:B) (2:C) (3:D)"
std::cout << '\n';

auto m = s | std::views::enumerate | std::ranges::to<std::map>();
for (auto [k, v] : m)
    std::cout << '[' << k << "]:" << v << ' ';
// prints "[0]:A [1]:B [2]:C [3]:D"
std::cout << '\n';
```

`views::zip`（C++23引入）生成由每个范围对应位置元素的元组构成的视图，到最短的范围结束为止（类似于Python的`zip()`函数）。例如：

```cpp
std::vector a = {1, 2, 3, 4};
std::array b = {'A', 'B', 'C', 'D', 'E', 'F'};
std::list<std::string> c = {"α", "β", "γ", "δ", "ε"};
for (const auto& [x, y, z] : std::views::zip(a, b, c))
    std::cout << x << ' ' << y << ' ' << z << '\n';
```

输出如下：

```
1 A α
2 B β
3 C γ
4 D δ
```

### 3.5 自定义视图
本节通过使用C++范围库实现几个常用的Python函数来介绍如何自定义视图。

#### 3.5.1 实现range
在Python中，函数`range(start, stop, step)`生成从`start`到`stop`（不包含）、步长为`step`的整数序列。例如：

```python
>>> list(range(0, 10))
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>>> list(range(1, 11))
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
>>> list(range(0, 30, 5))
[0, 5, 10, 15, 20, 25]
>>> list(range(0, 10, 3))
[0, 3, 6, 9]
>>> list(range(0, 0))
[]
>>> list(range(1, 0))
[]
>>> list(range(10, -1, -1))
[10, 9, 8, 7, 6, 5, 4, 3, 2, 1, 0]
>>> list(range(12, 2, -3))
[12, 9, 6, 3]
```

C++中的`views::iota`视图具有类似的功能。与Python的`range()`一样，元素都是在迭代时生成的。但是`views::iota`不支持指定步长。另外，当上界小于下界时会导致无限循环，而不是生成空序列。

为了解决这两个问题，可以使用`views::take_while`和`views::stride`（C++23引入）。另外，可以使用`auto`声明来避免编写复杂的返回类型。如下所示：

```cpp
#include <iostream>
#include <ranges>
#include <stdexcept>

auto range(int start, int stop, int step = 1) {
    if (step == 0)
        throw std::invalid_argument("step must not be zero");
    return std::views::iota(start)
        | std::views::take_while([stop](int i) { return i < stop; })
        | std::views::stride(step);
}

void print(auto&& r) {
    std::cout << "[ ";
    for (const auto& e : r)
        std::cout << e << ' ';
    std::cout << "]\n";
}

int main() {
    print(range(0, 10));
    print(range(1, 11));
    print(range(0, 30, 5));
    print(range(0, 10, 3));
    print(range(0, 0));
    print(range(1, 0));
    return 0;
}
```

输出如下：

```
[ 0 1 2 3 4 5 6 7 8 9 ]
[ 1 2 3 4 5 6 7 8 9 10 ]
[ 0 5 10 15 20 25 ]
[ 0 3 6 9 ]
[ ]
[ ]
```

现在已经实现了正数步长。为了实现负数步长，首先尝试使用`if`语句增加一个分支：

```cpp
auto range(int start, int stop, int step = 1) {
    if (step == 0)
        throw std::invalid_argument("step must not be zero");
    if (step > 0) {
        return std::views::iota(start)
            | std::views::take_while([stop](int i) { return i < stop; })
            | std::views::stride(step);
    }
    else {
        return std::views::iota(stop + 1)
            | std::views::take_while([start](int i) { return i <= start; })
            | std::views::reverse
            | std::views::stride(-step);
    }
}
```

然而，这会导致编译失败，得到一堆晦涩难懂的错误消息（详见<https://godbolt.org/z/s3h5qW9PG>）：

```
error: inconsistent deduction for auto return type: 'std::ranges::stride_view<std::ranges::take_while_view<std::ranges::iota_view<int, std::unreachable_sentinel_t>, range(int, int, int)::<lambda(int)> > >' and then 'std::ranges::stride_view<std::ranges::reverse_view<std::ranges::take_while_view<std::ranges::iota_view<int, std::unreachable_sentinel_t>, range(int, int, int)::<lambda(int)> > > >'
```

简单来说就是`if`语句两个分支的返回类型不同（虽然都是`ranges::stride_view`，但模板参数不同，因此编译器认为是不同的类型）。一个函数具有多种返回类型在C++中是不合法的。

为了解决这一困难，考虑换一种思路——使用等差数列通项公式：

$$ r_i = start + i * step \ (0 \le i < length) $$

$$ length = \left \lfloor \frac{\vert stop - start \vert - 1}{\vert step \vert} \right \rfloor + 1 $$

这样可以将步长为正数和负数的两种情况统一起来：

```cpp
int range_length(int start, int stop, int step) {
    int lo, hi;

    if (step > 0) {
        lo = start;
        hi = stop;
    }
    else {
        lo = stop;
        hi = start;
        step = -step;
    }

    if (lo >= hi)
        return 0;
    return (hi - lo - 1) / step + 1;
}

auto range(int start, int stop, int step = 1) {
    if (step == 0)
        throw std::invalid_argument("step must not be zero");
    int length = range_length(start, stop, step);
    return std::views::iota(0, length)
        | std::views::transform([start, step](int i) { return start + i * step; });
}
```

现在可以使用负数步长了：

```cpp
print(range(10, -1, -1));
print(range(12, 2, -3));
```

```
[ 10 9 8 7 6 5 4 3 2 1 0 ]
[ 12 9 6 3 ]
```

注：这里的实现参考了CPython的[rangeobject.c](https://github.com/python/cpython/blob/main/Objects/rangeobject.c)。

#### 3.5.2 视图概念
自定义视图可能比较复杂，难以通过组合标准视图得到。在这种情况下，就需要实现一个视图类。

要使一个类被认为是视图，必须满足概念[ranges::view](https://en.cppreference.com/w/cpp/ranges/view)，其定义如下：

```cpp
template<class T>
concept view = ranges::range<T> && std::movable<T> && ranges::enable_view<T>;

template<class T>
constexpr bool enable_view = std::derived_from<T, view_base> || std::derived_from<T, ranges::view_interface<T>>;
```

也就是说，“视图”就是可移动且派生自[ranges::view_interface](https://en.cppreference.com/w/cpp/ranges/view_interface)的范围。另外，`std::string_view`和`std::span`也被认为是视图。

视图类必须提供成员函数`begin()`和`end()`，分别返回开始和结束迭代器。`ranges::view_interface`类会根据迭代器类别提供其他成员函数，如`size()`、`empty()`、`operator[]`等。迭代器通常实现为视图类的内部类，并在`operator++`和`operator*`中实现核心计算逻辑。

下面是用视图类实现的`range()`。

```cpp
#include <cmath>
#include <iostream>
#include <iterator>
#include <ranges>
#include <stdexcept>

class range : public std::ranges::view_interface<range> {
public:
    class iterator {
    public:
        using iterator_category = std::forward_iterator_tag;
        using difference_type = ptrdiff_t;
        using value_type = int;
        using pointer = const int*;
        using reference = const int&;

        iterator() = default;
        iterator(int value, int step) :value_(value), step_(step) {}

        int operator*() const { return value_; }
        iterator& operator++() { value_ += step_; return *this; }
        iterator operator++(int) { iterator tmp = *this; ++*this; return tmp; }

        bool operator==(const iterator& other) const { return value_ == other.value_; }
        bool operator!=(const iterator& other) const { return !(*this == other); }

    private:
        int value_;
        int step_;
    };

    range(int start, int stop, int step = 1) :start_(start), step_(step) {
        if (step == 0)
            throw std::invalid_argument("step must not be zero");
        stop_ = (step > 0 && start >= stop || step < 0 && start <= stop) ? start
                : start + step * ceil((stop - start) / static_cast<double>(step));
    }

    explicit range(int stop) :range(0, stop, 1) {}

    iterator begin() const { return iterator(start_, step_); }
    iterator end() const { return iterator(stop_, step_); }

private:
    int start_, stop_, step_;
};

void print(auto&& r) {
    std::cout << "[ ";
    for (const auto& e : r)
        std::cout << e << ' ';
    std::cout << "]\n";
}

int main() {
    static_assert(std::forward_iterator<range::iterator>);
    static_assert(std::ranges::forward_range<range>);
    static_assert(std::ranges::view<range>);

    print(range(10));
    print(range(1, 11));
    print(range(0, 30, 5));
    print(range(0, 10, 3));
    print(range(0, 0));
    print(range(1, 0));
    print(range(10, -1, -1));
    print(range(12, 2, -3));
    print(range(1000000, 2000000) | std::views::take(5));
    return 0;
}
```

程序输出如下：

```
[ 0 1 2 3 4 5 6 7 8 9 ]
[ 1 2 3 4 5 6 7 8 9 10 ]
[ 0 5 10 15 20 25 ]
[ 0 3 6 9 ]
[ ]
[ ]
[ 10 9 8 7 6 5 4 3 2 1 0 ]
[ 12 9 6 3 ]
[ 1000000 1000001 1000002 1000003 1000004 ]
```

从最后一个打印语句可以看到，自定义视图`range`可以和标准库视图一样通过管道运算符组合。

#### 3.5.3 实现itertools.accumulate
C++20的范围库并没有提供数值算法（如`std::accumulate()`），下面用视图类实现Python的`itertools.accumulate()`函数。

与C++的`accumulate()`算法不同，Python的`accumulate()`函数生成一个累加序列，而不是只返回最终结果。另外，还可以提供一个函数以自定义“累加”操作。例如：

```python
>>> from itertools import accumulate
>>> import operator
>>> data = [3, 4, 6, 2, 1, 9, 0, 7, 5, 8]
>>> list(accumulate(data))  # prefix sum
[3, 7, 13, 15, 16, 25, 25, 32, 37, 45]
>>> list(accumulate(data, operator.mul))  # prefix product
[3, 12, 72, 144, 144, 1296, 0, 0, 0, 0]
>>> list(accumulate(data, max))  # prefix max
[3, 4, 6, 6, 6, 9, 9, 9, 9, 9]
```

下面是用视图类实现的`accumulate()`。

```cpp
#include <iostream>
#include <iterator>
#include <functional>
#include <ranges>
#include <vector>

template<class Range, class BinaryOperation = std::plus<>, class T = typename Range::value_type>
class accumulate : public std::ranges::view_interface<accumulate<Range, BinaryOperation, T>> {
public:
    class iterator {
    public:
        using iterator_category = std::forward_iterator_tag;
        using difference_type = ptrdiff_t;
        using value_type = T;
        using pointer = const T*;
        using reference = const T&;

        iterator() = default;
        iterator(std::ranges::iterator_t<Range> current, accumulate* parent)
                :current_(current), parent_(parent), accumulated_(parent->init_) {
            acc();
        }

        reference operator*() const { return accumulated_; }

        iterator& operator++() {
            ++current_;
            acc();
            return *this;
        }

        iterator operator++(int) {
            iterator tmp = *this;
            ++*this;
            return tmp;
        }

        bool operator==(const iterator& other) const { return current_ == other.current_; }
        bool operator!=(const iterator& other) const { return !(*this == other); }

    private:
        void acc() {
            if (current_ != std::ranges::end(parent_->base_))
                accumulated_ = std::invoke(parent_->op_, accumulated_, *current_);
        }

        std::ranges::iterator_t<Range> current_;
        accumulate* parent_;
        T accumulated_;
    };

    explicit accumulate(Range base, BinaryOperation op = {}, T init = {})
        :base_(std::move(base)), op_(std::move(op)), init_(std::move(init)) {}

    iterator begin() { return iterator(std::ranges::begin(base_), this); }
    iterator end() { return iterator(std::ranges::end(base_), this); }

private:
    Range base_;
    BinaryOperation op_;
    T init_;
};

void print(auto&& r) {
    std::cout << "[ ";
    for (const auto& e : r)
        std::cout << e << ' ';
    std::cout << "]\n";
}

int main() {
    static_assert(std::forward_iterator<accumulate<std::vector<int>>::iterator>);
    static_assert(std::ranges::forward_range<accumulate<std::vector<int>>>);
    static_assert(std::ranges::view<accumulate<std::vector<int>>>);

    std::vector<int> data = {3, 4, 6, 2, 1, 9, 0, 7, 5, 8};
    print(accumulate(data));
    print(accumulate(data, std::multiplies<>(), 1));
    auto max = [](int a, int b) { return a > b ? a : b; };
    print(accumulate(data, max, -1));
    return 0;
}
```

程序输出如下：

```
[ 3 7 13 15 16 25 25 32 37 45 ]
[ 3 12 72 144 144 1296 0 0 0 0 ]
[ 3 4 6 6 6 9 9 9 9 9 ]
```

可以看出，用C++实现自定义视图类时，不得不编写大量的样板代码（如迭代器的5个成员类型、后缀`++`、`==`和`!=`等）。作为对比，下面是用Python实现的`accumulate()`。利用`yield`语句，代码要简短得多。

```python
def accumulate(iterable, function, initial):
    total = initial
    yield total
    for element in iterable:
        total = function(total, element)
        yield total
```

## 参考
* [Ranges library - cppreference](https://en.cppreference.com/w/cpp/ranges)
* [Constrained algorithms - cppreference](https://en.cppreference.com/w/cpp/algorithm/ranges)
* [\<ranges\> - Microsoft Learn](https://learn.microsoft.com/en-us/cpp/standard-library/ranges)
* [Modernes C++](https://www.modernescpp.com/index.php/table-of-content/)
* [C++ 20 新特性 ranges 精讲](https://blog.csdn.net/qq_42896106/article/details/128737878)
