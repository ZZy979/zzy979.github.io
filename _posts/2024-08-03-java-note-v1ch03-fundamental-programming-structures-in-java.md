---
title: 《Java核心技术》笔记 第3章 Java的基本编程结构
date: 2024-08-03 11:24:35 +0800
categories: [Java, Core Java]
tags: [java, comment, data type, rounding error, unicode, variable, declaration, initialization, symbolic constant, enumeration, operator, mathematical function, integer division, division rounding, type conversion, assignment, switch expression, string, text block, io, formatting, file access, block, if statement, while statement, do-while statement, for statement, switch statement, break statement, continue statement, array, command-line argument, sort, two-dimensional array]
math: true
---
本章主要介绍如何在Java中实现基本编程概念，例如数据类型、分支和循环。

## 3.1 一个简单的Java程序
下面仔细分析一个最简单的Java程序——只是向控制台打印一条消息：

[程序清单3-1 FirstSample/FirstSample.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch03/FirstSample/FirstSample.java)

这个程序虽然很简单，但所有的Java应用都具有这种结构，因此值得花些时间来研究。首先，**Java区分大小写**。如果出现了大小写错误，程序将无法运行。

下面逐行地查看这段源代码。关键字`public`称为**访问修饰符**(access modifier)，用于控制程序的其他部分对这段代码的访问级别（详见第5章）。关键字`class`用于定义一个类（第4章将详细介绍类，现在只需将类看作是程序逻辑的容器）。类是所有Java应用的构建块，Java程序中的**所有内容**都必须放在类中。

关键字`class`后面跟着类名。Java中的类名必须以字母开头，后面可以跟字母和数字的任意组合。长度基本上没有限制。但不能使用Java保留字（例如`public`或`class`）作为类名。

标准命名约定：类名是以大写字母开头的名词。如果名字由多个单词组成，每个单词的首字母都大写。这种方式称为**驼峰命名法**(CamelCase)，例如`FirstSample`。

源代码的文件名必须与公共类的类名相同，并用.java作为扩展名。因此，这段代码必须存储在名为FirstSample.java的文件中。

编译这段源代码：

```shell
javac FirstSample.java
```

将得到一个包含这个类的字节码的文件。Java编译器自动将这个字节码文件命名为FirstSample.class，并与源文件存储在同一个目录下（注：可以使用`javac`命令的`-d`选项指定输出目录）。最后，使用下面的命令运行这个程序：

```shell
java FirstSample
```

程序执行后，控制台上将显示字符串 "We will not use 'Hello, World!'" 。

当使用`java ClassName`命令运行已编译的程序时，Java虚拟机将从指定的类中的`main()`方法开始执行。因此为了使代码能够执行，类的源代码中**必须**包含一个`main()`方法。

在Java中，用大括号`{}`划分程序的各个部分（通常称为**块**(block)）。Java中任何方法的代码都必须以`{`开始，以`}`结束。

暂且不考虑`static void`，学习完第4章之后就会理解。现在只需记住：每个Java应用都必须有一个`main()`方法，其声明格式如下：

```java
public class ClassName {
    public static void main(String[] args) {
        program statements
    }
}
```

在Java中，每个**语句**(statement)必须用分号结束。特别需要说明，回车不是语句的结束标志，因此如果需要，一条语句可以跨多行。

这个`main()`方法体中只包含一条语句：将一个文本行输出到控制台。这里使用了`System.out`对象，并调用其`println()`方法。注意，点号(`.`)用于调用方法。Java的**方法**(method)调用语法是

```java
object.method(parameters)
```

在这里，`println()`方法接收一个字符串参数，用于将这个字符串显示在控制台上并换行。注意，Java使用双引号分隔字符串。

与其他编程语言中的函数一样，Java中的方法可以有零个、一个或多个**参数**(parameter)。即使一个方法没有参数，也必须使用空括号。例如，不带参数的`println()`方法只打印一个空行：

```java
System.out.println();
```

注释：`System.out`还有一个`print()`方法，它在输出之后不换行。

## 3.2 注释
与大多数编程语言一样，Java中的**注释**(comment)不会出现在可执行程序中。因此可以根据需要添加任意多的注释，而不必担心代码膨胀。在Java中有三种标记注释的方式。最常用的形式是`//`，注释内容从`//`开始到行结尾。

```java
System.out.println("We will not use 'Hello, World!'"); // is this too cute?
```

当需要更长的注释时，可以在每一行的前面加`//`，也可以使用`/*`和`*/`将一段比较长的注释括起来。

最后，第三种注释用于自动生成文档。这种注释以`/**`开始，以`*/`结束。在程序清单3-1中可以看到这种注释。有关这种注释以及自动生成文档的更多内容参见第4章。

警告：在Java中，`/* */`注释不能嵌套。也就是说，不能简单地使用`/*`和`*/`将代码注释掉，因为这段代码本身可能包含`*/`。

## 3.3 数据类型
Java是一种**强类型语言**，这意味着必须为每个变量声明一种类型。在Java中有8种**基本类型**(primitive type)：4种整数类型(`byte`, `short`, `int`, `long`)、2种浮点类型(`float`, `double`)、1种字符类型`char`和1种真值类型`boolean`。

### 3.3.1 整数类型
整数类型用于表示没有小数部分的数值，可以是负数。Java提供了4种整数类型，如下表所示。

| 类型 | 存储需求 | 取值范围 |
| --- | --- | --- |
| `byte` | 1字节 | -128~127 (-2<sup>7</sup>~2<sup>7</sup>-1) |
| `short` | 2字节 | -32768~32767 (-2<sup>15</sup>~2<sup>15</sup>-1) |
| `int` | 4字节 | -2147483648~2147483647 (-2<sup>31</sup>~2<sup>31</sup>-1) |
| `long` | 8字节 | -9223372036854775808~9223372036854775807 (-2<sup>63</sup>~2<sup>63</sup>-1) |

在大多数情况下，`int`类型最常用。但如果想要表示地球上的人口数，就需要使用`long`类型。`byte`和`short`类型主要用于特定的应用，例如底层文件处理。

在Java中，整型的范围与运行Java代码的机器无关。这解决了软件移植的主要问题。

长整型字面值使用后缀`L`或`l`表示（例如`4000000000L`）。十六进制数使用前缀`0x`或`0X`（例如`0xCAFE`，等于十进制的51966）。八进制数使用前缀`0`（例如`010`，等于十进制的8）。二进制数使用前缀`0b`或`0B`（例如`0b1001`就是9）。

可以为数字字面值添加下划线。例如`1_000_000`（或`0b1111_0100_0010_0100_0000`）表示一百万。这些下划线只是为了让人更易读，Java编译器会删除它们。

C++注释：在C和C++中，`int`和`long`等类型的大小取决于目标平台，这给编写跨平台的程序带来了很大难度。注意，Java没有任何无符号(unsigned)整数类型。

### 3.3.2 浮点类型
浮点类型用于表示有小数部分的数值。在Java中有两种浮点类型，如下表所示。

| 类型 | 存储需求 | 取值范围 |
| --- | --- | --- |
| `float` | 4字节 | 大约±3.4028235×10<sup>38</sup>（6~7位有效数字） |
| `double` | 8字节 | 大约±1.7976931348623157×10<sup>308</sup>（15位有效数字） |

名字`double`表示其精度是`float`类型的两倍，因此也叫**双精度**(double-precision)浮点数。很多情况下，`float`的精度都不能满足需求。只有当所使用的库需要单精度，或者需要存储大量数据时才使用`float`。

`float`字面值使用后缀`F`或`f`（例如`3.14F`）。没有后缀的浮点数（例如`3.14`）默认为`double`类型，但也可以添加后缀`D`或`d`（例如`3.14D`）。

`E`或`e`表示指数。例如，`1.729e3`等于1729。

注释：可以使用十六进制表示浮点数字面值。例如，0.125=1.0×2<sup>-3</sup>可以表示成`0x1.0p-3`。在十六进制表示法中，使用`p`而不是`e`表示指数。注意，尾数是十六进制，指数是十进制，基数是2而不是10。

所有的浮点计算都遵循IEEE 754规范。特别地，有3个特殊的浮点值表示溢出和错误：
* 正无穷大
* 负无穷大
* NaN (not a number)

例如，正浮点数除以0的结果为正无穷大（整数除以0会抛出`ArithmeticException`），计算0/0或负数的平方根结果为NaN。

注释：常量`Double.POSITIVE_INFINITY`、`Double.NEGATIVE_INFINITY`和`Double.NaN`（以及对应的`Float`常量）分别表示这三个特殊值。特别要说明的是，不能用`x == Double.NaN`来检测结果是否等于NaN，因为任何值都和NaN不相等（包括NaN本身）。应该使用`Double.isNaN(x)`。

警告：浮点数不适用于无法接受**舍入误差**的金融计算。例如，`System.out.println(2.0 - 1.1)`将打印0.8999999999999999，而不是期望的0.9。如果需要精确的数值计算，应该使用`BigDecimal`类，本章3.9节将介绍。

注：由于存在舍入误差，不要使用`==`检测两个浮点数是否相等。例如，`2.0 - 1.1 == 0.9`为`false`。应该使用`Math.abs(x - y) < 1e-6`。

### 3.3.3 char类型
`char`类型原本用于表示单个字符。然而，现在情况已经有所变化。如今，有些Unicode字符可以用一个`char`值表示，而其他Unicode字符则需要两个`char`值。有关详细信息参见下一节。

`char`类型的字面值要用单引号括起来。例如，`'A'`是编码值为65的字符常量。`char`类型的值可以表示为十六进制，范围从`\u0000`到`\uFFFF`（注意，Java的`char`类型长度为2字节）。例如，`\u2122`是商标符号(™)，`\u03C0`是希腊字母π。

注：Java的`\u`转义序列相当于C++和Python的`\x`转义序列（扩展到2字节）。而Python的`\u`转义序列用于直接指定Unicode码点。

除了`\u`转义序列外，还有一些用于表示特殊字符的转义序列，如下表所示。

| 转义序列 | 名称 | Unicode值 |
| --- | --- | --- |
| `\b` | 退格(backspace) | `\u0008` |
| `\t` | 制表(tab) | `\u0009` |
| `\n` | 换行(line feed) | `\u000A` |
| `\r` | 回车(carriage return) | `\u000D` |
| `\f` | 换页(form feed) | `\u000C` |
| `\"` | 双引号 | `\u0022` |
| `\'` | 单引号 | `\u0027` |
| `\\` | 反斜杠 | `\u005C` |
| `\s` | 空格（用于在文本块中保留末尾空白符） | `\u0027` |
| `\'` | 单引号 | `\u0020` |
| \newline | 只在文本块中使用，连接这一行和下一行 | - |

可以在字符字面值或字符串中使用这些转义序列，例如`'\u2122'`或`"Hello\n"`。`\u`转义序列甚至还可以在字符常量和字符串之外使用（但其他转义序列不可以）。例如：

```java
public static void main(String\u005B\u005D args)
```

是完全合法的——`\u005B`和`\u005D`分别是`[`和`]`的编码。

警告：Unicode转义序列会在解析代码之前处理。例如，`"\u0022+\u0022"`并不是由两个引号包围加号构成的字符串(`"\"+\""`)，而是会转换为`""+""`，也就是一个空串。

更隐秘地，一定要当心注释中的`\u`。例如，注释`// \u000A is a newline`会产生语法错误，因为`\u000A`会被替换为一个换行符。类似地，注释`// look inside c:\users`也会产生语法错误，因为`\u`后面并没有跟着4位十六进制数。

### 3.3.4 Unicode和char类型
[Unicode](https://en.wikipedia.org/wiki/Unicode)是一种国际编码标准，用于为世界上几乎所有语言中的每个字符分配统一的编码。

**码点**(code point)是指编码表中一个字符对应的编码值。在Unicode标准中，码点用十六进制书写，并加上前缀U+。例如，U+0041就是字母A的码点。Unicode的码点可以分成17个**代码平面**(code plane)。第一个代码平面称为**基本多语言平面**(basic multilingual plane)，由经典Unicode字符组成，码点从U+0000到U+FFFF。其余16个代码平面包含**补充字符**(supplementary character)，码点从U+10000到U+10FFFF。

注：Unicode字符的完整列表见 <https://symbl.cc/en/unicode-table/> 。

为了能够用二进制表示Unicode码点，同时避免浪费存储空间，需要一种将码点映射到**代码单元**(code unit)序列的变长编码规则。这种编码规则称为Unicode Transformation Format (UTF)，常见的有UTF-8、UTF-16和UTF-32。参考：
* [UTF-16 - Wikipedia](https://en.wikipedia.org/wiki/UTF-16)
* [Unicode FAQ](https://www.unicode.org/faq/utf_bom.html)

Java采用UTF-16编码。UTF-16编码使用1个或2个16位的代码单元表示一个码点。例如，希腊字母π的码点为U+03C0，编码为一个代码单元`\u03C0`；数学符号𝕆的码点为U+1D546，编码为两个代码单元`\uD835`和`\uDD46`。（编码算法的具体描述见[RFC 2781](https://datatracker.ietf.org/doc/html/rfc2781)）。

```java
jshell> '\u03C0'
$1 ==> 'π'

jshell> "\uD835\uDD46"
$2 ==> "𝕆"
```

在Java中，`char`类型描述了UTF-16编码中的一个代码单元（因为`char`类型刚好是16位）。

强烈建议不要在程序中使用`char`类型，除非确实需要处理UTF-16代码单元。最好将字符串当作抽象数据类型（见3.6节）。

### 3.3.5 boolean类型
`boolean`（布尔）类型只有两个值：`false`和`true`，用来判定逻辑条件。整数和布尔值之间**不能**相互转换。

## 3.4 变量和常量
与所有编程语言一样，Java使用**变量**(variable)来存储值。**常量**(constant)就是值不变的变量。

### 3.4.1 声明变量
在Java中，每个变量都有一个**类型**(type)。声明变量时，先指定类型，然后是变量名。下面是一些示例：

```java
double salary;
int vacationDays;
long earthPopulation;
boolean done;
```

注意每个声明都以分号结束。

作为变量名（以及其他名字）的标识符由字母、数字、货币符号($)以及下划线组成，不能以数字开头。字母区分大小写。标识符的长度基本上没有限制。

与大多数语言相比，Java中“字母”的范围更大，可以是一种语言中表示字母的**任何**Unicode字符。例如ä、π或汉字。但实际上，大多数程序员只使用A-Z、a-z、0-9和下划线。

提示：
* 如果想知道哪些Unicode字符可以用在标识符中，可以使用`Character`类的`isJavaIdentifierStart()`和`isJavaIdentifierPart()`方法来检查。
* 尽管$是一个合法的标识符字符，不要在自己的代码中使用它。它只用于Java编译器或其他工具生成的名字。

也不能使用Java关键字（例如`class`）作为变量名。

从Java 9开始，单下划线`_`是一个保留字。未来的版本可能用作通配符。

可以在一行中声明多个变量：

```java
int i, j; // both are integers
```

不过不提倡这种风格。分别声明每个变量可以提高程序的可读性。

提示：一般来说，不要让两个名字只有大小写区别。不过，有时确实很难给变量取一个好名字。许多程序员将变量命名为类型名。例如：

```java
Box box; // "Box" is the type and "box" is the variable name
```

### 3.4.2 初始化变量
千万不要使用未初始化的变量的值。例如，Java编译器会认为下面的语句是错误的：

```java
int vacationDays;
System.out.println(vacationDays); // ERROR--variable not initialized
```

使用等号(`=`)对已声明的变量进行赋值。

```java
int vacationDays;
vacationDays = 12;
```

也可以将变量的声明和初始化放在同一行。例如：

```java
int vacationDays = 12;
```

在Java中可以将声明放在代码中的任何地方。变量的声明尽可能靠近第一次使用的地方，这是一种良好的编程风格。

注释：从Java 10开始，如果可以从初始值推断出局部变量的类型，就无需声明类型，只需使用关键字`var`：

```java
var vacationDays = 12; // vacationDays is an int
var greeting = "Hello"; // greeting is a String
```

C++注释：C和C++区分变量的声明和定义，Java不区分。

### 3.4.3 常量
在Java中，使用关键字`final`指示常量。例如：

[Constants/Constants.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch03/Constants/Constants.java)

关键字`final`表示这个变量**只能被赋值一次**，一旦赋值就不能再更改了。习惯上，常量名使用全大写加下划线。

在Java中，经常需要一个常量可以在一个类的多个方法中使用，通常称为**类常量**(class constant)。使用关键字`static final`设置类常量。下面是一个使用类常量的示例：

[Constants/Constants2.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch03/Constants/Constants2.java)

注意，类常量的定义位于`main()`方法之外，从而同一个类的其他方法也可以使用这个常量。另外，如果一个常量被声明为`public`，那么其他类的方法也可以使用这个常量，例如这个例子中的`Constants2.CM_PER_INCH`。

C++注释：`const`是Java保留的关键字，但目前并没有使用。必须使用`final`定义常量。

注：Java的`final`常量并不完全等同于C++的`const`常量！
* C++常量必须初始化；而Java常量可以声明之后再赋值，但只能赋值一次。例如，在C++中：

```cpp
const int x;  // error: uninitialized const 'x'
x = 42;  // error: assignment of read-only variable 'x'
```

而在Java中：

```java
final int x;
x = 42;  // OK
x = 88;  // error: already assigned to 'x'
```

* C++中常量对象或数组本身及其成员或元素都不能修改；而Java中**可变类型**的常量对象或数组**本身**只能赋值一次，但字段或元素仍然可以修改（即Java中的`final T t;`相当于C++中的`T* const t`而不是`const T* t`）。例如，在C++中：

```cpp
const Point p = {3, 4};
p.x = 33;  // error: assignment of member 'Point::x' in read-only object
p = {6, 8};  // error: passing 'const Point' as 'this' argument discards qualifiers

const int a[] = {0, 1, 2, 3, 4};
a[2] = 22;  // error: assignment of read-only location 'a[2]'
```

而在Java中：

```java
final Point p = new Point(3, 4);
p.x = 33;  // OK
p = new Point(6, 8);  // error: cannot assign to final variable 'p'

final int[] a = {0, 1, 2, 3, 4};
a[2] = 22;  // OK
a = new int[] {5, 6, 7}; // error: cannot assign to final variable 'a'
```

### 3.4.4 枚举类型
有时，变量的取值应该是一个有限的集合。例如，销售的服装只有小、中、大和超大这四种尺寸。可以将这些尺寸编码为整数或字符，但很容易出错。

针对这种情况，可以自定义**枚举类型**(enumerated type)。枚举类型具有有限个命名的值。例如，

```java
enum Size { SMALL, MEDIUM, LARGE, EXTRA_LARGE }
```

现在，可以声明这种类型的变量：

```java
Size s = Size.MEDIUM;
```

`Size`类型的变量只能存储类型声明中所列出的值之一，或者特殊值`null`，表示这个变量没有设置任何值。

5.7节将更详细地讨论枚举类型。

## 3.5 运算符
### 3.5.1 算术运算符
在Java中，使用算术运算符`+`、`-`、`*`、`/`表示加、减、乘、除运算。当两个操作数都是整数时，`/`表示整数除法，否则表示浮点除法。整数求余（也称为**取模**(modulus)）用`%`表示。例如，`15 / 2`等于7，`15 % 2`等于1，`15.0 / 2`等于7.5。

### 3.5.2 数学函数和常量
`Math`类包含各种数学函数。

要计算一个数的平方根，可以使用`sqrt()`方法：

```java
double x = 4;
double y = Math.sqrt(x);
System.out.println(y); // prints 2.0
```

Java没有幂运算符，必须使用`Math.pow()`方法：`Math.pow(x, a)`返回x<sup>a</sup>。

整数的`/`和`%`运算符计算**向零取整**除法和对应的余数，而`Math.floorDiv()`和`Math.floorMod()`方法计算**向下取整**除法和对应的余数。例如：

| x | y | `x / y` | `x % y` | `Math.floorDiv(x, y)` | `Math.floorMod(x, y)` |
| --- | --- | --- | --- | --- | --- |
| 5 | 2 | 2 | 1 | 2 | 1 |
| -5 | 2 | -2 | -1 | -3 | 1 |
| 5 | -2 | -2 | 1 | -3 | -1 |
| -5 | -2 | 2 | -1 | 2 | -1 |

以下等式恒成立：
* `x == (x / y) * y + x % y`
* `x == Math.floorDiv(x, y) * y + Math.floorMod(x, y)`

注：另见[【Python】除法舍入问题]({% post_url 2021-03-20-python-division-rounding %})。

`Math`类提供的常用的三角函数：`sin()`、`cos()`、`tan()`、`asin()`、`acos()`、`atan()`。

还有指数函数和对数函数：`exp()`、`log()`、`log10()`。

最后，还提供了两个常量来表示π和e的近似值：`Math.PI`和`Math.E`。

提示：不必在数学函数和常量名前添加前缀`Math`，只要在源文件顶部加上下面这行代码即可。4.8.3节将介绍静态导入。

```java
import static java.lang.Math.*;
```

### 3.5.3 数值类型之间的转换
通常，将一种数值类型转换为另一种数值类型是有必要的。下图给出了数值类型之间的合法转换。

![数值类型之间的合法转换](/assets/images/java-note-v1ch03-fundamental-programming-structures-in-java/数值类型之间的合法转换.png)

实线箭头表示无信息丢失的转换，虚线箭头表示可能有精度损失的转换。例如：

```java
int n = 123456789;
float f = n; // f is 1.23456792E8
```

当使用二元运算符组合两个值时（例如`n + f`），先要将两个操作数转换为同一种类型，然后再进行计算。
* 如果其中一个操作数是`double`类型，则将另一个转换为`double`类型。
* 否则，如果其中一个操作数是`float`类型，则将另一个转换为`float`类型。
* 否则，如果其中一个操作数是`long`类型，则将另一个转换为`long`类型。
* 否则，将两个操作数都转换为`int`类型。

### 3.5.4 强制类型转换
在必要时，`int`类型的值会自动转换为`double`类型。另一方面，有时也需要将`double`转换成`int`。这种可能丢失信息的转换要通过**强制类型转换**(cast)来完成。强制类型转换的语法是：在圆括号中指定目标类型，后面跟着变量名。例如：

```java
double x = 9.997;
int nx = (int) x; // nx is 9
```

浮点数转换为整数会丢弃小数部分。

如果想将一个浮点数**舍入**(round)为最接近的整数，可以使用`Math.round()`方法：

```java
double x = 9.997;
int nx = (int) Math.round(x); // nx is 10
```

注意，当参数为`double`类型时，`round()`的返回值为`long`，因此仍然需要使用强制类型转换`(int)`。

警告：如果将一个数值强制转换为另一种类型，而又超出了目标类型的表示范围，结果就会截断成一个完全不同的值。例如，`(byte) 300`实际上为`44`（`0x0000012c`截断为`0x2c`）。

C++注释：Java禁止在`boolean`与任何数值类型之间转换，这样可以防止一些常见的错误。只有极少数情况下需要将`boolean`值转换为数字，可以使用条件表达式`b ? 1 : 0`。

### 3.5.5 赋值
在赋值中使用二元运算符有一种方便的简写形式。例如，`x += 4;`等价于`x = x + 4;`。

警告：如果运算符的结果类型与左侧变量不同，就会发生强制类型转换。例如，如果`x`是一个`int`，则`x += 3.5;`等价于`x = (int)(x + 3.5);`。

注意，在Java中赋值是一个**表达式**(expression)，其值为所赋的那个值。考虑以下语句：

```java
int x = 1;
int y = x += 4; // y is 5
```

### 3.5.6 自增和自减运算符
Java提供了自增、自减运算符：`n++`将`n`的值加1，`n--`将`n`的值减1。例如：

```java
int n = 12;
n++; // n is 13
```

由于这些运算符会改变变量的值，因此不能用于数字本身。例如，`4++`就不是一个合法的语句。

这些运算符还有前缀形式：`++n`和`--n`。前缀和后缀形式都会使变量的值改变1，但用在表达式中时二者就有区别了：前缀形式先加1再使用变量的值，后缀形式先使用变量的值再加1。

```java
int m = 7;
int n = 7;
int a = 2 * ++m; // now a is 16, m is 8
int b = 2 * n++; // now b is 14, n is 8
```

这种行为容易让人困惑。在Java中，很少在表达式中使用`++`。

### 3.5.7 关系和逻辑运算符
要检测相等，使用两个等号`==`。例如，`3 == 7`的值为`false`。使用`!=`检测不相等。例如，`3 != 7`的值为`true`。另外，还有经常使用的`<`、`>`、`<=`和`>=`。

Java使用`&&`表示逻辑“与”运算符，`||`表示逻辑“或”运算符，`!`是逻辑“非”运算符。`&&`和`||`运算符是按照“短路”方式求值的：如果第一个操作数已经能够确定表达式的值，则不计算第二个操作数。

对于表达式`expr1 && expr2`，如果`expr1`的值为`false`，则结果一定为`false`，因此不会计算`expr2`的值。可以利用这种行为来避免错误。例如：

```java
x != 0 && 1 / x > x + y // no division by 0
```

类似地，对于表达式`expr1 || expr2`，如果`expr1`的值为`true`，则结果一定为`true`，不会计算`expr2`。

### 3.5.8 条件运算符
Java提供了条件运算符`?:`，可以根据条件选择一个值。当条件为真时，表达式`condition ? expr1 : expr2`的值为`expr1`，否则为`expr2`。例如，`x < y ? x : y`返回`x`和`y`中较小者。

### 3.5.9 switch表达式
需要在两个以上的值中选择时，可以使用`switch`表达式（Java 14引入）。例如：

```java
String seasonName = switch (seasonCode) {
    case 0 -> "Spring";
    case 1 -> "Summer";
    case 2 -> "Fall";
    case 3 -> "Winter";
    default -> "???";
};
```

`case`标签还可是字符串或枚举常量。

可以为每个`case`提供多个标签，用逗号分隔：

```java
int numLetters = switch (seasonName) {
    case "Spring", "Summer", "Winter" -> 6;
    case "Fall" -> 4;
    default -> -1;
};
```

在`switch`表达式中使用枚举常量时，不需要为每个标签提供枚举名，这可以从`switch`值推导出来。例如：

```java
enum Size { SMALL, MEDIUM, LARGE, EXTRA_LARGE };
...
Size itemSize = ...;
String label = switch (itemSize) {
    case SMALL -> "S"; // no need to use Size.SMALL
    case MEDIUM -> "M";
    case LARGE -> "L";
    case EXTRA_LARGE -> "XL";
};
```

警告：
* 在`switch`表达式中使用整数或字符串时，必须有`default`；使用枚举值时，如果`case`标签覆盖了所有可能的取值，则可以省略`default`，否则不能省略。
* 如果操作数为`null`，则抛出`NullPointerException`。

注释：从Java 14起，`switch`有四种形式。3.8.5节全面讨论了`switch`表达式和语句的所有形式。

### 3.5.10 位运算符
对于整数类型，还有一些运算符可以直接处理整数的二进制**位**(bit)。位运算符包括`&`（与）、`|`（或）、`^`（异或）和`~`（取反）。

可以使用掩码(masking)技术来获取数字中的单个二进制位。例如，如果`n`是一个整数，则当`n`的二进制表示中从右边数第4位为1时，下面的结果为1，否则为0。

```java
int fourthBitFromRight = (n & 0b1000) / 0b1000;
```

注：另见[《C程序设计语言》笔记 第2章]({% post_url 2022-02-24-tcpl-note-ch2-types-operators-and-expressions %}) 2.9节。

注释：`&`和`|`运算符用于布尔值上时也会得到一个布尔值。这些运算符类似于`&&`和`||`，但不采用“短路”方式求值，也就是说，两个操作数都需要计算。

另外，还有移位运算符`>>`和`<<`可以将位模式右移或左移。这些运算符在需要建立位模式来完成位掩码时会很方便：

```java
int fourthBitFromRight = (n & (1 << 3)) >> 3;
```

逻辑右移运算符`>>>`用0填充最高位，而算术右移运算符`>>`用符号位填充最高位。没有`<<<`运算符。

警告：移位运算符的左操作数是`int`类型时，右操作数会模32；左操作数是`long`类型是，右操作数会模64；左操作数是其他整数类型时，自动转换为`int`类型。例如，`1 << 35`等同于`1 << 3`或`8`。

### 3.5.11 括号和运算符级别
下表给出了运算符的优先级（从高到低）。如果不使用圆括号，就按照运算符优先级次序进行计算。同一个级别的运算符按照从左到右的次序进行计算（右结合运算符除外）。例如，`a && b || c`等价于`(a && b) || c`，`a += b += c`等价于`a += (b += c)`。

| 运算符 | 结合性 |
| --- | --- |
| `[] () .` | 从左到右 |
| `! ~ ++ -- +（一元） -（一元） (type) new` | 从右到左 |
| `* / %` | 从左到右 |
| `+ -` | 从左到右 |
| `<< >> >>>` | 从左到右 |
| `< <= > >= instanceof` | 从左到右 |
| `== !=` | 从左到右 |
| `&` | 从左到右 |
| `^` | 从左到右 |
| `|` | 从左到右 |
| `&&` | 从左到右 |
| `||` | 从左到右 |
| `?:` | 从右到左 |
| `= += -= *= /= %= &= |= ^= <<= >>= >>>=` | 从右到左 |

## 3.6 字符串
从概念上讲，Java字符串就是Unicode字符序列。Java标准库提供了字符串类`String`。每个用双引号括起来的字符串都是`String`类的一个实例。

```java
String e = ""; // an empty string
String greeting = "Hello";
```

### 3.6.1 子串
`String`类的`substring()`方法可以提取子串。

```java
String greeting = "Hello";
String s = greeting.substring(0, 3); // "Hel"
```

两个参数分别是开始位置（包含）和结束位置（不包含）。字符串`s.substring(a, b)`的长度为`b - a`。

### 3.6.2 拼接
Java允许使用`+`连接（拼接）两个字符串。

```java
String expletive = "Expletive";
String PG13 = "deleted";
String message = expletive + PG13; // "Expletivedeleted"
```

将一个字符串与一个非字符串的值进行拼接时，后者将被转换成字符串（在第5章中可以看到，任何Java对象都可以转换成字符串）。例如：

```java
int age = 13;
String rating = "PG" + age; // "PG13"
```

这种特性通常用在输出语句中。例如：

```java
System.out.println("The answer is " + answer);
```

如果需要把多个字符串放在一起，用分隔符分隔，可以使用静态方法`join()`：

```java
String all = String.join(" / ", "S", "M", "L", "XL"); // "S / M / L / XL"
```

在Java 11中，还提供了`repeat()`方法：

```java
String repeated = "Java".repeat(3); // "JavaJavaJava"
```

### 3.6.3 字符串不可变
`String`类没有提供用于修改字符串中字符的方法。如果希望将`greeting`的内容修改为`"Help!"`，不能直接将最后两个字符修改为`'p'`和`'!'`，需要将保留的子串拼接上替换的字符串：

```java
String greeting = "Hello";
greeting = greeting.substring(0, 3) + "p!";
```

由于不能修改Java字符串中的字符，所以在Java文档中将`String`类的对象称为**不可变的**(immutable)。如同数字`3`永远是3，字符串`"Hello"`永远包含字符H、e、l、l、o的代码单元序列。你不能修改这些值。不过，可以修改字符串**变量**的内容，让它**引用另外一个字符串**（如下图所示）。

![给字符串变量赋值](/assets/images/java-note-v1ch03-fundamental-programming-structures-in-java/给字符串变量赋值.png)

不可变字符串有一个很大的优点：编译器可以让字符串**共享**。Java的设计者认为共享带来的高效率远超过编辑字符串（提取子串和拼接）带来的低效率。实际上，大多数时候都不会修改字符串，而只是进行比较。（有一种例外情况，将单个字符或较短字符串组装成更大的字符串。为此，Java提供了一个单独的类，见3.6.9节）

### 3.6.4 检测字符串是否相等
使用`equals()`方法检测两个字符串是否相等。如果字符串`s`和`t`相等，则表达式`s.equals(t)`返回`true`，否则返回`false`。注意，`s`和`t`可以是字符串变量，也可以是字符串字面值。例如，表达式`"Hello".equals(greeting)`是合法的。

要检测两个字符串是否相等，而不区分大小写，使用`equalsIgnoreCase()`方法。

**不要使用`==`运算符检测两个字符串是否相等！** 这个运算符只能够确定两个字符串是否存放在同一个位置上（是否是同一个对象）。如果两个字符串存放在同一个位置上，它们必然相等。但是，完全有可能将多个内容相同的副本存放在不同位置上。

```java
String greeting = "Hello"; // initialize greeting to a string
if (greeting == "Hello") ...
  // probably true
if (greeting.substring(0, 3) == "Hel") ...
  // probably false
```

如果Java虚拟机始终将内容相同的字符串共享，就可以使用`==`检测相等。但实际上只有字符串字面值是共享的，而`+`或`substring()`等操作得到的结果并不共享。因此，千万不要使用`==`比较字符串，以免在程序中出现最糟糕的一种bug——间歇性地随机出现。

### 3.6.5 空串和Null串
空串`""`是长度为0的字符串。可以使用`s.length() == 0`、`s.equals("")`或`s.isEmpty()`检查一个字符串是否为空。

`String`变量还可以存放一个特殊值，名为`null`，表示目前没有任何对象与该变量关联（详见4.2.1和4.3.6节）。要检测一个字符串是否为`null`，使用`s == null`。

有时要检查一个字符串既不是`null`也不是空串，可以使用`s != null && s.isEmpty()`。

### 3.6.6 码点和代码单元
Java字符串是`char`值序列。在3.3.3节已经看到，`char`类型用于表示采用UTF-16编码的Unicode码点的代码单元。最常用的Unicode字符可以用一个代码单元表示，而补充字符（例如Emoji）需要两个代码单元。

`length()`方法返回字符串的代码单元（`char`值）个数。例如：

```java
String greeting = "Hello";
int n = greeting.length(); // is 5
String sentence = "🍺Beer";
int m = sentence.length(); // is 6
```

![字符串和代码单元](/assets/images/java-note-v1ch03-fundamental-programming-structures-in-java/字符串和代码单元.png)

要得到实际长度，即码点个数，调用`codePointCount()`：

```java
int cpCount = greeting.codePointCount(0, greeting.length()); // is 5
int cpCount2 = sentence.codePointCount(0, sentence.length()); // is 5
```

调用`s.charAt(n)`返回位置`n`的代码单元（可能不是一个完整字符），`n`介于0~`s.length()-1`之间。例如：

```java
char first = greeting.charAt(0); // first is 'H'
char last = greeting.charAt(4); // last is 'o'
char ch = sentence.charAt(1); // ch is '\uDF7A'
```

注意：`ch`不是`'B'`，而是🍺的第二个代码单元。

要得到第i个码点，使用以下语句：

```java
int index = greeting.offsetByCodePoints(0, i);
int cp = greeting.codePointAt(index);
```

`codePointAt()`的返回值是Unicode码点，用`int`表示。例如：

```java
int cp1 = sentence.codePointAt(sentence.offsetByCodePoints(0, 0)); // 0x1F37A (🍺)
int cp2 = sentence.codePointAt(sentence.offsetByCodePoints(0, 1)); // 0x42 (B)
```

如果要遍历一个字符串，并依次查看每个码点，可以使用以下语句：

```java
int i = 0;
while (i < str.length()) {
    int cp = str.codePointAt(i);
    // do something with cp
    if (Character.isSupplementaryCodePoint(cp)) i += 2;
    else i++;
}
```

可以使用以下语句反向遍历：

```java
int i = str.length();
while (i > 0) {
    i--;
    if (Character.isSurrogate(str.charAt(i))) i--;
    int cp = str.codePointAt(i);
    // do something with cp
}
```

显然，这很麻烦。更容易的办法是使用`codePoints()`方法，得到码点数组。

```java
int[] codePoints = str.codePoints().toArray();
```

反之，要把一个码点数组转换为字符串，使用构造器。

```java
String str = new String(codePoints, 0, codePoints.length);
```

要把单个码点转换为字符串，可以使用`Character.toString()`方法。

```java
int codePoint = 0x1F37A;
String str = Character.toString(codePoint); // "🍺"
```

注释：Java虚拟机不一定把字符串实现为`char`数组。Java 9使用了一种更紧凑的表示：只包含单字节代码单元的字符串使用`byte`数组，其他字符串使用`char`数组。

### 3.6.7 String API
Java中的`String`类包含近100个方法。完整列表参见[在线文档](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/String.html)。

### 3.6.8 阅读在线API文档
标准库中有上千个类，方法数量更加惊人。要想记住所有有用的类和方法显然不太可能。因此，学会使用在线API文档十分重要，从中可以查找标准库的所有类和方法。可以在浏览器中访问 <https://docs.oracle.com/en/java/javase/17/docs/api/index.html> 。

### 3.6.9 构建字符串
有时，需要由较短的字符串（例如按键或文件中的单词）构建字符串。采用字符串拼接的方式达到这个目的效率比较低。每次拼接字符串时，都会构建一个新的`String`对象，既耗时又浪费内存。使用`StringBuilder`类可以避免这个问题。

首先，构建一个空的`StringBuilder`：

```java
StringBuilder builder = new StringBuilder();
```

调用`append()`方法添加字符或字符串：

```java
builder.append(ch); // appends a single character
builder.append(str); // appends a string
```

字符串构建完成后，调用`toString()`方法，得到一个`String`对象。

```java
String completedString = builder.toString();
```

注：
* `StringBuilder`类的`append()`方法返回当前对象，这意味着可以链式调用。例如`builder.append(ch).append(str)`。
* Java的`StringBuilder`类似于C++的`std::ostringstream`。

注释：在JDK 5.0引入`StringBuilder`前，还有一个具有相同功能的`StringBuffer`类。其效率不如`StringBuilder`，但允许多线程添加或删除字符（即线程安全）。这两个类的API的一样的。

### 3.6.10 文本块
Java 15添加了**文本块**(text block)特性，可以很容易地提供跨多行的字符串字面值。文本块以`"""`开始，后面是一个换行符，并以另一个`"""`结束。

```java
String greeting = """
Hello
World
""";
```

文本块比相应的字符串字面值更易于读写：

```java
"Hello\nWorld\n"
```

开始`"""`后面的换行符不包含在字符串字面值中。如果不想要最后一行后面的换行符，可以让结束`"""`紧跟在最后一个字符后面：

```java
String prompt = """
Hello, my name is Hal.
Please enter your name: """;
```

文本块特别适合用于包含其他语言的代码，例如SQL或HTML。

```java
String html = """
<div class="Warning">
    Beware of those who say "Hello" to the world
</div>
""";
```

注意，一般不用对引号转义。只有两种情况下需要对引号转义：
* 文本块以引号结尾。
* 文本块包含三个或更多引号的序列。

遗憾的是，**所有反斜杠仍然需要转义**。

常规字符串的所有转义序列都可以在文本块中使用（即Java的文本块相当于Python的长字符串而不是原始字符串）。有一个转移序列只能在文本块中使用：行尾的`\`会把这一行和下一行连接起来。例如，

```java
"""
Hello, my name is Hal. \
Please enter your name: """;
```

等同于

```java
"Hello, my name is Hal. Please enter your name: "
```

文本块会对行尾进行标准化，删除末尾的空白符，并将Windows换行符(`\r\n`)改为简单换行符(`\n`)。如果确实需要保留末尾的空格，可以把最后一个空格替换为`\s`。

对于前导空白符更为复杂。考虑一个典型的变量声明，文本块也可以缩进：

```java
String html = """
    <div class="Warning">
        Beware of those who say "Hello" to the world
    </div>
    """;
```

文本块中所有行的公共缩进将被去除。因此实际字符串为

```java
"<div class=\"Warning\">\n    Beware of those who ... \" to the world\n</div>\n"
```

警告：当心缩进文本块的公共前缀中混用制表符和空格的情况（一个制表符只能匹配一个空格）。例如，

```java
String s = """
    This line is indented with spaces.
	This line is indented with tab.
    """;
```

实际字符串为（注意第一行开头是三个空格）

```java
"   This line is indented with spaces.\nThis line is indented with tab.\n"
```

提示：如果文本块中包含非Java代码，最好沿左边界对齐。这样可以与Java代码区分开，而且可以为长代码行留出更多空间。

## 3.7 输入和输出
### 3.7.1 读取输入
前面已经看到，将输出打印到“标准输出流”`System.out`（即控制台窗口）非常容易，只需调用`System.out.println()`。读取“标准输入流”`System.in`就没有那么简单了。要读取控制台输入，首先需要构造一个与`System.in`关联的`Scanner`。

```java
Scanner in = new Scanner(System.in);
```

现在，就可以使用`Scanner`类的各种方法读取输入了。例如，`nextLine()`方法读取下一行（丢弃换行符）。

```java
System.out.print("What is your name? ");
String name = in.nextLine();
```

要读取一个单词（以空白符分隔），调用`next()`。

```java
String firstName = in.next();
```

要读取一个整数，使用`nextInt()`方法。

```java
System.out.print("How old are you? ");
int age = in.nextInt();
```

类似地，`nextDouble()`方法读取下一个浮点数。

程序清单3-2的程序首先询问用户姓名和年龄，然后打印一条消息。最后，注意程序开头的`import`指令。`Scanner`类定义在`java.util`包中。当使用的类不是定义在基本`java.lang`包中时，需要使用`import`指令。有关包和`import`指令的细节参见第4章。

[程序清单3-2 InputTest/InputTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch03/InputTest/InputTest.java)

注释：`Scanner`类不适合从控制台读取密码，因为输入是可见的。可以使用`Console`类的`readPassword()`方法达到这个目的。

注：
* `Scanner`类的每个`nextXxx()`方法都有一个对应的`hasNextXxx()`方法，用于判断输入中是否还有下一个值，但不读取任何输入。
* `nextXxx()`方法会先跳过空白符，而只有`nextLine()`会在读取完成后将换行符读走。例如输入`" 123\nabc\n"`，调用`nextInt()`返回`123`（缓冲区剩余`"\nabc\n"`），然后调用`nextLine()`返回空串（缓冲区剩余`"abc\n"`），再次调用`nextLine()`返回`"abc"`。
* `nextXxx()`方法优先从缓冲区读取输入，当缓冲区为空时等待控制台输入。如果遇到输入结尾(EOF)则抛出`NoSuchElementException`，如果输入数据不是期望的格式则抛出`InputMismatchException`。因此，调用`nextXxx()`前应该先用`hasNextXxx()`检查，或者捕获异常。

### 3.7.2 格式化输出
可以使用`System.out.print(x)`将数值`x`打印到控制台。这个命令将以最大非0位数打印`x`。例如，`System.out.print(10000.0 / 3.0);`将打印3333.3333333333335。

要自定义格式，可以使用`printf()`方法，它沿用了C语言库的约定。例如，调用`System.out.printf("%8.2f", x);`以8个字符的**字段宽度**(field width)和2位小数的**精度**(precision)打印浮点数`x`，输出 " 3333.33" 。

可以为`printf()`提供多个参数。例如：

```java
System.out.printf("Hello, %s. Next year, you'll be %d", name, age + 1);
```

每个以`%`字符开头的**格式说明符**(format specifier)都替换为相应的参数。格式说明符末尾的**转换字符**(conversion character)指示要格式化的值的类型：`d`表示十进制整数，`f`表示浮点数，`s`表示字符串。下表列出了所有的转换字符。

| 转换字符 | 类型 | 示例 |
| --- | --- | --- |
| `d` | 十进制整数 | 159 |
| `x`或`X` | 十六进制整数 | 9f |
| `o` | 八进制整数 | 237 |
| `f` | 定点浮点数 | 15.9 |
| `e`或`E` | 指数浮点数（科学记数法） | 1.59e+01 |
| `g`或`G` | 通用浮点数（`e`和`f`中较短者） | - |
| `a`或`A` | 十六进制浮点数 | 0x1.fccdp3 |
| `s`或`S` | 字符串 | Hello |
| `c`或`C` | 字符 | H |
| `b`或`B` | 布尔值 | true |
| `h`或`H` | 散列码 | 42628b2 |
| `t`或`T` | 日期时间（遗留，应使用`java.time`） | - |
| `%` | 百分号 | % |
| `n` | 平台相关的换行符 | - |

注释：可以使用`%s`格式化任意对象。如果对象实现了`Formattable`接口，则调用`formatTo()`方法。否则，调用`toString()`方法将对象转换为字符串。

注：
* 格式说明符的语法为`%[argument_index$][flags][width][.precision]conversion`，详细说明参见Java文档[Format String Syntax](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Formatter.html#syntax)。
* 关于C语言的`printf()`函数参见[《C程序设计语言》笔记第7章]({% post_url 2022-06-02-tcpl-note-ch7-input-and-output %}) 7.2节。

另外，还可以指定控制格式化输出的各种**标志**(flag)。下表列出了所有的标志。

| 标志 | 作用 | 示例 |
| --- | --- | --- |
| `+` | 打印正数和负数的符号 | +3333.33 |
| 空格 | 在正数前添加空格 | \| 3333.33\| |
| `0` | 添加前导0 | 003333.33 |
| `-` | 左对齐 | \|3333.33 \| |
| `(` | 将负数括在括号内 | (3333.33) |
| `,` | 添加分组分隔符 | 3,333.33 |
| `#`（对于`f`） | 始终包含小数点（当精度为0时，例如`%#.0f`） | 3333. |
| `#`（对于`x`或`o`） | 添加前缀0x或0 | 0xcafe |
| `$` | 指定要格式化的参数索引（从1开始）<br>（例如，`%1$d %1$x`以十进制和十六进制打印第一个参数） | 159 9f |
| `<` | 格式化前面指定的同一个值<br>（例如，`%d %<x`以十进制和十六进制打印同一个数） | 159 9f |

例如，逗号标志添加分组分隔符。即`System.out.printf("%,.2f", 10000.0 / 3.0);`会打印 "3,333.33" 。

可以使用多个标志。例如，`%,(.2f`添加分组分隔符，并将负数括在括号内。

可以使用静态方法`String.format()`创建一个格式化的字符串，而不打印：

```java
String message = String.format("Hello, %s. Next year, you'll be %d", name, age + 1);
```

注释：从Java 15起，可以使用`formatted()`方法，可以少敲5个字符：

```java
String message = "Hello, %s. Next year, you'll be %d".formatted(name, age + 1);
```

### 3.7.3 文件输入和输出
要读取一个文件，需要用文件名构造一个`Scanner`对象，如下所示：

```java
Scanner in = new Scanner(Path.of("myfile.txt"), StandardCharsets.UTF_8);
```

注：字符编码参数也可以是字符串，例如`"UTF-8"`。

如果文件名包含反斜杠，则需要转义，例如`"C:\\mydirectory\\myfile.txt"`。

现在就可以使用之前介绍过的`Scanner`方法读取文件了。

注释：读取文本文件时，需要知道它的字符编码。如果省略字符编码，则会使用运行这个Java程序的机器的“默认编码”（由`Charset.defaultCharset()`指定）。这不是一个好主意，程序的行为可能因运行的机器而异。

警告：可以使用一个字符串参数来构造`Scanner`，但`Scanner`会把字符串解释为要读取的数据，而不是文件名。

要写入一个文件，需要构造一个`PrintWriter`，并提供文件名和字符编码：

```java
PrintWriter out = new PrintWriter("myfile.txt", StandardCharsets.UTF_8);
```

如果文件不存在则创建。可以像输出到`System.out`一样使用`print()`、`println()`和`printf()`方法。

注释：当指定相对文件名时（例如`"myfile.txt"`、`"mydirectory/myfile.txt"`或`"../myfile.txt"`），将相对于启动Java程序的目录来定位文件。如果从命令行运行程序，启动路径就是命令行的当前工作目录；如果使用IDE，启动目录由IDE控制。可以使用`System.getProperty("user.dir")`找到启动目录。如果觉得文件定位太麻烦，可以考虑使用绝对路径，例如`"C:\\mydirectory\\myfile.txt"`或者`"/home/me/mydirectory/myfile.txt"`。

如你所见，访问文件与使用`System.in`和`System.out`一样容易。要记住一点：如果用一个不存在的文件构造`Scanner`，或者用一个无法创建的文件名构造`PrintWriter`，就会产生异常。在第7章中将会学习各种处理异常的方法。目前，只需告诉编译器：你已经知道有可能出现“输入/输出”异常。为此，用一个`throws`子句标记`main()`方法，如下所示：

```java
public static void main(String[] args) throws IOException {
    Scanner in = new Scanner(Path.of("myfile.txt"), StandardCharsets.UTF_8);
    ...
}
```

注释：从命令行启动程序时，可以使用重定向语法将文件关联到标准输入和标准输出，这样就不必担心处理`IOException`异常了。例如：

```shell
java MyProg < myfile.txt > output.txt
```

其中，`<`表示将标准输入重定向到文件myfile.txt，`>`表示将标准输出重定向到文件output.txt。

## 3.8 控制流
### 3.8.1 块作用域
**块**(block)即**复合语句**(compound statement)，由若干条Java语句组成，并用一对大括号括起来。块确定了变量的作用域。一个块可以**嵌套**在另一个块中。例如，下面是嵌套在`main()`方法块中的一个块：

```java
public static void main(String[] args) {
    int n;
    ...
    {
        int k;
        ...
    } // k is only defined up to here
}
```

不能在嵌套的两个块中声明同名的变量。例如，下面的代码无法通过编译：

```java
public static void main(String[] args) {
    int n;
    ...
    {
        int k;
        int n; // ERROR--can't redeclare n in inner block
        ...
    }
}
```

### 3.8.2 条件语句
在Java中，条件语句的形式为

```java
if (condition) statement
```

如果条件为真，则执行后面的语句。

如果希望执行多条语句，可以使用语句块。例如：

```java
if (yourSales >= target) {
    performance = "Satisfactory";
    bonus = 100;
}
```

更一般的条件语句形式如下

```java
if (condition) statement1 else statement2
```

如果条件为真，则执行语句1，否则执行语句2。例如：

```java
if (yourSales >= target) {
    performance = "Satisfactory";
    bonus = 100 + 0.01 * (yourSales - target);
}
else {
    performance = "Unsatisfactory";
    bonus = 0;
}
```

`else`部分是可选的。`else`子句与最临近的`if`匹配。因此，在以下语句中

```java
if (x <= 0) if (x == 0) sign = 0; else sign = -1;
```

`else`与第二个`if`匹配。当然，使用大括号和缩进会让这段代码更加清晰：

```java
if (x <= 0) {
    if (x == 0)
        sign = 0;
    else
        sign = -1;
}
```

反复使用`if ... else if ...`很常见。例如：

```java
if (yourSales >= 2 * target) {
    performance = "Excellent";
    bonus = 1000;
}
else if (yourSales >= 1.5 * target) {
    performance = "Fine";
    bonus = 500;
}
else if (yourSales >= target) {
    performance = "Satisfactory";
    bonus = 100;
}
else {
    System.out.println("You're fired");
}
```

注：实际上Java并没有 "else-if" 语法，上面的第二个`if`及以后的部分都属于第一个`else`子句。

### 3.8.3 while循环
`while`循环在条件为真时重复执行一条语句（可以是语句块）。一般形式为

```java
while (condition) statement
```

如果开始时条件就为假，那么语句一次也不执行。

程序清单3-3中的程序计算存下一定数量的退休金需要多长时间，假定每年存入相同金额，而且利率是固定的。

[程序清单3-3 Retirement/Retirement.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch03/Retirement/Retirement.java)

`while`循环在最前面检测条件，因此循环体中的代码有可能一次都不执行。如果希望循环体至少执行一次，需要使用`do/while`循环，将检测放在最后。其语法如下：

```java
do statement while (condition);
```

这种循环先执行语句（通常是语句块），然后再检查条件。如果条件为真，则重复执行语句，然后再次检查条件，以此类推。

在程序清单3-4中，每次循环首先计算退休账户中新的余额，然后询问是否打算退休。

[程序清单3-4 Retirement2/Retirement2.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch03/Retirement2/Retirement2.java)

只要用户回答 "N" ，循环就会重复执行。这是一个需要至少执行一次的循环的很好的例子，因为用户必须先看到余额才能决定是否满足退休所用。

### 3.8.4 for循环
`for`循环由一个计数器或者类似的变量控制循环次数，每次循环后更新这个变量。例如，下面的循环打印数字1~10：

```java
for (int i = 1; i <= 10; i++)
    System.out.println(i);
```

`for`语句的第一部分通常是对计数器初始化；第二部分给出每次循环前要检测的条件；第三部分指定如何更新计数器。

尽管Java允许在`for`循环的各个部分放置任何表达式，但有一条不成文的规则：`for`循环的三个部分应该对**同一个**计数器变量进行初始化、检测和更新。若不遵守这一规则，编写的循环可能非常晦涩难懂。

即使受这个规则所限，仍有无尽可能。例如，可以编写倒计数的循环：

```java
for (int i = 10; i > 0; i--)
    System.out.println("Counting down ... " + i);
System.out.println("Blastoff!");
```

警告：小心在循环中检测浮点数相等。循环`for (double x = 0; x != 10; x += 0.1)`可能永远不会结束。因为0.1无法精确地用二进制表示，`x`将从9.99999999999998跳到10.09999999999998。因此循环条件应改为`x < 10`。

如果在`for`语句的第一部分或循环体内部定义一个变量，这个变量的作用域就是整个`for`循环体，不能在循环之外使用。

```java
for (int i = 1; i <= 10; i++) {
    ...
}
// i no longer defined here
```

因此，如果希望在`for`循环之外使用循环计数器的最终值，就要确保在循环外部声明这个变量。

```java
int i;
for (i = 1; i <= 10; i++) {
    ...
}
// i is still defined here
```

另一方面，可以在不同的`for`循环中定义同名的变量。

`for`循环只是`while`循环的一种简化形式（除非循环体包含`continue`语句）。例如，

```java
for (int i = 10; i > 0; i--)
    System.out.println("Counting down ... " + i);
```

可以重写为

```java
int i = 10;
while (i > 0) {
    System.out.println("Counting down ... " + i);
    i--;
}
```

程序清单3-5给出了一个`for`循环的典型示例。这个程序计算抽奖的中奖概率。一般地，如果从n个数字中选择k个，有 $C_n^k$ 种可能的组合。

$$
C_n^k = \frac{n(n-1)(n-2)...(n-k+1)}{1 \times 2 \times 3 \times ... \times k}
$$

[程序清单3-5 LotteryOdds/LotteryOdds.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch03/LotteryOdds/LotteryOdds.java)

注：`for`循环体中的`lotteryOdds = lotteryOdds * (n - i + 1) / i;`确实是整数除法，但每次循环`lotteryOdds`都可以整除`i`。如果改成`lotteryOdds *= (n - i + 1) / i;`结果就不对了。

注释：3.10.3节将会介绍for each循环。

### 3.8.5 多重选择：switch语句
在处理同一个表达式的多个选项时，`if/else`结构会显得有些笨拙。`switch`语句会让这个工作变得容易，特别是使用Java 14引入的形式。

例如，如果建立一个菜单系统，其中包含4个选项，可以使用以下代码：

```java
Scanner in = new Scanner(System.in);
System.out.print("Select an option (1, 2, 3, 4) ");
int choice = in.nextInt();
switch (choice) {
    case 1 ->
        ...
    case 2 ->
        ...
    case 3 ->
        ...
    case 4 ->
        ...
    default ->
        System.out.println("Bad input");
}
```

`case`标签可以是：
* 整型常量表达式
* 枚举常量
* 字符串字面值
* 多个上述标签，用逗号分隔

例如：

```java
switch (input.toLowerCase()) {
    case "yes", "y" ->
        ...
    case "no", "n" ->
        ...
    default ->
        ...
}
```

`switch`语句的“经典”形式可以追溯到C语言。其形式如下：

```java
switch (choice) {
    case 1:
        ...
        break;
    case 2:
        ...
        break;
    case 3:
        ...
        break;
    case 4:
        ...
        break;
    default:
        // bad input
        ...
        break;
}
```

`switch`语句从与选项值相匹配的`case`标签开始执行，直到遇到`break`语句或者`switch`语句结束。如果没有匹配的`case`标签，则执行`default`子句（如果有）。

警告：如果一个分支的末尾没有`break`语句，就会接着执行下一个分支！这种行为相当危险，常常会引发错误。为了检测这种问题，可以在编译代码时加上`-Xlint:fallthrough`选项。如果你确实是想使用这种“直通”(fallthrough)行为，可以为其外围方法加一个注解`@SuppressWarnings("fallthrough")`。

这两种形式都是`switch` **语句**。在3.5.9节中已经见过一个`switch` **表达式**，它生成一个值，没有直通行为。

为了对称，Java 14还引入了有直通行为的`switch`表达式。总共有4种形式的`switch`：

（1）语句，有直通

```java
switch (seasonName) {
    case "Spring":
        System.out.println("spring time!");
    case "Summer", "Winter":
        numLetters = 6;
        break;
    case "Fall":
        numLetters = 4;
        break;
    default:
        numLetters = -1;
        break;
}
```

（2）表达式，有直通

```java
int numLetters = switch (seasonName) {
    case "Spring":
        System.out.println("spring time!");
    case "Summer", "Winter":
        yield 6;
    case "Fall":
        yield 4;
    default:
        yield -1;
};
```

（3）语句，无直通

```java
switch (seasonName) {
    case "Spring" -> {
        System.out.println("spring time!");
        numLetters = 6;
    }
    case "Summer", "Winter" ->
        numLetters = 6;
    case "Fall" ->
        numLetters = 4;
    default ->
        numLetters = -1;
}
```

（4）表达式，无直通

```java
int numLetters = switch (seasonName) {
    case "Spring" -> {
        System.out.println("spring time!");
        yield 6;
    }
    case "Summer", "Winter" -> 6;
    case "Fall" -> 4;
    default -> -1;
};
```

在有直通行为的形式中，每个`case`以冒号结束。如果`case`以箭头(`->`)结束，则没有直通行为。

注意`switch`表达式中的`yield`关键字。与`break`类似，`yield`会终止执行。但不同的是，`yield`还会生成一个值，即`switch`表达式的值。

要在无直通行为的`switch`表达式的一个分支中使用语句，就必须使用大括号和`yield`。

`switch`表达式的每个分支必须生成一个值。最常见的做法是，每个值跟在一个箭头后面。如果无法做到，就使用`yield`语句。

注释：在`switch`表达式的一个分支中抛出异常是合法的。

警告：`switch`表达式的关键是生成一个值（或者因异常而失败）。不允许“跳出”`switch`表达式。具体地说，不能在`switch`表达式中使用`return`、`break`或`continue`语句。

首选`switch`表达式而不是语句。

### 3.8.6 中断控制流的语句
尽管Java的设计者将`goto`作为保留字，但并不打算在语言中包含`goto`。通常，`goto`语句被认为是糟糕的风格。

`break`语句也可以用于跳出循环。例如：

```java
while (years <= 100) {
    balance += payment;
    double interest = balance * interestRate / 100;
    balance += interest;
    if (balance >= goal) break;
    years++;
}
```

与C++不同，Java还提供了带标签的`break`语句，能够跳出多层嵌套的循环。标签必须放在最外层循环之前，并紧跟一个冒号。例如：

```java
Scanner in = new Scanner(System.in);
int n;
read_data:
while (...) { // this loop statement is tagged with the label
    ...
    for (...) { // this inner loop is not labeled
        System.out.print("Enter a number >= 0: ");
        n = in.nextInt();
        if (n < 0) // should never happen—can't go on
            break read_data; // break out of read_data loop
        ...
    }
}
// this statement is executed immediately after the labeled break
if (n < 0) { // check for bad situation
    // deal with bad situation
}
else {
    // carry out normal processing
}
```

如果输入有误，带标签的`break`会跳转到有匹配标签的语句块末尾。任何使用`break`语句的代码都需要检测循环是正常结束，还是由`break`跳出。

注释：有意思的是，可以将标签用于任何语句，甚至是`if`语句或者语句块。注意，只能跳出语句块，而不能跳入语句块。

最后，还有一个`continue`语句。`continue`语句将控制转移到最内层循环的首部。例如：

```java
Scanner in = new Scanner(System.in);
while (sum < goal) {
    System.out.print("Enter a number: ");
    n = in.nextInt();
    if (n < 0) continue;
    sum += n; // not executed if n < 0
}
```

如果n < 0，`continue`语句立即跳到循环首部（循环条件）。

如果在`for`循环中使用`continue`语句，会跳转到“更新”部分。例如：

```java
for (int count = 1; count <= 100; count++) {
    System.out.print("Enter a number: ");
    n = in.nextInt();
    if (n < 0) continue;
    sum += n; // not executed if n < 0
}
```

如果n < 0，则跳到`count++`语句。

还有一种带标签的`continue`语句，跳转到有匹配标签的循环首部。

## 3.9 大数值
如果基本的整数和浮点数精度不能够满足需求，那么可以使用`java.math`包中两个很有用的类：`BigInteger`和`BigDecimal`。`BigInteger`类实现任意精度的整数运算，`BigDecimal`实现任意精度的浮点数运算。

使用静态的`valueOf()`方法将普通数值转换为大数值：

```java
BigInteger a = BigInteger.valueOf(100);
```

对于更长的数，使用带字符串参数的构造器：

```java
BigInteger reallyBig = new BigInteger("222232244629420445529739893461909967206666939096499764990979600");
```

另外还有一些常量：`BigInteger.ZERO`、`BigInteger.ONE`和`BigInteger.TEN`，Java 9之后还有`BigInteger.TWO`。

警告：对于`BigDecimal`类，应当始终使用带字符串参数的构造器。构造器`BigDecimal(double)`天然容易产生舍入误差：`new BigDecimal(0.1)`包含数位0.1000000000000000055511151231257827021181583404541015625。

遗憾的是，不能使用熟悉的算术运算符（如`+`和`*`）组合大数值，而必须使用大数类的方法（如`add()`和`multiply()`）。

```java
BigInteger c = a.add(b); // c = a + b
BigInteger d = c.multiply(b.add(BigInteger.valueOf(2))); // d = c * (b + 2)
```

C++注释：与C++不同，Java不支持运算符重载。Java的设计者确实为字符串拼接重载了`+`运算符，但没有重载其他的运算符，也没有给Java程序员在自己的类中重载运算符的机会。

程序清单3-6是对程序清单3-5中奖概率程序的改进，改为使用大数值。

[程序清单3-6 BigIntegerTest/BigIntegerTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch03/BigIntegerTest/BigIntegerTest.java)

## 3.10 数组
### 3.10.1 声明数组
**数组**(array)是一种数据结构，用来存储同一类型值的集合。可以通过一个整型**索引**(index)（或下标）访问数组中元素。例如，如果`a`是一个整型数组，`a[i]`就是数组中的第i个整数。

在声明数组变量时，需要指定数组类型（元素类型后跟`[]`）和数组变量名。例如，`int[] a;`声明了整型数组`a`。

使用`new`运算符创建数组。

```java
int[] a = new int[100]; // or var a = new int[100];
```

这条语句声明并初始化了包含100个整数的数组（所有元素都初始化为0）。

数组长度不要求是常量：`new int[n]`创建长度为n的数组。

一旦创建了数组，就不能再改变它的长度（当然，可以改变单个数组元素）。如果程序运行中经常需要扩展数组的长度，应该使用数组列表(array list)，参见5.3节。

Java提供了一种创建数组并提供初始值的简写形式。例如：

```java
int[] smallPrimes = { 2, 3, 5, 7, 11, 13 };
```

注意，这种语法不需要使用`new`，也不用指定长度。

最后一个值后面允许有逗号。对于不断添加值的数组，这样会很方便：

```java
String[] authors = {
    "James Gosling",
    "Bill Joy",
    "Guy Steele",
    // add more names here and put a comma after each name
};
```

可以声明**匿名数组**(anonymous array)：

```java
new int[] { 17, 19, 23, 29, 31, 37 }
```

这个表达式会分配一个新数组并填入大括号中提供的值。它会统计初始值个数，并相应地设置数组长度。可以使用这种语法重新初始化一个数组而无需创建新变量。例如：

```java
smallPrimes = new int[] { 17, 19, 23, 29, 31, 37 };
```

注释：长度为0的数组是合法的。注意，长度为0的数组与`null`并不相同。

### 3.10.2 访问数组元素
数组元素**从0开始**编号。长度为n的数组合法下标范围是0~n-1。

警告：如果试图访问这个范围之外的下标（如`a[n]`），就会引发“数组下标越界”异常。

一旦创建了数组，就可以给数组元素赋值。例如，使用循环：

```java
int[] a = new int[100];
for (int i = 0; i < a.length; i++)
    a[i] = i; // fills the array with numbers 0 to 99
```

要获得数组的元素个数，使用`array.length`。

创建数值数组时，所有元素都初始化为0；`boolean`数组会初始化为`false`；对象数组则初始化为特殊值`null`。

### 3.10.3 for each循环
Java有一种强大的循环结构，可以遍历数组（或其他任何集合）中的元素，而无需使用索引。这种增强的`for`循环形式如下：

```java
for (variable : collection) statement
```

它将给定变量依次设置为集合中的每一个元素，然后执行语句。表达式`collection`必须是数组或者实现了`Iterable`接口的类对象（例如`ArrayList`）。

例如，

```java
for (int x : a)
    System.out.println(x);
```

打印数组`a`中的每一个元素，一个元素占一行。这个循环应该读作 "for each `x` in `a`" 。

当然，也可以使用传统的`for`循环达到同样的效果：

```java
for (int i = 0; i < a.length; i++)
    System.out.println(a[i]);
```

但是，for each循环更加简洁、更不易出错，因为不必为起始和终止索引值而操心。

提示：有一个更容易的方法可以打印数组中的所有值：`Arrays.toString(a)`返回一个包含数组元素的字符串，例如`"[2, 3, 5, 7, 11, 13]"`。

### 3.10.4 数组拷贝
可以将一个数组变量拷贝到另一个数组变量，但是两个变量将**引用同一个数组**：

```java
int[] luckyNumbers = smallPrimes;
luckyNumbers[5] = 12; // now smallPrimes[5] is also 12
```

下图显示了拷贝的结果。

![拷贝数组变量](/assets/images/java-note-v1ch03-fundamental-programming-structures-in-java/拷贝数组变量.png)

如果确实希望将一个数组的所有值拷贝到一个新的数组中，就要使用`Arrays.copyOf()`方法：

```java
int[] copiedLuckyNumbers = Arrays.copyOf(luckyNumbers, luckyNumbers.length);
```

第二个参数是新数组的长度。

C++注释：Java数组与栈上的C++数组有很大不同，但本质上与分配在堆上的数组的指针一样。也就是说，

```java
int[] a = new int[100]; // Java
```

不同于

```cpp
int a[100]; // C++
```

而等同于

```cpp
int* a = new int[100]; // C++
```

Java中的`[]`运算符会执行越界检查。另外，没有指针运算，这意味着不能将`a`加1指向数组中的下一个元素。

### 3.10.5 命令行参数
每个Java程序都有一个带`String[] args`参数的`main()`方法。这个参数表明`main()`方法接收一个字符串数组，即**命令行参数**(command-line arguments)。

例如，考虑下面这个程序：

[Message/Message.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch03/Message/Message.java)

如果如下调用这个程序：

```shell
java Message -g cruel world
```

`args`数组将包含`{"-g", "cruel", "world"}`，程序会打印下面的消息：

```
Goodbye, cruel world!
```

C++注释：在Java程序的`main()`方法中，程序名并不存储在`args`数组中。例如，当使用命令`java Message -h world`运行程序时，`args[0]`是`"-h"`，而不是`"Message"`或`"java"`。

### 3.10.6 数组排序
要对数值型数组进行排序，可以使用`Arrays.sort()`方法。这个方法使用了优化的快速排序(QuickSort)算法，对于大多数数据集合都很高效。`Arrays`类还为数组提供了另外一些便捷的方法，详见API文档。

程序清单3-7中的程序使用数组为抽奖游戏生成一个随机的数字组合。例如，假如从49个数字中选择6个，程序可能的输出结果为：

```
Bet the following combination. It'll make you rich!
4
7
8
19
30
44
```

[程序清单3-7 LotteryDrawing/LotteryDrawing.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch03/LotteryDrawing/LotteryDrawing.java)

`Math.random()`方法返回一个[0, 1)之间的随机浮点数。将结果乘以n并转换为整数，就可以得到0~n-1之间的随机数。

因为所有抽奖数字必须各不相同，因此每次抽取后用数组中的最后一个数覆盖`number[r]`，并将`n`减1。

### 3.10.7 多维数组
多维数组使用多个索引访问数组元素，适用于表格或其他更复杂的排列形式。

假设你想制作一张表格，用来显示在不同利率下投资10000美元有多少收益，利息每年支付并再投资（如下表所示）。

[不同利率下的投资收益情况](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch03/testdata/CompoundInterest_output.txt)

可以使用二维数组（矩阵）来存储这些信息。

在Java中声明二维数组非常简单。例如：

```java
double[][] balances = new double[NYEARS][NRATES];
```

如果知道数组元素，就可以不调用`new`，而使用简写形式对二维数组进行初始化。例如：

```java
int[][] magicSquare = {
    {16, 3, 2, 13},
    {5, 10, 11, 8},
    {9, 6, 7, 12},
    {4, 15, 14, 1}
};
```

可以使用两个下标访问二维数组的单个元素，例如`balances[i][j]`。

示例程序用到了一个存储利率的一维数组`interestRate`和一个存储账户余额的二维数组`balances`（第一维表示年度，第二维表示利率）。程序清单3-8给出了完整的程序。

[程序清单3-8 CompoundInterest/CompoundInterest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch03/CompoundInterest/CompoundInterest.java)

注释：for each循环不能自动遍历二维数组的所有元素，而是会遍历行，每一行本身就是一维数组。要访问二维数组`a`的所有元素，需要两层嵌套循环，如下所示：

```java
for (double[] row : a)
    for (double value : row)
        // do something with value
```

提示：要想快速地打印二维数组的元素列表，可以调用`Arrays.deepToString(a)`，返回的字符串格式为`[[16, 3, 2, 13], [5, 10, 11, 8], [9, 6, 7, 12], [4, 15, 14, 1]]`。

### 3.10.8 不规则数组
Java实际上没有多维数组，只有一维数组。多维数组是“数组的数组”伪装的。

例如，在前面的示例中，`balances`数组实际上是一个包含10个元素的数组，而每个元素又是由6个浮点数组成的数组，如下图所示。

![二维数组](/assets/images/java-note-v1ch03-fundamental-programming-structures-in-java/二维数组.png)

表达式`balances[i]`是第i个子数组，即表格的第i行，`balances[i][j]`是第i行的第j个元素。

由于可以单独访问二维数组的某一行，所以可以交换两行（如下图所示）。

```java
double[] temp = balances[i];
balances[i] = balances[i + 1];
balances[i + 1] = temp;
```

![交换二维数组的行](/assets/images/java-note-v1ch03-fundamental-programming-structures-in-java/交换二维数组的行.png)

还可以很容易地构造不规则数组，即数组的每一行有不同的长度。下面是一个典型的示例（杨辉三角形）：创建一个数组，第i行第j列的元素等于“从i个数中选择j个数”可能的组合数（即 $C_i^j$ ）。

```
1
1 1
1 2 1
1 3 3 1
1 4 6 4 1
1 5 10 10 5 1
1 6 15 20 15 6 1
```

要创建一个不规则数组，首先分配一个包含这些行的数组：

```java
int[][] odds = new int[NMAX + 1][];
```

接下来分配这些行：

```java
for (int n = 0; n <= NMAX; n++)
    odds[n] = new int[n + 1];
```

程序清单3-9给出了完整的程序。

[程序清单3-9 LotteryArray/LotteryArray.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch03/LotteryArray/LotteryArray.java)

C++注释：在C++中，Java声明

```java
double[][] balances = new double[10][6]; // Java
```

不同于

```cpp
double balances[10][6]; // C++
```

也不同于

```cpp
double (*balances)[6] = new double[10][6]; // C++
```

而是分配了一个包含10个指针的数组，然后指针数组的每个元素填充一个包含6个数字的数组：

```cpp
double** balances = new double*[10]; // C++
for (i = 0; i < 10; i++)
    balances[i] = new double[6];
```

庆幸的是，在Java中这个循环将自动地执行。需要不规则数组时，只能单独创建行数组。

注：C++的二维数组详见[【C++】二维数组的行指针和列指针]({% post_url 2022-05-11-cpp-two-dimensional-array-and-pointer %})。
