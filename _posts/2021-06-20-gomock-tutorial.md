---
title: gomock使用教程
date: 2021-06-20 18:02:21 +0800
categories: [Go]
tags: [golang, gomock, unit test]
---
gomock是Go官方提供的模拟(mock)框架，提供了打桩和验证调用的功能

mockgen工具用于针对接口生成mock对象代码（不能mock单独的函数或方法）

GitHub仓库：<https://github.com/golang/mock>

## 安装
```
go get github.com/golang/mock/gomock
go get github.com/golang/mock/mockgen
```

安装完成后可以在命令行中直接使用mockgen命令（需要将$GOPATH/bin添加到PATH环境变量）

## 示例
假设foo/foo.go中定义了一个接口`Foo`和一个函数`Baz()`：

```go
package foo

type Foo interface {
	Bar(x int) int
}

func Baz(foo Foo, x int) int {
	return foo.Bar(x) + 1
}
```

其中`Foo`接口有一个`Bar()`方法（代表数据库、文件、网络等操作），函数`Baz()`接收一个`Foo`类型的参数，调用其`Bar()`方法并返回进一步计算后的结果

正常情况下针对`Baz()`函数的测试用例需要调用`Foo.Bar()`方法，如果不想真正调用该方法就需要针对`Foo`接口生成mock对象：

```
mockgen -source=foo\foo.go -destination=foo\mock\mock_foo.go
```

自动生成的代码包名默认为"mock_"+源文件包名，可以通过`-package`参数指定

mockgen工具会自动生成`Foo`接口的mock对象代码，此时可以在测试用例中使用mock对象：

```go
package foo_test

import (
	"github.com/golang/mock/gomock"
	"gomock_demo/foo"
	mock_foo "gomock_demo/foo/mock"
	"testing"
)

func TestBaz(t *testing.T) {
	ctrl := gomock.NewController(t)
	defer ctrl.Finish()

	m := mock_foo.NewMockFoo(ctrl)
	m.EXPECT().Bar(gomock.Eq("abc")).Return(123)

	expected := 124
	if got := foo.Baz(m, "abc"); got != expected {
		t.Errorf(`Baz(m, "abc") = %d, want %d`, got, expected)
	}
}
```

* mockgen自动生成的`mock_foo.NewMockFoo()`函数返回一个`Foo`接口的mock对象，该对象实现了`Foo`接口，因此可以被传给`Baz()`函数，也可以直接对`m.Bar()`方法进行测试
* 通过mock对象指定了`Bar()`方法当参数等于`"abc"`时返回`123`，同时断言一定以`"abc"`为参数调用了`Bar()`方法，这一过程叫做打桩

## 打桩
打桩(stub)用于指定方法的返回值，同时检测调用次数、调用顺序等

### 匹配参数
调用`m.EXPECT().Bar()`时可以指定参数满足的条件，将原方法的参数替换为对应的`Matcher`
* `Eq(x)` 等于x
* `Not(x)` 不等于x
* `Any()` 任何值
* `Nil()` 值是nil
* `Len(i)` 长度为i

```go
type testCase struct {
	arg  string
	want int
}

func TestArgumentMatcher(t *testing.T) {
	ctrl := gomock.NewController(t)
	defer ctrl.Finish()

	m := mock_foo.NewMockFoo(ctrl)
	m.EXPECT().Bar(gomock.Eq("abc")).Return(123)
	m.EXPECT().Bar(gomock.Len(0)).Return(0)
	m.EXPECT().Bar(gomock.Any()).Return(-1)

	tests := []testCase{
		{"abc", 124},
		{"", 1},
		{"def", 0},
	}
	for _, test := range tests {
		if got := foo.Baz(m, test.arg); got != test.want {
			t.Errorf(`Baz(m, %q) = %d, want %d`, test.arg, got, test.want)
		}
	}
}
```

### 返回值
`m.EXPECT().Bar()`返回的是`*gomock.Call`类型的值，可使用以下方法指定被调用时的行为：
* `Return(rets)` 返回指定的值
* `Do(f)` 调用指定的函数（参数必须与被调用方法匹配），忽略返回值
* `DoAndReturn(f)` 调用指定的函数（参数和返回值必须与被调用方法匹配），返回该函数的返回值

注意：这些方法的返回值仍然是`*Call`类型，因此可以链式调用，下同
如果没有指定任何返回值则返回零值

```go
func TestReturn(t *testing.T) {
	ctrl := gomock.NewController(t)
	defer ctrl.Finish()

	m := mock_foo.NewMockFoo(ctrl)
	m.EXPECT().Bar(gomock.Eq("abc")).Return(123)
	m.EXPECT().Bar(gomock.Any()).DoAndReturn(func(x string) int { return len(x) })

	tests := []testCase{
		{"abc", 124},
		{"def", 4},
	}
	for _, test := range tests {
		if got := foo.Baz(m, test.arg); got != test.want {
			t.Errorf(`Baz(m, %q) = %d, want %d`, test.arg, got, test.want)
		}
	}
}
```

### 调用次数
`*Call`的以下方法用于断言方法的调用次数
* `Times(n)` 被调用n次
* `MinTimes(n)` 至少被调用n次
* `MaxTimes(n)` 至多被调用n次
* `AnyTimes()` 可以被调用任意次（包括0次）

如果不指定则默认为1次，因此如果创建了mock对象但没有调用其方法则测试会失败，打印错误信息controller.go:269: missing call(s) to *mock_foo.MockFoo.Bar(is equal to abc (string))

```go
func TestCallTimes(t *testing.T) {
	ctrl := gomock.NewController(t)
	defer ctrl.Finish()

	m := mock_foo.NewMockFoo(ctrl)
	m.EXPECT().Bar(gomock.Eq("abc")).Return(123)
	m.EXPECT().Bar(gomock.Eq("def")).Return(456).Times(2)
	m.EXPECT().Bar(gomock.Eq("ghi")).Return(789).AnyTimes()

	tests := []testCase{
		{"abc", 124},
		{"def", 457},
		{"def", 457}, // 删除该用例会报错
		{"ghi", 790}, // 删除该用例不会报错
	}
	for _, test := range tests {
		if got := foo.Baz(m, test.arg); got != test.want {
			t.Errorf(`Baz(m, %q) = %d, want %d`, test.arg, got, test.want)
		}
	}
}
```

### 调用顺序
`gomock.InOrder()`函数断言调用的出现顺序

```go
func TestInOrder(t *testing.T) {
	ctrl := gomock.NewController(t)
	defer ctrl.Finish()

	m := mock_foo.NewMockFoo(ctrl)
	c1 := m.EXPECT().Bar(gomock.Eq("abc")).Return(123)
	c2 := m.EXPECT().Bar(gomock.Eq("def")).Return(456)
	gomock.InOrder(c1, c2)

	tests := []testCase{
		{"abc", 124},
		{"def", 457}, // 交换测试用例的顺序会报错
	}
	for _, test := range tests {
		if got := foo.Baz(m, test.arg); got != test.want {
			t.Errorf(`Baz(m, %q) = %d, want %d`, test.arg, got, test.want)
		}
	}
}
```
