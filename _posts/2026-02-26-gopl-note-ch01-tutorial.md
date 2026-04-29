---
title: 《Go程序设计语言》笔记 第1章 入门
date: 2026-02-26 22:05:17 +0800
categories: [Go]
tags: [go, hello world, package, import statement, command-line argument, slice, comment, assignment, for statement, if statement, map, io, formatting, file access, image processing, constant, struct, http, concurrency, goroutine, channel]
---
本章介绍Go语言的基本组件。本章将展示可以用Go编写的程序的多样性，从简单的文件处理、图形，到并发的网络客户端和服务器。

## 1.1 Hello, World
我们从经典的 "hello, world" 例子开始。

[gopl.io/ch1/helloworld](https://github.com/ZZy979/gopl.io/blob/main/ch1/helloworld/main.go)

Go是一门编译型语言，Go工具链会将源程序及其依赖转换成本地机器指令。这些工具可以通过`go`命令来调用。该命令有许多子命令，其中最简单的是`run`，该命令会编译一个或多个名字以.go结尾的源文件，将其与库链接，然后运行生成的可执行文件。在gopl.io/ch1/helloworld目录中执行以下命令：

```shell
$ go run main.go
Hello, 世界
```

或者在代码根目录gopl.io中执行以下命令：

```shell
$ go run gopl.io/ch1/helloworld
```

Go原生支持Unicode，因此可以处理全世界任何语言的文本。

如果程序不只是一次性实验，那么你可能希望将编译结果保存下来以备将来之用。这可以用`go build`命令来实现，在代码根目录中执行：

```shell
$ go build gopl.io/ch1/helloworld
```

这会创建一个名为helloworld的二进制可执行文件（注：在Windows上是helloworld.exe），可以像这样运行：

```shell
$ ./helloworld
Hello, 世界
```

注：在Windows上直接输入`helloworld.exe`或`helloworld`。

本书中的所有例子都有一个标签，例如`gopl.io/ch1/helloworld`。如果运行命令`go get gopl.io/ch1/helloworld`，就会自动获取源代码并放在`GOPATH`环境变量对应的目录中。详见2.6和10.7节。

Go代码组织成**包**(package)。每个包由同一个目录中的一个或多个.go源文件组成。每个源文件都以`package`声明开头，声明该文件所属的包，后面跟着一系列`import`声明（该文件导入的其他包），再之后是文件本身的代码。

Go标准库有100多个包。例如，`fmt`包包含用于打印格式化输出和读取输入的函数。`Println()`是其中的一个基本输出函数，用于打印一个或多个值，用空格分隔，并在末尾添加换行符。

`main`是一个特殊的包，它定义了一个独立的可执行程序，而不是一个库。其中的`main()`函数是程序的入口。

`import`声明告诉编译器这个源文件需要用到哪些包。**必须准确地导入所需的包。** 如果缺少导入或存在不必要的导入，程序将编译失败。这一严格要求防止了随着程序的演化累积越来越多未使用的包。

`import`声明必须紧跟在`package`声明之后。在此之后是函数、变量、常量和类型的声明（分别由关键字`func`、`var`、`const`和`type`引入）。对于这些部分，声明的顺序无关紧要。

Go不要求在语句或声明的末尾使用分号，除非两个或多个语句出现在同一行。事实上，特定符号后面的换行符会被转换为分号，因此换行符的位置对Go代码的正确解析很重要。例如，函数的左花括号`{`必须在`func`声明同一行的末尾，而不是单独一行；在表达式`x + y`中，允许在`+`之后添加换行符，但不能在`+`之前（注：以`+`结尾不会添加分号，但以`x`结尾会添加分号，导致编译失败，原因是`+y`的值没有被使用）。

Go对于代码格式采取很强硬的态度。`gofmt`工具会将代码重写为标准格式（没有任何可调整的选项），`go fmt`子命令将`gofmt`应用于指定包（默认为当前目录）中的所有文件。另一个相关的工具`goimports`会根据需要添加和删除`import`声明。这个工具并不在标准分发包中，但可以使用以下命令安装：

```shell
go get golang.org/x/tools/cmd/goimports
```

## 1.2 命令行参数
程序获取输入的一种方式是**命令行参数**(command-line arguments)。程序可以通过`os`包中名为`Args`的变量来访问命令行参数。

变量`os.Args`是一个字符串的**切片**(slice)。切片是Go中的一个基本概念，将在4.2节详细介绍。目前可以将切片视为动态大小的数组`s`，其中单个元素为`s[i]`，可以用`s[m:n]`访问连续子序列，元素数量为`len(s)`。与大多数编程语言一样，Go中的索引使用左闭右开区间：包括第一个索引但不包括最后一个。例如，切片`s[m:n]`包含n-m个元素(`s[m]`~`s[n-1]`)，其中0 ≤ m ≤ n ≤ `len(s)`。

命令行参数的第一个元素`os.Args[0]`是可执行文件的名字（例如上一个例子中的`helloworld`或`helloworld.exe`）。其他元素是执行程序时提供的参数，可以通过切片`os.Args[1:len(os.Args)]`访问。在切片表达式`s[m:n]`中，如果省略`m`或`n`，则分别默认为0和`len(s)`。因此之前的切片可以简写为`os.Args[1:]`。

下面的例子是Unix `echo`命令的简化版实现，用于在一行内打印其命令行参数。这个程序导入了两个包，并用括号括起来，而不是使用两个单独的`import`声明。两种形式都是合法的，但习惯上使用第一种。导入的顺序无关紧要，`gofmt`工具会将包名按照字母顺序排序。

[gopl.io/ch1/echo1](https://github.com/ZZy979/gopl.io/blob/main/ch1/echo1/main.go)

可以像这样运行该程序：

```shell
$ go run gopl.io/ch1/echo1 hello world
hello world
```

**注释**(comment)以`//`开头。从`//`到行尾的所有文本都是注释，编译器会忽略这些文本。按照惯例，在包声明之前用注释来描述这个包；对于`main`包，此注释是一个或多个完整的句子，用于描述整个程序。

关键字`var`用于声明**变量**(variable)。这个程序声明了两个`string`类型的变量`s`和`sep`。变量可以在声明时初始化。如果没有显式初始化，则会隐式初始化为其类型的**零值**(zero value)：对于数值类型为0，对于字符串为空串`""`。因此在这个例子中，`s`和`sep`都隐式初始化为空字符串。第2章将详细介绍变量和声明。

对于数值，Go提供了通常的算术和逻辑运算符。而对于字符串，`+`运算符表示拼接。因此表达式`sep + os.Args[i]`表示字符串`sep`和`os.Args[i]`的拼接。

语句`s += sep + os.Args[i]`是一个**赋值语句**(assignment statement)，等价于`s = s + sep + os.Args[i]`。运算符`+=`是**赋值运算符**(assignment operator)。每个数值和逻辑运算符（如`+`和`*`）都有对应的赋值运算符。

`echo1`程序通过不断地向末尾添加文本构建了一个字符串，这个过程具有二次时间复杂度（所有参数长度的前缀累加和），当参数数量巨大时可能会很低效。稍后将会展示一些改进版本来解决这一问题。

在`for`循环的第一部分声明了循环索引变量`i`。符号`:=`是**短变量声明**(short variable declaration)，该语句声明一个或多个变量并初始化，根据初始值赋予适当的类型。短变量声明只能在函数内使用。

自增语句`i++`将`i`加1，等价于`i += 1`和`i = i + 1`。对应的自减语句`i--`将`i`减1。与C语言不同，在Go中这些是语句而不是表达式，因此`j = i++`是非法的。另外，它们只有后缀形式，因此`++i`也是非法的。

`for`循环是Go中唯一的循环语句。它有多种形式，其中一种如下：

```go
for initialization; condition; post {
	// zero or more statements
}
```

`for`循环的三部分周围没有圆括号。但**花括号是强制的，并且左花括号必须和`post`语句在同一行**（注：三部分之间可以换行）。`initialization`语句是可选的，在循环开始之前执行。如果存在，必须是一个**简单语句**，即短变量声明、赋值语句或函数调用。`condition`是一个布尔表达式，在每次循环开始时求值，如果值为`true`则执行循环体中的语句。`post`语句在循环体之后执行，然后进入下一次循环。当条件变为`false`时循环结束。

这三部分中的任何一个都可以省略。如果只有条件部分，则分号也可以省略（此时等价于传统的`while`循环）：

```go
// a traditional "while" loop
for condition {
	// ...
}
```

如果在任何形式中把条件完全省略，就变成无限循环。例如：

```go
// a traditional infinite loop
for {
	// ...
}
```

无限循环可以用其他方式终止，例如`break`或`return`语句。

另一种形式的`for`循环遍历一个**范围**(range)内的值，例如字符串或切片。下面是第二个版本的`echo`：

[gopl.io/ch1/echo2](https://github.com/ZZy979/gopl.io/blob/main/ch1/echo2/main.go)

在每次循环中，`range`都会生成一对值：索引和对应的元素。这种方式比`echo1`中手动计算索引更简单。在这个例子中不需要索引，但Go不允许未使用的局部变量（会导致编译错误）。解决方法是使用**空白标识符**(blank identifier)，名为`_`。

下面几种声明变量的方式是等价的：

```go
s := ""
var s string
var s = ""
var s string = ""
```

在实际中，一般只使用前两种形式。

正如前面提到的，在循环中使用`+=`拼接字符串是很低效的。一种更简单且高效的方式是使用`strings`包中的`Join()`函数：

[gopl.io/ch1/echo3](https://github.com/ZZy979/gopl.io/blob/main/ch1/echo3/main.go)

如果不关心格式，只是想查看值（例如为了调试），可以让`Println()`进行格式化：

```go
fmt.Println(os.Args[1:])
```

这个语句的输出和`strings.Join()`类似，但用方括号括起来（例如`[foo bar]`）。所有切片都会以这种格式打印。

练习1.1 修改`echo`程序，使其打印`os.Args[0]`（命令名称）。

练习1.2 修改`echo`程序，打印每个参数的索引和值，每行一个。

练习1.3 测量程序`echo2`和`echo3`运行时间的差异（1.6节介绍了`time`包，11.4节说明了如何编写基准测试以进行系统的性能评估）。

随机生成100000个长度为10的字符串，分别使用两种方式进行拼接。测试结果：`strings.Join()`耗时5.57 ms，循环拼接耗时89294.05 ms。

## 1.3 查找重复的行
本节将展示一个名为`dup`的程序的三种变体，该程序用于查找相邻的重复行（灵感来自Unix的`uniq`命令）。

第一个版本的`dup`打印标准输入中出现多次的行，并在前面加上次数。这个程序引入了`if`语句、`map`数据类型和`bufio`包。

[gopl.io/ch1/dup1](https://github.com/ZZy979/gopl.io/blob/main/ch1/dup1/main.go)

与`for`一样，`if`语句的条件不加圆括号，但语句体的花括号是必需的。可选的`else`部分会在条件为`false`时执行。

**映射**(map)是存储键/值对的数据结构，能够提供常数时间的存取操作。键可以是任意能够用`==`比较的类型，值可以是任意类型。在这个例子中，键是字符串，值是整数。使用内置函数`make()`来创建空映射。4.3节将会详细讨论映射。

`dup`程序每次读取一行输入，就会将该行作为映射的键，将相应的值加1。语句`counts[input.Text()]++`等价于

```go
line := input.Text()
counts[line] = counts[line] + 1
```

如果映射不包含指定的键就返回值类型的零值。当遇见一个新行时，右侧的`counts[line]`表达式的值为0。

为了打印结果，程序使用基于`range`的`for`循环来遍历`counts`映射。在每次循环中生成两个结果：键和对应的值。**映射的迭代顺序是随机的**，每次运行都不同。这种设计是有意为之，为了防止程序依赖于任何特定的顺序。

`bufio`包有助于高效便捷地处理输入和输出。其中最有用的特性之一是`Scanner`类型，可以读取输入并拆分为行或单词。这通常是处理按行输入的最简单方式。

程序中创建了读取标准输入的scanner：

```go
input := bufio.NewScanner(os.Stdin)
```

每次调用`input.Scan()`都会读取下一行，并删除末尾的换行符，通过`input.Text()`获取结果。如果没有更多输入，则`Scan()`返回`false`。

函数`fmt.Printf()`类似于C语言的`printf()`函数，用于格式化输出。第一个参数是**格式字符串**，用于指定后续参数的格式。每个参数的格式由**转换字符**（百分号后的字母）决定。例如，`%d`表示十进制整数，`%s`表示字符串。

`Printf()`有很多这样的转换，Go程序员称之为**动词**(verb)。下表列出了常用的动词，完整列表参见文档[fmt package - Printing](https://pkg.go.dev/fmt#hdr-Printing)。

| 动词 | 含义 |
| --- | --- |
| `%d` | 十进制整数 |
| `%x`, `%o`, `%b` | 十六进制、八进制、二进制整数 |
| `%f`, `%e`, `%g` | 浮点数：固定位数、科学计数法、自动选择 |
| `%t` | 布尔值 |
| `%c` | rune（Unicode码点） |
| `%s` | 字符串 |
| `%q` | 带引号的字符串或rune |
| `%v` | 以默认格式打印任何值 |
| `%T` | 值的类型 |
| `%%` | 百分号 |

`dup1`中的格式字符串还包含制表符`\t`和换行符`\n`。字符串字面值可以包含这样的**转义序列**(escape sequence)，用于表示不可见的字符。

`Printf()`不会自动添加换行符。按照惯例，名字以`f`结尾的格式化函数（如`log.Printf()`和`fmt.Errorf()`）使用`fmt.Printf()`的格式规化则，而名字以`ln`结尾的函数遵循`Println()`的规则：用`%v`格式化其参数，并添加换行符。

很多Unix程序都可以从一系列文件读取输入，如果未指定文件则读取标准输入。下一个版本的`dup`就采用了这种方式。

[gopl.io/ch1/dup2](https://github.com/ZZy979/gopl.io/blob/main/ch1/dup2/main.go)

使用`os.Open()`打开文件，该函数返回两个值。第一个是文件指针(`*os.File`)，用于`Scanner`读取（注：`os.Stdin`是表示标准输入的文件指针）。第二个是内置`error`类型的值，如果`err`等于`nil`说明文件成功打开。然后读取文件，完成后使用`Close()`关闭文件以释放资源。如果`err`不是`nil`则说明出现了错误。这个程序简单地使用`Fprintf()`在标准错误流打印一条消息，然后继续处理下一个文件。`continue`语句跳转到`for`循环的下一次迭代。

本书早期的例子对于错误处理比较随意，忽略了不太可能发生的错误（例如使用`input.Scan()`读取文件，在代码中进行了标注）。5.4节将详细介绍错误处理。

注意，`countLines()`函数的调用在声明之前。函数和其他包级别的实体可以以任意顺序声明。

映射是一种**引用**(reference)类型（类似于C++的指针）。当映射被传递给函数时，函数会接收到引用的副本（即形参和实参是指向同一对象的两个“指针”），因此被调用函数对映射所做的修改对于调用者是可见的。

以上版本的`dup`是以“流”模式处理的，即按需读取输入并拆分成行，因此理论上这些程序可以处理任意数量的输入。另一种方式是一次性将整个输入读到内存中，将其分割为行然后处理。下面的`dup3`就采用了这种方式。它使用`os.ReadFile()`函数读取指定文件的全部内容，并使用`strings.Split()`将字符串分割为子字符串的切片（`strings.Join()`的反向操作）。

因为`ReadFile()`需要文件名，因此`dup3`只读取文件，不读取标准输入。另外，我们将行计数逻辑移回到`main()`，因为现在只有一个地方需要。

[gopl.io/ch1/dup3](https://github.com/ZZy979/gopl.io/blob/main/ch1/dup3/main.go)

`ReadFile()`返回一个`byte`切片，需要将其转换为`string`。

在底层，`bufio.Scanner`和`os.ReadFile()`都使用了`*os.File`的`Read()`方法（来自`io.Reader`接口）。

练习1.4 修改`dup2`，打印每个重复的行出现的所有文件的名称。

## 1.4 GIF动画
下一个程序演示了Go标准库`image`包的基本用法。我们将使用它创建一系列位图图像，然后将其编码为GIF动画。这些图像叫做**李萨如图形**(Lissajous figures)，是由两个维度的简谐震荡（例如输入到示波器x和y方向的两个正弦波）合成的曲线。图1.1显示了一些示例。

![李萨如图形1](/assets/images/gopl-note-ch01-tutorial/李萨如图形1.png) ![李萨如图形2](/assets/images/gopl-note-ch01-tutorial/李萨如图形2.png) ![李萨如图形3](/assets/images/gopl-note-ch01-tutorial/李萨如图形3.png) ![李萨如图形4](/assets/images/gopl-note-ch01-tutorial/李萨如图形4.png)

[gopl.io/ch1/lissajous](https://github.com/ZZy979/gopl.io/blob/main/ch1/lissajous/main.go)

导入了路径包含多个部分的包（如`image/color`）后，就可以用最后一个部分（如`color`）引用这个包。因此变量`color.White`属于`image/color`包，`gif.GIF`属于`image/gif`包。

`const`声明（3.6节）用于为常量命名。**常量**(constant)即在编译时固定的值。与`var`声明一样，`const`声明可以出现在包级别（在整个包中可见）或函数内（仅在该函数内可见）。常量的值必须是数字、字符串或布尔值。

表达式`[]color.Color{...}`和`gif.GIF{...}`是**复合字面值**(composite literal)（4.2和4.4.1节），是一种从元素值实例化复合类型的紧凑语法。在这里第一个是切片，第二个是结构体。

类型`gif.GIF`是一个结构体类型（4.4节）。**结构体**(struct)是一组**字段**(field)的集合（通常是不同类型的）。变量`anim`是`gif.GIF`类型的结构体。结构体字面值`gif.GIF{LoopCount: nframes}`创建了一个结构体，其`LoopCount`字段设置为`nframes`，其他所有字段都为零值。可以使用`.`访问结构的各个字段，例如`anim.Delay`。

`lissajous()`函数有两层嵌套循环。外层循环运行64次，每次生成动画的一帧。它使用包含两种颜色（白色和黑色）的调色板创建了一个新的201×201大小的图像。所有像素最初都设置为调色板的零值（白色）。在内层循环中通过将一些像素设置为黑色来生成一个新图像，之后使用内置函数`append()`将结果添加到`anim`的帧列表中，并指定80 ms的延迟。最后，帧和延迟序列被编码为GIF格式，并写入输出流`out`。

内层循环计算两个震荡。x震荡就是正弦函数。y震荡也是正弦曲线，但其相对于x震荡的频率是0~3的随机数，相位初始为0、每帧递增。内层循环会运行直到x震荡完成5个完整周期。在每一步中，使用`SetColorIndex()`将(x, y)对应的像素设置为黑色。

`main()`函数调用`lissajous()`函数，使其写入标准输出。使用以下命令生成GIF动画：

```shell
$ go build gopl.io/ch1/lissajous
$ ./lissajous > lissajous.gif
```

![lissajous](/assets/images/gopl-note-ch01-tutorial/lissajous.gif)

练习1.5 修改`lissajous`程序的调色板，改为黑色背景、绿色曲线。使用`color.RGBA{0xRR, 0xGG, 0xBB, 0xff}`创建颜色，其中十六进制数`RR`、`GG`和`BB`分别表示红、绿、蓝分量。

练习1.6 修改`lissajous`程序，通过向`palette`添加更多值，然后以某种有趣的方式修改`SetColorIndex()`的第三个参数来生成多种颜色的图像。

## 1.5 获取URL
对于许多应用程序来说，访问互联网信息与访问本地文件系统同样重要。Go提供了一组`net`包，可以很容易地通过互联网发送和接收信息、建立底层网络连接以及创建服务器。

为了说明通过HTTP获取信息的最简单方式，下面的程序`fetch`获取指定URL的内容并将其文本打印出来（灵感来自Unix命令`curl`）。我们将在本书中经常使用这个程序。

[gopl.io/ch1/fetch](https://github.com/ZZy979/gopl.io/blob/main/ch1/fetch/main.go)

这个程序使用了`net/http`和`io`包中的函数。`http.Get()`函数发起HTTP请求，如果没有错误则返回响应结构体`http.Response`，其`Body`字段包含可读取的服务器响应流。之后，`io.ReadAll()`读取整个响应，结果保存在`byte`切片中。最后关闭`Body`流以避免资源泄露，并使用`Printf()`将响应写到标准输出。

像这样运行该程序：

```shell
$ go build gopl.io/ch1/fetch
$ ./fetch http://gopl.io
<html>
<head>
  <title>The Go Programming Language</title>
...
```

如果HTTP请求失败，使用`os.Exit(1)`终止进程，程序会报告错误：

```shell
$ ./fetch http://bad.gopl.io
fetch: Get "http://bad.gopl.io": dial tcp: lookup bad.gopl.io: no such host
exit status 1
```

练习1.7 函数`io.Copy(dst, src)`将数据从`src`复制到`dst`。使用该函数而不是`io.ReadAll()`将响应体复制到`os.Stout`，从而不需要缓冲区来存放整个响应。务必检查函数返回的错误。

练习1.8 修改`fetch`，向每个缺少前缀`http://`的参数URL添加该前缀。使用函数`strings.HasPrefix()`。

练习1.9 修改`fetch`，同时打印HTTP状态代码`resp.Status`。

## 1.6 并发获取URL
Go语言最有趣、最新颖的特性之一是对并发编程的支持。这是一个很大的话题，将在第8章和第9章专门讨论。现在仅仅尝试一下Go的主要并发机制——goroutine和channel。

下一个程序`fetchall`与前一个示例相同，但它并发地获取多个URL。因此所需的时间不会超过最长的获取时间，而不是所有获取时间的总和。这个版本的`fetchall`会丢弃响应，而是报告每个响应的大小和花费的时间。

[gopl.io/ch1/fetchall](https://github.com/ZZy979/gopl.io/blob/main/ch1/fetchall/main.go)

下面是一个运行示例：

```shell
$ go build gopl.io/ch1/fetchall
$ ./fetchall https://go.dev https://pkg.go.dev https://www.gopl.io/
1.00s    64029  https://go.dev
1.31s    33470  https://pkg.go.dev
1.42s     4154  https://www.gopl.io/
1.42s elapsed
```

**goroutine**是一种并发的函数执行。**channel**是一种通信机制，允许一个goroutine将指定类型的值传递给另一个goroutine。`main()`函数运行在主goroutine中，`go`语句创建额外的goroutine。

`main()`函数使用`make()`创建了一个字符串channel。第一个循环中的`go`语句对于命令行参数中的每个URL启动一个新的goroutine，异步地调用`fetch()`获取URL。`io.Copy()`函数读取响应体，并通过写入`io.Discard`将其丢弃，返回拷贝的字节数。当结果到达时，`fetch()`向channel `ch`发送一行摘要（耗时或错误信息）。`main()`中的第二个循环接收并打印这些行。

当一个goroutine尝试在channel上发送或接收时，它会阻塞，直到另一个goroutine执行相应的接收或发送操作，此时值会被传输，然后两个goroutines继续执行。在这个例子中，每个`fetch()`会发送一个值(`ch <- expression`)，`main()`接收所有这些值(`<-ch`)。让`main()`执行所有打印操作可以确保每个goroutine的输出作为一个单元进行处理，避免两个goroutine同时完成时输出相互交错。

练习1.10 找一个会产生大量数据的网站，调研缓存机制：连续两次运行`fetchall`并查看输出的时间是否变化很大。修改`fetchall`将其输出打印到文件中，以便检查每次得到的内容是否相同。

练习1.11 尝试使用更长的参数列表运行`fetchall`，例如从alexa.com上排名前100万的网站中选取。如果一个网站一直没有响应，程序会怎么样？（8.9节描述了处理这种情况的机制）

## 1.7 Web服务器
Go标准库使得编写Web服务器非常容易。在本节中，将展示一个微型服务器，返回请求URL的路径部分。也就是说，如果请求是 http://localhost:8000/hello ，则响应是`URL.Path = "/hello"`。

[gopl.io/ch1/server1](https://github.com/ZZy979/gopl.io/blob/main/ch1/server1/main.go)

该程序只有十几行，这是因为库函数完成了大部分工作。`main()`函数将处理器(handler)函数`echo()`关联到以`/`开头的URL（即所有请求），并在8000端口启动服务器监听请求。请求表示为`http.Request`类型的结构体，其中`URL`字段表示请求的URL。当请求到达时，它被传递给`echo()`函数，该函数从请求URL中提取路径部分（如`/hello`），并将其发送到响应。Web服务器将在第7.7节中详细介绍。

在macOS或Linux上，可以在命令末尾添加`&`在后台启动服务器。在Windows上，需要在单独的命令窗口中运行命令，按Ctrl+C终止。

```shell
$ go run gopl.io/ch1/server1 &
```

然后可以使用1.5节中的`fetch`程序发送客户端请求：

```shell
$ go build gopl.io/ch1/fetch

$ ./fetch http://localhost:8000
URL.Path = "/"

$ ./fetch http://localhost:8000/help
URL.Path = "/help"
```

或者，也可以在浏览器中访问服务器，如下图所示。

![echo服务器的响应](/assets/images/gopl-note-ch01-tutorial/echo服务器的响应.png)

向这个服务器添加功能很容易。一个有用的功能是返回服务器某种状态的URL。这个版本的服务器会对请求进行计数，访问URL `/count`返回当前计数（不包括`/count`本身的请求）。

[gopl.io/ch1/server2](https://github.com/ZZy979/gopl.io/blob/main/ch1/server2/main.go)

这个服务器有两个处理器函数，请求URL决定调用哪一个：对`/count`的请求调用`count()`，其他的都调用`echo()`。以`/`结尾的模式会匹配任何以该模式为前缀的URL。在底层，服务器会对每个请求在一个单独的goroutine中运行处理器函数，以便可以同时处理多个请求。然而，如果两个并发请求同时试图更新`count`，结果可能会不正确，这称为**竞争条件**(race condition)（见9.1节）。为了避免这个问题，必须确保一次最多只有一个goroutine访问该变量，这就是使用`mu.Lock()`和`mu.Unlock()`调用将访问`count`的代码括起来的目的。第9章将详细介绍使用共享变量的并发。

```shell
$ ./fetch http://localhost:8000     
URL.Path = "/"

$ ./fetch http://localhost:8000/help
URL.Path = "/help"

$ ./fetch http://localhost:8000/count
Count 2
```

下面是一个更复杂的例子，处理器函数会返回请求的标头(header)和表单(form)数据，可用于检查和调试请求：

[gopl.io/ch1/server3](https://github.com/ZZy979/gopl.io/blob/main/ch1/server3/main.go)

请求 <http://localhost:8000/?q=query&page=2> 会产生如下输出：

```
GET /?q=query&page=2 HTTP/1.1
Header["Accept-Encoding"] = ["gzip, deflate, br, zstd"]
Header["Accept-Language"] = ["zh-CN,zh;q=0.9"]
Header["Connection"] = ["keep-alive"]
Header["Accept"] = ["text/html,application/xhtml+xml,application/xml...]
Header["User-Agent"] = ["Mozilla/5.0 (Windows NT 10.0; Win64; x64)..."]
Host = "localhost:8000"
RemoteAddr = "127.0.0.1:12418"
Form["q"] = ["query"]
Form["page"] = ["2"]
```

注意，`ParseForm()`调用嵌套在了`if`语句中。Go允许在`if`条件之前添加一个简单语句（例如局部变量声明），这对于错误处理特别有用。可以将其写成

```go
err := r.ParseForm()
if err != nil {
	log.Print(err)
}
```

但是组合语句更紧凑，并且缩小了变量`err`的作用域，这是一种好的做法。2.7节将介绍作用域。

在这些程序中，有三种不同的类型用作输出流。`fetch`程序将HTTP响应拷贝到`os.Stout`（一个文件），就像`lissajous`程序一样。`fetchall`程序通过将响应拷贝到`io.Discard`将其丢弃。上面的Web服务器则使用`fmt.Fprintf()`将响应写入`http.ResponseWriter`。

尽管这三种类型在实现细节上有所不同，但它们都满足一个共同的**接口**(interface)，使其可用于需要输出流的任何地方。该接口名为`io.Writer`，将在7.1节中讨论。

利用接口机制，可以很容易地将Web服务器与`lissajous()`函数结合起来，这样GIF动画会被写入HTTP客户端而不是标准输出。只需修改`lissajous`程序的`main()`函数：

```go
func main() {
	if len(os.Args) > 1 && os.Args[1] == "web" {
		http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
			lissajous(w)
		})
		log.Fatal(http.ListenAndServe("localhost:8000", nil))
		return
	}
	lissajous(os.Stdout)
}
```

`HandleFunc()`的第二个参数是**函数字面值**(function literal)，也就是在使用时定义的匿名函数。将在5.6节中介绍。

做完这个修改之后，使用以下命令启动服务器：

```shell
$ go run gopl.io/ch1/lissajous web
```

之后在浏览器中访问 <http://localhost:8000> 。每次加载页面时都会看到一个新的动画，如下图所示。

![浏览器中的李萨如图形动画](/assets/images/gopl-note-ch01-tutorial/浏览器中的李萨如图形动画.png)

练习1.12 修改`lissajous`服务器，从URL读取参数值。例如，URL <http://localhost:8000/?cycles=20> 将周期数设置为20而不是默认的5。使用`strconv.Atoi()`函数将字符串转换为整数。

## 1.8 未尽事项
Go语言还有很多方面在本章中没有覆盖到。以下是一些仅仅提及或完全忽略的话题。

**控制流**：本章介绍了两种基本的控制流语句：`if`和`for`，但没有介绍`switch`语句。`switch`是一种多路分支语句。下面是一个简单的例子：

```go
switch coinflip {
case "heads":
	heads++
case "tails":
	tails++
default:
	fmt.Println("landed on edge!")
}
```

该语句会将`coinflip`与每个case的值依次进行比较，并执行第一个匹配的case。如果所有case都不匹配，则执行（可选的）默认case。Go的`switch`语句不会像C语言那样自动执行(fall through)下一个case，因此不需要在每个case后添加`break`（但有一个很少使用的`fallthrough`语句可以覆盖这种行为）。

`switch`可以没有操作数，只列出case，其中每个都是布尔表达式：

```go
func Sgn(x int) int {
	switch {
	case x > 0:
		return 1
	case x < 0:
		return -1
	default:
		return 0
	}
}
```

这种形式叫做**无标签**(tagless) `switch`，等价于`switch true`。

与`for`和`if`语句一样，`switch`语句也可以包含一个可选的简单语句（短变量声明、赋值语句或函数调用），用于在测试之前赋值。

`break`和`continue`语句会改变控制流。`break`语句会使控制流转移到最内层的`for`、`switch`或`select`语句之后的下一条语句。`continue`语句会使最内层的`for`循环开始下一次迭代。语句可以被标记，以便在`break`和`continue`语句中引用，例如跳出多层嵌套循环，或者开始最外层循环的下一次迭代。还有`goto`语句，但它是用于机器生成的代码，而不是给程序员使用的。

**命名类型**：`type`声明允许为现有类型命名。例如：

```go
type Point struct {
	X, Y int
}
var p Point
```

类型声明和命名类型在第2章中介绍。

**指针**：Go语言提供指针，即包含变量地址的值。在有些语言（如C语言）中，指针几乎不受限制；在另外一些语言（如Java）中，指针被伪装成“引用”，除了传递之外几乎无法进行其他操作。Go介于两者之间。指针是显式可见的，`&`运算符返回变量的地址，`*`运算符获取指针指向变量的值，但没有指针运算。2.3.2节将介绍指针。

**方法和接口**：方法是与命名类型关联的函数。Go的特殊之处在于，方法可以附加到几乎任何命名类型。方法在第6章中介绍。接口是抽象类型，能够以相同的方式处理不同的具体类型——基于其拥有的方法，而不是表示或实现方式。接口在第7章中介绍。

**包**：Go自带的标准库包含许多有用的包，Go社区创建并分享了更多的包。编程往往更多地是使用现有的包，而不是编写原创代码。本书中将介绍一些最重要的标准包。

在开始编写任何新程序之前，最好先查看是否有可以帮你更轻松地完成工作的包。可以在 <https://pkg.go.dev/std> 找到标准库包索引，在 <https://pkg.go.dev/> 找到社区贡献的包。`go doc`工具可以从命令行访问这些文档：

```shell
$ go doc http.ListenAndServe
package http // import "net/http"

func ListenAndServe(addr string, handler Handler) error
    ListenAndServe listens on the TCP network address addr and then calls
    Serve with handler to handle requests on incoming connections...
```

**注释**：我们已经提到过在程序或包的开头添加文档注释。在每个函数声明之前编写描述其行为的注释也是一种良好的编程风格。这些约定很重要，因为像`godoc`这样的工具会使用它们来定位和显示文档（见10.7.4节）。

对于跨越多行或者出现在表达式或语句内的注释，还可以使用`/* ... */`注释。这种注释有时用于文件开头的大段解释性文本，以避免每行都添加`//`。

在注释内部，`//`和`/*`没有特殊含义，因此注释不能嵌套。
