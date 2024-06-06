---
title: 【C++】thread_local变量
date: 2024-06-06 10:34 +0800
categories: [C/C++]
tags: [cpp, multi-thread, thread-local]
---
在C++多线程环境下，全局变量、局部变量、静态变量和`thread_local`变量的区别如下：

| | 全局变量 | 局部变量 | 静态变量 | `thread_local`变量 |
| --- | --- | --- | --- | --- |
| 存储期 | 程序运行期间 | 函数调用期间 | 程序运行期间 | 线程运行期间 |
| 线程安全 | × | √ | × | √ |

示例程序

```cpp
#include <iostream>
#include <thread>

int g = 0;

void global_test() {
    std::cout << &g << '\n';
    for (int i = 0; i < 10000; ++i) ++g;
    std::cout << g << '\n';
}

void local_test() {
    int n = 0;
    std::cout << &n << '\n';
    for (int i = 0; i < 10000; ++i) ++n;
    std::cout << n << '\n';
}

void static_test() {
    static int n = 0;
    std::cout << &n << '\n';
    for (int i = 0; i < 10000; ++i) ++n;
    std::cout << n << '\n';
}

void thread_local_test() {
    thread_local int n = 0;
    std::cout << &n << '\n';
    for (int i = 0; i < 10000; ++i) ++n;
    std::cout << n << '\n';
}

int main() {
    for (auto f : {global_test, local_test, static_test, thread_local_test}) {
        std::thread t1(f), t2(f);
        t1.join();
        t2.join();
    }
    return 0;
}
```

可能的输出结果：

```
0x10b7fb01c
0x10b7fb01c
10148
19117
0x7000078cdf2c
0x700007950f2c
10000
10000
0x108d0001c
0x108d0001c
15854
15936
0x600001008000
0x600001004050
10000
10000
```

参考
* [Storage class specifiers - cppreference](https://en.cppreference.com/w/cpp/language/storage_duration)
