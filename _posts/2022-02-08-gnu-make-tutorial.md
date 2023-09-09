---
title: GNU Make构建工具使用教程
date: 2022-02-08 17:40:01 +0800
categories: [Build Tools]
tags: [build tools, make]
---
## 1.基本概念
[GNU Make](https://www.gnu.org/software/make/)是一种常用的构建工具，主要用于C语言项目。但是GNU Make并不限于某种编程语言，也不限于编译代码的场景。任何“只要某个文件发生变化，就需要重新构建”的场景都可以用GNU Make构建。

将源代码变成可执行文件叫做编译(compile)；先编译这个，还是先编译那个（即编译的安排）叫做**构建**(build)。

Linux系统自带了**make命令**；Windows系统需要安装[MinGW](https://sourceforge.net/projects/mingw/)，使用安装目录下的bin\mingw32-make.exe。

## 2.Makefile文件语法
Make命令从当前目录下名为**Makefile**的文件中读取构建规则。

### 2.1 规则
Makefile文件是一个[规则](https://www.gnu.org/software/make/manual/make.html#Rules)(rule)的集合，每条规则描述如何做一件特定的事情。规则的格式如下：

```
<目标>: <前置条件>
	<命令>
```

* 目标(target)：通常是文件名，也可以是操作的名字（称为伪目标）
* 前置条件(prerequisites)：目标依赖的一个或多个文件，用空格分隔
* [命令](https://www.gnu.org/software/make/manual/make.html#Recipes)(commands/recipes)：构建目标所要执行的操作，由一行或多行shell命令组成，每行必须以制表符(tab)开头

目标是必需的；前置条件和命令都是可选的，但二者必须至少存在一个。

每条规则定义两件事：构建目标的前置条件是什么，以及如何构建。

例如，以下Makefile文件定义了构建a.txt的规则：

```
a.txt: b.txt c.txt
	cat b.txt c.txt > a.txt
```

这条规则的含义是：要制作（构建）出文件a.txt（目标），需要文件b.txt和c.txt（前置条件），制作方法（命令）是使用`cat`命令（Windows系统需要使用`type`命令）合并这两个文件。

make命令的格式如下（其他选项见[选项总结](https://www.gnu.org/software/make/manual/make.html#Options-Summary)）：

```bash
make [options] [target...]
```

make命令从当前目录下名为Makefile的文件中读取构建规则，并构建指定的目标，如果未指定则构建第一个目标。

例如，要使用上面的Makefile构建a.txt，首先确保b.txt和c.txt已经存在（与Makefile在同一目录下，假设内容分别为bbb和ccc），执行`make a.txt`即可：

```bash
$ make a.txt 
cat b.txt c.txt > a.txt
$ cat a.txt 
bbb
ccc
```

make根据文件的**最后修改时间**来判断是否需要重新构建。如果目标文件比所有依赖文件都要新，则认为这个目标是最新的，不需要重新构建；否则会重新生成该目标。

例如，再次执行`make a.txt`将什么都不做，但如果修改了b.txt再执行`make a.txt`就会重新构建a.txt：

```bash
$ make a.txt 
make: 'a.txt' is up to date.
$ echo BBB > b.txt 
$ make a.txt 
cat b.txt c.txt > a.txt
$ cat a.txt 
BBB
ccc
```

注意：每行命令都在一个单独的shell中运行，因此上一行命令设置的环境变量无法在下一行命令中使用。（`$`在Makefile中有特殊含义，因此需要转义）

```
var-lost:
	export foo=bar
	echo "foo=[$$foo]"
```

```bash
$ make var-lost 
export foo=bar
echo "foo=[$foo]"
foo=[]
```

可以使用分号和反斜杠使两行命令在同一个进程中运行。

```
var-kept:
	export foo=bar; \
	echo "foo=[$$foo]"
```

```bash
$ make var-kept 
export foo=bar; \
echo "foo=[$foo]"
foo=[bar]
```

### 2.2 注释
Makefile的注释以`#`开头。

### 2.3 回显
make默认会打印运行的每一条命令（包括注释），称为[回显](https://www.gnu.org/software/make/manual/make.html#Echoing)(echoing)。在行开头加上`@`则关闭这一行的回显，make命令的`-s`选项关闭所有回显。

例如：

```
tutorial:
	# this is a comment
	echo "hello, world"
```

```bash
$ make tutorial 
## this is a comment
echo "hello, world"
hello, world
$ make -s tutorial 
hello, world
```

关闭回显：

```
tutorial:
	@# this is a comment
	@echo "hello, world"
```

```bash
$ make tutorial 
hello, world
```

由于在构建过程中，需要了解当前在执行哪条命令，所以通常只在注释和echo命令前加上`@`。

### 2.4 伪目标
[伪目标](https://www.gnu.org/software/make/manual/make.html#Phony-Targets)不是真实的文件名，而是操作的名字。

例如，以下Makefile定义了一个名为`clean`的伪目标，执行`make clean`将删除所有的.o文件。为了避免与名为clean的文件冲突，使用`.PHONY`将其声明为伪目标：

```
.PHONY: clean
clean:
	rm *.o
```

伪目标可以是其他伪目标的前置条件，此时相当于子程序。例如，对于以下Makefile，运行`make cleanall`将删除所有的.o文件、.diff文件和program文件：

```
.PHONY: cleanall cleanobj cleandiff

cleanall: cleanobj cleandiff
	rm program

cleanobj:
	rm *.o

cleandiff:
	rm *.diff
```

伪目标的另一种常见用法是组合多个目标。由于make命令默认运行第一个目标，因此通常定义个名为`all`的伪目标，并将其他目标作为其前置条件。例如：

```
all: prog1 prog2 prog3
.PHONY: all

prog1: prog1.h prog1.c
	cc -o prog1 prog1.c

prog2: prog2.h prog2.c
	cc -o prog2 prog2.c

prog3: prog3.h prog3.c
	cc -o prog3 prog3.c
```

此时直接执行`make`命令即可构建所有目标。
（这也是一个简单的构建C语言项目的例子）

### 2.5 变量
在Makefile中可以定义[变量](https://www.gnu.org/software/make/manual/make.html#Using-Variables)来表示文本字符串。变量可用于在目标、前置条件、命令等任何部分进行**文本替换**，可以表示文件名列表、编译器参数、要运行的程序等任何字符串可以表示的内容。

使用`=`给变量赋值，使用`$(foo)`引用变量`foo`。

例如，假设foo.h和bar.h分别定义了两个函数，并在foo.c和bar.c中实现，main.c调用了这两个函数，则构建main程序的Makefile可以这样写：

```
objects = main.o foo.o bar.o
main: $(objects)
	cc -o main $(objects)

main.o: main.c foo.h bar.h
	cc -c main.c
foo.o: foo.h foo.c
	cc -c foo.c
bar.o: bar.h bar.c
	cc -c bar.c
```

其中`cc`是C编译器命令的别名（符号链接），可能是`gcc`或`clang`，取决于系统具体使用的编译器。

除了`=`，还有`:=`、`?=`和`+=`三种赋值方式（见[设置变量](https://www.gnu.org/software/make/manual/make.html#Setting)）：
* `=`：执行时扩展，允许递归扩展
* `:=`：定义时扩展
* `?=`：只有在变量为空时才赋值
* `+=`：将文本添加到变量尾部，并自动添加一个空格

### 2.6 隐式规则
有些标准的构建目标的方式非常常用，例如.o文件由.c文件通过C编译器得到，即

```
foo.o: foo.c
	cc -c -o foo.o foo.c
```

因此，make提供了一系列[隐式规则](https://www.gnu.org/software/make/manual/make.html#Implicit-Rules)，根据目标和依赖文件的扩展名即可自动推断构建规则，从而不必全部写出。

例如，上面的规则符合编译C代码的隐式规则，从而可以简写为`foo.o: foo.c`或者完全省略（除非需要指定其他的前置条件）。

隐式规则的完整列表见[内置规则目录](https://www.gnu.org/software/make/manual/make.html#Catalogue-of-Rules)。

#### 2.6.1 隐式变量
编译C代码的隐式规则使用的实际命令是`$(CC) -c $(CPPFLAGS) $(CFLAGS)`，其中`CC`、`CPPFLAGS`和`CFLAGS`都是[隐式变量](https://www.gnu.org/software/make/manual/make.html#Implicit-Variables)。隐式变量分为两类：程序名称和程序参数。例如：

* `CC`：C编译器，默认为 "cc"
* `CXX`：C++编译器，默认为 "g++"
* `CFLAGS`：C编译器选项
* `CXXFLAGS`：C++编译器选项
* `CPPFLAGS`：C预处理器选项

可以通过给这些变量赋值来指定要使用的编译器或传递给编译器的选项。

#### 2.6.2 模式规则
可以使用[模式规则](https://www.gnu.org/software/make/manual/make.html#Pattern-Rules)来自定义隐式规则。模式规则与普通规则的唯一区别是目标中包含**一个** `%`，此时目标名作为文件名的匹配模式，`%`可以匹配任何非空子串（匹配的部分叫做stem）。前置条件也可以包含`%`，表示与目标的匹配部分相同。

例如，模式规则`%.o: %.c`表示从任意的xxx.c构建xxx.o。内置的编译C代码的隐式规则可用模式规则表示为：

```
%.o: %.c
	$(CC) -c $(CFLAGS) $(CPPFLAGS) $< -o $@
```

其中自动变量`$@`表示目标文件名(xxx.o)，`$<`表示源文件名(xxx.c)。

另一个例子，构建任何xxx.txt的方式是使用echo命令写入目标文件，内容为xxx：

```
%.txt:
	echo $* > $@
```

自动变量`$*`表示`%`匹配的部分。例如，运行`make abc.txt`，则abc.txt的内容为abc。

#### 2.6.3 自动变量
使用模式规则时，要在命令中使用实际匹配的目标文件名和依赖文件名，需要使用[自动变量](https://www.gnu.org/software/make/manual/make.html#Automatic-Variables)。自动变量只能在规则的命令部分使用。

* `$@`：目标文件名
* `$<`：第一个依赖文件名
* `$^`：所有依赖文件名，空格分隔
* `$?`：所有比目标新的依赖文件名，空格分隔
* `$*`：模式规则中`%`匹配的部分(stem)
* `$(@D)`和`$(@F)`：分别表示目标名称的目录和文件名部分
* `$(<D)`和`$(<F)`：分别表示第一个依赖名称的目录和文件名部分
* `$(^D)`和`$(^F)`：分别表示所有依赖名称的目录和文件名部分
* `$(?D)`和`$(?F)`：分别表示所有比目标新的依赖名称的目录和文件名部分
* `$(*D)`和`$(*F)`：分别表示`%`匹配的部分(stem)的目录和文件名部分
* `$$`：$本身

### 2.7 函数
在Makefile中可以使用[函数](https://www.gnu.org/software/make/manual/make.html#Functions)进行文本处理，结果将被替换到函数调用处，就像变量替换一样。

函数调用的语法：

```
$(函数名 参数)
```

参数之间用逗号分隔，参数可以是字符串、变量或函数调用，不能包含逗号和未匹配的括号。

#### 2.7.1 常用函数
##### subst
```
$(subst from,to,text)
```

执行文本替换。

例如，`$(subst ee,EE,feet on the street)`返回`fEEt on the strEEt`。

##### patsubst
```
$(patsubst pattern,replacement,text)
```

将`text`中空白符分隔的匹配`pattern`的单词替换为`replacement`。`pattern`可以包含`%`来匹配任意数量的任意字符。如果`replacement`中也包含`%`，则`%`被替换为`pattern`中的`%`匹配的文本。

例如，`$(patsubst %.c,%.o,x.c.c bar.c)`返回`x.c.o bar.o`。

[替换引用](https://www.gnu.org/software/make/manual/make.html#Substitution-Refs)用于替换文件名的后缀，可以达到与`parsubst`函数相同的效果：

```
$(var:suffix=replacement)
```

等价于

```
$(patsubst %suffix,%replacement,$(var))
```

例如，假设有一个变量：

```
objects = foo.o bar.o baz.o
```

则`$(objects:.o=.c)`和`$(patsubst %.o,%.c,$(objects))`都返回`foo.c bar.c baz.c`。

##### shell
`shell`函数将参数作为shell命令，并将命令的输出作为返回结果（相当于shell中反引号的作用），make会将换行符转换为空格。

例如，`contents := $(shell cat foo)`将`content`设置为文件foo的内容，行之间用空格（而不是换行符）分隔。

### 2.8 递归使用make
[递归使用make](https://www.gnu.org/software/make/manual/make.html#Recursion)是指在Makefile中使用make命令。例如，一个较大的项目中每个子模块有一个单独的Makefile。

例如，要在当前目录的Makefile中运行子目录foo中的Makefile，可以这样写：

```
.PHONY: foo
foo:
	$(MAKE) -C foo
```

## 3.示例
官方文档[Makefiles介绍](https://www.gnu.org/software/make/manual/make.html#Introduction)一节给出了一个构建C语言项目的Makefile示例，以及如何使用变量和隐式规则简化Makefile。

## 4.缺点
GNU Make对于只有几个源文件、没有子模块、构建目标是单个可执行文件的小型C语言项目来说就足够了。可以参照这个模板编写Makefile：[A makefile for 99% of your programs](http://nuclear.mutantstargoat.com/articles/make/#practical-makefile)。

但是在包含许多子模块的大型C/C++项目中，GNU Make使用起来就会比较困难。例如，考虑下面包含两个模块的项目：

```
project/
    module1/
        Makefile
        foo.h
        foo.c
        bar.h
        bar.c
    module2/
        Makefile
        baz.c
```

模块module1提供了`foo`和`bar`两个函数，其中`bar`依赖`foo`，对应的Makefile如下（这两个规则并没有指定命令，而是使用了由.c文件构建.o文件的隐式规则）：

```
foo.o: foo.c foo.h
bar.o: bar.c bar.h foo.h
```

模块module2包含一个可执行程序baz，其中调用了`bar`函数，对应的Makefile如下：

```
baz: baz.o ../module1/bar.o ../module1/foo.o
	$(CC) -o $@ $^

../module1/%.o:
	$(MAKE) -C ../module1 $@
```

由此可以看出Make的不方便之处：
* **必须手动添加间接依赖**：虽然baz的直接依赖只有bar.o，但必须同时添加间接依赖foo.o（bar.o与foo.o之间的依赖关系没有体现在module1/Makefile中，因为生成.o文件是只编译不链接）
* **跨目录依赖必须手动编写构建规则**：为了能够在module2目录下构建foo.o和bar.o，必须手动编写一个规则来调用module1/Makefile中的相应规则（如果不写这个规则也能编译，但构建foo.o和bar.o使用的不是module1/Makefile中的规则，而是当前Makefile的隐式规则`../module1/foo.o: ../module1/foo.c`和`../module1/bar.o: ../module1/bar.c`，缺失了对.h文件的依赖关系）

由于这些缺点，在大型C/C++项目中一般不使用Make作为构建工具，而是使用[CMake](https://cmake.org/)、[Bazel](https://bazel.build/)、[Blade](https://github.com/chen3feng/blade-build)等更适合大型项目的构建工具。

## 5.参考
* [GNU Make手册](https://www.gnu.org/software/make/manual/make.html)
* [Make 命令教程](https://www.ruanyifeng.com/blog/2015/02/make.html)
* [Makefile文件教程](https://gist.github.com/isaacs/62a2d1825d04437c6f08)
* [Managing Projects with GNU Make, 3rd Edition](https://www.oreilly.com/library/view/managing-projects-with/0596006101/)
