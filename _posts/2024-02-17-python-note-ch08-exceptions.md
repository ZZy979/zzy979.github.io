---
title: 《Python基础教程》笔记 第8章 异常
date: 2024-02-17 19:59:31 +0800
categories: [Python, Beginning Python]
tags: [python, exception, error handling, try statement]
---
编写计算机程序时，通常能够区分正常和异常情况。为了处理这些异常事件，可以在每个可能发生这些事件的地方都使用条件语句。然而，这样做不仅效率低下、缺乏灵活性，还会让程序难以阅读。Python提供了强大的替代方案——**异常处理机制**(exception-handling mechanism)。

本章将介绍如何创建和引发异常，以及各种异常处理方式。

## 8.1 异常是什么
Python使用**异常**(exception)对象来表示异常情况，并在遇到错误时**引发**(raise)异常。如果异常对象未被处理（**捕获**），程序将终止并显示一条错误信息(traceback)。

```python
>>> 1 / 0
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ZeroDivisionError: division by zero
```

每个异常都是某个类（这里是`ZeroDivisionError`）的实例，可以被引发并以多种方式捕获，从而进行错误处理，而不是让整个程序失败。

## 8.2 按自己的方式出错
### 8.2.1 raise语句
要引发异常，使用`raise`语句，并将一个类（必须是`Exception`的子类）或实例作为参数。使用类时，将自动创建一个实例。创建异常实例时，可以指定错误信息。例如：

```python
>>> raise Exception
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
Exception
>>> raise Exception('hyperdrive overload')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
Exception: hyperdrive overload
```

有很多内置异常类，下表描述了最重要的几个。完整列表见官方文档[Built-in Exceptions](https://docs.python.org/3/library/exceptions.html)。

| 类名 | 描述 |
| --- | --- |
| `Exception` | （几乎）所有异常的基类 |
| `AttributeError` | 属性引用或赋值失败时引发 |
| `OSError` | 操作系统不能执行指定的任务（如打开文件）时引发，有多个子类 |
| `IndexError` | 使用序列中不存在的索引时引发 |
| `KeyError` | 使用映射中不存在的键时引发 |
| `NameError` | 找不到名称（变量）时引发 |
| `SyntaxError` | 代码存在语法错误时引发 |
| `TypeError` | 将内置操作或函数用于错误类型的对象时引发 |
| `ValueError` | 将内置操作或函数用于类型正确、但值不合适的对象时引发 |
| `ZeroDivisionError` | 除法或取模运算的第二个参数为零时引发 |

### 8.2.2 自定义异常类
有时可能需要创建自己的异常类。在下一节将看到，可以基于异常所属的类选择性地处理异常。

创建异常类就像其他类一样，但要确保（直接或间接地）继承`Exception`。

```python
class SomeCustomException(Exception): pass
```

（如果愿意，也可以在自定义异常类中添加方法）

## 8.3 捕获异常
异常最有趣的地方是可以对其进行处理，通常称为**捕获**(trap/catch)异常。为此，使用`try/except`语句。语法如下：

```python
try:
    block
except E1:
    block
except E2:
    block
...
else:     # optional
    block
finally:  # optional
    block
```

假设创建了一个程序，让用户输入两个数，再将它们相除：

```python
x = int(input('Enter the first number: '))
y = int(input('Enter the second number: '))
print(x / y)
```

当用户输入的第二个数为零时程序将报错：

```
Enter the first number: 10
Enter the second number: 0
Traceback (most recent call last):
  File "exceptions.py", line 3, in <module>
    print(x / y)
ZeroDivisionError: division by zero
```

为了捕获这种异常并进行错误处理（这里只是打印一条错误信息），可以像这样重写这个程序：

[异常示例](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch08/exception_example.py)

使用一条`if`语句来检查`y`的值看起来好像更简单，就本例而言确实是这样。然而，如果程序有多次除法运算，则每个除法都需要一条`if`语句，而使用`try/except`只需要一个错误处理器。

注意：如果没有捕获异常，它就会向外传播到调用函数的地方。如果在这里也没有被捕获，异常将向程序的最顶层传播。这意味着你可以使用`try/except`捕获其他人的函数引发的异常。详见8.4节。

### 8.3.1 不用提供参数
捕获异常后，如果要重新引发它（即继续向上传播），可以使用无参数的`raise`。

例如，下面的计算器类能够“抑制”(muffle) `ZeroDivisionError`异常。

[无参数raise示例](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch08/muffled_calculator.py)

如果无法处理异常，在`except`子句中使用无参数的`raise`通常的不错的选择。但有时可能想引发**别的**异常。在这种情况下，导致进入`except`子句的异常将被作为异常的**上下文**存储起来，并出现在最终的错误信息中。例如：

```python
>>> try:
...     1 / 0
... except ZeroDivisionError:
...     raise ValueError
...
Traceback (most recent call last):
  File "<stdin>", line 2, in <module>
ZeroDivisionError: division by zero

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "<stdin>", line 4, in <module>
ValueError
```

可以使用`raise ... from ...`语句来提供自己的异常上下文，也可以使用`None`来禁用上下文。

```python
>>> try:
...     1 / 0
... except ZeroDivisionError:
...     raise ValueError from None
...
Traceback (most recent call last):
  File "<stdin>", line 4, in <module>
ValueError
```

### 8.3.2 多个except语句
再次运行前一节的程序，如果输入一个非数字值，将引发另一种异常（`int()`引发`ValueError`）。因为`except`子句只捕获了`ZeroDivisionError`。为了同时捕获这种异常，可以在`try`语句中再添加一个`except`子句。

[多个except子句示例](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch08/except_clause_example.py)

这次使用`if`语句将更加困难。检查一个值能否用于除法，最好的方式就是直接相除，看看是否可行。

### 8.3.3 一箭双雕
如果要用一个`except`子句捕获多种异常，可以用一个元组指定这些异常。例如：

```python
try:
    ...
except (ZeroDivisionError, ValueError, NameError):
    print('Your numbers were bogus ...')
```

注意，`except`子句中的圆括号很重要。一种常见的错误是漏掉圆括号，这可能导致不想要的结果（在新版本的Python中这是语法错误）。

### 8.3.4 捕获对象
如果要在`except`子句中访问异常对象本身，可以使用`except ... as ...`。如果希望程序继续运行，但需要打印错误信息，这很有用。例如：

```python
try:
    ...
except (ZeroDivisionError, ValueError) as e:
    print(e)
```

### 8.3.5 一网打尽
如果确实要捕获一段代码中的**所有**异常，只需省略`except`子句中的异常类。

```python
try:
    ...
except:
    print('Something wrong happened ...')
```

像这样捕获所有异常是很危险的，因为这不仅会隐藏你准备处理的错误，还会隐藏你没有考虑过的错误。这还会捕获用户使用Ctrl+C终止执行的企图、调用函数`sys.exit()`终止执行的企图等。在大多数情况下，更好的选择是使用`except Exception as e`并对异常对象`e`做些检查。这将允许少数不是`Exception`子类的异常通过（包括`SystemExit`和`KeyboardInterrupt`）。

### 8.3.6 万事大吉
在有些情况下，当没有出现异常时执行一个代码块很有用。为此，可以给`try`语句添加一个`else`子句。

例如，可以不断地要求用户输入数字，直到能够执行除法运算为止。

[try语句else子句示例](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch08/try_else_clause_example.py)

在这里，仅当没有发生异常时才会跳出循环。

### 8.3.7 最后
最后，还有`finally`子句，用于在发生异常后执行清理工作（例如关闭文件或网络套接字）。不管`try`语句中是否发生异常、发生什么异常，都会执行`finally`子句。

```python
x = None
try:
    x = 1 / 0
finally:
    print('Cleaning up ...')
    del x
```

清理工作会在程序崩溃前执行：

```
Cleaning up ...
Traceback (most recent call last):
  File "<stdin>", line 2, in <module>
ZeroDivisionError: division by zero
```

可以在一条语句中组合使用`try`、`except`、`else`和`finally`。

注：后三个子句都是可选的，但`except`和`finally`至少存在一个。

## 8.4 异常和函数
异常和函数能很自然地一起工作。如果函数中引发的异常没有被处理，它将向上传播到调用函数的地方。如果在那里也没有被处理，异常将继续传播，直至到达主程序（全局作用域）。如果主程序也没有处理异常，程序将终止并显示栈跟踪消息(stack trace)。例如：

```python
def faulty():
    raise Exception('Something is wrong')

def ignore_exception():
    faulty()

def handle_exception():
    try:
        faulty()
    except:
        print('Exception handled')
```

```python
>>> ignore_exception()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 2, in ignore_exception
  File "<stdin>", line 2, in faulty
Exception: Something is wrong
>>> handle_exception()
Exception handled
```

## 8.5 异常之禅
在很多情况下，使用`try/except`语句比`if/else`更自然、更符合Python风格("Pythonic")。应该养成尽可能使用`try/except`语句的习惯。

注：这种习惯通常用一句名言来解释： "It's easier to ask forgiveness than permission." 这种策略可以总结为习语 "Leap Before You Look" ——直接去做，有问题再处理，而不是预先做大量的检查。

## 8.6 不那么异常的情况
如果只想发出**警告**(warning)，指出事情并不完全符合预期，可以使用`warnings`模块的`warn()`函数。

```python
>>> from warnings import warn
>>> warn("I've got a bad feeling about this.")
<stdin>:1: UserWarning: I've got a bad feeling about this.
```

警告只显示一次。如果再次运行最后一行代码，什么都不会发生。

其他代码可以使用`filterwarnings()`函数来抑制你发出的警告（或特定类型的警告），并指定要采取的措施（如 "error" 或 "ignore" ）。

```python
>>> from warnings import filterwarnings
>>> filterwarnings('ignore')
>>> warn('Anyone out there?')
>>> filterwarnings('error')
>>> warn('Something is very wrong!')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
UserWarning: Something is very wrong!
```

发出警告时，可以指定一个异常类（即警告类别），该异常应该是`Warning`的子类。如果将警告转换为错误，将引发指定的异常。另外，还可以根据异常来过滤掉特定类型的警告。

```python
>>> filterwarnings('error')
>>> warn('This function is really old...', DeprecationWarning)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
DeprecationWarning: This function is really old...
>>> filterwarnings('ignore', category=DeprecationWarning)
>>> warn('Another deprecation warning.', DeprecationWarning)
```
