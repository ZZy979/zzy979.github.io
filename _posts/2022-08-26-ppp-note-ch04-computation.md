---
title: 《C++程序设计原理与实践》笔记 第4章 计算
date: 2022-08-26 00:09:30 +0800
categories: [C/C++, PPP]
tags: [cpp, symbolic constant, operator, type conversion, if statement, switch statement, while statement, for statement, break statement, continue statement, function, vector]
---
本章将介绍一些与计算相关的基本概念。

## 4.1 计算
程序所做的事情就是计算(compute)，即接受输入、产生输出。

![程序](/assets/images/ppp-note-ch04-computation/程序.png)

从编程的角度看，最重要也是最有趣的两类输入、输出是“从其他程序输入/输出”和“从程序的其他部分输入/输出”。本书后续的大部分内容可以视为后一种类型的实例：在协作完成一个大的软件时，应该如何合理地设计程序结构，并能够保证每一个子程序之间都能够正确地共享和交互数据？这是编程的核心问题。

![大的程序](/assets/images/ppp-note-ch04-computation/大的程序.png)

其中I/O是 "input/output" 的缩写，一部分代码的输出是下一部分代码的输入。代码之间的数据共享可以通过内存、硬盘或者网络完成。

## 4.2 目标和工具
程序员的任务就是将计算表达出来，并且做到**正确、简单、高效**。一个输出错误结果的快速程序是没有任何意义的。同样，一个正确、高效但是非常复杂的程序，最终的结果往往是被放弃或者重写。为了适应不同的需求和硬件环境，有用的程序总会被无数次改写。因此，一个程序，或者它的任一子程序，应该以尽可能简单的方式来实现。

在我们开始编写代码的时候，就要特别关注这三个基本原则，这也是专业程序员的最重要职责。

组织程序的主要工具是把一个大的计算任务划分为许多小任务。这一技术主要包括两类方法：
* 抽象(abstraction)：将使用一个功能不需要了解的实现细节隐藏在接口之后。例如，为了实现电话簿的排序，不需要了解排序算法的细节，只需要调用C++标准库的`sort`函数即可。
* 分治(divide and conquer)：将一个大的问题分为几个小问题分别解决。例如，为了建立一个字典，可以将这一任务分为三个子任务：读取数据、排序和输出数据。

如果你一心想构建能长久留存的东西，就应该在设计过程中对代码结构和组织投入更多关注，而不是在发生错误后再被迫回过头来学习。

## 4.3 表达式
**表达式**(expression)是程序的最基本组成单元。表达式从多个**操作数**(operand)计算一个值。最简单的表达式是**字面值**(literal value)，例如`10`、`'a'`、`3.14`和`"Norah"`。

变量名也是一种表达式，表示与名字对应的那个对象。例如：

```cpp
// compute area:
int length = 20;
int width = 40;
int area = length * width;
```

注意：**变量名用于赋值运算符左边和右边的含义是不同的**。在初始化（或赋值）语句中，`length`在左边，表示“名为`length`的对象”（`length`的**左值**(lvalue)）；在计算面积的表达式中，`length`在右边，表示“名为`length`的对象的值”（`length`的**右值**(rvalue)）。下图可以更清楚地解释这个概念：

![整型对象](/assets/images/ppp-note-ch04-computation/整型对象.png)

上图表示了一个名为`length`的整型对象，其值为99。当`length`是左值时，表示这个对象本身；当`length`是右值时，表示这个对象的值。

可以使用运算符组合表达式得到更复杂的表达式，例如：

```cpp
int perimeter = (length + width) * 2;
```

运算符有[优先级](https://en.cppreference.com/w/cpp/language/operator_precedence)，可以使用括号改变默认优先级。例如，`a*b+c/d`表示`(a*b)+(c/d)`，`a*(b+c)/d`表示`(a*(b+c))/d`。

使用括号的第一原则是“如果对运算符优先级不确定，就用括号”。当然，要尽量熟悉优先级规则，像`a*b+c/d`这种简单的表达式还加括号的话，无疑会降低程序的可读性。

绝对不要在程序中使用非常复杂的表达式，例如`a*b+c/d*(e-f/g)/h+7`。

### 4.3.1 常量表达式
程序中经常会用到常量。C++提供了**符号常量**(symbolic constant)，即初始化后就不能再赋值的命名对象。例如：

```cpp
const double pi = 3.14159;
pi = 7;              // error: assignment to const
int v = 2 * pi / r;  // OK: we just read pi; we don't try to change it
```

使用常量一方面能够维护程序的可读性，例如看到299792458或2.54可能无法猜到它的含义；另一方面方便修改，如果要求把`pi`的精度提高到12位有效数字，则只需修改常量`pi`的定义即可。

除个别情况外（例如0和1），程序中应该尽量少用字面值，而是使用有意义名字的符号常量。像2.54这种不能直接识别含义的字面值通常称为**魔数**(magic number)，要避免使用魔数。

**常量表达式**(constant expression)仅由符号常量和字面值组成。例如：

```cpp
const int max = 17;
int val = 19;

max+2  // a constant expression: a const plus a literal
val+2  // not a constant expression: it uses a variable
```

### 4.3.2 运算符
下表是最常用的运算符：

| 运算符 | 名称 |
| --- | --- |
| `f(a)` | 函数调用(function call) |
| `a[b]` | 下标(subscription) |
| `++a` | 递增(increment) |
| `--a` | 递减(decrement) |
| `!a` | 逻辑非(logical NOT) |
| `-a` | 负(unary minus) |
| `a*b` | 乘(multiplication) |
| `a/b` | 除(division) |
| `a%b` | 余数(remainder) |
| `a+b` | 加(addition) |
| `a-b` | 减(subtraction) |
| `a<<b` | 左移(left shift)或输出 |
| `a>>b` | 右移(right shift)或输入 |
| `a<b` | 小于(less than) |
| `a<=b` | 小于等于(less than or equal) |
| `a>b` | 大于(greater than) |
| `a>=b` | 大于等于(greater than or equal) |
| `a==b` | 等于(equal) |
| `a!=b` | 不等于(not equal) |
| `a&&b` | 逻辑与(logical AND) |
| `a||b` | 逻辑或(logical OR) |
| `a=b` | 赋值(assignment) |
| `a+=b` | 复合赋值(Compound assignment) |

完整列表：[Operators](https://en.cppreference.com/w/cpp/language/expressions#Operators)

其中，递增、递减和赋值运算符要求a是左值。复合运算符`a+=b`等价于`a=a+b`，对`-`、`*`、`/`、`%`等运算符同理。

注意**比较运算符不能连用**，表达式`a<b<c`意味着`(a<b)<c`，而`a<b`的结果是布尔值：`true`或`false`，因此表达式`a<b<c`的值等于`true<c`或`false<c`，这显然是错误的。要想表达“b的值是否介于a和c之间”应该使用`a<b && b<c`。

增量表达式有以下三种形式：`++a`、`a+=1`和`a=a+1`。建议使用第一种形式，因为它直观地表达了增量的含义。类似地，建议使用`a*=scale`而不是`a=a*scale`。

### 4.3.3 类型转换
表达式中允许存在不同的数据类型，例如`2.5/2`。对于目前遇到的数据类型，转换规则是：如果运算符的一个操作数是`double`类型，则进行浮点算术运算，结果为`double`；否则进行整型算术运算，结果为`int`。例如：
* `5/2`结果是2（不是2.5）
* `2.5/2`意味着`2.5/double(2)`，结果是1.25
* `'a'+1`意味着`int('a')+1`，结果是98

其中运算符`T(v)`表示将值`v`转换为`T`类型。

注意，在包含浮点操作数的表达式中容易忘记整数除法。例如，摄氏温度与华氏温度的转换公式为：f=9/5*c+32，但以下代码是错误的：

```cpp
double c;
cin >> c;
double f = 9/5 * c + 32;  // beware!
```

因为`9/5`的值是1而不是1.8（乘法和除法运算符是从左到右结合的，`9/5*c`等价于`(9/5)*c`，而括号中是整数除法）。应该将9或5（或二者）转换为`double`类型：`9.0/5 * c + 32`。

## 4.4 语句
C++通过**语句**(statement)来实现循环、条件判断、输入输出等功能。

目前已经见过两种语句：表达式语句和声明。**表达式语句**(expression statement)是以分号结尾的表达式。例如：

```cpp
a = b;
++b;
```

注意，表达式语句需要“起作用”（即包含赋值、I/O、函数调用等操作），单纯计算一个值而没有使用结果的表达式语句是没有用的，例如：

```cpp
1+2;
a*b;
```

### 4.4.1 选择语句
C++的**选择**(selection)语句包括`if`语句和`switch`语句。

#### 4.4.1.1 if语句
`if`语句是最简单的选择语句，一般形式为

```cpp
if (表达式)
    语句1
else
    语句2
```

如果表达式为真则执行语句1，否则执行语句2，`else`部分是可选的。

注：
* 括号中的条件还可以是一个带初始值的变量声明语句，变量的值必须能够转换为`bool`。例如：

```cpp
// Base* bp;
if (Derived* p = dynamic_cast<Derived*>(bp))
    p->f();
```

* C++17引入了带有初始化的`if`语句，可以在`if`语句的作用域内声明一个变量。例如：

```cpp
// map<int, string> m;
if (auto it = m.find(10); it != m.end())
    cout << it->second << endl;
```

C++中并没有else-if语句，但可以通过在`else`部分使用另一条`if`语句来实现：

```cpp
if (表达式1)
    语句1
else if (表达式2)
    语句2
else
    语句3
```

下面的程序执行厘米和英寸之间的转换：

[厘米-英寸转换](https://github.com/ZZy979/PPP-code/blob/main/ch04/convert_length_unit.cpp)

注意：如果判断单位的`if`语句写成`if (unit == 'i') ... else ...`，则只是区分了`'i'`和其他所有情况，而没有专门针对`'c'`进行判断。此时如果用户输入错误的单位（例如`15f`），则会执行`else`部分，即厘米到英寸的转换，而这个结果是错误的。

我们**必须始终使用错误的输入来测试程序**，因为无法避免用户（有意或无意的）错误输入，程序应该能够正确处理。

[试一试](https://github.com/ZZy979/PPP-code/blob/main/ch04/convert_currency.cpp)

#### 4.4.1.2 switch语句
上面示例中`unit`与`'i'`、`'c'`的比较是最常见的选择形式：基于一个值与多个常量比较的选择。因此C++专门提供了一个语句：`switch`语句，一般形式为

```cpp
switch (表达式) {
    case 常量表达式1:
        语句1
    case 常量表达式2:
        语句2
    ...
    default:
        语句n
}
```

关键字`switch`后括号中的值与一组常量进行比较，每个常量在一个`case`标签之后。如果该值与某一常量相等，将执行该`case`后的语句，通常每个`case`都以`break`结束；如果该值与任何常量都不相等，则执行`default`后的语句。

`default`不是必需的，但建议加上，除非你能够完全确定列出了所有可能的分支。注意，编程将教会你世界上没有绝对确定的事情。

利用`switch`语句可以将前面的程序改写为：

[厘米-英寸转换(switch)](https://github.com/ZZy979/PPP-code/blob/main/ch04/convert_length_unit_v2.cpp)

#### 4.4.1.3 switch技术细节
* `switch`语句括号中的值必须是整数、字符或枚举类型，**不能是字符串**
* `case`标签后的值必须是常量表达式，不能使用变量
* 不能在两个`case`标签中使用相同的值
* 可以多个`case`标签使用相同的语句（见下面的示例）
* 不要忘记在每个`case`末尾加上`break`，注意编译器不会给出任何警告

如果希望对多个值采取同样的操作，只需在语句前加上多个`case`标签即可，而不是重复写同样的代码。例如：

```cpp
switch (a) {
    case '0': case '2': case '4': case '6': case '8':
        cout << "is even\n";
        break;
    case '1': case '3': case '5': case '7': case '9':
        cout << "is odd\n";
        break;
    default:
        cout << "is not a digit\n";
        break;
}
```

**使用switch语句最常犯的错误是忘记在case末尾加上break**。在前面`switch`版本的单位转换程序中，如果删除两个`break`语句，则编译器不会报错，但是当执行完`case 'i'`的代码后，会接着执行`case 'c'`的代码。如果输入 "2i" ，程序将输出

```
2in == 5.08cm
2cm == 0.787402in
```

注：这种情况术语叫作 "fallthrough" ，通常是一个逻辑错误，只有在多个`case`使用相同操作的情况下是有用的（例如上面判断奇偶数的程序）

[试一试](https://github.com/ZZy979/PPP-code/blob/main/ch04/convert_currency_v2.cpp)

### 4.4.2 循环语句
**循环**(repetition/loop)语句用于重复执行某些操作，在对数据结构的一系列元素进行同样处理时也称为**迭代**(iteration)。

#### 4.4.2.1 while语句
世界上第一台能存储程序的计算机(EDSAC)上运行的第一个程序就是一个循环，其目的是打印一个简单的平方数表：

```
0	0
1	1
2	4
3	9
4	16
...
98	9604
99	9801
```

每一行是一个数字，后面跟着一个制表符(tab, `'\t'`)，然后是该数的平方。该程序的C++版本如下：

[打印平方数表(while)](https://github.com/ZZy979/PPP-code/blob/main/ch04/print_squares_table.cpp)

其中`square(i)`是计算平方的函数，在4.5节介绍。

`while`语句的一般形式为

```cpp
while (表达式)
    语句
```

每次循环开始时判断**循环条件**（括号中的表达式）是否成立，如果成立则执行**循环体**（后面的语句），之后进入下一次循环；否则结束循环。循环条件使用**循环变量**来控制，`while`语句的循环变量必须在`while`语句之前定义和初始化。

在上面的例子中，`while`语句的循环变量是`i`，循环条件是`i < 100`，循环体输出平方数表的一行，并将`i`的值加1。由于`i`的初始值为0，因此循环体将执行100次（`i`依次取值0~99，循环结束时`i`的值为100）。

编写循环语句的关键是循环条件的设置和所有变量的初始化。

[试一试](https://github.com/ZZy979/PPP-code/blob/main/ch04/print_character_table.cpp)

#### 4.4.2.2 程序块
用大括号`{}`括起来的语句序列称为**程序块**(block)或**复合语句**(compound statement)。程序块也是一种语句。

注：上面示例中`while`语句的循环体就是包含两条语句的程序块，如果不加大括号，则循环体只包含输出语句（这将导致死循环，因为循环变量的值没有改变）：

```cpp
while (i < 26)
    cout << char('a' + i) << '\t' << ('a' + i) << '\n';
    ++i;
```

等价于

```cpp
while (i < 26) {
    cout << char('a' + i) << '\t' << ('a' + i) << '\n';
}
++i;
```

缩进只影响代码的可读性，对编译器没有任何作用。

#### 4.4.2.3 for语句
对数字序列（例如0~9）进行迭代非常普遍，因此C++提供了专门的语法——`for`语句，将循环变量的控制集中放在开头，便于阅读和理解。其语法如下：

```cpp
for (初始化; 循环条件; 表达式)
    语句
```

首先执行初始化部分；每次循环开始时判断循环条件，如果成立则执行循环体和表达式（一般用于递增循环变量），之后进入下一次循环；否则结束循环。`for`语句等价于以下`while`语句：

```cpp
初始化;
while (循环条件) {
    语句
    表达式;
}
```

除非循环体中包含`continue`语句。

注：
* `break`语句用于直接退出循环
* `continue`语句用于直接开始下一次循环，对于`while`语句是执行测试部分，对于`for`语句是执行循环变量递增部分

使用`for`语句打印平方数表的示例如下：

[打印平方数表(for)](https://github.com/ZZy979/PPP-code/blob/main/ch04/print_squares_table_v2.cpp)

当一个循环的初始化、循环条件和递增操作都很简单时，使用`for`语句会使代码更容易理解和维护；否则才使用`while`语句。

注意：**不要在for语句的循环体内修改循环变量的值。** 虽然这种操作没有语法错误，但它违背了读者对于循环的普遍理解。

[试一试](https://github.com/ZZy979/PPP-code/blob/main/ch04/print_character_table_v2.cpp)

## 4.5 函数
**函数**(function)是一个命名的语句序列，能够返回计算结果（称为**返回值**(return value)）。**函数定义**(function definition)的语法如下：

```cpp
返回值类型 函数名(参数表) {
    函数体
}
```

其中**参数表**(parameter list)的每一个元素称为**参数**(parameter)或**形式参数**(formal argument)。**函数体**(function body)是实现具体功能的程序块，使用`return`语句返回结果。

标准库提供了许多有用的函数，也可以编写自己的函数。“打印平方数表”示例中，`square`函数定义如下：

```cpp
// return the square of x
int square(int x) {
    return x * x;
}
```

定义表示这是一个名为`square`的函数，接受一个`int`参数（名为`x`），返回一个`int`。

**函数调用**(function call)的语法如下：

```cpp
函数名(参数表)
```

其中参数表的每一个元素称为**实际参数**(argument/actual argument)。函数调用的结果是一个返回值类型的表达式，可以用在任何允许使用该类型表达式的地方（例如变量初始化、其他表达式、函数参数等）。

例如，在上面的示例的输出语句中，表达式`square(i)`使用实际参数`i`调用了`square`函数，整体是一个`int`值。

在调用函数时，必须严格按照函数的定义传递参数（参数个数及类型必须完全一致），否则编译器会给出错误信息。例如：

```cpp
square(2);               // probably a mistake: unused return value
int v1 = square();       // error: argument missing
int v2 = square;         // error: parentheses missing
int v3 = square(1, 2);   // error: too many arguments
int v4 = square("two");  // error: wrong type of argument – int expected
```

参数表可以为空，如果不需要返回任何结果，返回值类型可以设置为`void`。例如：

```cpp
// take no argument; return no value
void write_sorry() {
    cout << "Sorry\n";
}
```

### 4.5.1 使用函数的原因
当需要将一部分计算任务独立实现时，可以将其定义为一个函数，因为这样可以：
* 使计算逻辑分离
* 使代码更清晰（通过使用函数名）
* 在程序中可以多次使用该函数
* 简化测试

计算平方的函数`square`看起来没有必要（可以直接使用表达式`x * x`代替），这只是因为该函数很简单。对于求平方根的函数`sqrt`，其实现代码很长、很复杂，而使用函数不需要了解求平方根的实现细节，只需要知道`sqrt(x)`返回`x`的平方根就可以了。

如果想让主函数中的循环更简单，可以将程序改写为

```cpp
int print_square(int v) {
    cout << v << '\t' << v * v << '\n';
}

int main() {
    for (int i = 0; i < 100; ++i)
        print_square(i);
    return 0;
}
```

之所以不使用`print_square()`是因为：
* `print_square()`是一个比较特殊的函数，以后很难再次用到它，而`square()`可以多次使用
* `square()`几乎不需要任何说明文档，而`print_square()`显然需要解释

根本原因是`print_square()`执行了两个逻辑上独立的操作：计算平方和输出结果。**如果每个函数只完成单一的操作，那么程序将更易于编写和理解。**

试一试：

```cpp
int square(int x) {
    int s = 0;
    for (int i = 1; i <= x; ++i)
        s += x;
    return s;
}
```

### 4.5.2 函数声明
注意到调用函数所需的所有信息（返回值类型、函数名和参数表）都已经包含在函数定义的第一行。例如，看到`int square(int x)`就可以写出`int x = square(44);`，而不需要看函数体。

大多数情况下，我们只关心如何调用函数。C++提供了一种将该信息与函数定义分离的方法，称为**函数声明**(function declaration)。例如：

```cpp
int square(int);      // declaration of square
double sqrt(double);  // declaration of sqrt
```

注意，函数声明以分号结束（代替了函数定义中的函数体部分），形式参数名可以省略。

如果想使用某个函数，只需要写出该函数的声明（通常是通过`#include`语句），而该函数的定义可以在程序的其他部分。函数声明和定义的分离对于大型程序是至关重要的，因为这使得读者不必阅读大部分（函数定义）代码，而专注于程序的一部分。

## 4.6 向量
在编写程序之前，首先要准备好相关的数据。最简单、最常用的数据存储形式是**向量**(`vector`)。

向量是可以通过**索引**(index)访问的元素序列。例如，下面是一个名为`v`的向量：

![向量](/assets/images/ppp-note-ch04-computation/向量.png)

其中，第一个元素的索引是0，第二个是1，以此类推。可以通过向量名加元素的索引（下标）来引用一个元素，例如`v[0]`的值是5，`v[1]`的值是7，等等。向量元素的索引从0开始，每次加1。图中特别强调了向量“知道自己的大小”，即向量不仅存储元素，也存储它的大小（元素个数）。

可以像这样创建一个向量：

```cpp
vector<int> v = {5, 7, 9, 4, 6, 8};  // vector of 6 ints
```

注：
* C++11之后才有这种初始化语法
* 使用向量需要包含标准库头文件\<vector\>


创建一个向量需要指定元素类型和初始元素的集合。元素类型在`vector`后的尖括号(`<>`)中，在这里是`int`。下面是另一个示例：

```cpp
vector<string> philosopher = {"Kant", "Plato", "Hume", "Kierkegaard"};  // vector of 4 strings
```

向量元素可以像普通变量一样使用和赋值，只能存储向量所声明的元素类型的值：

```cpp
v[1] = 99;      // OK
v[2] = "Hume";  // error: trying to assign a string to an int
```

也可以通过指定大小和初始值来创建向量，如果未指定初始值则使用元素类型的默认值，如果未指定大小则创建一个空向量。例如：

```cpp
vector<int> v;                  // empty vector
vector<int> vi(6);              // vector of 6 ints initialized to 0
vector<string> vs(4);           // vector of 4 strings initialized to ""
vector<double> vd(1000, -1.2);  // vector of 1000 doubles initialized to -1.2
```

注意不能引用一个不存在的元素（例如`vd[20000]`），否则将导致运行时错误。

### 4.6.1 遍历向量
注：第一版书中没有这一节

可以这样打印向量的元素：

```cpp
for (int i = 0; i < v.size(); ++i)
    cout << v[i] << '\n';
```

`v.size()`返回向量`v`的元素个数，`size()`是向量的一个成员函数。由于索引从0开始，因此向量元素的索引范围是`[0, v.size())`（这种左闭右开区间在C++标准库中普遍使用）。

从C++11开始，可以使用更简单的**基于范围的for循环**(range-based for loop)来遍历一系列元素（例如向量的元素）：

```cpp
for (int x : v)  // for each x in v
    cout << x << '\n';
```

### 4.6.2 增长向量
在使用向量时，一般是从空向量开始，根据需要逐步添加数据。这里的关键操作是`push_back()`，它将一个新元素添加到向量结尾。例如：

```cpp
vector<double> v;  // v is empty
v.push_back(2.7);  // v == {2.7}, v.size() == 1
v.push_back(5.6);  // v == {2.7, 5.6}, v.size() == 2
v.push_back(7.9);  // v == {2.7, 5.6, 7.9}, v.size() == 3
```

`push_back`是向量的另一个成员函数。

**成员函数**(member function)调用的语法如下：

```cpp
对象名.成员函数名(参数表)
```

向量类似于C语言中的数组，但不需要事先指定大小，而且可以往向量中添加任意多个元素。

### 4.6.3 一个数值计算的例子
我们经常会把一系列数据读入程序来处理，例如根据数据绘制图形、计算平均值和中位数、找出最大值、排序、查找、与其他数据结合、与其他数据比较等。在做各种处理之前，首先要把数据读入内存中。下面的程序读取一系列温度数据（事先不知道有多少个），并计算平均值和中位数：

[计算温度的平均值和中位数](https://github.com/ZZy979/PPP-code/blob/main/ch04/mean_median_temperature.cpp)

其中，读取循环每次读取一个`double`并添加到向量结尾。使用读取操作`cin >> temp`作为循环条件，如果正确读取了一个值，则`cin >> temp`为`true`，否则为`false`。因此`while`循环将读取所有`double`，直到读到其他类型的数据为止（例如字符 '|'）。任何非`double`类型的数据都可以作为输入结束的标志，也可以使用EOF终止输入（见3.5.1节）。

为了计算平均值，只需将所有元素累加到`sum`中，然后除以元素个数(`temps.size()`)即可。

为了计算中位数，需要先将元素排序，这里使用了标准库排序算法`sort()`（定义在头文件\<algorithm\>中）。`sort()`的两个参数分别是待排序元素序列的开始和结束位置（左闭右开），向量的成员函数`begin()`和`end()`分别返回这两个位置。温度数据排好序之后，求中位数就很简单了：假设元素个数是n，如果n为奇数，则中位数是索引为`n/2`的元素；如果n为偶数，则中位数是索引为`n/2-1`和`n/2`的两个元素的平均数。例如：

```
n=5: 0 1 [2] 3 4
n=4: 0 [1 2] 3
```

注：该程序没有检查向量为空的情况，此时计算中位数会导致下标越界

### 4.6.4 一个文本处理的例子
下面的例子建立一个简单的字典：

[简单字典](https://github.com/ZZy979/PPP-code/blob/main/ch04/simple_dictionary.cpp)

输入一些单词（空白符分隔的字符串），程序将按字典序输出这些单词，同时删除重复的单词。

如何停止字符串输入？在读取数字时，可以通过输入非数值字符来结束输入。但在这里不行，因为所有的（普通字符）都可以被读入字符串。可以利用“非普通”的字符——EOF（见3.5.1节），在Windows中使用Ctrl+Z，在UNIX中使用Ctrl+D。

判断重复单词的语句检查前一个单词是否与当前单词相同(`words[i - 1] != words[i])`)，如果不同就输出这个单词，否则不输出。显然，第一个单词(`i == 0`)不存在前一个单词，因此单独判断。两个条件使用或运算符(`||`)组合起来。

注意，字符串的关系运算符（`<`、`>`等）使用**字典序**，因此`"Ape"`在`"Apple"`和`"Chimpanzee"`之前。

[试一试](https://github.com/ZZy979/PPP-code/blob/main/ch04/bleep_disliked_words.cpp)

## 简单练习
[长度统计](https://github.com/ZZy979/PPP-code/blob/main/ch04/length_statistics.cpp)

## 习题
* [4-2](https://github.com/ZZy979/PPP-code/blob/main/ch04/mean_median_temperature.cpp)
* [4-3](https://github.com/ZZy979/PPP-code/blob/main/ch04/exec4-3.cpp)
* [4-4](https://github.com/ZZy979/PPP-code/blob/main/ch04/exec4-4.cpp)
* [4-5](https://github.com/ZZy979/PPP-code/blob/main/ch04/exec4-5.cpp)
* [4-6](https://github.com/ZZy979/PPP-code/blob/main/ch04/exec4-6.cpp)
* [4-9](https://github.com/ZZy979/PPP-code/blob/main/ch04/exec4-9.cpp)
* [4-10](https://github.com/ZZy979/PPP-code/blob/main/ch04/exec4-10.cpp)
* [4-12](https://github.com/ZZy979/PPP-code/blob/main/ch04/exec4-12.cpp)
* [4-14](https://github.com/ZZy979/PPP-code/blob/main/ch04/exec4-14.cpp)
* [4-16](https://github.com/ZZy979/PPP-code/blob/main/ch04/exec4-16.cpp)
* [4-17](https://github.com/ZZy979/PPP-code/blob/main/ch04/exec4-17.cpp)
* [4-18](https://github.com/ZZy979/PPP-code/blob/main/ch04/exec4-18.cpp)
* [4-19~4-21](https://github.com/ZZy979/PPP-code/blob/main/ch04/exec4-21.cpp)
