---
title: 【C++】动态内存管理
date: 2026-07-28 10:54 +0800
categories: [C/C++]
tags: [cpp, dynamic memory allocation, placement new, allocator]
---
C++将内存管理的控制权完全交给了程序员。这种设计赋予了C++卓越的性能优势，同时也带来了内存泄漏、悬空指针等潜在风险。

本文将系统性地介绍C++中的动态内存分配方式，从传统的C函数到现代C++特性，对比各种方式的特点，并总结常见陷阱和最佳实践。

## 1.malloc和free
C++继承了C语言的内存分配函数`malloc()`和`free()`，定义在头文件`<cstdlib>`中：

```cpp
void* malloc(std::size_t size);
void free(void* ptr);
```

* `malloc(n)`分配n个字节的**未初始化**内存，返回指向首字节的指针，如果分配失败则返回空指针。
* `free(p)`释放由`malloc()`分配的内存空间，如果`p`不是由`malloc()`返回的指针则行为未定义。

下面的示例程序分配了10个`int`和一个`std::vector<int>`的内存。注意，基本类型可以直接向未初始化的内存赋值，而类类型不可以，必须手动构造/销毁对象。其中，第18行的语法叫做placement new（将在2.3节介绍），会调用对象的构造函数；第22行是调用析构函数。

```cpp
#include <cstdlib>
#include <iostream>
#include <vector>

int main() {
    // allocate memory of 10 ints
    if (auto* p = (int*) malloc(10 * sizeof(int))) {
        for (int i = 0; i < 10; i++)
            p[i] = i;
        for (int i = 0; i < 10; i++)
            std::cout << p[i] << ' ';
        std::cout << '\n';
        free(p);  // free memory
    }

    // allocate memory of a vector<int>
    if (auto* pv = (std::vector<int>*) malloc(sizeof(std::vector<int>))) {
        new(pv) std::vector<int>{1, 2, 3, 4, 5};  // construct object
        for (int x : *pv)
            std::cout << x << ' ';
        std::cout << '\n';
        pv->~vector<int>();  // destroy object
        free(pv);  // free memory
    }
    return 0;
}
```

程序输出如下：

```
0 1 2 3 4 5 6 7 8 9 
1 2 3 4 5 
```

`malloc`/`free`的特点：
* 位于C标准库中，简单直接。
* 需要手动计算内存大小，容易出错。
* 返回`void*`，需要强制类型转换。
* 只分配/释放原始内存，不调用构造函数/析构函数，必须手动构造/销毁对象。

虽然`malloc`/`free`在C++中依然可用，但通常仅在特殊场景使用（例如与C语言库交互、实现自定义内存池）。它们无法处理对象的构造和析构，因此在面向对象编程中基本不使用。

## 2.new和delete表达式
`new`表达式和`delete`表达式是C++中最常用的动态内存分配方式。它们不仅分配和释放内存，还会自动调用构造函数和析构函数。

### 2.1 基本用法
[new表达式](https://en.cppreference.com/cpp/language/new)用于在动态内存上创建并初始化对象：
* 表达式`new T(args...)`或`new T{args...}`分配一个`T`类型对象的内存并初始化，返回指向该对象的指针。如果省略了括号和参数则为[默认初始化](https://en.cppreference.com/cpp/language/default_initialization)。
* 表达式`new T[n]{...}`分配包含n个`T`类型对象的数组的内存并初始化所有元素，返回指向首元素的指针。数组大小`n`不必是常量，如果指定了初始值列表则可以省略。如果省略了初始值列表则所有元素均为默认初始化。

[delete表达式](https://en.cppreference.com/cpp/language/delete)用于销毁由`new`表达式创建的对象并释放内存：
* 表达式`delete p`调用`p`指向对象的析构函数，并释放内存。
* 表达式`delete[] p`调用`p`指向数组中每个元素的析构函数，并释放内存。

如果`p`是空指针则什么都不做。如果`p`不是由`new`表达式返回的指针，或者数组/非数组形式混用，则行为未定义。

注：调用`delete[]`时并没有指定元素个数，那么`delete[]`表达式如何知道要释放的数组大小？一种实现方式是`new[]`在返回的内存地址之前额外分配几个字节，用于保存数组大小，这样`delete[]`根据提供的指针向前查看几个字节即可。参见 <https://en.cppreference.com/cpp/language/new#Allocation> ：

> Array allocation may supply unspecified overhead, which may vary from one call to `new` to the next, unless the allocation function selected is the standard non-allocating form. The pointer returned by the `new` expression will be offset by that value from the pointer returned by the allocation function. Many implementations use the array overhead to store the number of objects in the array which is used by the `delete[]` expression to call the correct number of destructors.

例如：

```cpp
auto* pv = new std::vector<int>(5, 42);  // allocate and construct a vector<int>
for (int x : *pv)
    std::cout << x << ' ';
std::cout << '\n';
delete pv;  // destroy and deallocate

auto* pa = new int[]{1, 2, 3, 4, 5};  // allocate and construct an array of 5 ints
for (int i = 0; i < 5; ++i)
    std::cout << pa[i] << ' ';
std::cout << '\n';
delete[] pa;  // destroy and deallocate
```

由`new`表达式创建的对象会一直存在，直到被`delete`表达式释放。如果原始指针丢失了，对象将无法被释放，就会发生**内存泄露**。例如：

```cpp
int* p = new int(7);
p = new int(8); // memory leak

void f() {
    int* p = new int(7);
} // memory leak

void f2() {
    int* p = new int(7);
    g();      // may throw
    delete p; // okay if no exception
} // memory leak if g() throws
```

为了简化动态分配对象的管理，通常将`new`表达式的结果存储在智能指针（`std::unique_ptr`或`std::shared_ptr`）中。智能指针会保证在上述情况下正确释放内存。

### 2.2 不抛出异常版本
当`new`分配内存失败时，会抛出`std::bad_alloc`异常（因此无需判断返回的指针是否为空）。

可以使用不抛出异常的版本避免异常，当分配失败时会返回空指针：

```cpp
#include <new>

std::size_t n = UINT64_MAX;
if (auto* p = new(std::nothrow) int[n]) {
    delete[] p;
} else {
    std::cout << "allocation failure\n";
}
```

### 2.3 定位new
[定位new](https://en.cppreference.com/cpp/language/new#Placement_new) (placement new)是`new`表达式的一种特殊形式，用于**在已分配的内存上构造对象**。

表达式`new(p) T(args...)`不分配内存，在指针`p`指向的未初始化内存上“原地”构造一个`T`类型的对象。这是在C++中手动调用构造函数的唯一方式。

定位`new`的一种用途是实现分配器，详见第4节。另外，定位`new`在需要精确控制对象布局或实现内存池时也非常有用。例如：

```cpp
alignas(T) unsigned char buf[sizeof(T)];  // Statically allocate storage large enough for object of type T.
T* p = new(buf) T;  // Construct a T object directly into pre-allocated storage at buf.
p->~T();  // You must manually call the object's destructor.
// Leaving this block scope automatically deallocates buf.
```

注意事项：
* 必须手动调用析构函数。
* 确保内存对齐满足对象要求。
* 不要对定位new返回的指针使用`delete`表达式。

## 3.分配函数和释放函数
头文件`<new>`提供了一组[分配函数](https://en.cppreference.com/cpp/memory/new/operator_new)(allocation function) `operator new`，以及一组[释放函数](https://en.cppreference.com/cpp/memory/new/operator_delete)(deallocation function) `operator delete`，用于分配和释放未初始化内存。

注意：`new`/`delete`表达式和`operator new`/`operator delete`函数是两个不同的东西——前者是语言层面的运算符，后者是库函数。前者会调用后者来完成内存分配或释放，详见3.1节。

### 3.1 标准operator new/delete
#### 分配函数
分配函数`operator new`的主要声明如下：

```cpp
// (1) usual allocation functions
void* operator new  (std::size_t sz);
void* operator new[](std::size_t sz);

// (2) non-throwing allocation functions
void* operator new  (std::size_t sz, const std::nothrow_t& tag) noexcept;
void* operator new[](std::size_t sz, const std::nothrow_t& tag) noexcept;

// (3) non-allocating placement allocation functions
void* operator new  (std::size_t sz, void* ptr);
void* operator new[](std::size_t sz, void* ptr);
```

标准库为这些分配函数提供了默认实现：
* (1) 分配指定字节的未初始化内存，返回指向首字节的指针。如果分配失败，则调用[new处理器](https://en.cppreference.com/cpp/memory/new/set_new_handler)或者抛出`std::bad_alloc`。（注：该重载的效果等同于`malloc()`，但C++标准并未指定通过调用`malloc()`来实现）
* (2) 同(1)，但分配失败时返回空指针。
* (3) 不分配内存，直接返回`ptr`。

`new`表达式通过调用适当的分配函数来分配内存。对于非数组类型，函数名为`operator new`；对于数组类型，函数名为`operator new[]`。调用时，将请求的字节数作为第一个参数，对于非数组类型为`sizeof(T)`。
* `new T`调用重载(1) `operator new(sizeof(T))`
* `new T[n]`调用重载(1) `operator new[](n * sizeof(T))`
* `new(std::nothrow) T`调用重载(2) `operator new(sizeof(T), std::nothrow)`
* `new(p) T`调用重载(3) `operator new(sizeof(T), p)`

#### 释放函数
释放函数`operator delete`的主要声明如下：

```cpp
// (1) size-unaware deallocation functions
void operator delete  (void* ptr) noexcept;
void operator delete[](void* ptr) noexcept;

// (2) size-aware deallocation functions
void operator delete  (void* ptr, std::size_t sz) noexcept;
void operator delete[](void* ptr, std::size_t sz) noexcept;

// (3) non-allocating placement deallocation functions (C++14)
void operator delete  (void* ptr, void* place) noexcept;
void operator delete[](void* ptr, void* place) noexcept;
```

标准库为这些释放函数提供了默认实现：
* (1) 释放由`operator new`/`operator new[]`分配的内存。如果`ptr`是空指针则什么都不做。如果`ptr`不是由匹配的分配函数返回的指针，则行为未定义。
* (2) 同(1)，可以通过自定义重载使用`sz`参数。
* (3) 什么都不做。

`delete`表达式在调用对象的析构函数之后会调用释放函数。
* `delete p`调用重载(1) `operator delete(p)`或重载(2) `operator delete(p, sizeof *p)`
* `delete[] p`调用重载(1) `operator delete[](p)`或重载(2) `operator delete[](p, n * sizeof *p)`
* `new(p) T`在初始化失败时调用重载(3) `operator delete(p, p)`

#### 小结
* `new`表达式 = `operator new` + 构造函数
* `delete`表达式 = 析构函数 + `operator delete`

### 3.2 重载operator new/delete
类可以通过重载`operator new`和`operator delete`来实现自定义内存分配策略，需要将其定义为**公有静态**成员函数（关键字`static`可省略）。这样`new`和`delete`表达式就会使用这些函数来分配或释放该类对象的内存（除非使用`::new`和`::delete`形式）。

重载的`operator new`/`operator delete`还可以接受任意的额外参数，称为“自定义placement new/delete”。

下面是一个示例：

```cpp
#include <cstddef>
#include <iostream>

// class-specific allocation/deallocation functions
struct X {
    static void* operator new(std::size_t sz) {
        std::cout << "custom new for size " << sz << '\n';
        return ::operator new(sz);
    }

    static void operator delete(void* ptr, std::size_t sz) {
        std::cout << "custom delete for size " << sz << '\n';
        ::operator delete(ptr);
    }

    static void* operator new[](std::size_t sz) {
        std::cout << "custom new[] for size " << sz << '\n';
        return ::operator new[](sz);
    }

    static void operator delete[](void* ptr, std::size_t sz) {
        std::cout << "custom delete[] for size " << sz << '\n';
        ::operator delete[](ptr);
    }

    // custom placement new
    static void* operator new(std::size_t sz, bool b) {
        std::cout << "custom placement new called, b = " << b << '\n';
        return ::operator new(sz);
    }

    // custom placement delete
    static void operator delete(void* ptr, bool b) {
        std::cout << "custom placement delete called, b = " << b << '\n';
        ::operator delete(ptr);
    }
};

int main() {
    std::cout << "sizeof(X) = " << sizeof(X) << '\n';

    X* p1 = new X;
    delete p1;

    X* p2 = new X[10];
    delete[] p2;

    X* p3 = new(true) X;  // custom placement delete will be called if X::X() throws an exception
    delete p3;
}
```

程序可能的输出如下：

```
sizeof(X) = 1
custom new for size 1
custom delete for size 1
custom new[] for size 18
custom delete[] for size 18
custom placement new called, b = 1
custom delete for size 1
```

### 3.3 替换全局operator new/delete
3.1节所述的全局`operator new`和`operator delete`是[可替换的](https://en.cppreference.com/cpp/language/replacement_function)。可以在程序中定义具有相同签名的函数（称为**替换函数**(replacement function)）来替代默认版本。

可以利用这种方式来实现全局内存追踪或调试。例如：

```cpp
#include <cstdio>
#include <cstdlib>
#include <new>

// no inline, required by [replacement.functions]/3
void* operator new(std::size_t sz) {
    std::printf("1) new(size_t), size = %zu\n", sz);
    if (sz == 0)
        ++sz; // avoid std::malloc(0) which may return nullptr on success

    if (void *ptr = std::malloc(sz))
        return ptr;

    throw std::bad_alloc{}; // required by [new.delete.single]/3
}

// no inline, required by [replacement.functions]/3
void* operator new[](std::size_t sz) {
    std::printf("2) new[](size_t), size = %zu\n", sz);
    if (sz == 0)
        ++sz; // avoid std::malloc(0) which may return nullptr on success

    if (void *ptr = std::malloc(sz))
        return ptr;

    throw std::bad_alloc{}; // required by [new.delete.single]/3
}

void operator delete(void* ptr) noexcept {
    std::printf("3) delete(void*)\n");
    std::free(ptr);
}

void operator delete(void* ptr, std::size_t size) noexcept {
    std::printf("4) delete(void*, size_t), size = %zu\n", size);
    std::free(ptr);
}

void operator delete[](void* ptr) noexcept {
    std::printf("5) delete[](void*)\n");
    std::free(ptr);
}

void operator delete[](void* ptr, std::size_t size) noexcept {
    std::printf("6) delete[](void*, size_t), size = %zu\n", size);
    std::free(ptr);
}

int main() {
    int* p1 = new int;
    delete p1;

    int* p2 = new int[10]; // guaranteed to call the replacement in C++11
    delete[] p2;
}
```

程序可能的输出如下：

```
1) new(size_t), size = 4
4) delete(void*, size_t), size = 4
2) new[](size_t), size = 40
5) delete[](void*)
```

**注意**：替换全局`operator new`/`operator delete`会影响整个程序，务必谨慎！通常推荐使用类级重载或分配器（见第4节）。

## 4.分配器
C++具名要求[Allocator](https://en.cppreference.com/cpp/named_req/Allocator)描述了**分配器**类型，用于封装内存分配/释放以及对象的构造/析构。所有需要动态分配内存的标准库容器都是通过分配器完成的。

简单来说，用于`T`类型的分配器类型`Alloc`需要满足以下要求：
* 成员类型`Alloc::value_type`定义为`T`的别名。
* `a.allocate(n)` 分配n个`T`类型对象的未初始化内存。
* `a.deallocate(p, n)` 释放`p`指向的n个`T`类型对象的内存。
* `a.construct(p, args...)` 在`p`指向的未初始化内存上构造一个`T`类型对象。
* `a.destroy(p)` 销毁`p`指向的`T`类型对象。

分配器将内存分配/释放与对象构造/销毁分离，这是`std::vector`等STL容器底层实现所必需的（详见[《C++程序设计原理与实践》笔记 第19章]({% post_url 2023-06-16-ppp-note-ch19-vector-templates-and-exceptions %}) 19.3.7节）。另外，还可以通过自定义分配器来实现自定义内存管理策略（见4.3节）。

### 4.1 std::allocator
类模板[std::allocator](https://en.cppreference.com/cpp/memory/allocator)定义在头文件`<memory>`中，是C++标准库提供的默认分配器。所有STL容器都默认使用该分配器（除非用户指定了其他分配器）。简化的实现如下：

```cpp
#include <utility>

template<class T>
class allocator {
public:
    using value_type = T;

    // allocate uninitialized storage for n objects of type T
    T* allocate(std::size_t n) {
        return static_cast<T*>(::operator new(n * sizeof(T)));
    }

    // deallocate n objects of type T starting at p
    void deallocate(T* p, std::size_t n) {
        ::operator delete(p, n * sizeof(T));
    }

    // construct a T with the value v in p
    template<class... Args>
    void construct(T* p, Args&&... args) {
        ::new(p) T(std::forward<Args>(args)...);
    }

    // destroy the T in p
    void destroy(T* p) {
        p->~T();
    }
};
```

默认分配器是无状态的，这意味着给定分配器类型的所有实例可以互换使用，一个实例可以释放由另一个实例分配的内存。

下面是一个示例：

```cpp
#include <iostream>
#include <memory>
#include <string>

int main() {
    // default allocator for strings
    std::allocator<std::string> a;

    // allocate space for 2 strings
    std::string* p = a.allocate(2);

    // construct the strings
    a.construct(p, "foo");
    a.construct(p + 1, "bar");

    std::cout << p[0] << ' ' << p[1] << '\n';

    // destroy the strings
    a.destroy(p + 1);
    a.destroy(p);

    // deallocate space for 2 strings
    a.deallocate(p, 2);
}
```

注：
* C++20移除了成员函数`construct()`和`destroy()`，应该使用`std::allocator_traits`的同名静态成员函数，后者分别调用`std::construct_at()`和`std::destroy_at()`。
* 对于非数组类型，`std::construct_at(p, args...)`等价于`new(p) T(args...)`，`std::destroy_at(p)`等价于`p->~T()`。

### 4.2 std::allocator_traits
类模板[std::allocator_traits](https://en.cppreference.com/cpp/memory/allocator_traits)提供了访问分配器的统一接口，使得可以使用相同的方式访问不同的分配器实现。

`std::allocator_traits<Alloc>`主要提供了以下成员类型（其中`Alloc`是分配器类型）：

| 成员类型 | 定义 |
| --- | --- |
| `allocator_type` | `Alloc` |
| `value_type` | `Alloc::value_type` |
| `pointer` | `Alloc::pointer` 或者 `value_type*` |
| `size_type` | `Alloc::size_type` 或者 `std::size_t` |

以及静态成员函数：
* `pointer allocate(Alloc& a, size_type n)` 调用`a.allocate(n)`
* `void deallocate(Alloc& a, pointer p, size_type n)` 调用`a.deallocate(p, n)`
* `void construct(Alloc& a, T* p, Args&&... args)` 调用`a.construct(p, std::forward<Args>(args)...)`或者`std::construct_at(p, std::forward<Args>(args)...)`
* `void destroy(Alloc& a, T* p)` 调用`a.destroy(p)`或者`std::destroy_at(p)`

例如：

```cpp
template<class Alloc>
void test(Alloc& alloc, std::size_t n) {
    using Traits = std::allocator_traits<Alloc>;
    auto* p = Traits::allocate(alloc, n);

    for (int i = 0; i < n; i++)
        Traits::construct(alloc, p + i);  // default construct

    for (int i = 0; i < n; i++)
        Traits::destroy(alloc, p + i);

    Traits::deallocate(alloc, p, n);
}

int main() {
    // use std::allocator
    std::allocator<std::string> alloc1;
    test(alloc1, 10);

    // use custom allocator
    my_allocator<std::string> alloc2;
    test(alloc2, 10);
}
```

（如果C++一开始提供了类似于Java接口的“分配器抽象基类”可能就不需要`allocator_traits`了）

### 4.3 自定义分配器
可以通过自定义分配器来控制内存分配策略，从而进行性能优化、实现对象池、调试和性能分析等。

实现自定义分配器的步骤如下：
1. 定义分配器类（通常是模板），成员类型`value_type`表示要分配的对象类型（通常是类模板参数）。
2. 定义成员函数`allocate()`和`deallocate()`，实现内存分配策略。通常不需要自定义`construct()`和`destroy()`，因为`std::allocator_traits`提供了默认实现。
3. 与STL容器集成。
4. 正确性和性能测试。

下面的示例实现了一个内存池分配器`PoolAllocator`。在构造函数中分配固定容量的内存空间，并标记每个块是否已被使用（如下图所示）。`allocate()`函数查找长度为n的连续空闲块，如果未找到则抛出异常。`deallocate()`函数将块重新标记为未使用。

![PoolAllocator](/assets/images/cpp-dynamic-memory-management/PoolAllocator.png)

```cpp
#include <cstdlib>

template<class T>
class PoolAllocator {
private:
    struct Block {
        T data;
        bool used = false;
    };

    Block* m_pool;
    size_t m_capacity;
    size_t m_used;

public:
    using value_type = T;

    explicit PoolAllocator(size_t capacity) :m_capacity(capacity), m_used(0) {
        m_pool = (Block*) std::malloc(m_capacity * sizeof(Block));
    }

    ~PoolAllocator() {
        std::free(m_pool);
    }

    PoolAllocator(const PoolAllocator& other) :m_capacity(other.m_capacity), m_used(0) {
        m_pool = (Block*) std::malloc(m_capacity * sizeof(Block));
    }

    PoolAllocator(PoolAllocator&& other) noexcept
        :m_pool(other.m_pool), m_capacity(other.m_capacity), m_used(other.m_used) {
        other.m_pool = nullptr;
        other.m_capacity = other.m_used = 0;
    }

    T* allocate(size_t n) {
        if (n == 0) return nullptr;
        if (n + m_used > m_capacity) throw std::bad_alloc();

        int start = find_free_blocks(n);
        if (start < 0)
            throw std::bad_alloc();

        for (size_t i = 0; i < n; ++i)
            m_pool[start + i].used = true;
        m_used += n;
        return &m_pool[start].data;
    }

    void deallocate(T* ptr, size_t n) {
        if (!ptr) return;

        size_t start = (Block*) ptr - m_pool;
        for (size_t i = 0; i < n; ++i)
            m_pool[start + i].used = false;
        m_used -= n;
    }

private:
    int find_free_blocks(size_t n) {
        int left = 0, right = 0, count = 0;
        while (right < m_capacity) {
            if (!m_pool[right++].used) {
                ++count;
                if (count == n)
                    return left;
            } else {
                left = right;
                count = 0;
            }
        }
        return -1;
    }
};
```

为了在STL容器中使用自定义分配器，需要将分配器类型指定为第二个模板参数（分配器的`value_type`必须与容器的`value_type`相同），并将分配器对象传递给构造器参数（如果省略则为默认构造）。例如：

```cpp
std::vector<int, PoolAllocator<int>> v(PoolAllocator<int>(100));
```

注：为了简化示例，这个分配器使用线性查找，遍历整个池子寻找连续空闲块，但存在性能较差、内存碎片问题。真正的标准库分配器实现要复杂得多，参见[《malloc和free的实现原理解析》](https://jacktang816.github.io/post/mallocandfree/)。

## 5.总结
下表总结了C++中分配内存和初始化对象的各种方式。

| 方式 | 分配内存 | 调用构造函数 |
| --- | --- | --- |
| `malloc()`<br>分配函数`operator new`<br>`std::allocator::allocate()` | ✅ | ❌ |
| 定位`new`<br>`std::allocator::construct()`<br>`std::construct_at()` | ❌ | ✅ |
| `new`表达式 | ✅ | ✅ |

下表总结了释放内存和销毁对象的各种方式。

| 方式 | 释放内存 | 调用析构函数 |
| --- | --- | --- |
| `free()`<br>释放函数`operator delete`<br>`std::allocator::deallocate()` | ✅ | ❌ |
| `std::allocator::destroy()`<br>`std::destroy_at()` | ❌ | ✅ |
| `delete`表达式 | ✅ | ✅ |

适用场景：
* 常规对象分配：使用`new`/`delete`表达式。
* STL容器：使用`std::allocator`（默认行为）。
* 与C语言库交互：使用`malloc()`/`free()`。
* 性能关键场景：自定义`operator new`或分配器。

### 5.1 常见陷阱

（1）new/delete不匹配

```cpp
int* p = new int[10];
delete p;  // undefined behavior: should use delete[]
```

（2）双重释放

```cpp
delete p;
delete p;  // undefined behavior: double free
```

这可能会在多线程程序中发生。

（3）忘记释放内存（内存泄漏）

```cpp
void func() {
    int* p = new int(42);
}  // memory leak
```

（4）使用已释放的内存

```cpp
int* p = new int(42);
delete p;
*p = 100;  // undefined behavior: use after free
```

（5）内存越界

```cpp
int* p = new int[10];
p[10] = 42;  // undefined behavior: access out of bounds
delete[] p;
```

（6）定位new不匹配析构

```cpp
void* buf = malloc(sizeof(std::vector<int>));
auto* p = new(buf) std::vector<int>{1, 2, 3, 4, 5};
// forgot to call p->~vector<int>()
free(buf);
```

### 5.2 最佳实践
下面总结了C++内存管理的最佳实践：
* 避免裸指针，优先使用智能指针。
* 优先使用STL容器。
* 遵守[RAII](https://en.cppreference.com/cpp/language/raii)原则：在构造函数中获取资源，在析构函数中释放资源。
* 必要时自定义内存分配策略，使用分配器模式。

C++提供了从底层到上层的完整内存管理工具。理解每种方式的优缺点和适用场景，有助于在不同的场景下做出正确的选择，写出更健壮、更高效的C++代码。
