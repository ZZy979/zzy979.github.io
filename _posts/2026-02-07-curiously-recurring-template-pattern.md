---
title: 【C++】奇异递归模板模式(CRTP)
date: 2026-02-07 23:22:29 +0800
categories: [C++]
tags: [cpp, crtp, template, template metaprogramming, deducing this]
---
## 1.引言
**奇异递归模板模式**(Curiously Recurring Template Pattern, CRTP)是C++模板编程的一种习惯用法，即**类`X`继承自类模板`Y`，并以`X`自身为模板参数**。例如：

```cpp
template<class T>
class Y {};
 
class X : public Y<X> {};
```

这种技术最早于1989年作为“[F-界量化](https://www.cs.utexas.edu/~wcook/papers/FBound89/CookFBound89.pdf)”(F-bounded quantification)被提出，Jim Coplien于1995年称之为 "CRTP" 。

## 2.用例
CRTP的用途主要有两类：实现编译期多态以及向派生类添加功能。

### 2.1 编译期多态
C++中的多态是基于虚函数表实现的，需要在运行时将虚函数调用分派到实际的派生类函数。CRTP可用于实现“编译期多态”（也叫“静态多态”）。考虑下面的例子：

```cpp
#include <iostream>

template<class T>
class Vehicle {
public:
    int num_wheels() { return static_cast<T*>(this)->num_wheels_impl(); }

protected:
    Vehicle() = default;  // creation of Vehicle objects is UB
};

class Bicycle : public Vehicle<Bicycle> {
public:
    int num_wheels_impl() { return 2; }
};

class Car : public Vehicle<Car> {
public:
    int num_wheels_impl() { return 4; }
};

int main() {
    Bicycle b;
    std::cout << "Bicycles have " << b.num_wheels() << " wheels\n";
    Car c;
    std::cout << "Cars have " << c.num_wheels() << " wheels\n";
    return 0;
}
```

输出结果如下：

```
Bicycles have 2 wheels
Cars have 4 wheels
```

在基类函数`Vehicle::num_wheels()`中，通过`static_cast`将`this`转换为模板参数`T`（同时也是派生类）的指针，并调用派生类函数`num_wheels_impl()`。这样**实现了类似多态的效果，同时避免了虚函数调用的开销**，因为所有调用都在编译期确定（性能对比参见[The cost of dynamic (virtual calls) vs. static (CRTP) dispatch in C++](https://eli.thegreenplace.net/2013/12/05/the-cost-of-dynamic-virtual-calls-vs-static-crtp-dispatch-in-c)）。这种模式广泛用于Windows的ATL和WTL等库中。

注：
* 这段代码之所以能通过编译，是因为类模板的成员函数体**只有在实际使用时才会被实例化**。当编译器遇到`Bicycle`类的定义时，会实例化`Vehicle<Bicycle>`，但并不会立即实例化`num_wheels()`的函数体，因为此时`Bicycle`还不是完整类型，无法获得`num_wheels_impl()`函数的地址。而在调用`b.num_wheels()`时，`Bicycle`已经是完整类型，`Vehicle<Bicycle>::num_wheels()`函数体被实例化为`return static_cast<Bicycle*>(this)->num_wheels_impl()`，即调用`Bicycle::num_wheels_impl()`，此时该函数的定义是已知的。
* Java中也有类似的用法，例如

```java
public class Item implements Comparable<Item> {
    private String name;

    @Override
    public int compareTo(Item other) {
        return name.compareTo(other.name);
    }
}
```

但由于Java对泛型类的处理方式与C++模板不同，因此无法实现编译期多态。

#### deducing this
使用C++23的[deducing this](https://en.cppreference.com/w/cpp/language/member_functions.html#Explicit_object_member_functions)特性可以简化CRTP的写法。

如果基类函数`num_wheels()`使用了[显式对象参数](https://en.cppreference.com/w/cpp/language/function.html#Explicit_object_parameter)(explicit object parameter)，基类`Vehicle`就不必是模板，因为`self`参数可以自动推导为正确的派生类型，而无需使用`static_cast`。

注：GCC 14以上版本才支持该特性。

```cpp
class Vehicle {
public:
    template<class Self>
    int num_wheels(this Self&& self) { return self.num_wheels_impl(); }
    // or
    // int num_wheels(this auto&& self) {...}

protected:
    Vehicle() = default;  // creation of Vehicle objects is UB
};

class Bicycle : public Vehicle {
public:
    int num_wheels_impl() { return 2; }
};

class Car : public Vehicle {
public:
    int num_wheels_impl() { return 4; }
};
```

<https://godbolt.org/z/vaYhaKra9>

### 2.2 添加功能
CRTP的另一个用途是向派生类添加功能。在上一节的例子中可以看到，基类可以利用模板参数和对`this`类型转换来访问派生类的函数。利用这一点，基类可以提供一些通用功能，可以被多个派生类复用（注：这类似于Python的混入类）。

在下面的例子中，利用CRTP向派生类添加数值计算函数（来自[Jonathan Boccara](https://www.fluentcpp.com/2017/05/16/what-the-crtp-brings-to-code/)）。假设`Sensitivity`有一个值：

```cpp
class Sensitivity {
public:
    explicit Sensitivity(double v) :value_(v) {}

    double value() const { return value_; }
    void set_value(double value) { value_ = value; }
    // rest of the sensitivity's rich interface...

private:
    double value_;
};
```

现在希望添加一些数值计算函数，例如缩放（乘以一个常数）、平方、相反数等。为了让其他类也能复用这些操作，将这三个函数放在一个基类`NumericalFunctions`中：

```cpp
template<class T>
class NumericalFunctions {
public:
    void scale(double multiplicator) {
        T& self = static_cast<T&>(*this);
        self.set_value(self.value() * multiplicator);
    }

    void square() {
        T& self = static_cast<T&>(*this);
        self.set_value(self.value() * self.value());
    }

    void opposite() {
        scale(-1.0);
    }
};

class Sensitivity : public NumericalFunctions<Sensitivity> {
    // ...
};
```

`NumericalFunctions`使用了派生类的`value()`和`set_value()`完成数值计算。任何定义了这两个函数的类`X`都可以通过继承`NumericalFunctions<X>`获得这些数值计算函数。如果不使用CRTP，则需要将`value()`和`set_value()`放在基类中，并声明为纯虚函数，由派生类覆盖这两个函数。

在这个CRTP例子中，“继承”的含义与其他情况不同。通常，
* 派生类“是一个”基类。
* 基类提供接口，派生类提供实现。
* 在泛型代码中通过基类调用虚函数，由多态动态分派到派生类函数。

对于CRTP的这种用法，情况则完全不同：
* 派生类不“是一个”基类，而是通过继承基类来**扩展接口**，以添加更多功能。
* 派生类向基类提供接口，基类使用派生类函数（如`value()`和`set_value()`）。
* 直接使用派生类，而永远不会使用基类。

因此，CRTP有时也称为倒置继承(upside-down inheritance)。

### 2.3 对象计数器
下面是另一个添加功能的例子。类模板`Counter`可以统计一个类的对象创建和销毁的数据，这可以很容易地使用CRTP实现。

```cpp
#include <iostream>
#include <vector>

template<class T>
class Counter {
public:
    static int objects_created;
    static int objects_alive;

    Counter() {
        ++objects_created;
        ++objects_alive;
    }

    Counter(const Counter&) {
        ++objects_created;
        ++objects_alive;
    }

protected:
    ~Counter() {  // objects should never be removed through pointers of this type
        --objects_alive;
    }
};

template<class T> int Counter<T>::objects_created = 0;
template<class T> int Counter<T>::objects_alive = 0;

class X : public Counter<X> {};
class Y : public Counter<Y> {};

int main() {
    X xs[10];
    std::vector<Y> ys(20);
    ys.resize(15);
    std::cout << "X: " << X::objects_created << " objects created, " << X::objects_alive << " alive\n";
    std::cout << "Y: " << Y::objects_created << " objects created, " << Y::objects_alive << " alive\n";
    return 0;
}
```

输出结果如下：

```
X: 10 objects created, 10 alive
Y: 20 objects created, 15 alive
```

每次创建类`X`的对象时，`Counter<X>`的构造函数会将创建计数和存活计数各加1；每次销毁类`X`的对象时，`Counter<X>`的析构函数会将存活计数减1。`Counter<X>`和`Counter<Y>`是两个独立的类，因此能分别对`X`和`Y`进行计数。在这个例子中，这种类的区别是模板参数`T`的唯一用途，这也是不能使用非模板基类的原因。

### 2.4 std::enable_shared_from_this
标准库中使用CRTP的一个例子是头文件\<memory\>中的类模板`std::enable_shared_from_this`。继承`std::enable_shared_from_this<T>`会为类`T`提供成员函数`shared_from_this()`，该函数返回一个新的`std::shared_ptr<T>`，能够正确地与已有的管理当前对象的智能指针共享所有权。

智能指针`std::shared_ptr`能够共享一个对象的所有权，并在引用计数降为0时自动销毁对象。这一机制能正确工作的前提是管理同一个对象的所有`std::shared_ptr`必须**共享引用计数**，否则会导致对象被多次销毁（未定义行为，可能导致程序崩溃）。

假设一个类`T`的对象当前已经由一个名为`pt`的`std::shared_ptr`管理。为了获得与`pt`共享所有权的其他`std::shared_ptr`，应该直接拷贝`pt`：

```cpp
std::shared_ptr<T> pt(new T);
auto pt2 = pt;
```

但是，在类`T`内部无法访问`pt`。要在成员函数中创建一个管理当前对象的`std::shared_ptr`，直接返回`std::shared_ptr<T>(this)`是错误的。例如：

```cpp
#include <iostream>
#include <memory>

class Bad {
public:
    ~Bad() { std::cout << "Bad::~Bad()\n"; }

    std::shared_ptr<Bad> getptr() {
        return std::shared_ptr<Bad>(this);
    }
};

struct X {
    std::shared_ptr<Bad> p;
    explicit X(Bad* b) :p(b->getptr()) {}
};

void test_bad() {
    // Bad, each shared_ptr thinks it is the only owner of the object
    Bad* b = new Bad;
    std::shared_ptr<Bad> p(b);
    X x(b);
    std::cout << "p.use_count() = " << p.use_count() << '\n';
    std::cout << "x.p.use_count() = " << x.p.use_count() << '\n';
} // UB: double-delete of Bad

int main() {
    test_bad();
    return 0;
}
```

GCC编译器得到的结果如下：

<https://godbolt.org/z/fhcPa33Td>

```
p.use_count() = 1
x.p.use_count() = 1
Bad::~Bad()
Bad::~Bad()
free(): double free detected in tcache 2
Program terminated with signal: SIGSEGV
```

这里的问题在于`p`和`x.p`管理了同一个`Bad`对象，但它们的引用计数是独立的，在函数退出时会各调用一次`Bad`的析构函数，导致重复销毁（如果`Bad`的析构函数包含`delete`语句就会引发coredump）。

正确的做法是继承`std::enable_shared_from_this`（使用CRTP），并调用`shared_from_this()`来创建`std::shared_ptr`。需要注意的是，只能对已经由`std::shared_ptr`管理的对象调用该函数，否则会抛出异常`std::bad_weak_ptr`。

```cpp
#include <iostream>
#include <memory>

class Good : public std::enable_shared_from_this<Good> {
public:
    std::shared_ptr<Good> getptr() {
        return shared_from_this();
    }
};

struct X {
    std::shared_ptr<Good> p;
    explicit X(Good* g) :p(g->getptr()) {}
};

void test_good() {
    // Good: the two shared_ptr's share the same object
    Good* g = new Good;
    std::shared_ptr<Good> p(g);
    X x(g);
    std::cout << "p.use_count() = " << p.use_count() << '\n';
    std::cout << "x.p.use_count() = " << x.p.use_count() << '\n';
}

void misuse_good() {
    // Bad: shared_from_this is called without having std::shared_ptr owning the caller
    try {
        Good g;
        std::shared_ptr<Good> p = g.getptr();
    }
    catch (std::bad_weak_ptr& e) {
        std::cout << e.what() << '\n';
    }
}

int main() {
    test_good();
    misuse_good();
    return 0;
}
```

输出结果如下：

<https://godbolt.org/z/4hTPWv4vT>

```
p.use_count() = 2
x.p.use_count() = 2
bad_weak_ptr
```

* 在`test_good()`中，`x.p`是通过`Good::shared_from_this()`得到的，与已有的`p`共享引用计数，因此是正确的。
* 在`misuse_good()`中，对象`g`是在栈上创建的，调用`shared_from_this()`时还没有任何`std::shared_ptr`管理该对象，因此会抛出`std::bad_weak_ptr`。

#### 实现原理
类模板`std::enable_shared_from_this<T>`有一个`std::weak_ptr<T>`类型的成员`weak_this`，用于记录管理当前对象的智能指针（`std::weak_ptr`类似于`std::shared_ptr`，但并不拥有对象的所有权，即不影响引用计数，如果其指向的对象被销毁，该指针就会“过期”）。

```cpp
template<class T>
class enable_shared_from_this {
protected:
    constexpr enable_shared_from_this() {}
    constexpr enable_shared_from_this(const enable_shared_from_this& other) {}
    ~enable_shared_from_this() {}
    enable_shared_from_this& operator=(const enable_shared_from_this& rhs) { return *this; }

public:
    shared_ptr<T> shared_from_this() { return shared_ptr<T>(weak_this); }
    shared_ptr<const T> shared_from_this() const { return shared_ptr<const T>(weak_this); }

    friend class shared_ptr<T>;

private:
    mutable weak_ptr<T> weak_this;
};
```

可以看到，`std::enable_shared_from_this`类本身非常简单：
* 在构造函数中，将`weak_this`默认初始化为空指针。
* 在首次创建管理当前对象的`std::shared_ptr`时，将其赋给该对象的`weak_this`成员。
* 函数`shared_from_this()`直接返回由`weak_this`构造的`std::shared_ptr`。如果`weak_this`仍是空指针，则抛出异常`std::bad_weak_ptr`。

主要的难点在于`std::shared_ptr`的构造函数需要判断被管理的对象是否继承了`std::enable_shared_from_this`以及它的`weak_this`成员是否为空指针，如果是则使用自身对其赋值。一种简化的实现如下：

```cpp
template<class T>
class shared_ptr {
public:
    explicit shared_ptr(T* ptr) {
        // ...
        if constexpr (std::is_base_of_v<enable_shared_from_this<T>, T>) {
            if (ptr && ptr->weak_this.expired())
                ptr->weak_this = *this;
        }
    }

    // ...
};
```

注意，实际的标准库实现可能使用指针转换（即`T*`能否转换为`std::enable_shared_from_this<T>*`）来判断继承关系，这种方式只能识别`public`继承（见[《C++程序设计原理与实践》笔记 第14章]({% post_url 2023-03-06-ppp-note-ch14-graphics-class-design %}) 14.3.4节）。因此在继承`std::enable_shared_from_this`时必须使用`public`继承，否则可能会导致`weak_this`无法被正确赋值。

主流编译器的标准库源码：
* gcc: [libstdc++/shared_ptr_base.h](https://github.com/gcc-mirror/gcc/blob/master/libstdc%2B%2B-v3/include/bits/shared_ptr_base.h) `std::__shared_ptr::_M_enable_shared_from_this_with()`
* clang: [libc++/shared_ptr.h](https://github.com/llvm/llvm-project/blob/main/libcxx/include/__memory/shared_ptr.h) `std::shared_ptr::__enable_weak_this()`
* MSVC: [STL/memory](https://github.com/microsoft/STL/blob/main/stl/inc/memory) `std::shared_ptr::Set_ptr_rep_and_enable_shared()`

## 3.陷阱
在使用CRTP时可能会遇到一些陷阱。

1.继承CRTP基类时模板参数错误，可能导致未定义行为。

```cpp
template<class T> class Base { ... };
class Derived1 : public Base<Derived1> { ... };
class Derived2 : public Base<Derived1> { ... };  // wrong template argument
```

为了避免这个问题，可以在基类中添加一个私有构造函数，并将模板参数`T`声明为友元：

```cpp
template<class T>
class Base {
    // ...
private:
    Base() {}
    friend T;
};
```

派生类的构造函数一定会调用基类的构造函数。`Base`的构造函数是私有的，只有友元类可以访问，而唯一的友元类是模板参数`T`。如果派生类和模板参数不同，代码将编译失败。

2.由于CRTP基类的函数不是虚函数，派生类中的函数会**隐藏**基类中的同名函数。

```cpp
template<class T>
class Base {
public:
    void do_something();
};

class Derived : public Base<Derived> {
public:
    void do_something();  // this hides Base::do_something() !
};
```

## 参考
* <https://en.cppreference.com/w/cpp/language/crtp.html>
* <https://en.wikipedia.org/wiki/Curiously_recurring_template_pattern>
* <https://www.fluentcpp.com/2017/05/12/curiously-recurring-template-pattern/>
* <https://www.sandordargo.com/blog/2019/03/13/the-curiously-recurring-templatep-pattern-CRTP>
* <https://www.cnblogs.com/yang-wen/p/8573269.html>
* <https://www.cnblogs.com/RioTian/p/17881026.html>
