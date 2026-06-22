---
title: 《Go程序设计语言》笔记 第2章 程序结构
date: 2026-05-23 21:33:47 +0800
categories: [Go, GOPL]
tags: [go, declaration, function, variable, pointer, assignment, data type, type conversion, package, import statement, initialization, scope, block]
---
和其他编程语言一样，在Go中，你从一小部分基本结构来构建大型程序。变量存储值。简单表达式通过加法和减法等操作组合成复杂表达式。基本类型被收集到数组和结构体等聚合类型中。表达式用于语句中，语句的执行顺序由`if`和`for`等控制流语句决定。语句组成函数以实现隔离和重用。函数收集到源文件和包中。

在本章中，我们将更详细地介绍Go程序的基本结构元素。

## 2.1 命名
Go语言函数、变量、常量、类型、语句标签和包的命名都遵循一个简单的规则：名字以字母或下划线开头，后面可以跟任意数量的字母、数字和下划线，区分大小写。

Go有25个**关键字**(keyword)（如`if`和`switch`），只能在特定的语法中使用，不能用作名字。

```go
break     default      func    interface  select
case      defer        go      map        struct
chan      else         goto    package    switch
const     fallthrough  if      range      type
continue  for          import  return     var
```

此外，还有30多个**预定义名字**（如`int`和`true`），用于内置常量、类型和函数。

```go
常量：
    true false iota nil
类型：
    int int8 int16 int32 int64
    uint uint8 uint16 uint32 uint64 uintptr
    float32 float64 complex128 complex64
    bool byte rune string error
函数：
    make len cap new append copy close delete
    complex real imag
    panic recover
```

这些名字不是保留的，因此可以在声明中使用，但要注意可能造成的混淆。

如果一个实体在函数内部声明，则它仅在该函数内可见；如果在函数外部声明，则对其所属包的所有文件都可见。名字首字母的大小写决定了它在包外部的可见性：如果名字以大写字母开头，则它是**导出的**(exported)，对外部包可见（例如`fmt.Printf`）；否则仅对当前包可见。包名本身总是小写的。（注：中文被视为小写字母，这一规则可能会在Go 2.0中改变，详见[issue#5763](https://github.com/golang/go/issues/5763#issuecomment-66081539)）

名字长度没有限制，但Go程序的惯例和风格倾向于使用短名字，特别是对于局部变量（例如循环索引变量名为`i`而不是`theLoopIndex`）。一般来说，名字的作用域越大，就应该越长、越有意义。

命名风格上，Go程序员使用“驼峰命名法”（如`parseRequestLine`）而不是内部下划线（如`parse_request_line`）。对于像ASCII和HTML等首字母缩略词避免使用大小写混合的写法，例如一个函数可能被称为`htmlEscape`、`HTMLEscape`或`escapeHTML`但不会是`escapeHtml`。

## 2.2 声明
**声明**(declaration)命名了一个程序实体并指定其属性。有四种主要的声明：`var`、`const`、`type`和`func`。本章讨论变量和类型，第3章讨论常量，第5章讨论函数。

Go程序存储在一个或多个名字以.go结尾的文件中。每个文件都以一个`package`声明开头，之后是`import`声明，然后是**包级别**的类型、变量、常量和函数声明，顺序可以是任意的（注：函数内部的名字必须先声明后使用）。

例如，这个程序声明了一个常量、一个函数和几个变量：

[gopl.io/ch2/boiling](https://github.com/ZZy979/gopl.io/blob/main/ch2/boiling/main.go)

常量`boilingF`和函数`main()`是包级别声明，而`f`和`c`是函数`main()`的局部变量。包级别的实体对整个包的所有文件都可见，而局部声明只在函数内部或其中一小部分可见。

函数声明由名字、参数列表、可选的结果列表和函数体组成。如果函数没有返回值则省略结果列表。函数的执行从函数体的第一条语句开始，直到遇到`return`语句或到达函数末尾（没有返回值），然后控制权和结果（如果有）返回给调用者。

第5章还会详细讨论函数，因此这里只是简述。下面的`fToC()`函数封装了温度转换逻辑，因此只需定义一次就可以多次使用。`main()`使用不同的值调用了两次`fToC()`函数：

[gopl.io/ch2/ftoc](https://github.com/ZZy979/gopl.io/blob/main/ch2/ftoc/main.go)

## 2.3 变量
`var`声明创建一个特定类型的变量，并设置其初始值。变量声明的一般形式如下：

```go
var name type = expression
```

其中`type`和`= expression`部分可以省略一个，但不能同时省略。如果省略了类型，则由初始化表达式确定。如果省略表达式，则初始值是类型的**零值**(zero value)：对于数字为`0`，对于布尔值为`false`，对于字符串为`""`，对于接口和引用类型（切片、指针、映射、channel、函数）为`nil`。对于像数组或结构体这样的聚合类型，零值的所有元素或字段都是零值。

零值机制可以确保每个变量总是持有一个良好定义的值。在Go中不存在未初始化的变量，这可以简化代码，并确保边界条件的合理行为。例如，

```go
var s string // ""
fmt.Println(s)
```

会打印一个空字符串，而不是导致某种错误或者不可预测的行为。Go程序员通常应该让更复杂类型的零值是有意义的。

可以在单个声明中声明（并初始化）多个变量。如果省略类型，可以声明多个不同类型的变量：

```go
var i, j, k int                 // int, int, int
var b, f, s = true, 2.3, "four" // bool, float64, string
```

包级别的变量在`main()`开始前初始化，局部变量在函数执行到其声明时初始化。

也可以通过调用返回多个值的函数来初始化多个变量：

```go
var f, err = os.Open(name) // os.Open returns a file and an error
```

### 2.3.1 短变量声明
在函数内部，可以使用**短变量声明**(short variable declaration)来声明和初始化局部变量。形式如下：

```go
name := expression
```

变量的类型由表达式决定。下面是`lissajous()`函数（1.4节）中的三个短变量声明：

```go
anim := gif.GIF{LoopCount: nframes}
freq := rand.Float64() * 3.0
t := 0.0
```

由于其简洁性和灵活性，短变量声明被用于大多数局部变量的声明和初始化。`var`声明通常用于需要显式指定类型（与初始化表达式不同），或者稍后赋值的局部变量。包级别的变量必须用`var`声明。

与`var`声明一样，短变量声明也可以声明和初始化多个变量：

```go
i, j := 0, 1
```

但是这种形式只应该在可以提高可读性的地方使用，例如`for`循环的初始化部分。

注意，`:=`是声明，而`=`是赋值。不要将多变量声明与元组赋值（2.4.1节）混淆：

```go
i, j = j, i // swap values of i and j
```

短变量声明也可以用于返回多个值的函数：

```go
f, err := os.Open(name)
```

一个微妙但重要的点是：短变量声明不一定**声明**左侧的所有变量。如果有些变量已经在相同的作用域中声明过，那么短变量声明只会对这些变量进行**赋值**。例如，在下面的代码中，第一个语句声明了`in`和`err`，第二个语句声明了`out`并对已有变量`err`赋值。

```go
in, err := os.Open(infile)
// ...
out, err := os.Create(outfile)
```

短变量声明必须至少声明一个新变量，因此下面的代码会编译失败：

```go
f, err := os.Open(infile)
// ...
f, err := os.Create(outfile) // compile error: no new variables
```

解决方法是将第二个语句改为赋值。

### 2.3.2 指针
**变量**(variable)是一块包含值的存储空间。**指针**(pointer)是变量的**地址**，即值在内存中的存储位置。并不是每个值都有地址，但每个变量都有。通过指针可以**间接地**读取或更新变量的值。

对于一个`int`类型的变量`x`，表达式`&x`（“`x`的地址”）会产生一个`*int`类型的值（“指向`int`的指针”）。假设这个值被称为`p`，则说“`p`指向`x`”，或者“`p`包含`x`的地址”。表达式`*p`产生`p`指向的变量的值，它可以出现在赋值语句的左边，表示更新变量的值。

```go
x := 1
p := &x         // p, of type *int, points to x
fmt.Println(*p) // "1"
*p = 2          // equivalent to x = 2
fmt.Println(x)  // "2"
```

`&`运算符的操作数必须是**可寻址的**(addressable)，包括：变量（如`x`）、数组索引（如`x[i]`）、结构体字段（如`x.f`）、复合字面值（如`Point{3, 4}`）等。

指针类型的零值是`nil`。如果`p`指向某个变量则`p != nil`为真。指针是可比较的，两个指针相等当且仅当它们指向同一个变量或都是`nil`。

```go
var x, y int
fmt.Println(&x == &x, &x == &y, &x == nil) // "true false false"
```

在Go中，**函数返回局部变量的地址是安全的**。例如，在下面的代码中，调用`f()`创建的局部变量`v`即使在函数返回后仍然存在，指针`p`仍然指向它：

```go
func f() *int {
	v := 1
	return &v
}

var p = f()
```

每次调用`f()`都会返回不同的值：

```go
fmt.Println(f() == f()) // "false"
```

注：在C/C++中，保存局部变量地址的指针在函数返回后会变成“悬垂指针”，对其解引用是未定义行为。而在Go中之所以可以这样做，是因为Go编译器会执行**逃逸分析**(escape analysis)：如果发现局部变量的地址被返回并可能在函数外部使用，编译器就会在堆上而不是栈上分配该变量（称该变量“逃逸到堆上”(escape to the heap)），由垃圾收集器负责回收它。参见文档[Where Go Values Live](https://go.dev/doc/gc-guide#Where_Go_Values_Live)。

由于指针包含变量的地址，向函数传递指针参数可以使函数间接更新被指向的变量。例如，函数`incr()`将其参数指向的变量加1并返回新值：

```go
func incr(p *int) int {
	*p++ // increments what p points to; does not change p
	return *p
}

v := 1
incr(&v)              // side effect: v is now 2
fmt.Println(incr(&v)) // "3" (and v is 3)
```

每次获取变量的地址或复制指针时，都会创建相同变量的一个新的别名。例如，在上面的例子中`*p`是`v`的别名。指针别名允许在不使用变量名的情况下访问变量，但这是一把双刃剑：要找到访问一个变量的所有语句，必须知道它的所有别名。创建别名的不仅仅是指针，当我们复制其他引用类型（如切片、映射和channel，以及包含这些类型的结构体、数组和接口）的值时，也会创建别名。

指针是`flag`包的关键，它使用程序的命令行参数来设置变量的值，这些变量可能会分布在整个程序中。为了说明这一点，下面的`echo`程序变体添加了两个可选标志：`-n`使其省略末尾的换行符，`-s sep`使其用字符串`sep`而不是默认的空格分隔输出参数。

[gopl.io/ch2/echo4](https://github.com/ZZy979/gopl.io/blob/main/ch2/echo4/main.go)

函数`flag.Bool()`创建一个新的`bool`标志变量，返回指向它的指针。它有三个参数：标志的名字、默认值以及帮助消息（如果用户提供了无效参数、无效标志或者`-h`或`-help`就会打印）。类似地，`flag.String()`创建带字符串参数的标志变量。

在使用标志之前，必须先调用`flag.Parse()`解析命令行参数并更新标志变量的值。非标志参数可以通过`flag.Args()`获得。如果解析遇到错误，会打印帮助消息并调用`os.Exit(2)`终止程序。

下面运行一些测试用例：

```shell
$ go build gopl.io/ch2/echo4
$ echo4 a bc def
a bc def
$ echo4 -s / a bc def
a/bc/def
$ echo4 -n a bc def
a bc def$ 
$ echo4 -help
Usage of echo4:
  -n    omit trailing newline
  -s string
        separator (default " ")
```

### 2.3.3 new函数
创建变量的另一种方式是使用内置函数`new`。表达式`new(T)`创建了一个`T`类型的**匿名变量**，将其初始化为`T`的零值，并返回其地址（即`*T`类型的值）。

```go
p := new(int)   // p, of type *int, points to an unnamed int variable
fmt.Println(*p) // "0"
*p = 2          // sets the unnamed int to 2
fmt.Println(*p) // "2"
```

使用`new`创建的变量除了没有名字外与普通局部变量没有什么不同。因此`new`只是一种语法糖，而不是一个新的基本概念。下面的两个`newInt()`函数具有相同的行为：

```go
func newInt() *int {
	return new(int)
}

func newInt() *int {
	var dummy int
	return &dummy
}
```

注：
* `new()`函数的参数也可以是一个表达式，用于指定变量的初始值，例如`new(123)`。
* 不同于C++的`new`运算符，Go的`new()`函数创建的变量不一定分配在堆上。

`new()`函数用得相对较少，因为最常见的匿名变量是结构体类型，而对于这些类型来说结构体字面值语法（4.4.1节）更灵活。

`new`不是关键字，因此可以在函数中使用这个名字，例如：

```go
func delta(old, new int) int {
	return new - old
}
```

当然，在`delta()`函数内部无法使用内置函数`new()`。

### 2.3.4 变量的生存期
变量的**生存期**(lifetime)是程序执行期间变量存在的时间间隔。包级别变量的生存期是程序的整个执行过程。而局部变量具有动态生存期：每次执行声明语句都会创建一个新实例，直到变量**不可达**（即没有任何引用），此时其存储可能会被回收。函数参数和结果也是局部变量，它们在函数每次被调用时创建。

例如，在1.4节`lissajous`程序的以下片段中：

```go
for t := 0.0; t < cycles*2*math.Pi; t += res {
	x := math.Sin(t)
	y := math.Sin(t*freq + phase)
	img.SetColorIndex(size+int(x*size+0.5), size+int(y*size+0.5), blackIndex)
}
```

变量`t`在`for`循环开始时创建，变量`x`和`y`在循环的每次迭代中创建。

变量的生存期只取决于它是否可达，因此局部变量可能会在函数返回后继续存在（例如2.3.2节中返回局部变量地址的例子）。

编译器会自动选择在栈上还是堆上分配局部变量，但这并不取决于变量是用`var`还是`new`声明的（如2.3.3节所述，这两种方式本质上没有任何不同）。例如：

```go
var global *int

func f() {
	var x int // heap-allocated
	x = 1
	global = &x
}

func g() {
	y := new(int) // stack-allocated
	*y = 1
}
```

在这里，`x`必须在堆上分配（由垃圾收集器负责回收），因为它在`f()`返回后仍然通过`global`可达，我们称`x`从`f`中**逃逸**(escape from)。相反，当`g()`返回后，变量`*y`就不可达了，因此编译器可以在栈上分配`*y`。逃逸并不会导致内存泄露，但需要额外分配内存，因此在性能优化时需要考虑。

## 2.4 赋值
使用赋值语句更新一个变量的值，最简单的形式是`name = expression`。

```go
x = 1                       // named variable
*p = true                   // indirect variable
person.name = "bob"         // struct field
count[x] = count[x] * scale // array or slice or map element
```

每个二元算术运算符都有对应的**赋值运算符**(assignment operator)。例如，上面最后一个语句可以写为

```go
count[x] *= scale
```

这可以省去对表达式的重复书写和求值。

数值变量也可以使用`++`和`--`语句递增和递减：

```go
v := 1
v++     // same as v = v + 1; v becomes 2
v--     // same as v = v - 1; v becomes 1 again
```

注：`v++`不是表达式，因此`x = v++`是错误的。

### 2.4.1 元组赋值
**元组赋值**(tuple assignment)允许同时给多个变量赋值（注：类似于Python的序列解包）。所有右侧表达式都会在更新左侧变量之前求值，这对于同时出现在两边的变量很有用。例如，交换两个变量的值：

```go
x, y = y, x
a[i], a[j] = a[j], a[i]
```

或是计算两个整数的最大公约数(GCD)：

```go
func gcd(x, y int) int {
	for y != 0 {
		x, y = y, x%y
	}
	return x
}
```

或是计算第n个斐波那契数：

```go
func fib(n int) int {
	x, y := 0, 1
	for i := 0; i < n; i++ {
		x, y = y, x+y
	}
	return x
}
```

元组赋值也可以使一系列简单赋值更加紧凑：

```go
i, j, k = 2, 3, 5
```

但是如果表达式太复杂，应该避免使用元组赋值，一系列单独赋值语句的可读性会更好。

有些表达式会产生多个值，例如调用返回多个结果的函数。在赋值语句中使用这种表达式时，左边变量的个数必须和值的个数一致（注：并且右边不能有其他表达式）。

```go
f, err = os.Open("foo.txt")  // function call returns two values
```

通常，函数会用额外的返回值表示某种类型的错误，通过返回一个`error`或者`bool`（通常称为`ok`）。在后面的章节将会看到，有三种运算符也采用了这种方式：映射查找（4.3节）、类型断言（7.10节）和channel接收（8.4.2节）：

```go
v, ok = m[key]  // map lookup
v, ok = x.(T)   // type assertion
v, ok = <-ch    // channel receive
```

和变量声明一样，可以将不需要的值赋给空标识符(`_`)：

```go
_, err = io.Copy(dst, src)  // discard byte count
_, ok = x.(T)               // check type but discard result
```

### 2.4.2 可赋值性
赋值语句是一种显式赋值，但是程序中还有许多地方会发生**隐式**赋值：函数调用将实参值隐式赋给相应的形参变量；`return`语句将返回值隐式赋给结果变量；复合类型的字面值表达式（例如切片`medals := []string{"gold", "silver", "bronze"}`）会隐式地给每个元素赋值。映射和channel也有类似的隐式赋值。

一般地，只有当右侧（值）**可赋值给**左侧（变量）的类型时，赋值才是合法的。

**可赋值性**(assignability)规则对于不同类型有不同的要求，我们将在介绍每种新类型时进行解释。对于到目前为止讨论过的类型，规则很简单：类型必须完全匹配（注：Go没有任何隐式类型转换），`nil`可以赋值给任何指针、接口或引用类型的变量。常量（3.6节）有更灵活的可赋值性规则，可以避免大部分显式类型转换。

注：完整规则参见文档[Assignability](https://go.dev/ref/spec#Assignability)。

两个值是否可以用`==`和`!=`比较与可赋值性有关：在任何比较中，第一个操作数必须可赋值给第二个操作数的类型，反之亦然。与可赋值性一样，我们将在介绍每种新类型时解释**可比较性**(comparability)规则。

## 2.5 类型声明
变量的类型定义了它可存储的值的特征，例如大小、内部表示、支持的操作和方法等。在程序中会有一些底层表示相同、但表达完全不同概念的变量。例如，`int`可用于表示循环索引、时间戳、文件描述符或者月份；`float64`可能表示速度(m/s)或不同单位的温度；`string`可能表示密码或颜色名称。

`type`声明定义了一个新的**命名类型**(named type)，与某种现有类型具有相同的**底层类型**(underlying type)，语法如下。命名类型提供了一种方式来分隔底层类型的不同用法。

```go
type name underlying-type
```

类型声明通常出现在包级别，这样命名类型在整个包中可见。如果名字是导出的（以大写字母开头），则其他包也可以访问。

为了说明类型声明，下面将不同的温度单位定义为不同的类型：

[gopl.io/ch2/tempconv0](https://github.com/ZZy979/gopl.io/blob/main/ch2/tempconv0/celsius.go)

这个包定义了两种类型`Celsius`和`Fahrenheit`，分别表示摄氏温度和华氏温度。尽管二者具有相同的底层类型`float64`，但它们是**不同的类型**，因此不能在算术表达式中比较或组合。区分类型可以避免无意中混合不同单位的温度的错误（例如相加或赋值）。需要使用**显式类型转换** `Celsius(t)`或`Fahrenheit(t)`将`float64`转换为对应的类型。这两个转换不会改变值本身，但会明确指出含义的改变。

注：将`float64`变量赋给`Celsius`变量需要显式类型转换，但浮点数常量不需要。

对于每种类型`T`，都有相应的转换操作`T(x)`，将值`x`转换为类型`T`。如果二者具有相同的底层类型，或二者都是指向相同底层类型的指针类型，则允许从一种类型转换为另一种类型；这些转换只改变类型，不改变值的表示。如果`x`可赋值给`T`，则允许转换，但通常是多余的。

数值类型之间、以及字符串和某些切片类型之间也允许转换。这些转换可能会改变值的表示。例如，将浮点数转换为整数会丢弃小数部分，将字符串转换为`[]byte`切片会创建字符串数据的副本。在任何情况下，转换都不会在运行时失败（注：即只要编译通过就不会有问题）。

命名类型的底层类型决定了其表示方式，以及支持的内置运算符（与直接使用底层类型相同）。例如，`float64`的算术运算符同样适用于`Celsius`和`Fahrenheit`。

```go
fmt.Printf("%g\n", BoilingC-FreezingC) // "100" (°C)
boilingF := CToF(BoilingC)
fmt.Printf("%g\n", boilingF-CToF(FreezingC)) // "180" (°F)
fmt.Printf("%g\n", boilingF-FreezingC)       // compile error: type mismatch
```

比较运算符（如`==`和`<`）可用于比较一个命名类型的值和另一个相同类型的值，或者和一个底层类型（非命名类型）的值（如`Celsius`和`float64`）。但是两个不同命名类型的值（如`Celsius`和`Fahrenheit`）不能直接比较。

```go
var c Celsius
var f Fahrenheit
fmt.Println(c == 0)          // "true"
fmt.Println(f >= 0)          // "true"
fmt.Println(c == f)          // compile error: type mismatch
fmt.Println(c == Celsius(f)) // "true"
```

命名类型还可以定义新的行为。这些行为表示为一组与该类型关联的函数，称为**方法**。第6章将详细讨论方法，这里只做简单介绍。在下面的声明中，`Celsius`类型的参数`c`出现在函数名之前，将一个名为`String`的方法与`Celsius`类型关联，该方法返回`c`的字符串表示：

```go
func (c Celsius) String() string { return fmt.Sprintf("%g°C", c) }
```

许多类型都会声明`String()`方法，用于控制`fmt`包如何将该类型的值打印为字符串（将在7.1节介绍）。（注：类似于Java的`Object.toString()`方法）

```go
c := FToC(212.0)
fmt.Println(c.String()) // "100°C"
fmt.Printf("%v\n", c)   // "100°C"; no need to call String explicitly
fmt.Printf("%s\n", c)   // "100°C"
fmt.Println(c)          // "100°C"
fmt.Printf("%g\n", c)   // "100"; does not call String
fmt.Println(float64(c)) // "100"; does not call String
```

## 2.6 包和文件
Go中的包类似于其他语言中的模块，用于支持模块化、封装、单独编译和复用。包的源代码位于一个或多个.go文件中，通常位于导入路径对应的目录中。例如，`gopl.io/ch1/helloworld`包对应的目录是$GOPATH/src/gopl.io/ch1/helloworld。

每个包都作为一个单独**命名空间**。例如，`image`包和`unicode/utf16`包中的`Decode`是两个不同的函数。要从包外部引用标识符，必须用包名**限定**，例如`image.Decode`或`utf16.Decode`。

包还可以通过控制哪些名字是外部可见的（即**导出的**）来隐藏信息：以大写字母开头的标识符是导出的。

为了演示包的基本用法，假设我们要将温度转换包发布到Go社区。下面创建一个名为`gopl.io/ch2/tempconv`的包，这是前一个示例的变体。为了演示如何访问不同文件中的声明，这个包存储在两个文件中。

我们把类型声明、常量和方法放在tempconv.go文件中：

[gopl.io/ch2/tempconv/tempconv.go](https://github.com/ZZy979/gopl.io/blob/main/ch2/tempconv/tempconv.go)

转换函数放在conv.go文件中：

[gopl.io/ch2/tempconv/conv.go](https://github.com/ZZy979/gopl.io/blob/main/ch2/tempconv/conv.go)

每个文件都以`package`声明开头，用来指定包名。当包被导入后，以`tempconv.CToF`等形式访问其成员。包级别的名字（例如类型、常量和函数）对同一个包的其他文件都是可见的，就好像代码都在一个文件里一样。

由于包级别的常量名以大写字母开头，可以像这样在外部访问：

```go
fmt.Printf("Brrrr! %v\n", tempconv.AbsoluteZeroC) // "Brrrr! -273.15°C"
```

要将摄氏温度转换为华氏温度，需要导入`gopl.io/ch2/tempconv`包并使用以下代码：

```go
import "gopl.io/ch2/tempconv"

fmt.Println(tempconv.CToF(tempconv.BoilingC)) // "212°F"
```

包声明前的**文档注释**(doc comment)（详见10.7.4节）对包的整体功能进行说明。每个包中应该只有一个文件有包文档注释。如果文档注释很多，通常放在一个单独的文件中（习惯上称为doc.go）。

练习2.1 向`tempconv`包添加用于处理开尔文温度的类型、常量和函数。0 K是-273.15 °C，1 K和1 °C的间隔是一样的。

### 2.6.1 导入
在Go中，每个包都由一个唯一字符串标识，称为其**导入路径**(import path)。导入路径是出现在`import`声明中的字符串，如`"gopl.io/ch2/tempconv"`。Go语言规范并没有定义这些字符串的来源或含义，其解释方式取决于工具。使用`go`工具（第10章）时，导入路径表示包的源文件所在目录。

除了导入路径，每个包还有一个**包名**(package name)，即出现在`package`声明中的短名字（不一定唯一）。按照惯例，包名是导入路径的最后一段。例如，`gopl.io/ch2/tempconv`的包名是`tempconv`。

[gopl.io/ch2/cf](https://github.com/ZZy979/gopl.io/blob/main/ch2/cf/main.go)

导入声明将导入的包绑定到一个短名字，用于访问其内容。在上面的例子中，可以使用像`tempconv.CToF`这样的**限定标识符**(qualified identifier)来引用`gopl.io/ch2/tempconv`包中的名字。默认情况下，短名字是包名（在这里是`tempconv`），但导入声明可以指定一个别名以避免冲突（见10.4节）。

程序`cf`将数字参数分别转换为摄氏温度和华氏温度：

```shell
$ go build gopl.io/ch2/cf
$ ./cf 32
32°F = 0°C, 32°C = 89.6°F
$ ./cf 212
212°F = 100°C, 212°C = 413.6°F
$ ./cf -40
-40°F = -40°C, -40°C = -40°F
```

导入一个包但不使用是编译错误。这项检查可以帮助消除无用依赖，尽管在调试期间可能有点麻烦，因为注释掉像`log.Print("got here!")`这样的代码可能会删除`log`包的唯一引用，导致编译器报错，此时需要手动删除不必要的导入。

更好的方法是使用`golang.org/x/tools/cmd/goimports`工具，可以根据需要自动添加或删除导入声明。

练习2.2 编写一个类似于`cf`的通用单位转换程序，从命令行参数或标准输入（如果没有参数）读取数字并将转换其他单位，例如温度（摄氏度和华氏度）、长度（英尺和米）、重量（磅和千克）等。

### 2.6.2 包初始化
包初始化首先按照声明的顺序初始化包级别变量，例外是被依赖的变量会先求值：

```go
var a = b + c   // a initialized third, to 3
var b = f()     // b initialized second, to 2, by calling f
var c = 1       // c initialized first, to 1

func f() int { return c + 1 }
```

如果包有多个源文件，则按照提供给编译器的顺序初始化。`go`工具在调用编译器之前会按文件名对.go文件排序。

有些变量（例如数据表）可能无法使用简单表达式设置初始值。在这种情况下，可以使用`init()`函数机制。每个文件都可以包含任意多个声明如下的函数：

```go
func init() { /* ... */ }
```

这种`init()`函数不能被调用或引用。每个文件中的`init()`函数在程序启动时按照声明的顺序自动执行。

包按照程序中导入的顺序初始化，并先初始化依赖。如果包`p`导入了包`q`，就可以确保在`p`初始化开始前`q`已完成初始化。`main`包最后初始化。通过这种方式可以保证在`main()`函数开始前，所有依赖包都已初始化。

下面的包定义了`PopCount()`函数，用于返回一个`uint64`数字中值为1的二进制位个数（称为**population count**）。它使用`init()`函数为每个可能的8位值预先计算结果表`pc`，这样`PopCount()`函数就不需要64步，只需要8次查表。（这肯定不是最快的算法，但便于说明`init()`函数机制，以及展示预计算结果表这种常用的编程技术）

[gopl.io/ch2/popcount](https://github.com/ZZy979/gopl.io/blob/main/ch2/popcount/main.go)

注意，`init()`中的`range`循环只使用了索引，因此省略了值部分。这个循环也可以写为

```go
for i, _ := range pc
```

在下一节和10.5节还会看到其他使用`init()`函数的例子。

练习2.3 使用循环重写`PopCount()`。比较两个版本的性能。

练习2.4 用移位算法重写`PopCount()`：将参数右移64次，每次测试最右边的位。与查表算法比较性能。

练习2.5 表达式`x&(x-1)`会将`x`最右侧的非零位清零。利用这个性质重写`PopCount()`，并测试性能。

## 2.7 作用域
声明的**作用域**(scope)是指源代码中可以使用声明的名字的范围。

不要把作用域和生存期混为一谈。声明的作用域是一个程序文本区域，是编译时属性；变量的生存期是程序执行期间变量可以被引用的时间范围，是运行时属性。

**语法块**(syntactic block)是用花括号括起来的一系列语句，例如函数体或循环体。语法块内部声明的名字在块外不可见，因此块决定了其内部声明的作用域。可以将块的概念推广到其他未显式地用花括号括起来声明分组，称之为**词法块**(lexical block)。整个源代码的词法块称为**全局块**(universe block)；每个包，每个文件，每个`for`、`if`和`switch`语句，`switch`或`select`语句中的每个`case`以及每个语法块都有一个对应的词法块。

声明所在的词法块决定了其作用域。内置类型、函数和常量（如`int`、`len`和`true`）的声明位于全局块，可以在整个程序中引用。包级别的声明可以在同一个包的任何文件中引用。导入的包（例如`fmt`）是文件级别的声明，只能在当前文件中引用。局部声明只能在当前函数中（或其一部分）引用。

控制流标签（`break`、`continue`和`goto`语句使用）的作用域是所在函数。

一个程序可以包含多个同名声明，只要每个声明都在不同的词法块中。例如，可以声明一个与包级别变量同名的局部变量。或者如2.3.3节所示，声明一个名为`new`的函数参数，即使全局块声明了同名函数。但是不要滥用，重新声明的作用域越大，越可能导致程序难以阅读。

当编译器遇到一个名字引用时会查找其声明，从最内层词法块开始一直到全局块。如果未找到声明，则报告错误“未声明的名字”。如果一个名字在外层块和内层块中都声明了，内层声明会先被找到。在这种情况下，称内层声明**遮蔽**(shadow)或**隐藏**(hide)了外层声明。

```go
func f() {}

var g = "g"

func main() {
	f := "f"
	fmt.Println(f) // "f"; local var f shadows package-level func f
	fmt.Println(g) // "g"; package-level var
	fmt.Println(h) // compile error: undefined: h
}
```

在函数中，词法块可以嵌套任意深度。大多数块是由`if`和`for`等控制流语句创建的。下面的程序有三个不同的名为`x`的变量，每个都出现在不同的词法块中（只是为了说明作用域规则，不是好的编程风格）。

```go
func main() {
	x := "hello!"
	for i := 0; i < len(x); i++ {
		x := x[i]
		if x != '!' {
			x := x + 'A' - 'a'
			fmt.Printf("%c", x) // "HELLO" (one letter per iteration)
		}
	}
}
```

前面提到过，并非所有词法块都对应显式的花括号分隔的语句序列，有些是隐式的。上面的`for`循环创建了两个词法块：循环体的显式块和额外包含初始化子句声明的变量（如`i`）的隐式块。隐式块中声明的变量的作用域是条件、后置语句(`i++`)和循环体。

与`for`循环一样，`if`和`switch`语句除了语句体外也会创建隐式块。下面的代码显示了`x`和`y`的作用域：

```go
if x := f(); x == 0 {
	fmt.Println(x)
} else if y := g(x); x == y {
	fmt.Println(x, y)
} else {
	fmt.Println(x, y)
}
fmt.Println(x, y) // compile error: x and y are not visible here
```

第二个`if`语句嵌套在第一个（的`else`子句）中，因此在第一个语句的初始化部分中声明的变量在第二个语句中可见。`switch`语句也有类似的规则：条件部分和每个`case`分别有一个块。

在包级别，声明的顺序不影响作用域，因此一个声明可以引用它自身或后面的声明，这让我们可以声明递归的或相互递归的类型和函数。但是，常量或变量声明不能引用自身。

在这个程序中：

```go
if f, err := os.Open(fname); err != nil { // compile error: unused: f
	return err
}
f.Stat()    // compile error: undefined f
f.Close()   // compile error: undefined f
```

`f`的作用域只有`if`语句，因此后面的语句无法访问（编译器可能还会报告局部变量`f`没有被使用的错误）。因此需要在`if`语句之前声明`f`：

```go
f, err := os.Open(fname)
if err != nil {
	return err
}
f.Stat()
f.Close()
```

你可能会考虑通过将`Stat()`和`Close()`调用移到`else`块来避免在外部声明`f`和`err`，但在Go中通常的做法是在`if`块中处理错误并返回，这样正常执行路径不需要缩进。

要特别注意短变量声明的作用域。考虑下面的程序，获取当前工作目录并将其保存在包级别变量中。

```go
var cwd string

func init() {
	cwd, err := os.Getwd() // compile error: unused: cwd
	if err != nil {
		log.Fatalf("os.Getwd failed: %v", err)
	}
}
```

由于`cwd`和`err`都没有在`init()`函数的块中声明过，因此`:=`语句会将它们都声明为局部变量（见2.3.1节末尾）。内层的`cwd`会遮蔽外层的，因此这段代码不会按预期更新包级别`cwd`变量。

Go编译器会检测局部变量`cwd`没有被使用并报告错误，但并不严格要求执行此检查。另外，一个微小的更改就会使这个检查失败，例如添加一个引用局部`cwd`的日志语句：

```go
var cwd string

func init() {
	cwd, err := os.Getwd() // NOTE: wrong!
	if err != nil {
		log.Fatalf("os.Getwd failed: %v", err)
	}
	log.Printf("Working directory = %s", cwd)
}
```

全局变量`cwd`仍然未被初始化，而看似正常的日志输出让这个bug更加隐晦。

有许多方式可以处理这个潜在的问题。最直接的方法是通过单独声明`err`变量来避免`:=`声明：

```go
var cwd string

func init() {
	var err error
	cwd, err = os.Getwd()
	if err != nil {
		log.Fatalf("os.Getwd failed: %v", err)
	}
}
```
