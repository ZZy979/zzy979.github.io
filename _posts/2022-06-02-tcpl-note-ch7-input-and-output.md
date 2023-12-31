---
title: 《C程序设计语言》笔记 第7章 输入与输出
date: 2022-06-02 23:00:09 +0800
categories: [C/C++, TCPL]
tags: [c, io, formatting, variadic argument, file access]
---
本章将讲述标准库，介绍输出/输出、字符串处理、存储管理与数学函数以及其他一些服务的函数。本章的重点将放在输入/输出上。

## 7.1 标准输入/输出
如第1章所述，标准库实现了简单的文本输入/输出模型。文本流由一系列行组成，每一行以一个换行符结尾。如果系统没有遵循这种模型，则标准库会通过一些措施使其看起来遵循了这种模型。例如，标准库可能在输入时将回车符(carriage return, CR, `\r`)和换行符(line feed, LF, `\n`)都转换为换行符，而在输出时进行反向转换。

注：不同操作系统的默认换行符是不同的，Windows是CRLF (`\r\n`)，Unix/Linux是LF (`\n`)，macOS是CR (`\r`)，参考：
* [Carriage Return, Line Feed and New Line](https://stackoverflow.com/questions/12386803/carriage-return-line-feed-and-new-line)
* [The Great Newline Schism](https://blog.codinghorror.com/the-great-newline-schism/)

最简单的输入机制是使用`getchar`函数从**标准输入**（一般为键盘）一次读取一个字符。`getchar`函数在每次被调用时返回下一个输入字符，如果遇到文件结尾则返回`EOF`。符号常量`EOF`定义在\<stdio.h\>中，其值一般为-1，但程序中应该使用名字`EOF`来测试从而不依赖于特定值。

函数`putchar(c)`将字符`c`送至**标准输出**（默认为屏幕）。`putchar`返回输出的字符，如果发生错误则返回`EOF`。函数`printf`也将输出发送到标准输出，可以和`putchar`函数交替使用。

使用输入/输出库函数的源文件必须包含`#include <stdio.h>`。

在命令行中，可以使用`<`符号来**重定向输入**，将键盘输入替换为文件输入：如果程序`prog`中使用了函数`getchar`，则命令行

```bash
prog <infile
```

将使得程序`prog`从输入文件infile（而不是键盘）读取字符。

类似地，可以使用`>`符号来**重定向输出**：

```bash
prog >outfile
```

将程序`prog`的标准输出重定向到文件outfile。

字符串 "<infile" 和 ">outfile" 不包含在命令行参数`argv`中。

**管道**(pipe)机制也可以用于重定向标准输入/输出：

```bash
prog | otherprog
```

将程序`prog`的标准输出重定向到程序`otherprog`的标准输入。

注：Windows CMD和Linux Shell命令行都支持输入/输出重定向和管道机制。

对于只从一个输入流读取数据、只向一个输出流写入数据的程序，使用`getchar`、`putchar`和`printf`函数实现输入/输出就足够了。考虑程序`lower`，将输入转换为小写形式：

[将输入转换为小写形式](https://github.com/ZZy979/TCPL-code/blob/main/ch7/lower.c)

头文件\<stdio.h\>中的`getchar`和`putchar`以及\<ctype.h\>中的`tolower` **一般都是宏**，这样就避免了对每个字符都进行函数调用的开销。

例如，假设编译出的可执行文件为lower.out，则命令行

```bash
./lower.out <a.txt
```

将文件a.txt的内容转换为小写并打印到标准输出；

利用打印文件的Linux命令cat或Windows命令type（1.5.1节编写过类似功能的程序）和管道机制，以下命令和上面的等价：

```bash
cat a.txt | ./lower.out  # Linux
type a.txt | lower.exe  # Windows
```

输入/输出重定向可以同时使用：

```bash
./lower.out <a.txt >b.txt
```

将文件a.txt的内容转换为小写并输出到文件b.txt。

[练习7-1](https://github.com/ZZy979/TCPL-code/blob/main/ch7/exec7-1.c) 编写一个程序，根据它被调用的名字(`argv[0]`)实现将大写字母转换为小写或将小写字母转换为大写的功能。

## 7.2 格式化输出——printf
输出函数`printf`将内部值（整数、浮点数等）转换为字符。本节将介绍该函数最典型的用法。

```c
int printf(char *format, arg1, arg2, ...)
```

`printf`函数在`format`的控制下，对其参数进行转换与格式化，并在标准输出上打印出来。它**返回打印的字符数**。

格式字符串包含两种对象：普通字符和转换说明(conversion specification)。普通字符将被原样复制到输出流，而每一个转换说明将转换并打印`printf`的下一个参数。每个转换说明以`%`开始、以转换字符结束，格式为`%[-][宽度][.精度][h/l]转换字符`。在%和转换字符之间可能依次包含：
* 负号，指定左对齐
* 数字，指定最小宽度，如果大于实际宽度则用空格填充（默认右对齐）
* 小数点，用于分隔宽度和精度
* 数字，指定精度，即字符串要打印的最大字符数、浮点数小数点后的位数，或者整数要打印的最少数字数目
* 字母`h`或`l`，`h`表示将整数作为`short`类型打印，`l`表示将整数或浮点数作为`long`类型打印

下表列出了所有的转换字符。如果`%`后面的字符不是转换说明，则行为是未定义的。

| 转换字符 | 参数类型 | 输出形式 |
| --- | --- | :-- |
| `d`, `i` | `int` | 十进制整数 |
| `u` | `int` | 无符号十进制整数 |
| `o` | `int` | 无符号八进制整数（没有前导0） |
| `x`, `X` | `int` | 无符号十六进制整数（没有前导0x或0X） |
| `c` | `int` | 字符 |
| `s` | `char *` | 字符串，直到遇到`'\0'`或由精度指定的字符数为止 |
| `f` | `double` | 浮点数，其中小数位数由精度指定（默认为6） |
| `e`, `E` | `double` | 浮点数科学记数法，小数位数由精度指定（默认为6） |
| `g`, `G` | `double` | 浮点数，有效数字位数由精度指定（默认为6），如果指数小于-4或大于等于精度则使用`%e`或`%E`，否则使用`%f`，尾部的0和小数点不打印 |
| `p` | `void *` | 指针 |
| `%` |  | 不转换参数，打印一个% |

在转换说明中，宽度或精度可以指定为`*`，此时宽度或精度的值由下一个参数（必须是`int`类型）指定。例如，下列语句打印字符串`s`的至多`max`个字符：

```c
printf("%.*s", max, s);
```

（注：转换说明`%.*s`将“消耗”两个参数）

注：完整列表见[printf - cplusplus.com](https://cplusplus.com/reference/cstdio/printf/)和[printf - cppreference.com](https://en.cppreference.com/w/cpp/io/c/fprintf)。

下表展示了在打印字符串`"hello, world"`（12个字符）时不同转换说明的效果（在两端添加了冒号）：

```
%s        :hello, world:
%10s      :hello, world:
%15s      :   hello, world:
%-10s     :hello, world:
%-15s     :hello, world   :
%.10s     :hello, wor:
%.15s     :hello, world:
%15.10s   :     hello, wor:
%-15.10s  :hello, wor     :
```

下表是打印浮点数1234.5678时不同转换说明的效果：

```
%f        :1234.567800:
%8f       :1234.567800:
%15f      :    1234.567800:
%-8f      :1234.567800:
%-15f     :1234.567800    :
%.2f      :1234.57:
%.8f      :1234.56780000:
%15.2f    :        1234.57:
%-15.2f   :1234.57        :

%e        :1.234568e+03:
%8e       :1.234568e+03:
%15e      :   1.234568e+03:
%-8e      :1.234568e+03:
%-15e     :1.234568e+03   :
%.2e      :1.23e+03:
%.8e      :1.23456780e+03:
%15.2e    :       1.23e+03:
%-15.2e   :1.23e+03       :

%g        :1234.57:
%8g       : 1234.57:
%15g      :        1234.57:
%-8g      :1234.57 :
%-15g     :1234.57        :
%.2g      :1.2e+03:
%.8g      :1234.5678:
%15.2g    :        1.2e+03:
%-15.2g   :1.2e+03        :
```

下表是打印整数12345678时不同转换说明的效果：

```
%d        :12345678:
%6d       :12345678:
%15d      :       12345678:
%-6d      :12345678:
%-15d     :12345678       :
%.6d      :12345678:
%.10d     :0012345678:
%15.10d   :     0012345678:
%-15.10d  :0012345678     :
```

注意：`printf`函数使用第一个参数（格式字符串）判断后面参数的个数及类型。如果参数的个数不够或者类型错误，则将得到错误的结果。注意下面两个函数调用之间的区别：

```c
printf(s);        /* 如果s包含%将会出错 */
printf("%s", s);  /* 正确 */
```

函数`sprintf`执行与函数`printf`相同的转换，但将结果保存到一个字符串中，而不是标准输出：

```c
int sprintf(char *output, char *format, arg1, arg2, ...);
```

字符串`output`必须足够大以存放输出结果。

[练习7-2](https://github.com/ZZy979/TCPL-code/blob/main/ch7/exec7-2.c) 编写一个程序，以合理的方式打印任意输入。该程序至少能够根须用户习惯以八进制或十六进制打印非图形字符，并截断长文本行。

## 7.3 变长参数表
本节通过实现`printf`函数的一个最简单版本`minprintf`来介绍如何以可移植的方式编写处理变长参数表的函数。由于重点关注参数处理，`minprintf`只处理格式字符串和参数，而通过调用`printf`函数实现格式转换。

`printf`函数的正确声明形式为

```c
int printf(char *format, ...);
```

其中省略号表示这些参数的数量和类型是可变的。**省略号只能出现在参数表的尾部。**

`minprintf`函数声明为
```c
void minprintf(char *format, ...);
```

编写`minprintf`函数的关键在于如何处理一个甚至连名字都没有的参数表。 **标准头文件\<stdarg.h\>** 包含一组宏定义，它们定义了如何遍历参数表。该头文件在不同机器上的实现是不同的，但提供的接口是一致的。
* `va_list`类型用于声明一个变量，该变量将依次引用各参数（在`minprintf`中，该变量称为`ap`，意思是“参数指针”）
* 宏`va_start`将`ap`初始化为指向第一个无名参数的指针。在使用`ap`之前，该宏必须被调用一次。**参数表必须至少包括一个有名参数**，因为`va_start`将最后一个有名参数作为起点。
* 每次调用`va_arg`都将返回一个参数，并将`ap`指向下一个参数
* `va_end`完成必要的清理工作，它必须在函数返回之前调用

简化的`printf`函数实现如下：

[minprintf函数](https://github.com/ZZy979/TCPL-code/blob/main/ch7/minprintf.c)

[练习7-3](https://github.com/ZZy979/TCPL-code/blob/main/ch7/minprintf2.c) 改写`minprintf`函数，使它能完成`printf`函数的更多功能。

## 7.4 格式化输入——scanf
输入函数`scanf`对应于输出函数`printf`，在相反的方向上提供同样的转换功能。`scanf`函数的声明如下：

```c
int scanf(char *format, ...)
```

`scanf`函数从标准输入读取字符，按照`format`中的格式说明解释读取的字符，并把结果保存到其余的参数中。格式参数将在下面介绍；**其他参数都必须是指针**，用于指定相应输入经过转换后应保存的位置。

当`scanf`函数扫描完格式字符串（同时读取输入），或者某些输入无法与转换说明匹配时，该函数将终止，同时**返回成功匹配并赋值的输入项的个数**，如果到达文件结尾则返回`EOF`。注意，返回`EOF`与0是不同的，返回0表示下一个输入字符与格式字符串中的第一个转换说明不匹配。下一次调用`scanf`函数将从已转换的最后一个字符的下一个字符开始继续搜索。

还有一个输入函数`sscanf`，它从一个字符串而不是标准输入读取字符：

```c
int sscanf(char *input, char *format, ...)
```

格式字符串可以包含：
* 空白符，将被忽略
* 普通字符（不包括%），用于匹配输入流中的下一个非空白字符
* 转换说明，格式为`%[*][宽度][h/l/L]转换字符`，由一个`%`、一个可选的赋值禁止符`*`、一个可选的数字（指定最大宽度）、一个可选的`h`、`l`或`L`（指定目标对象的宽度）以及一个转换字符组成

每个转换说明符控制下一个输入字段的转换。一般来说，转换结果存放在对应参数指向的变量中。但是，如果转换说明中有`*`，则跳过该输入字段，不进行赋值。**输入字段定义为一个不包含空白符的字符串**，其边界为下一个空白符或达到指定的字段宽度（如果有）。这意味着`scanf`函数可能越过行边界读取输入（空白符包括空格、制表符`\t`、换行符`\n`、回车符`\r`、纵向制表符`\v`以及换页符`\f`）。

转换字符指定对输入字段的解释，对应的参数必须是指针。下表列出了这些转换字符。

| 转换字符 | 输入数据 | 参数类型 |
| --- | :-- | --- |
| `d` | 十进制整数 | `int *` |
| `i` | 整数，可以是八进制（以0开头）或十六进制（以0x或0X开头） | `int *` |
| `u` | 无符号十进制整数 | `unsigned int *` |
| `o` | 八进制整数（可以有或没有前导0） | `int *` |
| `x` | 十六进制整数（可以有或没有前导0x或0X） | `int *` |
| `c` | 字符，不跳过空白符（要读取下一个非空白符，使用`%1s`） | `char *` |
| `s` | 字符串（不加引号），参数指向足以存放该字符串的字符数组，结尾将被添加`'\0'` | `char *` |
| `e`, `f`, `g` | 浮点数，包括可选的符号、小数点和指数部分 | `float *` |
| `%` | 字符%，不进行赋值 | |

转换字符`d`、`i`、`u`、`o`和`x`前面可以添加字符`h`，表明对应参数是指向`short`而不是`int`的指针；或者添加字符`l`，表明对应参数是指向`long`的指针。类似地，转换字符`e`、`f`和`g`前面也可以添加字符`l`，表明对应参数是指向`double`而不是`float`的指针。

注：完整列表见[scanf - cplusplus.com](https://cplusplus.com/reference/cstdio/scanf/)和[scanf - cppreference.com](https://en.cppreference.com/w/cpp/io/c/fscanf)。

第一个例子改写第4章中的简单计算器，通过`scanf`函数执行输入转换：

[简单计算器](https://github.com/ZZy979/TCPL-code/blob/main/ch7/rudimentary_calculator.c)

假设要读取包含下列日期格式的输入行：

```
25 Dec 1988
```

相应的`scanf`语句为

```c
int day, year;
char monthname[20];

scanf("%d %s %d", &day, monthname, &year);
```

`monthname`的前面没有`&`，因为数组名本身就是指针。

字符字面值也可以出现在`scanf`的格式字符串中，**它们必须与输入中相同的字符匹配**。因此，可以使用以下`scanf`语句读取`mm/dd/yyyy`形式的日期数据：

```c
int day, month, year;

scanf("%d/%d/%d", &month, &day, &year);
```

关于空白符：
* `scanf`格式字符串中的一个空白符匹配输入中任意数量（包括0个）的空白符
* 在读取输入时，首先会**跳过空白符**，这意味着读取的字符串不会包含前导空白符
* 输入字段以空白符结束，这意味着**无法读取包含空格的字符串**（`fgets`可以）

如果要读取格式不固定的输入，最好每次读取一行，然后用`sscanf`读取。例如，假设需要读取包含上述任意一种格式的日期数据的输入行，可以这样编写程序：

```c
while (getline(line, sizeof(line)) > 0) {
    if (sscanf(line, "%d %s %d", &day, monthname, &year) == 3)
        printf("valid: %s\n", line);  /* 25 Dec 1988格式 */
    else if (sscanf(line, "%d/%d/%d", &month, &day, &year) == 3)
        printf("valid: %s\n", line);  /* mm/dd/yyyy格式 */
    else
        printf("invalid: %s\n", line);  /* 无效格式 */
}
```

`scanf`函数可以和其他输入函数混合使用，下一个输入函数的调用将从`scanf`没有读取的第一个字符处开始读取数据。

注意，`scanf`和`sscanf`的参数**必须是指针**。最常见的错误是将`scanf("%d", &n)`写成`scanf("%d", n)`。编译器一般检测不到这类错误。

[练习7-4](https://github.com/ZZy979/TCPL-code/blob/main/ch7/minscanf.c) 类似于上一节中的函数`minprintf`，编写一个`scanf`函数的简化版本。

[练习7-5](https://github.com/ZZy979/TCPL-code/tree/main/ch7/reverse_polish_calculator) 改写第4章中的后缀计算器程序，用`scanf`和（或）`sscanf`函数实现输入及数字的转换。

## 7.5 文件访问
到目前为止，所有的例子都是读取标准输入、写标准输出。标准输入和标准输出是操作系统自动提供给程序访问的。

下面编写一个访问文件的程序cat，用于把一些文件拼接(concatenate)到标准输出上。cat可用来在屏幕上打印文件内容，对于无法通过名字访问文件（只能读取标准输入）的程序，也可以用作输入收集器（见7.1节最后的示例）。例如，命令

```bash
cat a.txt b.txt
```

将在标准输出上依次打印文件a.txt和b.txt的内容。

问题在于如何通过文件名读取文件。在读写文件之前，必须通过\<stdio.h\>定义的库函数`fopen`打开该文件，该函数接收文件名和打开模式作为参数，返回一个随后可用于读写文件的指针。

```c
FILE *fopen(const char *filename, const char *mode);
```

`FILE`是一个通过`typedef`定义的类型名，而不是结构标记。

在程序中可以这样使用：

```c
FILE *fp = fopen("a.txt", "r");
```

其中第一个参数是文件名，第二个参数是访问**模式**，用于指定文件的使用方式。允许的模式包括读(`"r"`, read)、写(`"w"`, write)和追加(`"a"`, append)。如果是二进制文件，还需要在模式字符串后添加`"b"`。

注：如果文件名是相对路径，则相对于程序的工作目录（启动程序时命令行的当前目录）

如果以写或追加方式打开一个不存在的文件，该文件将被创建（如果可能的话）；如果以写方式打开一个已存在的文件，该文件原来的内容将被覆盖，而以追加方式打开则会保留原来的内容；读一个不存在的文件将导致错误，其他一些操作也可能导致错误（例如没有权限）。如果发生错误，`fopen`将返回`NULL`。

文件被打开后，就需要一种读写文件的方式。有多种可能的方式，最简单的是`getc`和`putc`：

```c
int getc(FILE *fp);
int putc(int c, FILE *fp);
```

`getc`返回文件`fp`中的下一个字符，如果遇到文件尾或错误则返回`EOF`。`putc`将字符`c`写入文件`fp`并返回写入的字符，如果发生错误则返回`EOF`。类似于`getchar`和`putchar`，`getc`和`putc`可能是宏而不是函数。

启动一个C语言程序时，操作系统环境负责打开三个文件，并提供它们的指针。这三个文件分别是**标准输入、标准输出和标准错误**，相应的文件指针分别叫做`stdin`、`stdout`和`stderr`，它们在\<stdio.h\>中声明。通常`stdin`连接到键盘，`stdout`和`stderr`连接到屏幕，但可以通过7.1节所述的方式重定向到文件或管道。

注：标准错误可以使用`2>`重定向：

```bash
prog 2>errfile
```

将程序`prog`的标准错误重定向到文件errfile；

```bash
prog >outfile 2>&1
```

将程序`prog`的标准输出和标准错误都重定向到文件outfile，该命令等价于

```bash
prog &>outfile
```

`getchar`和`putchar`可以定义如下：

```c
#define getchar()  getc(stdin)
#define putchar(c) putc((c), stdout)
```

对于文件的格式化输入/输出，可以使用`fscanf`和`fprintf`函数。它们与`scanf`和`printf`的区别仅在于第一个参数是文件指针：

```c
int fscanf(FILE *fp, const char *format, ...);
int fprintf(FILE *fp, const char *format, ...);
```

下面编写拼接文件的程序cat。如果有命令行参数，则将其解释为文件名，并按顺序处理。如果没有参数，则处理标准输入。

[拼接文件（版本1）](https://github.com/ZZy979/TCPL-code/blob/main/ch7/cat.c)

函数`fclose`执行与`fopen`相反的操作，它断开由`fopen`建立的文件指针和外部名之间的连接（即关闭文件），并释放文件指针以供其他文件使用。因为大多数操作系统都限制了一个程序可以同时打开的文件数，因此**当文件指针不再需要时就应该释放**，这是一个好的编程习惯。对输出文件调用`fclose`还有另外一个原因：它将把`putc`输出的缓冲区刷新(flush)到文件中。当程序正常终止时，程序会自动为每个打开的文件调用`fclose`。

## 7.6 错误处理——stderr和exit
cat程序的错误处理并不完善。问题在于，如果其中一个文件因为某种原因无法访问，错误信息将被打印到拼接输出的末尾。当输出到屏幕时，这种处理方法尚可接受，但如果输出到文件或者通过管道输出到另一个程序时，就无法接受了。

为了更好地处理这种情况，可以使用另一个输出流——标准错误`stderr`。即使对标准输出进行了重定向，写到标准错误中的输出通常也会显示在屏幕上（除非使用`2>`进行了重定向）。

下面改写cat程序，将错误信息写到标准错误上。

[拼接文件（版本2）](https://github.com/ZZy979/TCPL-code/blob/main/ch7/cat_v2.c)

该程序通过两种方式发出错误信号。首先，将`fprintf`产生的错误信息输出到`stderr`，因此错误信息将会显示在屏幕上，而不是输出到管道或文件中。错误信息包含了程序名(`argv[0]`)，因此当该程序和其他程序一起运行时，可以识别错误的来源。

其次，程序使用了标准库函数`exit`（定义在\<stdlib.h\>），当该函数被调用时将终止程序执行。任何调用该程序的进程都可以获取`exit`的参数值（返回码），因此可以测试该程序成功或失败。**按照管理，返回值0表示一切正常，而非0值通常表示出现了异常情况。** `exit`为每个已打开的输出文件调用`fclose`，以将缓冲区中的输出写到相应的文件中。

在`main`函数中，`return expr`等价于`exit(expr)`。但是`exit`有一个有点：它可以从其他函数中调用，并且可以用类似于第5章中描述的模式查找程序查找这些调用（使用现代IDE也很容易查找）。

如果文件`fp`中出现错误，则函数`ferror(fp)`返回一个非0值。尽管输出错误很少见，但还是存在的（例如磁盘已满），因此成熟的产品程序应该检查这种错误。

函数`feof`与`ferror`类似，如果指定的文件到达文件结尾，则返回一个非0值。

在为了说明问题的小程序中，通常不太关心程序的退出状态。但对于重要的程序来说，都应该返回有意义且有用的值。

## 7.7 行输入和输出
标准库提供了一个输入函数`fgets`，它和前面几章中用到的`getline`函数类似。

```c
char *fgets(char *line, int maxline, FILE *fp);
```

`fgets`函数从文件`fp`中读取下一个输入行（**包括换行符**，除非超过了最大行长度），并存放在字符数组`line`中，最多读取`maxline-1`个字符。读取的行将以`'\0'`结尾。通常情况下，`fgets`返回`line`，如果遇到文件结尾或发生错误则返回`NULL`（我们的`getline`函数返回行长度，这个值更有用，0意味着到达文件结尾）。

注：`fgets`每次读取一整行，因此可以读取包含空格的字符串（`scanf`不行）

输出函数`fputs`将一个字符串写入文件中（**不会自动添加换行符**）：

```c
int fputs(const char *line, FILE *fp);
```

如果发生错误则返回`EOF`，否则返回0。

库函数`gets`和`puts`类似于`fgets`和`fputs`，但是对`stdin`和`stdout`进行操作。令人困惑的是，`gets`删除结尾的换行符，而`puts`自动添加换行符。

注：`gets`函数没有最大长度参数，可能导致数组下标越界，因此在新版标准库中被标记为弃用

下面的标准库中`fgets`和`fputs`函数的代码。可以看出，这两个函数并没有什么特别的地方。

```c
/* fgets：从文件iop读取最多n-1个字符 */
char *fgets(char *s, int n, FILE *iop) {
    register int c;
    register char *cs;

    cs = s;
    while (--n > 0 && (c = getc(iop)) != EOF)
        if ((*cs++ = c) == '\n')
            break;
    *cs = '\0';
    return (c == EOF && cs == s) ? NULL : s;
}

/* fputs：将字符串s输出到文件iop */
int fputs(char *s, FILE *iop) {
    int c;

    while (c = *s++)
        putc(c, iop);
    return ferror(iop) ? EOF : 0;
}
```

使用`fgets`很容易实现`getline`函数：

[getline函数](https://github.com/ZZy979/TCPL-code/blob/main/ch7/getline.c)

[练习7-6](https://github.com/ZZy979/TCPL-code/blob/main/ch7/diff.c) 编写一个程序，比较两个文件并打印第一个不相同的行。

[练习7-7](https://github.com/ZZy979/TCPL-code/blob/main/ch7/find.c) 修改第5章的模式查找程序，使它从一组命名文件中读取输入，如果没有文件名参数，则从标准输入读取。当发现一个匹配行时，是否应该将相应的文件名打印出来？

练习7-8 编写一个程序，打印一组文件，每个文件从新的一页开始打印，并为每个文件打印标题和页数。

## 7.8 其他函数
标准库提供了各种各样的函数。完整列表可以参考[C++ Reference](http://cplusplus.com/reference/)中C Library一节。

练习7-9 类似于`isupper`这样的函数可以通过某种方式实现以达到节省空间或时间的目的。考虑这两种实现方式。
