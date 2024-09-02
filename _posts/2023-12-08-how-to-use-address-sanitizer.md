---
title: AddressSanitizer使用教程
date: 2023-12-08 15:17 +0800
categories: [C/C++]
tags: [cpp, address sanitizer, dynamic memory allocation, memory error, memory leak]
---
## 1.简介
AddressSanitizer（简称为ASan）是Google开发的一个C/C++内存错误检测工具，通过在编译时插入探测代码来检测释放后使用、缓冲区溢出（下标越界）、内存泄露等内存错误。该工具非常快，被检测程序平均只变慢2倍左右。

官方文档：<https://github.com/google/sanitizers/wiki/AddressSanitizer>

## 2.用法
Clang编译器（3.1版本以上）和GCC编译器（4.8版本以上）自带了AddressSanitizer。

要使用AddressSanitizer，需要添加编译选项`-fsanitize=address`。例如，下面的程序是一个“释放后使用”(use after free)的例子：

```cpp
int main() {
    int* a = new int[10];
    delete[] a;
    a[1] = 42;  // use after free
    return 0;
}
```

编译命令如下：

```shell
g++ -std=c++17 -fsanitize=address -g -o test test.cc
```

运行程序，将输出类似下面的错误信息：

```
==1636==ERROR: AddressSanitizer: heap-use-after-free on address 0x604000000054 at pc 0x00000050a2de bp 0x7ffd0e787540 sp 0x7ffd0e787538
WRITE of size 4 at 0x604000000054 thread T0
    #0 0x50a2dd in main test.cc:4
    #1 0x7f548e42c554 in __libc_start_main (/lib64/libc.so.6+0x22554)
    #2 0x40743b  (test+0x40743b)

0x604000000054 is located 4 bytes inside of 40-byte region [0x604000000050,0x604000000078)
freed by thread T0 here:
    #0 0x4d1100 in operator delete[](void*) (test+0x4d1100)
    #1 0x50a29e in main test.cc:3
    #2 0x7f548e42c554 in __libc_start_main (/lib64/libc.so.6+0x22554)

previously allocated by thread T0 here:
    #0 0x4d03b0 in operator new[](unsigned long) (test+0x4d03b0)
    #1 0x50a287 in main test.cc:2
    #2 0x7f548e42c554 in __libc_start_main (/lib64/libc.so.6+0x22554)

SUMMARY: AddressSanitizer: heap-use-after-free test.cc:4 in main
```

注：
* 如果GCC编译器报错 "ASan runtime does not come first in initial library list; you should either link runtime to your application or manually preload it with LD_PRELOAD" ，则需要添加`-static-libasan`选项手动连接`libasan`库：

```shell
g++ -std=c++17 -fsanitize=address -g -o test test.cc -static-libasan
```

或者设置环境变量LD_PRELOAD指定ASan库文件位置：

```shell
LD_PRELOAD=/path/to/libasan.so ./test
```

或者设置选项verify_asan_link_order禁止检测链接顺序：

```shell
ASAN_OPTIONS=verify_asan_link_order=0 ./test
```

参考：<https://github.com/google/sanitizers/issues/796>

* 如果使用CMake、Blade等构建工具，则需要通过特定的变量或配置来添加编译选项。例如CMake的[`CMAKE_CXX_FLAGS`](https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_FLAGS.html)变量或[`target_compile_options`](https://cmake.org/cmake/help/latest/command/target_compile_options.html)命令、Blade的[`extra_cppflags`](https://github.com/chen3feng/blade-build/blob/master/doc/en/build_rules/cc.md)属性。

## 3.错误类型
AddressSanitizer能够检测的内存错误类型包括释放后使用、缓冲区溢出（下标越界）、内存泄露、静态初始化顺序问题等。

### 3.1 释放后使用
（1）**释放后使用**(use after free)：访问已释放的动态分配内存，示例如第2节所示。

（2）**返回后使用**(use after return)：在函数返回后访问局部变量的地址。

```cpp
int* p;

void f() {
    int x = 8;
    p = &x;
}

int main() {
    f();
    *p = 42;  // use after return
    return 0;
}
```

```
==22518==ERROR: AddressSanitizer: stack-use-after-return on address 0x7f9e8c100020 at pc 0x00000050a3b9 bp 0x7ffc8cc89470 sp 0x7ffc8cc89468
WRITE of size 4 at 0x7f9e8c100020 thread T0
    #0 0x50a3b8 in main test.cc:10
    #1 0x7f9e8f6fd554 in __libc_start_main (/lib64/libc.so.6+0x22554)
    #2 0x40743b  (test+0x40743b)

Address 0x7f9e8c100020 is located in stack of thread T0 at offset 32 in frame
    #0 0x50a285 in f() test.cc:3

  This frame has 1 object(s):
    [32, 36) 'x' <== Memory access at offset 32 is inside this variable
HINT: this may be a false positive if your program uses some custom stack unwind mechanism or swapcontext
      (longjmp and C++ exceptions *are* supported)
SUMMARY: AddressSanitizer: stack-use-after-return test.cc:10 in main
```

注：AddressSanitizer默认不检测此类错误，需要设置环境变量`ASAN_OPTIONS`：

```shell
ASAN_OPTIONS=detect_stack_use_after_return=true ./test
```

环境变量`ASAN_OPTIONS`支持的选项详见文档 [AddressSanitizerFlags](https://github.com/google/sanitizers/wiki/AddressSanitizerFlags) 和
[SanitizerCommonFlags](https://github.com/google/sanitizers/wiki/SanitizerCommonFlags) 。

（3）**作用域外使用**(use after scope)：在作用域结束后访问局部变量的地址。

```cpp
int main() {
    int* p;
    {
        int x = 8;
        p = &x;
    }
    *p = 42;  // use after scope
    return 0;
}
```

```
==26509==ERROR: AddressSanitizer: stack-use-after-scope on address 0x7ffc55a44540 at pc 0x00000050a371 bp 0x7ffc55a44500 sp 0x7ffc55a444f8
WRITE of size 4 at 0x7ffc55a44540 thread T0
    #0 0x50a370 in main test.cc:7
    #1 0x7efdda924554 in __libc_start_main (/lib64/libc.so.6+0x22554)
    #2 0x40743b  (test+0x40743b)

Address 0x7ffc55a44540 is located in stack of thread T0 at offset 32 in frame
    #0 0x50a285 in main test.cc:1

  This frame has 1 object(s):
    [32, 36) 'x' <== Memory access at offset 32 is inside this variable
HINT: this may be a false positive if your program uses some custom stack unwind mechanism or swapcontext
      (longjmp and C++ exceptions *are* supported)
SUMMARY: AddressSanitizer: stack-use-after-scope test.cc:7 in main
```

### 3.2 缓冲区溢出（下标越界）
（1）**堆缓冲区溢出**(heap buffer overflow)：动态分配数组的下标越界访问。

```cpp
int main() {
    int* a = new int[10];
    a[20] = 42;  // heap buffer overflow
    delete[] a;
    return 0;
}
```

```
==28579==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x6040000000a0 at pc 0x00000050a2cb bp 0x7fffcbbd4d20 sp 0x7fffcbbd4d18
WRITE of size 4 at 0x6040000000a0 thread T0
    #0 0x50a2ca in main test.cc:3
    #1 0x7fc1d653c554 in __libc_start_main (/lib64/libc.so.6+0x22554)
    #2 0x40743b  (test+0x40743b)

Address 0x6040000000a0 is a wild pointer.
SUMMARY: AddressSanitizer: heap-buffer-overflow test.cc:3 in main
```

（2）**栈缓冲区溢出**(stack buffer overflow)：栈数组的下标越界访问。

```cpp
int main() {
    int a[10];
    a[20] = 42;  // stack buffer overflow
    return 0;
}
```

```
==29358==ERROR: AddressSanitizer: stack-buffer-overflow on address 0x7ffc3df46870 at pc 0x00000050a322 bp 0x7ffc3df467f0 sp 0x7ffc3df467e8
WRITE of size 4 at 0x7ffc3df46870 thread T0
    #0 0x50a321 in main test.cc:3
    #1 0x7fa1bb04d554 in __libc_start_main (/lib64/libc.so.6+0x22554)
    #2 0x40743b  (test+0x40743b)

Address 0x7ffc3df46870 is located in stack of thread T0 at offset 112 in frame
    #0 0x50a285 in main test.cc:1

  This frame has 1 object(s):
    [32, 72) 'a' <== Memory access at offset 112 overflows this variable
HINT: this may be a false positive if your program uses some custom stack unwind mechanism or swapcontext
      (longjmp and C++ exceptions *are* supported)
SUMMARY: AddressSanitizer: stack-buffer-overflow test.cc:3 in main
```

（3）**全局缓冲区溢出**(global buffer overflow)：全局数组的下标越界访问。

```cpp
int a[10];

int main() {
    a[20] = 42;  // global buffer overflow
    return 0;
}
```

```
==30628==ERROR: AddressSanitizer: global-buffer-overflow on address 0x0000011a86b0 at pc 0x00000050a2aa bp 0x7ffce8460870 sp 0x7ffce8460868
WRITE of size 4 at 0x0000011a86b0 thread T0
    #0 0x50a2a9 in main test.cc:4
    #1 0x7f7a8212b554 in __libc_start_main (/lib64/libc.so.6+0x22554)
    #2 0x40743b  (test+0x40743b)

0x0000011a86b0 is located 40 bytes to the right of global variable 'a' defined in 'test.cc:1:5' (0x11a8660) of size 40
SUMMARY: AddressSanitizer: global-buffer-overflow test.cc:4 in main
```

### 3.3 内存泄露
**内存泄露**(memory leaks)：动态分配的内存没有被释放。

示例1：

```cpp
int main() {
    int* a = new int[10];
    a[1] = 42;
    // memory leaks
    return 0;
}
```

```
==12833==ERROR: LeakSanitizer: detected memory leaks

Direct leak of 40 byte(s) in 1 object(s) allocated from:
    #0 0x4d03b0 in operator new[](unsigned long) (test+0x4d03b0)
    #1 0x50a287 in main test.cc:2
    #2 0x7f8a1005e554 in __libc_start_main (/lib64/libc.so.6+0x22554)

SUMMARY: AddressSanitizer: 40 byte(s) leaked in 1 allocation(s).
```

示例2：

```cpp
int* f() {
  return new int[10];
}

int main() {
    int* a = f();
    a[1] = 42;
    return 0;
}
```

```
==13887==ERROR: LeakSanitizer: detected memory leaks

Direct leak of 40 byte(s) in 1 object(s) allocated from:
    #0 0x4d03b0 in operator new[](unsigned long) (test+0x4d03b0)
    #1 0x50a283 in f() test.cc:2
    #2 0x50a292 in main test.cc:6
    #3 0x7f39b8cf7554 in __libc_start_main (/lib64/libc.so.6+0x22554)

SUMMARY: AddressSanitizer: 40 byte(s) leaked in 1 allocation(s).
```

### 3.4 静态初始化顺序问题
**静态初始化顺序问题**(static initialization order fiasco)：在同一个翻译单元中的全局变量按它们出现的顺序初始化，但不同翻译单元中的全局变量的初始化顺序是不确定的。

例如：

foo.cc

```cpp
int a = 1;
int b1 = a + 2;  // b1 becomes 3
```

test.cc

```cpp
#include <iostream>

extern int b1;
int b2 = b1 + 2;  // b2 becomes 2 or 5

int main() {
    std::cout << b1 << ' ' << b2 << '\n';
    return 0;
}
```

程序的结果可能随着编译命令的变化而变化：

```shell
$ g++ -std=c++17 -o test test.cc foo.cc && ./test 
3 2
$ g++ -std=c++17 -o test foo.cc test.cc && ./test 
3 5
```

此类错误的检测默认是关闭的，需要使用设置环境变量`ASAN_OPTIONS`的`check_initialization_order=true`或`strict_init_order=true`选项开启：

```shell
ASAN_OPTIONS=check_initialization_order=true ./test
ASAN_OPTIONS=check_initialization_order=true:strict_init_order=true ./test
```

```
==30838==ERROR: AddressSanitizer: initialization-order-fiasco on address 0x0000011a99c0 at pc 0x00000050a3ea bp 0x7ffdeae0cf90 sp 0x7ffdeae0cf88
READ of size 4 at 0x0000011a99c0 thread T0
    #0 0x50a3e9 in __static_initialization_and_destruction_0 test.cc:4
    #1 0x50a444 in _GLOBAL__sub_I_b2 test.cc:9
    #2 0x50a58c in __libc_csu_init (test+0x50a58c)
    #3 0x7f176c6284e4 in __libc_start_main (/lib64/libc.so.6+0x224e4)
    #4 0x40747b  (test+0x40747b)

0x0000011a99c0 is located 0 bytes inside of global variable 'b1' defined in 'foo.cc:2:5' (0x11a99c0) of size 4
  registered at:
    #0 0x41b320 in __asan_register_globals.part.11 (test+0x41b320)
    #1 0x50a530 in _GLOBAL__sub_I_00099_1_a (test+0x50a530)
    #2 0x50a58c in __libc_csu_init (test+0x50a58c)

SUMMARY: AddressSanitizer: initialization-order-fiasco test.cc:4 in __static_initialization_and_destruction_0
```

另见：
* [What’s the “static initialization order ‘fiasco’ (problem)”?](https://isocpp.org/wiki/faq/ctors#static-init-order)
* [《C++程序设计原理与实践》笔记 第8章]({% post_url 2022-12-24-ppp-note-ch08-technicalities-functions-etc %}) 8.6.2节

## 4.实例
真实C++程序的内存错误远比上面的示例更加隐蔽、更加难以发现。下面通过一些从真实业务代码中抽象出来的示例说明如何利用AddressSanitizer来帮助发现内存错误。

### 4.1 未检查空指针

```cpp
#include <iostream>
#include <map>
#include <memory>

struct BiddingPositionData {
    uint64_t cost;
};

using BiddingPositionDataMap = std::map<uint64_t, BiddingPositionData>;

class BiddingPositionDataDict {
public:
    bool LoadData() {
        std::shared_ptr<BiddingPositionDataMap> new_data = std::make_shared<BiddingPositionDataMap>();
        // load data ...
        if (new_data->empty()) {
            return false;
        }

        data_ = new_data;
        return true;
    }

    const BiddingPositionDataMap* GetData() const {
        return data_.get();
    }

private:
    std::shared_ptr<BiddingPositionDataMap> data_;
};

class BiddingDataInterface {
public:
    BiddingDataInterface(BiddingPositionDataDict* data_dict) :dict_(data_dict) {}

    uint64_t GetCost(uint64_t start_time, uint64_t end_time) const {
        const auto& data = *dict_->GetData();  // (1)
        auto begin = data.lower_bound(start_time), end = data.upper_bound(end_time);
        uint64_t sum_cost = 0;
        for (auto it = begin; it != end; ++it)
            sum_cost += it->second.cost;
        return sum_cost;
    }

private:
    BiddingPositionDataDict* dict_;
};

int main() {
    BiddingPositionDataDict data_dict;
    data_dict.LoadData();

    BiddingDataInterface data_interface(&data_dict);
    std::cout << data_interface.GetCost(202312080000ULL, 202312082359ULL);
    return 0;
}
```

直接运行结果：段错误

```shell
$ ./test
Program terminated with signal: SIGSEGV
```

AddressSanitizer检测结果：

```
==4762==ERROR: AddressSanitizer: SEGV on unknown address 0x000000000010 (pc 0x00000050bb53 bp 0x7fff27e05940 sp 0x7fff27e05930 T0)
==4762==The signal is caused by a READ memory access.
==4762==Hint: address points to the zero page.
    #0 0x50bb52 in std::_Rb_tree<unsigned long, std::pair<unsigned long const, BiddingPositionData>, std::_Select1st<std::pair<unsigned long const, BiddingPositionData> >, std::less<unsigned long>, std::allocator<std::pair<unsigned long const, BiddingPositionData> > >::_M_begin() const /usr/include/c++/8.2.0/bits/stl_tree.h:759
    #1 0x50b8ab in std::_Rb_tree<unsigned long, std::pair<unsigned long const, BiddingPositionData>, std::_Select1st<std::pair<unsigned long const, BiddingPositionData> >, std::less<unsigned long>, std::allocator<std::pair<unsigned long const, BiddingPositionData> > >::lower_bound(unsigned long const&) const /usr/include/c++/8.2.0/bits/stl_tree.h:1207
    #2 0x50b332 in std::map<unsigned long, BiddingPositionData, std::less<unsigned long>, std::allocator<std::pair<unsigned long const, BiddingPositionData> > >::lower_bound(unsigned long const&) const /usr/include/c++/8.2.0/bits/stl_map.h:1265
    #3 0x50ae00 in BiddingDataInterface::GetCost(unsigned long, unsigned long) const test.cc:38
    #4 0x50a46e in main test.cc:54
    #5 0x7fc48e520554 in __libc_start_main (/lib64/libc.so.6+0x22554)
    #6 0x4074eb  (test+0x4074eb)

AddressSanitizer can not provide additional info.
SUMMARY: AddressSanitizer: SEGV /usr/include/c++/8.2.0/bits/stl_tree.h:759 in std::_Rb_tree<unsigned long, std::pair<unsigned long const, BiddingPositionData>, std::_Select1st<std::pair<unsigned long const, BiddingPositionData> >, std::less<unsigned long>, std::allocator<std::pair<unsigned long const, BiddingPositionData> > >::_M_begin() const
==4762==ABORTING
```

可以看出，段错误是由`BiddingDataInterface::GetCost()`函数调用`map::lower_bound()`导致的内存读错误。结合`BiddingPositionDataDict::GetData()`函数的代码，可以推断出该函数返回的是空指针。

结论：`dict_->GetData()`返回的是空指针。

解决方法：在`BiddingDataInterface::GetCost()`开头增加判断：

```cpp
if (!dict_->GetData())
    return 0;
```

### 4.2 返回局部变量的指针

```cpp
#include <chrono>
#include <condition_variable>
#include <iostream>
#include <memory>
#include <mutex>
#include <string>
#include <thread>
#include <vector>

struct DictConfig {
    int32_t update_interval;
};

template<class DictData>
class DictBase {
public:
    virtual ~DictBase() {
        if (!stop_) Stop();
    }

    void Init(const DictConfig& dict_conf) {
        dict_conf_ = dict_conf;
        thread_ = std::thread([this] { Update(); });
    }

    void Update() {
        while (!stop_) {
            {
                std::unique_lock<std::mutex> lock(mutex_);
                if (stop_) break;
                ready_ |= DictLoad(data_);
                condition_.notify_all();
                std::this_thread::yield();
            }
            std::this_thread::sleep_for(std::chrono::seconds(dict_conf_.update_interval));
        }
    }

    virtual bool DictLoad(std::shared_ptr<DictData> data) {
        std::shared_ptr<DictData> new_data = std::make_shared<DictData>();
        if (!LoadData(new_data.get())) return false;
        data_ = new_data;  // (2)
        return true;
    }

    virtual bool LoadData(DictData* data) = 0;

    std::shared_ptr<const DictData> GetData() const { return data_; }

    bool Ready() const { return ready_; }

    void WaitUntilReady() {
        std::unique_lock<std::mutex> lock(wait_mutex_);
        condition_.wait(lock, [this] { return ready_; });
    }

    void Stop() {
        stop_ = true;
        std::unique_lock<std::mutex> lock(mutex_);
        thread_.join();
    }

private:
    DictConfig dict_conf_;
    std::shared_ptr<DictData> data_;

    std::mutex mutex_;
    std::mutex wait_mutex_;
    std::condition_variable condition_;
    std::thread thread_;
    bool stop_ = false;
    bool ready_ = false;
};

struct StrategyInfo {
    uint32_t strategy_id;
    std::string strategy_name;
};

struct StrategyInfos {
    std::vector<StrategyInfo> strategies;
};

const StrategyInfos MOCK_STRATEGY_CONF = { { {1, "FooStrategy"}, {2, "BarStrategy"} } };

class StrategyConfDict : public DictBase<StrategyInfos> {
public:
    bool LoadData(StrategyInfos* data) override {
        *data = MOCK_STRATEGY_CONF;
        return true;
    }
};

class BaseStrategy {
public:
    virtual ~BaseStrategy() = default;

    void Init(const StrategyInfo* strategy_info) {
        strategy_info_ = strategy_info;
    }

    void DoProcess() {
        std::cout << "Strategy " << strategy_info_->strategy_id
                << ": " << strategy_info_->strategy_name << '\n';
    }

private:
    const StrategyInfo* strategy_info_;
};

class StrategyManager {
public:
    StrategyManager(StrategyConfDict* dict) :dict_(dict) {}

    ~StrategyManager() {
        RemoveStrategies();
    }

    void Init() {
        RemoveStrategies();
        auto config = dict_->GetData();
        for (auto strategy_info : config->strategies) {  // (1)
            BaseStrategy* strategy = new BaseStrategy;
            strategy->Init(&strategy_info);
            strategies_.push_back(strategy);
        }
    }

    void DoProcess() {
        Init();
        for (auto strategy : strategies_)
            strategy->DoProcess();
    }

    void RemoveStrategies() {
        for (BaseStrategy* s : strategies_)
            delete s;
        strategies_.clear();
    }

private:
    StrategyConfDict* dict_;
    std::vector<BaseStrategy*> strategies_;
};

int main() {
    DictConfig dict_conf{1};
    StrategyConfDict strategy_conf_dict;
    strategy_conf_dict.Init(dict_conf);
    strategy_conf_dict.WaitUntilReady();

    StrategyManager strategy_manager(&strategy_conf_dict);
    strategy_manager.DoProcess();
    return 0;
}
```

直接运行结果：输出乱码

```shell
./test
Strategy 0: 
Strategy 3131132832: 
�����	���
�����
����0~����/:���02E���H
```

注意，可能需要添加编译选项`-pthread`。

AddressSanitizer检测结果（需要设置`ASAN_OPTIONS=detect_stack_use_after_return=true`）：

```
==10571==ERROR: AddressSanitizer: stack-use-after-return on address 0x7fa854200520 at pc 0x00000050c508 bp 0x7fff984e0270 sp 0x7fff984e0268
READ of size 4 at 0x7fa854200520 thread T0
    #0 0x50c507 in BaseStrategy::DoProcess() test.cc:103
    #1 0x50cd02 in StrategyManager::DoProcess() test.cc:132
    #2 0x50b7a1 in main test.cc:153
    #3 0x7fa8575bb554 in __libc_start_main (/lib64/libc.so.6+0x22554)
    #4 0x40861b  (test+0x40861b)

Address 0x7fa854200520 is located in stack of thread T0 at offset 288 in frame
    #0 0x50c70b in StrategyManager::Init() test.cc:119

  This frame has 5 object(s):
    [32, 40) '__for_begin'
    [96, 104) '__for_end'
    [160, 168) 'strategy'
    [224, 240) 'config'
    [288, 304) 'strategy_info' <== Memory access at offset 288 is inside this variable
HINT: this may be a false positive if your program uses some custom stack unwind mechanism or swapcontext
      (longjmp and C++ exceptions *are* supported)
SUMMARY: AddressSanitizer: stack-use-after-return test.cc:103 in BaseStrategy::DoProcess()
```

可以看出，`BaseStrategy::DoProcess()`函数中发生了“返回后使用”错误，即`strategy_info_`指针指向的对象已经被销毁。该成员通过`BaseStrategy::Init()`函数参数传递进来，查看调用者`StrategyManager::Init()`的代码，发现注释(1)处循环变量`strategy_info`是值拷贝的，循环体中的`strategy->Init(&strategy_info)`获取了局部变量的地址，导致当前函数返回后该指针变成野指针。

结论：循环变量错误地写成值拷贝，导致获取了局部变量的地址。

解决方法：循环变量声明由`auto`改为`const auto&`。

修改后程序正确输出：

```shell
./test
Strategy 1: FooStrategy
Strategy 2: BarStrategy
```

### 4.3 普通指针+shared_ptr被销毁(1)
上一节修改后的程序还有一个更隐蔽的bug，因为涉及线程之间的数据竞争。在`BaseStrategy::DoProcess()`开头添加一行（模拟处理耗时）：

```cpp
std::this_thread::sleep_for(std::chrono::seconds(2));  // processing...
```

直接运行结果：段错误

```shell
./test
Program terminated with signal: SIGSEGV
```

AddressSanitizer检测结果：

```
==15968==ERROR: AddressSanitizer: heap-use-after-free on address 0x603000001840 at pc 0x00000050c616 bp 0x7ffcf725c2c0 sp 0x7ffcf725c2b8
READ of size 4 at 0x603000001840 thread T0
    #0 0x50c615 in BaseStrategy::DoProcess() test.cc:104
    #1 0x50cd36 in StrategyManager::DoProcess() test.cc:133
    #2 0x50b7a1 in main test.cc:154
    #3 0x7fe9e0688554 in __libc_start_main (/lib64/libc.so.6+0x22554)
    #4 0x40861b  (test+0x40861b)

0x603000001840 is located 0 bytes inside of 32-byte region [0x603000001840,0x603000001860)
freed by thread T1 here:
    #0 0x4d2140 in operator delete(void*) (test+0x4d2140)
    #1 0x51466f in __gnu_cxx::new_allocator<StrategyInfo>::deallocate(StrategyInfo*, unsigned long) /usr/include/c++/8.2.0/ext/new_allocator.h:125
    #2 0x512afc in std::allocator_traits<std::allocator<StrategyInfo> >::deallocate(std::allocator<StrategyInfo>&, StrategyInfo*, unsigned long) /usr/include/c++/8.2.0/bits/alloc_traits.h:462
    #3 0x51062f in std::_Vector_base<StrategyInfo, std::allocator<StrategyInfo> >::_M_deallocate(StrategyInfo*, unsigned long) /usr/include/c++/8.2.0/bits/stl_vector.h:304
    #4 0x5102b7 in std::_Vector_base<StrategyInfo, std::allocator<StrategyInfo> >::~_Vector_base() /usr/include/c++/8.2.0/bits/stl_vector.h:285
    #5 0x50dc05 in std::vector<StrategyInfo, std::allocator<StrategyInfo> >::~vector() /usr/include/c++/8.2.0/bits/stl_vector.h:570
    #6 0x517bf3 in StrategyInfos::~StrategyInfos() test.cc:80
    #7 0x517f07 in void __gnu_cxx::new_allocator<StrategyInfos>::destroy<StrategyInfos>(StrategyInfos*) /usr/include/c++/8.2.0/ext/new_allocator.h:140
    #8 0x517ec0 in void std::allocator_traits<std::allocator<StrategyInfos> >::destroy<StrategyInfos>(std::allocator<StrategyInfos>&, StrategyInfos*) /usr/include/c++/8.2.0/bits/alloc_traits.h:487
    #9 0x517cb2 in std::_Sp_counted_ptr_inplace<StrategyInfos, std::allocator<StrategyInfos>, (__gnu_cxx::_Lock_policy)2>::_M_dispose() /usr/include/c++/8.2.0/bits/shared_ptr_base.h:552
    #10 0x510150 in std::_Sp_counted_base<(__gnu_cxx::_Lock_policy)2>::_M_release() /usr/include/c++/8.2.0/bits/shared_ptr_base.h:155
    #11 0x50d71b in std::__shared_count<(__gnu_cxx::_Lock_policy)2>::~__shared_count() /usr/include/c++/8.2.0/bits/shared_ptr_base.h:706
    #12 0x50d061 in std::__shared_ptr<StrategyInfos, (__gnu_cxx::_Lock_policy)2>::~__shared_ptr() /usr/include/c++/8.2.0/bits/shared_ptr_base.h:1145
    #13 0x50d07d in std::shared_ptr<StrategyInfos>::~shared_ptr() /usr/include/c++/8.2.0/bits/shared_ptr.h:103
    #14 0x511e14 in DictBase<StrategyInfos>::Update() test.cc:31
    #15 0x50f8a7 in DictBase<StrategyInfos>::Init(DictConfig const&)::{lambda()#1}::operator()() const test.cc:23
    #16 0x513c66 in void std::__invoke_impl<void, DictBase<StrategyInfos>::Init(DictConfig const&)::{lambda()#1}>(std::__invoke_other, DictBase<StrategyInfos>::Init(DictConfig const&)::{lambda()#1}&&) /usr/include/c++/8.2.0/bits/invoke.h:60
    #17 0x511fd0 in std::__invoke_result<DictBase<StrategyInfos>::Init(DictConfig const&)::{lambda()#1}>::type std::__invoke<DictBase<StrategyInfos>::Init(DictConfig const&)::{lambda()#1}>(std::__invoke_result&&, (DictBase<StrategyInfos>::Init(DictConfig const&)::{lambda()#1}&&)...) /usr/include/c++/8.2.0/bits/invoke.h:95
    #18 0x517f33 in decltype (__invoke((_S_declval<0ul>)())) std::thread::_Invoker<std::tuple<DictBase<StrategyInfos>::Init(DictConfig const&)::{lambda()#1}> >::_M_invoke<0ul>(std::_Index_tuple<0ul>) /usr/include/c++/8.2.0/thread:234
    #19 0x517edb in std::thread::_Invoker<std::tuple<DictBase<StrategyInfos>::Init(DictConfig const&)::{lambda()#1}> >::operator()() /usr/include/c++/8.2.0/thread:243
    #20 0x517e7f in std::thread::_State_impl<std::thread::_Invoker<std::tuple<DictBase<StrategyInfos>::Init(DictConfig const&)::{lambda()#1}> > >::_M_run() /usr/include/c++/8.2.0/thread:186
    #21 0x7fe9e142230e  (/usr/includelib64/libstdc++.so.6+0xc430e)

previously allocated by thread T1 here:
    #0 0x4d13d0 in operator new(unsigned long) (test+0x4d13d0)
    #1 0x5158be in __gnu_cxx::new_allocator<StrategyInfo>::allocate(unsigned long, void const*) /usr/include/c++/8.2.0/ext/new_allocator.h:111
    #2 0x5145e3 in std::allocator_traits<std::allocator<StrategyInfo> >::allocate(std::allocator<StrategyInfo>&, unsigned long) /usr/include/c++/8.2.0/bits/alloc_traits.h:436
    #3 0x51299f in std::_Vector_base<StrategyInfo, std::allocator<StrategyInfo> >::_M_allocate(unsigned long) /usr/include/c++/8.2.0/bits/stl_vector.h:296
    #4 0x5108cf in StrategyInfo* std::vector<StrategyInfo, std::allocator<StrategyInfo> >::_M_allocate_and_copy<__gnu_cxx::__normal_iterator<StrategyInfo const*, std::vector<StrategyInfo, std::allocator<StrategyInfo> > > >(unsigned long, __gnu_cxx::__normal_iterator<StrategyInfo const*, std::vector<StrategyInfo, std::allocator<StrategyInfo> > >, __gnu_cxx::__normal_iterator<StrategyInfo const*, std::vector<StrategyInfo, std::allocator<StrategyInfo> > >) /usr/include/c++/8.2.0/bits/stl_vector.h:1398
    #5 0x50de32 in std::vector<StrategyInfo, std::allocator<StrategyInfo> >::operator=(std::vector<StrategyInfo, std::allocator<StrategyInfo> > const&) /usr/include/c++/8.2.0/bits/vector.tcc:214
    #6 0x50c418 in StrategyInfos::operator=(StrategyInfos const&) test.cc:80
    #7 0x50c440 in StrategyConfDict::LoadData(StrategyInfos*) test.cc:89
    #8 0x513a02 in DictBase<StrategyInfos>::DictLoad(std::shared_ptr<StrategyInfos>) test.cc:41
    #9 0x511daa in DictBase<StrategyInfos>::Update() test.cc:31
    #10 0x50f8a7 in DictBase<StrategyInfos>::Init(DictConfig const&)::{lambda()#1}::operator()() const test.cc:23
    #11 0x513c66 in void std::__invoke_impl<void, DictBase<StrategyInfos>::Init(DictConfig const&)::{lambda()#1}>(std::__invoke_other, DictBase<StrategyInfos>::Init(DictConfig const&)::{lambda()#1}&&) /usr/include/c++/8.2.0/bits/invoke.h:60
    #12 0x511fd0 in std::__invoke_result<DictBase<StrategyInfos>::Init(DictConfig const&)::{lambda()#1}>::type std::__invoke<DictBase<StrategyInfos>::Init(DictConfig const&)::{lambda()#1}>(std::__invoke_result&&, (DictBase<StrategyInfos>::Init(DictConfig const&)::{lambda()#1}&&)...) /usr/include/c++/8.2.0/bits/invoke.h:95
    #13 0x517f33 in decltype (__invoke((_S_declval<0ul>)())) std::thread::_Invoker<std::tuple<DictBase<StrategyInfos>::Init(DictConfig const&)::{lambda()#1}> >::_M_invoke<0ul>(std::_Index_tuple<0ul>) /usr/include/c++/8.2.0/thread:234
    #14 0x517edb in std::thread::_Invoker<std::tuple<DictBase<StrategyInfos>::Init(DictConfig const&)::{lambda()#1}> >::operator()() /usr/include/c++/8.2.0/thread:243
    #15 0x517e7f in std::thread::_State_impl<std::thread::_Invoker<std::tuple<DictBase<StrategyInfos>::Init(DictConfig const&)::{lambda()#1}> > >::_M_run() /usr/include/c++/8.2.0/thread:186
    #16 0x7fe9e142230e  (/usr/includelib64/libstdc++.so.6+0xc430e)

Thread T1 created by T0 here:
    #0 0x4309d0 in pthread_create (test+0x4309d0)
    #1 0x7fe9e14225d4 in std::thread::_M_start_thread(std::unique_ptr<std::thread::_State, std::default_delete<std::thread::_State> >, void (*)()) (/usr/includelib64/libstdc++.so.6+0xc45d4)
    #2 0x50fa10 in DictBase<StrategyInfos>::Init(DictConfig const&) test.cc:23
    #3 0x50b766 in main test.cc:150
    #4 0x7fe9e0688554 in __libc_start_main (/lib64/libc.so.6+0x22554)

SUMMARY: AddressSanitizer: heap-use-after-free test.cc:104 in BaseStrategy::DoProcess()
```

可以看出，`BaseStrategy::DoProcess()`函数中发生了“释放后使用”错误，即`strategy_info_`指针指向的对象已经被释放。该成员在`StrategyManager::Init()`中初始化，指向的对象由`DictBase::data_`持有。当主线程(T0)执行到`StrategyManager::DoProcess()`函数内部`Init()`和`strategy->DoProcess()`之间时，`DictBase`的更新线程(T1)刚好重新加载了数据，导致旧数据被释放（注释(2)处）。而此时`BaseStrategy::strategy_info_`仍然指向旧数据，因此`BaseStrategy::DoProcess()`访问它指向的对象时发生了段错误。

结论：在一个线程中通过`shared_ptr`获取了普通指针，在另一个线程中`shared_ptr`被销毁，导致该指针指向已释放的对象。

解决方法：将成员`BaseStrategy::strategy_info_`的类型改为`StrategyInfo`（即保存一个副本）。

### 4.4 普通指针+shared_ptr被销毁(2)

```cpp
#include <cassert>
#include <functional>
#include <map>
#include <memory>
#include <stdexcept>
#include <string>

class WinNoticeHandler {
public:
    explicit WinNoticeHandler(uint32_t adx_id) :adx_id_(adx_id) {}

    virtual ~WinNoticeHandler() = default;

    virtual bool DecryptWinPrice(const std::string& encrypted_win_price, uint64_t* decrypted_win_price) const {
        return true;
    }

private:
    uint32_t adx_id_;
};

class GdtWinNoticeHandler : public WinNoticeHandler {
public:
    GdtWinNoticeHandler() :WinNoticeHandler(6) {}

    bool DecryptWinPrice(const std::string& encrypted_win_price, uint64_t* decrypted_win_price) const override {
        try {
            *decrypted_win_price = std::stoull(encrypted_win_price);
            return true;
        } catch (std::invalid_argument&) {
            return false;
        }
    }
};

class WinNoticeHandlerFactory {
public:
    using WinNoticeHandlerCreator = std::function<WinNoticeHandler*()>;

    void RegisterWinNoticeHandlerCreator(uint32_t adx_id, WinNoticeHandlerCreator win_notice_handler_creator) {
        creator_map_[adx_id] = win_notice_handler_creator;
    }

    std::shared_ptr<WinNoticeHandler> RetrieveWinNoticeHandler(uint32_t adx_id) const {
        auto it = creator_map_.find(adx_id);
        return it != creator_map_.end() ? std::shared_ptr<WinNoticeHandler>(it->second()) : nullptr;
    }

private:
    std::map<uint32_t, WinNoticeHandlerCreator> creator_map_;
};

class GdtWinNoticeHandlerTest {
public:
    GdtWinNoticeHandlerTest(const WinNoticeHandlerFactory& win_notice_handler_factory) {
        auto win_notice_handler = win_notice_handler_factory.RetrieveWinNoticeHandler(6);
        win_notice_handler_ = dynamic_cast<GdtWinNoticeHandler*>(win_notice_handler.get());
    }

    void TestDecryptWinPrice() {
        uint64_t decrypted_win_price = 0;
        assert(win_notice_handler_->DecryptWinPrice("123", &decrypted_win_price));
        assert(decrypted_win_price == 123);
    }

private:
    GdtWinNoticeHandler* win_notice_handler_;
};

int main() {
    WinNoticeHandlerFactory win_notice_handler_factory;
    win_notice_handler_factory.RegisterWinNoticeHandlerCreator(6, [] { return new GdtWinNoticeHandler; });

    GdtWinNoticeHandlerTest gdt_win_notice_handler_test(win_notice_handler_factory);
    gdt_win_notice_handler_test.TestDecryptWinPrice();
    return 0;
}
```

直接运行结果：段错误

```shell
./test
Program terminated with signal: SIGSEGV
```

AddressSanitizer检测结果：

```
==4690==ERROR: AddressSanitizer: heap-use-after-free on address 0x602000000010 at pc 0x00000050c91c bp 0x7fffb6811690 sp 0x7fffb6811688
READ of size 8 at 0x602000000010 thread T0
    #0 0x50c91b in GdtWinNoticeHandlerTest::TestDecryptWinPrice() test.cc:62
    #1 0x50b68c in main test.cc:75
    #2 0x7f46b4a77554 in __libc_start_main (/lib64/libc.so.6+0x22554)
    #3 0x4085ab  (test+0x4085ab)

0x602000000010 is located 0 bytes inside of 16-byte region [0x602000000010,0x602000000020)
freed by thread T0 here:
    #0 0x4d2758 in operator delete(void*, unsigned long) (test+0x4d2758)
    #1 0x512f2c in GdtWinNoticeHandler::~GdtWinNoticeHandler() test.cc:22
    #2 0x512fce in std::_Sp_counted_ptr<WinNoticeHandler*, (__gnu_cxx::_Lock_policy)2>::_M_dispose() /usr/include/c++/8.2.0/bits/shared_ptr_base.h:377
    #3 0x50ea2e in std::_Sp_counted_base<(__gnu_cxx::_Lock_policy)2>::_M_release() /usr/include/c++/8.2.0/bits/shared_ptr_base.h:155
    #4 0x50db47 in std::__shared_count<(__gnu_cxx::_Lock_policy)2>::~__shared_count() /usr/include/c++/8.2.0/bits/shared_ptr_base.h:706
    #5 0x50c40b in std::__shared_ptr<WinNoticeHandler, (__gnu_cxx::_Lock_policy)2>::~__shared_ptr() /usr/include/c++/8.2.0/bits/shared_ptr_base.h:1145
    #6 0x50c427 in std::shared_ptr<WinNoticeHandler>::~shared_ptr() /usr/include/c++/8.2.0/bits/shared_ptr.h:103
    #7 0x50c786 in GdtWinNoticeHandlerTest::GdtWinNoticeHandlerTest(WinNoticeHandlerFactory const&) test.cc:56
    #8 0x50b67c in main test.cc:74
    #9 0x7f46b4a77554 in __libc_start_main (/lib64/libc.so.6+0x22554)

previously allocated by thread T0 here:
    #0 0x4d1360 in operator new(unsigned long) (test+0x4d1360)
    #1 0x50b514 in operator() test.cc:72
    #2 0x50ba81 in _M_invoke /usr/include/c++/8.2.0/bits/std_function.h:282
    #3 0x50dad7 in std::function<WinNoticeHandler* ()>::operator()() const /usr/include/c++/8.2.0/bits/std_function.h:687
    #4 0x50c59c in WinNoticeHandlerFactory::RetrieveWinNoticeHandler(unsigned int) const test.cc:46
    #5 0x50c71a in GdtWinNoticeHandlerTest::GdtWinNoticeHandlerTest(WinNoticeHandlerFactory const&) test.cc:56
    #6 0x50b67c in main test.cc:74
    #7 0x7f46b4a77554 in __libc_start_main (/lib64/libc.so.6+0x22554)

SUMMARY: AddressSanitizer: heap-use-after-free test.cc:62 in GdtWinNoticeHandlerTest::TestDecryptWinPrice()
```

可以看出，`GdtWinNoticeHandlerTest::TestDecryptWinPrice()`函数中发生了“释放后使用”错误，即`win_notice_handler_`指针指向的对象已经被释放。在`GdtWinNoticeHandlerTest`的构造函数中，调用`WinNoticeHandlerFactory::RetrieveWinNoticeHandler()`创建handler对象，该函数返回的`shared_ptr`是handler对象的唯一引用。构造函数返回后，`shared_ptr`被销毁，因此`win_notice_handler_`成员变成野指针。

结论：指向对象的唯一`shared_ptr`在函数返回后被销毁，导致通过它获取的普通指针指向已释放的对象。

解决方法：将成员`GdtWinNoticeHandlerTest::win_notice_handler_`的类型改为`shared_ptr`：

```cpp
class GdtWinNoticeHandlerTest {
public:
    GdtWinNoticeHandlerTest(const WinNoticeHandlerFactory& win_notice_handler_factory) {
        auto win_notice_handler = win_notice_handler_factory.RetrieveWinNoticeHandler(6);
        win_notice_handler_ = std::dynamic_pointer_cast<GdtWinNoticeHandler>(win_notice_handler);
    }

    void TestDecryptWinPrice() {
        // ...
    }

private:
    std::shared_ptr<GdtWinNoticeHandler> win_notice_handler_;
};
```

### 4.5 普通指针+push_back临时对象

```cpp
#include <iostream>
#include <memory>
#include <string>
#include <vector>

struct AdResponse {
    uint32_t ad_id;
};

struct PosResponse {
    uint64_t pos_id;
    std::vector<AdResponse> ad_responses;
};

class ResponseAdData;

class WinNoticeUrl {
public:
    explicit WinNoticeUrl(const ResponseAdData* response_ad_data) :response_ad_data_(response_ad_data) {}

    std::string String() const;

private:
    const ResponseAdData* response_ad_data_;
};

class ResponseAdData {
public:
    explicit ResponseAdData(const AdResponse& ad_response)
            :ad_response_(&ad_response),
            win_notice_url_(std::make_shared<WinNoticeUrl>(this)) {}  // (1)

    void Init() { GenerateViewId(); }

    void GenerateViewId() { view_id_ = std::to_string(ad_response_->ad_id); }

    const std::string& ViewId() const { return view_id_; }

    std::string GetWinNoticeUrl() const { return win_notice_url_->String(); }

private:
    const AdResponse* ad_response_;
    std::string view_id_;
    std::shared_ptr<WinNoticeUrl> win_notice_url_;
};

std::string WinNoticeUrl::String() const {
    return "https://win.example.com/win_notice?viewid="
            + response_ad_data_->ViewId() + "&win_price=${WIN_PRICE}";
}

class ResponsePosData {
public:
    ResponsePosData(const PosResponse& pos_response) :pos_response_(&pos_response) {}

    void Init() {
        BuildResponseAdData();
        for (auto& ad_data : ad_data_)
            ad_data.Init();
    }

    void BuildResponseAdData() {
        for (const auto& ad_response : pos_response_->ad_responses)
            ad_data_.push_back(ResponseAdData(ad_response));  // (2)
    }

    const std::vector<ResponseAdData> ad_data() const { return ad_data_; }

private:
    const PosResponse* pos_response_;
    std::vector<ResponseAdData> ad_data_;
};

void Render(const ResponsePosData& response_pos_data) {
    for (const auto& response_ad_data : response_pos_data.ad_data())
        std::cout << response_ad_data.GetWinNoticeUrl() << '\n';
}

int main() {
    PosResponse pos_response{12345, { {111}, {222}, {333} }};
    ResponsePosData response_pos_data(pos_response);
    response_pos_data.Init();
    Render(response_pos_data);
    return 0;
}
```

直接运行结果：段错误

```shell
./test
Program terminated with signal: SIGSEGV
```

AddressSanitizer检测结果（需要设置`ASAN_OPTIONS=detect_stack_use_after_return=true`）：

```
==665==ERROR: AddressSanitizer: stack-use-after-return on address 0x7f88b01003a8 at pc 0x00000050e64d bp 0x7ffce5e4b170 sp 0x7ffce5e4b168
READ of size 8 at 0x7f88b01003a8 thread T0
    #0 0x50e64c in std::string::_M_data() const /usr/include/c++/8.2.0/bits/basic_string.h:3305
    #1 0x50e6bf in std::string::_M_rep() const /usr/include/c++/8.2.0/bits/basic_string.h:3313
    #2 0x50ebe1 in std::string::size() const /usr/include/c++/8.2.0/bits/basic_string.h:3827
    #3 0x50d444 in std::basic_string<char, std::char_traits<char>, std::allocator<char> > std::operator+<char, std::char_traits<char>, std::allocator<char> >(char const*, std::basic_string<char, std::char_traits<char>, std::allocator<char> > const&) /usr/include/c++/8.2.0/bits/basic_string.tcc:1165
    #4 0x50b695 in WinNoticeUrl::String() const test.cc:49
    #5 0x50c54c in ResponseAdData::GetWinNoticeUrl() const test.cc:39
    #6 0x50b9a3 in Render(ResponsePosData const&) test.cc:76
    #7 0x50bcdc in main test.cc:83
    #8 0x7f88b358a554 in __libc_start_main (/lib64/libc.so.6+0x22554)
    #9 0x40855b  (test+0x40855b)

Address 0x7f88b01003a8 is located in stack of thread T0 at offset 168 in frame
    #0 0x50c7bb in ResponsePosData::BuildResponseAdData() test.cc:62

  This frame has 3 object(s):
    [32, 40) '__for_begin'
    [96, 104) '__for_end'
    [160, 192) '<unknown>' <== Memory access at offset 168 is inside this variable
HINT: this may be a false positive if your program uses some custom stack unwind mechanism or swapcontext
      (longjmp and C++ exceptions *are* supported)
SUMMARY: AddressSanitizer: stack-use-after-return /usr/include/c++/8.2.0/bits/basic_string.h:3305 in std::string::_M_data() const
```

可以看出，`WinNoticeUrl::String()`函数访问`response_ad_data_->ViewId()`时发生了“返回后使用”错误，即`response_ad_data_`指针指向的对象已经被销毁。该成员在注释(1)处通过构造函数参数传递进来，它指向的`ResponseAdData`对象在注释(2)处构造。而调用`push_back()`时发生了拷贝构造（或移动构造），因此`WinNoticeUrl::response_ad_data_`指向一个临时对象（即调用`push_back()`的参数），而不是实际插入到`vector`的对象，调用结束后该成员就变成野指针。另外，如果`push_back()`的过程中发生向量扩容，或者`ResponsePosData`对象被拷贝，都会导致该问题。

结论：使用`push_back()`添加`ResponseAdData`对象时发生了拷贝构造（或移动构造），而`WinNoticeUrl`保存的是临时对象的指针。

解决方法一：改为使用`emplace_back()`添加元素，避免生成临时对象，并使用`reserve()`避免向量扩容：

```cpp
void ResponsePosData::BuildResponseAdData() {
    ad_data_.reserve(pos_response_->ad_responses.size());
    for (const auto& ad_response : pos_response_->ad_responses)
        ad_data_.emplace_back(ad_response);  // (2)
}
```

但这种方法仍然无法解决`ResponsePosData`对象被拷贝时的问题，应该为`ResponseAdData`定义适当的拷贝构造和拷贝赋值操作。

解决方法二：删除`ResponseAdData::win_notice_url_`成员，在`GetWinNoticeUrl()`函数中直接创建`WinNoticeUrl`的临时对象：

```cpp
std::string ResponseAdData::GetWinNoticeUrl() const {
    return WinNoticeUrl(this).String();
}
```

修改后程序正确输出：

```shell
$ ./test 
https://win.example.com/win_notice?viewid=111&win_price=${WIN_PRICE}
https://win.example.com/win_notice?viewid=222&win_price=${WIN_PRICE}
https://win.example.com/win_notice?viewid=333&win_price=${WIN_PRICE}
```

## 5.总结

慎用普通指针，尤其不要与智能指针混用！

## 参考
* [AddressSanitizer wiki](https://github.com/google/sanitizers/wiki/AddressSanitizer)
* [GCC文档 -fsanitize=address选项](https://gcc.gnu.org/onlinedocs/gcc/Instrumentation-Options.html#index-fsanitize_003daddress)
* [Clang文档 AddressSanitizer](https://clang.llvm.org/docs/AddressSanitizer.html)
