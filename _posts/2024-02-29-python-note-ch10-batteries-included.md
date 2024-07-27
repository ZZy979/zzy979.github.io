---
title: 《Python基础教程》笔记 第10章 自带电池
date: 2024-02-29 22:00:08 +0800
categories: [Python, Beginning Python]
tags: [python, module, package, pip, command-line argument, file access, set, date and time, random number, json, regular expression]
math: true
---
Python不仅核心语言非常强大，还提供了更多工具以供使用。标准安装包含一组称为**标准库**(standard library)的模块。本章简要介绍模块的工作原理，然后概述标准库，重点介绍几个很有用的模块。

## 10.1 模块
你已经知道如何创建和执行程序（**脚本**），还知道如何使用`import`将函数从外部模块导入到程序中。下面看看如何编写自己的模块。

注：另见官方教程[Modules](https://docs.python.org/3/tutorial/modules.html)一节。

### 10.1.1 模块是程序
任何Python程序都可以作为**模块**(module)导入，文件名（除了扩展名`.py`）即为模块名。假设编写了如下程序，并将其保存在文件hello.py中。

```python
print('Hello, world!')
```

保存位置也很重要，详见10.1.3节。这里假设这个文件存储在目录C:\\python (Windows)或~/python (UNIX/macOS)。

要告诉解释器去哪里查找这个模块，可执行以下命令：

```python
>>> import sys
>>> sys.path.append(r'C:\python')
```

提示：在UNIX中，不能直接将字符串`'~/python'`添加到`sys.path`中，必须使用完整路径（如`'/home/yourusername/python'`）。如果希望自动扩展，可使用`os.path.expanduser('~/python')`。

之后就可以导入这个模块了。

```python
>>> import hello
Hello, world!
```

可以看到，导入模块时执行了其中的代码。但如果再次导入，什么事都不会发生。

```python
>>> import hello
>>>
```

因为模块并不是用来执行操作（例如打印文本）的，而是用于定义的（例如变量、函数、类等）。导入模块多次和导入一次的效果相同。

#### 为何只导入一次
在大多数情况下，只导入一次(import-only-once)是重要的优化，并且在一种特殊情况下尤为重要——两个模块相互导入。否则会导致无限循环（无穷递归）。

如果一定要重新加载模块，可以使用`importlib`模块中的`reload()`函数。

```python
>>> import importlib
>>> hello = importlib.reload(hello)
Hello, world!
```

### 10.1.2 模块用于定义
模块在首次被导入程序时执行。模块有自己的作用域，这意味着在模块中定义的类和函数以及赋值的变量都将成为**模块的属性**。

#### 在模块中定义函数
假设编写了如下模块，并存储在文件hello2.py中。将其放在Python解释器能够找到的地方。

```python
def hello():
    print('Hello, world!')
```

提示：Python解释器可以直接执行脚本（见1.9.1节），也可以使用`-m`选项来执行模块。命令`python -m progname args`将使用命令行参数`args`来执行模块`progname`，只要progname.py在Python模块搜索路径下。

可以导入该模块：

```python
>>> import hello2
```

模块会被执行，这意味着在这个模块的作用域内定义函数`hello()`，因此可以这样访问这个函数：

```python
>>> hello2.hello()
Hello, world!
```

使用模块的主要原因是**代码重用**(code reuse)。将代码放在模块中，就可以在多个程序/模块中使用它。

#### 在模块中添加测试代码
有时在模块中添加一些测试代码是很有用的。如代码清单10-4所示。

[代码清单10-4 带有条件测试代码的模块](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch10/hello4.py)

这里使用变量`__name__`避免在其他程序中将其作为模块导入时也执行测试代码。

```python
>>> __name__
'__main__'
>>> hello4.__name__
'hello4'
```

在主程序（包括交互式解释器）中，变量`__name__`的值是`'__main__'`。而在导入的模块中，它被设置为模块名。因此，可以将只需要在主程序中执行的代码放在语句`if __name__ == '__main__'`中。

另见官方文档[\_\_main\_\_ — Top-level code environment](https://docs.python.org/3/library/__main__.html)。

注意：如果要编写更详尽的测试代码，将其放在一个单独的程序中会更好。详见第16章。

### 10.1.3 让模块可用
**[sys.path](https://docs.python.org/3/library/sys.html#sys.path)是一个目录（用字符串表示）列表，Python解释器在这些目录中查找模块。** 要让你的模块可用，有两种方法：将模块放在正确的位置，或者告诉解释器去哪里查找。如果要让别人能够容易地使用你的模块，就是另外一回事了，可参考[Python Packaging User Guide](https://packaging.python.org/)。

#### 将模块放在正确的位置
要让解释器能够找到模块，可以将其放在`sys.path`中的任何一个目录中。

```python
>>> for p in sys.path: print(p)
...

D:\Python\Python311\python311.zip
D:\Python\Python311\Lib
D:\Python\Python311\DLLs
D:\Python\Python311
D:\Python\Python311\Lib\site-packages
```

其中，`D:\Python\Python311`是Python安装目录，`Lib`是Python标准库，`Lib\site-packages`是第三方库。

注：Python自带了包管理工具`pip`，用于安装第三方库。参考：
* [pip documentation](https://pip.pypa.io/)
* [Python Package Index](https://pypi.org/)
* [Installing Python Modules](https://docs.python.org/3/installing/index.html)
* [Installing Packages](https://packaging.python.org/en/latest/tutorials/installing-packages/)

另见[Python导入模块报错问题的分析]({% post_url 2020-11-05-python-import-error-analysis %})。

#### 告诉解释器去哪里查找
如果想将模块放在其他地方，就必须告诉解释器去哪里查找。可以直接修改`sys.path`，但标准做法是将模块所在目录包含在环境变量[PYTHONPATH](https://docs.python.org/3/using/cmdline.html#envvar-PYTHONPATH)中。

在Windows命令提示符(CMD)中，使用以下命令将目录 C:\\python 添加到环境变量PYTHONPATH末尾：

```shell
set PYTHONPATH=%PYTHONPATH%;C:\python
```

多个路径用分号分隔。这种方式设置的环境变量只在当前CMD窗口有效。如果要永久生效，右键点击此电脑 → 属性 → 高级系统设置 → 环境变量，新建或编辑PYTHONPATH变量。

在UNIX shell中：

```shell
export PYTHONPATH=$PYTHONPATH:~/python
```

多个路径用冒号分隔。要让环境变量对所有shell会话生效，可以将该命令添加到主目录中的.bashrc文件中。

除此之外，还可以使用路径配置文件(.pth)，详见[site模块文档](https://docs.python.org/3/library/site.html)。

### 10.1.4 包
为了组织模块，可以将其分组为**包**(package)。包本质上是另一种模块，但可以包含其他模块。模块是一个文件(.py)，而包是一个目录。

要被Python视为包，目录必须包含文件 **\_\_init\_\_.py** 。如果像普通模块一样导入包，该文件的内容就是包的内容。例如，如果有一个名为`constants`的包，文件constants/\_\_init\_\_.py包含语句`PI = 3.14`，就可以像下面这样做：

```python
import constants
print(constants.PI)
```

**要将模块放在包中，只需将模块文件放在包目录中。** 还可以在包中嵌套其他包。例如，要创建一个名为`drawing`的包，其中包含模块`shapes`和`colors`，需要创建以下文件和目录：

```
~/python/            # PYTHONPATH中的目录
    drawing/         # 包目录（drawing包）
        __init__.py  # 包代码（drawing模块）
        colors.py    # colors模块
        shapes.py    # shapes模块
```

[包示例](https://github.com/ZZy979/Beginning-Python-code/tree/main/ch10/drawing)

按照这个设置，下面的语句都是合法的：

```python
import drawing             # (1) Imports the drawing package
import drawing.colors      # (2) Imports the colors module
from drawing import shapes # (3) Imports the shapes module
```

* 执行第1条语句后，文件drawing/\_\_init\_\_.py的内容是可用的，但`shapes`和`colors`模块不可用（除非\_\_init\_\_.py包含类似`from . import colors`这样的语句）。
* 执行第2条语句后，`colors`模块可用，但只能通过全名(`drawing.colors`)来使用。
* 执行第3条语句后，`shapes`模块可用，可以通过全名或短名(`shapes`)来使用。

注意，不必先导入包再导入其中的模块，可以直接导入包中的模块。

## 10.2 探索模块
在介绍一些标准库模块前，先向你展示如何自行探索模块。这是一种很有价值的技能，因为在你的Python程序员职业生涯中，会遇到很多有用的模块，而这里无法一一介绍。

### 10.2.1 模块中有什么
要探索模块，最直接的方式是使用Python解释器进行研究。首先要将模块导入。假设你听说过有一个名为`copy`的标准库模块。

```python
>>> import copy
```

#### 使用dir
要查看模块包含的内容，可以使用`dir()`函数，它列出对象的所有属性（对于模块，列出所有的函数、类、变量等）。`dir(copy)`包含以下划线开头的名字，（按照惯例）意味着它们并非供外部使用。因此使用一个简单的列表推导式将这些名称过滤掉。

```python
>>> [n for n in dir(copy) if not n.startswith('_')]
['Error', 'copy', 'deepcopy', 'dispatch_table', 'error']
```

#### 变量\_\_all\_\_
变量`__all__`包含一个列表，与`dir()`类似，但它是在模块内部设置的。

```python
>>> copy.__all__
['Error', 'copy', 'deepcopy']
```

它在`copy`模块([copy.py](https://github.com/python/cpython/blob/main/Lib/copy.py))中就是这样设置的：

```python
__all__ = ["Error", "copy", "deepcopy"]
```

该变量旨在定义模块的公共接口。更具体地说，它告诉解释器从这个模块导入所有名称(`from module import *`)意味着什么。

编写模块时，像这样设置`__all__`很有用。如果不设置`__all__`，则`import *`默认导入所有不以下划线开头的全局名称。

### 10.2.2 使用help获取帮助
标准函数`help()`可以提供通常需要的所有信息。

```python
>>> help(copy.copy)
Help on function copy in module copy:

copy(x)
    Shallow copy operation on arbitrary Python objects.

    See the module's __doc__ string for more info.
```

帮助信息指出，`copy()`只接受一个参数`x`，是“浅拷贝操作”。`__doc__`是第6章提到的文档字符串。文档字符串就是在函数、模块或类开头编写的字符串，可以通过`__doc__`属性访问。实际上，前面的帮助信息就是从`copy()`函数的文档字符串(`copy.copy.__doc__`)提取的。

尝试直接对模块本身调用`help()`。这将打印大量的信息，包括对`copy()`和`deepcopy()`之间区别的详细讨论（本质上，`deepcopy(x)`创建`x`的属性值的副本，以此类推；而`copy(x)`只拷贝`x`，并将副本的属性绑定到`x`的属性值）。

### 10.2.3 文档
有关模块/类/函数信息的自然来源是其文档。例如，如果想知道“`range`的参数是什么”，不用搜索标准Python文档，而可以直接查看：

```python
>>> print(range.__doc__)
range(stop) -> range object
range(start, stop[, step]) -> range object

Return an object that produces a sequence of integers from start (inclusive)
to stop (exclusive) by step.  range(i, j) produces i, i+1, i+2, ..., j-1.
start defaults to 0, and stop is omitted!  range(4) produces 0, 1, 2, 3.
These are exactly the valid indices for a list of 4 elements.
When step is given, it specifies the increment (or decrement).
```

就学习Python编程而言，最有用的文档是“[Python标准库参考](https://docs.python.org/3/library/index.html)”。其他标准文档都可以在Python网站(<https://docs.python.org/3/index.html>)上找到。

### 10.2.4 使用源代码
事实上，除了自己编写代码外，阅读源代码是学习Python最好的方式。

模块的`__file__`属性是源代码位置。

```python
>>> print(copy.__file__)
D:\Python\Python311\Lib\copy.py
```

注：
* 如果使用IDLE，也可以通过菜单File → Open Module...打开模块源代码（千万不要保存任何所做的修改）。
* GitHub上的Python标准库源代码：<https://github.com/python/cpython/tree/main/Lib>

注意，有些模块没有Python源代码。它们可能是解释器内置的（例如`sys`模块），或者可能是用C语言编写的（参见第17章）。

## 10.3 标准库：一些最爱
在Python中，短语“自带电池”([Batteries Included](https://docs.python.org/3/tutorial/stdlib.html#batteries-included))指的是Python丰富的标准库。安装Python后，你就“免费”获得了大量有用的模块（“电池”）。这里只介绍几个我喜欢的标准模块，以激发你的探索兴趣。

完整参考手册：[Python Module Index](https://docs.python.org/3/py-modindex.html)

### 10.3.1 sys
`sys`模块让你能够访问与Python解释器紧密相关的变量和函数，下表列出了其中的一些。

| 函数/变量 | 描述 |
| --- | --- |
| `argv` | 命令行参数，包括脚本名 |
| `exit([arg])` | 退出当前程序，可选参数指定返回值或错误信息 |
| `modules` | 将模块名映射到已加载模块的字典 |
| `path` | 模块查找目录的列表 |
| `platform` | 平台标识符，例如`win32`或`linux` |
| `stdin`、`stdout`、`stderr` | 标准输入/输出/错误流，一个类似文件(file-like)的对象 |

注：除了`sys.platform`，也可以通过`os.name`或`platform`模块访问操作系统平台信息。

从命令行调用Python脚本时，可以指定一些参数——所谓的**命令行参数**(command-line argument)（例如`python prog.py arg1 arg2 ...`）。这些参数将放在列表`sys.argv`中，其中`sys.argv[0]`是Python脚本名。下面的程序按相反的顺序打印命令行参数。

[代码清单10-5 反序打印命令行参数](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch10/reverseargs.py)

```shell
$ python reverseargs.py this is a test
test a is this
```

在上面的示例中，`sys.argv == ['reverseargs.py', 'this', 'is', 'a', 'test']`。

### 10.3.2 os
`os`模块让你能够访问多个操作系统服务。该模块的内容很多，下表只描述了其中几个最有用的函数和变量。

| 函数/变量 | 描述 |
| --- | --- |
| `environ` | 包含环境变量的映射 |
| `system(command)` | 在子shell中执行命令 |
| `sep` | 路径中使用的分隔符（`'/'`或`'\\'`） |
| `linesep` | 行分隔符（`'\n'`、`'\r'`或`'\r\n'`） |

除此之外，`os`及其子模块`os.path`还包含一些查看、创建和删除目录和文件的函数，以及一些操作路径的函数（例如`os.path.join()`和`os.path.split()`）。另外，`pathlib`模块提供了面向对象的路径操作接口。

提示：`subprocess`模块融合了`os.system()`、`execv()`和`popen()`函数的功能。

### 10.3.3 fileinput
第11章将深入介绍如何读写文件，这里先做个预览。`fileinput`模块让你能够轻松地遍历一系列文本文件中的所有行。

下表描述了`fileinput`模块最重要的函数。

| 函数 | 描述 |
| --- | --- |
| `input([files[, inplace[, backup]]) ` | 迭代多个文件中的行 |
| `filename()` | 返回当前文件名 |
| `lineno()` | 返回当前（累计的）行号 |
| `filelineno()` | 返回在当前文件中的行号 |
| `isfirstline()` | 检查当前行是否是文件的第一行 |
| `isstdin()` | 检查上一行是否来自`sys.stdin` |
| `nextfile()` | 关闭当前文件并移到下一个文件 |
| `close()` | 关闭序列 |

`fileinput.input()`是其中最重要的函数，它返回一个可迭代对象（每次返回一行，保留换行符）。
* 参数`files`指定要迭代的文件名序列，如果只有一个文件可用字符串表示，`'-'`表示`stdin`。如果未提供`files`参数，则文件名序列由命令行参数`sys.argv[1:]`指定；如果命令行参数也为空则使用`stdin`。
* 如果参数`inplace`为`True`，则进行原地处理。对于访问的每一行，都需要打印出替代内容，这些内容将被写回到当前输入文件中（注：这是通过临时将`sys.stdout`替换为文件对象实现的，从而`print()`不会打印到屏幕，而是输出到文件）。
* 进行原地处理时，可选参数`backup`用于给从原始文件创建的备份文件指定扩展名（默认为.bak）。

注：
* `fileinput.input()`函数类似于UNIX命令`cat`，将多个文件拼接起来并逐行迭代。
* `fileinput.input()`函数创建了一个全局的`FileInput`对象，其他函数直接操作这个全局对象，因此这些函数都是“有状态的”。

下面的示例给一个Python脚本添加行号（在每行末尾添加注释）。

[代码清单10-6 给Python脚本添加行号](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch10/numberlines.py)

如果对程序自身运行这个程序：

```shell
$ python numberlines.py numberlines.py
```

程序将变成下面这样。

```
import fileinput                                   #  1
                                                   #  2
for line in fileinput.input(inplace=True):         #  3
    line = line.rstrip()                           #  4
    num = fileinput.lineno()                       #  5
    print('{:<50} # {:2d}'.format(line, num))      #  6
```

10.3.6节提供了另一个使用`fileinput`的示例。

### 10.3.4 集合、堆和双端队列
#### 集合
**集合**(set)是**无重复**的可散列对象组成的**无序**集，由内置类`set`实现。

可以使用`set()`函数从序列（或其他可迭代对象）创建集合，也可以使用花括号显式地指定。

```python
>>> set(range(10))
{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
>>> {1, 2, 3}
{1, 2, 3}
```

注：也可以使用集合推导式，例如`{x * x for x in range(10)}`

注意，`{}`是空字典，必须使用`set()`创建空集合。

```python
>>> type({})
<class 'dict'>
>>> set()
set()
```

集合主要用于成员资格检查，因此重复的元素将被忽略：

```python
>>> {0, 1, 2, 3, 0, 1, 2, 3, 4, 5}
{0, 1, 2, 3, 4, 5}
```

和字典一样，集合元素的顺序是不确定的，因此不能依赖这一点。

除了成员资格检查外，还可以执行各种标准集合操作：
* `len(s)` 返回`s`中的元素数量（基数(cardinality) $\vert S \vert$ ）
* `x in s` 检查`s`是否包含`x` ($x \in S$)
* `a & b`或`a.intersection(b)` 交集($A \cap B$)
* `a | b`或`a.union(b)` 并集($A \cup B$)
* `a - b`或`a.difference(b)` 差集($A - B$)
* `a ^ b`或`a.symmetric_difference(b)` 对称差($A \triangle B = A \cup B - A \cap B$)
* `a <= b`或`a.issubset(b)` 子集($A \subseteq B$)
* `a < b` 真子集($A \subsetneq B$)
* `a >= b`或`a.issuperset(b)` 超集($A \supseteq B$)
* `a > b` 真超集($A \supsetneq B$)

注：
* 运算符的操作数必须是集合，而方法的参数可以是任何可迭代对象。
* 这些操作支持多个集合，例如`a | b | c`或`a.union(b, c)`。

```python
>>> a = {1, 2, 3}
>>> b = {2, 3, 4}
>>> c = {2, 3}
>>> a | b
{1, 2, 3, 4}
>>> a & b
{2, 3}
>>> a - b
{1}
>>> a ^ b
{1, 4}
>>> c <= a
True
>>> c >= a
False
>>> a.copy()
{1, 2, 3}
>>> a.copy() is a
False
```

另外，还有一些原地运算符和对应的方法（例如`|=`和`update()`）以及基本方法`add()`、`remove()`和`discard()`。详见官方文档[class set](https://docs.python.org/3/library/stdtypes.html#set)。

提示：可以使用`union()`方法的未绑定版本来计算两个集合的并集（`set.union(a, b)`等价于`a.union(b)`）。这可能很有用，例如与`reduce()`一起使用。

```python
>>> my_sets = [{7, 1, 2}, {2, 4}, {5, 4, 1, 8}, {9, 2}]
>>> from functools import reduce
>>> reduce(set.union, my_sets)
{1, 2, 4, 5, 7, 8, 9}
```

集合是可变的，因此不能用作字典中的键。另一个问题是，集合只能包含不可变（可散列）的值，因此不能包含其他集合。所幸有`frozenset`类型，表示**不可变**（可散列）的集合。

```python
>>> { {1, 2}, {3, 4} }
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unhashable type: 'set'
>>> {frozenset({1, 2}), frozenset({3, 4})}
{frozenset({3, 4}), frozenset({1, 2})}
```

#### 堆
另一种著名的数据结构是**堆**(heap)，它是一种优先队列(priority queue)。优先队列能够以任意顺序添加元素，并随时找到（并删除）最小的元素。堆比对列表使用`min()`高效得多（前者的时间复杂度为O(log n)，后者为O(n)）。

实际上，Python并没有独立的堆类型，只有一个包含堆操作函数的模块`heapq`（其中`q`表示队列(queue)），使用列表表示堆对象。

| 函数 | 描述 |
| --- | --- |
| `heappush(heap, x)` | 将`x`压入堆中 |
| `heappop(heap)` | 弹出堆中最小的元素 |
| `heapify(heap)` | 将任意列表转换为堆 |
| `heapreplace(heap, x)` | 弹出最小的元素，并压入`x` |
| `nlargest(n, iter, key=None)` | 返回`iter`中`n`个最大的元素 |
| `nsmallest(n, iter, key=None)` | 返回`iter`中`n`个最小的元素 |

```python
>>> from heapq import *
>>> heap = [5, 8, 0, 3, 6, 7, 9, 1, 4, 2]
>>> heapify(heap)
>>> heap
[0, 1, 5, 3, 2, 7, 9, 8, 4, 6]
>>> heappush(heap, 0.5)
>>> heap
[0, 0.5, 5, 3, 1, 7, 9, 8, 4, 6, 2]
```

元素虽然不是严格排序的，但必须保证一点：`a[i] <= a[2*i+1]`且`a[i] <= a[2*i+2]`。这是底层堆算法的基础，称为堆的性质。

注：实现原理详见[heapq - Theory](https://docs.python.org/3/library/heapq.html#theory)。

`heappop()`函数弹出最小的元素（总是位于索引0处），并确保剩余元素中最小的那个位于索引0处（保持堆的性质）。

```python
>>> heappop(heap)
0
>>> heappop(heap)
0.5
>>> heappop(heap)
1
>>> heap
[2, 3, 5, 4, 6, 7, 9, 8]
```

`heapreplace()`函数从堆中弹出最小的元素，再压入一个新元素。它比依次执行`heappop()`和`heappush()`的效率更高。

```python
>>> heapreplace(heap, 10)
2
>>> heap
[3, 4, 5, 8, 6, 7, 9, 10]
```

#### 双端队列
**双端队列**(double-ended queue, deque)在需要按添加的顺序删除元素时很有用。`collections`模块包含`deque`类型以及其他几个集合(collection)类型。

| 类型 | 描述 |
| --- | --- |
| `deque` | 双端队列 |
| `Counter` | 字典子类，用于计数对象 |
| `OrderedDict` | 字典子类，能记住键值对插入顺序 |
| `defaultdict` | 字典子类，为键提供默认值 |

`deque`是从可迭代对象创建的，包含多个有用的方法。

```python
>>> from collections import deque
>>> q = deque(range(5))
>>> q.append(5)
>>> q.appendleft(6)
>>> q
deque([6, 0, 1, 2, 3, 4, 5])
>>> q.pop()
5
>>> q.popleft()
6
>>> q.rotate(3)
>>> q
deque([2, 3, 4, 0, 1])
>>> q.rotate(-1)
>>> q
deque([3, 4, 0, 1, 2])
>>> q.extend([5, 6, 7])
>>> q.extendleft([8, 9, 10])
>>> q
deque([10, 9, 8, 3, 4, 0, 1, 2, 5, 6, 7])
```

双端队列支持在队首（左端）高效地添加和弹出元素，而列表无法这样做。

### 10.3.5 time
`time`模块包含用于获取当前时间、操作和格式化日期和时间的函数。

日期和时间可表示为
* 实数：从“纪元”(epoch)开始经过的秒数（时间戳），纪元是1970-1-1 00:00:00 (UTC)。例如，3600表示UTC时间1970-1-1 01:00:00或北京时间1970-1-1 09:00:00。
* 9个整数的元组：`(year, month, day, hour, minute, second, weekday, year_day, daylight_savings)`。例如，`(2008, 1, 21, 12, 2, 56, 0, 21, 0)`表示2008年1月21日12:02:56，星期一，当年的第21天，非夏令时。

`time`模块最重要的函数如下表所示。

| 函数 | 描述 |
| --- | --- |
| `time()` | 当前时间（从纪元开始的秒数，UTC时间） |
| `asctime([tuple])` | 将时间元组转换为字符串 |
| `gmtime([secs])` | 将秒数转换为时间元组（UTC时间） |
| `localtime([secs])` | 将秒数转换为时间元组（本地时间） |
| `mktime(tuple)` | 将时间元组（本地时间）转换为秒数 |
| `strptime(string[, format])` | 将字符串解析为时间元组 |
| `strftime(format[, tuple])` | 将时间元组格式化为字符串 |
| `sleep(s)` | 休眠（什么都不做）`s`秒 |

注：该模块类似于C语言的[\<time.h\>](https://en.cppreference.com/w/c/chrono)。

另外，还有两个较新的与时间相关的模块：`datetime`（支持日期和时间算术）和`timeit`（对代码段进行计时），详见官方文档。

### 10.3.6 random
`random`模块包含生成伪随机数的函数，有助于编写模拟程序或生成随机输出的程序。

注意：伪随机数看起来好像是完全随机的，但背后的系统是可预测的。如果需要真随机（例如用于密码或安全相关的功能），应使用`os`模块的`urandom()`函数或`random`模块的`SystemRandom`类。

下表列出了这个模块中一些重要的函数。

| 函数 | 描述 |
| --- | --- |
| `random()` | 返回[0, 1)内的随机实数 |
| `randrange([start, ]stop[, step])` | 返回`range(start, stop, step)`中的随机数 |
| `randint(a, b)` | 返回[a, b]内的随机整数 |
| `choice(seq)` | 返回序列`seq`中的随机元素 |
| `choices(seq, weights=None, n=1)` | 从`seq`中随机选择`n`个元素（有重复） |
| `sample(seq, n)` | 从`seq`中随机选择`n`个不重复的元素 |
| `shuffle(seq)` | 原地打乱序列`seq` |

编写与统计学相关的程序时，可以使用返回各种分布随机数的函数。

| 函数 | 分布名称 |
| --- | --- |
| `uniform(a, b)` | 均匀分布 |
| `binomialvariate(n=1, p=0.5)` | 二项分布 |
| `normalvariate(mu=0.0, sigma=1.0)` | 正态分布 |
| `expovariate(lambd=1.0)` | 指数分布 |
| `betavariate(alpha, beta)` | 贝塔分布 |

下面的示例生成2016年内的随机时间：

[随机时间](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch10/random_time.py)

将时间元组的后三项（星期、儒略日和夏令时）设置为-1，让Python去计算它们的正确值。

在下一个示例中，询问用户要掷多少个骰子、每个骰子有多少面，输出骰子点数总和。掷骰子机制是使用`randrange()`和`for`循环实现的。

[掷骰子](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch10/throw_dice.py)

现在假设创建了一个文本文件，其中每行包含一句谚语（作者原文 "fortune" 想表达的应该不是“财富”，而是“改变人生的机遇”）。下面的示例使用`fileinput`模块将其放入一个列表中，再随机选择一个。

[随机谚语](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch10/random_fortune.py)

```shell
$ python random_fortune.py fortunes.txt
A journey of a thousand miles begins with a single step.
```

最后一个示例，用户每次按回车键都发一张牌，同时确保每张牌都不同。

[随机发牌](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch10/deal_cards.py)

### 10.3.7 shelve和json
下一章将介绍如何将数据存储到文件中。但如果只需要非常简单的存储方案，`shelve`模块可以完成大部分工作——只需提供一个文件名即可。对于`shelve`模块，唯一感兴趣的函数是`open()`。该函数将文件名作为参数，并返回一个`Shelf`对象，可用于存储数据。可以把它当作普通字典（只是键必须是字符串）。操作完成后，调用其`close()`方法（保存到磁盘）。

#### 潜在的陷阱
很重要的一点是认识到`shelve.open()`返回的对象并不是普通映射，如下例所示：

```python
>>> import shelve
>>> s = shelve.open('test.dat')
>>> s['x'] = [1, 2, 3]
>>> s['x'].append(4)
>>> s['x']
[1, 2, 3]
```

要正确地修改使用`shelve`模块存储的对象，必须将获取的副本赋给一个临时变量，并在修改后再次存储：

```python
>>> temp = s['x']
>>> temp.append(4)
>>> s['x'] = temp
>>> s['x']
[1, 2, 3, 4]
```

还有另一种避免这个问题的方式：将`open()`函数的`writeback`参数设置为`True`。这样，从`Shelf`对象读取或赋值的所有数据结构都将缓存到内存中，并且只有在关闭`Shelf`对象时才写回磁盘。如果处理的数据量不大，且不想操心这些问题，那么这是个不错的方法。必须确保在处理完毕后关闭`Shelf`对象。为此，一种方法是将`Shelf`对象用作上下文管理器，这将在下一章讨论。

#### 简单的数据库示例
代码清单10-8展示了一个使用`shelve`模块的简单数据库应用。

[代码清单10-8 简单的数据库应用](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch10/database.py)

提示：如果希望以其他语言编写的程序能够轻松读取的格式保存数据，可以使用[JSON](https://json.org/)格式。Python标准库提供了`json`模块来处理JSON字符串以及与Python值之间相互转换（JSON对象↔Python字典，JSON数组↔Python列表）。

### 10.3.8 re
> Some people, when confronted with a problem, think, "I know, I'll use regular expressions." Now they have two problems.
>
> ——Jamie Zawinski

`re`模块提供了对正则表达式的支持。

提示：除了标准文档外，文章[Regular Expression HOWTO](https://docs.python.org/3/howto/regex.html)也是很有用的Python正则表达式学习资源。

#### 正则表达式是什么
**正则表达式**(regular expressions)（也叫regex或regexp）是可以匹配文本片段的模式。可用于在文本中查找模式，将特定模式替换为计算后的值，以及将文本分割成片段。

最简单的正则表达式就是普通字符串，匹配其自身。例如，正则表达式`python`匹配字符串 "python" 。

注意：在这里，术语**匹配**是指与整个字符串匹配，而`re.match()`函数只要求模式与字符串开头匹配。

（1）通配符

句点`.`匹配任意字符（换行符除外），叫做**通配符**(wildcard)。例如，正则表达式`.ython`匹配字符串 "python" 、 "jython" 和 " ython" ，但不匹配 "cpython" 和 "ython" 。

（2）转义特殊字符

要让特殊字符的行为与普通字符一样，可以对其进行**转义**(escape)：在前面加上反斜杠。例如，`python\.org`匹配 "python.org" 。

注意，为了表示正则表达式的单个反斜杠，需要在字符串中写两个反斜杠：`'python\\.org'`。这里包含两层转义：解释器的转义和正则表达式的转义。如果厌烦了双反斜杠，可以使用原始字符串，例如`r'python\.org'`。

（3）字符集

可以使用方括号创建**字符集**(character set)。字符集匹配它包含的任何一个字符。例如，`[pj]ython`匹配 "python" 和 "jython" ，不匹配其他字符串。

也可以使用**范围**(range)，例如`[a-z]`匹配a到z的任何字符。还可以组合范围，例如`[a-zA-Z0-9]`匹配大写字母、小写字母和数字。注意，字符集只能匹配**一个**字符。

注：特殊字符`\d`匹配数字，`\w`匹配字母、数字或下划线，`\s`匹配空白符。

要反转字符集，可以在开头添加`^`。例如，`[^abc]`匹配除a、b和c外的任何字符。

字符集中的特殊字符：在字符集中，通常无需对特殊字符进行转义（尽管是完全合法的）。然而，应该记住以下规则：
* `^`位于字符集开头时需要转义，除非表示否定运算符。
* `]`和`-`要么放在字符集开头，要么转义。`-`也可以放在字符集末尾。

（4）选择和子模式

`|`表示**选择**(alternatives)，即匹配两个模式之一。例如，`python|perl`只匹配 "python" 和 "perl" 。

如果只想将选择运算符用于模式的一部分，可以将其放在圆括号内，称为**子模式**(subpattern)。前面的例子可以写成`p(ython|erl)`。

（5）可选和重复模式

在模式后面加上`?`表示可选的。例如，`(http://)?(www\.)?python\.org`匹配以下字符串之一：

```
http://www.python.org
http://python.org
www.python.org
python.org
```

其他重复运算符：
* `*`：重复零次或多次
* `+`：重复一次或多次
* `{m,n}`：重复m到n次

（6）字符串的开头和末尾

`^`表示字符串开头，`$`表示字符串末尾。例如，`^python`匹配 "python.org" ，但不匹配 "http://python.org" 。

注：完整的正则表达式运算符列表参考[Regular Expression Syntax](https://docs.python.org/3/library/re.html#regular-expression-syntax)。

#### re模块的内容
`re`模块包含一些使用正则表达式的函数，如下表所示。

| 函数 | 描述 |
| --- | --- |
| `compile(pattern[, flags])` | 从正则表达式字符串创建模式对象 |
| `search(pattern, string)` | 在字符串中查找模式 |
| `match(pattern, string)` | 在字符串开头匹配模式 |
| `fullmatch(pattern, string)` | 将整个字符串匹配模式 |
| `split(pattern, string[, maxsplit])` | 根据模式分割字符串 |
| `findall(pattern, string)` | 返回模式在字符串中所有匹配的列表 |
| `finditer(pattern, string)` | 返回模式在字符串中所有匹配的迭代器 |
| `sub(pat, repl, string[, count])` | 将字符串中模式的所有匹配替换为`repl` |
| `escape(string)` | 转义字符串中所有的正则表达式特殊字符 |

函数`re.compile()`将正则表达式字符串转换为模式对象`Pattern`，多次使用时可以提高匹配效率。`re`模块普通函数的模式参数既接受字符串也接受模式对象。`re.match(pattern, string)`等价于`re.compile(pattern).match(string)`。

函数`re.search()`在给定字符串中查找第一个与指定正则表达式匹配的子串。如果找到则返回一个`Match`对象（值为真），否则返回`None`（值为假）。因此可以这样使用：

```python
if re.search(pat, string):
    print('Found it!')
```

`Match`对象将在下一节介绍。

函数`re.match()`尝试在给定字符串开头匹配正则表达式（不要求与整个字符串匹配）。

```python
>>> import re
>>> bool(re.search('th', 'python'))
True
>>> bool(re.match('python', 'python.org'))
True
>>> bool(re.match('python', 'www.python.org'))
False
>>> bool(re.fullmatch(r'python\.org', 'python.org'))
True
```

函数`re.split()`根据模式的匹配项来分割字符串，`maxsplit`参数指定最多分隔的次数（默认不限制）。

```python
>>> some_text = 'alpha, beta,,,,gamma    delta'
>>> re.split('[, ]+', some_text)
['alpha', 'beta', 'gamma', 'delta']
>>> re.split('[, ]+', some_text, maxsplit=2)
['alpha', 'beta', 'gamma    delta']
>>> re.split('[, ]+', some_text, maxsplit=1)
['alpha', 'beta,,,,gamma    delta']
>>> re.split('o(o)', 'foobar')
['f', 'o', 'bar']
```

函数`re.findall()`返回给定模式所有匹配项的列表。例如，要找出字符串中的所有单词，可以这样做：

```python
>>> pat = '[a-zA-Z]+'
>>> text = '"Hm... Err -- are you sure?" he said, sounding insecure.'
>>> re.findall(pat, text)
['Hm', 'Err', 'are', 'you', 'sure', 'he', 'said', 'sounding', 'insecure']
```

或者查找标点符号：

```python
>>> pat = r'[.?\-",]+'
>>> re.findall(pat, text)
['"', '...', '--', '?"', ',', '.']
```

函数`re.sub()`从左往右将模式的非重叠匹配替换为指定内容。例如：

```python
>>> re.sub('aaa', 'AAA', 'aaaabbbbaaaa')
'AAAabbbbAAAa'
```

“替换中的组号和函数”一节将介绍如何更有效地使用这个函数。

函数`re.escape()`对字符串中所有可能被解释为正则表达式运算符的字符进行转义。例如，字符串很长而不想输入大量的反斜杠，或者要将来自用户的字符串用作正则表达式。

#### 匹配对象和组
`re`模块的模式匹配函数都在找到匹配项时返回`Match`对象。这些对象包含与模式匹配的子串的信息，即模式的哪部分匹配子串的哪部分。这些“部分”叫做**组**(group)。

组就是圆括号内的子模式。组是根据左括号的出现顺序编号的，组0就是整个模式。例如，在下面的模式中：

```
There (was a (wee) (cooper)) who (lived in Fyfe)
```

包含这些组：

```
0 There was a wee cooper who lived in Fyfe
1 was a wee cooper
2 wee
3 cooper
4 lived in Fyfe
```

通常，组中包含通配符等特殊字符。例如，在模式`www\.(.+)\.com`中，组1包含 "www." 和 ".com" 之间的内容。通过创建类似这样的模式，可以提取字符串中感兴趣的部分。

下表描述了`Match`对象的一些重要方法。

| 方法 | 描述 |
| --- | --- |
| `group(g=0)`或`m[g]` | 返回给定组匹配的子串 |
| `groups()` | 返回所有组（不包括0）匹配的子串构成的元组 |
| `start(g=0)` | 返回给定组匹配子串的开始位置 |
| `end(g=0)` | 返回给定组匹配子串的结束位置（不包含） |
| `span(g=0)` | 返回给定组匹配子串的开始和结束位置 |

注意：除了整体匹配（组0）外，最多只能有99个组，编号1-99。

```python
>>> m = re.match(r'www\.(.+)\.(.{3})', 'www.python.org')
>>> m.groups()
('python', 'org')
>>> m.group(1)
'python'
>>> m.start(1)
4
>>> m.end(1)
10
>>> m.group(2)
'org'
>>> m.start(2)
11
>>> m.end(2)
14
```

#### 替换中的组号和函数
`re.sub()`的替换字符串中可以使用组号，例如`\0`、`\1`等。

例如，假设要将纯文本文档表示的强调`*something*`替换为相应的HTML代码`<em>something</em>`。

```python
>>> emphasis_pattern = r'\*([^\*]+)\*'
>>> re.sub(emphasis_pattern, r'<em>\1</em>', 'Hello, *world*!')
'Hello, <em>world</em>!'
```

**用函数作为替换内容**可以使替换功能更加强大。该函数将`Match`对象作为唯一参数，返回的字符串将用作替换内容。“模板系统示例”一节将介绍一种用途。

贪婪和懒惰模式：重复运算符默认是**贪婪**(greedy)的，这意味着它会匹配尽可能多的内容。假设前面的示例使用模式`'\*(.+)\*'`：

```python
>>> emphasis_pattern = r'\*(.+)\*'
>>> re.sub(emphasis_pattern, r'<em>\1</em>', '*This* is *it*!')
'<em>This* is *it</em>!'
```

显然这不是想要的结果。在这种情况下，只需使用重复运算符的非贪婪版本：在后面加上`?`。

```python
>>> emphasis_pattern = r'\*(.+?)\*'
>>> re.sub(emphasis_pattern, r'<em>\1</em>', '*This* is *it*!')
'<em>This</em> is <em>it</em>!'
```

#### 找出发件人
如果将邮件保存为文本文件，会发现开头有大量难以理解的文本，如下所示。

[一组邮件头](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch10/testdata/message.eml)

下面尝试找出这封邮件的发送人。包含发件人的行以`'From: '`开头，以尖括号（`<`和`>`）内的邮件地址结尾。要提取的是尖括号前的文本。解决这个问题的程序如代码清单10-10所示。

注意：这个问题也可以不使用正则表达式解决，可以使用`email`模块。

[代码清单10-10 查找发件人](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch10/find_sender.py)

```shell
$ python find_sender.py message.eml
Foo Fie
```

下面的程序列出邮件头中的所有邮件地址。

[列出邮件地址](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch10/list_email_addresses.py)

```shell
$ python list_email_addresses.py message.eml
Mr.Gumby@bar.baz
foo@bar.baz      
foo@baz.com      
magnus@bozz.floop
```

#### 模板系统示例
**模板**(template)是一种包含占位符的文件，可以放入具体的值来得到最终的文本。Python已经提供了一种高级模板机制：字符串格式化。然而，使用正则表达式可以使模板更加高级。

假设要把所有的`[expr]`（“字段”）都替换为将`expr`作为Python表达式求值的结果。例如，模板

```
The sum of 7 and 9 is [7 + 9].
```

应转换为

```
The sum of 7 and 9 is 16.
```

另外，还希望能够在字段中进行赋值，从而模板

```
[name="Mr. Gumby"]Hello, [name]
```

应转换为

```
Hello, Mr. Gumby
```

这看起来很复杂，但是看看可用的工具：
* 可以使用正则表达式来匹配字段并提取其内容。
* 可以使用`eval()`求值表达式字符串，并提供包含作用域的字典。
* 可以使用`exec()`执行赋值字符串。
* 可以使用`re.sub()`将计算结果替换到模板字符串。

提示：如果一项任务令人望而却步，将其分解为较小的部分几乎总是有帮助的。另外，要对手头的工具进行评估，确定如何解决问题。

代码清单10-11提供了一个示例实现。

[代码清单10-11 一个模板系统](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch10/templates.py)

注意：在以前版本的Python中，将字符串放入列表再合并(`join()`)比在`for`循环中使用`+=`高效得多。因为每次赋值都将创建一个新的字符串，这可能会浪费资源，导致程序运行缓慢。而在较新的版本中，使用`+=`可能更快。另见[Python高效字符串拼接]({% post_url 2019-03-03-python-efficient-string-concat %})。如果想更优雅地读取文件中的所有文本，参见第11章。

只用15行代码（不包括空行和注释）就创建了一个强大的模板系统。可见使用标准库时Python有多么强大。下面来测试一下这个模板系统。

对于简单模板[simple_template.txt](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch10/testdata/simple_template.txt)，执行`python templates.py simple_template.txt`，应该得到输出 "The sum of 2 and 3 is 5." 。

由于使用了`fileinput`，因此可以依次处理多个文件。这意味着可以用一个文件定义变量的值，用另一个文件作为模板。例如，[magnus.txt](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch10/testdata/magnus.txt)包含定义，[email_template.txt](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch10/testdata/email_template.txt)是模板文件。运行程序`python templates.py magnus.txt email_template.txt`，将得到输出[email_template_output.txt](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch10/testdata/email_template_output.txt)。

### 10.3.9 其他有趣的标准模块
虽然本章介绍的内容很多，但这只是标准库的冰山一角。下面快速介绍一些很棒的库。
* `argparse`：解析命令行参数和选项
* `cmd`：编写命令解释器
* `csv`：读写CSV(comma-separated values)文件
* `datetime`：处理日期和时间
* `difflib`：比较两个序列的相似程度
* `enum`：枚举类型
* `functools`：高阶函数和可调用对象操作
* `hashlib`：哈希算法（如MD5、SHA等）
* `itertools`：大量用于创建和合并迭代器的工具
* `logging`：日志系统
* `statistics`：数学统计函数
* `timeit`、`profile`和`trace`：代码计时、代码性能分析和覆盖率分析
