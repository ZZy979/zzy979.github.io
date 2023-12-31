---
title: 《C程序设计语言》笔记 第3章 控制流
date: 2022-03-26 21:17:41 +0800
categories: [C/C++, TCPL]
tags: [c, if statement, block, binary search, switch statement, while statement, for statement, sort, do-while statement, break statement, continue statement, goto statement]
---
## 3.1 语句与程序块
在表达式之后加上一个分号就变成了**语句**(statement)（表达式语句）。例如：

```c
x = 0;
i++;
printf("Hello, world\n");
```

在C语言中，**分号是语句结束符**。

用一对花括号把一组声明和语句括在一起就构成了一个**复合语句**(compound statement)，也叫作**程序块**(block)。**复合语句在语法上等价于单条语句。** 例如函数体，以及`if`、`else`、`while`和`for`之后的语句。右花括号用于结束程序块，其后不需要分号。

## 3.2 if-else语句
`if-else`语句用于条件判定。其语法如下：

```c
if (表达式)
    语句1
else
    语句2
```

其中`else`部分是可选的。该语句执行时，先计算表达式的值，如果其值为真（即表达式的值为非0），则执行语句1；如果其值为假（即表达式的值为0），并且该语句包含`else`部分，则执行语句2。其中语句既可以是单条语句，也可以是括在花括号内的复合语句。

由于`if`语句只是简单地测试表达式的数值，因此`if (expr != 0)`等价于`if (expr)`；`if (expr == 0)`等价于`if (!expr)`。

因为`if-else`语句的`else`部分是可选的，所以在嵌套的`if`语句中省略`else`部分将导致歧义。解决的方法是将每个`else`与最近的一个没有`else`配对的`if`进行匹配（**就近原则**）。例如，在下列语句中：

```c
if (n > 0)
    if (a > b)
        z = a;
    else
        z = b;
```

`else`部分与内层的`if`匹配，通过缩进也可以看出来（**缩进并不能决定匹配关系**，即使`else`不缩进也会与内层`if`匹配，因为编译器会忽略所有的空白符）。如果想要与外层`if`匹配，则必须使用花括号（没有实际意义，只是为了说明）：

```c
if (n > 0) {
    if (a > b)
        z = a;
}
else
    z = b;
```

这种错误很难发现，因此建议在有嵌套的`if`语句时使用花括号。

## 3.3 else-if语句
在C语言中经常会用到下列结构：

```c
if (表达式1)
    语句1
else if (表达式2)
    语句2
else if (表达式3)
    语句3
...
else
    语句n
```

这种`if`语句序列是编写多路判定最常用的方法。其中的各表达式将被依次求值，一旦某个表达式结果为真，则执行对应的语句，并终止整个语句序列的执行。其中各语句既可以是单条语句，也可以是括在花括号内的复合语句。最后一个`else`部分用于处理“上述条件均不成立”的情况或默认情况，可省略。

这里通过一个**二分查找**（折半查找）函数`binsearch`说明三路判定程序的用法。该函数用于判定**已排序的**数组v中是否存在某个特定的值x。如果v中包含x，则该函数返回x在v中的位置(0~n-1)；否则返回-1。

在二分查找时，首先将x与数组v的**中间元素**进行比较。如果x小于中间元素，则在数组的前半部分查找；否则在数组的后半部分查找。这个过程一直进行下去，直到找到指定的值或查找范围为空。

[binsearch函数](https://github.com/ZZy979/TCPL-code/blob/main/ch3/binsearch.c)

例如，下图是在数组{2, 10, 18, 22, 24, 30, 35}中查找24的过程：

![二分查找](/assets/images/tcpl-note-ch3-control-flow/二分查找.png)

[练习3-1](https://github.com/ZZy979/TCPL-code/blob/main/ch3/exec3-1.c) 上面的折半查找在循环内执行了两次测试，其实只要一次就足够（代价是将更多的测试在循环外执行）。重写该函数，使得在循环内部只执行一次测试。比较两种版本函数的运行时间。

| 执行次数（万） | binsearch运行时间(ms) | binsearch2运行时间(ms) |
| --- | --- | --- |
| 100 | 101 | 94 |
| 200 | 205 | 194 |
| 500 | 509 | 484 |
| 1000 | 991 | 954 |
| 2000 | 2058 | 1971 |
| 3000 | 3061 | 2965 |
| 4000 | 4089 | 3924 |
| 5000| 5161 | 4897 |

## 3.4 switch语句
`switch`语句是一种多路判定语句，它测试表达式是否与一些**整形常量**中的某一个值匹配，并执行相应的分支动作。其语法如下：

```c
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

**各分支表达式必须互不相同。** 如果所有分支都不匹配则执行`default`分支，`default`分支是可选的。如果所有分支都不匹配也没有`default`分支，则`switch`语句不执行任何动作。各分支及`default`分支的排列次序是任意的（但习惯上将`default`分支放在最后）。

在第1章曾用`if-else`结构编写过一个统计字符出现次数的程序。下面用`switch`语句改写该程序如下：

[统计字符的出现次数](https://github.com/ZZy979/TCPL-code/blob/main/ch3/character_counting_switch.c)

`break`语句导致程序的执行立即从`switch`语句中退出。在`switch`语句中，`case`的作用只是一个标号，因此**某个分支的代码执行完后，程序将进入(fall through)下一分支继续执行，除非在程序中显式地跳转** 。跳出`switch`语句最常用的方法是`break`语句和`return`语句。

依次执行各分支(fall through)的做法有优点也有缺点。优点是它可以让若干个分支执行同一个动作，如上例中对数字的处理。但是，正常情况下为了防止直接进入下一个分支执行，每个分支必须以一个`break`语句结束。**从一个分支直接进入下一个分支的做法在程序修改时很容易出错**，应尽量减少这种做法。在不得不使用的情况下应该加上适当的注释。

作为一种良好的程序设计风格，在`switch`语句最后一个分支（即`default`分支）后也加上一个`break`语句。这样做在逻辑上没有必要，但当我们需要向末尾添加其他分支时，这种防范措施会降低犯错误的可能性。

[练习3-2](https://github.com/ZZy979/TCPL-code/blob/main/ch3/escape.c) 编写一个函数`escape(s, t)`，将字符串t复制到字符串s中，并在复制过程中将换行符、制表符等不可见字符分别转换为`\n`、`\t`等相应的可见的转义字符序列。要求使用`switch`语句。再编写一个具有相反功能的函数，在复制过程中将转义字符序列转换为实际字符。

## 3.5 while循环与for循环
`while`循环语句：

```c
while (表达式)
    语句
```

首先求表达式的值。如果其值为真（非0），则执行语句，并再次求该表达式的值。这一过程一直进行下去，直到该表达式的值为假（0）为止。

`for`循环语句：

```c
for (表达式1; 表达式2; 表达式3)
    语句
```

等价于

```c
表达式1;
while (表达式2) {
    语句
    表达式3;
}
```

但当语句中包含`continue`语句时就不一定等价了。

从语法角度看，`for`循环的三个组成部分都是任意表达式。最常见的情况是，表达式1与表达式3是赋值表达式或函数调用，表达式2是关系表达式。**这三个组成部分中的任何部分都可以省略，但分号必须保留。** 如果省略表达式1与表达式3，它就退化成了`while`循环；**如果省略表达式2，则认为其值永远是真**。因此

```c
for (;;) {
    ...
}
```

是一个“无限”循环，需要借助其他手段（如`break`或`return`）才能终止执行。

使用`while`循环还是`for`循环主要取决于个人偏好。`while`循环更适合没有初始化或重新初始化操作的情况；`for`循环更适合语句中需要执行简单的初始化和变量递增的情况，它将循环控制语句集中放在循环的开头，结构更紧凑、更清晰。例如，C语言处理数组前n个元素的一种习惯性用法：

```c
for (i = 0; i < n; ++i)
    ...
```

下面的例子重新编写将字符串转换为对应数值的函数`atoi`。这个版本比第2章中的`atoi`函数更通用，它可以处理可选的前导空白符以及一个可选的正(+)或负(-)号。下面是程序的结构：

```
如果有空白符，则跳过
如果有符号，则读取符号
读取整数部分，并执行转换
```

当遇到第一个不能转换为数字的字符时，整个处理过程终止。

[atoi函数](https://github.com/ZZy979/TCPL-code/blob/main/ch3/atoi.c)

标准库\<stdlib.h\>中提供了两个更完善的函数`atoi`和`strtol`。

把循环控制部分集中在一起，对于多重嵌套循环，优势更为明显。下面的函数是对整型数组排序的Shell排序算法。其基本思想是：先比较距离较远的元素，而不是像简单交换排序算法那样先比较相邻的元素。这样可以快速减少大量的无序情况，从而减轻后续的工作。被比较的元素之间的距离逐步减少，直到减少为1，这时排序变成了相邻元素的互换。

[shellsort函数](https://github.com/ZZy979/TCPL-code/blob/main/ch3/shellsort.c)

逗号运算符(`,`)是C语言优先级最低的运算符，在`for`语句中经常会用到它。被逗号分隔的一对表达式将按照**从左到右**的顺序进行求值，**右边表达式**的类型和值即为其结果的类型和值。这样在`for`语句中，可以将多个表达式放在各个部分中，比如同时处理两个循环控制变量。

以下面的函数`reverse(s)`为例，该函数用于反转字符串s：

[reverse函数](https://github.com/ZZy979/TCPL-code/blob/main/ch3/reverse.c)

分隔函数参数的逗号、分隔声明中变量的逗号等并不是逗号运算符，不保证从左到右的求值顺序。

应该慎用逗号运算符。逗号运算符最适用于关系紧密的结构中，比如上面的`reverse`函数内的`for`语句，以及需要在单个表达式中进行多步计算的宏。逗号表达式还适用于`reverse`函数中的元素交换，这样元素交换可以看成单步操作：

```c
for (i = 0, j = strlen(s) - 1; i < j; ++i, --j)
    c = s[i], s[i] = s[j], s[j] = c;
```

[练习3-3](https://github.com/ZZy979/TCPL-code/blob/main/ch3/expand.c) 编写函数`expand(s1, s2)`，将字符串s1中类似于a-z的速记符号在字符串s2中扩展为等价的完整列表abc...xyz。该函数可以处理大小写字母和数字，并可以处理a-b-c、a-z0-9和-a-z等类似的情况。前导和尾随的-字符原样保留。

## 3.6 do-while循环
`do-while`循环的语法如下：

```c
do
    语句
while (表达式);
```

先执行语句部分，然后再求表达式的值。如果表达式的值为真，则再次执行语句，以此类推。当表达式的值变为假，则循环终止。

`while`和`for`循环在循环体执行前对终止条件进行测试，而`do-while`循环在循环体执行后测试终止条件，因此**循环体至少被执行一次**。

经验表明，`do-while`循环比`while`和`for`循环用得少得多，但有时还是很有用的，下面通过`itoa`函数来说明这一点。`itoa`函数是`atoi`函数的逆函数，将数字转换为字符串：首先生成反序的字符串，然后再反转该字符串。

[itoa函数](https://github.com/ZZy979/TCPL-code/blob/main/ch3/itoa.c)

[练习3-4](https://github.com/ZZy979/TCPL-code/blob/main/ch3/itoa2.c) 在数的对二的补码表示中，我们编写的`itoa`函数不能处理最小的负数，即n等于-2<sup>字长-1</sup>的情况。请解释其原因。修改该函数，使它在任何机器上运行时都能打印出正确的值。

[练习3-5](https://github.com/ZZy979/TCPL-code/blob/main/ch3/itob.c) 编写函数`itob(n, s, b)`，将整数n转换为b进制数，并将结果以字符串的形式保存到s中。

[练习3-6](https://github.com/ZZy979/TCPL-code/blob/main/ch3/itoa3.c) 修改`itoa`函数，使得该函数可以接收第三个参数。其中，第三个参数为最小字段宽度。为了保证转换后的字符串具有第三个参数指定的最小宽度，在必要时应在结果的左边填充一定的空格。（注：类似于`printf`函数中`%6d`指定的宽度）

## 3.7 break语句与continue语句
`break`语句可用于从`for`、`while`与`do-while`循环中提前退出，如同从`switch`语句中提前退出一样。`break`语句能使程序从**最内层**循环中立即跳出。

下面的`trim`函数用于删除字符串尾部的空格、制表符与换行符：

[trim函数](https://github.com/ZZy979/TCPL-code/blob/main/ch3/trim.c)

`continue`语句用于使`for`、`while`或`do-while`语句开始下一次循环的执行。在`while`与`do-while`语句中，这意味着立即执行测试部分；在`for`语句中，则意味着使控制转移到递增循环变量部分。`continue`语句只用于循环语句，不用于`switch`语句。

例如，下面这段程序用于处理数组a中的非负元素：

```c
for (i = 0; i < n; ++i) {
    if (a[i] < 0)  /* 跳过负元素 */
        continue;
    ...  /* 处理正元素 */
}
```

当循环后面的部分比较复杂时，常常会用到`continue`语句。

## 3.8 goto语句与标号
C语言提供了**可随意滥用**的`goto`语句以及标记跳转位置的标号。从理论上讲，`goto`语句是没有必要的，实践中不使用`goto`语句也可以很容易地写出代码。

但是，在某些场合下`goto`语句还是用得着的。最常见的用法是**跳出多层循环**。不能直接使用`break`语句，因为它只能跳出最内层循环。下面是使用`goto`语句的一个例子：

```c
    for (...)
        for (...) {
            ...
            if (disaster)
                goto error;
        }
    ...
error:
    // 处理错误情况
```

在该例子中，如果错误处理代码很重要，并且错误可能出现在多个地方，使用`goto`语句将会比较方便。

标号和变量名的形式相同，后面跟着一个冒号。标号可位于`goto`语句所在函数的任何语句前面，标号的作用域是整个函数。

所有使用了`goto`语句的代码都能改写成不带`goto`语句的，但可能会增加一些额外的重复测试或变量。

大多数情况下，使用`goto`语句的代码比不使用`goto`语句的代码要难以理解和维护，建议尽可能少地使用`goto`语句。
