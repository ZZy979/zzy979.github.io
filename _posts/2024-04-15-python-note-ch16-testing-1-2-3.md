---
title: 《Python基础教程》笔记 第16章 测试
date: 2024-04-15 22:33:47 +0800
categories: [Python, Beginning Python]
tags: [python, unit test]
---
本章介绍测试的基本知识，告诉你如何养成在编程中进行测试的习惯，并介绍一些编写测试的有用工具。

## 16.1 先测试，后编码
为了计划改变和灵活性，为程序的各个部分编写测试（所谓的**单元测试**(unit test)）非常重要。极限编程(Extreme Programming)的那群人引入了非常有用、但有些违反直觉的格言“测试一点，编码一点”，而不是直观的“编码一点，测试一点”。换句话说，先测试，后编码。这也称为**测试驱动编程**(test-driven programming)。

### 16.1.1 准确的需求说明
开发软件时，必须先知道软件要解决什么问题、要实现什么样的目标。可以通过编写**需求说明**(requirement specification)（描述需求的文档）来阐明程序的目标，这样以后就很容易核实需求是否确实得到了满足。

这里的理念是先编写测试程序，再编写能通过测试的程序。测试程序就是需求说明，可以帮助你在开发过程中紧扣这些需求。

来看一个简单的示例。假设你要编写一个模块，其中包含一个根据矩形的宽和高计算面积的函数。开始编写代码前，先编写一个单元测试，其中包含一些你知道答案的例子。这个测试程序可能类似于代码清单16-1。

[代码清单16-1 简单的测试程序](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch16/area_test.py)

在这个示例中，使用高=3和宽=4调用函数`rect_area()`（尚未编写），再将结果与正确答案12比较（当然，只测试这一种情况并不能让你确信代码是正确的，真正的测试程序可能要详尽得多）。

如果（在文件area.py中）不小心将`rect_area()`实现为下面这样，并尝试运行测试程序，就会得到错误消息。

```python
def rect_area(height, width):
    return height * height # This is wrong ...
```

接下来可以检查代码，看看哪里出错了，并将返回的表达式替换为`height * width`。

先测试后编码并不仅仅是为了发现bug——它是为了检查代码到底能否工作。“除非有相应的测试，否则该特性就并不存在（或者说不是真正的特性）。”

### 16.1.2 计划改变
自动化测试不仅可以在编写程序时提供极大的帮助，还有助于在修改代码时避免积累错误，随着程序规模的增长，这一点尤其重要。正如第19章讨论的，你必须做好修改代码的准备，而不是固守既有代码。

#### 代码覆盖率
**覆盖率**(coverage)是测试中一个重要的概念。优秀测试套件的目标之一是达到较高的覆盖率。实现这一目标的一种方式是使用覆盖率工具，它测量测试期间实际运行的代码所占的比例。可以使用Python标准库模块`trace`。

### 16.1.3 测试四步曲
在深入介绍编写测试的细节之前，先来看看测试驱动开发过程的各个阶段：
1. 确定需要实现的新特性。可以将其记录下来，然后为其编写一个测试。
2. 编写实现功能的框架代码(skeleton code)，让程序能够运行（不存在语法错误之类的问题），但测试仍然失败。看到测试失败是很重要的，这样才能确定它**可能**失败。如果测试有问题，在任何情况下都能成功，那么它实际上什么都没有测试。在试图让测试**成功**前，先要看到它**失败**。
3. 为框架编写虚设代码(dummy code)，无需准确地实现功能，只要让测试通过即可。这样，在整个开发阶段，都能够让所有的测试通过（除了首次运行测试）。
4. 重写（或**重构**(refactor)）代码以实现所需的功能，同时确保测试依然成功。

提交代码时，必须确保代码处于健康状态，即没有任何测试是失败的。在任何情况下，都不应该将存在失败测试的代码提交到公共代码库。

## 16.2 测试工具
标准库有两个很棒的模块可以替你自动完成测试过程。
* `unittest`：通用的单元测试框架
* `doctest`：测试文档字符串中的交互式示例

### 16.2.1 doctest
本书自始至终都使用直接取自交互式解释器的示例。这是演示工作原理的一种有效方法，而且有这样的例子时，自己也很容易测试。实际上，交互式会话是一种很有用的文档，可以将其放在文档字符串中。

例如，假设编写了一个计算平方数的函数，并在其文档字符串中添加了一个示例。

```python
def square(x):
    """
    Squares a number and returns the result.
    >>> square(2)
    4
    >>> square(3)
    9
    """
    return x * x
```

假设该函数定义在my_math.py中，在末尾添加如下代码：

```python
if __name__ == '__main__':
    import doctest
    doctest.testmod()
```

这有什么用呢？让我们试一试。

```shell
$ python my_math.py
$
```

看起来什么都没发生，但这是件好事。函数`doctest.testmod()`读取指定模块（默认为`__main__`）中的所有文档字符串，查找看起来像是来自交互式解释器的示例（以`'>>> '`开头），之后检查这些示例是否反映了实际情况。

为了获取更多输出，可以在运行脚本时指定开关`-v`("verbose")。

```shell
$ python my_math.py -v
Trying:
    square(2)
Expecting:
    4
ok
Trying:
    square(3)
Expecting:
    9
ok
1 items had no tests:
    __main__
1 items passed all tests:
   2 tests in __main__.square
2 tests in 2 items.
2 passed and 0 failed.
Test passed.
```

可以看到，幕后发生了很多事情。函数`testmod()`检查模块文档字符串（不包含测试）和函数文档字符串（包含两个测试，都成功了）。

有了测试，就可以放心地修改代码了。假设要使用幂运算符，即将`x * x`改为`x ** 2`，但不小心写成了`x ** x`。尝试一下，然后运行脚本对代码进行测试，输出如下：

```
**********************************************************************
File "my_math.py", line 6, in __main__.square
Failed example:
    square(3)
Expected:
    9
Got:
    27
**********************************************************************
1 items had failures:
   1 of   2 in __main__.square
***Test Failed*** 1 failures.
```

捕捉到了bug，并清楚地指出错误在什么地方。现在修复这个问题应该不难了。

有关`doctest`模块的详细信息，见官方文档[doctest](https://docs.python.org/3/library/doctest.html)。

### 16.2.2 unittest
尽管`doctest`使用起来很容易，`unittest`（基于流行的Java测试框架JUnit）更灵活、更强大。这里只进行简要介绍。

提示：标准库之外还有两个单元测试工具的替代品：pytest (<https://docs.pytest.org/>)和nose (<https://nose.readthedocs.io/>)。

下面来看一个简单的示例。假设你要编写一个名为`my_math`的模块，其中包含一个计算乘积的函数`product()`。首先，当然是编写一个测试（在文件test_my_math.py中），使用`unittest`模块中的`TestCase`类，如代码清单16-2所示。

[代码清单16-2 使用unittest框架的简单测试](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch16/test_my_math.py)

函数`unittest.main()`负责运行测试：实例化所有`TestCase`的子类，并运行所有名称以`test`开头的方法。

提示：如果定义了`setUp()`和`tearDown()`方法，它们将分别在每个测试方法之前和之后执行。可以使用这些方法为所有测试提供公共初始化和清理代码，称为**测试夹具**(test fixture)。

诸如`assertEqual()`等方法检查指定的条件，以判断测试成功还是失败。`TestCase`类包含很多类似的方法，如下表所示。

| 方法 | 检查条件 |
| --- | --- |
| `assertEqual(a, b)` | `a == b` |
| `assertNotEqual(a, b)` | `a != b` |
| `assertTrue(x)` | `bool(x) is True` |
| `assertFalse(x)` | `bool(x) is False` |
| `assertIs(a, b)` | `a is b` |
| `assertIsNot(a, b)` | `a is not b` |
| `assertIsNone(x)` | `x is None` |
| `assertIsNotNone(x)` | `x is not None` |
| `assertIn(a, b)` | `a in b` |
| `assertNotIn(a, b)` | `a not in b` |
| `assertIsInstance(a, b)` | `isinstance(a, b)` |
| `assertNotIsInstance(a, b)` | `not isinstance(a, b)` |
| `assertAlmostEqual(a, b)` | `round(a-b, 7) == 0` |
| `assertNotAlmostEqual(a, b)` | `round(a-b, 7) != 0` |
| `assertGreater(a, b)` | `a > b` |
| `assertGreaterEqual(a, b)` | `a >= b` |
| `assertLess(a, b)` | `a < b` |
| `assertLessEqual(a, b)` | `a <= b` |
| `assertRegex(s, r)` | `r.search(s)` |
| `assertNotRegex(s, r)` | `not r.search(s)` |
| `assertCountEqual(a, b)` | `a`和`b`具有同样数量的相同元素，无论其顺序如何 |
| `assertRaises(exc, fun, *args, **kwds)` | `fun(*args, **kwds)`引发`exc` |
| `assertRaisesRegex(exc, r, fun, *args, **kwds)` | `fun(*args, **kwds)`引发`exc`，且错误消息匹配正则表达式`r` |

注：所有断言方法都接受可选的`msg`参数，如果指定了该参数，它将被用作测试失败时的错误消息。

`unittest`模块区分**错误**(error)和**失败**(failure)。失败是指断言方法检查的条件不满足（引发`AssertionError`），错误是指引发了其他异常。

下一步是编写框架代码，创建文件my_math.py，包含如下内容：

```python
def product(x, y):
    pass
```

如果现在运行测试，将出现两条`FAIL`消息，如下所示：

```shell
$ python test_my_math.py
FF
======================================================================
FAIL: test_floats (__main__.ProductTestCase.test_floats)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "test_my_math.py", line 20, in test_floats
    self.assertEqual(p, x * y, 'Float multiplication failed')
AssertionError: None != 1.0 : Float multiplication failed

======================================================================
FAIL: test_integers (__main__.ProductTestCase.test_integers)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "test_my_math.py", line 12, in test_integers
    self.assertEqual(p, x * y, 'Integer multiplication failed')
AssertionError: None != 100 : Integer multiplication failed

----------------------------------------------------------------------
Ran 2 tests in 0.001s

FAILED (failures=2)
```

下一步就是让代码能工作：

```python
def product(x, y):
    return x * y
```

现在输出如下：

```shell
$ python test_my_math.py
..
----------------------------------------------------------------------
Ran 2 tests in 0.001s

OK
```

开头的`.`表示测试成功，`F`表示失败。

提示：有关更高级的面向对象代码的测试，请参阅`unittest.mock`模块。另见[【Python】模拟对象模块unittest.mock]({% post_url 2021-03-12-python-unittest-mock %})。

## 16.3 单元测试以外
测试显然很重要，但还有其他方式来探测(probulate)程序。这里介绍两个工具：源代码检查和性能分析。源代码检查是一种发现代码中常见错误或问题的方式（有点像静态类型语言中编译器的作用，但远不止如此）。性能分析是指搞清楚程序的运行速度到底有多快。

### 16.3.1 使用PyChecker和PyLint检查源代码
很长一段时间，PyChecker(<https://pychecker.sourceforge.net/>)都是检查Python代码的唯一工具，能够找出给函数提供的参数不对等错误。之后又出现了Pylint(<https://pylint.org/>)，它支持PyChecker的大部分特性，还有很多其他的功能（例如变量名是否符合指定的命名约定、是否遵守了自己的编码标准等）。

### 16.3.2 性能分析
正如Donald Knuth所说：“在编程中，过早的优化是万恶之源。”如果程序的速度已经足够快，那么代码清晰、简单、易懂的价值远高于细微的速度提升。毕竟几个月后就可能有速度更快的硬件面世。

但如果程序的速度达不到你的要求而必须优化，首先绝对应该对其进行性能分析。因为除非程序非常简单，否则很难猜到瓶颈在什么地方。

标准库包含一个性能分析模块`profile`，还有一个更快的C语言版本`cProfile`。使用性能分析器非常简单，只需使用命令字符串参数调用`run()`函数。

```python
>>> import cProfile
>>> from my_math import product
>>> cProfile.run('product(1, 2)')
         4 function calls in 0.000 seconds

   Ordered by: standard name

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.000    0.000    0.000    0.000 <string>:1(<module>)
        1    0.000    0.000    0.000    0.000 my_math.py:12(product)
        1    0.000    0.000    0.000    0.000 {built-in method builtins.exec}
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
```

这将输出如下信息：各个函数和方法被调用了多少次，以及执行它们花费了多长时间。如果`run()`的第二个参数提供一个文件名（例如`'my_math.profile'`），分析结果将保存到这个文件中。之后可以使用`pstats`来检查分析结果。

```python
>>> cProfile.run('product(1, 2)', 'my_math.profile')
>>> import pstats
>>> p = pstats.Stats('my_math.profile')
>>> p.print_stats()
Thu Apr 18 23:44:24 2024    my_math.profile

         4 function calls in 0.000 seconds

   Random listing order was used

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.000    0.000    0.000    0.000 <string>:1(<module>)
        1    0.000    0.000    0.000    0.000 my_math.py:12(product)
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
        1    0.000    0.000    0.000    0.000 {built-in method builtins.exec}
```

提示：标准库还包含一个名为`timeit`的模块，提供了对一小段Python代码进行计时的简单方式。
