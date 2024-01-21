---
title: 《Python基础教程》笔记 第6章 抽象
date: 2024-01-04 21:31:19 +0800
categories: [Python, Beginning Python]
tags: [python, function, definition, keyword argument, default argument, scope, recursion, binary search, functional programming, lambda expression]
math: true
---
本章将介绍如何将语句组合成函数，详细介绍参数和作用域，还将讨论递归是什么及其在程序中的用途。

## 6.1 懒惰即美德
如果你在一个地方编写了一些代码，在另一个地方也需要使用。当然可以再写一遍，但是如果已编写好的代码更复杂呢？真正的程序员不会这样做。真正的程序员很懒——“懒”不是贬义词，而是指不做无谓的工作。懒惰即美德(laziness is a virtue)。真正的程序员会让程序更**抽象**(abstract)。

## 6.2 抽象和结构
抽象可以节省人力。更重要的是，抽象是程序能够被人理解的关键所在。程序应该非常抽象，例如“下载网页，计算词频，打印每个单词的频率”。将这段描述翻译成Python程序：

```python
page = download_page()
freqs = compute_frequencies(page)
for word, freq in freqs:
    print(word, freq)
```

看到这段代码，任何人都知道这个程序是做什么的。然而，没有明确地说如何做，这些操作的具体细节将在其他地方给出——单独的**函数定义**。

## 6.3 创建自己的函数
**函数**(function)是可以调用的（可能带有参数），它执行特定的操作并返回一个值。可以使用内置函数`callable()`判断某个对象是否可调用。

```python
>>> import math
>>> callable(1)
False
>>> callable(math.sqrt)
True
```

函数是结构化编程的核心。使用`def`("function definition")语句定义函数：

```python
def function_name(parameter_list):
    function_body
```

例如：

```python
def hello(name):
    return 'Hello, ' + name + '!'
```

`return`语句用于从函数中返回值。

注：
* 在Python函数定义中，不需要声明参数类型和返回类型。但可以通过函数注解进行说明（不会强制检查），详见文档[Function Annotations](https://docs.python.org/3/tutorial/controlflow.html#function-annotations)。
* 对于函数`hello()`，任何支持与字符串`+`操作的对象都可以作为`name`参数（否则会报错）（在这一点上类似于C++的函数模板），条件语句的不同分支也可以返回不同类型的值。

运行这段代码后，将得到一个名为`hello`的函数，它返回一个字符串，包含向参数指定的人发出的问候语。可以像内置函数一样使用使用这个函数。

```python
>>> print(hello('world'))
Hello, world!
>>> print(hello('Gumby'))
Hello, Gumby!
```

编写一个函数，返回斐波那契数列（的前n项）：

```python
def fibs(n):
    result = [0, 1]
    for i in range(n - 2):
        result.append(result[-2] + result[-1])
    return result
```

```python
>>> fibs(10)
[0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
>>> fibs(15)
[0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377]
```

### 6.3.1 为函数编写文档
要给函数编写文档，以确保其他人能够理解，可以添加注释（以`#`开头）。另一种方式是**文档字符串**(docstring)。这种字符串可以放在`def`语句后面（以及模块和类的开头）。例如：

```python
def square(x):
  'Calculates the square of the number x.'
  return x * x
```

文档字符串可以通过函数的`__doc__`属性或者内置函数`help()`访问。

```python
>>> square.__doc__
'Calculates the square of the number x.'
>>> help(square)
Help on function square in module __main__:

square(x)
    Calculates the square of the number x.
```

详见官方文档[Documentation Strings](https://docs.python.org/3/tutorial/controlflow.html#documentation-strings)。

注：[PEP 287](https://peps.python.org/pep-0287/)推荐的文档字符串格式是[reStructuredText](https://docutils.sourceforge.io/rst.html)，使用标签`:param`和`:return`对参数和返回值进行说明。参考：
* [What are the most common Python docstring formats?](https://stackoverflow.com/questions/3898572/what-are-the-most-common-python-docstring-formats)
* [Document source code - PyCharm](https://www.jetbrains.com/help/pycharm/documenting-source-code.html)

### 6.3.2 并非真正函数的函数
数学意义上的函数总是返回根据参数计算得到的结果。在Python中，有些函数什么都不返回——没有`return`语句，或者`return`后面没有值。

```python
def test():
    print('This is printed')
    return
    print('This is not')
```

这里使用`return`语句只是为了结束函数。

```python
>>> x = test()
This is printed
>>> x
>>> print(x)
None
```

实际上，所有的函数都有返回值，如果没有指定返回什么，将返回`None`。

警告：不要让这种默认行为带来麻烦。如果在`if`之类的语句中返回值，务必确保其他分支也返回值。

## 6.4 参数魔法
### 6.4.1 值从哪里来
参数的值来自调用函数时提供的实参。

注意：在`def`语句中，位于函数名后面的变量通常称为**形式参数/形参**(formal parameter)，而调用函数时提供的值叫做**实际参数/实参**(actual parameter / argument)。在必要时，会将实参称为**值**。

### 6.4.2 我能修改参数吗
函数通过参数获得了一系列的值。参数不过是变量而已，行为与你预期的完全相同。**在函数内部给参数赋值对外部没有任何影响。**

```python
>>> def try_to_change(n):
...     n = 'Mr. Gumby'
...
>>> name = 'Mrs. Entity'
>>> try_to_change(name)
>>> name
'Mrs. Entity'
```

在`try_to_change()`内部，将新值赋给参数`n`，这对变量`name`没有影响。如下图所示。

![给参数赋值](/assets/images/python-note-ch06-abstraction/给参数赋值.png)

其效果类似于下面这样：

```python
>>> name = 'Mrs. Entity'
>>> n = name            # This is almost what happens when passing a parameter
>>> n = 'Mr. Gumby'     # This is done inside the function
>>> name
'Mrs. Entity'
```

字符串（以及数字和元组）是**不可变**(immutable)的，这意味着你不能修改它们，只能替换为新值。但如果参数是**可变的**(mutable)数据结构（如列表）呢？

```python
>>> def change(n):
...     n[0] = 'Mr. Gumby'
...
>>> names = ['Mrs. Entity', 'Mrs. Thing']
>>> change(names)
>>> names
['Mr. Gumby', 'Mrs. Thing']
```

在这个示例中，参数被修改了。这是与前一个示例的重要区别。

![修改参数](/assets/images/python-note-ch06-abstraction/修改参数.png)

不用函数调用再做一次：

```python
>>> names = ['Mrs. Entity', 'Mrs. Thing']
>>> n = names           # Again pretending to pass names as a parameter
>>> n[0] = 'Mr. Gumby'  # Change the list
>>> names
['Mr. Gumby', 'Mrs. Thing']
```

你已经见过这种情况（2.3.3节`copy()`方法），两个变量引用同一个列表。要避免这种情况，必须创建列表的**副本**(copy)（使用切片或`copy()`方法）。

```python
>>> names = ['Mrs. Entity', 'Mrs. Thing']
>>> change(names[:])
>>> names
['Mrs. Entity', 'Mrs. Thing']
```

#### 为什么要修改参数
使用函数改变数据结构（例如列表或字典）是一种将程序抽象化的好方法。假设要编写一个程序，能存储姓名，并允许用户根据姓、名或中间名查找联系人。

[简单联系人应用](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch06/simple_contacts.py)

注意：这里使用的是过程式编程（全局变量作为数据结构、函数作为操作）。这类应用很适合面向对象编程（类作为数据结构、方法作为操作），这将在下一章介绍。

#### 如果参数是不可变的
在有些语言（例如C++）中，给形参赋值可以影响实参（例如引用参数）。在Python中这是不可能的，只能修改参数对象本身，如果参数不可变则没有办法。这种情况下，应该从函数返回需要的值。例如，可以这样编写将变量的值加1的函数：

```python
def inc(x):
    return x + 1
```

### 6.4.3 关键字参数和默认值
目前为止使用的参数都是**位置参数**(positional parameter)，因为它们的位置很重要。考虑下面的函数：

```python
def hello(greeting, name):
    print('{}, {}!'.format(greeting, name))
```

```python
>>> hello('Hello', 'world')
Hello, world!
>>> hello('world', 'Hello')
world, Hello!
```

当参数很多时，顺序可能难以记住。为此，在调用函数时可以指定参数名——这叫做**关键字参数**(keyword argument)，形式为`kwarg=value`。

```python
>>> hello(greeting='Hello', name='world')
Hello, world!
>>> hello(name='world', greeting='Hello')
Hello, world!
```

此时，参数的顺序无关紧要，但名称很重要。

在定义函数时，可以指定参数的默认值，形式类似于关键字参数：`param=default_value`。

```python
def hello(greeting='Hello', name='world'):
    print('{}, {}!'.format(greeting, name))
```

对于有默认值的参数，可以根据需要，不提供、提供部分或提供全部参数值。

```python
>>> hello()
Hello, world!
>>> hello('Greetings')
Greetings, world!
>>> hello('Greetings', 'universe')
Greetings, universe!
>>> hello(name='Gumby')
Hello, Gumby!
```

调用函数`hello()`时，如果使用位置参数，则提供`name`时必须提供`greetings`，即只能省略最右侧的参数。如果只想提供`name`，可以使用关键字参数。

位置参数和关键字参数可以结合使用，只要将位置参数放在前面即可。

例如，函数`hello()`要求必须指定姓名，而问候语和标点是可选的。

```python
def hello(name, greeting='Hello', punctuation='!'):
    print('{}, {}{}'.format(greeting, name, punctuation))
```

调用这个函数的方式很多，例如：

```python
>>> hello('Mars')
Hello, Mars!
>>> hello('Mars', 'Howdy')
Howdy, Mars!
>>> hello('Mars', 'Howdy', '...')
Howdy, Mars...
>>> hello('Mars', punctuation='.')
Hello, Mars.
>>> hello('Mars', greeting='Top of the morning to ya')
Top of the morning to ya, Mars!
>>> hello()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: hello() missing 1 required positional argument: 'name'
```

注：
* Python不支持函数重载，只能使用默认参数实现这种功能。
* 默认情况下，所有参数都既可以按位置提供，也可以按关键字提供。但是，在定义函数时，可以在参数列表中使用两个特殊的标记：`/`和`*`。`/`之前的参数只能按位置提供(positional-only)，`*`之后的参数只能按关键字提供(keyword-only)，二者之间的参数两种形式都可以。详见官方文档[Special parameters](https://docs.python.org/3/tutorial/controlflow.html#special-parameters)。
* 默认值只求值一次。当默认值是可变对象（例如列表或字典）时，可能导致意外的结果。例如：

```python
def f(a, l=[]):
    l.append(a)
    return l
```

```python
>>> f(1)
[1]
>>> f(2)
[1, 2]
>>> f(3)
[1, 2, 3]
```

### 6.4.4 收集参数
有时，允许用户提供任意数量的参数是有用的。

为此，需要添加一个`*args`形式的参数，多余的位置参数将被收集到一个名为`args`的元组中。

```python
def print_params(title, *params):
    print(title)
    print(params)
```

```python
>>> print_params('Params:', 1, 2, 3)
Params:
(1, 2, 3)
>>> print_params('Nothing:')
Nothing:
()
```

带星号的参数也可以放在其他位置（而不是最后），但其后面的参数必须使用名称指定。

```python
def in_the_middle(x, *y, z):
    print(x, y, z)
```

```python
>>> in_the_middle(1, 2, 3, 4, 5, z=7)
1 (2, 3, 4, 5) 7
>>> in_the_middle(1, 2, 3, 4, 5, 7)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: in_the_middle() missing 1 required keyword-only argument: 'z'
```

`*`不收集关键字参数。要收集关键字参数，可以使用两个星号：`**kwargs`，多余的关键字参数将被收集到一个名为`kwargs`的字典中。

```python
def print_params(x, y, z=3, *args, **kwargs):
    print(x, y, z)
    print(args)
    print(kwargs)
```

```python
>>> print_params(1, 2, 3, 5, 6, 7, foo=1, bar=2)
1 2 3
(5, 6, 7)
{'foo': 1, 'bar': 2}
>>> print_params(1, 2)
1 2 3
()
{}
```

通过结合这些技术，可以做很多事情。如果你想知道某种结合方式如何工作（或者是否允许），动手试一试即可！

### 6.4.5 解包参数
前面介绍了如何将参数收集到元组和字典中。用同样的两个运算符（`*`和`**`）也可以执行相反的操作——解包参数。假设有如下函数：

```python
def add(x, y):
    return x + y
```

注意：`operator`模块提供了这个函数的高效版本。

```python
>>> params = (1, 2)
>>> add(*params)
3
```

这与前面的操作是相反的——不是收集参数，而是**分配**(distribute)/**解包**(unpacking)参数。这是通过在调用函数（而不是定义函数）时使用`*`运算符实现的。

使用`**`运算符可以将字典分配给关键字参数。

```python
>>> params = {'name': 'Sir Robin', 'greeting': 'Well met'}
>>> hello(**params)
Well met, Sir Robin!
```

注：
* 调用函数时使用`*`意味着将元组（或列表）的各元素依次传给各参数，元素个数必须与参数个数相同，否则报错。
* 使用`*`和`**`解包也可用于列表和字典字面值，这叫做**可迭代解包**(iterable unpacking)和**字典解包**(dictionary unpacking)，详见[PEP 448](https://peps.python.org/pep-0448/)。例如：

```python
>>> a = [1, 2, 3]
>>> [*a, 4, 5]
[1, 2, 3, 4, 5]
>>> *range(4), 4
(0, 1, 2, 3, 4)
>>> d = {'a': 1, 'b': 2}
>>> {**d, 'c': 3}
{'a': 1, 'b': 2, 'c': 3}
```

在定义和调用函数时都使用`*`或`**`仅传递元组或字典，因此没有什么意义。

```python
def with_stars(**kwargs):
    print(kwargs['name'], 'is', kwargs['age'], 'years old')

def without_stars(kwargs):
    print(kwargs['name'], 'is', kwargs['age'], 'years old')
```

```python
>>> d = {'name': 'Mr. Gumby', 'age': 42}
>>> with_stars(**d)
Mr. Gumby is 42 years old
>>> without_stars(d)
Mr. Gumby is 42 years old
```

因此，只有在定义**或**调用函数时使用星号才有用。

提示：使用`*`和`**`运算符来“转发”参数很有用，因为无需担心参数个数之类的问题。例如：

```python
def foo(x, y, z, m=0, n=0):
    print(x, y, z, m, n)

def call_foo(*args, **kwds):
    print('Calling foo!')
    foo(*args, **kwds)
```

这在调用超类的构造函数时特别有用（详见第9章）。

### 6.4.6 参数练习
面对这么多提供和接受参数的方式，很容易犯晕。
* 定义函数：普通参数、默认参数、收集参数
* 调用函数：位置参数、关键字参数、解包参数

下面来看一个综合示例。首先定义一些函数：

[参数练习](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch06/parameter_practice.py)

下面来尝试一下：

```python
>>> print(story(job='king', name='Gumby'))
Once upon a time, there was a king called Gumby.
>>> print(story(name='Sir Robin', job='brave knight'))
Once upon a time, there was a brave knight called Sir Robin.
>>> params = {'job': 'language', 'name': 'Python'}
>>> print(story(**params))
Once upon a time, there was a language called Python.
>>> del params['job']
>>> print(story(job='stroke of genius', **params))
Once upon a time, there was a stroke of genius called Python.
```

```python
>>> power(2, 3)
8
>>> power(3, 2)
9
>>> power(y=3, x=2)
8
>>> params = (5,) * 2
>>> power(*params)
3125
>>> power(3, 3, 'Hello, world')
Received redundant parameters: ('Hello, world',)
27
```

```python
>>> interval(10)
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>>> interval(1, 5)
[1, 2, 3, 4]
>>> interval(3, 12, 4)
[3, 7, 11]
>>> power(*interval(3, 7))
Received redundant parameters: (5, 6)
81
```

## 6.5 作用域
赋值语句`x = 1`将名字`x`指向值`1`，这类似于字典（键指向值）。内置函数`var()`返回这个“看不见的”字典（变量名到值的映射）。

```python
>>> x = 1
>>> scope = vars()
>>> scope['x']
1
>>> scope['x'] += 1
>>> x
2
```

警告：一般而言，不应该修改`vars()`返回的字典，其结果是未定义的。

这种“看不见的字典”称为**命名空间**(namespace)或**作用域**(scope)。除了全局作用域外，每个函数调用都会创建一个新的作用域。

```python
>>> def foo(): x = 42
...
>>> x = 1
>>> foo()
>>> x
1
```

注：在Python中，只有模块、类和函数才会引入新的作用域，其他代码块（例如`if`、`for`等）不会。

在函数内使用的变量称为**局部变量**（相对于**全局变量**）。参数类似于局部变量，因此参数与全局变量同名不会有问题。

在函数中可以访问全局变量。

```python
>>> def combine(parameter): print(parameter + external)
...
>>> external = 'berry'
>>> combine('Shrub')
Shrubberry
```

警告：全局变量是众多bug的根源。慎用全局变量。

### 遮盖问题
如果有一个局部变量或参数与全局变量同名，就无法直接访问全局变量，因为它被局部变量**遮盖**(shadow)了。

如果需要，可以使用`globals()`函数来访问全局变量。该函数返回包含全局变量的字典（`locals()`返回包含局部变量的字典）。

```python
>>> def combine(parameter):
...     print(parameter + globals()['parameter'])
...
>>> parameter = 'berry'
>>> combine('Shrub')
Shrubberry
```

给全局变量赋值是另一回事。在函数内部给变量赋值时，该变量默认为局部变量，除非使用`global`语句声明。

```python
>>> x = 1
>>> def change_global():
...     global x
...     x = x + 1
...
>>> change_global()
>>> x
2
```

### 嵌套作用域
Python函数可以嵌套，即可以将一个函数放在另一个函数内。例如：

```python
def foo():
    def bar():
        print("Hello, world!")
    bar()
```

嵌套通常用处不大，但有一个很突出的用途：使用一个函数来“创建”另一个。例如：

```python
def multiplier(factor):
    def multiplyByFactor(number):
        return number * factor
    return multiplyByFactor
```

**外层函数返回内层函数。** 内层函数被返回后仍然能够访问其定义所在的作用域。换句话说，它携带了自己的环境（和相关的局部变量）！

每次调用外层函数`multiplier()`时，都将重新定义内层函数`multiplyByFactor()`，而变量`factor`每次都有一个新值。

```python
>>> double = multiplier(2)
>>> double(5)
10
>>> triple = multiplier(3)
>>> triple(3)
9
>>> multiplier(5)(4)
20
```

像`multiplyByFactor()`这样存储其所在作用域的函数称为**闭包**(closure)。

通常，不能给外层作用域内的变量赋值，除非使用`nonlocal`语句声明（例如，`multiplyByFactor()`要给`factor`赋值，则用`nonlocal factor`声明）。

## 6.6 递归
函数可以**调用自身**，这称为**递归**(recursion)。

```
recursion /rɪˈkɜː.ʒən/ n: see recursion.
```

一个类似的函数定义如下：

```python
def recursion():
    return recursion()
```

该函数显然什么都做不了，这种递归称为**无穷递归**（就像以`while True`开头且不包含`break`或`return`语句的循环称为**无限循环**）。

有用的递归函数包含下面两部分：
* **基本情况**(base case)：函数直接返回一个值（递归出口）
* **递归情况**(recursive case)：包含一个或多个对于问题较小部分的递归调用

这里的关键是将问题分解为较小的部分。

### 6.6.1 两个经典案例：阶乘和幂
首先，计算数字n的**阶乘**(factorial)：n! = 1×2×...×n

可以使用循环：

```python
def factorial(n):
    result = n
    for i in range(1, n):
        result *= i
    return result
```

也可以使用递归，关键在于阶乘的数学定义：
* 1的阶乘为1
* 对于n>1，n的阶乘为n乘以n-1的阶乘

理解这个定义后，实现起来非常简单：

[阶乘](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch06/factorial.py)

注：这个实现对于n≤0会导致无穷递归，这种情况应该引发异常。`math`模块提供了更完善的`factorial()`函数。

考虑另外一个例子：计算x的n次幂（就像内置函数`pow()`和运算符`**`）。使用循环很容易实现：

```python
def power(x, n):
    result = 1
    for i in range(n):
        result *= x
    return result
```

也可以使用递归定义：
* x的0次幂为1
* 对于n>0，x的n次幂为x乘以x的n-1次幂

[幂](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch06/power.py)

注：这个实现只能处理n为整数且非负的情况。

那么使用递归有什么意义？在大多数情况下可以使用循环代替递归，而且循环的效率可能更高。然而，在很多情况下，递归的可读性更高，当你理解了函数的递归定义时尤其如此。

### 6.6.2 另一个经典案例：二分查找
最后一个递归示例是**二分查找**(binary search)算法。

一个常见的问题是：查找一个数字是否在（已排序的）序列中，或者在什么位置。可以采用二分查找策略：“这个数字是否在序列中间的右边？”如果是，“是否在右半部分的右边？”如果不是，“是否在左半部分的右边？”以此类推。记下数字**可能**在的位置上下限，每个问题都将区间分成两半。

这个算法自然地引出了递归定义：
* 如果上限和下限相同，说明都指向数字所在的位置，因此将其返回。
* 否则，找出区间的中间位置（上下限的平均值），确定数字在左半部分还是右半部分。然后继续在数字所在的那部分中查找。

这个递归情况的关键是序列是已排序的。（注意，这个算法返回数字**应该**在的位置，如果数字本身不在序列中，则这个位置上是其他数字）

下面实现二分查找算法：

[二分查找](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch06/binary_search.py)

然而，为什么要如此麻烦，而不直接使用列表方法`index()`或者使用循环实现（线性查找）呢？因为序列很大时，线性查找效率低下。对于长度为n的序列，线性查找平均需要比较 $\frac{n}{2}$ 次，而二分查找只需要比较 $\lceil \log_2 n \rceil$ 次。

| n | 线性查找 | 二分查找 |
| --- | --- | --- |
| 100 | 50 | 7 |
| 100000000 | 50000000 | 27 |
| 10<sup>35</sup> | 5×10<sup>34</sup> | 117 |

提示：模块`bisect`提供了标准的二分查找实现。

### 函数式编程
函数可以像其他对象一样使用：将其赋给变量，作为参数传递以及从其他函数中返回。尽管在Python中通常不会如此倚重函数，但完全可以这样做。这称为**函数式编程**(functional programming)。

Python提供了一些对于函数式编程有用的函数：`map()`、`filter()`和`reduce()`。

`map()`将给定函数应用于序列的所有元素。

```python
>>> list(map(str, range(10))) # Equivalent to [str(i) for i in range(10)]
['0', '1', '2', '3', '4', '5', '6', '7', '8', '9']
```

`filter()`根据布尔函数对元素进行过滤。

```python
>>> def func(x): return x.isalnum()
...
>>> seq = ["foo", "x41", "?!", "***"]
>>> list(filter(func, seq))  # Equivalent to [x for x in seq if x.isalnum()]
['foo', 'x41']
```

实际上，Python有一种名为**Lambda表达式**(lambda expression)的特性，使你能创建简单的匿名函数。

```python
>>> list(filter(lambda x: x.isalnum(), seq))
['foo', 'x41']
```

然而列表推导式的可读性更高。

`reduce()`使用给定函数组合序列的前两个元素，再组合结果与第3个元素，以此类推，直到处理完整个序列并得到一个结果。例如，计算序列中所有数字的和：

```python
>>> numbers = [72, 101, 108, 108, 111, 44, 32, 119, 111, 114, 108, 100, 33]
>>> from functools import reduce
>>> reduce(lambda x, y: x + y, numbers)
1161
```

可以使用`operator`模块的`add()`函数代替该Lambda表达式，也可以直接使用内置函数`sum()`。

```python
>>> import operator
>>> reduce(operator.add, numbers)
1161
>>> sum(numbers)
1161
```
