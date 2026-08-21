---
title: 《Go程序设计语言》笔记 第3章 基本数据类型
date: 2026-06-24 21:39:55 +0800
categories: [Go, GOPL]
tags: [go, data type, operator, integer division, overflow, complex number, string, raw string, unicode, constant, enumeration]
math: true
---
Go的类型分为四类：**基本类型**(basic type)、**聚合类型**(aggregate type)、**引用类型**(reference type)和**接口类型**(interface type)。本章的主题是基本类型，包括数字、字符串和布尔值。聚合类型——数组（4.1节）和结构体（4.4节）——将简单类型组合成更复杂的数据类型。引用类型包括指针（2.3.2节）、切片（4.2节）、映射（4.3节）、函数（第5章）和channel（第8章），共同点是它们都**间接**引用程序变量或状态，因此对一个引用的修改会被该引用的所有副本观察到。最后，将在第7章讨论接口类型。

## 3.1 整数
Go提供了有符号和无符号整数。有4种不同大小的有符号整数——8、16、32、64位，分别由类型`int8`、`int16`、`int32`和`int64`表示。对应的无符号类型是`uint8`、`uint16`、`uint32`和`uint64`。

还有两种类型`int`和`uint`，表示特定平台上最高效大小的有符号和无符号整数；`int`是使用最广泛的数值类型。这两种类型具有相同的大小，32或64位，但不能对此做任何假设。即使在相同的硬件上，不同的编译器也可能做出不同的选择。

即使`int`的大小是32位，`int`和`int32`、`uint`和`uint32`也不是相同的类型。在需要`int32`的地方使用`int`值需要显式类型转换，反之亦然。

类型`rune`是`int32`的同义词，通常用于表示一个Unicode码点，这两个名字可以互换使用。类型`byte`是`uint8`的同义词，用于强调该值是一个原始字节数据，而不是一个小整数。（注：这两对类型之间不需要显式类型转换）

最后，还有一个无符号整数类型`uintptr`，其宽度未指定，但足以容纳指针值。`uintptr`类型仅用于底层编程，例如Go程序与C语言库或操作系统的边界。在第13章处理`unsafe`包时将会看到这样的例子。

有符号整数采用**2的补码**形式表示，其中最高位是符号位（正数为0、负数为1），n位有符号数的范围为-2<sup>n-1</sup>到2<sup>n-1</sup>-1。无符号整数使用全部二进制位表示非负值，因此范围为0到2<sup>n</sup>−1。例如，`int8`的范围是-128到127，而`uint8`的范围则是0到255。

Go的算术、逻辑和比较二元运算符按优先级降序排列如下（一元运算符的优先级最高，因此未列出）：

```
*   /   %   <<  >>  &   &^
+   -   |   ^
==  !=  <   <=  >   >=
&&
||
```

二元运算符只有五种优先级。同一级别的运算符从左到右结合。可以使用括号来明确或改变求值顺序，例如`mask & (1 << 28)`。

上表中前两行的每个运算符（如`+`）都有一个相应的**赋值运算符**(assignment operator)（如`+=`），可用于简化赋值语句。
 
算术运算符`+`、`-`、`*`和`/`可用于整数、浮点数和复数，但余数运算符`%`（取模）仅用于整数。负数取模的行为因编程语言而异（详见[《【Python】除法舍入问题》]({% post_url 2021-03-20-python-division-rounding %})）。在Go中，余数的符号总是与被除数相同，因此`-5%3`和`-5%-3`都是-2。`/`的行为取决于其操作数是否为整数，因此`5.0/4.0`为1.25，但`5/4`为1、`-5/4`为-1，因为整数除法会将结果**向零**截断。

如果算术运算的结果超过了结果类型的范围，则称为**溢出**(overflow)。超出的高位部分将被丢弃。对于有符号类型，如果最左边的位是1，结果就是负数。例如：

```go
var u uint8 = 255
fmt.Println(u, u+1, u*u) // "255 0 1"

var i int8 = 127
fmt.Println(i, i+1, i*i) // "127 -128 1"
```

两个相同类型的整数可以使用下面的比较运算符进行比较，比较表达式的类型的`bool`。

```
==      equal to
!=      not equal to
<       less than
<=      less than or equal to
>       greater than
>=      greater than or equal to
```

事实上，所有基本类型（布尔值、数字和字符串）都是**可比较的**(comparable)，也就是说两个相同类型的值可以用`==`和`!=`进行比较。此外，只有整数、浮点数和字符串是**有序的**(ordered)，可以用`<`等进行比较。许多其他类型都是不可比较的，其他任何类型都不是有序的。当遇到每种类型时，我们将介绍其可比较性规则。

下面是一元加法和减法运算符：

```
+       unary positive (no effect)
-       unary negation
```

Go还提供了以下按位运算符，其中前4个不区分有符号性：

```
&       bitwise AND
|       bitwise OR
^       binary: bitwise XOR; unary: bitwise negation (NOT)
&^      bit clear (AND NOT)
<<      left shift
>>      right shift
```

`x &^ y`等价于`x & (^y)`，即：如果`y`中的位为1，则结果对应位为0，否则为`x`的对应位。

下面的代码演示了如何使用位操作将`uint8`值解释为8个独立位的集合。动词`%b`以二进制格式打印数字，`08`表示将结果用0填充到8个字符宽度。

```go
var x uint8 = 1<<1 | 1<<5
var y uint8 = 1<<1 | 1<<2

fmt.Printf("%08b\n", x)    // "00100010", the set {1, 5}
fmt.Printf("%08b\n", y)    // "00000110", the set {1, 2}

fmt.Printf("%08b\n", x&y)  // "00000010", the intersection {1}
fmt.Printf("%08b\n", x|y)  // "00100110", the union {1, 2, 5}
fmt.Printf("%08b\n", x^y)  // "00100100", the symmetric difference {2, 5}
fmt.Printf("%08b\n", x&^y) // "00100000", the difference {5}

for i := uint(0); i < 8; i++ {
	if x&(1<<i) != 0 { // membership test
		fmt.Println(i) // "1", "5"
	}
}

fmt.Printf("%08b\n", x<<1) // "01000100", the set {2, 6}
fmt.Printf("%08b\n", x>>1) // "00010001", the set {0, 4}
```

（另见6.5节实现的位向量）

在移位操作`x<<n`和`x>>n`中，`n`表示要移位的位数，必须是无符号的；`x`可以是无符号的或有符号的。从算术上讲，左移n位等价于乘以2<sup>n</sup>，右移n位等价于除以2<sup>n</sup>并下取整。

左移和无符号数的右移会用零填充空位，而有符号数的右移会用符号位填充空位。因此，在将整数当作位模式处理时，应该使用无符号算术。

虽然Go提供了无符号整数，但即使是对于不可能为负的量（比如数组长度），我们还是倾向于使用有符号的`int`，尽管`uint`似乎是更合理的选择。其实，内置函数`len()`返回有符号的`int`，可以像这样逆序遍历切片：

```go
medals := []string{"gold", "silver", "bronze"}
for i := len(medals) - 1; i >= 0; i-- {
	fmt.Println(medals[i]) // "bronze", "silver", "gold"
}
```

而另一种选择将是灾难性的。如果`len()`返回一个`uint`，则条件`i >= 0`将永远为真。当`i == 0`时，语句`i--`不会导致`i`变成-1，而是`uint`的最大值（例如2<sup>64</sup>-1），然后访问`medals[i]`将导致下标越界(panic)。

因此，无符号数往往只在需要位运算或特殊算术运算时才使用，例如实现位集(bitset)、解析二进制文件，或者哈希和加密等。它们通常不用于仅仅表达非负的量。

一般来说，需要显式地将值从一种类型转换为另一种类型，算术和逻辑二元运算符（移位除外）必须具有相同类型的操作数。虽然这偶尔会导致很长的表达式，但它消除了隐式类型转换带来的问题，使程序更容易理解。

考虑下面的例子：

```go
var apples int32 = 1
var oranges int16 = 2
var compote int = apples + oranges // compile error
```

这段代码会产生编译错误：

```
invalid operation: apples + oranges (mismatched types int32 and int16)
```

最直接的修复方法是将二者转换为公共类型：

```go
var compote = int(apples) + int(oranges)
```

如果不超过取值范围，整数之间转换不会改变值，只是告诉编译器按什么类型解释一个值。但是，将大整数缩窄为小整数，或者整数和浮点数之间的转换可能会改变值或丢失精度：

```go
f := 3.141 // a float64
i := int(f)
fmt.Println(f, i)   // "3.141 3"
f = 1.99
fmt.Println(int(f)) // "1"
```

浮点数到整数的转换会丢弃小数部分，向零截断。应该避免操作数超出目标类型范围的转换，因为行为依赖于具体实现：

```go
f := 1e100  // a float64
i := int(f) // result is implementation-dependent
```

任何大小和类型的整数字面值都可以写成十进制数、以`0`开头的八进制数（如`0666`），或者以`0x`或`0x`开头的十六进制数（如`0xdeadbeef`）。十六进制数可以是大写或小写。八进制数通常只用于一个目的——POSIX系统的文件权限；十六进制数通常用于强调数字的位模式，而不是其数值。

使用`fmt`包打印数字时，可以用动词`%d`、`%o`和`%x`控制基数和格式，如下所示：

```go
o := 0666
fmt.Printf("%d %[1]o %#[1]o\n", o) // "438 666 0666"
x := int64(0xdeadbeef)
fmt.Printf("%d %[1]x %#[1]x %#[1]X\n", x) // "3735928559 deadbeef 0xdeadbeef 0XDEADBEEF"
```

注意两个`fmt`技巧的使用。通常，包含多个`%`动词的格式字符串需要相同数量的额外操作数，但`%`后的“副词”(adverb)`[1]`告诉`Printf()`仍然使用第一个操作数。另外，`%o`、`%x`或`%X`的副词`#`告诉`Printf()`分别输出`0`、`0x`或`0X`前缀。

字符字面值用单引号将字符括起来，例如`'a'`。也可以直接或通过数字转义来表示Unicode字符。

字符用`%c`打印，如果需要单引号则用`%q`：

```go
ascii := 'a'
unicode := '国'
newline := '\n'
fmt.Printf("%d %[1]c %[1]q\n", ascii)   // "97 a 'a'"
fmt.Printf("%d %[1]c %[1]q\n", unicode) // "22269 国 '国'"
fmt.Printf("%d %[1]q\n", newline)       // "10 '\n'"
```

## 3.2 浮点数
Go提供了两种精度的浮点数，`float32`和`float64`。其算术属性由[IEEE 754](https://ieeexplore.ieee.org/document/8766229)标准定义，所有现代CPU都实现了该标准。

浮点数的取值限制可以在`math`包中找到。最大值`math.MaxFloat32`约为3.4e38，`math.MaxFloat64`约为1.8e308。最小正值`math.SmallestNonzeroFloat32`约为1.4e-45，`math.SmallestNonzeroFloat64`约为4.9e-324。

`float32`提供大约6位数字的精度，而`float64`提供大约15位数字的精度。在大多数情况下`float64`应该是首选，因为`float32`的计算会迅速累积误差，并且`float32`能精确表示的最大正整数仅为2<sup>24</sup>-1：

```go
var f float32 = 16777216 // 1 << 24
fmt.Println(f == f+1)    // "true"!
```

浮点数字面值用小数表示，例如：

```go
const e = 2.71828 // (approximately)
```

小数点之前或之后的数字可以省略（例如`.707`或`1.`）。很小或很大的数字最好用科学记数法表示：

```go
const Avogadro = 6.02214129e23
const Planck = 6.62606957e-34
```

浮点值可以用`Printf()`的`%g`动词打印，将选择具有足够精度的最紧凑表示。但对于表格数据，`%e`（科学记数法）或`%f`（固定位数）形式可能更合适。这三个动词都可以控制宽度和精度。

```go
for x := 0; x < 8; x++ {
	fmt.Printf("x = %d    eˣ = %8.3f\n", x, math.Exp(float64(x)))
}
```

上面的代码打印e的幂，3位小数精度、8字符宽：

```
x = 0    eˣ =    1.000
x = 1    eˣ =    2.718
x = 2    eˣ =    7.389
x = 3    eˣ =   20.086
x = 4    eˣ =   54.598
x = 5    eˣ =  148.413
x = 6    eˣ =  403.429
x = 7    eˣ = 1096.633
```

`math`包中除了大量常用的数学函数外，还提供了创建和检测特殊浮点值的函数：
* `Inf()`和`IsInf()`：正负无穷大，表示溢出或除以零的结果。
* `NaN()`和`IsNaN()`： "not a number" ，表示无效的数学运算结果（如`0/0`或`Sqrt(-1)`）。

```go
var z float64
fmt.Println(z, -z, 1/z, -1/z, z/z) // "0 -0 +Inf -Inf NaN"
```

必须用`math.IsNaN()`来测试一个值是否是NaN，因为任何与NaN的比较都返回`false`（除了`!=`）：

```go
nan := math.NaN()
fmt.Println(nan == nan, nan < nan, nan > nan) // "false false false"
```

如果一个返回浮点数结果的函数可能失败，最好用单独的返回值报告失败：

```go
func compute() (value float64, ok bool) {
	// ...
	if failed {
		return 0, false
	}
	return result, true
}
```

下面的程序演示了浮点图形计算。它使用可缩放矢量图形(Scalable Vector Graphics, SVG)（一种用于线条绘制的XML表示法）将二元函数`z = f(x, y)`的图像绘制为线网三维曲面。下图展示了函数`z = sin(r)/r`的输出图形，其中`r = sqrt(x*x+y*y)`。

![函数sin(r)/r的图像](/assets/images/gopl-note-ch03-basic-data-types/surface.svg)

[gopl.io/ch3/surface](https://github.com/ZZy979/gopl.io/blob/main/ch3/surface/main.go)

程序的本质是三个不同坐标系之间的映射，如下图所示。

![三个不同的坐标系](/assets/images/gopl-note-ch03-basic-data-types/三个不同的坐标系.png)

第一个是100×100的二维网格（即曲面网格在底面的投影），由整数坐标(i, j)表示。我们从后到前绘制，因为远处的多边形可能被近处的遮挡。

第二个是三维坐标(x, y, z)，其中x和y是i和j的线性函数：原点平移到中心，缩放系数为`xyrange`。高度z是函数值f(x, y)。

第三个坐标系是二维图像画布（即SVG图像），由坐标(sx, sy)表示，原点在左上角。我们使用正等测投影(isometric projection)将每个三维点(x, y, z)映射到二维画布上。z的缩放系数0.4是一个任意选择的参数。

注：函数`corner(i, j)`首先计算坐标系1中的网格顶点(i, j)在坐标系2的xOy平面中的坐标(x, y)以及对应的函数值z = f(x, y)。然后将三维坐标(x, y, z)映射到二维SVG画布坐标(sx, sy)，可分解为以下三步：

①缩放

$$
\begin{bmatrix}
x_1 \newline
y_1 \newline
z_1
\end{bmatrix}
=
\begin{bmatrix}
s_{xy} & 0 & 0 \newline
0 & s_{xy} & 0 \newline
0 & 0 & s_z
\end{bmatrix}
\begin{bmatrix}
x \newline
y \newline
z
\end{bmatrix}
=
\begin{bmatrix}
s_{xy} x \newline
s_{xy} y \newline
s_z z
\end{bmatrix}
$$

②正等测投影

$$
\begin{bmatrix}
x_2 \newline
y_2 \newline
\end{bmatrix}
=
\begin{bmatrix}
\cos 30° & -\cos 30° & 0 \newline
\sin 30° & \sin 30° & -1
\end{bmatrix}
\begin{bmatrix}
x_1 \newline
y_1 \newline
z_1
\end{bmatrix}
=
\begin{bmatrix}
\frac{\sqrt 3}{2}(x_1 - y_1) \newline
\frac{1}{2}(x_1 + y_1) - z_1
\end{bmatrix}
$$

③平移

$$
\begin{bmatrix}
sx \newline
sy \newline
\end{bmatrix}
=
\begin{bmatrix}
1 & 0 & \frac{w}{2} \newline
0 & 1 & \frac{h}{2}
\end{bmatrix}
\begin{bmatrix}
x_2 \newline
y_2 \newline
1
\end{bmatrix}
=
\begin{bmatrix}
x_2 + \frac{w}{2} \newline
y_2 + \frac{h}{2}
\end{bmatrix}
$$

对于二维网格中的每个单元格，`main()`函数计算多边形ABCD的四个顶点在图像画布上的坐标，其中B对应(i, j)，然后打印一条SVG指令`<polygon>`来绘制它（注：这意味着输出的图形不包含任何曲线，而是由直线段构成的）。

练习3.1 如果函数`f`返回的`float64`值是无穷大，SVG文件将包含无效的`<polygon>`元素（尽管许多SVG渲染器能够处理这类问题）。修改程序跳过无效的多边形。

练习3.2 尝试`math`包中其他函数的可视化。例如[鸡蛋盒](https://mathcurve.com/surfaces.gb/boiteaoeufs/boiteaoeufs.shtml)(egg box) ($z = \sin x + \sin y$)、雪丘(moguls) ($z = \sin x\sin y$)、[马鞍面](https://mathcurve.com/surfaces.gb/paraboloidhyperbolic/paraboloidhyperbolic.shtml)(saddle) ($z = x^2 - y^2$)等。

练习3.3 根据高度给多边形着色，使峰顶为红色(`#ff0000`)，谷底为蓝色(`#0000ff`)。

练习3.4 按照1.7节中Lissajous示例的方法，构建一个web服务器，计算曲面并返回SVG数据给客户端。服务器必须设置`Content-Type`标头：

```go
w.Header().Set("Content-Type", "image/svg+xml")
```

（在Lissajous示例中不需要此步骤，因为服务器会根据响应的前512个字节识别PNG等常见格式，并生成正确的标头。）允许客户端通过HTTP请求参数指定高度、宽度和颜色等。

## 3.3 复数
Go提供了两种精度的复数类型：`complex64`和`complex128`，分别对应`float32`和`float64`。内置函数`complex()`用于创建复数，`real()`和`imag()`分别返回复数的实部和虚部：

```go
var x complex128 = complex(1, 2) // 1+2i
var y complex128 = complex(3, 4) // 3+4i
fmt.Println(x*y)                 // "(-5+10i)"
fmt.Println(real(x*y))           // "-5"
fmt.Println(imag(x*y))           // "10"
```

如果浮点数字面值或十进制整数字面值后面跟着`i`（例如`3.141592i`或`2i`），就是**虚数字面值**(imaginary literal)，表示一个实部为0的复数：

```go
fmt.Println(1i * 1i) // "(-1+0i)"
```

复数常量可以与其他常量（整数、浮点数或复数）相加，因此可以用自然的方式来写复数，例如`x := 1 + 2i`。

复数可以用`==`和`!=`进行相等比较。如果两个复数的实部和虚部分别相等，则这两个复数相等（注：浮点数相等比较需要小心精度问题）。

`math/cmplx`包提供了处理复数的数学函数，例如：

```go
fmt.Println(cmplx.Sqrt(-1)) // "(0+1i)"
```

下面的程序使用复数算术生成了一个曼德勃罗集。

注：曼德勃罗集的定义详见[《Java核心技术》笔记 卷II 第11章]({% post_url 2025-12-09-java-note-v2ch11-advanced-swing-and-graphics %}) 11.4.2.1节。

[gopl.io/ch3/mandelbrot](https://github.com/ZZy979/gopl.io/blob/main/ch3/mandelbrot/main.go)

两层嵌套循环遍历1024×1024灰度栅格图像中的每个点，该图像表示复平面的-2到+2部分。该程序测试对于每个点`z`，反复计算`v = v*v + z`（曼德勃罗集定义中的迭代公式）是否最终会“逃逸”出半径为2的圆（“发散”的近似判断）。如果是，则根据逃逸所需的迭代次数对该点进行着色（颜色越浅表示发散得越快）。如果不是，则该点属于曼德勃罗集，用黑色表示。最后，程序将PNG图像写到标准输出，如下图所示。

![曼德勃罗集](/assets/images/gopl-note-ch03-basic-data-types/mandelbrot.png)

练习3.5 实现彩色曼德勃罗集，使用函数`image.NewRGBA()`创建图像，使用类型`color.RGBA`或`color.YCbCr`表示颜色。

练习3.6 超级采样(supersampling)是一种抗锯齿技术，计算每个像素内几个点的颜色值并取平均值。最简单的方法是将每个像素划分为四个“子像素”。实现这种方法。

练习3.7 另一个简单的分形使用牛顿法来寻找方程的复解（例如z<sup>4</sup>-1=0）。根据从每个点找到四个根之一所需的迭代次数确定其灰度，根据每个点到达的根确定其颜色。

练习3.8 渲染高倍放大的分形需要很高的算术精度。使用四种不同的数字表示来实现相同的分形：`complex64`、`complex128`、`big.Float`和`big.Rat`（后两种类型可以在`math/big`包中找到）。它们在性能和内存使用方面相比如何？放大多少倍开始出现模糊？

练习3.9 编写一个渲染分形的web服务器，并将图像数据写入客户端。允许客户端通过HTTP请求参数指定x、y和缩放值。

## 3.4 布尔值
布尔(boolean)类型`bool`只有两个可能的值：`true`和`false`。`if`和`for`语句中的条件是布尔值，`==`和`<`等比较运算符会产生布尔值结果。一元运算符`!`是逻辑否定，`!true`是`false`。

布尔值可以用`&&` (AND)和`||` (OR)运算符组合，这两个运算符具有**短路**(short-circuit)行为：如果结果可以由左操作数的值确定，则右操作数不会被求值，因此下面的写法是安全的：

```go
s != "" && s[0] == 'x'
```

对空字符串应用`s[0]`会导致panic。

`&&`的优先级比`||`高，因此下面的条件不需要加括号：

```go
if 'a' <= c && c <= 'z' ||
	'A' <= c && c <= 'Z' ||
	'0' <= c && c <= '9' {
	// ...ASCII letter or digit...
}
```

不存在从布尔值到数值0或1的隐式转换，反之亦然。必须使用显式的`if`：

```go
i := 0
if b {
	i = 1
}
```

如果经常需要这个操作，可以写一个转换函数：

```go
// btoi returns 1 if b is true and 0 if false.
func btoi(b bool) int {
	if b {
		return 1
	}
	return 0
}
```

逆操作非常简单，不过为了对称这里也将其写成函数：

```go
// itob reports whether i is non-zero.
func itob(i int) bool { return i != 0 }
```

## 3.5 字符串
字符串是不可变的字节序列。字符串可以包含任意字节数据，但通常包含人类可读的文本。文本字符串通常被解释为UTF-8编码的Unicode码点(rune)序列，稍后会详细探讨。

内置函数`len()`返回字符串中的字节数（不是字符数）。索引操作`s[i]`获取字符串`s`的第`i`个字节，其中0 ≤ `i` < `len(s)`。

```go
s := "hello, world"
fmt.Println(len(s))     // "12"
fmt.Println(s[0], s[7]) // "104 119" ('h' and 'w')
```

尝试访问超出这个范围的索引会导致panic：

```go
c := s[len(s)] // panic: index out of range
```

字符串的第i个字节不一定是第i个字符，因为非ASCII字符的UTF-8编码需要两个或更多字节。稍后将讨论如何处理字符。

子串操作`s[i:j]`产生一个新的字符串，由原始字符串从索引`i`到`j`（不包括）的字节组成，结果包含`j-i`个字节。

```go
fmt.Println(s[0:5]) // "hello"
```

同样，如果任一索引越界或者`j`小于`i`会导致panic。

起始和终止都可以省略，默认值分别为0和`len(s)`：

```go
fmt.Println(s[:5]) // "hello"
fmt.Println(s[7:]) // "world"
fmt.Println(s[:])  // "hello, world"
```

运算符`+`将两个字符串拼接为一个新字符串：

```go
fmt.Println("goodbye" + s[5:]) // "goodbye, world"
```

字符串可以用比较运算符（如`==`和`<`）进行比较。比较是逐字节进行的，因此结果是自然词典序。

字符串值是**不可变的**：字符串**值**包含的字节序列永远不能更改，不过可以给字符串**变量**赋一个新值。例如，可以像这样将一个字符串追加到另一个字符串：

```go
s := "left foot"
t := s
s += ", right foot"
```

这样`s`持有拼接后的新字符串，而`t`仍然包含旧字符串。

```go
fmt.Println(s) // "left foot, right foot"
fmt.Println(t) // "left foot"
```

由于字符串是不可变的，尝试修改字符串数据的操作是被禁止的：

```go
s[0] = 'L' // compile error: cannot assign to s[0]
```

不可变性意味着一个字符串的两个副本、或者一个字符串及其子串可以安全地共享相同的底层内存。这使得字符串的拷贝和子串操作都具有常数时间复杂度，不会分配新的内存。下图展示了一个字符串及其两个子字符串如何共享相同的底层数组。

![字符串和子串](/assets/images/gopl-note-ch03-basic-data-types/字符串和子串.png)

注：Go的`string`类型类似于C++的`std::string_view`。

### 3.5.1 字符串字面值
字符串可以写成**字符串字面值**(string literal)，用双引号括起来：

```go
"Hello, 世界"
```

因为Go源文件总是用UTF-8编码，并且Go字符串通常也解释为UTF-8，因此可以在字符串字面值中包含Unicode字符。

在双引号字符串字面值中，以反斜杠`\`开头的**转义序列**(escape sequence)可用于表示任意字节。以下转义表示ASCII控制字符：

```
\a    响铃(alert)
\b    退格(backspace)
\f    换页(form feed)
\n    换行(newline)
\r    回车(carriage return)
\t    制表符(tab)
\v    垂直制表符(vertical tab)
\'    单引号（仅用于rune字面值'\''）
\"    双引号（仅用于双引号字符串字面值）
\\    反斜杠
```

也可以使用十六进制或八进制转义表示任意字节。十六进制转义写作`\xhh`，包含2个十六进制数字`h`（大小写均可）。八进制转义写作`\ooo`，包含3个八进制数字`o`，不超过`\377`。

**原始字符串字面值**(raw string literal)写作`` `...` ``，使用反引号而不是双引号。在原始字符串字面值中不处理转义序列，内容按原样保留（包括反斜杠和换行符），因此可以跨越多行。唯一的处理是会删除回车符(`\r`)，使得字符串的值在所有平台上都是相同的。

原始字符串字面值用于编写正则表达式会很方便，因为往往有很多反斜杠。对于HTML模板、JSON文本、命令用法等需要延伸到多行的场景也很有用。

```go
const GoUsage = `Go is a tool for managing Go source code.

Usage:
    go command [arguments]
...`
```

### 3.5.2 Unicode
很久以前，只有一种字符集：美国信息交换标准码(American Standard Code for Information Interchange, ASCII)。ASCII使用7位表示128个字符：大小写英文字母、数字、标点符号和设备控制符。对于早期的计算机来说这就足够了，但是这导致世界上很大一部分人无法在计算机中使用自己的语言。随着互联网的发展，各种语言的数据变得很常见。如何有效地处理这种多样性？

答案是Unicode ([unicode.org](https://home.unicode.org/))。它收集了世界上所有书写系统中的所有字符，并为每个字符分配一个标准编号，称为Unicode**码点**(code point)，用Go的术语叫做**rune**。

Unicode第8版为100多种语言的超过120000个字符定义了码点。在Go中，用于保存单个Unicode码点的数据类型是`int32`，其同义词`rune`专门用于这个目的。

我们可以将每个码点表示为一个`int32`值，这称为UTF-32或UCS-4，每个码点的编码都是32位。这种方式简单且统一，但会浪费很多存储空间，因为大多数计算机可读文本都是ASCII，每个字符只需要8位或1字节。而所有常用的字符也远少于65536个，用16位就能表示。更好的编码方式是UTF-8。

注：Unicode码点和码元的概念另见[《Java核心技术》笔记 卷I 第3章]({% post_url 2024-08-03-java-note-v1ch03-fundamental-programming-structures-in-java %}) 3.3.4和3.6.6节。

### 3.5.3 UTF-8
UTF-8是一种将Unicode码点编码为字节序列的变长编码。UTF-8使用1到4个字节来表示每个rune，ASCII字符只使用1个字节，大多数常用字符只使用2或3个字节。编码第一个字节的高比特位指示总共有几个字节（如下表所示）。例如，最高位`0`表示只有1个字节，等价于ASCII。最高位`110`表示编码占2个字节，第二个字节以`10`开头。

| rune | UTF-8编码 |
| --- | --- |
| 0−127 (ASCII) | `0xxxxxxx` |
| 128−2047 | `110xxxxx 10xxxxxx` |
| 2048−65535 | `1110xxxx 10xxxxxx 10xxxxxx` |
| ≥65536 | `11110xxx 10xxxxxx 10xxxxxx 10xxxxxx` |

注：UTF-8编码的详细规则和示例参见[《Java核心技术》笔记 卷II 第2章]({% post_url 2025-04-07-java-note-v2ch02-input-and-output %}) 2.1.8节。

变长编码使得无法直接通过索引访问字符，但UTF-8有许多其他优点：
* 编码紧凑，兼容ASCII，并且具有自同步性（至多回溯3个字节就能找到字符的开始位置）。
* 是前缀码（没有任何字符的编码是其他字符编码的前缀）。因此可以从左到右解码，不会有任何歧义，也不需要向前查看。
* 没有任何字符的编码是其他字符编码（或编码序列）的子串。因此可以直接通过搜索字节来搜索字符，而无需担心上下文。
* 编码的字典序与Unicode码点的顺序一致，因此可以直接对UTF-8编码进行排序。
* 没有嵌入式0字节(NUL)，这对于使用0作为字符串结尾的编程语言来说很方便。

Go源文件始终采用UTF-8编码，UTF-8也是Go程序操作文本字符串的首选编码。`unicode`包提供了处理单个rune的函数（例如区分字母和数字、大小写转换等），`unicode/utf8`包提供了UTF-8编码和解码函数。

许多Unicode字符无法用键盘输入，或者很难在视觉上与外观相似的字符区分开（例如分号 ";" (U+003B)和希腊问号 ";" (U+037E)、中文的“龍”(U+9F8D)和日文的“龍”(U+F9C4)），有些甚至是不可见的。Go字符串字面值中的**Unicode转义**允许通过码点来指定Unicode字符。有两种形式：`\uhhhh`表示16位值，`\Uhhhhhhhh`表示32位值，其中每个`h`都是十六进制数字。Unicode转义表示指定码点的UTF-8编码。因此，以下字符串字面值都表示相同的6字节字符串：

```go
"世界"
"\xe4\xb8\x96\xe7\x95\x8c"
"\u4e16\u754c"
"\U00004e16\U0000754c"
```

Unicode转义也可以用在rune字面值中。下面三个字符是等价的：

```go
'世'  '\u4e16'  '\U00004e16'
```

值小于256的rune也可以用单个十六进制转义表示，例如`'\x41'`等价于`'A'`。但是对于更大的值，必须使用`\u`或`\U`转义，因此`'\xe4\xb8\x96'`不是合法的rune字面值。

得益于UTF-8的良好特性，许多字符串操作都不需要解码。可以像这样测试一个字符串是否是另一个字符串的前缀、后缀或者子串：

```go
func HasPrefix(s, prefix string) bool {
	return len(s) >= len(prefix) && s[:len(prefix)] == prefix
}

func HasSuffix(s, suffix string) bool {
	return len(s) >= len(suffix) && s[len(s)-len(suffix):] == suffix
}

func Contains(s, substr string) bool {
	for i := 0; i < len(s); i++ {
		if HasPrefix(s[i:], substr) {
			return true
		}
	}
	return false
}
```

如果确实需要处理单个Unicode字符，就必须使用其他机制。考虑字符串`"Hello, 世界"`，下图展示了它在内存中的表示。

![遍历字符串](/assets/images/gopl-note-ch03-basic-data-types/遍历字符串.png)

该字符串包含9个字符，用UTF-8编码为13个字节：

```go
import "unicode/utf8"

s := "Hello, 世界"
fmt.Println(len(s))                    // "13"
fmt.Println(utf8.RuneCountInString(s)) // "9"
```

为了处理字符，需要使用`unicode/utf8`包提供的UTF-8解码器：

```go
for i := 0; i < len(s); {
	r, size := utf8.DecodeRuneInString(s[i:])
	fmt.Printf("%d\t%c\n", i, r)
	i += size
}
```

函数`DecodeRuneInString(s)`返回字符串`s`中的第一个rune及其UTF-8编码占用的字节数（用于更新索引`i`）。但这种循环很笨拙。幸运的是，当Go的`range`循环用于字符串时，会隐式执行UTF-8解码。以下循环的输出如上图所示（注意，对于非ASCII字符，索引步长大于1）。

```go
for i, r := range "Hello, 世界" {
	fmt.Printf("%d\t%q\t%d\n", i, r, r)
}
```

可以使用一个简单的`range`循环来统计字符串中的字符数：

```go
n := 0
for range s {
	n++
}
```

或者可以直接调用`utf8.RuneCountInString(s)`（注：该函数的实现方式就是上面的循环）。

前面提到过，在Go中文本字符串采用UTF-8编码只是一个惯例。但为了正确地对字符串使用`range`循环，这就是必需的。如果遍历非UTF-8或者任意二进制数据，当UTF-8解码器（无论是显式调用`DecodeRuneInString()`还是隐式的`range`循环）遇到非法输入字节时，都会生成一个特殊的Unicode**替换字符**(replacement character) `'\uFFFD'`，显示为 "�" 。当程序遇到这个rune值时，通常表明生成字符串数据的某个上游系统没有正确处理文本编码。

UTF-8作为交换格式非常方便，但在程序内部rune序列可能更方便，因为大小统一，可以很容易在数组和切片中索引。

将UTF-8编码的字符串转换为`[]rune`返回原始Unicode码点序列（相当于解码操作）：

```go
// "program" in Japanese katakana
s := "プログラム"
fmt.Printf("% x\n", s) // "e3 83 97 e3 83 ad e3 82 b0 e3 83 a9 e3 83 a0"
r := []rune(s)
fmt.Printf("%x\n", r)  // "[30d7 30ed 30b0 30e9 30e0]"
```

（动词`% x`在每对十六进制数之间插入一个空格）

将`[]rune`转换为字符串返回UTF-8编码序列（相当于编码操作）：

```go
fmt.Println(string(r)) // "プログラム"
```

将一个整数值转换为字符串会将该整数解释为rune值，生成对应的UTF-8表示：

```go
fmt.Println(string(65))     // "A", not "65"
fmt.Println(string(0x4eac)) // "京"
```

对于无效的rune值返回替换字符：

```go
fmt.Println(string(1234567)) // "�"
```

### 3.5.4 字符串和byte切片
标准库中有四个包对于字符串处理尤为重要：`bytes`、`strings`、`strconv`和`unicode`。
* `strings`包提供了`Contains()`、`Replace()`、`Compare()`、`Trim()`、`Split()`和`Join()`等字符串函数。
* `bytes`包提供了用于操作`[]byte`的类似函数。由于字符串是不可变的，通过拼接(`+=`)增量构建字符串会导致很多内存分配和拷贝。在这种情况下可以使用`bytes.Buffer`类型。
* `strconv`包提供了布尔值、整数和浮点数与字符串相互转换的函数，以及用于增删引号的函数。
* `unicode`包提供了`IsDigit()`、`IsLetter()`、`IsUpper()`和`IsLower()`等字符分类函数，以及`ToUpper()`和`ToLower()`等字符转换函数（`strings`包中的同名函数用于处理字符串）。

下面的`basename()`函数灵感来自同名的Unix shell工具。在我们的版本中，`basename(s)`删除`s`中由`/`分隔的文件系统路径前缀，以及由`.`分隔的文件类型后缀：

```go
fmt.Println(basename("a/b/c.go")) // "c"
fmt.Println(basename("c.d.go"))   // "c.d"
fmt.Println(basename("abc"))      // "abc"
```

第一个版本不使用任何库函数：

[gopl.io/ch3/basename1](https://github.com/ZZy979/gopl.io/blob/main/ch3/basename1/main.go)

更简单的版本使用了库函数`strings.LastIndex()`：

[gopl.io/ch3/basename2](https://github.com/ZZy979/gopl.io/blob/main/ch3/basename2/main.go)

`path`和`path/filepath`包提供了一组更通用的路径名操作函数。`path`处理斜杠分隔的路径，不应该用于文件名，但适用于其他域（例如URL的路径组件）。`path/filepath`使用主机平台的规则来处理文件名（例如POSIX的`/foo/bar`或Windows的`c:\foo\bar`）。

下面继续另一个字符串示例，将整数转换为字符串，并插入千位分隔符，例如`"12,345"`。这个版本仅适用于整数，处理浮点数留作练习。

[gopl.io/ch3/comma](https://github.com/ZZy979/gopl.io/blob/main/ch3/comma/main.go)

函数`comma()`的参数是一个字符串。如果其长度小于等于3，则不需要逗号。否则，使用除最后三个字符外的子串递归调用自身，并在递归调用的结果后添加逗号和最后三个字符。

字符串包含一个`byte`数组，一旦创建就是不可变的。相反，`byte`切片的元素可以自由修改。

字符串和`byte`切片可以相互转换：

```go
s := "abc"
b := []byte(s)
s2 := string(b)
```

从概念上讲，转换`[]byte(s)`会分配一个新的字节数组，包含`s`的字节的副本，并生成一个引用这整个数组的切片，这样即使`b`被修改`s`也不会改变。反向转换`string(b)`也会创建一个副本，以确保`s2`是不可变的。

为了避免转换和不必要的内存分配，`bytes`包直接提供了许多与`strings`包中对应的辅助函数。例如，`strings`包中的几个函数`Contains()`、`Count()`、`Fields()`、`HasPrefix()`、`Index()`和`Join()`，在`bytes`包中有对应的同名函数，唯一的区别是参数和返回类型中的`string`替换为`[]byte`。

`bytes`包提供了`Buffer`类型，用于高效操作字节切片。`Buffer`一开始是空的，可以向其写入`string`、`byte`和`[]byte`类型的数据。`bytes.Buffer`变量不需要初始化，因为其零值是可用的。

[gopl.io/ch3/printints](https://github.com/ZZy979/gopl.io/blob/main/ch3/printints/main.go)

写入任意rune的UTF-8编码时，最好使用`WriteRune()`方法，但`WriteByte()`可用于ASCII字符。（注：只有值小于128的rune字面值才能赋给`byte`类型）

`bytes.Buffer`类型用途极其广泛。在第7章讨论接口时将会看到，它可以充当I/O函数需要的字节接收器(`io.Writer`)（例如上面的`Fprintf()`）或字节源(`io.Reader`)。

练习3.10 使用`bytes.Buffer`而不是字符串拼接编写非递归版本的`comma`。

练习3.11 修改`comma`使其正确处理浮点数和可选的正负号。

练习3.12 编写一个函数，判断两个字符串是否互为变位词，即以不同的顺序包含相同的字母。

### 3.5.5 字符串和数字转换
通常需要在数值和字符串之间进行转换，为此可以使用`strconv`包中的函数。

要将整数转换为字符串，一种方法是使用`fmt.Sprintf()`；另一种方法是使用函数`strconv.Itoa()` ("integer to ASCII")：

```go
x := 123
y := fmt.Sprintf("%d", x)
fmt.Println(y, strconv.Itoa(x)) // "123 123"
```

`FormatInt()`和`FormatUint()`可用于格式化不同基数（进制）的数字：

```go
fmt.Println(strconv.FormatInt(int64(x), 2)) // "1111011"
```

`Printf`动词`%b`、`%d`、`%o`和`%x`通常比`Format`函数更方便，特别是需要包含数字之外的其他信息：

```go
s := fmt.Sprintf("x=%b", x) // "x=1111011"
```

要将字符串解析为整数，使用`strconv.Atoi()`、`ParseInt()`或`ParseUint()`：

```go
x, err := strconv.Atoi("123")             // x is an int
y, err := strconv.ParseInt("123", 10, 64) // base 10, up to 64 bits
```

`ParseInt()`的第三个参数用于指定结果的整数类型大小，例如16表示`int16`，特殊值0表示`int`。在任何情况下，结果`y`的类型始终是`int64`，之后可以将其转换为更小的类型。

`fmt.Scanf()`可用于解析字符串和数字混合在一行的输入，但它不太灵活，特别是在处理不完整或不规则的输入时。

## 3.6 常量
**常量**(constant)是值对于编译器已知的表达式，其求值发生在编译时而不是运行时。常量的底层类型只能是`bool`、`string`或数字。

`const`声明用于定义常量，可以防止在程序运行期间意外修改。例如：

```go
const pi = 3.14159 // approximately; math.Pi is a better approximation
```

与变量一样，多个常量可以出现在一个声明中，这适合一组相关的值：

```go
const (
	e = 2.71828182845904523536028747135266249775724709369995957496696763
	pi = 3.14159265358979323846264338327950288419716939937510582097494459
)
```

许多常量计算可以在编译时完成，这可以减少运行时的工作，也便于其他编译器优化。当操作数是常量时，一些运行时错误可以在编译时被发现，例如整数除以零、字符串索引越界、无效的浮点运算等。

对于常量的所有算术、逻辑和比较运算的结果也是常量，类型转换和某些内置函数（如`len()`、`cap()`、`real()`、`imag()`、`complex()`和`unsafe.Sizeof()`）的调用也是一样。

常量表达式可以出现在类型中，例如数组类型的长度：

```go
const IPv4Len = 4

// parseIPv4 parses an IPv4 address (d.d.d.d).
func parseIPv4(s string) IP {
	var p [IPv4Len]byte
	// ...
}
```

常量声明可以指定类型和值，如果没有指定类型，则从右侧的表达式推断出来。在下面的示例中，`time.Duration`的底层类型是`int64`，`time.Minute`是该类型的常量。因此下面声明的两个常量的类型都是`time.Duration`：

```go
const noDelay time.Duration = 0
const timeout = 5 * time.Minute
fmt.Printf("%T %[1]v\n", noDelay)     // "time.Duration 0"
fmt.Printf("%T %[1]v\n", timeout)     // "time.Duration 5m0s"
fmt.Printf("%T %[1]v\n", time.Minute) // "time.Duration 1m0s"
```

当声明一组常量时，除了第一个外，其他的右侧表达式都可以省略，这表示使用前一个表达式。例如：

```go
const (
	a = 1
	b
	c = 2
	d
)

fmt.Println(a, b, c, d) // "1 1 2 2"
```

如果隐式复制的右侧表达式的值总是相同的，这就不是很有用。但如果它可以变化呢？这就是`iota`。

### 3.6.1 常量生成器iota
`const`声明可以使用**常量生成器** `iota`，它用于创建一系列相关的值，而无需显式写出每个值。`iota`的值从0开始，每个常量加1。

下面的例子来自`time`包，它为表示星期的`Weekday`类型定义了命名常量。这种类型通常称为**枚举**(enumeration)。

```go
type Weekday int

const (
	Sunday Weekday = iota
	Monday
	Tuesday
	Wednesday
	Thursday
	Friday
	Saturday
)
```

`Sunday`是0，`Monday`是1，以此类推。

也可以在更复杂的表达式中使用`iota`。下面的例子来自`net`包，用于给无符号整数的最低5位中的每个位指定一个名字：

```go
type Flags uint

const (
	FlagUp Flags = 1 << iota // is up
	FlagBroadcast            // supports broadcast access capability
	FlagLoopback             // is a loopback interface
	FlagPointToPoint         // belongs to a point-to-point link
	FlagMulticast            // supports multicast access capability
)
```

这些常量是连续的2的幂，分别对应一个比特位（位开关）。可以使用这些常量测试、设置或清除一个或多个比特位：

[gopl.io/ch3/netflag](https://github.com/ZZy979/gopl.io/blob/main/ch3/netflag/main.go)

下面是一个更复杂的例子，声明了1024的幂的名字：

```go
const (
	_ = 1 << (10 * iota)
	KiB // 1024
	MiB // 1048576
	GiB // 1073741824
	TiB // 1099511627776              (exceeds 1 << 32)
	PiB // 1125899906842624
	EiB // 1152921504606846976
	ZiB // 1180591620717411303424     (exceeds 1 << 64)
	YiB // 1208925819614629174706176
)
```

不过`iota`机制也有局限性。例如，无法生成1000的幂（KB、MB等），因为没有幂运算符。

练习3.13 编写KB、MB直到YB的常量声明，尽可能地紧凑。

### 3.6.2 无类型常量
Go语言的常量有点不同寻常。尽管常量可以具有任意基本数据类型（如`int`或`float64`），但未明确指定类型、用字面值初始化的常量称为**无类型常量**(untyped constant)（例如前面的`KiB`、`MiB`等）。无类型常量具有比基本类型的值高得多的算术精度，可以假定至少有256位精度。无类型常量有6种“风格”(flavor)：布尔值、整数、浮点数、复数、rune和字符串。

通过延迟确定类型，无类型常量不仅可以保持更高的精度，并且可以直接用于表达式而无需类型转换。例如，上面例子中的`ZiB`和`YiB`太大而无法存储在任何整数变量中，但下面的表达式是合法的（因为`YiB/ZiB`在编译时求值，结果是1024）：

```go
fmt.Println(YiB/ZiB) // "1024"
```

另一个例子是，浮点常量`math.Pi`可用于任何需要浮点数或复数值的地方：

```go
var x float32 = math.Pi
var y float64 = math.Pi
var z complex128 = math.Pi
```

如果`math.Pi`被指定为一种特定的类型（如`float64`），结果就不会那么精确，并且赋给其他类型时需要类型转换：

```go
const Pi64 float64 = math.Pi

var x float32 = float32(Pi64)
var y float64 = Pi64
var z complex128 = complex128(Pi64)
```

字面值常量的语法决定了其“风格”：字面值`0`、`0.0`、`0i`和`'\u0000'`都表示同一个常量值，但分别是无类型整数、浮点数、复数和rune。`true`和`false`是无类型布尔值，字符串字面值是无类型字符串。

前面提到过，`/`可能表示整数或浮点除法，取决于其操作数。因此，字面值的写法可能会影响常量除法表达式的结果：

```go
var f float64 = 212
fmt.Println((f - 32) * 5 / 9)     // "100"; (f - 32) * 5 is a float64
fmt.Println(5 / 9 * (f - 32))     // "0"; 5/9 is an untyped integer, 0
fmt.Println(5.0 / 9.0 * (f - 32)) // "100"; 5.0/9.0 is an untyped float
```

只有常量可以是无类型的。当无类型常量被赋值给变量，或者出现在具有显式类型的变量声明的右侧时，该常量就会被隐式转换为变量的类型。

```go
var f float64 = 3 + 0i // untyped complex -> float64
f = 2                  // untyped integer -> float64
f = 1e123              // untyped floating-point -> float64
f = 'a'                // untyped rune -> float64
```

无论显式还是隐式转换，都要求目标类型能够表示原始值。浮点数和复数允许舍入：

```go
const (
	deadbeef = 0xdeadbeef // untyped int with value 3735928559
	a = uint32(deadbeef)  // uint32 with value 3735928559
	b = float32(deadbeef) // float32 with value 3735928576 (rounded up)
	c = float64(deadbeef) // float64 with value 3735928559 (exact)
	d = int32(deadbeef)   // compile error: constant overflows int32
	e = float64(1e309)    // compile error: constant overflows float64
	f = uint(-1)          // compile error: constant underflows uint
)
```

在没有显式类型的变量声明中（包括短变量声明），无类型常量的风格隐式决定了变量的默认类型。例如：

```go
i := 0      // untyped integer; implicit int(0)
r := '\000' // untyped rune; implicit rune('\000')
f := 0.0    // untyped floating-point; implicit float64(0.0)
c := 0i     // untyped complex; implicit complex128(0i)
```

无类型整数转换为`int`（大小不确定），但无类型浮点数和复数分别转换为确定大小的`float64`和`complex128`类型。Go语言没有不确定大小的`float`和`complex`类型。

要让变量是不同的类型，必须将无类型常量显式转换为期望的类型，或者在变量声明中指定类型。例如：

```go
var i = int8(0)
var i int8 = 0
```

在将无类型常量转换为接口值时，这些默认类型尤为重要，因为它们决定了其动态类型。

```go
fmt.Printf("%T\n", 0)      // "int"
fmt.Printf("%T\n", 0.0)    // "float64"
fmt.Printf("%T\n", 0i)     // "complex128"
fmt.Printf("%T\n", '\000') // "int32" (rune)
```
