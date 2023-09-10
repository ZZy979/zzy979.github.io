---
title: 《C++程序设计原理与实践》笔记 第7章 完成一个程序
date: 2022-10-30 02:49:06 +0800
categories: [C/C++, PPP]
tags: [cpp]
math: true
---
编写程序需要不断地改进你要实现的功能及其表达方式。第6章给出了一个能够工作的计算器程序的最初版本，本章将对其进一步改进。“完成程序”意味着使程序更易于使用和维护——包括改进用户接口、做一些仔细的错误处理工作、增加一些有用的特性、重构代码使之易于理解和修改。

## 7.1 引言
当程序第一次“正常”运行时，你大约只完成了一半的工作。一旦程序“基本可以工作”，真正的乐趣就开始了！此时我们可以在初步版本上试验各种不同的想法。

本章将引导你从一名专业程序员的角度来优化第6章的计算器程序。

## 7.2 输入和输出
6.3节的示例使用 "Expression:" 提示用户输入，使用 "Result:" 输出计算结果。然而表达式和结果才是我们真正关心的内容，另一方面应该把用户输入和程序输出区分开，因此使用了 "=" 表示结果。类似地，可以用一个简短的提示符 ">" 来提示用户输入：

```
> 2+3;
= 5
> 5*7;
= 35
> 
```

只需对`main()`函数的主循环做一点改动即可实现：

```cpp
cout << "> ";   // print prompt
while (cin) {
    Token t = ts.get();
    if (t.kind == 'q')  // 'q' for "quit"
        break;
    else if (t.kind == ';') {           // ';' for "print"
        cout << "= " << val << endl;    // print result
        cout << "> ";                   // print prompt
    }
    else
        ts.putback(t);
    val = parser.expression();
}
```

注：书中给出的代码有错误，会在每次输出结果前打印提示符：

```
> 2+3;
> = 5
5*7;
> = 35
```

但是，如果在一行中输入多个表达式，其输出仍然比较混乱：

```
> 2+3; 5*7; 2+9;
= 5
> = 35
> = 11
> 
```

要想不打印“多余”的提示符实现起来很麻烦，因为输出 ">" 时无法判断后面是否跟着 "=" 。根本原因是词法分析器忽略了换行符，无法区分用户输入的表达式在一行还是多行，要实现上述输出形式必须对`Token_stream`做较大改动。**为了获得一点小的改进而大改程序结构是不明智的**，因此决定保持现有输出形式。

## 7.3 错误处理
当你的程序“基本可以工作”时，你应该做的第一件事就是打破它——即尝试各种输入，以发现尽可能多的错误并修正，这种技术称为**测试**。关于“能否对程序进行系统测试从而发现所有的错误？”这一问题没有一个普适的答案。为了系统性地设计测试用例，可以使用正确的、不正确的和“不合理”的输入来测试程序。下面是为计算器程序设计的测试用例：

[计算器程序测试](https://github.com/ZZy979/PPP-code/blob/main/ch06/calculator_v1/parser_test.cpp)

计算器程序还存在一个问题。考虑下面两个输入："1+2; q" 和 "1+2 q" 。我们期望程序对于这两个输入都能在输出结果3之后退出，但实际上前者引发了 "primary expected" 错误（如6.6节结尾所述），而后者未打印任何结果直接退出了。重新审视`main()`函数的代码，发现在处理分号并打印结果后直接调用了`parser.expression()`，而不是检测字符q。`expression()`首先调用`term()`，`term()`首先调用`primary()`，`primary()`读取到q，而字符q不是`Primary`（数字或括号表达式），因此输出了错误信息。因此应该在检测分号之后检测字符q：

[解决 "q" 命令处理位置不正确的问题](https://github.com/ZZy979/PPP-code/commit/5505a85576e36f5754a0f2b17a64d75aa58a4c17)

接下来可以开始考虑从其他方面改进计算器程序了。

## 7.4 负数
计算器程序无法处理负数："-1/2" 将返回错误信息，而必须写成 "(0-1)/2" ，这是不可接受的。

在这种情况下，需要修改文法来支持一元符号(unary minus)，顺便也支持一元正号(unary plus)：

```
Primary:
    Number
    "(" Expression ")"
    "+" Primary
    "-" Primary
```

注：在第6章中，语法分析器的函数使用`if-else`实现。为了方便扩展，这里按书中代码的方式改用`switch`实现。

只需在语法分析器的`primary()`函数中增加两个`case`即可：

```cpp
case '+':
    return primary();
case '-':
    return -primary();
```

[支持负数](https://github.com/ZZy979/PPP-code/commit/d5795f0f47d101c3cce53fb379f781305c5705b0)

修改之后，以前一些“错误的”表达式将变为合法的，例如 "-1/2" 、 "+1" 、 "1++2" 等。

## 7.5 模运算：%
最初设计计算器表达式文法时包含了模运算%，但C++的`%`运算符不支持浮点数，因此未实现。现在重新考虑摸运算，实现很简单：
* 修改词法分析器，增加单词`%`
* 修改语法分析器，定义模运算

整数模运算很简单，而浮点数模运算的含义需要自己定义，有以下几种方式：
* 禁止浮点数模运算，如果操作数不是整数则报错（第一版书中采用的方法）
* 使用`fmod()`函数（第二版书中采用的方法）
* 使用`remquo()`函数

其中`fmod()`和`remquo()`函数都定义在头文件\<cmath\>中，区别只是商的取整方式不同，但都满足恒等式 $n=qm+r, q \in \mathbb{Z}, r \in [0, m)$.

例如：

| n | m | n / m | int(n / m) | fmod(n, m) | q | r |
| --- | --- | --- | --- | --- | --- | --- |
| 5.8 | 2.3 | 2.52174 | 2 | 1.2 | 3 | -1.1 |
| -5.8 | 2.3 | -2.52174 | -2 | -1.2 | -3 | 1.1 |
| 5.8 | -2.3 | -2.52174 | -2 | 1.2 | -3 | -1.1 |
| -5.8 | -2.3 | 2.52174 | 2 | -1.2 | 3 | 1.1 |

其中，`int(n / m)`是向零取整的商，`fmod(n, m)`是对应的余数，满足`n == m * int(n / m) + fmod(n, m)`；q是`remquo()`函数计算的向最近整数取整的商，r是对应的余数，满足`n = q * m + r`。

总的来说，浮点数模运算定义为**被除数减去除数乘以取整后的商**，即`n % m = n - m * [n / m]`。无论商采用哪种取整方式（向零取整、向下取整、向最近的整数取整），余数都用相同的方式计算。

回到计算器程序，只需在`Token_stream::get()`函数中增加

```cpp
case '(': case ')': case '+': case '-': case '*': case '/': case '%':
    return Token{ch};
```

之后在`Parser::term()`函数中增加

```cpp
case '%': {
    double d = primary();
    if (d == 0)
        throw Parser_error("divided by zero");
    left = fmod(left, d);
    op = ts.get();
    break;
}
```

[支持模运算](https://github.com/ZZy979/PPP-code/commit/069c17b09f888c97bc2049dbdac5db7b1d3b33cf)

## 7.6 清理代码
目前已经对程序做过几次改进，现在是一个很好的实际来重新检查代码，看能否使其更加清晰、简洁，改进注释等。换句话说，只有当程序达到易于他人接管和维护的状态才算是编写完成。

### 7.6.1 符号常量
在`primary()`函数中，使用`'8'`表示数值型单词很奇怪，必须使用注释说明：

```cpp
case '8':            // we use '8' to represent a number
    return t.value;  // return the number's value
```

这样不容易记忆，很容易出错。这里的`'8'`就是4.3.1节中提到的“魔数”。应该引入一个符号常量：

```cpp
const char number = '8';    // t.kind == number means that t is a number Token
```

这样`primary()`函数的相应代码片段就不再需要注释了：

```cpp
case number:
    return t.value;  // return the number's value
```

类似地，`Token_stream::get()`函数也相应修改：

```cpp
return Token{number, val};
```

**能够直接用代码表达清楚的就不要用注释。** 重复地使用注释来解释某些内容，通常表明代码应该改进了。

注：
* 如果按照6.9节的方法拆分源文件，则`number`常量应该定义在lexer.h中，因为lexer.cpp和parser.cpp都直接或间接地包含了lexer.h，因此都可以使用。
* 如果`primary()`函数使用`switch`实现，则`number`常量不能使用`extern`声明，必须直接在lexer.h中定义，否则编译器会报错“number不是常量表达式”。

运算符本身就能明确地表达其含义，因此不需要为每个单词引入一个符号常量。检查其他单词，只有`';'`和`'q'`有些不妥，为什么不用`'p'` (print)和`'e'` (exit)呢？在大型程序中，这种模糊而随意的表示方式迟早会引起问题。因此引入两个符号常量：

```cpp
const char quit = 'q';      // t.kind == quit means that t is a quit Token
const char print = ';';     // t.kind == print means that t is a print Token
```

并修改`Token_stream::get()`和`main()`函数。这样，如果之后决定改用`'p'`和`'e'`表示“打印”和“退出”就不需要修改`main()`函数。

字符串常量`"> "`和`"= "`也存在类似的问题，因此引入符号常量：

```cpp
const string prompt = "> ";     // prompt user input
const string result = "= ";     // used to indicate that what follows is a result
```

并相应修改`main()`函数。

注：由于这两个字符串只在`main()`函数中出现过，因此将其定义在main.cpp中即可。

[引入符号常量](https://github.com/ZZy979/PPP-code/commit/2e598f79f4959558b2a2cf57cb3d8e74ea503c68)

### 7.6.2 使用函数
**程序所使用的函数应该反应程序的结构，函数名应该标识代码中逻辑独立的部分。** 到目前为止，计算器程序在这方面做得很好：`expression()`、`term()`和`primary()`直接反映出我们对表达式文法的理解，`get()`则处理输入和单词识别。而`main()`做了逻辑上相互独立的两件事：
* 程序的整体框架：启动程序、结束程序、处理错误
* 计算循环

一种显然的改进方法是将计算循环抽取成一个单独的函数`calculate()`：

[将计算循环抽取成单独的函数](https://github.com/ZZy979/PPP-code/commit/585e79fc6e7c81e84cb400190e18f93482c85286)

修改后的代码更直接地反映了程序结构，因此更易于理解。

### 7.6.3 代码布局
寻找丑陋的代码，发现

```cpp
switch (ch) {
    case print:
    case quit:
    case '(': case ')': case '+': case '-': case '*': case '/': case '%':
        return Token{ch};   // let each character represent itself
    ...
}
```

在加入了`print`、`quit`和`'%'`后变得有些混乱，可以让每个`case`单独占一行并添加一些注释。

也可以把每个数字单独放在一行，但那样似乎并不能使代码更加清晰，而且会导致`get()`函数的代码太长而不能完整显示在屏幕上。理想情况下，每个函数的代码都能完整显示在一个屏幕中——**屏幕水平或竖直方向之外看不到的地方是最有可能隐藏bug的地方。** 因此，代码布局非常重要。

在清理代码时，有可能会意外地引入错误。因此，在清理之后一定要重新测试代码。最好每做一点改动就测试一次，以便出现错误时还能记得做了什么改动。记住：**及早测试、经常测试**。

### 7.6.4 注释
好的注释是代码的重要组成部分。当我们回过头来清理代码时，是一个很好的时机来检查程序每一部分所写的注释
* 是否仍然有效（写完注释后可能又修改了代码）
* 对读者来说是否充分（通常不是）
* 是否简短清晰，不至于分散读者的注意力

**最好的注释就是让代码本身来表达。** 避免用注释来解释一些意义已经很明确的代码，例如：

```cpp
x = b + c;  // add b and c and assign the result to x
```

注释用于代码无法表达的内容。例如程序的目的：代码只能表达它做了什么，但不能表达它的目的。对于计算器程序，表达式文法很适合放在注释或文档中：

[增加注释](https://github.com/ZZy979/PPP-code/commit/964c691e0b88d6de545381a092955eaf0c5c67ba)

这里使用了**块注释**(block comment)，从`/*`开始，到`*/`结束，可以跨越多行。

## 7.7 错误恢复
目前的程序遇到错误时直接退出，但是可以给出一个错误信息并继续运行，毕竟用户经常会造成小的输入错误。因此，下面尝试从错误中恢复。这意味着需要捕获异常，清理遗留的“混乱”之后继续运行。

目前，所有的异常都由`main()`处理。为了实现错误恢复，`calculate()`函数必须捕获异常，并在计算下一个表达式之前清理“混乱”(clean up mess)：

```cpp
void calculate() {
    while (cin) {
        try {
            // ...
        }
        catch (exception& e) {
            cerr << e.what() << endl;
            clean_up_mess();
        }
    }
}
```

“清理混乱”是必要的，因为错误处理之后准备好进行下面的计算意味着所有数据都处于良好的、可预测的状态。在计算器程序中，`Token_stream`是唯一保存在函数之外的数据，因此要清理与错误表达式相关的所有单词，避免影响下一个表达式。

例如， "1\*\*2+3; 4+5;" 的第二个 "\*" 将引发一个异常，此时 "2+3; 4+5;" 仍然留在`cin`的缓冲区中。有两种选择：
* 清除所有单词，即 "2+3; 4+5;"
* 清除与当前表达式相关的所有单词，即 "2+3;"

这里选择第二种方式。

`clean_up_mess()`函数的一种实现是使用`get()`读取单词直到遇到分号：

```cpp
void clean_up_mess() {
    while (ts.get().kind != print)
        ;
}
```

但是，这种方式并不能处理所有的情况。例如，输入 "1@z; 1+3;" ， "@" 会导致程序进入`calculate()`函数`while`循环的`catch`子句，进而调用`clean_up_mess()`查找下一个分号；而`get()`函数读取到 "z" 时，由于 "z" 不是一个单词，因此产生另一个错误，导致程序退出。

**错误处理很困难，而错误处理过程中发生的错误更难处理。** 因此需要设计一种不抛出异常就能从`Token_stream`清除字符的方法。计算器程序获取输入的唯一途径是`get()`函数，但它会抛出异常。因此需要一个新的操作`ignore(c)`，用于从输入中读取字符（并丢弃），直到遇到指定的**字符**（不是单词）。`clean_up_mess()`函数只需调用`ts.ignore(print)`即可。

完整改动如下：

[错误恢复](https://github.com/ZZy979/PPP-code/commit/be97ff2f7dc92d82f448864301ae98b6cbd18cb3)

注：虽然现在程序能够从错误中恢复，但输出会变得更混乱，例如：

```
> 1**2+3; 4+5;
= > = 9
primary expected
> 1@z; 1+3;
= > = 4
Bad token
> 
```

错误处理总是很棘手的。由于很难想像到会出现什么样的错误，因此需要进行大量的实验与测试。错误处理的质量是程序员专业程度的标志。

## 7.8 变量
计算器程序目前已经工作得很好了，下面进一步完善程序功能：支持变量。变量能够使得用户更好地表达更长的计算，另外希望支持内置的命名常量，例如`pi`和`e`。

增加变量和常量是对计算器程序的重大扩展，会改动程序的大部分代码。如果没有充分的理由和时间，最好不要着手做这种扩展。

为了支持变量，需要对程序做以下改动：
* 增加`Variable`类表示一个变量，使用`vector<Variable>`存储所有的变量和常量
* 文法：增加声明和使用变量的产生式规则
* 词法分析器：增加标识符、`let`和`=`三种单词，并在`Token`类增加成员`name`表示变量名
* 语法分析器：增加或修改函数，以支持声明和使用变量
* 错误处理：非法变量名、重复定义、使用未声明的变量

### 7.8.1 变量和定义
显然，对于变量和内置常量，关键是保存(name, value)对，从而可以通过名字来访问相应的值。为此使用一个**符号表**(symbol table)来存储变量（将其抽象为一个类`Symbol_table`）。符号表可以使用`vector<Variable>`实现，但使用`unordered_map<string, double>`更简单。

[符号表](https://github.com/ZZy979/PPP-code/blob/main/ch07/calculator_v2/variable.h)

`Symbol_table`类支持以下四个操作：
* `get_value(name)`：查询指定变量的值，如果不存在则抛出异常
* `set_value(name, value)`：设置指定变量的值，如果不存在则抛出异常
* `is_declared(name)`：判断指定的变量是否已定义
* `define_name(name, value)`：定义变量，如果已存在则抛出异常

### 修改文法
下面考虑在计算器程序中声明变量的语法。可以考虑采用C++的语法：`double var = 7.2;`，但计算器中所有变量都是`double`类型，而省略`double`又无法与赋值操作区分：`var = 7.2;`。因此选择关键字`let`，例如`let var = 7.2;`。

相关文法如下：

```
Calculation:
    Statement
    Calculation Statement
    Print
    Quit
Statement:
    Declaration
    Expression
Declaration:
    "let" Name "=" Expression
```

除此之外，还需要增加产生式`Primary -> Name`，以及单词`identifier`（标识符）：

```
Primary:
    ...
    Name
Name:
    identifier
```

### 7.8.2 引入名字（修改词法分析器）
新增的文法引入了三种新的单词：`=`、`let`和标识符，其中`=`可以像普通运算符一样用其本身表示类别，`let`用`'L'`表示，标识符用`'a'`表示。类似于浮点数，标识符除了类别外还有“值”，在这里是变量名，因此需要给`Token`增加一个字符串成员`name`。

注：创建标识符单词时只需给`kind`和`name`两个成员赋值，但C++的列表初始化语法不支持跳过第二个成员`value`，因此需要通过构造函数来实现。

另外，必须修改`Token_stream::get()`函数来识别这些单词。单词`=`只需在`switch`语句中增加一个`case`即可，而`let`和标识符需要在`default`部分判断。“标识符”的定义是**由字母和数字组成且不以数字开头的字符串**，但这里要对`let`单独处理。

注：`istream::get()`函数用于读取单个字符，与`>>`运算符的区别是不会跳过空白符。

完整改动如下：

[修改词法分析器](https://github.com/ZZy979/PPP-code/commit/c30b303e40153ec35b95c0aad6b7cf59a544be00)

其中，函数`isalpha(c)`用于判断字符`c`是否是字母(A-Z, a-z)，定义在头文件\<cctype\>中，详见11.6节。

### 修改语法分析器
在修改后的文法中，`Calculation`是新的顶层产生式规则，表示`calculate()`中的计算循环，它依赖`Statement`处理表达式和声明。非终结符`Statement`对应处理函数的伪代码如下：

```
Statement() {
    t = get_token()
    if (t == "let")
        return Declaration()
    else {
        putback(t)
        return Expression()
    }
}
```

现在可以用`statement()`代替`calculate()`中的`expression()`。

函数`statement()`在遇到单词`let`后调用函数`declaration()`，该函数应该处理变量声明的剩余部分，即`Name "=" Expression`，其伪代码如下：

```
Declaration() {
    t = get_token()
    if (t != name)
        error()

    t2 = get_token()
    if (t2 != '=')
        error()

    d = Expression()
    define_name(t.name, d)
    return d
}
```

另外，`primary()`函数需要增加一个`case`，当遇到标识符时返回对应变量的值：

```cpp
double Parser::primary() {
    Token t = ts.get();
    switch (t.kind) {
        // ...
        case name:
            return var_table.get_value(t.name);
        // ...
    }
}
```

注：
* 由于语法分析器的成员函数`declaration()`和`primary()`都需要访问符号表，因此将其作为`Parser`类的成员。
* 非法变量名（如`1a`、`a$`）由词法分析器或语法分析器处理；重复定义、使用未声明的变量由符号表处理。
* 赋值语句将在习题2实现。

完整改动如下：

[修改语法分析器](https://github.com/ZZy979/PPP-code/commit/683a544e6732bd9d27468bf5cf9b7e4e8fbc638d)

### 7.8.3 预定义名字
现在可以很容易地预定义一些常用的名字，例如`pi`和`e`。可以将这些定义放在`Parser`类的构造函数中。

[增加预定义名字](https://github.com/ZZy979/PPP-code/commit/1756a2ada7937167084e78a8ffcd9dfbb5ce4d37)

### 7.8.4 我们到达目的地了吗
并没有。我们对程序做了很多修改，需要对程序进行测试、清理代码和修改注释等。另外，还可以做更多的扩展，例如提供赋值运算符（习题2）、区分变量和常量（习题3）。

一般来说，如果一个改进会使程序的代码量和复杂度增加50%，则更像是基于原来的版本重写了一个新的程序。特别地，像计算器程序这样分阶段编写和测试程序，比一下子就写完整个程序要好得多。

## 简单练习
[支持函数](https://github.com/ZZy979/PPP-code/commit/e8119bd8c104890cd12b031d200c477bee98995e)

## 习题
[支持赋值运算符、常量，增加帮助信息](https://github.com/ZZy979/PPP-code/commit/b442f139f6b78058718dc24afb220bd05e95775e)
