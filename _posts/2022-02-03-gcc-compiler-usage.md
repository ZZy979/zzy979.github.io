---
title: GCC编译器的使用方法
date: 2022-02-03 00:45:39 +0800
categories: [C/C++]
tags: [cpp, gcc, compiler]
---
## 1.编译流程
GCC编译器在编译C代码时需要经过以下4个步骤：
* 预处理(preprocessing)：对.c源文件进行预处理（头文件展开和宏替换），生成.i文件
* 编译(compilation)：对.i文件进行编译，生成.s汇编文件
* 汇编(assembly)：对.s文件进行汇编，生成.o目标文件
* 链接(linking)：将.o文件与库文件进行链接，生成可执行文件

## 2.gcc命令
### 2.1 命令格式
gcc命令格式及常用选项如下：

```
gcc [-c|-S|-E] [-std=standard] [-Idir] [-Ldir] [-o outfile] infile... [-lxxx]
```

`infile`参数是所有的输入文件，例如源文件、目标文件、库文件等。

默认情况下一次性完成预处理、编译和链接，即直接从源代码转换为可执行文件，也可以通过`-c`、`-S`或`-E`选项控制，见下一节。

编译C++代码应使用g++命令，格式同gcc。

### 2.2 常用选项
`-o outfile`：指定输出文件名

`-E`：仅对源文件进行预处理，不进行编译，如果未指定`-o`选项则结果输出到标准输出

`-S`：对源文件进行预处理、编译，不进行汇编，如果未指定`-o`选项则结果保存到与.c文件同名的.s文件

`-c`：对源文件进行预处理、编译和汇编，不进行链接，如果未指定`-o`选项则结果保存到与.c文件同名的.o文件

`-std=standard`：指定语言标准版本

`-D name[=definition]`：预定义宏，如果省略定义则默认为`1`

`-Wall`：打开所有编译警告

`-Werror`：将所有警告当成错误

`-Idir`：将指定路径添加到头文件搜索路径列表

`-Ldir`：将指定路径添加到库文件搜索路径列表

`-lxxx`：手动指定函数库xxx参与链接

## 3.示例
### 3.1 编译C代码
假设hello.h和hello.c定义了一个函数`hello()`：

hello.h

```c
void hello(const char *to);
```

hello.c

```c
#include "hello.h"
#include <stdio.h>

void hello(const char *to) {
    printf("Hello, %s\n", to);
}
```

main.c包含了hello.h并调用了函数`hello()`：

main.c

```c
#include "hello.h"

int main() {
    hello("world");
    return 0;
}
```

以下命令将生成可执行文件hello_world：

```bash
gcc -o hello_world main.c hello.c
```

也可以分别执行编译和链接：

```bash
gcc -c -o hello.o hello.c
gcc -c -o main.o main.c
gcc -o hello_world main.o hello.o
```

程序输出结果如下：

```bash
$ ./hello_world 
Hello, world
```

### 3.2 编译C++代码
假设函数`hello()`使用C++实现：

hello.h

```cpp
#include <string>

void hello(const std::string& to);
```

hello.cpp

```cpp
#include "hello.h"
#include <iostream>

void hello(const std::string& to) {
    std::cout << "Hello, " << to << std::endl;
}
```

main.cpp内容同main.c，则编译命令只需将gcc改为g++：

```bash
g++ -o hello_world main.cpp hello.cpp
```

注意：在C++中，模板函数必须在头文件中定义，此时头文件也必须作为输入文件。

### 3.3 指定C++标准
可用`-std`参数指定C/C++标准版本，例如：

```bash
g++ -std=c++17 -o hello_world main.cpp hello.cpp
```

支持的C标准版本包括c90、c99、c11、c17等，C++标准版本包括c++98、c++11、c++14、c++17、c++20、c++23等。完整列表及默认值见 [gcc文档](https://gcc.gnu.org/onlinedocs/gcc/C-Dialect-Options.html#C-Dialect-Options) `-std`选项（默认值可能会随gcc版本变化）。

### 3.4 指定头文件搜索路径
GCC编译器默认的头文件搜索路径包括/usr/include（标准库头文件所在目录）、/usr/local/include等，可以使用`-I`选项指定自定义包含目录。

例如，如果头文件hello.h放在include目录中，include目录与main.cpp在同一目录下，则包含hello.h的语句应改为`#include include/hello.h`。另一种方法是使用`-I`选项：

```bash
g++ -o hello_world -Iinclude main.cpp hello.cpp
```

该选项可多次指定。

### 3.5 静态链接库
[静态链接库](https://tldp.org/HOWTO/Program-Library-HOWTO/static-libraries.html)(static library)是一组对象文件的集合，包含已经编译好的类或函数的代码，链接时将被包含到可执行文件中。静态链接库的优点是速度快，可执行文件可以直接使用；缺点是可执行文件体积大，当库文件发生变化时需要重新编译可执行文件。

静态库文件的命名格式为lib+名字+.a。例如，静态库foo的文件名为libfoo.a（在Windows系统上名为foo.lib）。

例如，将`hello()`函数编译为静态链接库hello，可使用以下命令：

```bash
g++ -c -o hello.o hello.cpp
ar rcs libhello.a hello.o
```

得到静态库文件libhello.a，其中ar是一个创建压缩文件的命令。

要在链接时使用静态库文件，有两种方式。假设libhello.a放在lib目录下，lib目录与main.cpp在同一目录下。第一种方式是使用`-l`选项指定库名称（即lib和.a之间的部分），`-L`选项指定库文件搜索路径：

```bash
g++ -o hello_world -Llib main.cpp -lhello
```

注意：`-lhello`必须放在`main.cpp`之后。

第二种方式是直接将库文件作为输入文件：

```bash
g++ -o hello_world main.cpp lib/libhello.a
```

静态链接的过程如下图所示：

![静态链接](/assets/images/gcc-compiler-usage/静态链接.png)

### 3.6 动态链接库
[动态链接库](https://tldp.org/HOWTO/Program-Library-HOWTO/shared-libraries.html)也叫共享库(shared library)，类似于静态链接库，但并不包含在可执行文件中，而是在程序运行时才会被加载。动态链接库的优点是可执行文件体积小，当库文件发生变化时不需要重新编译可执行文件；缺点是速度慢。

动态库文件的命名格式为lib+名字+.so。例如，动态库foo的文件名为libfoo.so（在Windows系统上名为foo.dll）。

例如，将`hello()`函数编译为动态链接库hello，可使用以下命令：

```bash
g++ -c -o hello.o -fPIC hello.cpp
g++ -shared -o libhello.so hello.o
```

动态库文件的使用方式与静态库类似：

```bash
g++ -o hello_world -Llib main.cpp -lhello
```

注意，这样生成的可执行文件不能直接运行，因为找不到动态库文件：

```bash
$ ./hello_world 
./hello_world: error while loading shared libraries: libhello.so: cannot open shared object file: No such file or directory
```

解决方法是将库文件所在路径添加到LD_LIBRARY_PATH环境变量：

```bash
$ export LD_LIBRARY_PATH=/path/to/lib
$ ./hello_world 
Hello, world
```

也可以将库文件放到系统默认的库文件搜索路径，例如/usr/lib、/usr/local/lib等。

使用动态库文件的另一种方式是直接将库文件作为g++命令的参数：

```bash
g++ -o hello_world main.cpp lib/libhello.so
```
这样生成的可执行文件会“记录”库文件路径，因此可以直接运行。

动态链接的过程如下图所示：

![动态链接](/assets/images/gcc-compiler-usage/动态链接.png)

### 3.7 指定库文件搜索路径
系统默认的库文件搜索路径包括/usr/lib、/usr/local/lib等，可以使用使用`-L`选项指定自定义库文件搜索路径，如3.5和3.6节所示。

## 参考
* [GCC官方文档](https://gcc.gnu.org/onlinedocs/gcc/)
* [gcc编译器的使用方法](https://www.cnblogs.com/zhengxunjie/p/11332427.html)
* [Linux基础——gcc编译、静态库与动态库（共享库）](https://blog.csdn.net/daidaihema/article/details/80902012)
* [GCC编译器30分钟入门教程](http://c.biancheng.net/gcc/)
