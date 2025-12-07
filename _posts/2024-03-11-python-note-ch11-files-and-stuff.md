---
title: 《Python基础教程》笔记 第11章 文件
date: 2024-03-11 21:58:36 +0800
categories: [Python, Beginning Python]
tags: [python, file access, with statement, context manager]
---
本章将介绍文件和流，使你能够永久存储数据以及处理来自其他程序的数据。

## 11.1 打开文件
可以使用`open()`函数来打开文件。该函数将文件名作为唯一的必需参数，并返回一个文件对象。

假设在当前目录（命令行当前工作目录，`os.getcwd()`）中有一个名为somefile.txt的文本文件，可以这样打开它：

```python
>>> f = open('somefile.txt')
```

也可以指定完整路径。如果文件不存在则引发`FileNotFoundError`。如果要通过写入文本来**创建**文件，可以使用模式参数。

### 11.1.1 文件模式
模式参数`mode`可能的取值如下表所示。

| 值 | 描述 |
| --- | --- |
| `'r'` | 读模式（默认） |
| `'w'` | 写模式（先清空文件，如果文件不存在则创建） |
| `'x'` | 独占写入模式（如果文件已存在则失败） |
| `'a'` | 追加模式（如果文件存在则在末尾追加） |
| `'b'` | 二进制模式（添加到其他模式） |
| `'t'` | 文本模式（默认，添加到其他模式） |
| `'+'` | 读写模式（添加到其他模式） |

默认模式为`'r'`（或`'rt'`），这意味着将文件视为经过编码的Unicode文本，默认编码取决于具体平台（并不一定是UTF-8）。其他编码和Unicode错误处理策略可以使用关键字参数`encoding`和`errors`指定。

通常，Python使用**通用换行模式**(universal newline mode)，按行读取时（迭代或`readline()`）能够识别所有合法的换行符（`'\n'`、`'\r'`或`'\r\n'`）。还将自动转换换行符：读取时，自动将其他行尾替换为`'\n'`；写入时，将`'\n'`替换为系统默认行尾(`os.linesep`)。可以使用`newline`参数改变默认行为。

如果文件包含非文本的**二进制数据**，例如声音或图像，可以使用二进制模式（如`'rb'`）来禁用与文本相关的功能。

`'+'`可以添加到其他任何模式，表示既可读又可写。例如，要打开一个文本文件进行读写，可使用`'r+'`（可能需要结合使用`seek()`，详见本章后面的“随机访问”）。注意，`'r+'`与`'w+'`的区别：后者会清空(truncate)文件，而前者不会。

其他高级的可选参数详见官方文档[open()](https://docs.python.org/3/library/functions.html#open)或运行`help(open)`。

## 11.2 基本文件方法
本节介绍文件对象的一些基本方法以及其他类文件(file-like)对象（有时称为**流**(stream)）。

注：另见官方文档[io.TextIOBase](https://docs.python.org/3/library/io.html#io.TextIOBase)和[Reading and Writing Files](https://docs.python.org/3/tutorial/inputoutput.html#tut-files)。

#### 三个标准流
10.3.1节提到了三个标准流。这些流都是类文件对象，可以使用大部分文件操作。

标准输入流是`sys.stdin`。当程序从标准输入读取时（例如`input()`），可以通过键盘输入文本，也可以使用**管道**将其链接到其他程序的标准输出，这将在11.2.2节介绍。

标准输出流是`sys.stdout`，例如`print()`打印的文本和`input()`的提示信息。写入标准输出流的数据通常出现在屏幕上，但也可以通过管道将其重定向到其他程序的标准输入。

标准错误流是`sys.stderr`，用于输出错误消息（例如栈跟踪）。类似于`sys.stdout`，但可以单独重定向。

### 11.2.1 读取和写入
文件最重要的功能是读写数据。对于文件对象，可以使用`read()`方法读取数据，使用`write()`方法写入数据。对于文本和二进制模式，数据类型分别为`str`和`bytes`。

每次调用`write()`，提供的字符串都将写入到文件已有内容的后面。

注：`'w'`模式只在打开文件时清空，`write()`方法是追加写入，返回写入的字符数。

```python
>>> f = open('somefile.txt', 'w')
>>> f.write('Hello, ')
7
>>> f.write('World!')
6
>>> f.close()
```

注意，使用完文件后调用了`close()`方法。文件somefile.txt的内容为 "Hello, World!" （没有换行符）。

读取也一样简单。`read()`方法的参数指定至多读取的字符数，如果未指定则读取所有剩余内容，如果达文件末尾则返回空字符串。

```python
>>> f = open('somefile.txt', 'r')
>>> f.read(4)
'Hell'
>>> f.read()
'o, World!'
>>> f.read()
''
```

### 11.2.2 管道输出
在bash等shell中，可以使用**管道**(pipe)(`|`)将多个命令链接起来，例如：

```shell
$ cat somefile.txt | python somescript.py | sort
```

**管道将一个命令的标准输出链接到下一个命令的标准输入。** 因此，somescript.py从其标准输入读取数据（`cat`写入的），并将结果写入其标准输出（`sort`从这里获取数据）。

注：管道是操作系统的功能，而不是Python的。Windows也支持管道，但`cat`命令叫做`type`。

代码清单11-1所示的脚本统计标准输入中的单词数并打印出来。

[代码清单11-1 单词计数](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch11/wordcount.py)

将[somefile.txt](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch11/testdata/somefile.txt)作为输入，运行结果如下：

```shell
$ cat somefile.txt | python wordcount.py
Wordcount: 11
```

注：在shell中，可以分别使用运算符`<`、`>`和`2>`将标准输入、标准输出和标准错误重定向到文件。例如，上面的命令等价于`python wordcount.py < somefile.txt`。

#### 随机访问
在本章中，将文件都视为流，只能按顺序从头到尾读取。实际上，可以在文件中移动，只访问感兴趣的部分，称为**随机访问**(random access)。为此，使用文件对象的两个方法`seek()`和`tell()`。

方法`seek(offset)`将当前位置（执行读取或写入的位置）移到指定的地方，`offset`是字符（字节）数。例如：

```python
>>> f = open('somefile.txt', 'w')
>>> f.write('01234567890123456789')
20
>>> f.seek(5)
5
>>> f.write('Hello, World!')
13
>>> f.close()
>>> open('somefile.txt').read()
'01234Hello, World!89'
```

方法`tell()`返回当前位置。例如：

```python
>>> f = open('somefile.txt')
>>> f.read(3)
'012'
>>> f.read(2)
'34'
>>> f.tell()
5
```

### 11.2.3 读取和写入行
可以使用`readline()`方法读取一行（从当前位置到下一个换行符的文本），保留换行符。也可以指定一个非负整数参数，表示最多读取的字符数。`readlines()`方法读取文件中的所有行并以列表返回。

注：文件对象本身就可以按行迭代，见11.3.5节。

`writelines()`方法接受一个字符串列表（或任何可迭代对象），并将所有字符串都写入到文件（或流）。注意，写入时**不会**添加换行符，必须自行添加。

### 11.2.4 关闭文件
应记得调用`close()`方法关闭文件。通常，程序退出时将自动关闭文件对象，因此是否关闭**读取**的文件并不那么重要。然而，关闭文件并没有坏处，在有些操作系统中还可以避免无意义地锁定文件，另外还可以避免用完系统的文件打开配额。

一定要关闭**写入**的文件，因为Python可能会**缓冲**(buffer)你写入的数据。如果程序因某种原因崩溃，数据可能根本不会写入到文件中。安全的做法是，使用完文件后就将其关闭。

要确保文件被关闭，可以使用`try/finally`语句，并在`finally`子句中调用`close()`。

实际上，有一种专门为这种情况设计的语句——`with`语句。

```python
with expr [as target]:
    block
```

在语义上大致等价于

```python
manager = expr
hit_except = False

try:
    target = manager.__enter__()
    block
except:
    hit_except = True
    if not manager.__exit__(*sys.exc_info()):
        raise
finally:
    if not hit_except:
        manager.__exit__(None, None, None)
```

例如：

```python
with open("somefile.txt") as somefile:
    do_something(somefile)
```

该`with`语句打开文件并将其赋给变量`somefile`。在语句体中，将数据写入文件（还可能做其他事情）。到达该语句末尾时，将自动关闭文件，即便出现异常也是如此。

#### 上下文管理器
`with`语句实际上是一个非常通用的结构，允许你使用**上下文管理器**(context manager)。上下文管理器是支持两个方法的对象：`__enter__`和`__exit__`。
* `__enter__()`方法在进入`with`语句时被调用，其返回值被赋给`as`后面的变量。
* `__exit__()`方法在离开语句时被调用。

文件可以用作上下文管理器。其`__enter__()`方法返回文件对象本身，而`__exit__()`方法关闭文件。详见官方文档[The with statement](https://docs.python.org/3/reference/compound_stmts.html#with)和[With Statement Context Managers](https://docs.python.org/3/reference/datamodel.html#context-managers)。

### 11.2.5 使用基本文件方法
假设文件somefile.txt包含以下文本：

```
Welcome to this file
There is nothing here except
This stupid haiku
```

来试试前面介绍过的方法。

```python
>>> f = open('somefile.txt')
>>> f.read(7)
'Welcome'
>>> f.read(4)
' to '
>>> f.close()
```

```python
>>> f = open('somefile.txt')
>>> print(f.read())
Welcome to this file
There is nothing here except
This stupid haiku

>>> f.close()
```

```python
>>> f = open('somefile.txt')
>>> for i in range(3):
...     print(f'{i}: {f.readline()}', end='')
...
0: Welcome to this file
1: There is nothing here except
2: This stupid haiku
>>> f.close()
```

```python
>>> import pprint
>>> pprint.pprint(open('somefile.txt').readlines())
['Welcome to this file\n',
 'There is nothing here except\n',
 'This stupid haiku\n']
```

下面来尝试写入。

```python
>>> f = open('somefile.txt', 'w')
>>> f.write('this\nis no\nhaiku')
16
>>> f.close()
```

运行上述代码后，文件内容如下：

```
this
is no
haiku
```

```python
>>> f = open('somefile.txt')
>>> lines = f.readlines()
>>> f.close()
>>> lines[1] = "isn't a\n"
>>> f = open('somefile.txt', 'w')
>>> f.writelines(lines)
>>> f.close()
```

运行上述代码后，文件内容如下：

```
this
isn't a
haiku
```

## 11.3 迭代文件内容
一种常见的文件操作是迭代其内容，有很多方法可以做到。在本节的所有示例中都将使用一个虚构函数`process()`来表示对字符或行所做的处理。

### 11.3.1 按字符处理
一种最基本的文件内容迭代方式是在`while`循环中使用`read()`，按字符迭代。

```python
with open(filename) as f:
    while char := f.read(1):
        process(char)
```

这个程序之所以可行，是因为到达文件末尾时，`read()`方法返回一个空字符串（假），在此之前都返回只包含一个字符的字符串（真）。

### 11.3.2 按行处理
使用`while`循环和`readline()`方法可以迭代行。

```python
with open(filename) as f:
    while line := f.readline():
        process(line)
```

### 11.3.3 读取所有内容
如果文件不太大，可以使用无参数的`read()`方法或`readlines()`方法一次读取整个文件。

```python
with open(filename) as f:
    for char in f.read():
        process(char)
```

```python
with open(filename) as f:
    for line in f.readlines():
        process(line)
```

### 11.3.4 使用fileinput实现懒惰行迭代
有时需要迭代非常大的文件中的行，此时`readlines()`会占用太多内存。当然，可以使用`while`循环和`readline()`。但在Python中，应首选`for`循环，这里就是这种情况。可以使用`fileinput`实现懒惰行迭代（“懒惰”是因为每次只读取一行）。

```python
import fileinput
for line in fileinput.input(filename):
    process(line)
```

### 11.3.5 文件迭代器
下面是最酷（也是最常用）的方法。文件实际上是**可迭代**的，这意味着可以直接在`for`循环中迭代行。

```python
with open(filename) as f:
    for line in f:
        process(line)
```

如果让Python负责关闭文件，可以进一步简化这个示例。

```python
for line in open(filename):
    process(line)
```

注意，与其他文件一样，`sys.stdin`也是可迭代的，因此可以像这样迭代标准输入中的所有行：

```python
import sys
for line in sys.stdin:
    process(line)
```

另外，可以对迭代器做的事都可以对文件做，例如使用`list()`将其转换为字符串列表（等价于`readlines()`）。

```python
>>> f = open('somefile.txt', 'w')
>>> print('First', 'line', file=f)
>>> print('Second', 'line', file=f)
>>> print('Third', 'and final', 'line', file=f)
>>> f.close()
>>> lines = list(open('somefile.txt'))
>>> lines
['First line\n', 'Second line\n', 'Third and final line\n']
```

注：`print()`函数的`file`参数指定输出文件对象，默认为`sys.stdout`。
