---
title: Go语言入门教程
date: 2021-04-03 10:55 +0800
categories: [Go]
tags: [go, hello world, package, module, error handling, slice, map, unit test, generic programming, database, mysql]
render_with_liquid: false
---
## 1.简介
Go语言是由Google开发的开源编程语言，以其简洁高效的语法著称。Go内置并发支持（goroutine和channel），编译速度快，特别适合构建高并发网络服务和云原生应用。

* 网站：<https://go.dev/>（或 <https://golang.google.cn/> ）
* 官方文档：<https://go.dev/doc/>
* 官方教程：<https://go.dev/doc/tutorial/>
* Go Playground: <https://go.dev/play/>

## 2.安装
下载地址：<https://go.dev/dl/>

安装指引：<https://go.dev/doc/install>

对于Windows系统，下载MSI安装包（例如go1.25.6.windows-amd64.msi）。安装程序会自动将安装目录中的bin目录添加到Path环境变量。

对于Linux系统，下载.tar.gz压缩包（例如go1.25.6.linux-amd64.tar.gz）。将其解压到/usr/local目录，之后将/usr/local/go/bin添加PATH环境变量。

```shell
tar -C /usr/local -xzf go1.25.6.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
```

验证安装成功：

```shell
$ go version
go version go1.25.6 windows/amd64
```

Go语言源文件的扩展是.go。在任意目录下创建一个文件hello.go，内容如下：

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello, World!")
}
```

使用`go`命令执行上面的代码，输出结果如下：

```shell
$ go run hello.go
Hello, World!
```

## 3.入门
[Tutorial: Get started with Go](https://go.dev/doc/tutorial/getting-started)

本教程将简要介绍Go语言，包括：
* 编写简单的 "Hello, world" 程序
* 使用`go`命令运行代码
* 调用外部模块的函数

### 3.1 Hello world
1.打开一个命令行窗口并切换到主目录。

2.创建一个helloworld目录。

```shell
mkdir helloworld
cd helloworld
```

3.开启依赖管理。

代码中依赖的其他模块由项目根目录中名为go.mod的文件定义（类似于Python的requirements.txt）。

运行[go mod init](https://go.dev/ref/mod#go-mod-init)命令将创建go.mod文件并开启依赖管理，需要指定模块名称。名称是模块路径（通常是仓库位置，例如`github.com/myname/mymodule`），详见[Managing dependencies](https://go.dev/doc/modules/managing-dependencies#naming_module)。

在本教程中，使用`example/hello`作为模块名称：

```shell
$ go mod init example/hello
go: creating new go.mod: module example/hello
```

4.使用文本编辑器创建一个文件hello.go。

5.将以下代码粘贴到hello.go并保存（注意缩进使用Tab而不是空格）：

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello, World!")
}
```

这段Go代码
* 声明了一个`main`包（包由同一个目录下的所有文件组成）。
* 导入了[fmt](https://pkg.go.dev/fmt/)包，包含格式化文本和控制台打印函数。这个包是[标准库](https://pkg.go.dev/std)的一部分。
* 实现了一个`main()`函数用于向控制台打印消息。当运行`main`包时将默认执行`main()`函数。

6.运行代码。

```shell
$ go run .
Hello, World!
```

[go run](https://pkg.go.dev/cmd/go#hdr-Compile_and_run_Go_program)命令用于编译并运行Go程序。可以使用`go help`命令查看所有命令的列表。

### 3.2 调用外部包
你的代码可以调用由其他人编写的包中的函数。

1.查找外部包
* 访问 <https://pkg.go.dev/> ，搜索 "quote" 。
* 在搜索结果中找到`rsc.io/quote`包，并点击 "Other major versions" 中的[v1](https://pkg.go.dev/rsc.io/quote)。
* Documentation - Index一节列出了可以调用的函数。这里使用`Go()`函数。
* 在页面顶端可以看到`quote`包包含在`rsc.io/quote`模块中。

注：
* pkg.go.dev是一个可以搜索Go包的网站，类似于Python的pypi.org。
* Go的“包”和“模块”的含义与Python不同。在Python中，模块=文件，包=目录；在Go中，模块=项目，包=同一个目录下的所有文件。

2.在代码中导入`rsc.io/quote`包并调用其中的`Go()`函数。

```go
package main

import "fmt"

import "rsc.io/quote"

func main() {
	fmt.Println(quote.Go())
}
```

3.添加模块依赖。

```shell
$ go mod tidy
```

`go mod tidy`命令会自动更新go.mod和go.sum文件（用于认证模块，详见[Authenticating modules](https://go.dev/ref/mod#authenticating)），添加缺失的依赖，删除无用的依赖，并自动下载依赖的模块。

注：
* 模块会被下载到$GOPATH/pkg/mod目录。环境变量GOPATH默认为$HOME/go，可通过`go env`命令查看。
* `go mod tidy`命令可能会报错：

```
go: example/hello imports
        rsc.io/quote: module rsc.io/quote: Get "https://proxy.golang.org/rsc.io/quote/@v/list": dial tcp 142.251.45.145:443: connectex: A connection attempt failed because the connected party did not properly respond after a period of time, or established connection failed because connected host has failed to respond.
```

这是因为默认的代理proxy.golang.org无法访问（由环境变量GOPROXY指定）。解决方法：使用代理服务goproxy.io，只需设置环境变量GOPROXY即可：

```shell
$ go env -w GOPROXY=https://goproxy.io,direct
```

再次尝试即可下载成功：

```shell
$ go mod tidy
go: finding module for package rsc.io/quote
go: downloading rsc.io/quote v1.5.2
go: found rsc.io/quote in rsc.io/quote v1.5.2
go: downloading rsc.io/sampler v1.3.0
go: downloading golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c
```

* 如果使用GoLand，需要开启Go模块集成：打开设置 - Go - Go Modules，勾选Enable Go modules integration。

4.运行代码

```shell
$ go run .
Don't communicate by sharing memory, share memory by communicating.
```

完整代码：[helloworld/hello.go](https://github.com/ZZy979/go-tutorials/blob/main/helloworld/hello.go)

## 4.模块
在本教程中，将创建两个模块。第一个是库，可以被其他库或应用程序导入。第二个是应用程序，将使用第一个模块。

### 4.1 创建模块
[Tutorial: Create a Go module](https://go.dev/doc/tutorial/create-module)

Go的代码组织成**包**，包组织成**模块**。模块定义了运行代码所需的依赖，包括Go版本以及依赖的其他模块。

1.在主目录（或其他任意目录）中创建一个`greetings`模块。

```shell
mkdir greetings
cd greetings
go mod init example.com/greetings
```

2.创建一个文件greetings.go，内容如下：


```go
package greetings

import "fmt"

// Hello returns a greeting for the named person.
func Hello(name string) string {
	// Return a greeting that embeds the name in a message.
	message := fmt.Sprintf("Hi, %v. Welcome!", name)
	return message
}
```

在这段代码中
* 声明了一个`greetings`包。
* 定义了一个`Hello()`函数，用于返回问候语。这个函数接受一个`string`类型的参数`name`，返回一个`string`。在Go中，名字以大写字母开头的函数可以被其他包中的函数调用，这叫做**导出名字**(exported name)。

![function-syntax](/assets/images/go-tutorial/function-syntax.png)

* 在函数`Hello()`中声明了一个变量`message`来存放问候语。运算符`:=`是在一行内声明和初始化变量的快捷方式，等价于：

```go
var message string
message = fmt.Sprintf("Hi, %v. Welcome!", name)
```

* 使用标准库`fmt`包的`Sprintf()`创建问候语消息（类似于C语言的`sprintf()`函数）。第一个参数是格式字符串，使用`name`的值替换`%v`（格式字符串语法参见[Printing](https://pkg.go.dev/fmt#hdr-Printing)）。
* 返回格式化后的问候语。

下一步将从另一个模块调用这个函数。

### 4.2 调用模块
[Call your code from another module](https://go.dev/doc/tutorial/call-module-code)

在本节中，将编写一个可执行程序，并调用`greetings`模块中的函数。

1.在`greetings`所在目录中创建另一个模块`hello`。

```shell
cd ..
mkdir hello
cd hello
go mod init example.com/hello
```

目录结构如下：

```
home/
    greetings/
        go.mod
        greetings.go
    hello/
        go.mod
```

2.在hello目录中创建一个文件hello.go，内容如下：

```go
package main

import (
	"fmt"

	"example.com/greetings"
)

func main() {
	// Get a greeting message and print it.
	message := greetings.Hello("Gladys")
	fmt.Println(message)
}
```

在这段代码中
* 声明了一个`main`包。在Go中，可执行程序的代码必须在`main`包中。
* 导入了两个包：`fmt`和`example.com/greetings`，从而代码可以调用这些包中的函数。
* 调用`greetings`包的`Hello()`函数获取问候语，然后调用`fmt`包的`Println()`函数将其打印到控制台。

3.在本地找到`greetings`模块。

上面的代码导入了`example.com/greetings`模块。默认情况下，Go会从这个URL查找并下载该模块。但目前该模块还未发布，因此需要让`hello`模块在本地文件系统找到`greetings`模块。

为此，使用[go mod edit](https://go.dev/ref/mod#go-mod-edit)命令编辑`hello`模块：

```shell
go mod edit -replace example.com/greetings=../greetings
```

该命令将模块路径`example.com/greetings`重定向到本地目录../greetings。执行该命令后，hello目录中的go.mod文件将增加一个`replace`指令：

```
replace example.com/greetings => ../greetings
```

接着，执行[go mod tidy](https://go.dev/ref/mod#go-mod-tidy)命令，同步`hello`模块的依赖：

```shell
$ go mod tidy
go: found example.com/greetings in example.com/greetings v0.0.0-00010101000000-000000000000
```

执行该命令后，`hello`模块的go.mod文件将再增加一个`require`指令：

```
require example.com/greetings v0.0.0-00010101000000-000000000000
```

模块路径后面的数字是伪版本号。为了引用已发布的模块，go.mod文件应该省略`replace`指令并使用带有真实版本号的`require`指令，例如

```
require example.com/greetings v1.1.0
```

关于版本号详见[Module version numbering](https://go.dev/doc/modules/version-numbers)。

4.运行代码，在hello目录中执行

```shell
$ go run .
Hi, Gladys. Welcome!
```

完整代码：
* [greetings/greetings.go](https://github.com/ZZy979/go-tutorials/blob/54d54709a1e1e29d09dbe51430d2d1f35cc16bab/greetings/greetings.go)
* [hello/hello.go](https://github.com/ZZy979/go-tutorials/blob/54d54709a1e1e29d09dbe51430d2d1f35cc16bab/hello/hello.go)

### 4.3 错误处理
[Return and handle an error](https://go.dev/doc/tutorial/handle-errors)

处理错误是可靠代码的一个基本特征。在本节中，将在`greetings`模块中添加返回错误的代码，并在调用代码中处理错误。

1.修改greetings/greetings.go，如果`name`为空则返回错误：

```go
package greetings

import (
	"errors"
	"fmt"
)

// Hello returns a greeting for the named person.
func Hello(name string) (string, error) {
	// If no name was given, return an error with a message.
	if name == "" {
		return "", errors.New("empty name")
	}

	// If a name was received, return a value that embeds the name in a greeting message.
	message := fmt.Sprintf("Hi, %v. Welcome!", name)
	return message, nil
}
```

在这段代码中
* 修改了函数使其返回两个值：一个`string`和一个`error`。调用者会检查第二个值以查看是否发生了错误。
* 导入了标准库`errors`包，以便使用`errors.New()`函数。
* 添加了一个`if`语句以检查参数，如果参数无效（`name`为空字符串）则返回一个错误。`errors.New()`函数返回一个具有给定错误消息的`error`。
* 成功返回时第二个值为`nil`（表示没有错误），这样调用者就能知道函数成功了。

2.修改hello/hello.go，处理`Hello()`函数返回的错误：

```go
package main

import (
	"fmt"
	"log"

	"example.com/greetings"
)

func main() {
	// Set properties of the predefined Logger, including
	// the log entry prefix and a flag to disable printing
	// the time, source file, and line number.
	log.SetPrefix("greetings: ")
	log.SetFlags(0)

	// Request a greeting message.
	message, err := greetings.Hello("")
	// If an error was returned, print it to the console and exit the program.
	if err != nil {
		log.Fatal(err)
	}

	// If no error was returned, print the returned message to the console.
	fmt.Println(message)
}
```

在这段代码中
* 配置了`log`模块打印日志消息的前缀(`"greetings: "`)。
* 将`Hello()`的两个返回值分别赋给变量。
* 将调用`Hello()`的参数改为空字符串，以便测试错误处理代码。
* 如果错误不是`nil`，使用标准库`log`包的`Fatal()`函数打印错误信息并终止程序。

3.在hello目录中运行代码：

```shell
$ go run .
greetings: empty name
exit status 1
```

这就是Go中常见的错误处理方式：**将错误作为返回值，由调用者检查**（Go没有异常和`try`语句）。

完整代码：
* [greetings/greetings.go](https://github.com/ZZy979/go-tutorials/blob/c95cb8184d3923d3e12f9728a12fd7669cab127b/greetings/greetings.go)
* [hello/hello.go](https://github.com/ZZy979/go-tutorials/blob/c95cb8184d3923d3e12f9728a12fd7669cab127b/hello/hello.go)

### 4.4 返回随机问候语——使用切片
[Return a random greeting](https://go.dev/doc/tutorial/random-greeting)

在本节中将修改代码，返回几个预定义问候语中的随机一个，而不是每次返回固定的问候语。为此，要使用Go的**切片**(slice)。

切片类似于数组，但大小会随着添加和删除元素动态变化。关于切片详见[Go slices](https://go.dev/blog/slices-intro)。（注：Go的切片相当于Python的`list`，与Python的切片不是一回事）

1.修改greetings/greetings.go，添加一个包含三条问候语消息的切片，然后随机返回其中一条。

```go
package greetings

import (
	"errors"
	"fmt"
	"math/rand"
)

// Hello returns a greeting for the named person.
func Hello(name string) (string, error) {
	// If no name was given, return an error with a message.
	if name == "" {
		return "", errors.New("empty name")
	}

	// Create a message using a random format.
	message := fmt.Sprintf(randomFormat(), name)
	return message, nil
}

// randomFormat returns one of a set of greeting messages. The returned
// message is selected at random.
func randomFormat() string {
	// A slice of message formats.
	formats := []string{
		"Hi, %v. Welcome!",
		"Great to see you, %v!",
		"Hail, %v! Well met!",
	}

	// Return a randomly selected message format by specifying
	// a random index for the slice of formats.
	return formats[rand.Intn(len(formats))]
}
```

在这段代码中
* 添加了一个`randomFormat()`函数，返回随机选择的问候语格式字符串。注意，以小写字母开头的函数只能在同一个包中访问（即非导出的）。
* 在`randomFormat()`中，声明了一个切片`formats`，包含三个格式字符串。在声明切片时，省略方括号中的大小，例如`[]string`。这意味着底层数组的大小可以动态改变。
* 使用[math/rand](https://pkg.go.dev/math/rand)包中的`Intn()`函数生成一个[0, 3)之间的随机数，用于从切片中选择一项。
* 在`Hello()`中，将固定问候语改为调用`randomFormat()`。

2.将hello/hello.go中调用`Hello()`函数的参数恢复为非空值。

```go
message, err := greetings.Hello("Gladys")
```

3.在hello目录中运行代码。多次运行，注意到问候语会变化。

```shell
$ go run .
Great to see you, Gladys!

$ go run .
Hi, Gladys. Welcome!

$ go run .
Hail, Gladys! Well met!
```

完整代码：
* [greetings/greetings.go](https://github.com/ZZy979/go-tutorials/blob/f93e5028e17b0a4762ba808867a27f8e7e11287b/greetings/greetings.go)
* [hello/hello.go](https://github.com/ZZy979/go-tutorials/blob/f93e5028e17b0a4762ba808867a27f8e7e11287b/hello/hello.go)

### 4.5 为多个人返回问候语——使用映射
[Return greetings for multiple people](https://go.dev/doc/tutorial/greetings-multiple-people)

在本节中，将支持为多个人分别返回问候语。换句话说，向函数传递一组名字（使用切片），返回每个人对应的问候语。为此，需要使用**映射**(map)。

但是，直接修改函数`Hello()`的参数类型会改变函数的签名。如果你已经发布了`example.com/greetings`模块并且其他人已经编写了调用`Hello()`的代码，这种改变将破坏他们的程序。在这种情况下，更好的选择是编写一个具有不同名称的新函数，以保持旧函数[向后兼容](https://go.dev/blog/module-compatibility)。

1.修改greetings/greetings.go，添加一个函数`Hellos()`。

```go
// Hellos returns a map that associates each of the named people with a greeting message.
func Hellos(names []string) (map[string]string, error) {
	// A map to associate names with messages.
	messages := make(map[string]string)
	// Loop through the received slice of names, calling
	// the Hello function to get a message for each name.
	for _, name := range names {
		message, err := Hello(name)
		if err != nil {
			return nil, err
		}
		// In the map, associate the retrieved message with the name.
		messages[name] = message
	}
	return messages, nil
}
```

在这段代码中
* 添加了一个`Hellos()`函数，其参数是一个切片而不是单个`string`，返回类型从`string`改为`map[string]string`，从而可以返回名字到问候语的映射。
* 让新的`Hellos()`函数调用已有的`Hello()`函数，这有助于减少重复。
* 创建了一个映射`messages`，将每个名字（作为键）关联到生成的问候语（作为值）。在Go中，使用语法`make(map[KeyType]ValueType)`初始化一个映射。关于映射详见[Go maps in action](https://go.dev/blog/maps)。
* 遍历函数接收的`names`切片，检查每一个都非空，然后将每个名字关联一条消息。在这个`for`循环中，`range`返回两个值：当前项的索引和值的拷贝。这里不需要索引，因此使用[空白标识符](https://go.dev/doc/effective_go#blank)(`_`)忽略它。（注：Go的关键字`range`类似于Python的`enumerate()`函数）

2.在hello/hello.go中调用`Hellos()`函数，传递一个名字的切片，然后打印返回的映射。

```go
// A slice of names.
names := []string{"Gladys", "Samantha", "Darrin"}

// Request greeting messages for the names.
messages, err := greetings.Hellos(names)
// If an error was returned, print it to the console and exit the program.
if err != nil {
	log.Fatal(err)
}

// If no error was returned, print the returned map of messages to the console.
fmt.Println(messages)
```

3.运行代码，将输出名字/消息映射的字符串表示。

```shell
$ go run .
map[Darrin:Hail, Darrin! Well met! Gladys:Hi, Gladys. Welcome! Samantha:Hail, Samantha! Well met!]
```

完整代码：
* [greetings/greetings.go](https://github.com/ZZy979/go-tutorials/blob/f602aaa9cbb79df9db5b89867d84228837b1e9f6/greetings/greetings.go)
* [hello/hello.go](https://github.com/ZZy979/go-tutorials/blob/f602aaa9cbb79df9db5b89867d84228837b1e9f6/hello/hello.go)

### 4.6 单元测试
[Add a test](https://go.dev/doc/tutorial/add-a-test)

本节将为`Hello()`函数添加测试。Go提供了对单元测试的内置支持：使用命名约定、[testing](https://pkg.go.dev/testing)包和[go test](https://pkg.go.dev/cmd/go#hdr-Test_packages)命令可以快速编写和执行测试。

1.在greetings目录中，创建一个名为greetings_test.go的文件。以 **_test.go** 结尾的文件名告诉`go test`命令这个文件包含测试函数。

2.文件greetings_test.go的内容如下：

```go
package greetings

import (
	"regexp"
	"testing"
)

// TestHelloName calls greetings.Hello with a name, checking for a valid return value.
func TestHelloName(t *testing.T) {
	name := "Gladys"
	want := regexp.MustCompile(`\b` + name + `\b`)
	msg, err := Hello(name)
	if !want.MatchString(msg) || err != nil {
		t.Errorf(`Hello("Gladys") = %q, %v, want match for %#q, nil`, msg, err, want)
	}
}

// TestHelloEmpty calls greetings.Hello with an empty string, checking for an error.
func TestHelloEmpty(t *testing.T) {
	msg, err := Hello("")
	if msg != "" || err == nil {
		t.Errorf(`Hello("") = %q, %v, want "", error`, msg, err)
	}
}
```

在这段代码中
* 创建了两个测试函数来测试`greetings.Hello()`，与被测函数在同一个包中。
  * `TestHelloName()`使用一个名字调用`Hello()`，应该返回合法的消息。如果函数返回了一个错误或者不符合预期的消息（不包含传入的名字），则测试失败。
  * `TestHelloEmpty()`用一个空字符串调用`Hello()`，应该返回错误。如果函数返回了非空字符串或者未返回错误，则测试失败。
* 测试函数名的形式为`TestName`，其中`Name`表示特定测试的信息（如被测函数、测试点）。测试函数接受一个指向[testing.T](https://pkg.go.dev/testing#T)类型的指针参数，可以使用这个参数的方法报告错误和输出日志。

3.在greetings目录中，使用`go test`命令执行测试。可以添加`-v`标志获得详细输出，这将列出所有测试及其结果。

```shell
$ go test 
PASS
ok      example.com/greetings   0.038s

$ go test -v
=== RUN   TestHelloName
--- PASS: TestHelloName (0.00s)
=== RUN   TestHelloEmpty
--- PASS: TestHelloEmpty (0.00s)
PASS
ok      example.com/greetings   0.036s
```

4.使测试失败。

测试函数`TestHelloName()`检查`Hello()`的返回值包含参数指定的名字。为了查看失败的测试结果，修改`Hello()`函数（假设出现了错误）：

```go
// message := fmt.Sprintf(randomFormat(), name)
message := fmt.Sprint(randomFormat())
```

5.再次运行测试，`TestHelloName()`应该会失败。

```shell
$ go test   
--- FAIL: TestHelloName (0.00s)
    greetings_test.go:14: Hello("Gladys") = "Hail, %v! Well met!", <nil>, want match for `\bGladys\b`, nil
FAIL
exit status 1
FAIL    example.com/greetings   0.035s
```

完整代码：[greetings/greetings_test.go](https://github.com/ZZy979/go-tutorials/blob/bfefe2b35aac7bd8ffa19352b421ecbec1cba929/greetings/greetings_test.go)

### 4.7 编译和安装应用
[Compile and install the application](https://go.dev/doc/tutorial/compile-install)

`go run`命令是编译和运行程序的快捷方式，但它不会保留可执行文件（生成在临时目录中，运行后自动删除）。本节将介绍另外两个命令：
* [go build](https://pkg.go.dev/cmd/go#hdr-Compile_packages_and_dependencies)命令编译包及其依赖。
* [go install](https://go.dev/ref/mod#go-install)命令编译包及其依赖并安装。

1.打开命令行，在hello目录中执行`go build`命令，将代码编译成可执行文件。

2.运行生成的可执行文件hello（或hello.exe）。

在Linux或Mac上：

```shell
$ ./hello
map[Darrin:Great to see you, Darrin! Gladys:Hail, Gladys! Well met! Samantha:Hail, Samantha! Well met!]
```

在Windows上：

```shell
$ hello.exe
map[Darrin:Great to see you, Darrin! Gladys:Hail, Gladys! Well met! Samantha:Hail, Samantha! Well met!]
```

现在已经把应用程序编译为可执行文件，但是命令行需要位于可执行文件所在目录，或者指定完整路径才能运行它。接下来将安装可执行文件，以便不指定路径（即在任意目录中直接输入`hello`）即可运行。

3.使用[go list](https://pkg.go.dev/cmd/go#hdr-List_packages_or_modules)命令查看安装路径。

```shell
go list -f '{{.Target}}'
```

例如，该命令可能输出 /home/gopher/go/bin/hello ，表示安装路径为 /home/gopher/go/bin 。

注：安装路径由环境变量GOBIN指定，默认为$GOPATH/bin。

4.将安装路径添加到PATH环境变量，这样无需指定文件位置即可运行程序。

在Linux或Mac上：

```shell
export PATH=$PATH:/home/gopher/go/bin
```

在Windows上：

```shell
set PATH=%PATH%;C:\Users\gopher\go\bin
```

5.运行`go install`命令编译并安装`hello`包。

```shell
go install
```

6.打开一个新的命令行窗口，在其他目录中直接输入`hello`即可运行程序。

```shell
$ hello
map[Darrin:Hail, Darrin! Well met! Gladys:Great to see you, Gladys! Samantha:Hail, Samantha! Well met!]
```

## 5.泛型
[Tutorial: Getting started with generics](https://go.dev/doc/tutorial/generics)

本教程将介绍Go中**泛型**(generics)的基础知识。使用泛型可以编写能够用于多种类型的函数或类型。

注：泛型是[Go 1.18](https://go.dev/doc/go1.18)中引入的。

### 5.1 创建目录
在主目录中创建一个`generics`模块。

```shell
mkdir generics
cd generics
go mod init example/generics
```

### 5.2 添加非泛型函数
1.创建一个文件main.go，内容如下：

```go
package main

import "fmt"

func main() {
	// Initialize a map for the integer values
	ints := map[string]int64{
		"first":  34,
		"second": 12,
	}

	// Initialize a map for the float values
	floats := map[string]float64{
		"first":  35.98,
		"second": 26.99,
	}

	fmt.Printf("Non-Generic Sums: %v and %v\n", SumInts(ints), SumFloats(floats))
}

// SumInts adds together the values of m.
func SumInts(m map[string]int64) int64 {
	var s int64
	for _, v := range m {
		s += v
	}
	return s
}

// SumFloats adds together the values of m.
func SumFloats(m map[string]float64) float64 {
	var s float64
	for _, v := range m {
		s += v
	}
	return s
}
```

在这段代码中
* 声明了两个函数`SumInts()`和`SumFloats()`，用于计算映射中所有值的和，值类型分别为`int64`和`float64`。
* 在`main()`函数中初始化了一个`int64`值的映射和一个`float64`值的映射，调用上述两个函数计算值的和，并打印结果。

2.在generics目录中运行代码。

```shell
$ go run .
Non-Generic Sums: 46 and 62.97
```

### 5.3 添加泛型函数来处理多种类型
可以看到，`SumInts()`和`SumFloats()`这两个函数除了值类型外完全相同。在本节中，将编写一个泛型函数，能够处理包含整型**或**浮点型值的映射，从而用一个函数代替之前的两个函数。

泛型函数除了普通参数外还需要声明**类型参数**(type parameter)，使其能够处理不同的类型。调用泛型函数时需要指定**类型实参**(type argument)。

每个类型参数都有一个**类型约束**(type constraint)，用于指定允许使用的类型实参。类型约束通常表示一组类型（例如“`int64`和`float64`”），但在编译时类型参数仅代表单个类型，即调用代码提供的类型实参（例如“`int64`或`float64`之一”）。如果类型实参不满足类型约束，代码将无法编译。

类型参数必须支持泛型代码对其执行的所有操作。例如，如果函数试图对约束包含数值类型的类型参数执行字符串操作，代码将无法编译。

1.在main.go末尾添加以下代码：

```go
// SumIntsOrFloats sums the values of map m.
// It supports both int64 and float64 as map values.
func SumIntsOrFloats[K comparable, V int64 | float64](m map[K]V) V {
	var s V
	for _, v := range m {
		s += v
	}
	return s
}
```

在这段代码中
* 声明了一个泛型函数`SumIntsOrFloats()`，具有两个类型参数（在方括号中）`K`和`V`，参数类型为`map[K]V`，返回类型为`V`。
* 类型参数`K`的约束为`comparable`，它允许任意支持比较运算符`==`和`!=`的类型。Go要求映射的键是可比较的。
* 类型参数`V`的约束为两种类型的并集：`int64`和`float64`。使用`|`指定两种类型的并集，这意味着约束允许二者中任何一种类型。
* 参数`m`的类型为`map[K]V`，这里使用了类型参数。

2.在`main()`函数末尾添加以下代码：

```go
fmt.Printf("Generic Sums: %v and %v\n",
	SumIntsOrFloats[string, int64](ints),
	SumIntsOrFloats[string, float64](floats))
```

在这段代码中
* 分别使用两个映射调用了泛型函数`SumIntsOrFloats()`。
* 在方括号中指定了类型实参，编译器会将类型参数替换为类型实参（例如`K=string, V=int64`）。

3.运行代码。

```shell
$ go run .
Non-Generic Sums: 46 and 62.97
Generic Sums: 46 and 62.97
```

### 5.4 调用泛型函数时省略类型实参
在大多数情况下，可以省略泛型函数调用中的类型实参，编译器能够根据函数参数自动推断。注意，这并不总是可行的。例如，如果需要调用一个无参数的泛型函数，就需要显式指定类型实参。

在`main()`函数末尾添加以下代码：

```go
fmt.Printf("Generic Sums, type parameters inferred: %v and %v\n",
	SumIntsOrFloats(ints),
	SumIntsOrFloats(floats))
```

运行代码：

```shell
$ go run .
Non-Generic Sums: 46 and 62.97
Generic Sums: 46 and 62.97
Generic Sums, type parameters inferred: 46 and 62.97
```

### 5.5 声明类型约束
在本节中，将把之前定义的类型约束移到接口中，以便在其他地方复用。

可以将类型约束声明为接口，该约束将允许任何实现了该接口的类型。例如，如果声明一个包含三个方法的类型约束接口，然后将其用于泛型函数的类型参数，那么用于调用该函数的类型实参必须实现这三个方法。（直接使用接口类型的函数参数也能实现同样的效果？）

类型约束接口也可以引用特定的类型，如下所示。

1.在`main()`函数上方添加以下代码：

```go
type Number interface {
	int64 | float64
}
```

这段代码声明了用作类型约束的接口类型`Number`，定义为`int64`和`float64`的并集。

2.在main.go末尾添加以下代码：

```go
// SumNumbers sums the values of map m.
// It supports both int64 and float64 as map values.
func SumNumbers[K comparable, V Number](m map[K]V) V {
	var s V
	for _, v := range m {
		s += v
	}
	return s
}
```

这段代码声明了一个泛型函数`SumNumbers()`，逻辑与之前的`SumIntsOrFloats()`相同，但使用了接口而不是类型并集作为类型约束。

3.在`main()`函数末尾添加以下代码：

```go
fmt.Printf("Generic Sums with Constraint: %v and %v\n",
	SumNumbers(ints),
	SumNumbers(floats))
```

4.运行代码。

```shell
$ go run .
Non-Generic Sums: 46 and 62.97
Generic Sums: 46 and 62.97
Generic Sums, type parameters inferred: 46 and 62.97
Generic Sums with Constraint: 46 and 62.97
```

完整代码：[generics/main.go](https://github.com/ZZy979/go-tutorials/blob/main/generics/main.go)

## 6.访问数据库
[Tutorial: Accessing a relational database](https://go.dev/doc/tutorial/database-access)

本教程将介绍如何使用Go访问关系型数据库。在本教程中，将创建一个复古爵士乐唱片的数据库，然后编写访问该数据库的代码。

标准库的[database/sql](https://pkg.go.dev/database/sql)包包含用于连接数据库、执行查询等操作的类型和函数。详见[Accessing relational databases](https://go.dev/doc/database/)。

### 6.1 前置条件
安装[MySQL](https://www.mysql.com/)数据库。

### 6.2 创建目录
在主目录中创建一个`data-access`模块。

```shell
mkdir data-access
cd data-access
go mod init example/data-access
```

### 6.3 创建数据库
使用[MySQL命令行客户端](https://dev.mysql.com/doc/refman/8.4/en/mysql.html)创建一个复古爵士乐唱片数据库。

1.打开一个命令行窗口，登录到MySQL。

```shell
$ mysql -u root -p
Enter password: 

mysql>
```

2.在mysql命令行中创建一个数据库`recordings`，并切换到该数据库。

```shell
mysql> create database recordings;
Query OK, 1 row affected (0.02 sec)

mysql> use recordings;
Database changed
```

3.在data-access目录中创建一个文件create-tables.sql，内容如下：

```sql
DROP TABLE IF EXISTS album;
CREATE TABLE album (
  id         INT AUTO_INCREMENT NOT NULL,
  title      VARCHAR(128) NOT NULL,
  artist     VARCHAR(255) NOT NULL,
  price      DECIMAL(5,2) NOT NULL,
  PRIMARY KEY (`id`)
);

INSERT INTO album
  (title, artist, price)
VALUES
  ('Blue Train', 'John Coltrane', 56.99),
  ('Giant Steps', 'John Coltrane', 63.99),
  ('Jeru', 'Gerry Mulligan', 17.99),
  ('Sarah Vaughan', 'Sarah Vaughan', 34.98);
```

在这段SQL代码中
* 删除名为`album`的表，以便可以重复运行这个脚本。
* 创建了一张表`album`，包含4列：`id`、`title`、`artist`和`price`。其中`id`是自动递增的。
* 向`album`表插入了4行记录。

4.在mysql命令行中使用`source`命令运行这个SQL脚本：

```shell
mysql> source data-access/create-tables.sql
```

注意：在Windows中，文件路径要使用正斜杠而不是反斜杠，例如 C:/Users/yourname/data-access/create-tables.sql

5.使用`SELECT`语句验证成功创建了表和数据。

```shell
mysql> select * from album;
+----+---------------+----------------+-------+
| id | title         | artist         | price |
+----+---------------+----------------+-------+
|  1 | Blue Train    | John Coltrane  | 56.99 |
|  2 | Giant Steps   | John Coltrane  | 63.99 |
|  3 | Jeru          | Gerry Mulligan | 17.99 |
|  4 | Sarah Vaughan | Sarah Vaughan  | 34.98 |
+----+---------------+----------------+-------+
4 rows in set (0.00 sec)
```

### 6.4 查找数据库驱动
接下来，需要寻找一个能够将`database/sql`包中的函数调用转换为数据库可理解的请求的数据库驱动。

1.在浏览器打开[SQL Database Drivers](https://go.dev/wiki/SQLDrivers)。

2.在本教程中，使用[github.com/go-sql-driver/mysql](https://github.com/go-sql-driver/mysql/)来访问MySQL（注意，仓库的URL就是模块名）。

3.在data-access目录中创建一个文件main.go并添加以下代码：

```go
package main

import "github.com/go-sql-driver/mysql"
```

4.使用[go get](https://pkg.go.dev/cmd/go#hdr-Add_dependencies_to_current_module_and_install_them)命令将`github.com/go-sql-driver/mysql`模块作为依赖添加到go.mod文件，并下载该依赖。

```shell
$ go get .
go: added filippo.io/edwards25519 v1.1.0
go: added github.com/go-sql-driver/mysql v1.9.3
```

其中`.`表示“为当前目录中的代码获取依赖”。详见[Adding a dependency](https://go.dev/doc/modules/managing-dependencies#adding_dependency)。

注：也可以先使用`go get github.com/go-sql-driver/mysql`命令下载依赖，然后在代码中导入，最后执行`go mod tidy`。

### 6.5 连接数据库
导入驱动后，就可以开始编写访问数据库的代码了。

1.在main.go中添加以下代码：

```go
var db *sql.DB

func main() {
	// Capture connection properties.
	cfg := mysql.NewConfig()
	cfg.User = os.Getenv("DBUSER")
	cfg.Passwd = os.Getenv("DBPASS")
	cfg.Net = "tcp"
	cfg.Addr = "127.0.0.1:3306"
	cfg.DBName = "recordings"

	// Get a database handle.
	var err error
	db, err = sql.Open("mysql", cfg.FormatDSN())
	if err != nil {
		log.Fatal(err)
	}

	pingErr := db.Ping()
	if pingErr != nil {
		log.Fatal(pingErr)
	}
	fmt.Println("Connected!")
}
```

在这段代码中
* 声明了一个`*sql.DB`类型的全局变量`db`。[sql.DB](https://pkg.go.dev/database/sql#DB)类型表示数据库**句柄**(handle)，可用于访问数据库。在实际代码中，应该避免全局变量，而是通过函数参数传递。
* 使用MySQL驱动的[Config](https://pkg.go.dev/github.com/go-sql-driver/mysql#Config)类型收集连接参数，并使用`FormatDSN()`方法将其格式化为DSN (data source name)字符串。
* 调用`sql.Open()`打开数据库连接，如果失败则打印错误消息。
* 调用`DB.Ping()`确认连接成功并打印消息 "Connected!" 。

2.在命令行中设置程序使用的环境变量`DBUSER`和`DBPASS`，指定数据库用户名和密码。

在Linux或Mac上：

```shell
export DBUSER=username
export DBPASS=password
```

在Windows上：

```shell
set DBUSER=username
set DBPASS=password
```

3.在data-access目录中运行代码。

```shell
$ go run .
Connected!
```

### 6.6 查询多行
接下来将使用Go执行一个SQL查询。对于可能返回多行的SQL语句，使用`DB.Query()`方法执行查询，并遍历它返回的行。

1.在`main()`函数上方添加`Album`结构体的定义，用于保存查询返回的行数据。结构体字段名和类型与数据库表`album`的列名和类型对应。

```go
type Album struct {
	ID     int64
	Title  string
	Artist string
	Price  float32
}
```

2.在`main()`函数下方添加`albumsByArtist()`函数，用于根据艺术家名字查询专辑。

```go
// albumsByArtist queries for albums that have the specified artist name.
func albumsByArtist(name string) ([]Album, error) {
	// An albums slice to hold data from returned rows.
	var albums []Album

	rows, err := db.Query("SELECT * FROM album WHERE artist = ?", name)
	if err != nil {
		return nil, fmt.Errorf("albumsByArtist %q: %v", name, err)
	}
	defer rows.Close()
	// Loop through rows, using Scan to assign column data to struct fields.
	for rows.Next() {
		var alb Album
		if err := rows.Scan(&alb.ID, &alb.Title, &alb.Artist, &alb.Price); err != nil {
			return nil, fmt.Errorf("albumsByArtist %q: %v", name, err)
		}
		albums = append(albums, alb)
	}
	if err := rows.Err(); err != nil {
		return nil, fmt.Errorf("albumsByArtist %q: %v", name, err)
	}
	return albums, nil
}
```

在这段代码中
* 声明了一个`Album`类型的切片`albums`，用于保存查询结果。
* 使用`DB.Query()`执行`SELECT`语句来查询具有指定艺术家名字的专辑。第一个参数是SQL语句，之后可以传递零个或多个任意类型的参数，用于提供SQL语句中占位符(`?`)的值。与直接拼接字符串相比，这种方式可以避免SQL注入风险。
* 使用`defer`语句关闭`rows`，以便在函数退出时释放它持有的资源。
* 使用`for`语句遍历返回的行，使用`Rows.Scan()`将每行各个列的值赋给`Album`的字段。
  * `Scan()`接受一个指针列表，用于写入列的值。这里使用`&`运算符获取`alb`变量各字段的指针。
  * 在Go中，只有条件部分的`for`循环相当于Java中的`while`循环。`Rows.Next()`返回一个布尔值，表示是否还有下一行。
* 在循环内部，将新的`alb`添加到`albums`切片。
* 循环结束后，使用`Rows.Err()`检查整个查询是否有错误。注意，如果查询本身失败了，在这里检查错误是发现结果不完整的唯一方式。

3.在`main()`函数末尾调用`albumsByArtist()`函数并打印结果。

```go
albums, err := albumsByArtist("John Coltrane")
if err != nil {
	log.Fatal(err)
}
fmt.Printf("Albums found: %v\n", albums)
```

4.运行代码。

```shell
$ go run .
Connected!
Albums found: [{1 Blue Train John Coltrane 56.99} {2 Giant Steps John Coltrane 63.99}]
```

### 6.7 查询一行
对于至多会返回一行的SQL语句，可以使用`QueryRow()`，这比`Query()`循环更简单。

1.在`albumsByArtist()`函数下方添加`albumByID()`函数，用于根据ID查询专辑。

```go
// albumByID queries for the album with the specified ID.
func albumByID(id int64) (Album, error) {
	// An album to hold data from the returned row.
	var alb Album

	row := db.QueryRow("SELECT * FROM album WHERE id = ?", id)
	if err := row.Scan(&alb.ID, &alb.Title, &alb.Artist, &alb.Price); err != nil {
		if err == sql.ErrNoRows {
			return alb, fmt.Errorf("albumsById %d: no such album", id)
		}
		return alb, fmt.Errorf("albumsById %d: %v", id, err)
	}
	return alb, nil
}
```

在这段代码中
* 使用`DB.QueryRow()`执行`SELECT`语句来查询具有指定ID的专辑（ID是唯一的，因此至多会返回一行），返回一个`sql.Row`。为了简化调用代码，该方法不返回错误，而是之后在`Rows.Scan()`中返回查询错误（例如`sql.ErrNoRows`）。
* 使用`Rows.Scan()`将列值拷贝到结构体字段，并检查错误。特殊错误`sql.ErrNoRows`表示查询未返回任何行。
* 在Go中，结构体类型的值不能为`nil`，因此在发生错误时仍然需要返回`alb`（此时所有字段都是零值）。

2.在`main()`函数末尾调用`albumByID()`函数并打印结果。

```go
// Hard-code ID 2 here to test the query.
alb, err := albumByID(2)
if err != nil {
	log.Fatal(err)
}
fmt.Printf("Album found: %v\n", alb)
```

3.运行代码。

```shell
$ go run .
Connected!
Albums found: [{1 Blue Train John Coltrane 56.99} {2 Giant Steps John Coltrane 63.99}]
Album found: {2 Giant Steps John Coltrane 63.99}
```

### 6.8 插入数据
在本节中，将使用Go执行`INSERT`语句来向数据库添加新行。为了执行不返回结果的SQL语句，需要使用`DB.Exec()`。

1.在`albumByID()`函数下方添加`addAlbum()`函数，用于在数据库中插入一个新的专辑。

```go
// addAlbum adds the specified album to the database,
// returning the album ID of the new entry
func addAlbum(alb Album) (int64, error) {
	result, err := db.Exec("INSERT INTO album (title, artist, price) VALUES (?, ?, ?)",
		alb.Title, alb.Artist, alb.Price)
	if err != nil {
		return 0, fmt.Errorf("addAlbum: %v", err)
	}
	id, err := result.LastInsertId()
	if err != nil {
		return 0, fmt.Errorf("addAlbum: %v", err)
	}
	return id, nil
}
```

在这段代码中
* 使用`DB.Exec()`执行了`INSERT`语句。与`Query()`类似，`Exec()`也接受一个SQL语句和若干个参数值。
* 该方法返回一个`sql.Result`，使用其`LastInsertId()`方法获取插入到数据库的行ID。

2.在`main()`函数末尾调用`addAlbum()`函数并打印结果。

```go
albID, err := addAlbum(Album{
	Title:  "The Modern Sound of Betty Carter",
	Artist: "Betty Carter",
	Price:  49.99,
})
if err != nil {
	log.Fatal(err)
}
fmt.Printf("ID of added album: %v\n", albID)
```

3.运行代码。

```shell
$ go run .
Connected!
Albums found: [{1 Blue Train John Coltrane 56.99} {2 Giant Steps John Coltrane 63.99}]
Album found: {2 Giant Steps John Coltrane 63.99}
ID of added album: 5
```

完整代码：[data-access/main.go](https://github.com/ZZy979/go-tutorials/blob/main/data-access/main.go)
