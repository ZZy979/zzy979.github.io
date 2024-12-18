---
title: C++20 Ranges库
date: 2024-12-15 10:32:43 +0800
categories: [C/C++]
tags: [cpp, ranges library, functional programming]
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

本文将介绍范围、视图等基本概念，以及范围算法相比于传统STL算法的优势。

注：GCC编译器支持范围库的最低版本是10，编译时需要添加选项`-std=c++20`。参见[C++ compiler support](https://en.cppreference.com/w/cpp/compiler_support)。

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
头文件\<ranges\>中的概念`ranges::range`给出了“范围”的准确定义：

```cpp
template<class T>
concept range = requires(T& t) {
    ranges::begin(t);
    ranges::end(t);
};
```

简单来说，范围就是可以传递给`ranges::begin()`和`ranges::end()`的类型。这两个函数的定义如下：
* 如果`t`是数组类型`T[N]`，则分别返回`t + 0`和`t + N`。
* 否则，返回`t.begin()`和`t.end()`，如果`T`定义了该成员函数，且返回类型是输入或输出迭代器（`std::input_or_output_iterator`，即支持`*`、`++`、`==`和`!=`操作）。例如STL容器。
* 否则，分别返回`begin(t)`和`end(t)`，如果该表达式合法且返回类型是输入或输出迭代器。
* 否则，该函数调用是非法的。

注：
* 范围的`end()`代器（也叫做**哨兵**(sentinel)）类型不必和`begin()`相同。例如，可以用`'\0'`结束一个C风格字符串，空字符串结束一个单词列表，空指针结束一个链表，或者-1结束一个非负数列表。
* 在C++20之前，标准库还有一对类似的函数`std::begin()`和`std::end()`，定义在各容器对应的头文件中。二者的区别详见[What is the difference between std::ranges::begin and std::begin?](https://stackoverflow.com/questions/62183286/what-is-the-difference-between-stdrangesbegin-and-stdbegin)
* 概念`concept`也是C++20的新特性，详见[《C++20之概念》]({% post_url 2024-03-24-cpp20-concept %})。

可以使用概念`ranges::range`检查一个类型是否是范围。例如：

```cpp
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

static_assert(std::ranges::range<int[10]>);
static_assert(std::ranges::range<std::vector<int>>);
static_assert(std::ranges::range<std::string>);
static_assert(std::ranges::range<std::map<int, int>>);
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

第一种形式等价于`std::sort(first, last, comp)`，但额外提供了`proj`参数，用于对元素进行映射操作（即对映射后的结果而不是元素本身进行排序，相当于Python中`sort()`函数的`key`参数），默认为恒等映射`std::identity`。第二种形式等价于`ranges::sort(ranges::begin(r), ranges::end(r), comp, proj)`。

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

传统STL算法的谓词参数可以是普通函数、Lambda表达式或函数对象，但不能是成员指针（除非使用`std::mem_fn`包装）。这是由于成员指针的调用语法与其他函数对象不同（必须使用`.*`或`->*`而不是`()`调用）。而范围算法支持所有情况。

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

    User users[] = {{"Alice", 17}, {"Bob", 25}, {"Cindy", 19}, {"Dale", 20}, {"Eric", 22}};
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
    std::vector<pair> pairs = {{1, "one"}, {2, "two"}, {3, "three"}};

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

### 2.3 容器操作
C++23为STL容器增加了基于范围的构造和插入操作，可以将范围作为单一参数传递。

ranges::to
构造函数参数：from_range

例如，对于向量`v`：
* `v.insert_range(p, r)`将范围`r`中的元素插入`p`之前，等价于`v.insert(p, r.begin(), r.end())`
* `v.append_range(r)`将范围`r`中的元素添加到向量末尾，等价于`v.insert(v.end(), r.begin(), r.end())`

## 3.视图
**视图**(view)是间接表示范围的轻量级对象。标准库提供的视图定义在头文件\<ranges\>中，主要包括两大类：
* 范围工厂：在迭代时生成元素，如`ranges::iota_view`。
* 范围适配器：对底层范围的包装器，在迭代时进行惰性计算，如`ranges::filter_view`。

视图是一种特殊的范围，因此也可用于foreach循环或者传递给范围算法。STL容器是范围，但不是视图。

注：C++中的范围和视图可分别类比为Python中的可迭代对象(iterable)和生成器(generator)。

### 3.1 范围工厂
**范围工厂**(range factory)是在迭代时生成元素的视图。

例如，视图`ranges::iota_view`用于生成连续的整数序列：
* `ranges::iota_view(m, n)`生成有限序列m, m+1, ..., n-1
* `ranges::iota_view(n)`生成无限序列n, n+1, ...（因此不能直接对其迭代）

大多数视图都有一个对应的函数，用于创建这种视图。例如，`ranges::views::iota(m, n)`和`ranges::views::iota(n)`分别等价于`ranges::iota_view(m, n)`和`ranges::iota_view(n)`。

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

标准库为命名空间`std::ranges::views`提供了一个别名`std::views`。下面将其简称为`views`。

### 3.2 范围适配器
**范围适配器**(range adaptor)是对底层范围的包装器，在迭代时进行惰性计算（如过滤、映射等），并且支持通过管道运算符(`|`)将多个操作串联起来。

例如，回看本文开头打印偶数平方的例子，表达式

```cpp
arr | views::filter(even) | views::transform(square)
```

返回一个新的视图，该视图是由`arr`中的元素经过谓词`even`过滤、再经过函数`square`转换构成的。另外，范围适配器还支持函数调用语法。因此，上面的表达式等价于

```cpp
views::transform(views::filter(arr, even), square)
```

这种表达式的返回类型是某种视图，具体类型并不重要，只需知道可以直接对其迭代、传递给另一个视图或者传递给范围算法。如果需要将中间结果保存到变量中，可以使用`auto`声明。

范围算法是立即执行的，可能会修改范围中的元素；范围适配器是惰性计算的，会返回一个新视图，不会修改原始数据。

#### 范围适配器对象和范围适配器闭包对象
对比上述两种形式会发现，范围适配器有两种调用方式：接受一个范围参数和额外参数（如`views::filter(arr, even)`），或者只有额外参数（如`views::filter(even)`）。

要弄清底层实现原理，需要了解两个基本概念：范围适配器对象和范围适配器闭包对象。

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

组合视图时甚至可以混合使用不同的形式：

```cpp
views::transform(views::filter(arr, even), square)
arr | views::filter(even) | views::transform(square)
(views::filter(even) | views::transform(square))(arr)
views::transform(arr | views::filter(even), square)
...
```

常用的视图及其等价的Python函数如下表所示。范围适配器的完整列表参见[Range adaptors](https://en.cppreference.com/w/cpp/ranges#Range_adaptors)。

| 视图 | 等价的Python函数 |
| --- | --- |
| `views::iota(m, n)` | `range(m, n)` |
| `views::iota(n)` | `itertools.count(n)` |
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
| `views::elements<N>(r)` | `map(operator.itemgetter(N), r)` |
| `views::keys` | `map(operator.itemgetter(0), r)` |
| `views::values` | `map(operator.itemgetter(1), r)` |
| `views::enumerate(r)` | `enumerate(r)` |
| `views::zip(rs...)` | `zip(rs...)` |
| `views::chunk(r, n)` | `itertools.batched(r, n)` |
| `views::stride(r, n)` | `itertools.islice(r, None, None, n)` |

### 3.3 示例

不能用于临时对象
auto firstThree = {1, 2, 3, 4, 5} | std::views::drop(3); 错

打印map的keys和values

示例：拼接向量

```cpp
long long maxSpending(vector<vector<int>>& values) {
    auto vec = values | views::join | ranges::to<vector>();
    ranges::sort(vec);
    return *ranges::fold_left_first(vec, [i = 2zu](auto a, int b)mutable { return a + b * i++; });
}
```

Python的iterable，range、map等内置函数、itertools模块

### string_view和span
### 自定义视图
视图概念
模拟range
模拟itertools.accumulate


## 参考
* [Ranges library - cppreference](https://en.cppreference.com/w/cpp/ranges)
* [Constrained algorithms - cppreference](https://en.cppreference.com/w/cpp/algorithm/ranges)
* [\<ranges\> | Microsoft Learn](https://learn.microsoft.com/en-us/cpp/standard-library/ranges)
* [Modernes C++](https://www.modernescpp.com/index.php/table-of-content/)
* [C++ 20 新特性 ranges 精讲](https://blog.csdn.net/qq_42896106/article/details/128737878)
