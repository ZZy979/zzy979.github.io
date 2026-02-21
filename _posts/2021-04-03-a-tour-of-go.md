---
title: A Tour of Go
date: 2021-04-03 23:25 +0800
categories: [Go]
tags: [go, hello world, package, import statement, function, variable, initialization, data type, complex number, type conversion, constant, for statement, if statement, switch statement, defer statement, pointer, struct, array, slice, map, method, interface, error handling, io, generic programming, concurrency, goroutine, channel, select statement, lock]
---
本教程覆盖了Go语言最重要的特性，包括：
* 基础知识：变量和函数、流控制语句、内置类型
* 方法和接口
* 泛型
* 并发

官方网站：<https://go.dev/tour/list>

代码：<https://github.com/ZZy979/go-tutorials/tree/main/tour>

## 1.Hello World
<https://go.dev/tour/welcome>

下面是Go语言的Hello World程序：

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello, 世界")
}
```

可以使用[Go Playground](https://go.dev/play/)在线运行。也可以本地[安装Go](https://go.dev/doc/install)并在命令行中运行：

```shell
$ go run hello.go 
Hello, 世界
```

## 2.基础知识
### 2.1 包、函数和声明
<https://go.dev/tour/basics>

#### 2.1.1 包
Go程序由**包**(package)组成，程序从`main`包开始运行。

下面的程序使用了导入路径为`"fmt"`和`"math/rand"`的包。

[basics/packages.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/basics/packages.go)

包名与导入路径的最后一个元素相同。例如，`math/rand`包中的文件以`package rand`开头。

#### 2.1.2 导入
多个导入语句可以使用括号组织在一起，称为 "factored" （提取公因式的）。例如，

```go
import (
	"fmt"
	"math"
)
```

等价于

```go
import "fmt"
import "math"
```

使用factored导入语句是一种良好的编程风格。

[basics/imports.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/basics/imports.go)

#### 2.1.3 导出名字
在Go中，以大写字母开头的名字称为**导出的**(exported)。例如`Pi`是导出的，而`pi`不是导出的。

当导入一个包时，只能访问其导出的名字，未导出的名字只能在包内访问。（注：二者的区别相当于Java的`public`和包可见）

例如，以下代码将报错 "name cos not exported by package math" 。应将`math.cos`改为`math.Cos`。

[basics/exported-names.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/basics/exported-names.go)

#### 2.1.4 函数
**函数**(function)使用关键字`func`定义，可以接受零个或多个参数。注意Go的声明语法中类型在变量名之后。

例如，函数`add()`接受两个`int`类型的参数，返回类型为`int`。

```go
func add(x int, y int) int {
	return x + y
}
```

[basics/functions.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/basics/functions.go)

当两个或两个以上连续的参数类型相同时，可以只保留最后一个。例如，`x int, y int`可简写为`x, y int`。

函数可以返回任意个结果。例如，函数`swap()`返回两个字符串：

```go
func swap(x, y string) (string, string) {
	return y, x
}
```

[basics/multiple-results.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/basics/multiple-results.go)

注意：和Java一样，Go的函数是**传值调用**（包括数组和结构体类型），因此无法通过形参修改实参。但切片和映射等都是引用类型，形参和实参是指向同一对象的两个“指针”，因此可以通过形参修改实参指向的值。

#### 2.1.5 命名的返回值
Go的函数返回值可以被命名，可以将其当作在函数顶部定义的变量。这些名字应当标明返回值的含义。

无参数的`return`语句返回这些命名返回值，称为“裸`return`”。裸`return`语句只应当用在短函数中，如下例所示。

```go
func split(sum int) (x, y int) {
	x = sum * 4 / 9
	y = sum - x
	return
}
```

[basics/named-results.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/basics/named-results.go)

#### 2.1.6 变量
`var`语句用于声明**变量**(variable)，类型放在最后。

`var`语句可以位于包级别或函数级别，如下例所示。

```go
var c, python, java bool

func main() {
	var i int
	fmt.Println(i, c, python, java)
}
```

[basics/variables.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/basics/variables.go)

变量声明可以包含初始值。此时可以省略类型，变量类型由初始值决定。

```go
var i, j int = 1, 2

func main() {
	var c, python, java = true, false, "no!"
	fmt.Println(i, j, c, python, java)
}
```

[basics/variables-with-initializers.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/basics/variables-with-initializers.go)

在函数内，可以用`:=`短赋值语句替代有初始值的`var`声明。在函数外不能使用`:=`。

例如：

```go
k := 3
c, python, java := true, false, "no!"
```

[basics/short-variable-declarations.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/basics/short-variable-declarations.go)

#### 2.1.7 基本类型
Go的基本类型有：
* `bool`
* `string`
* `int`, `int8`, `int16`, `int32`, `int64`, `uint`, `uint8`, `uint16`, `uint32`, `uint64`, `uintptr`
* `byte`（`uint8`的别名）
* `rune`（`int32`的别名，表示Unicode码点）
* `float32`, `float64`
* `complex64`, `complex128`

下面的例子展示了几种类型的变量。与`import`语句一样，变量声明也可以写成 "factored" 形式。

```go
var (
	ToBe   bool       = false
	MaxInt uint64     = 1<<64 - 1
	z      complex128 = cmplx.Sqrt(-5 + 12i)
)

func main() {
	fmt.Printf("Type: %T Value: %v\n", ToBe, ToBe)
	fmt.Printf("Type: %T Value: %v\n", MaxInt, MaxInt)
	fmt.Printf("Type: %T Value: %v\n", z, z)
}
```

[basics/basic-types.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/basics/basic-types.go)

`int`、`uint`和`uintptr`通常在32位系统上是32位宽，在64位系统上是64位宽。当需要整数值时，应该使用`int`，除非有特殊原因使用固定宽度或无符号整数类型。

#### 2.1.8 零值
没有显式初始化的变量将被赋予**零值**(zero value)。各种类型的零值如下表所示：

| 类型 | 零值 |
| --- | --- |
| 数值类型 | `0` |
| 布尔类型 | `false` |
| 字符串 | `""` |
| 指针、切片、映射、函数、接口、channel | `nil` |
| 数组 | 所有元素都是零值 |
| 结构体 | 所有每个字段都是零值 |

[basics/zero.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/basics/zero.go)

#### 2.1.9 类型转换
表达式`T(v)`将值`v`转换为类型`T`。例如：

```go
i := 42
f := float64(i)
u := uint(f)
```

与C语言不同，在Go中不同类型间赋值必须进行显式类型转换。删除下面代码中的`float64`或`uint`转换将导致编译失败。

[basics/type-conversions.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/basics/type-conversions.go)

#### 2.1.9 类型推断
当声明变量而没有显式指定类型时，变量的类型将通过右侧的值推断得到。

当右侧值是一个变量时，新变量的类型与其相同：

```go
var i int
j := i // j is an int
```

当右侧值是数字常量时，整数、浮点数和复数的类型分别为`int`、`float64`和`complex128`：

```go
i := 42           // int
f := 3.142        // float64
g := 0.867 + 0.5i // complex128
```

[basics/type-inference.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/basics/type-inference.go)

#### 2.1.10 常量
常量使用关键字`const`声明，不能使用`:=`语法。例如：

```go
const Pi = 3.14
```

[basics/constants.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/basics/constants.go)

数字常量是高精度的值。无类型的常量会根据其上下文需要采用相应的类型。

例如，在下面的程序中打印`needInt(Big)`会导致编译失败，因为将无类型常量`Big`赋给`int`值会溢出。

[basics/numeric-constants.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/basics/numeric-constants.go)

### 2.2 流控制语句：for、if、switch和defer
<https://go.dev/tour/flowcontrol>

#### 2.2.1 for语句
Go只有一种循环语句，即`for`语句，语法如下：

```go
for init; condition; post {
	statements
}
```

例如：

```go
sum := 0
for i := 0; i < 10; i++ {
	sum += i
}
```

[flowcontrol/for.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/flowcontrol/for.go)

注意：
* 与C和Java不同，Go的`for`语句三部分周围没有小括号，而大括号是必需的。
* `init`部分通常是短赋值语句(`:=`)，声明的变量仅在`for`语句内部可见。
* Go的`++`运算符没有前缀形式。

`for`语句的`init`和`post`部分是可选的。例如：

```go
sum := 1
for ; sum < 1000; {
	sum += sum
}
```

[flowcontrol/for-continued.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/flowcontrol/for-continued.go)

如果只有`condition`部分，则可以将分号省略。此时`for`语句相当于C语言的`while`语句。

```go
sum := 1
for sum < 1000 {
	sum += sum
}
```

[flowcontrol/for-is-gos-while.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/flowcontrol/for-is-gos-while.go)

如果省略循环条件，就是无限循环：

```go
for {
}
```

[flowcontrol/forever.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/flowcontrol/forever.go)

#### 2.2.2 if语句
`if`语句的语法如下：

```go
if condition {
	statements
}
```

与`for`语句类似，条件周围没有小括号，但大括号是必需的。例如：

```go
func sqrt(x float64) string {
	if x < 0 {
		return sqrt(-x) + "i"
	}
	return fmt.Sprint(math.Sqrt(x))
}
```

[flowcontrol/if.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/flowcontrol/if.go)

`if`语句也可以包含变量声明，作用域仅在`if`语句内部。

```go
func pow(x, n, lim float64) float64 {
	if v := math.Pow(x, n); v < lim {
		return v
	}
	return lim
}
```

[flowcontrol/if-with-a-short-statement.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/flowcontrol/if-with-a-short-statement.go)

`if-else`语句的语法如下：

```go
if condition {
	statements
} else {
	statements
}
```

例如：

```go
func pow(x, n, lim float64) float64 {
	if v := math.Pow(x, n); v < lim {
		return v
	} else {
		fmt.Printf("%g >= %g\n", v, lim)
	}
	// can't use v here, though
	return lim
}
```

[flowcontrol/if-and-else.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/flowcontrol/if-and-else.go)

注意：
* `if`语句声明的变量在`else`部分也可以访问。
* 大括号的位置不能改变！

##### 练习：牛顿法求平方根
使用[牛顿法](https://en.wikipedia.org/wiki/Newton%27s_method)实现平方根函数`Sqrt(x)`：初始猜测值z=1.0，使用迭代公式

```
z -= (z*z - x) / (2*z)
```

直到z<sup>2</sup>足够接近x。

[flowcontrol/exercise-loops-and-functions.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/flowcontrol/exercise-loops-and-functions.go)

#### 2.2.3 switch语句
`switch`语句是一种比`if-else`语句序列更简洁的方式，它会执行第一个值与条件表达式相等的case对应的语句。语法如下：

```go
switch expr {
case val1:
	statement1
case val2:
	statement2
default:
	statement
}
```

例如：

```go
fmt.Print("Go runs on ")
switch os := runtime.GOOS; os {
case "darwin":
	fmt.Println("macOS.")
case "linux":
	fmt.Println("Linux.")
default:
	fmt.Printf("%s.\n", os)
}
```

[flowcontrol/switch.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/flowcontrol/switch.go)

与C和Java不同，Go的`switch`语句只执行匹配的**一个**分支（即没有fall through），相当于C语言的`switch`每个分支结尾自动添加了`break`。

另一个重要区别是，Go的`switch`中每个case的值不要求是常量，也不要求是整数。

`switch`的求值顺序为从上到下，直到一个case匹配成功。例如，

```go
switch i {
case 0:
case f():
}
```

如果`i==0`则不会调用`f()`。

[flowcontrol/switch-evaluation-order.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/flowcontrol/switch-evaluation-order.go)

没有条件的`switch`语句等价于`switch true`（即执行等于true的分支）。例如：

```go
t := time.Now()
switch {
case t.Hour() < 12:
	fmt.Println("Good morning!")
case t.Hour() < 17:
	fmt.Println("Good afternoon.")
default:
	fmt.Println("Good evening.")
}
```

[flowcontrol/switch-with-no-condition.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/flowcontrol/switch-with-no-condition.go)

#### 2.2.4 defer语句
`defer`语句用于将函数的执行延迟到其所在函数返回时，但参数会立即求值。

例如，下面的程序会先打印 "hello" ，当`main()`函数返回时再打印 "world" 。

```go
func main() {
	defer fmt.Println("world")

	fmt.Println("hello")
}
```

[flowcontrol/defer.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/flowcontrol/defer.go)

`defer`函数调用被压入一个栈中。当前函数返回时，按照后进先出的顺序执行这些调用。

关于`defer`语句的更多细节参见[Defer, Panic, and Recover](https://go.dev/blog/defer-panic-and-recover)。

例如，下面的程序

```go
func main() {
	fmt.Println("counting")

	for i := 0; i < 10; i++ {
		defer fmt.Println(i)
	}

	fmt.Println("done")
}
```

输出结果为

```
counting
done
9
8
7
6
5
4
3
2
1
0
```

[flowcontrol/defer-multi.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/flowcontrol/defer-multi.go)

### 2.3 内置类型：结构体、切片和映射
<https://go.dev/tour/moretypes>

Go的数据类型分为四类
* 基本类型：数字、字符串、布尔型
* 复合类型：数组、结构体
* 引用类型：指针、切片、映射、函数、channel
* 接口类型

#### 2.3.1 指针
Go有**指针**(pointer)。指针用于保存值的内存地址。

类型`*T`是指向`T`类型值的指针。指针的零值是`nil`。

运算符`&`生成指向一个值的指针：

```go
i := 42
p = &i // p points to i
```

运算符`*`表示指针指向的值，称为**解引用**(dereferencing)：

```go
fmt.Println(*p) // read i through the pointer p
*p = 21         // set i through the pointer p
```

与C语言不同，Go没有指针运算（如`p+1`或`p-q`）。

[moretypes/pointers.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/moretypes/pointers.go)

#### 2.3.2 结构体
**结构体**(struct)是一些**字段**(field)的集合。例如：

```go
type Vertex struct {
	X int
	Y int
}
```

像这样创建结构体类型的变量：

```go
v := Vertex{1, 2}
```

[moretypes/structs.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/moretypes/structs.go)

使用`.`运算符访问结构体的字段：

```go
v.X = 4
fmt.Println(v.X)
```

[moretypes/struct-fields.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/moretypes/struct-fields.go)

##### 指向结构体的指针
结构体字段可以通过结构体指针访问。要访问指针`p`指向的结构体的字段`X`，可以使用`(*p).X`，也可简写为`p.X`（注：Go没有`p->X`这种写法）。

```go
v := Vertex{1, 2}
p := &v
p.X = 1e9
```

[moretypes/struct-pointers.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/moretypes/struct-pointers.go)

##### 结构体字面值
结构体字面值表示一个新创建的结构体值，列出所有字段的值：

```go
v1 := Vertex{1, 2}  // has type Vertex
```

也可以使用`Name: Value`语法（类似于Python的关键字参数）。此时可以仅列出一部分字段（省略的字段使用对应类型的零值），字段的顺序无关紧要。

```go
v2 := Vertex{X: 1}  // Y:0 is implicit
v3 := Vertex{}      // X:0 and Y:0
```

[moretypes/struct-literals.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/moretypes/struct-literals.go)

注意：Go允许获取结构体字面值的指针，这在C++中是不合法的（右值不能取地址，除非是右值引用）。但Go不允许获取数字常量的指针。

```go
p := &Vertex{1, 2} // OK
q := &42 // error
```

#### 2.3.3 数组
类型`[n]T`是具有`n`个`T`类型值的数组。例如，

```go
var a [10]int
```

将变量`a`声明为包含10个整数的数组。

使用`[]`运算符访问数组元素：

```go
a[0] = "Hello"
a[1] = "World"
fmt.Println(a[0], a[1])
```

声明数组变量时可以使用`{}`提供元素的初始值，例如：

```go
primes := [6]int{2, 3, 5, 7, 11, 13}
```

数组的长度是其类型的一部分，因此`[3]int`和`[4]int`是两种不同的数组类型，并且数组不能改变大小。

[moretypes/array.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/moretypes/array.go)

#### 2.3.4 切片
数组的大小是固定的。**切片**(slice)是一种动态大小、灵活的视图，可以访问数组元素。实际上，切片比数组常用得多。

类型`[]T`是具有`T`类型元素的切片。

通过指定数组和两个索引（下界和上界）来构造切片：`a[low:high]`表示数组`a`的一个左闭右开区间（包括`a[low]`，不包括`a[high]`）。

例如：

```go
var s []int = primes[1:4]
```

[moretypes/slices.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/moretypes/slices.go)

**切片类似于数组的引用**
* 切片不存储数据，仅描述底层数组的一部分。
* 多个切片可以共享底层数组，并且引用的数组区间可能重叠。
* 改变切片的元素会修改底层数组的对应元素，共享同一个底层数组的其他切片也能看到修改。
* 切片之间不能进行相等比较。

[moretypes/slices-pointers.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/moretypes/slices-pointers.go)

##### 切片字面值
切片字面值类似于数组字面值，但没有长度。例如：

```go
[]bool{true, true, false}
```

切片字面值构造了一个包含相同元素的数组，并创建了一个引用它的切片。

[moretypes/slice-literals.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/moretypes/slice-literals.go)

##### 切片默认值
切片下界的默认值是0，上界的默认值是数组/切片的长度。

例如，对于长度为10的数组`a`，以下切片表达式是等价的：

```go
a[0:10]
a[:10]
a[0:]
a[:]
```

[moretypes/slice-bounds.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/moretypes/slice-bounds.go)

##### 切片长度和容量
切片具有**长度**(length)和**容量**(capacity)。
* 切片的长度是它包含元素的个数。
* 切片的容量是底层数组**从下界开始**的元素个数。

例如，对于长度为10的数组`a`，切片`a[3:8]`的长度为5、容量为7。

可以使用表达式`len(s)`和`cap(s)`获得切片`s`的长度和容量。

可以通过重新切片来扩展切片的长度，前提是长度不能超过容量。例如：

```go
s := []int{2, 3, 5, 7, 11, 13} // len=6 cap=6
s = s[:0]  // len=0 cap=6 [] 2 3 5 7 11 13
s = s[:4]  // len=4 cap=6 [2 3 5 7] 11 13
s = s[2:]  // len=2 cap=4 2 3 [5 7] 11 13
s = s[1:4] // len=3 cap=3 2 3 5 [7 11 13]
s = s[0:8] // runtime error: slice bounds out of range [:8] with capacity 3
```

[moretypes/slice-len-cap.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/moretypes/slice-len-cap.go)

##### Nil切片
切片的零值是`nil`。

`nil`切片的长度和容量都是0，没有底层数组。

[moretypes/nil-slices.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/moretypes/nil-slices.go)

##### 用make函数创建切片
可以使用内置函数`make()`创建切片——这就是创建动态数组的方式。

`make()`函数创建一个给定长度的零值数组，并返回引用该数组的切片：

```go
a := make([]int, 5)  // len(a)=5
```

可以通过第三个参数指定容量：

```go
b := make([]int, 0, 5) // len(b)=0, cap(b)=5

b = b[:cap(b)] // len(b)=5, cap(b)=5
b = b[1:]      // len(b)=4, cap(b)=4
```

[moretypes/making-slices.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/moretypes/making-slices.go)

##### 切片的切片
切片的元素可以是任意类型，包括切片。例如：

```go
board := [][]string{
	{"_", "_", "_"},
	{"_", "_", "_"},
	{"_", "_", "_"},
}
```

[moretypes/slices-of-slice.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/moretypes/slices-of-slice.go)

##### 向切片添加元素
Go的内置函数`append()`用于向切片添加新元素，返回新的切片。详见文档[func append](https://pkg.go.dev/builtin#append)。

```go
func append(s []T, vs ...T) []T
```

第一个参数`s`是切片，其余参数是要添加到切片的值。返回值是原切片加上新元素后的切片。例如：

```go
var s []int            // len=0 cap=0 []
s = append(s, 0)       // len=1 cap=1 [0]
s = append(s, 1)       // len=2 cap=2 [0 1]
s = append(s, 2, 3, 4) // len=5 cap=6 [0 1 2 3 4]
```

[moretypes/append.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/moretypes/append.go)

注：向切片中添加新元素时
* 如果切片的底层数组足够大，则会覆盖数组中切片尾部之后的元素；否则会创建一个新的数组，返回的切片将会引用新的数组，此时不会修改原来的底层数组。
* 由于不能确认新的切片和原始切片是否引用相同的底层数组，应该将`append()`的返回值直接赋值给输入的切片变量。例如：

```go
a := []int{0, 1, 2}
s := a[0:2]         // a=[0 1 2], s=[0 1]
s = append(s, 3)    // a=[0 1 3], s=[0 1 3]
s = append(s, 4, 5) // a=[0 1 3], s=[0 1 3 4 5]
```

关于切片的更多细节参见[Go Slices: usage and internals](https://go.dev/blog/slices-intro)。

#### 2.3.5 Range
`range`用于在`for`循环中遍历切片或映射。

使用`range`遍历切片时，每次返回两个值：索引和对应元素的拷贝（相当于Python中的`enumerate()`函数）。

```go
var pow = []int{1, 2, 4, 8, 16, 32, 64, 128}
for i, v := range pow {
	fmt.Printf("2**%d = %d\n", i, v)
}
```

[moretypes/range.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/moretypes/range.go)

如果不需要索引或值，可以将对应的变量名改为`_`：

```go
for i, _ := range s
for _, v := range s
```

如果只需要索引，可以省略第二个变量（此时相当于Python中的`for i in range(len(s))`）（为什么这种形式不是遍历值？只遍历值不是更常用？）：

```go
for i := range s
```

[moretypes/range-continued.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/moretypes/range-continued.go)

##### 练习：绘制函数图像
实现函数`Pic(dx, dy)`，返回一个长度为`dy`的二维切片`[][]uint8`，其中每个元素是长度为`dx`的8位无符号整数切片。

运行该程序时，将展示一张图像，将`Pic()`返回的二维切片作为像素灰度值。每个元素由其x和y坐标计算得到，计算方式可以任意选择，例如`(x+y)/2`、`x*y`或`x^y`（异或）。

[moretypes/exercise-slices.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/moretypes/exercise-slices.go)

不同计算函数得到的图像如下。

(x+y)/2

![avg](/assets/images/a-tour-of-go/avg.png)

x xor y

![xor](/assets/images/a-tour-of-go/xor.png)

xy

![mul](/assets/images/a-tour-of-go/mul.png)

x<sup>2</sup> + y<sup>2</sup>

![pol](/assets/images/a-tour-of-go/pol.png)

#### 2.3.6 映射
**映射**(map)将键映射到值。

类型`map[K]V`表示键类型为`K`、值类型为`V`的映射。映射的零值是`nil`。

使用`make()`函数创建映射：`make(map[K]V)`。

使用`[]`运算符访问与键关联的值。例如：

```go
m := make(map[string]Vertex)
m["Bell Labs"] = Vertex{40.68433, -74.39967}
```

[moretypes/maps.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/moretypes/maps.go)

##### 映射字面值
映射字面值类似于结构体字面值，但键是必需的：

```go
map[K]V{key1: value1, key2: value2, ...}
```

可以使用`map[K]V{}`创建空映射。

注意：如果写成多行的形式，最后一行的末尾也必须有逗号。例如：

```go
var m = map[string]Vertex{
	"Bell Labs": Vertex{40.68433, -74.39967},
	"Google":    Vertex{37.42202, -122.08408},
}
```

[moretypes/map-literals.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/moretypes/map-literals.go)

可以省略字面值中的结构体类型名。例如：

```go
var m = map[string]Vertex{
	"Bell Labs": {40.68433, -74.39967},
	"Google":    {37.42202, -122.08408},
}
```

[moretypes/map-literals-continued.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/moretypes/map-literals-continued.go)

##### 操作map
* 插入或更新元素：`m[key] = value`
* 读取元素：`v := m[key]`，如果键不存在则返回值类型的零值。
* 查询键是否存在：`v, ok := m[key]`，如果键存在则`ok`为`true`，`v`为对应的值；否则`ok`为`false`，`v`为值类型的零值。
* 删除元素：`delete(m, key)`，如果键不存在则什么都不做。
* 映射之间不能进行相等比较。

[moretypes/mutating-maps.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/moretypes/mutating-maps.go)

可以使用`range`遍历映射的键值对（相当于Python的`dict.items()`），顺序是不确定的。例如：

```go
m := map[string]int{"foo": 1, "bar": 2, "baz": 3}
for k, v := range m {
	fmt.Println(k, v)
}
```

Go语言没有提供set类型，但可以通过只使用映射的键实现。

##### 练习：单词计数
实现`WordCount(s)`函数，返回字符串`s`中每个“单次”的出现次数构成的映射。可以使用`strings.Fields()`将字符串按空白符分割成单词。

[moretypes/exercise-maps.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/moretypes/exercise-maps.go)

注：标准库[strings](https://pkg.go.dev/strings)和[strconv](https://pkg.go.dev/strconv)包提供了字符串操作。

#### 2.3.7 函数类型的值
在Go中函数也是值，可以用作函数参数和返回值。

例如：

```go
func compute(fn func(float64, float64) float64) float64 {
	return fn(3, 4)
}

func main() {
	hypot := func(x, y float64) float64 {
		return math.Sqrt(x*x + y*y)
	}
	fmt.Println(hypot(5, 12))
	fmt.Println(compute(hypot))
}
```

[moretypes/function-values.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/moretypes/function-values.go)

#### 2.3.8 函数闭包
**闭包**(closure)是引用了函数体外部变量的函数。该函数可以访问和赋值所引用的变量，此时称函数“绑定”到该变量。

例如，每次调用函数`adder()`都会返回一个闭包，每个闭包绑定到自己的`sum`变量。

```go
func adder() func(int) int {
	sum := 0
	return func(x int) int {
		sum += x
		return sum
	}
}
```

[moretypes/function-closures.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/moretypes/function-closures.go)

在上面的程序中，闭包`pos`和`neg`绑定的`sum`变量是独立的。

##### 练习：斐波那契闭包
实现`fibonacci()`函数，返回一个函数闭包，该闭包返回连续的斐波那契数(1, 1, 2, 3, 5, ...)。

[moretypes/exercise-fibonacci-closure.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/moretypes/exercise-fibonacci-closure.go)

注：这个例子类似于Python的生成器，每次调用`f`都会生成下一个值，但无法用`for`循环遍历。

## 3.方法和接口
<https://go.dev/tour/methods>

### 3.1 方法
Go没有类，但是可以在类型上定义方法。

**方法**(method)是带有特殊的**接收器**(receiver)参数的函数。（注：接收器类似于Python方法的`self`参数）

接收器出现在单独的参数列表中，位于`func`关键字和方法名之间。例如，

```go
func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}
```

在这个例子中，`Abs()`方法有一个`Vertex`类型的名为`v`的接收器。

使用`.`语法来调用方法，例如`v.Abs()`。

[methods/methods.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/methods/methods.go)

**方法就是带有接收器参数的特殊函数。** `Abs()`方法等价的普通函数写法如下：

```go
func Abs(v Vertex) float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}
```

[methods/methods-funcs.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/methods/methods-funcs.go)

也可以为非结构体类型定义方法。例如：

```go
type MyFloat float64

func (f MyFloat) Abs() float64 {
	if f < 0 {
		return float64(-f)
	}
	return float64(f)
}
```

注意：即使`MyFloat`是`float64`的别名，也必须使用类型转换。

方法接收器的类型必须与方法定义在同一个包中。不能为其他包中的类型定义方法（包括内置类型，例如`int`）。但可以通过定义类型别名实现，如上面的例子所示。

[methods/methods-continued.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/methods/methods-continued.go)

#### 指针接收器
可以声明带有指针接收器的方法，即接收器类型为`*T`，其中`T`本身不能是一个指针类型。

例如，下面的`Scale()`方法是对`*Vertex`定义的。

```go
func (v *Vertex) Scale(f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}
```

**带有指针接收器的方法可以修改接收器指针指向的值，而值接收器是原始值的副本。**

例如，如果将`Scale()`方法改为值接收器，那么`v.Scale(10)`无法修改`v`的字段，下面的程序将输出5而不是50。

[methods/methods-pointers.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/methods/methods-pointers.go)

注：与访问字段一样，通过指针调用方法也是使用`.`语法。

类似地，如果将`Scale()`重写为接受非指针参数的函数，则函数内修改的只是原始值的副本，不会影响实参。

[methods/methods-pointers-explained.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/methods/methods-pointers-explained.go)

调用带有指针参数的函数时，实参必须是指针：

```go
var v Vertex
ScaleFunc(v, 5)  // Compile error!
ScaleFunc(&v, 5) // OK
```

而调用带有指针接收器的方法时，接收器可以是指针也可以是值：

```go
var v Vertex
v.Scale(5)  // OK, same as (&v).Scale(5)
p := &v
p.Scale(10) // OK
```

这是因为Go会自动进行取地址，`v.Scale(5)`等价于`(&v).Scale(5)`。

[methods/indirection.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/methods/indirection.go)

反过来，调用带有值参数的函数时，实参必须是值：

```go
var v Vertex
fmt.Println(AbsFunc(v))  // OK
fmt.Println(AbsFunc(&v)) // Compile error!
```

而调用带有值接收器的方法时，接收器可以是指针也可以是值：

```go
var v Vertex
fmt.Println(v.Abs()) // OK
p := &v
fmt.Println(p.Abs()) // OK, same as (*p).Abs()
```

如果是指针则Go会自动解引用，即`p.Abs()`等价于`(*p).Abs()`。

[methods/indirection-values.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/methods/indirection-values.go)

选择指针接收器的原因有两个：
* 方法需要修改原始值。
* 避免值拷贝，对于大结构体将更加高效。

一般来说，一个类型的所有方法应该要么都是值接收器，要么都是指针接收器，不应该混合使用两者（在下一节中将解释原因）。

在下面的例子中，`Scale()`和`Abs()`方法的接收器类型都是`*Vertex`，尽管`Abs()`不需要修改原始值。

[methods/methods-with-pointer-receivers.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/methods/methods-with-pointer-receivers.go)

### 3.2 接口
**接口**(interface)类型定义为一组方法签名的集合。例如：

```go
type Abser interface {
	Abs() float64
}
```

接口`Abser`包含一个无参数、返回`float64`的方法`Abs()`。

**只要类型`T`实现了接口定义的所有方法，就实现了该接口。**

例如：

```go
type MyFloat float64

func (f MyFloat) Abs() float64 {
	if f < 0 {
		return float64(-f)
	}
	return float64(f)
}

type Vertex struct {
	X, Y float64
}

func (v *Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}
```

类型`MyFloat`和`*Vertex`都定义了`Abs()`方法，因此实现了`Abser`接口。

注意：
* 在上面的例子中，`Vertex`类型没有实现`Abser`接口，因为`Abs()`方法只对`*Vertex`类型定义了。
* 如果类型`T`定义了一个接口的所有方法，但有些是值接收器、有些是指针接收器，则`T`和`*T`都**没有**实现该接口。这就是为什么推荐一种类型的所有方法的接收器保持一致的原因。

接口类型的变量可以存放任何实现了该接口的类型的值。例如，在下面的程序中，`f`和`&v`都可以赋给`Abser`类型的变量`a`，但`v`不可以。

```go
func main() {
	var a Abser
	f := MyFloat(-math.Sqrt2)
	v := Vertex{3, 4}

	a = f  // MyFloat implements Abser
	a = &v // *Vertex implements Abser
	a = v  // error: Vertex does NOT implement Abser

	fmt.Println(a.Abs())
}
```

[methods/interfaces.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/methods/interfaces.go)

Go的接口是**隐式实现**的，不需要像Java一样用`implements`关键字显式声明。这使得接口的定义和实现可以解耦，不需要出现在同一个包中。

[methods/interfaces-are-satisfied-implicitly.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/methods/interfaces-are-satisfied-implicitly.go)

#### 接口值
接口值可以认为是值和实际类型的元组：`(value, type)`。

调用接口值的方法将会调用实际类型对应的同名方法（注：这类似于C++的虚函数调用）。

例如，假设类型`*T`和`F`都实现了接口`I`，则下面的两个`i.M()`分别调用`T.M()`和`F.M()`。

```go
var i I

i = &T{"Hello"}
i.M() // calls T.M()

i = F(math.Pi)
i.M() // calls F.M()
```

[methods/interface-values.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/methods/interface-values.go)

如果接口值的底层值是`nil`，则调用方法时接收器将会是`nil`。例如：

```go
var i I
var t *T
i = t // interface value with nil underlying value
i.M() // calls T.M() with nil receiver
```

在某些语言中这会引发空指针异常，但在Go中编写能够处理接收器为`nil`的方法是很常见的。例如下面例子中的方法`M()`。

```go
func (t *T) M() {
	if t == nil {
		fmt.Println("<nil>")
		return
	}
	fmt.Println(t.S)
}
```

[methods/interface-values-with-nil.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/methods/interface-values-with-nil.go)

注意：
* 底层值是`nil`的接口值**本身**不是`nil`。
* 在上面的例子中，如果接收器`t`是`nil`，则访问`t.S`仍然会报错 "runtime error: invalid memory address or nil pointer dereference" 。

`nil`接口值没有底层值和实际类型。对`nil`接口值调用方法将产生运行时错误，因为无法确定要调用哪个具体方法。

```go
var i I
i.M() // error: call method on nil interface value
```

[methods/nil-interface-values.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/methods/nil-interface-values.go)

#### 空接口
不包含任何方法的接口类型称为**空接口**：

```go
interface{}
```

空接口可以表示任何类型的值，因为所有类型都实现了至少0个方法（类似于Java的`Object`类）。

空接口通常用于处理未知类型的值的代码。例如，`fmt.Println()`接受任意个数的`interface{}`类型的参数。

[methods/empty-interface.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/methods/empty-interface.go)

#### 类型断言
**类型断言**(type assertion)用于访问接口值的底层值（即`(value, type)`中的`value`）。

```go
t := i.(T)
```

该语句断言接口值`i`的实际类型为`T`，并将底层值赋给变量`t`。如果`i`的实际类型不是`T`则会报错(panic)。

要**测试**一个接口值是否是特定的类型，可以使用返回两个值的类型断言：

```go
t, ok := i.(T)
```

如果`i`的实际类型是`T`，则`ok`是`true`，`t`是`i`的底层值；否则`ok`是`false`，`t`是`T`类型的零值。

[methods/type-assertions.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/methods/type-assertions.go)

#### 类型switch
**类型switch**类似于普通`switch`语句，但其中的case指定类型（而不是值），用于与接口值的实际类型进行比较。

例如：

```go
switch v := i.(type) {
case T:
	// here v has type T
case S:
	// here v has type S
default:
	// no match; here v has the same type as i
}
```

其中声明部分的语法与类型断言`i.(T)`相同，只是具体类型`T`被替换为关键字`type`。

这个`switch`语句判断接口值`i`的实际类型是否是`T`或`S`。在这两个case中，变量`v`的类型分别是`T`和`S`，值是`i`的底层值。在`default`分支中，`v`的类型和值与`i`相同。

[methods/type-switches.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/methods/type-switches.go)

#### Stringer接口
最常用的接口之一是`fmt`包定义的`Stringer`：

```go
type Stringer interface {
	String() string
}
```

`Stringer`表示可以描述为字符串的类型
（其`String()`方法类似于Java的`Object.toString()`和Python的`__str__()`方法）。

例如，`Person`类型实现了`Stringer`接口，可以将自身转换为字符串。

```go
type Person struct {
	Name string
	Age  int
}

func (p Person) String() string {
	return fmt.Sprintf("%v (%v years)", p.Name, p.Age)
}
```

如果一个类型实现了`Stringer`接口，那么`fmt`包中的打印函数会使用其`String()`方法返回的字符串，而不是默认表示。

[methods/stringer.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/methods/stringer.go)

#### 练习：Stringer
让`IPAddr`类型实现`fmt.Stringer`接口，以点分十进制格式打印IP地址。例如，`IPAddr{1, 2, 3, 4}`打印成`"1.2.3.4"`。

```go
type IPAddr [4]byte
```

[methods/exercise-stringer.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/methods/exercise-stringer.go)

### 3.3 错误
Go程序使用`error`值来表达错误状态。

`error`类型是一个内置接口：

```go
type error interface {
	Error() string
}
```

函数通常会返回一个`error`值，调用代码应当通过检查错误是否为`nil`来进行错误处理：错误为`nil`表示成功，否则表示失败。例如：

```go
i, err := strconv.Atoi("42")
if err != nil {
	fmt.Printf("couldn't convert number: %v\n", err)
	return
}
fmt.Println("Converted integer:", i)
```

[methods/errors.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/methods/errors.go)

注：这个例子使用了自定义错误类型`MyError`。可以使用标准库函数`errors.New()`来创建`error`值。

#### 练习：错误
修改之前的练习（2.2.2节）中牛顿法求平方根的`Sqrt()`函数，当参数为负数时返回一个`error`值。

创建一个新类型

```go
type ErrNegativeSqrt float64
```

并使其实现`error`接口：

```go
func (e ErrNegativeSqrt) Error() string
```

例如，`ErrNegativeSqrt(-2).Error()`返回`"cannot Sqrt negative number: -2"`。

注意：在`Error()`方法内部调用`fmt.Sprint(e)`会导致无限递归，因为该函数会调用`e.Error()`。为了避免这一问题，可以先将`e`转换为`float64`类型。

[methods/exercise-errors.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/methods/exercise-errors.go)

### 3.4 Reader接口
`io`包定义了`Reader`接口，用于读取流数据。Go标准库包含该接口的很多实现，包括文件、网络连接、压缩算法、加密算法等。

`io.Reader`接口有一个`Read()`方法：

```go
type Reader interface {
	Read(b []byte) (n int, err error)
}
```

`Read()`方法使用读取的数据填充给定的`byte`切片，返回填充的字节数和错误值。当流结束时返回`io.EOF`错误。

示例代码创建了一个`strings.Reader`，用于从字符串读取数据，每次读取8个字节。

[methods/reader.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/methods/reader.go)

#### 练习：Reader
实现一个`Reader`类型，生成字符`'A'`的无限流。

[methods/exercise-reader.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/methods/exercise-reader.go)

#### 练习：rot13Reader
一种常见的用法是用一个`io.Reader`包装另一个`io.Reader`，并以某种方式修改流。（注：这类似于Java中的组合输入流）

例如，`gzip.NewReader()`函数接受一个`io.Reader`参数（压缩的数据流），返回一个`*gzip.Reader`，该类型也实现了`io.Reader`（解压缩的数据流）。

编写一个类型`rot13Reader`，实现`io.Reader`，从另一个`io.Reader`读取数据，通过对所有字母字符应用[rot13](https://en.wikipedia.org/wiki/ROT13)替换加密来修改流。

也就是说，对于输入流中的字母按以下规则替换：
* 输入：<code><font color="#FF0000">ABCDEFGHIJKLM</font><font color="#0000FF">NOPQRSTUVWXYZ</font><font color="#FF0000">abcdefghijklm</font><font color="#0000FF">nopqrstuvwxyz</font></code>
* 输出：<code><font color="#0000FF">NOPQRSTUVWXYZ</font><font color="#FF0000">ABCDEFGHIJKLM</font><font color="#0000FF">nopqrstuvwxyz</font><font color="#FF0000">abcdefghijklm</font></code>
* 其他字符不变

例如，输入 "Lbh penpxrq gur pbqr!" ，应该输出 "You cracked the code!" 。

[methods/exercise-rot-reader.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/methods/exercise-rot-reader.go)

### 3.5 Image接口
`image`包定义了`Image`接口：

```go
type Image interface {
	ColorModel() color.Model
	Bounds() Rectangle
	At(x, y int) color.Color
}
```

该接口描述了一个矩形像素网格，`Bounds()`方法返回其矩形边界，`At(x, y)`返回坐标(x, y)处像素的颜色。详见文档[image.Image](https://pkg.go.dev/image#Image)。

例如：

```go
m := image.NewRGBA(image.Rect(0, 0, 100, 100))
fmt.Println(m.Bounds())
fmt.Println(m.At(0, 0).RGBA())
```

[methods/images.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/methods/images.go)

#### 练习：Image
实现一个类似于之前的图像生成器（2.3.5节），但这次返回一个`image.Image`接口值，而不是切片。

自定义一个`Image`类型，实现`image.Image`接口：
* `ColorModel()`应该返回`color.RGBAModel`。
* `Bounds()`方法应该返回一个`image.Rectangle`，例如`image.Rect(0, 0, w, h)`。
* `At()`应该返回一个颜色值，上一个练习中的`v`值在这里对应`color.RGBA{v, v, 255, 255}`。

[methods/exercise-images.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/methods/exercise-images.go)

## 4.泛型
<https://go.dev/tour/generics>

### 4.1 类型参数
Go的函数可以使用**类型参数**(type parameter)来处理多种类型，称为**泛型函数**(generic function)。函数的类型参数位于参数列表前的方括号内。例如：

```go
func Index[T comparable](s []T, x T) int
```

这个声明表示`s`是满足内置约束`comparable`的任意类型`T`的切片，`x`也是相同类型的值。

约束`comparable`允许对特定类型的值使用`==`和`!=`运算符。在这个例子中，我们使用它来将一个值与切片的所有元素进行比较，直到找到匹配项。

```go
// Index returns the index of x in s, or -1 if not found.
func Index[T comparable](s []T, x T) int {
	for i, v := range s {
		// v and x are type T, which has the comparable
		// constraint, so we can use == here.
		if v == x {
			return i
		}
	}
	return -1
}
```

这个`Index()`函数适用于任何支持比较的类型。

```go
// Index works on a slice of ints
si := []int{10, 20, 15, -10}
fmt.Println(Index(si, 15))

// Index also works on a slice of strings
ss := []string{"foo", "bar", "baz"}
fmt.Println(Index(ss, "hello"))
```

[generics/index.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/generics/index.go)

### 4.2 泛型类型
除了泛型函数，Go还支持**泛型类型**(generic type)。类型也可以带有类型参数，这对于实现泛型数据结构非常有用。

下面的例子展示了一个可以存储任何类型值的单向链表的类型声明。

```go
type Node[T any] struct {
	next *Node[T]
	val  T
}

// List represents a singly-linked list that holds values of any type.
type List[T any] struct {
	head *Node[T]
	size int
}
```

包含方法的完整实现：

[generics/list.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/generics/list.go)

## 5.并发
<https://go.dev/tour/concurrency>

### 5.1 Goroutine
**goroutine**是由Go运行时管理的轻量级线程。

```go
go f(x)
```

会启动一个新的goroutine来运行函数`f(x)`，参数的求值发生在当前goroutine中，`f`的执行发生在新的goroutine中。

goroutine在相同的地址空间中运行，因此对共享内存的访问必须要同步。`sync`包提供了一些有用的原语。

[concurrency/goroutines.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/concurrency/goroutines.go)

示例程序启动了两个goroutine（其中一个是主goroutine），交替打印 "hello" 和 "world" 。

### 5.2 channel
**channel**是一种管道，可以使用运算符`<-`通过它发送和接收值。

```go
ch <- v    // Send v to channel ch.
v := <-ch  // Receive from ch, and assign value to v.
```

（数据沿着箭头方向流动）

与映射和切片一样，channel也必须使用`make()`函数创建：

```go
ch := make(chan int)
```

默认情况下，发送和接收操作会阻塞，直到另一端准备就绪。这使得goroutine之间无需显式的锁或条件变量即可同步。

示例代码使用两个goroutine分别对切片前半部分和后半部分的数字求和。一旦两个goroutine都完成计算，就会计算最终结果。

[concurrency/channels.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/concurrency/channels.go)

#### 缓冲channel
channel可以带缓冲(buffered)。为了创建缓冲channel，需要将缓冲大小作为`make()`的第二个参数：

```go
ch := make(chan int, 100)
```

`cap(ch)`返回channel的缓冲长度。

对于缓冲channel，发送操作只有当缓冲满时才会阻塞，接收操作只有当缓冲为空时才会阻塞。

[concurrency/buffered-channels.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/concurrency/buffered-channels.go)

在上面的例子中，如果将`ch`的容量改为1，或者在第一个打印语句之前再加一个`ch <- 3`，或者在最后再增加一个打印语句`fmt.Println(<-ch)`，则程序会陷入死锁：

```
fatal error: all goroutines are asleep - deadlock!
```

#### range和close
发送者可以使用`close()`关闭一个channel，表示没有更多的值要发送。

接收者可以使用返回两个值的接收表达式来检查channel是否已经被关闭：

```go
v, ok := <-ch
```

如果没有可接收的值并且channel已关闭，则`ok`为`false`。

循环`for i := range c`不断地从channel接收值，直到它被关闭。

注意：
* 只有发送者应该关闭channel，接收者绝不能关闭。向已关闭的channel发送值会导致panic。
* channel不像文件，通常不需要关闭。除非必须要告诉接收者没有更多的值，例如终止`range`循环。

[concurrency/range-and-close.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/concurrency/range-and-close.go)

### 5.3 select语句
`select`语句允许goroutine等待多个通信操作。

`select`语句会阻塞直到某个case可以执行，然后执行该分支。如果有多个case准备就绪，则随机选择一个。

例如，假设`c`和`quit`是两个channel，下面的语句当`c`可写入时执行第一个分支，当`quit`可读取时执行第二个分支。

```go
select {
case c <- x:
	x, y = y, x+y
case <-quit:
	fmt.Println("quit")
	return
}
```

[concurrency/select.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/concurrency/select.go)

注：
* 与`switch`语句不同，`select`的各个case不是从上到下判断的。
* Go的`select`语句类似于Python的`select`模块。

如果`select`语句中没有其他分支准备就绪，则执行`default`分支。

可以使用`default`分支尝试发送或接收操作，而不会阻塞：

```go
select {
case i := <-c:
	// use i
default:
	// receiving from c would block
}
```

[concurrency/default-selection.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/concurrency/default-selection.go)

#### 练习：等价二叉树
可能有许多不同的二叉树存储相同的值序列。例如，以下是两棵存储序列1, 1, 2, 3, 5, 8, 13的二叉树。

![tree](/assets/images/a-tour-of-go/tree.png)

在这个练习中，将利用Go的并发和channel机制来编写一个检查两棵二叉树是否存储相同序列的函数。

二叉树类型定义如下：

```go
type Tree struct {
	Left  *Tree
	Value int
	Right *Tree
}
```

1.实现`Walk()`函数，用于对二叉树进行中序遍历，并将结果发送到channel。

```go
func Walk(t *tree.Tree, ch chan int)
```

2.测试`Walk()`函数。使用`tree.New(k)`创建一个随机结构但有序的二叉树，存储值k, 2k, ..., 10k。创建一个channel `ch`并启动一个goroutine来执行`Walk()`：

```go
go Walk(tree.New(1), ch)
```

从`ch`读取并打印10个值，结果应该为1, 2, ..., 10。

3.实现`Same()`函数，使用`Walk()`来确定`t1`和`t2`是否存储相同的值（即判断两棵二叉树的中序遍历结果是否相同）。

```go
func Same(t1, t2 *tree.Tree) bool
```

4.测试`Same()`函数。`Same(tree.New(1), tree.New(1))`应该返回`true`，`Same(tree.New(1), tree.New(2))`应该返回`false`。

关于`tree`包的细节参见[文档](https://pkg.go.dev/golang.org/x/tour/tree)。

[concurrency/exercise-equivalent-binary-trees.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/concurrency/exercise-equivalent-binary-trees.go)

### 5.4 sync.Mutex
使用channel进行goroutine间通信非常方便。但如果我们不需要通信，只是想确保同一时刻只有一个goroutine可以访问某个变量，以避免冲突呢？

这种概念称为**互斥**(mutual exclusion)，提供这种特性的数据结构的通常称为**互斥锁**(mutex)。

Go标准库提供了`sync.Mutex`类型，有`Lock()`和`Unlock()`两个方法（注：相当于Java中的锁）。

下面的例子实现了一个并发安全的计数器。`map`本身不是并发安全的，要在多个goroutine中访问同一个`map`必须进行同步。

可以通过使用`Lock()`和`Unlock()`包围一个代码块使其成为互斥执行的（即临界区），如`Inc()`方法所示。也可以使用`defer`来确保`Mutex`会被解锁，如`Value()`方法所示。

```go
// SafeCounter is safe to use concurrently.
type SafeCounter struct {
	mu sync.Mutex
	v  map[string]int
}

// Inc increments the counter for the given key.
func (c *SafeCounter) Inc(key string) {
	c.mu.Lock()
	// Lock so only one goroutine at a time can access the map c.v.
	c.v[key]++
	c.mu.Unlock()
}

// Value returns the current value of the counter for the given key.
func (c *SafeCounter) Value(key string) int {
	c.mu.Lock()
	// Lock so only one goroutine at a time can access the map c.v.
	defer c.mu.Unlock()
	return c.v[key]
}
```

[concurrency/mutex-counter.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/concurrency/mutex-counter.go)

### 练习：网络爬虫
在这个练习中，将使用Go的并发特性实现并行网络爬虫。

实现`Crawl()`函数，并行爬取URL，但不要爬取重复的URL。

注：
* `fakeResult`表示页面内容及其包含的URL。
* `fakeFetcher`是一个预定义的URL到`fakeResult`的映射。其`Fetch()`方法在映射中查找给定的URL，用于模拟页面爬取过程，如果URL不存在则返回一个错误。

[concurrency/exercise-web-crawler.go](https://github.com/ZZy979/go-tutorials/blob/main/tour/concurrency/exercise-web-crawler.go)
