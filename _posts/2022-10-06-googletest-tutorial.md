---
title: GoogleTest使用教程
date: 2022-10-06 15:08:10 +0800
categories: [C/C++]
tags: [googletest, cpp, unit test, gmock, mock]
---
## 1.简介
GoogleTest是由Google开发的一个C++测试框架，支持Linux、Windows和macOS操作系统，使用Bazel或CMake构建工具。

* 项目主页：<https://github.com/google/googletest>
* 官方文档：<https://google.github.io/googletest/>

## 2.基本概念
**断言**(assertion)：检查一个条件是否为真的语句，是测试的基本组成部分。断言的结果可以是**成功**(success)、**非致命失败**(nonfatal failure)或**致命失败**(fatal failure)。如果发生了致命失败，测试将立即终止，否则继续运行。

**测试**(test)：也叫**测试用例**(test case)，使用断言来验证被测试代码的行为。如果发生崩溃(coredump)或断言失败，则测试失败，否则成功。

**测试套件**(test suite)：包含一个或多个测试用例，用于组织测试用例以反映被测试代码的结构。当一个测试套件中的多个测试需要共用对象或子进程时，可以将其放入一个**测试夹具**(test fixture)类。

**测试程序**(test program)：包含多个测试套件的可执行程序。

## 3.快速入门
[Quickstart: Building with CMake - GoogleTest](https://google.github.io/googletest/quickstart-cmake.html)

下面介绍如何使用CMake运行GoogleTest。

### 3.1 前置条件
* 操作系统：Linux、Windows或macOS
* C++编译器：至少支持C++14
* 构建工具：CMake和Make

### 3.2 创建项目
首先创建项目根目录googletest-demo，之后在其中创建一个名为CMakeLists.txt的文件，内容如下：

```cmake
cmake_minimum_required(VERSION 3.14)
project(googletest-demo)

## GoogleTest requires at least C++14
set(CMAKE_CXX_STANDARD 14)

include(FetchContent)
FetchContent_Declare(
  googletest
  GIT_REPOSITORY https://github.com/google/googletest.git
  GIT_TAG release-1.12.1
)
FetchContent_MakeAvailable(googletest)
```

以上配置声明了对GoogleTest的依赖，执行构建时CMake将自动从GitHub下载GoogleTest源代码。

### 3.3 编写测试
创建一个名为hello_test.cc的源文件，内容如下：

```cpp
#include <gtest/gtest.h>

TEST(HelloTest, BasicAssertions) {
    EXPECT_STRNE("hello", "world");
    EXPECT_EQ(7 * 6, 42);
}
```

该文件使用`TEST()`宏定义了测试套件`HelloTest`中的一个测试用例`BasicAssertions`，包括两个断言。

注：此时找不到头文件<gtest/gtest.h>，因为还未下载GoogleTest源代码

### 3.4 运行测试
为了构建上面的代码，在CMakeLists.txt结尾添加以下内容：

```cmake
enable_testing()

add_executable(hello_test hello_test.cc)
target_link_libraries(hello_test GTest::gtest_main)

include(GoogleTest)
gtest_add_tests(TARGET hello_test)
```

其中`GTest::gtest_main`是GoogleTest定义的构建目标（源代码[gtest_main.cc](https://github.com/google/googletest/blob/main/googletest/src/gtest_main.cc)），包含测试程序入口，因此不需要自己编写`main()`函数，只需与该目标链接即可。

项目目录结构如下：

```
googletest-demo/
    CMakeLists.txt
    hello_test.cc
```

最后构建并运行测试，在项目根目录下执行以下命令：

```bash
$ cmake -S . -B build
-- The C compiler identification is GNU 7.4.0
-- The CXX compiler identification is GNU 7.4.0
...
-- Build files have been written to: .../googletest-demo/build

$ cmake --build build
...
[100%] Built target gmock_main

$ cd build && ctest
Test project .../googletest-demo/build
    Start 1: HelloTest.BasicAssertions
1/1 Test #1: HelloTest.BasicAssertions ........   Passed    0.02 sec

100% tests passed, 0 tests failed out of 1

Total Test time (real) =   0.13 sec
```

其中第一行命令用于配置构建系统，解析CMakeLists.txt并生成构建文件，同时自动克隆googletest仓库到build/_deps/googletest-src/目录下，头文件<gtest/gtest.h>位于该目录下的googletest/include/目录中。第二行命令用于执行编译链接操作，生成构建目标。第三行命令用于执行测试程序并报告测试结果。

注：以上方法通过CMake提供的`FetchContent`模块自动管理GoogleTest源代码，也可以先单独安装GoogleTest再手动添加到CMake项目中，参考[googletest README - Standalone CMake Project](https://github.com/google/googletest/tree/main/googletest#standalone-cmake-project)

## 4.断言
GoogleTest断言是类似于函数调用的宏，用于测试类或函数的行为。当断言失败时，GoogleTest将打印断言所在的源文件、行数以及错误信息。

每个断言都有两种版本：`ASSERT_*`版本的失败是致命失败，`EXPECT_*`版本的失败是非致命失败。

GoogleTest提供了一组断言，用于检查布尔值、使用比较运算符比较两个值、比较字符串以及浮点数等。所有断言都定义在头文件<gtest/gtest.h>中。

常用断言如下（每个断言都有对应的`ASSERT_*`版本，这里省略）：

| 断言 | 验证条件 |
| --- | --- |
| `EXPECT_TRUE(condition)` | `condition`为真 |
| `EXPECT_FALSE(condition)` | `condition`为假 |
| `EXPECT_EQ(val1, val2)` | `val1 == val2` |
| `EXPECT_NE(val1, val2)` | `val1 != val2` |
| `EXPECT_LT(val1, val2)` | `val1 < val2` |
| `EXPECT_LE(val1, val2)` | `val1 <= val2` |
| `EXPECT_GT(val1, val2)` | `val1 > val2` |
| `EXPECT_GE(val1, val2)` | `val1 >= val2` |
| `EXPECT_STREQ(str1, str2)` | C字符串`str1`和`str2`相等 |
| `EXPECT_STRNE(str1, str2)` | C字符串`str1`和`str2`不相等 |
| `EXPECT_STRCASEEQ(str1, str2)` | C字符串`str1`和`str2`相等，忽略大小写 |
| `EXPECT_STRCASENE(str1, str2)` | C字符串`str1`和`str2`不相等，忽略大小写 |
| `EXPECT_FLOAT_EQ(val1, val2)` | 两个`float`值`val1`和`val2`近似相等 |
| `EXPECT_DOUBLE_EQ(val1, val2)` | 两个`double`值`val1`和`val2`近似相等 |
| `EXPECT_NEAR(val1, val2, abs_error)` | `val1`和`val2`之差的绝对值不超过`abs_error` |
| `EXPECT_THROW(statement, exception_type)` | `statement`抛出`exception_type`类型的异常 |
| `EXPECT_ANY_THROW(statement)` | `statement`抛出任何类型的异常 |
| `EXPECT_NO_THROW(statement)` | `statement`不抛出任何异常 |
| `EXPECT_THAT(val, matcher)` | `val`满足匹配器`matcher` |

完整参考列表：[Assertions Reference](https://google.github.io/googletest/reference/assertions.html)

断言宏返回一个`ostream`对象，可以使用`<<`运算符输出自定义的失败信息。例如：

```cpp
EXPECT_TRUE(my_condition) << "My condition is not true";
```

## 5.简单测试
[Simple Tests](https://google.github.io/googletest/primer.html#simple-tests)

`TEST()`宏用于定义一个测试，语法如下：

```cpp
TEST(TestSuiteName, TestName) {
    test body
}
```

其中第一个参数是测试套件名称，第二个参数是测试用例名称，二者都必须是合法的C++标识符，并且不应该包含下划线。

测试体可以包含断言和任何C++语句。如果任何断言失败或者崩溃，则整个测试失败，否则成功。

注：`TEST()`宏实际上定义了一个名为`TestSuiteName_TestName_Test`的类，该类继承了`::testing::Test`类并覆盖了成员函数`TestBody()`，测试体就是其函数体。其（简化的）定义如下：

```cpp
#define TEST(TestSuiteName, TestName) \
class TestSuiteName##_##TestName##_Test : public ::testing::Test { \
private: \
    void TestBody() override; \
}; \
void TestSuiteName##_##TestName##_Test::TestBody()
```

### 5.1 示例
假设有一个计算阶乘的函数：

factorial.h

```cpp
#pragma once

// Returns the factorial of n
int Factorial(int n);
```

factorial.cc

```cpp
#include "factorial.h"

int Factorial(int n) {
    int p = 1;
    for (int i = 1; i <= n; ++i)
        p *= i;
    return p;
}
```

可以针对该函数编写测试：

factorial_test.cc

```cpp
#include <gtest/gtest.h>

#include "factorial.h"

// Tests factorial of 0.
TEST(FactorialTest, HandlesZeroInput) {
    EXPECT_EQ(Factorial(0), 1);
}

// Tests factorial of positive numbers.
TEST(FactorialTest, HandlesPositiveInput) {
    EXPECT_EQ(Factorial(1), 1);
    EXPECT_EQ(Factorial(2), 2);
    EXPECT_EQ(Factorial(3), 6);
    EXPECT_EQ(Factorial(8), 40320);
}
```

上述示例定义了一个名为`FactorialTest`的测试套件，其中包含`HandlesZeroInput`和`HandlesPositiveInput`两个测试用例。

在第3节目录结构的基础上，将上述三个文件放在factorial目录下，在根目录下的CMakeLists.txt文件结尾添加一行：

```
add_subdirectory(factorial)
```

factorial/CMakeLists.txt内容如下：

```cmake
add_library(factorial factorial.cc)
add_executable(factorial_test factorial_test.cc)
target_link_libraries(factorial_test factorial GTest::gtest_main)
gtest_add_tests(TARGET factorial_test)
```

目录结构如下：

```
googletest-demo/
    CMakeLists.txt
    factorial/
        CMakeLists.txt
        factorial.h
        factorial.cc
        factorial_test.cc
```

测试结果如下：

```bash
$ cmake -S . -B build
$ cmake --build build
$ cd build && ctest -R FactorialTest
Test project .../googletest-demo/build
    Start 1: FactorialTest.HandlesZeroInput
1/2 Test #1: FactorialTest.HandlesZeroInput .......   Passed    0.00 sec
    Start 2: FactorialTest.HandlesPositiveInput
2/2 Test #2: FactorialTest.HandlesPositiveInput ...   Passed    0.00 sec

100% tests passed, 0 tests failed out of 2

Total Test time (real) =   0.01 sec
```

## 6.测试夹具
[Test Fixtures](https://google.github.io/googletest/primer.html#same-data-multiple-tests)

测试夹具(text fixture)可以让多个测试用例共用相同的对象或数据。要创建一个fixture，只需继承`::testing::Test`类，在类中定义要使用的对象，在默认构造函数或`SetUp()`函数中进行初始化，在析构函数或`TearDown()`函数中进行清理（释放资源），此外还可以定义需要共用的函数。如下所示：

```cpp
// The fixture for testing class Foo.
class FooTest : public ::testing::Test {
protected:
    FooTest() {
        // You can do set-up work for each test here.
    }

    ~FooTest() override {
        // You can do clean-up work that doesn't throw exceptions here.
    }

    void SetUp() override {
        // Code here will be called immediately after the constructor (right before each test).
    }

    void TearDown() override {
        // Code here will be called immediately after each test (right before the destructor).
    }

    // Class members declared here can be used by all tests in the test suite for Foo.
};
```

要使用fixture类，使用`TEST_F()`宏而不是`TEST()`来定义测试：

```cpp
TEST_F(TestFixtureName, TestName) {
    test body
}
```

其中`TestFixtureName`既是fixture类名，也是测试套件名，在测试体中可以使用fixture类定义的数据成员。

对于每一个使用`TEST_F()`定义的测试，GoogleTest都会创建一个**新的** fixture对象，调用`SetUp()`初始化，运行测试，调用`TearDown()`清理，最后删除fixture对象。同一个测试套件中的不同测试使用不同的fixture对象，因此一个测试所做的改变不影响其他测试。

注：`TEST_F()`宏与`TEST()`唯一的区别是定义的类继承fixture类而不是`::testing::Test`：

```cpp
#define TEST_F(TestFixtureName, TestName) \
class TestFixtureName##_##TestName##_Test : public TestFixtureName { \
private: \
    void TestBody() override; \
}; \
void TestFixtureName##_##TestName##_Test::TestBody()
```

### 6.1 示例
首先编写一个队列类`Queue`（文档中并未给出实现，这里直接使用标准库`deque`类实现）：

queue.h

```cpp
#pragma once

#include <cstddef>
#include <deque>

template<class E>
class Queue {
public:
    ~Queue();
    void Enqueue(const E& element);
    E* Dequeue();
    size_t size() const;
private:
    std::deque<E*> q_;
};

template<class E>
Queue<E>::~Queue() {
    while (!q_.empty()) {
        delete q_.front();
        q_.pop_front();
    }
}

template<class E>
void Queue<E>::Enqueue(const E& element) {
    q_.push_back(new E(element));
}

template<class E>
E* Queue<E>::Dequeue() {
    if (q_.empty())
        return nullptr;
    E* e = q_.front();
    q_.pop_front();
    return e;
}

template<class E>
size_t Queue<E>::size() const {
    return q_.size();
}
```

queue.cc

```cpp
#include "queue.h"
```

注：类模板的成员函数必须在头文件中定义，本来不需要源文件，但在CMakeLists.txt的`add_library()`命令中使用头文件会报错：

```
CMake Error: Cannot determine link language for target "queue".
CMake Error: CMake can not determine linker language for target: queue
```

下面针对`Queue`类编写fixture和测试：

queue_test.cc

```cpp
#include <gtest/gtest.h>

#include "queue.h"

class QueueTest : public ::testing::Test {
protected:
    void SetUp() override {
        q1_.Enqueue(1);
        q2_.Enqueue(2);
        q2_.Enqueue(3);
    }

    Queue<int> q0_, q1_, q2_;
};

TEST_F(QueueTest, IsEmptyInitially) {
    EXPECT_EQ(q0_.size(), 0);
}

TEST_F(QueueTest, DequeueWorks) {
    int* n = q0_.Dequeue();
    EXPECT_EQ(n, nullptr);

    n = q1_.Dequeue();
    ASSERT_NE(n, nullptr);
    EXPECT_EQ(*n, 1);
    EXPECT_EQ(q1_.size(), 0);
    delete n;

    n = q2_.Dequeue();
    ASSERT_NE(n, nullptr);
    EXPECT_EQ(*n, 2);
    EXPECT_EQ(q2_.size(), 1);
    delete n;
}
```

在第3节目录结构的基础上，将上述三个文件放在queue目录下，在根目录下的CMakeLists.txt文件结尾添加一行：

```cmake
add_subdirectory(queue)
```

queue/CMakeLists.txt内容如下：

```cmake
add_library(queue queue.cc)
add_executable(queue_test queue_test.cc)
target_link_libraries(queue_test queue GTest::gtest_main)
gtest_add_tests(TARGET queue_test)
```

目录结构如下：

```
googletest-demo/
    CMakeLists.txt
    queue/
        CMakeLists.txt
        queue.h
        queue.cc
        queue_test.cc
```

测试结果如下：

```bash
$ cmake -S . -B build
$ cmake --build build
$ cd build && ctest -R QueueTest
Test project .../googletest-demo/build
    Start 1: QueueTest.IsEmptyInitially
1/2 Test #1: QueueTest.IsEmptyInitially .......   Passed    0.00 sec
    Start 2: QueueTest.DequeueWorks
2/2 Test #2: QueueTest.DequeueWorks ...........   Passed    0.01 sec

100% tests passed, 0 tests failed out of 2

Total Test time (real) =   0.01 sec
```

## 7.模拟对象
[gMock for Dummies](https://google.github.io/googletest/gmock_for_dummies.html)

在测试中使用真实对象有时是不可行的，因为真实对象依赖昂贵的、不可靠的资源（例如数据库、网络连接等）使得测试变慢或不稳定。**模拟对象**(mock object)与真实对象实现相同的接口，但可以在运行时指定**它将被如何使用**（调用什么方法、以什么参数、调用多少次、以什么顺序）以及**它应该做什么**（返回什么值）。

**假对象**(fake object)与模拟对象的区别：
* 假对象具有可用的实现，但采取了一些捷径（为了降低操作成本），例如内存文件系统；
* 模拟对象预先设定了期望接收到的调用，同时也可以指定调用时执行的动作。

二者最重要的区别是模拟对象允许你验证它和使用它的代码之间的交互方式。

个人理解：
* 假对象的作用是“替换行为”，而模拟对象既可以“替换行为”又可以“验证调用”。
* “验证调用”的基本思想是：“**相信底层接口是正确的，只需验证是否调用了正确的接口**”，这里的“底层接口”可能是系统调用、其他模块、第三方库等已经被测试或不需要在这里测试的代码。

**gMock**是一个C++ mock框架，用于解决C++中使用模拟对象困难的问题，类似于Java的jMock/EasyMock、Python的`unittest.mock`、Go的gomock。GoogleTest已经包含了gMock。

### 7.1 示例
假设正在开发一个画图程序。要想测试程序是否正确，可以对比屏幕上的绘制结果和正确的屏幕截图，但这种方式太繁琐、难以维护。实际上，在测试中不需要真正调用系统接口在屏幕上绘制图形，只需验证是否调用了正确的接口即可。

#### 7.1.1 接口定义
假设程序使用的画图接口`Turtle`如下（类似于Python的[turtle](https://docs.python.org/3/library/turtle.html)模块）：

turtle.h

```cpp
#pragma once

class Turtle {
public:
    virtual ~Turtle() = default;

    virtual void PenUp() = 0;
    virtual void PenDown() = 0;
    virtual void Forward(int distance) = 0;
    virtual void Circle(int radius) = 0;
    virtual void Turn(int degrees) = 0;
    virtual void GoTo(int x, int y) = 0;
    virtual void Head(int angle) = 0;
    virtual int GetX() const = 0;
    virtual int GetY() const = 0;
};
```

注意：`Turtle`的析构函数**必须**是虚函数，否则通过基类指针删除对象时派生类的析构函数不会被调用，从而对象无法被正确销毁。

该接口提供了控制一支画笔（可以想像为一只乌龟）的方式，可以使用`PenUp()`和`PenDown()`绘制轨迹，使用`Forward()`、`Circle()`、`Turn()`、`GoTo()`和`Head()`控制移动，使用`GetX()`和`GetY()`获取当前位置。

画图程序会使用该接口的真实实现（需要调用底层图形库接口），但在测试中使用mock实现，从而可以验证画图程序是否以正确的参数、正确的顺序调用了正确的接口，而不需要真正调用底层接口，使得测试更快、更加健壮、易于维护。

#### 7.1.2 编写mock类
为`Turtle`接口编写mock类的步骤如下：
* 从`Turtle`派生一个类`MockTurtle`；
* 选择想要mock的**虚函数**；
* 在子类的`public:`部分对每个要mock的函数编写一个`MOCK_METHOD()`宏；
* 将函数签名复制粘贴到宏参数中，并分别在返回类型和函数名之间以及函数名和参数表之间添加一个逗号；
* 对于`const`成员函数，添加第4个宏参数`(const)`；
* 覆盖虚函数建议添加`override`关键字：对于非`const`成员函数，第4个宏参数为`(override)`；对于`const`成员函数，第4个宏参数为`(const, override)`。

`MockTurtle`类的完整定义如下：

mock_turtle.h

```cpp
#pragma once

#include <gmock/gmock.h>

#include "turtle.h"

class MockTurtle : public Turtle {
public:
    ~MockTurtle() override = default;

    MOCK_METHOD(void, PenUp, (), (override));
    MOCK_METHOD(void, PenDown, (), (override));
    MOCK_METHOD(void, Forward, (int distance), (override));
    MOCK_METHOD(void, Circle, (int radius), (override));
    MOCK_METHOD(void, Turn, (int degrees), (override));
    MOCK_METHOD(void, GoTo, (int x, int y), (override));
    MOCK_METHOD(void, Head, (int angle), (override));
    MOCK_METHOD(int, GetX, (), (const, override));
    MOCK_METHOD(int, GetY, (), (const, override));
};
```

`MOCK_METHOD()`宏会生成函数的定义。

#### 7.1.3 在测试中使用mock类
在测试中使用mock类的典型方式如下：
* 创建mock对象；
* 指定期望的调用方式（什么方法、以什么参数、被调用几次等），同时也可以指定方法被调用时的行为；
* 在被测函数中使用mock对象，同时也可以使用断言检查函数结果；
* 当mock对象被销毁时（测试函数返回前），gMock会自动检查期望的调用是否满足，如果不满足测试将会失败。

假设画图程序的一部分`Painter`类利用`Turtle`接口实现了画直线、画长方形和画圆的三个功能：

painter.h

```cpp
#pragma once

#include "turtle.h"

class Painter {
public:
    explicit Painter(Turtle* turtle): turtle_(turtle) {}
    bool DrawLine(int x1, int y1, int x2, int y2);
    bool DrawRectangle(int x, int y, int length, int width);
    bool DrawCircle(int x, int y, int r);
private:
    Turtle* turtle_;
};
```

painter.cc

```cpp
#include "painter.h"

bool Painter::DrawLine(int x1, int y1, int x2, int y2) {
    turtle_->GoTo(x1, y1);
    turtle_->PenDown();
    turtle_->GoTo(x2, y2);
    turtle_->PenUp();
    return true;
}

bool Painter::DrawRectangle(int x, int y, int length, int width) {
    if (length <= 0 || width <= 0)
        return false;
    turtle_->GoTo(x, y);
    turtle_->Head(270);
    turtle_->PenDown();
    turtle_->Forward(width);
    turtle_->Turn(90);
    turtle_->Forward(length);
    turtle_->Turn(90);
    turtle_->Forward(width);
    turtle_->Turn(90);
    turtle_->Forward(length);
    turtle_->PenUp();
    return true;
}

bool Painter::DrawCircle(int x, int y, int r) {
    if (r <= 0)
        return false;
    turtle_->GoTo(x, y - r);
    turtle_->Head(0);
    turtle_->PenDown();
    turtle_->Circle(r);
    turtle_->PenUp();
    return true;
}
```

下面针对`DrawCircle()`函数编写一个简单的测试：

painter_test.cc

```cpp
#include <gmock/gmock.h>

#include "painter.h"
#include "mock_turtle.h"

using ::testing::AtLeast;

TEST(PainterTest, DrawCircle) {
    MockTurtle turtle;
    EXPECT_CALL(turtle, PenDown()).Times(AtLeast(1));

    Painter painter(&turtle);
    EXPECT_TRUE(painter.DrawCircle(0, 0, 10));
}
```

该测试验证mock对象`turtle`的`PenDown()`方法将被调用至少一次，如果该方法最终没有被调用，则测试失败。

#### 7.1.4 运行测试
在第3节目录结构的基础上，将上述五个文件放在graphics目录下，在根目录下的CMakeLists.txt文件结尾添加一行：

```cmake
add_subdirectory(graphics)
```

graphics/CMakeLists.txt内容如下：

```cmake
add_library(painter painter.cc)
add_executable(painter_test painter_test.cc)
target_link_libraries(painter_test painter GTest::gmock_main)
gtest_add_tests(TARGET painter_test)
```

目录结构如下：

```
googletest-demo/
    CMakeLists.txt
    graphics/
        CMakeLists.txt
        turtle.h
        mock_turtle.h
        painter.h
        painter.cc
        painter_test.cc
```

测试结果如下：

```bash
$ cmake -S . -B build
$ cmake --build build
$ cd build && ctest -R PainterTest
Test project .../googletest-demo/build
    Start 1: PainterTest.DrawCircle
1/1 Test #1: PainterTest.DrawCircle ...........   Passed    0.00 sec

100% tests passed, 0 tests failed out of 1

Total Test time (real) =   0.01 sec
```

**注意**：gMock要求期望必须在mock函数被调用**之前**设置（即`EXPECT_CALL()`必须在`painter.DrawCircle()`之前调用），否则行为是**未定义的**。`EXPECT_CALL()`的含义是期望**将来发生的调用**，而不是已经发生的调用。

上面的测试非常简单，仅仅验证了一个mock函数是否被调用。gMock还允许对参数和调用次数进行验证，此外还可以指定被调用时的行为。

### 7.2 设置期望
使用mock对象的关键是设置正确的期望（即验证调用）。

#### 7.2.1 通用语法
gMock使用`EXPECT_CALL()`宏来对mock函数设置期望，通用语法为：

```cpp
EXPECT_CALL(mock_object, method(matchers))
    .Times(cardinality)
    .WillOnce(action)
    .WillRepeatedly(action);
```

宏的第一个参数是mock对象，第二个参数是方法名和参数匹配器，另外还可以指定期望调用次数和被调用时的行为。

注：`Times()`、`WillOnce()`和`WillRepeatedly()`这三个子句必须按顺序写。

#### 7.2.2 匹配器：验证参数
**匹配器**(matcher)用于验证一个参数，可用于`EXPECT_THAT()`或`EXPECT_CALL()`断言。对于`EXPECT_CALL()`，匹配器由第二个参数中的参数表指定，用于验证mock函数的实际参数。

直接指定参数值表示期望参数等于指定的值：

```cpp
// 期望turtle.Forward(100)被调用
EXPECT_CALL(turtle, Forward(100));
```

这里的100等价于`Eq(100)`。

通配符`_`表示任意值：

```cpp
using ::testing::_;
// 期望turtle.GoTo(50, y)被调用，y为任意值
EXPECT_CALL(turtle, GoTo(50, _));
```

内置匹配器`Ge(value)`表示期望参数大于指定的值：

```cpp
using ::testing::Ge;
// 期望turtle.Forward(x)被调用，且x≥100
EXPECT_CALL(turtle, Forward(Ge(100)));
```

如果不对参数做任何限制，则可以省略参数表：

```cpp
// 期望turtle.Forward(x)被调用，x为任意值
EXPECT_CALL(turtle, Forward);
// 期望turtle.GoTo(x, y)被调用，x和y为任意值
EXPECT_CALL(turtle, GoTo);
```

匹配器完整参考列表：[Matchers Reference](https://google.github.io/googletest/reference/matchers.html)

#### 7.2.3 基数：验证调用次数
**基数**(cardinality)用于验证mock函数的调用次数，由`EXPECT_CALL()`后面跟着的`Times()`子句指定，至多使用一次。

直接指定数字表示恰好调用指定的次数：

```cpp
// 期望turtle.Forward(100)被调用恰好两次
EXPECT_CALL(turtle, Forward(100)).Times(2);
```

`AtLeast(n)`表示至少被调用n次：

```cpp
using ::testing::AtLeast; 
// 期望turtle.PenDown()被调用至少一次
EXPECT_CALL(turtle, PenDown()).Times(AtLeast(1));
```

如果没有指定`Times()`子句，则gMock会按以下规则推断基数：
* 如果没有`WillOnce()`和`WillRepeatedly()`，则基数为`Times(1)`
* 如果有n个`WillOnce()`、没有`WillRepeatedly()`，其中n≥1，则基数为`Times(n)`
* 如果有n个`WillOnce()`、一个`WillRepeatedly()`，其中n≥0，则基数为`Times(AtLeast(n))`

基数完整参考列表：[Times子句](https://google.github.io/googletest/reference/mocking.html#EXPECT_CALL.Times)

#### 7.2.4 动作：被调用时的行为
**动作**(action)用于指定mock函数被调用时的行为，也叫做**打桩**(stubbing)。

如果没有指定，则mock函数被调用时会执行**默认动作**：
* `void`函数直接返回
* 返回类型是内置类型的函数返回对应类型的默认值：`bool`类型为`false`，数值类型为0，指针类型为`nullptr`
* 返回类型有默认构造函数的则返回默认构造的值

可以使用一个或多个`WillOnce()`子句以及一个可选的`WillRepeatedly()`子句指定动作。例如：

```cpp
using ::testing::Return;
EXPECT_CALL(turtle, GetX())
    .WillOnce(Return(100))
    .WillOnce(Return(200))
    .WillOnce(Return(300));
```

表示`turtle.GetX()`将被调用恰好三次（按照7.2.3节所述的规则推断），分别返回100、200、300，动作`Return()`表示返回指定的值。

```cpp
EXPECT_CALL(turtle, GetY())
    .WillOnce(Return(100))
    .WillOnce(Return(200))
    .WillRepeatedly(Return(300));
```

表示`turtle.GetY()`将被调用至少两次，前两次分别返回100和200，之后返回300。

注意：每次调用会“消耗”一个`WillOnce()`。如果显式指定了`Times()`且基数大于`WillOnce()`的个数，则`WillOnce()`被用完后mock函数将会执行`WillRepeatedly()`指定的动作，如果未指定`WillRepeatedly()`则执行默认动作。例如：

```cpp
EXPECT_CALL(turtle, GetY())
    .Times(4)
    .WillOnce(Return(100));
```

表示`turtle.GetY()`将被调用4次，第一次返回100，之后返回0（默认动作）。

动作完整参考列表：[Using Actions](https://google.github.io/googletest/gmock_cook_book.html#using-actions)

注：
* 在GoogleTest中，设置期望和打桩是同时执行的（即`EXPECT_CALL().WillOnce()`），而Mockito的打桩（`when().thenReturn()`）和验证调用（`verify()`）是分开执行的。
* 如果需要只定义行为、不设期望，则使用`ON_CALL()`，详见[gMock Cookbook - Knowing When to Expect](https://google.github.io/googletest/gmock_cook_book.html#UseOnCall)。

#### 7.2.5 设置多个期望
到目前为止的示例都只设置了一个期望，即`EXPECT_CALL()`。如果对同一个mock对象的同一个方法设置了多个期望，则当该mock方法被调用时，gMock会按照与定义**相反的顺序**搜索匹配的期望（可理解为“新规则覆盖旧规则”）。当匹配到一个期望时，对应的调用次数加1，如果超过了该期望的基数则会报错（而不是继续搜索）。

例如：
```cpp
EXPECT_CALL(turtle, Forward(_));  // #1
EXPECT_CALL(turtle, Forward(10))  // #2
    .Times(2);
```

如果`Forward(10)`被调用了三次，则第三次调用会导致错误，因为第二个期望的调用次数已经超过两次。如果第三次调用改为`Forward(20)`，则没有问题，因为匹配了第一个期望。

#### 7.2.6 有序/无序调用
期望默认是无序的，即调用不需要按照期望定义的顺序发生。如果需要限制调用顺序，可以使用`InSequence`。例如：

```cpp
using ::testing::InSequence;
...
TEST(PainterTest, DrawLine) {
    MockTurtle turtle;
    {
        InSequence seq;
        EXPECT_CALL(turtle, GoTo(0, 10));
        EXPECT_CALL(turtle, PenDown());
        EXPECT_CALL(turtle, GoTo(100, 150));
        EXPECT_CALL(turtle, PenUp());
    }

    Painter painter(&turtle);
    EXPECT_TRUE(painter.DrawLine(0, 10, 100, 150));
}
```

在`InSequence`对象作用域内的期望是有序的。因此该测试验证`painter.DrawLine(0, 10, 100, 150)`会依次调用`GoTo(0, 10)`、`PenDown()`、`GoTo(100, 150)`、`PenUp()`，如果顺序错误则测试失败。

#### 7.2.7 期望是有粘性的
7.2.5节的示例说明gMock中的期望默认是“**有粘性**”(sticky)的，即达到调用次数上限后仍然是有效的。

另一个示例如下：

```cpp
for (int i = n; i > 0; i--) {
    EXPECT_CALL(turtle, GetX())
        .WillOnce(Return(10 * i));
}
```

直观上会认为其含义是`turtle.GetX()`将会被调用n次，并依次返回10、20、30……。但实际上并不是这样——第二次调用`GetX()`仍然会匹配最后一个（最新的）期望，并导致调用次数达到上限的错误。问题在于所有期望的参数列表都相同，因此前四个期望不可能被匹配到。

解决方法是将期望设置为非粘性的，即达到最大调用次数后就立即失效（“饱和后退休”）：

```cpp
for (int i = n; i > 0; i--) {
    EXPECT_CALL(turtle, GetX())
        .WillOnce(Return(10 * i))
        .RetiresOnSaturation();
}
```

#### 7.2.8 无关调用
没有设置任何期望的mock函数被调用时，gMock会给出警告（不是错误），称之为**无关调用**(uninteresting calls)。使用其他类型的mock对象可以改变这一行为，见[The Nice, the Strict, and the Naggy](https://google.github.io/googletest/gmock_cook_book.html#NiceStrictNaggy)。

### 7.3 局限性
虽然gMock的功能很强大，但其本身在使用上就存在局限性：
* 编写mock类要求接口类中**被mock的函数必须是虚函数**。尽管gMock也支持mock非虚函数（见[Mocking Non-virtual Methods](https://google.github.io/googletest/gmock_cook_book.html#MockingNonVirtualMethods)），但需要对被测代码做较大的改动（改为模板）。
* **无法mock普通函数**，除非使用类对其封装。
* 为了能够在被测函数中使用mock对象，**必须通过接口类指针或引用参数将mock对象传递给被测函数**（这就是[依赖注入](https://en.wikipedia.org/wiki/Dependency_injection)的基本思想）。例如，为了能够在`Painter`中使用`MockTurtle`，成员`turtle_`的类型必须是`Turtle*`（或`Turtle&`），并通过构造函数参数传递进来，而不能直接声明一个`Turtle`类型的成员。
* 在真实代码中，需要mock的类可能并不是继承自一个像`Turtle`这样的抽象接口类，其成员函数也不是虚函数。这种情况下无论将其改为虚函数还是mock非虚函数，都需要对代码做较大的改动，因此这样的代码难以编写mock类。文档[Alternative to Mocking Concrete Classes](https://google.github.io/googletest/gmock_cook_book.html#alternative-to-mocking-concrete-classes)一节有对这一问题的讨论，给出的建议是**面向接口编程**，根据具体问题权衡利弊。
* 总之，要使得代码易于测试，必须**在设计时就考虑如何测试**。通过使用抽象接口类、拆分成多个小的函数等方式，编写测试和维护代码都会更加容易。

## 8.使用Blade构建
见[Blade构建工具]({% post_url 2022-01-20-blade-build-tool %}) 8.1节。

## 参考
* [GoogleTest - CMake module](https://cmake.org/cmake/help/latest/module/GoogleTest.html)
* [googletest README](https://github.com/google/googletest/blob/main/googletest/README.md)
* [Google Test - CLion](https://www.jetbrains.com/help/clion/2022.1/creating-google-test-run-debug-configuration-for-test.html)
* [gtest-cmake-example](https://github.com/kaizouman/gtest-cmake-example)
