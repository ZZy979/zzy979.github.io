---
title: 《C++程序设计原理与实践》笔记 第5章 错误
date: 2022-09-25 00:44:25 +0800
categories: [C/C++, PPP]
tags: [cpp, error handling, exception, try statement, undefined behavior]
---
本章将讨论程序的正确性、错误和错误处理。

## 5.1 引言
在编写程序时，错误是不可避免的。而最后的程序必须是没有错误的，至少不存在不可接受的错误。

错误的分类有很多种，例如：
* **编译时错误**(compile-time errors)：由编译器发现的错误，例如语法错误、类型错误
* **链接时错误**(link-time errors)：链接器将对象文件链接为可执行程序时发现的错误
* **运行时错误**(run-time errors)：程序运行中发现的错误，可进一步分为
    * 由计算机（硬件或操作系统）检测出的错误
    * 由库（例如标准库）检测出的错误
    * 由用户代码检测出的错误
* **逻辑错误**(logic errors)：程序员在寻找错误结果的原因时发现的错误

除非特别说明，我们会假定你的程序：
* **对于所有合法输入应输入正确结果**
* **对于所有非法输入应输出错误信息**
* 不需要关心硬件故障
* 不需要关心系统软件故障
* 发现错误后允许程序终止

有以下三种方法来编写可接受的软件：
* 精心组织软件结构以减少错误
* 通过调试和测试消除大部分错误
* 确定剩下的错误是不重要的

上述任何一种方法都不能保证完全消除错误，我们必须同时使用上述三种方法。

## 5.2 错误的来源
* 不够明确(poor specification)：如果不具体明确程序应该做什么，就不可能充分检查所有“死角”，并确认所有可能的情况都被正确处理（即对于任意输入都能给出正确结果或者充分的错误信息）。
* 不完备的程序(incomplete programs)：在开发过程中，显然会有一些没有考虑到的情况，这是不可避免的。我们必须要达到的目标是知道何时能够处理所有情况。
* 意外的参数(unexpected arguments)：如果给函数传递了一个不能处理的参数，就会遇到问题，例如`sqrt(-1.2)`。5.5.3节将讨论这类问题。
* 意外的输入(unexpected input)：程序通常都会读取数据（来自键盘、文件、GUI、网络连接等）。程序会对输入做很多假设，例如用户会输入一个数字，如果用户输入的不是数字会怎样呢？5.6.3和10.6节将讨论这类问题。
* 意外的状态(unexpected state)：大部分程序都会保留很多数据（“状态”）以供系统的不同部分使用，例如4.6.3节读取温度程序中的`vector`。如果这些数据是不完整的或者错误的，程序的各个部分仍然应该正常运转。26.3.5节将讨论这类问题。
* 逻辑错误(logical errors)：即程序没有按照期望的方式运行，我们必须查找并修正这些问题。6.6和6.9节将给出这类问题的例子。

## 5.3 编译时错误
在编写程序时，编译器是检查错误的第一道防线。编译器发现的大部分错误都是低级错误。例如，考虑下面这个简单函数的一些调用：

```cpp
int area(int length, int width);  // calculate area of a rectangle
```

### 5.3.1 语法错误
如果按照以下方式调用`area()`：

```cpp
int s1 = area(7, 4;    // error: ) missing
int s2 = area(7, 4)    // error: ; missing
Int s3 = area(7, 4);   // error: Int is not a type
int s4 = area('7, 4);  // error: non-terminated character (' missing)
```

上面每一行都有一个语法错误。即使是一个小错误，编译器也会报告很多繁杂信息，甚至会指向程序中的其他行。**因此，如果你在编译器所指向的行中没有发现错误的话，还应该检查一下前几行是否有错误。**

对于同样的代码，不同编译器可能会给出不同的错误信息。例如对于`s3`的声明，Visual Studio使用的MSVC编译器报错如下（与书中给出的错误信息比较接近，也比较难以理解）：

![MSVC编译器错误信息](/assets/images/ppp-note-ch05-errors/MSVC编译器错误信息.png)

g++编译器报错如下（相比MSVC的错误信息更加“智能”，编译器甚至猜到是将`int`错误拼写为`Int`）：

![g++编译器错误信息](/assets/images/ppp-note-ch05-errors/g++编译器错误信息.png)

实际上，（MSVC编译器给出的）这些令人费解的信息可以解释为“在s3前有一个语法错误，需要检查一下Int或s3的类型”，这样理解就不难发现问题了。

注：如果使用IDE写代码，在编译之前IDE就能发现一些错误。例如Visual Studio的提示为

![Visual Studio编辑器提示](/assets/images/ppp-note-ch05-errors/Visual Studio编辑器提示.png)

CLion的提示为

![CLion编辑器提示](/assets/images/ppp-note-ch05-errors/CLion编辑器提示.png)

注：

**未定义行为**(undefined behavior)是指代码不符合C++语言规范，导致程序的行为不可预测的情况（可能会导致程序崩溃、产生错误结果或者随机行为等）。未定义行为包括访问未初始化的变量（3.9节）、数组下标越界（5.6.2节）、有符号整数溢出（[Issue #13](https://github.com/ZZy979/PPP-code/issues/13)）、空指针解引用（18.6.4节）、在同一个表达式中多次修改同一个变量的值（8.6.1节）等，详见[cppreference - Undefined behavior](https://en.cppreference.com/w/cpp/language/ub)。未定义行为在不同的编译器、操作系统和硬件平台上表现可能不同，因此程序员应该避免使用未定义行为的代码，以确保程序的正确性和可移植性。

**C++标准仅提供语言规范，并不规定如何实现。** 主流的C++编译器包括：
* [GCC](https://www.gnu.org/software/gcc/)(GNU Compiler Collection)：开源编译器，编译器及标准库(libstdc++)源代码[gcc-mirror/gcc](https://github.com/gcc-mirror/gcc)
* [Clang](https://clang.llvm.org/)：基于LLVM开发的编译器，编译器及标准库(libc++)源代码[llvm/llvm-project](https://github.com/llvm/llvm-project)
* [MSVC](https://learn.microsoft.com/en-us/cpp/build/reference/compiling-a-c-cpp-program)(Microsoft Visual C++)：微软开发的编译器，编译器不开源，标准库源代码[microsoft/STL](https://github.com/microsoft/STL)

因此对于未定义行为，不同编译器、甚至同一个编译器的不同优化模式可能产生不同的结果。另外，不同编译器对C++标准特性的支持程度也不同，详见[C++ compiler support](https://en.cppreference.com/w/cpp/compiler_support)。

### 5.3.2 类型错误
一旦排除了语法错误，编译器就会开始检查类型错误：即声明的变量、函数等的类型，以及赋值给变量、传递给函数参数的值或表达式的类型之间不匹配。例如：

```cpp
int x0 = arena(7);         // error: undeclared function
int x1 = area(7);          // error: wrong number of arguments
int x2 = area("seven",2);  // error: 1st argument has a wrong type
```

* 对于`arena(7)`，我们将`area`错写为`arena`，因此编译器会认为是调用函数`arena`。如果没有名为`arena`的函数，将会得到未定义函数的错误信息；如果确实有叫做`arena`的函数，并且该函数接受`7`作为参数，这将是一个更坏的情况：程序将会正确编译，但不会按照预期的方式运行（这是一个逻辑错误）。
* 对于`area(7)`，编译器将检测到参数个数错误。在C++中，函数调用必须提供正确的参数个数、类型和顺序。
* 对于`area("seven",2)`，第一个参数声明为`int`类型，但提供了`string`，编译器不会识别出 "seven" 表示的是数字7。

### 5.3.3 警告
你可能会希望编译器报告的一些错误并不是真正的错误，但是当你有了一定的编程经验后，你会希望编译器能够拒绝更多代码，而不是更少。考虑下面的例子：

```cpp
int x4 = area(10,-7);      // OK: but what is a rectangle with a width of minus 7?
int x5 = area(10.7,9.3);   // OK: but calls area(10,9)
char x6 = area(100,9999);  // OK: but truncates the result
```

* 对于`x4`，我们没有从编译器得到错误信息。对于编译器来说，`area(10,-7)`是正确的：`area()`需要两个整数，也提供了两个整数。没有人规定这些参数必须是正数。
* 对于`x5`，好的编译器应该给出警告信息：`double`型参数10.7和9.3将被截断为`int`型参数10和9。C++允许从`double`到`int`的隐式转换，因此编译器不会拒绝该函数调用。
* 对于`x6`，`int`类型的返回值999900被赋给一个`char`变量，`x6`最有可能的结果是截断后的值-36。同样，好的编译器应该给出警告信息。

## 5.4 链接时错误
一个程序一般由几个独立编译的部分组成（例如一个.cpp源文件或.h头文件），称为**翻译单元**(translation unit)。程序中的**每个函数在所有被使用的翻译单元中的声明类型必须严格一致**，我们使用头文件来保证这一点，详见8.3节。并且，**每个函数只能定义一次**。如果违反任意一条规则，链接器将报错。例如：

```cpp
// main.cpp
int area(int length, int width);
int main() {
    int x = area(2, 3);
}
```

如果直接将该源文件编译并链接为可执行文件，链接器将报错找不到`area()`的定义：

```bash
$ g++ -o main main.cpp 
/tmp/ccXjlgA6.o: in function `main':
main.cpp:(.text+0x13): undefined reference to `area(int, int)'
collect2: error: ld returned 1 exit status
```

注：准确来说是链接过程报错，而不是编译过程，因为编译过程只关心函数调用与声明是否一致，不需要函数定义。但实际上编译器会自动调用链接器，因此看起来错误信息是由编译器报告的。

参考：[GCC编译器的使用方法]({% post_url 2022-02-03-gcc-compiler-usage %})

`area()`的定义与调用必须具有严格相同的类型（包括返回值类型和参数类型）：

```cpp
int area(int x, int y) { /* ... */ }             // “our” area()
double area(double x, double y) { /* ... */ }    // not “our” area()
int area(int x, int y, char unit) { /* ... */ }  // not “our” area()
```

函数的链接规则同样适用于程序的其他实体，例如变量和类型：**同一名字的实体只能有一个定义，但可以有多个声明，并且所有声明的类型必须相同**。

## 5.5 运行时错误
如果程序没有编译错误和链接，那么它就能运行了。但是对于程序运行时发现的错误，可能很难确定如何处理它。例如：

```cpp
// calculate area of a rectangle
int area(int length, int width)  {
    return length * width;
}

// calculate area within frame
int framed_area(int x, int y) {
    return area(x - 2, y - 2);
}

int main() {
    int x = -1;
    int y = 2;
    int z = 4;
    // ...
    int area1 = area(x, y);
    int area2 = framed_area(1, z);
    int area3 = framed_area(y, z);
    double ratio = double(area1) / area3;
}
```

上面程序中存在的问题：
* 面积`area1`和`area2`为负数，违背了数学规律
* 在`ratio`的计算中，`area3`的值是0，导致除以0，这类错误就是运行时错误（经测试，在浮点数除法中除以0的结果是+inf或-inf，并不是像书中所说的硬件错误）

函数参数错误问题有两种解决办法：
* 让调用者来处理错误的参数
* 让被调用者（函数本身）来处理错误的参数

### 5.5.1 调用者处理错误
如果`area()`是一个由我们不能修改的库提供的函数，则必须选择这种方法：

```cpp
if (x <= 0 || y <= 0)
    error("non-positive area() argument");
int area1 = area(x, y);
```

注：`error()`是作者在std_lib_facilities.h中提供的函数，其作用是输出错误信息并终止程序运行（抛出异常）：

```cpp
inline void error(const string& message) {
    cerr << message << endl;
    throw runtime_error(message);
}
```

对于`framed_area()`，首先要将魔数2改为命名常量（否则当这个值被修改时，必须查找程序中所有该函数的调用，并修改错误处理代码）：

```cpp
const int frame_width = 2;
// calculate area within frame
int framed_area(int x, int y) {
    return area(x - frame_width, y - frame_width);
}
```

增加错误处理后的代码如下：

```cpp
if (1 - frame_width <= 0 || z - frame_width <= 0)
    error("non-positive argument for area() called by framed_area()");
int area2 = framed_area(1, z);
if (y - frame_width <= 0 || z - frame_width <= 0)
    error("non-positive argument for area() called by framed_area()");
int area3 = framed_area(y, z);
```

这段代码变得很丑陋（因此容易出错）！代码长度变为了三倍，并且暴露了`framed_area()`函数的实现细节。

### 5.5.2 被调用者处理错误
在`framed_area()`内部检查参数错误非常简单：

```cpp
// calculate area within frame
int framed_area(int x, int y) {
    const int frame_width = 2;
    if (x - frame_width <= 0 || y - frame_width <= 0)
        error("non-positive area() argument called by framed_area()");
    return area(x - frame_width, y - frame_width);
}
```

函数检查自己的参数的好处在于**参数检查代码只在一个地方**，不需要在整个程序中查找调用点。

将这一方法应用到`area()`：

```cpp
// calculate area of a rectangle
int area(int length, int width) {
    if (length <= 0 || width <= 0)
        error("non-positive area() argument");
    return length * width;
}
```

这将捕获对`area()`的调用中的所有错误，因此不再需要在`framed_area()`中进行检查。

因此，**在函数中检查参数，除非你有充分的理由不这样做**。

### 5.5.3 报告错误
考虑另外一个问题：当检查参数发现错误时应该做什么？有时可以**返回一个“错误值”**，例如：

```cpp
// ask user for a yes-or-no answer;
// return 'b' to indicate a bad answer (i.e., not yes or no)
char ask_user(string question) {
    cout << question << "?(yes or no)\n";
    string answer = " ";
    cin >> answer;
    if (answer == "y" || answer == "yes") return 'y';
    if (answer == "n" || answer == "no") return 'n';
    return 'b';     //'b' for "bad answer"
}
```

```cpp
// calculate the area of a rectangle
// return -1 to indicate a bad argument
int area(int length, int width) {
    if (length <= 0 || width <= 0) return -1;
    return length * width;
}
```

如上所述，可以让被调函数进行参数检查，让调用者按需处理错误。这种方式看上去是可行的，但存在几个问题，使得它在很多情况下不能使用：
* 被调函数和调用者都要进行检查
* 调用者可能会忘记检查，这可能导致程序不可预测的行为
* 很多函数并没有可以用于表示错误的“额外”返回值，例如从输入读取整数的函数（例如`cin`的运算符`>>`）返回值可以是任意整数，因此不能返回某个整数来表示读取失败

还有另外一种解决这一问题的方法：异常。

## 5.6 异常
C++提供了一种错误处理机制：**异常**(exception)。其基本思想是将错误检测（应该在被调函数中完成）和错误处理（应该在主调函数中完成）分离，同时保证检测到的错误不能被忽略。

异常的基本用法：如果一个函数发现了自己不能处理的错误，它不是正常返回，而是使用`throw`语句**抛出异常**来表示错误的发生；任何一个直接或间接调用者都可以使用`try-catch`语句**捕获异常**并指定如何处理（如果一个调用者没有捕获，则异常继续向上传播，直到`main`函数）；如果异常没有被任何调用者捕获，则程序终止运行。

### 5.6.1 参数错误
下面是使用异常的`area()`：

[计算面积](https://github.com/ZZy979/PPP-code/blob/main/ch05/area.cpp)

即如果参数正确则返回面积，否则抛出异常并希望这个异常能够被捕获并适当处理。`Bad_area`是一个自定义的新类型，其唯一的目的是标识从`area()`中抛出的异常，`Bad_area()`表示“使用默认值创建一个`Bad_area()`类型的对象”。

注：同5.5.2节，这里的`area()`函数处理并报告了错误，因此`framed_area()`不需要再做错误处理

现在调用代码可以改为：

```cpp
int main() {
    try {
        int x = -1;
        int y = 2;
        int z = 4;
        // ...
        int area1 = area(x, y);
        int area2 = framed_area(1, z);
        int area3 = framed_area(y, z);
        double ratio = area1 / area3;
    }
    catch (Bad_area) {
        cout << "Oops! bad arguments to area()\n";
    }
}
```

首先，上面的错误处理是针对所有对`area()`的调用，包括`main()`中的直接调用和两个通过`framed_area()`的间接调用。其次，注意处理错误与检测错误是如何分离的：`main()`不知道哪个函数抛出了`Bad_area`异常，`area()`不知道哪个函数（如果有）会捕获它抛出的`Bad_area`异常。

注：`try-catch`语句的语法如下

```cpp
try {
    函数调用
}
catch (异常1) {
    错误处理1
}
catch (异常2) {
    错误处理2
}
catch (...) {
    其他错误处理
}
```

注意`try`块和`catch`块的大括号均不能省略。

标准库头文件\<exception\>和\<stdexcept\>中定义了一组标准异常类型，其中`exception`是所有标准异常的基类，`invalid_argument`是专门表示参数错误的异常类（`area()`可以也使用该异常）。但C++并未要求抛出的异常必须是`exception`的子类，而可以是任何类型（甚至可以抛出一个`int`，但这并没有什么意义）。详见习题[14-17](https://github.com/ZZy979/PPP-code/blob/main/ch14/standard_exception_class_hierarchy.png)。

### 5.6.2 范围错误
在C++中，通常把“数据集合(collections of data)”称为**容器**(container)，最常用的标准库容器是`vector`。大小为n的向量的元素下标范围是[0, n)，访问这个范围之外的下标将导致**范围错误**(range error)，**这类错误是非常普遍的**，例如：

```cpp
vector<int> v;  // a vector of ints
int x;
while (cin >> x) v.push_back(i);  // get values
for (int i = 0; i <= v.size(); ++i)  // print values
    cout << "v[" << i << "] == " << v[i] << '\n';
```

这里`for`循环的终止条件应该是`i < v.size()`，但写成了`i <= v.size()`，因此最后一次循环将导致下标越界——这种错误完全是由我们自身原因导致的。

注：基于范围的`for`循环不会有这个问题，但无法获得元素的索引。

下面是具有相同错误的更简单版本：

```cpp
vector<int> v(5);
int x = v[5];
```

`vector`的下标运算符`[]`不做范围检查，访问越界下标将导致未定义行为；成员函数`at()`与下标运算符的功能相同，但会做范围检查，如果下标越界则抛出`out_of_range`异常。增加异常处理后的代码如下：

```cpp
int main() {
    try {
        vector<int> v;  // a vector of ints
        int x;
        while (cin >> x) v.push_back(x);  // get values
        for (int i = 0; i <= v.size(); ++i)  // print values
            cout << "v[" << i << "] == " << v.at(i) << endl;
    }
    catch (out_of_range) {
        cerr << "Oops! Range error\n";
        return 1;
    }
    catch (...) {  // catch all other exceptions
        cerr << "Exception: something went wrong\n";
        return 2;
    }
    return 0;
}
```

范围错误是参数错误的一个特例。向量的`at()`函数通过抛出异常来报告错误，但它不知道我们要如何处理，`vector`的作者甚至不知道他的代码会在什么程序中被使用。

### 5.6.3 输入错误
处理输入错误的细节在10.6节讨论，这里只展示如何判断输入操作是否成功。考虑读取一个浮点数：

```cpp
double d = 0;
cin >> d;
```

通过测试`cin`来判断最后一个输入操作是否成功：

```cpp
double some_function() {
    double d = 0;
    cin >> d;
    if (!cin) error("couldn't read a double in 'some_function()'");
    // do something useful
}
```

这里将`cin`当作一个布尔值判断，如果输入操作成功则结果为`true`，否则为`false`，`!`是逻辑非运算符。

导致输入操作失败可能的原因有输入的不是一个浮点数、到达输入结尾(EOF)等。`error()`函数的定义在5.5.1节，即输出错误信息并抛出标准异常`runtime_error`。在`main()`函数中可以捕获该异常：

```cpp
int main() {
    try {
        double d = some_function();
        // out program
        return 0;  // 0 indicates success
    }
    catch (runtime_error& e) {
        cerr << "runtime error: " << e.what() << '\n';
        return 1;  // 1 indicates failure
    }
}
```

其中`e.what()`返回异常对象中的错误信息，`&`表示“传引用”，将在8.5.5节介绍。`cerr`是**标准错误流**，与`cout`用法相同，但它是专门用于错误输出的。默认情况下，`cerr`和`cout`都输出到屏幕，可以在命令行中通过重定向将其输出到文件，例如：

```bash
prog > output.txt 2> error.txt
```

将程序prog的标准输出重定向到文件output.txt，标准错误重定向到文件error.txt。

`main()`的返回值传递给调用程序的“系统”（例如Shell命令行），返回值为0表示成功，非0表示发生了某些错误。

如果异常没有被捕获，你会得到一个系统错误（“未捕获的异常”），程序将终止运行：

```
libc++abi: terminating with uncaught exception of type std::runtime_error: couldn't read a double in 'some_function()'
Abort trap: 6
```

### 5.6.4 截断错误
3.9.2节展示了截断错误：如果给一个变量赋一个“太大”的值，这个值将被隐式截断。当时给出的建议是通过检查来避免截断错误。作者在std_lib_facilities.h中提供了一个模板函数`narrow_cast()`，如果赋值或初始化操作会引起值的改变则抛出`runtime_error`异常（单词cast的意思是“类型转换”(type conversion)）。例如：

```cpp
int x1 = narrow_cast<int>(2.9);     // throws
int x2 = narrow_cast<int>(2.0);     // OK
char c1 = narrow_cast<char>(1066);  // throws
char c2 = narrow_cast<char>(85);    // OK
```

其中尖括号中的是**模板参数**，表示转换的目标类型。该函数的基本思想是先将`A`类型的值转换为`R`类型，再转换回`A`类型，看二者是否相等：

```cpp
// run-time checked narrowing cast (type conversion).
template<class R, class A> R narrow_cast(const A& a) {
	R r = R(a);
	if (A(r) != a) error("info loss");
	return r;
}
```

## 5.7 逻辑错误
在解决了编译错误和链接错误后，程序就能运行了，但程序的输出有可能是错误的，因为程序存在逻辑错误。逻辑错误通常是最难发现和排除的，因为在这种情况下计算机所做的正是你让它做的事情，你需要做的是找出那为什么没有反应你的真实意愿。

考虑下面的例子，在一组数据中找出最低、最高和平均温度：

[统计温度(bug)](https://github.com/ZZy979/PPP-code/blob/main/ch05/temperature_stats_buggy.cpp)

输入[得克萨斯州拉伯克2004年2月16日的小时温度](https://github.com/ZZy979/PPP-code/blob/main/ch05/testdata/Lubbock_temperatures_20040216.txt)（华氏度），输出的是正确结果：

```
High temperature: 42.6
Low temperature: -26.1
Average temperature: 9.29583
```

但这并不能证明程序是没有错误的。用另外一组[2004年7月23日](https://github.com/ZZy979/PPP-code/blob/main/ch05/testdata/Lubbock_temperatures_20040723.txt)的数据再次测试，会发现输出了错误的结果：

```
High temperature: 112.4
Low temperature: 0
Average temperature: 89.2083
```

最低温度应该是70.1而不是0。

错误的原因在于`low_temp`的初值是0，因此除非有一个温度低于0，否则它将一直是0。另外，`high_temp`的初始化存在同样的问题：除非有一个温度高于0，否则`high_temp`将一直是0。

注：第一组数据没有测试出这两个问题，是因为其中的温度有正有负。而第二组数据都是正值，`low_temp`的问题就显示出来了。同样，对于全是负值的数据，`high_temp`将会出现问题。

这类错误是相当典型的：程序编译时不会出错，对于“合理的”输入也不会出错，但是我们忘记了考虑什么是“合理的”。改进的程序如下：

[试一试](https://github.com/ZZy979/PPP-code/blob/main/ch05/temperature_stats_fixed.cpp)

注：
* 如果将`high_temp`和`low_temp`的初值分别设为-1000和1000，则对于这个范围内的数据不会有问题，超过这个范围的数据仍然有问题（例如数据都小于-1000）。这两个初值决定了程序的适用范围（例如，对于地球表面的温度数据，[-1000, 1000]的范围是足够的，而对于太阳表面的温度就不够了）。这是所有求最小值、最大值的程序都存在的问题。
* 更一般的做法是使用`double`类型的最大值：标准库头文件\<cfloat\>中的`DBL_MAX`常量（约为1.79769×10<sup>308</sup>），分别使用`-DBL_MAX`和`DBL_MAX`作为`high_temp`和`low_temp`的初值，就可以处理`double`范围内的所有数据。
* 另一种方法是使用输入的第一个值来初始化`high_temp`和`low_temp`，这种方法不存在数据范围的问题，但对于没有输入数据的情况需要单独处理，代码会变得更加复杂。

## 5.8 估计
除非知道正确结果大概是什么，否则我们不能判断得到的结果是否合理。要知道大概的正确结果，就需要**估计**(estimation)。“正确答案”是编写的程序要告诉我们的，但只有确定了结果的合理性之后才能做下一步工作。

例如，编写了一个计算正六边形面积的程序，输入边长为2，输出-34.56，可以确定是错误的，因为面积不可能为负。修改后得到21.65685，很难立刻判断是否正确，因此需要估计一下这个结果是否合理——在纸上画一个边长为2的六边形，目测它接近一个3×3的正方形，该正方形的面积是9，因此21.65685不可能是对的。继续修改程序，结果为10.3923，这一次可能是正确的了。

## 5.9 调试
复杂的程序几乎不可能第一次就正确运行，因此当你写完代码后，必须找到并排除错误，这一过程称为**调试**(debugging)，错误称为**bug**（“bug”一词来源于真空管计算机时代昆虫进入电路板导致硬件故障）。

每当程序不能正常工作，就要找出问题并修正。调试是编程中最乏味、最费时间的工作，因此应该做好设计和编码工作，以使找bug的时间降到最低。

调试中的关键问题是：**我如何知道程序是否真正运行正确？** 如果不能回答这个问题，你将会陷入长时间、乏味的调试工作，而且你的用户也很可能陷入麻烦。

### 5.9.1 实用调试建议
首先要决定如何报告错误，使用`error()`并在`main()`中捕获异常是本书采用的方法。

**提高程序的易读性**，从而有更多机会找到bug：
* 为代码做好注释。代码本身能表达清楚的，不要用注释。注释应该使用尽量清楚、简洁的语言说清以下内容：
    * 程序的名称
    * 程序的目的
    * 谁在什么时候写了这个代码
    * 版本号
    * 复杂代码片段的目的是什么
    * 总体设计思想是什么
    * 源代码是如何组织的
    * 输入数据的前提假设是什么
    * 还缺少哪一部分代码，程序还不能处理哪些情况
* 使用有意义的名字
* 使用一致的代码层次结构
* 将代码分解成许多小函数，每个函数表达一个逻辑功能
* 尽量避免使用复杂的语句
    * 复杂代码是最容易隐藏bug的地方
* 在可能的情况下，使用库提供的功能，而不是你自己的代码

编译错误依靠编译器和IDE即可解决，最难的部分是逻辑错误，需要通过调试解决。

通常，当你没有找到问题时，原因是你只看到了自己希望看到的，而不是你所写的，例如：

```cpp
for (int i = 0; i<=max; ++j) {   // oops! (twice)
    for (int i=0; 0<max; ++i);   // print the elements of v
        cout << "v[" << i << "]==" << v[i] << '\n';
    // ...
}
```

通常，如果你没有找到问题所在，原因可能是执行的代码太多。大多数IDE都提供**单步调试**功能，对于查找问题很有帮助。对于简单的程序（或者无法使用IDE调试功能的情况），可以通过在程序中**临时插入一些输出语句**来帮助你了解程序运行情况（查看变量的值、函数返回结果等）。

在可能会隐藏bug的语句中，加入检查**不变式**(invariant)（即永远成立的条件，见9.4.3节）的语句，例如：

```cpp
// the arguments are positive and a < b < c
int my_complicated_function(int a, int b, int c) {
    if (!(0<a && a<b && b<c))  // ! means “not” and && means “and”
        error("bad arguments for my_complicated_function()");
    // ...
}
```

如果还是没有效果，在看起来不会有错的代码段中插入检查不变式的语句。如果还是找不到bug，几乎可以肯定是你找错地方了。

陈述不变式的语句称为**断言**(assertion)。

目前还没有一种最好的调试方法，但有一件事要始终牢记：**混乱的代码总是容易隐藏bug**。尽可能保持代码简洁、有逻辑性、格式良好，这将大大减少调试时间。

## 5.10 前置条件和后置条件
### 5.10.1 前置条件
函数对于它的参数的要求称为**前置条件**(pre-condition)：如果函数要正确执行，这一条件必须为真。例如上一节中`my_complicated_function()`函数的前置条件为`0 < a < b < c`。

建议**前置条件一定要在注释中说明**（这样调用者可以知道函数的要求），但不能保证调用者一定能满足前置条件。为了遵循“被调用者进行参数检查”这一原则，应该**让函数检查自己的前置条件**，除非有足够的理由不这么做。编写前置条件注释和检查代码有助于避免设计错误并及早发现使用错误。

### 5.10.2 后置条件
将前置条件的思想应用到返回值就是**后置条件**(post-condition)。如果函数返回一个值，总是要最返回值做出某种承诺。考虑面积函数`area()`，可以明确表达出前置条件和后置条件：

```cpp
// calculate the area of a rectangle
// pre-conditions: length and width are positive
// post-condition: return a positive value that is the area
int area(int length, int width) {
    if (length <= 0 || width <= 0) error("area() pre-condition");
    int a = length * width;
    if (a <= 0) error("area() post-condition");
    return a;
}
```

注：在数学上，该函数的前置条件成立时后置条件一定成立，但计算机表示的数据还有范围限制：`int`类型的最大值是2<sup>31</sup>-1=2147483647，当结果超过这个值时就会发生溢出，例如`area(50000, 50000)`（结果是-1794967296，不同编译器可能产生不同结果）。

## 5.11 测试
如何知道已经找到最后一个bug？答案是没有办法。“最后一个bug”是程序员之间的笑话：这是不存在的。对于大程序来说，我们永远不会找到“最后一个bug”，因为在查找bug的同时还要忙于按照新需求修改程序。

除了调试，还需要一种系统地查找错误的方法，称为**测试**(testing)。基本上，测试是以一个较大的、经过系统选择的数据集作为输入来执行程序，并将结果与期望值进行比较。使用一组给定输入的一次运行称为一个**测试用例**(test case)。

测试需要系统地选择测试用例，包括正确和不正确的输入数据。7.3节会给出示例。

注：真实程序可能会需要成千上万个测试用例，不可能手工一个一个输入，需要使用自动化测试框架（例如GoogleTest等）

## 习题
* [5-2~5-6](https://github.com/ZZy979/PPP-code/blob/main/ch05/temperature_converters.cpp)
* [5-7](https://github.com/ZZy979/PPP-code/blob/main/ch05/solve_equation.cpp)
* [5-8](https://github.com/ZZy979/PPP-code/blob/main/ch05/sum_n_integers.cpp)
* [5-11](https://github.com/ZZy979/PPP-code/blob/main/ch05/fibonacci_series.cpp)
* [5-12~5-13](https://github.com/ZZy979/PPP-code/blob/main/ch05/bulls_and_cows.cpp)
