---
title: C++重定义问题
date: 2022-10-12 22:40:23 +0800
categories: [C/C++]
tags: [cpp, redefinition]
---
C++支持声明和定义分离，通常的做法是将声明放在头文件中、定义放在源文件中，通过包含头文件来引入声明。多次声明一个变量或函数不会有问题，但多次定义则会导致重定义错误。下面在两种不同场景下进行分析。

## 1.场景1
假设头文件foo.h声明了一个全局常量`A`和一个函数`f`：

```cpp
extern const int A;
int f(int x);
```

a.cpp包含foo.h并定义了常量`A`：

```cpp
#include "foo.h"

const int A = 2;
```

f.cpp包含foo.h并定义了函数`f`：

```cpp
#include "foo.h"

int f(int x) {
    return x + A;
}
```

主程序main.cpp调用了函数`f`：

```cpp
#include <iostream>

#include "foo.h"

using namespace std;

int main() {
    cout << f(5) << endl;
    return 0;
}
```

文件包含关系如下：

![场景1-文件包含关系](/assets/images/cpp-redefinition/场景1-文件包含关系.png)

此时程序能够输出正确结果：

```bash
$ g++ -o main a.cpp f.cpp main.cpp
$ ./main
7
```

以上是正确的做法，下面分别尝试将常量`A`和函数`f`的定义移至头文件，看是否会有问题。

### 1.1 在头文件中定义常量
将常量`A`的定义移至foo.h：

```cpp
const int A = 2;
int f(int x);
```

由于f.cpp和main.cpp都包含了foo.h，因此常量`A`实际定义了两次。

此时不再需要a.cpp，编译运行结果如下：

```bash
$ g++ -o main f.cpp main.cpp
$ ./main
7
```

编译通过了，程序也输出了正确的结果，因此在不同源文件中重复定义常量不会导致编译错误（一般来说没有必要这样做，但用在`switch`语句`case`标签中的常量必须在声明的同时初始化，而不能用`extern`声明）。

### 1.2 在头文件中定义变量
将foo.h中的常量`A`改为变量：

```cpp
int A = 2;
int f(int x);
```

再次编译：

```bash
$ g++ -o main f.cpp main.cpp
/tmp/ccaD1mlU.o:(.data+0x0): multiple definition of `A'
/tmp/ccMz5gGb.o:(.data+0x0): first defined here
collect2: error: ld returned 1 exit status
```

发现链接器报错：变量`A`重定义，即使在开头增加`#pragma once`或`#ifndef`保护仍然会报错。

### 1.3 在头文件中定义函数
将函数`f`的定义移至foo.h中：

```cpp
extern const int A;

int f(int x) {
    return x + A;
}
```

编译结果如下：

```bash
$ g++ -o main a.cpp main.cpp
/tmp/cckzK4gH.o: In function `f(int)':
main.cpp:(.text+0x0): multiple definition of `f(int)'
/tmp/cc8Vn4bH.o:a.cpp:(.text+0x0): first defined here
collect2: error: ld returned 1 exit status
```

与变量的重定义类似，链接器报错函数重定义，即使使用`#pragma once`或`#ifndef`保护仍然会报错。

## 2.场景2
场景1的文件包含关系是一个树状结构，下面考虑图状结构。假设头文件a.h、f.h、g.h分别声明了常量`A`、函数`f`和函数`g`，并且f.h包含了a.h，g.h包含了f.h，各文件的内容如下：

a.h

```cpp
extern const int A;
```

a.cpp

```cpp
#include "a.h"

const int A = 2;
```

f.h

```cpp
#include "a.h"

int f(int x);
```

f.cpp

```cpp
#include "f.h"

int f(int x) {
    return x + A;
}
```

g.h

```cpp
#include "f.h"

int g(int x);
```

g.cpp

```cpp
#include "g.h"

int g(int x) {
    return f(x) * A;
}
```

main.cpp

```cpp
#include <iostream>

#include "f.h"
#include "g.h"

using namespace std;

int main() {
    cout << f(5) << endl;
    cout << g(5) << endl;
    return 0;
}
```

文件包含关系如下：

![场景2-文件包含关系](/assets/images/cpp-redefinition/场景2-文件包含关系.png)

程序运行结果如下：

```bash
$ g++ -o main a.cpp f.cpp g.cpp main.cpp
$ ./main
7
14
```

注：在这个例子中，头文件之间的包含关系理论上是没有必要的（只需在f.cpp中包含a.h中即可，而不需要在f.h中包含），这里只是为了说明问题。

下面仍然分别尝试将常量和函数的定义移至头文件中。

### 2.1 在头文件中定义常量
将常量`A`的定义移至a.h中，这次是编译器报错：

```bash
$ g++ -o main f.cpp g.cpp main.cpp
In file included from f.h:1:0,
                 from g.h:1,
                 from main.cpp:4:
a.h:1:11: error: redefinition of 'const int A'
 const int A = 2;
           ^
In file included from f.h:1:0,
                 from main.cpp:3:
a.h:1:11: note: 'const int A' previously defined here
 const int A = 2;
           ^
```

从错误信息可以分析出原因：在main.cpp中分别通过f.h和g.h **两次**包含了a.h，从而在同一个源文件中两次定义了常量`A`，因此在编译期间就报错。

如果给a.h加上`#pragma once`或`#ifndef`保护，则编译通过并正常运行：

```bash
$ g++ -o main f.cpp g.cpp main.cpp
$ ./main
7
14
```

这是因为编译器在main.cpp中展开g.h时，`#pragma once`阻止了再次包含a.h，因此main.cpp这个源文件中没有了重复定义（但f.cpp和g.cpp中仍然分别有一个常量`A`的定义）。

### 2.2 在头文件中定义变量
将a.h中的常量`A`改为变量，则不加`#pragma once`的编译结果如下：

```bash
$ g++ -o main f.cpp g.cpp main.cpp
In file included from f.h:1:0,
                 from g.h:1,
                 from main.cpp:4:
a.h:1:5: error: redefinition of 'int A'
 int A = 2;
     ^
In file included from f.h:1:0,
                 from main.cpp:3:
a.h:1:5: note: 'int A' previously defined here
 int A = 2;
     ^
```

编译器报错，原因同2.1节。

增加`#pragma once`的编译结果如下：

```bash
$ g++ -o main f.cpp g.cpp main.cpp
/tmp/ccEZX8VQ.o:(.data+0x0): multiple definition of `A'
/tmp/ccLS1UuT.o:(.data+0x0): first defined here
/tmp/ccMgRK4O.o:(.data+0x0): multiple definition of `A'
/tmp/ccLS1UuT.o:(.data+0x0): first defined here
collect2: error: ld returned 1 exit status
```

链接器报错，原因同1.2节。

### 2.3 在头文件中定义函数
将函数`f`的定义移至f.h中，类似地，不加`#pragma once`时编译器报错：

```bash
$ g++ -o main a.cpp g.cpp main.cpp
In file included from g.h:1:0,
                 from main.cpp:4:
f.h: In function 'int f(int)':
f.h:3:5: error: redefinition of 'int f(int)'
 int f(int x) {
     ^
In file included from main.cpp:3:0:
f.h:3:5: note: 'int f(int)' previously defined here
 int f(int x) {
     ^
```

加上`#pragma once`时链接器报错：

```bash
$ g++ -o main a.cpp g.cpp main.cpp
/tmp/ccqHxXhg.o: In function `f(int)':
main.cpp:(.text+0x0): multiple definition of `f(int)'
/tmp/cce5N9Pb.o:g.cpp:(.text+0x0): first defined here
collect2: error: ld returned 1 exit status
```

## 3.总结
* 在同一个源文件中多次定义同一个常量、变量或函数时，**编译器**会报重定义错误。
* 在不同源文件中多次定义同一个常量不会报错，而多次定义同一个变量或函数时**链接器**会报重定义错误。
* `#pragma once`或`#ifndef`保护只能解决**同一个源文件中**由于头文件传递包含导致的多次定义，不同源文件中的重复定义仍然会导致链接错误。
* 定义全局常量、变量或函数的最好做法是在头文件中声明（常量和变量使用`extern`声明），仅在**一个**源文件中定义，从而可以避免重定义错误。
* 例外：（1）模板函数必须在头文件中定义；（2）`switch`语句`case`标签使用的常量不能用`extern`声明。
