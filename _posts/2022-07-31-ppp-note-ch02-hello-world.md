---
title: 《C++程序设计原理与实践》笔记 第2章 Hello, World!
date: 2022-07-31 23:36:21 +0800
categories: [C/C++, PPP]
tags: [cpp, hello world, comment]
---
## 2.1 程序
为了使计算机能够做某件事，你需要在繁琐的细节上明确告诉它怎么做。对“怎么做”的描述称为**程序**(program)，**编程**(programming)是书写和测试这个程序的行为。为了向计算机精确描述这些事情，我们需要一种由特定语法精确定义的语言，这种语言称为**编程语言**，C++是为各种编程任务而设计的编程语言。

## 2.2 经典的第一个程序
这是经典的第一个程序的一个版本，它在屏幕上输出 "Hello, World!" ：

[Hello world](https://github.com/ZZy979/PPP-code/blob/main/ch02/hello_world.cpp)

可以将这段文字看作是交给计算机执行的一组指令，就像交给一个厨师的一张菜谱。

`cout << "Hello, World!\n";` 这一行打印字符串 "Hello, World!" ，并紧跟一个换行。

在C++中，字符串常量是由双引号来分隔，`\n`是一个表示换行符的特殊字符。名称`cout`是**标准输出流**，使用运算符`<<`放入`cout`的字符将显示在屏幕上（类似于C语言的`stdout`）。名称`cout`的发音是 "see-out" ，它是 "character output stream" 的缩写。

`// output "Hello, World!"` 是**注释**(comment)。在一行中的`//`之后的内容都是注释。注释将会被编译器忽略，但对人们读懂代码很有帮助。**人类是代码的主要读者。** 当你在一个星期或者一年后回过头来阅读代码，并且已经忘记为什么这样编写代码时，你最有可能通过注释得到帮助。因此，做好你的程序的文档工作。

`#include <iostream>` 是一个`#include`指令，指示计算机从名为iostream的文件中“包含”功能。（书中代码通过一个[std_lib_facilities.h](https://www.stroustrup.com/Programming/std_lib_facilities.h)头文件隐藏了使用C++标准库的细节，这里不使用）该文件使得程序可以使用标准C++ I/O流功能，在这里只使用标准输出流`cout`和它的输出运算符`<<`。使用`#include`包含的文件通常有.h后缀（C++标准库没有），称为**头文件**(header file)。头文件包含名字（类、函数、变量等）的定义，例如在程序中使用的`cout`。

每个C++程序必须有一个称为`main`的函数，从该函数开始执行。**函数**(function)是一个指令序列，计算机会按照顺序来执行。一个函数包括四部分：
* **返回值类型**，指定返回结果的类型，在这里是`int`（表示“整数”）
* **名字**，在这里是`main`
* **参数列表**，在一对圆括号中，在这里参数列表是空的
* **函数体**，在一对花括号中，列出了这个函数将要执行的动作（称为**语句**）

下面是最简单的C++程序，这个程序没有做任何事情：

```cpp
int main() {}
```

"Hello, World!" 程序的`main`函数有两条语句，首先在屏幕上打印 "Hello, World!" ，然后返回一个值0给它的调用者。由于`main`是由“系统”来调用的，因此我们不会使用返回值。但是在UNIX/Linux系统中，返回值可以用于检查程序是否成功。`main`返回0表示程序成功终止。

在C++程序中用于指定一个行为的部分称为**语句**(statement)（`#include`等预处理器指令除外），语句以分号结尾。

## 2.3 编译
C++是一种编译语言。要想使一个程序可以运行，必须将人类可读的**源代码**（程序文本）转换为计算机可以理解的**可执行代码**、**目标代码**或**机器代码**，这个过程称为**编译**(compilation)，由一个称为**编译器**(compiler)的程序来做。

![编译过程](/assets/images/ppp-note-ch02-hello-world/编译过程.png)

典型的C++源代码文件的扩展名为.cpp（例如hello_world.cpp，也可以是.cc、.cxx等），头文件的扩展名为.h或没有（例如std_lib_facilities.h、iostream），目标文件的扩展名为.obj（在Windows中）或.o（在UNIX中），可执行文件的扩展名为.exe（在Windows中）、.out或没有（在UNIX中）。

编译器会阅读源代码并检查语法错误。常见错误包括：缺少头文件、文件名拼写错误、引号不配对、关键字错误、运算符错误、单引号/双引号错误、忘记加分号。

编译器可能是你在编程时最好的朋友。

## 2.4 链接
程序通常由几个单独的部分组成，它们经常由不同的人来开发。例如， "Hello, World!" 程序包含我们编写的部分和C++标准库。这些单独的部分（有时称为**翻译单元**）必须被编译为目标代码，目标代码必须被链接起来以形成一个可执行程序。用于将这些部分链接起来的程序称为**链接器**(linker)。

![编译和链接过程](/assets/images/ppp-note-ch02-hello-world/编译和链接过程.png)

请注意**目标代码和可执行程序是不能在系统间移植的**。例如，在Windows上编译得到的可执行程序无法在Linux机器上运行。

**库**(library)是一些代码的集合，它们通常是由其他人编写的，我们用`#include`文件中的声明来访问这些代码。

由编译器发现的错误称为**编译时错误**(compile-time errors)（例如语法错误），由链接器发现的错误称为**链接时错误**(link-time errors)（例如函数未定义），直到程序运行时才发现的错误称为**运行时错误**(run-time errors)或**逻辑错误**(logic errors)（例如除以零、数组下标越界）。通常来说，编译时错误比链接时错误更容易理解和修正，链接时错误比运行时错误更容易发现和修正。

## 2.5 编程环境
如果使用命令行窗口，则需要自己编写编译和链接命令。如果使用IDE（继承开发环境），则只需点击运行按钮，IDE会自动完成编译和链接过程。

### Windows
对于Windows系统，可以使用[Code::Blocks](http://www.codeblocks.org/)、[Visual Studio](https://visualstudio.microsoft.com/zh-hans/vs/)或[CLion](https://www.jetbrains.com/clion/)等IDE，创建一个C++控制台工程，将源代码拷贝到一个源文件，并点击运行按钮，即可看到运行结果。由于IDE一般自带了编译器（例如Visual Studio自带的编译器是[MSVC](https://docs.microsoft.com/zh-cn/cpp/build/reference/compiling-a-c-cpp-program)），并自动调用编译器和链接器来生成可执行文件，因此不需要关心这些细节。

![在Visual Studio中运行](/assets/images/ppp-note-ch02-hello-world/在Visual Studio中运行.png)

### Linux
对于Linux系统，一般自带了[GCC](https://www.gnu.org/software/gcc/)或[Clang](https://clang.llvm.org/)编译器。在命令行中将源文件编译成可执行文件只需执行一行命令：

```bash
$ g++ -o hello_world hello_world.cpp
```

该命令使用GCC编译器(g++)将源文件hello_world.cpp编译成可执行文件hello_world ，之后执行hello_world即可：

```bash
$ ./hello_world 
Hello, World!
```

![在命令行执行](/assets/images/ppp-note-ch02-hello-world/在命令行执行.png)

这里的编译命令实际上一次性完成了编译和链接两个步骤，也可以分开执行：

```bash
$ g++ -c -o hello_world.o hello_world.cpp  # 编译
$ g++ -o hello_world hello_world.o  # 链接
```

详见：[GCC编译器的使用方法]({% post_url 2022-02-03-gcc-compiler-usage %})

## 习题
[2-1](https://github.com/ZZy979/PPP-code/blob/main/ch02/exec2-1.cpp)
