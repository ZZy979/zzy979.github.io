---
title: 《Python基础教程》笔记 第5章 条件、循环和其他语句
date: 2023-12-27 21:01:10 +0800
categories: [Python, Beginning Python]
tags: [python, import statement, assignment, block, if statement, operator, assertion, while statement, for statement, break statement, continue statement, list comprehension, dictionary comprehension, pass statement, del statement]
math: true
---
你已经见过几种语句（`print`语句、`import`语句和赋值），先来看看这些语句的一些其他用法，再深入探讨**条件语句**和**循环语句**。然后将介绍**列表推导式**，它虽然是表达式，但工作原理几乎与条件语句和循环语句相同。最后将介绍`pass`、`del`和`exec`。

## 5.1 再谈print和import
随着你对Python的认识更加深入，你可能发现有些自以为熟悉的方面隐藏着让人惊喜的特性。下面来看看`print`和`import`的几个好的特性。

### 5.1.1 打印多个参数
实际上，`print()`可以同时打印多个表达式，用逗号分隔。

```python
>>> print('Age:', 42)
Age: 42
```

可以看到，参数之间插入了一个空格字符。当你想组合文本和变量值，而又不想使用字符串格式化功能时，这种行为很有帮助。

```python
>>> name = 'Gumby'
>>> salutation = 'Mr.'
>>> greeting = 'Hello,'
>>> print(greeting, salutation, name)
Hello, Mr. Gumby
```

可以使用关键字参数`sep`自定义分隔符，默认为空格。

```python
>>> print('I', 'wish', 'to', 'register', 'a', 'complaint', sep='_')
I_wish_to_register_a_complaint
```

也可以使用关键字参数`end`自定义结束字符串，默认为换行符。例如，如果提供一个空字符串，则可以继续打印到当前行。

```python
print('Hello, ', end='')
print('world!')
```

上述代码打印 "Hello, world!" 。

注：这只在脚本中起作用。而在交互式解释器中，每条语句是单独执行的：

```python
>>> print('Hello, ', end='')
Hello, >>> print('world!')
world!
```

注：参数`file`见11.3.5节。

### 5.1.2 导入时重命名
从模块导入时，通常使用以下形式之一：

```python
import module
from module import function
from module import function, anotherfunction, yetanotherfunction
from module import *
```

仅当你确定要导入模块中的**一切**时，才使用第四种形式。

还可以在末尾添加`as`子句指定别名：

```python
import module as module_alias
from module import function as function_alias
```

例如：

```python
>>> import math as foobar
>>> foobar.sqrt(4)
2.0
```

```python
>>> from math import sqrt as foobar
>>> foobar(4)
2.0
```

如果两个模块包含同名函数，可以导入模块并通过模块名调用函数，也可以导入函数并指定不同的别名。

注：有些模块（例如`os.path`）组成了层次结构（一个模块位于另一个模块中），详见10.1.4节。

## 5.2 赋值魔法
### 5.2.1 序列解包
可以**同时**进行多个赋值操作。

```python
>>> x, y, z = 1, 2, 3
>>> print(x, y, z)
1 2 3
```

这等价于

```python
values = 1, 2, 3
x, y, z = values
```

使用这种方式还可以交换两个（或多个）变量的值：

```python
>>> x, y = y, x
>>> print(x, y)
2 1
```

这叫做**序列解包**(sequence unpacking)：将一个值序列解包到一个变量序列（即`=`右端的每个值分别赋给左端对应的变量）。

当函数或方法返回元组（或其他序列或可迭代对象）时这很有用。

```python
>>> scoundrel = {'name': 'Robin', 'girlfriend': 'Marion'}
>>> key, value = scoundrel.popitem()
>>> key
'girlfriend'
>>> value
'Marion'
```

要解包的序列包含的元素个数必须与`=`左端的变量个数相同，否则将引发异常。

```python
>>> x, y, z = 1, 2
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: not enough values to unpack (expected 3, got 2)
>>> x, y, z = 1, 2, 3, 4
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: too many values to unpack (expected 3)
```

可以使用星号运算符(`*`)来收集多余的值。

```python
>>> a, b, *rest = [1, 2, 3, 4]
>>> rest
[3, 4]
>>> name = 'Albus Percival Wulfric Brian Dumbledore'
>>> first, *middle, last = name.split()
>>> middle
['Percival', 'Wulfric', 'Brian']
```

这种收集方式也可用于函数参数列表（见第6章）。

注：
* `=`左端至多包含一个带星号的变量。
* 当`=`左端包含带星号的变量时，右端序列的元素个数至少为左端变量个数-1。
* 带星号变量的赋值结果总是一个列表。

```python
>>> a, *b, *c = 'abc'
  File "<stdin>", line 1
SyntaxError: multiple starred expressions in assignment
>>> a, *b, c = 'abbc'
>>> b
['b', 'b']
>>> a, *b, c = 'abc'
>>> b
['b']
>>> a, *b, c = 'ac'
>>> b
[]
>>> a, *b, c = 'a'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: not enough values to unpack (expected at least 2, got 1)
```

* 使用序列解包时，首先对`=`右端的各个表达式进行求值，然后再赋值给左端的变量。因此，当左端和右端存在“重叠”时（例如交换变量`a, b = b, a`），赋值看起来是同时发生的；而当左端内部存在“重叠”时，赋值是从左到右进行的。例如：

```python
>>> x = [0, 1]
>>> i = 0
>>> i, x[i] = 1, 2
>>> x
[0, 2]
```

### 5.2.2 链式赋值
**链式赋值**(chained assignment)是将多个变量绑定到同一个值的快捷方式。

```python
x = y = f()
```

等价于

```python
x = f()
y = x
```

注意，这不一定等价于

```python
x = f()
y = f()
```

详见5.4.6节`is`运算符。

注：与C++不同，Python的链式赋值是**从左到右**进行的。例如：

```python
>>> x = [0, 1]
>>> i = 0
>>> x[i] = i = 2
>>> x, i
([2, 1], 2)
>>> i = 0
>>> i = x[i] = 2
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IndexError: list assignment index out of range
```

其中，`x[i] = i = 2`等价于`x[i] = 2`（此时`i`为0）然后`i = 2`；`i = x[i] = 2`等价于`i = 2`然后`x[i] = 2`，因此下标越界。

### 5.2.3 增强赋值
`x = x + 1`可以写成`x += 1`，这称为**增强赋值**(augmented assignment)，适用于所有标准运算符。

```python
>>> x = 2
>>> x += 1
>>> x *= 2
>>> x
6
```

注：Python没有`++`和`--`运算符。

也可用于其他数据类型（只要二元运算符本身可用于这些数据类型）。

```python
>>> fnord = 'foo'
>>> fnord += 'bar'
>>> fnord *= 2
>>> fnord
'foobarfoobar'
```

增强赋值可以让代码更紧凑、更简洁，在很多情况下更易读。

## 5.3 代码块：缩进的乐趣
**代码块**(block)并不是一种语句，而是（在条件语句的条件为真时执行或者循环语句多次执行的）一**组**语句。代码块是通过**缩进**(indent)代码（即在前面加空格）来创建的。

注意：也可以使用制表符来缩进代码块，但标准风格是只使用空格，且每级缩进4个空格。

在Python中，冒号(`:`)用来标识代码块的开始，**代码块中的每一行都必须缩进相同的数量**。当返回到与之前相同的缩进量时，就表示当前代码块结束。

## 5.4 条件和条件语句
### 5.4.1 这就是布尔值的作用
现在终于要用到**真值**(truth value)（也叫作**布尔值**(Boolean value)）了。

用作布尔表达式（例如`if`语句的条件）时，下面的值被视为**假**(false)：

```python
False  None  0  ''  ()  []  {}
```

换句话说，标准值`False`和`None`、所有类型的数值0、空序列（例如空字符串、空元组和空列表）以及空映射（例如空字典）都被视为假，其他所有值都被视为**真**(true)，包括特殊值`True`。

注：
* 至少对于内置类型是这样，第9章将介绍如何决定自己创建的对象解释为真还是假。
* 这种区别类似于something vs. nothing，而不是true vs. false。

在Python中，**任何值都可以解释为布尔值**。标准布尔值是`True`和`False`。实际上，`True`和`False`不过是1和0的别名。

```python
>>> True == 1
True
>>> False == 0
True
>>> True + False + 42
43
```

注：在Python中，`bool`是`int`的子类。

布尔值`True`和`False`的类型是`bool`，而`bool`（与`list`、`str`和`tuple`一样）可以用来转换其他值。

```python
>>> bool('I think, therefore I am')
True
>>> bool(42)
True
>>> bool('')
False
>>> bool(0)
False
>>> bool('False')
True
```

注意：虽然`bool([]) == bool('') == False`，但`[] != ''`。

### 5.4.2 条件执行和if语句
`if`语句可以实现**条件执行**。语法如下：

```python
if condition:
    block
```

如果条件（表达式`condition`）为真，则执行代码块`block`，否则不执行。

[if语句示例](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch05/if_statement_example.py)

注：在Python中，`if x`和`if x is True`并不完全相同——前者的`x`可以是任何类型，而后者的`x`必须是`bool`类型。

### 5.4.3 else子句
可以使用`else`子句增加一种选择：

```python
if condition:
    block1
else:
    block2
```

如果`condition`为真，则执行`block1`，否则执行`block2`。

[else子句示例](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch05/else_clause_example.py)

`if`语句还有一个近亲——**条件表达式**(conditional expression)，类似于C语言的三目运算符：

```python
expr1 if condition else expr2
```

这是一个表达式（而不是语句），如果`condition`为真，则结果为`expr1`，否则为`expr2`。相当于C语言的`condition ? expr1 : expr2`。例如：

```python
status = 'friend' if name.endswith('Gumby') else 'stranger'
```

### 5.4.4 elif子句
如果要检查多个条件，可以使用`elif`（ "else if" 的缩写）：

```python
if expr1:
    block1
elif expr2:
    block2
elif expr3:
    block3
...
else:
    blockn
```

[elif子句示例](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch05/elif_clause_example.py)

注：Python没有`switch`语句，但Python 3.10引入了更强大的`match`语句。

### 5.4.5 嵌套代码块
可以将`if`语句放在其他`if`语句块中。例如：

[嵌套代码块示例](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch05/nested_blocks_example.py)

### 5.4.6 更复杂的条件
#### 比较运算符
在条件中最基本的运算符是**比较运算符**(comparison operator)，如下表所示。

| 表达式 | 描述 |
| --- | --- |
| `x == y` | `x`等于`y` |
| `x != y` | `x`不等于`y` |
| `x < y` | `x`小于`y` |
| `x > y` | `x`大于`y` |
| `x <= y` | `x`小于等于`y` |
| `x >= y` | `x`大于等于`y` |
| `x is y` | `x`和`y`是同一个对象 |
| `x is not y` | `x`和`y`是不同对象 |
| `x in y` | `x`是容器`y`的成员 |
| `x not in y` | `x`不是容器`y`的成员 |

与赋值一样，Python也支持**链式比较**，例如`0 < age < 100`。一般地，`e1 op1 e2 op2 e3 ... eN opN eN+1`等价于`e1 op1 e2 and e2 op2 e3 and ... and eN opN eN+1`。

有些比较运算符需要特别注意。

（1）相等运算符

**相等运算符**(equality operator) `==`用于比较两个对象是否相等。

```python
>>> 'foo' == 'foo'
True
>>> 'foo' == 'bar'
False
```

注意，相等运算符是两个等号，赋值运算符是一个等号。

```python
>>> 'foo' = 'bar'
  File "<stdin>", line 1
    'foo' = 'bar'
    ^^^^^
SyntaxError: cannot assign to literal here. Maybe you meant '==' instead of '='?
```

（2）`is`：同一性运算符

**同一性运算符**(identity operator) `is`检查**同一性**(identity)而不是**相等性**(equality)，即两个变量是否绑定到**同一个对象**。

注：Python中`is`和`==`的区别类似于C++中比较指针(`p == q`)和比较指针指向的对象(`*p == *q`)的区别。

```python
>>> x = y = [1, 2, 3]
>>> z = [1, 2, 3]
>>> x == y
True
>>> x == z
True
>>> x is y
True
>>> x is z
False
```

其中，`x`和`y`绑定到同一个列表，而`z`绑定到另外一个具有相同元素的列表。`x`和`z`相等，但不是同一个对象，如下图所示。

![is和==运算符](/assets/images/python-note-ch05-conditionals-loops-and-some-other-statements/is和==运算符.png)

注意：不要将`is`用于数字和字符串等基本的、不可变的类型。由于Python内部处理这些对象的方式，结果是不可预测的。

（3）`in`：成员资格运算符

**成员资格运算符**(membership operator) `in`已在2.2.5节介绍过。

```python
name = input('What is your name?')
if 's' in name:
    print('Your name contains the letter "s".')
else:
    print('Your name does not contain the letter "s".')
```

（4）字符串和序列比较

字符串是按照**字典序**(lexicographically)进行比较的。

```python
>>> 'alpha' < 'beta'
True
>>> 'a' < 'B'
False
>>> '2' > '10'
True
```

字符是根据顺序值(ordinal value)排序的（因此小写字母排在大写字母之后）。`ord()`函数返回字符的顺序值，而`chr()`函数相反。

```python
>>> ord('a')
97
>>> ord('B')
66
>>> chr(98)
'b'
>>> chr(25105)
'我'
```

其他序列的比较方式与此相同，只是元素可能不是字符，而是其他类型。

```python
>>> [1, 2] < [2, 1]
True
>>> [1, 2] < [1, 2, 3]
True
>>> [2, [1, 4]] < [2, [1, 5]]
True
>>> [1, 2] == (1, 2)
False
```

注：
* 两个集合相等必须满足：具有相同类型、相同长度，且对应元素分别相等。
* 支持比较大小的集合：排序与第一个不相等元素的排序相同（例如，`[1, 2, x] < [1, 2, y]`与`x < y`相同）。如果对应元素不存在，则较短的集合排序在前（例如，`[1, 2] < [1, 2, 3]`为真）。

#### 布尔运算符
如果需要检查多个条件，可以使用**布尔运算符**(Boolean operator)：`and`、`or`和`not`。例如，输入一个数字并检查是否位于1~10之间：

```python
number = int(input('Enter a number between 1 and 10: '))
if number <= 10 and number >= 1:
    print('Great!')
else:
    print('Wrong!')
```

注：可以使用链式比较`1 <= number <= 10`进一步简化这个示例。

这三个布尔运算符可以任意组合：

```python
if ((cash > price) or customer_has_good_credit) and not out_of_stock:
    give_goods()
```

注：优先级`not` > `and` > `or`。

##### 短路逻辑
布尔运算符有个有趣的特性：只做必要的求值。例如：
* `x and y`等价于条件表达式`x if not x else y`（如果`x`为假则`y`不会被求值）
* `x or y`等价于条件表达式`x if x else y`（如果`x`为真则`y`不会被求值）

这种行为叫做**短路逻辑**(short-circuit logic)（或**惰性求值**(lazy evaluation)）。

注意，Python的布尔运算符不仅可用于布尔值，还可用于任何类型的值：

```python
>>> 'foo' and 'bar'
'bar'
>>> '' and 'bar'
''
>>> 'foo' or 'bar'
'foo'
>>> '' or 'bar'
'bar'
```

下面的代码就利用了这种行为：

```python
name = input('Please enter your name: ') or '<unknown>'
```

如果没有输入名字，则`or`表达式的值为`'<unknown>'`。

### 5.4.7 断言
`assert`语句用于实现**断言**(assertion)：

```python
assert condition
```

这等价于

```python
if not expression:
    raise AssertionError
```

让程序在错误条件出现时立即崩溃好过以后再崩溃。如果必须满足特定条件（例如检查函数参数的属性或为初期测试和调试提供帮助），程序才能正确运行，可以在程序中添加`assert`语句充当检查点。

```python
>>> age = 10
>>> assert 0 < age < 100
>>> age = -1
>>> assert 0 < age < 100
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AssertionError
```

可以在条件后添加一个字符串，作为解释。

```python
>>> age = -1
>>> assert 0 < age < 100, 'The age must be realistic'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AssertionError: The age must be realistic
```

## 5.5 循环
### 5.5.1 while循环
`while`语句用于在条件为真时重复执行代码块：

```python
while condition:
    block
```

例如，打印1~100的所有数字：

```python
x = 1
while x <= 100:
  print(x)
  x += 1
```

还可以使用循环来确保用户输入名字：

[while语句示例](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch05/while_statement_example.py)

注：Python 3.8引入了赋值表达式：`v := expr`将表达式`expr`赋给变量`v`，同时返回`expr`的值，`:=`叫做海象运算符。赋值表达式可用于`if`和`while`语句在赋值的同时判断变量的值。例如，打印所有输入的数字，直到遇到负数：

```python
while (x := int(input())) >= 0:
    print(x)
```

详见官方文档[Assignment expressions](https://docs.python.org/3/reference/expressions.html#assignment-expressions)。

### 5.5.2 for循环
`for`循环用于遍历序列（或其他可迭代对象）的元素：

```python
for target in expr:
    block
```

注：`for`语句的本质是迭代器（详见第9章），上面的语句等价于

```python
it = iter(expr)
while True:
    try:
        target = next(it)
        block
    except StopIteration:
        break
```

例如：

```python
words = ['this', 'is', 'an', 'ex', 'parrot']
for word in words:
    print(word)
```

或

```python
numbers = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
for number in numbers:
    print(number)
```

由于迭代（即遍历）特定范围内的数字是很常见的，Python提供了一个创建范围的内置函数`range()`。

```python
>>> range(0, 10)
range(0, 10)
>>> list(range(0, 10))
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>>> list(range(1, 10, 2))
[1, 3, 5, 7, 9]
>>> list(range(9, -1, -1))
[9, 8, 7, 6, 5, 4, 3, 2, 1, 0]
```

范围类似于切片。`range()`函数有三个参数：`start`、`stop`和`step`。它包含下限，但不包含上限。下限默认为0，步长默认为1。

下面的程序打印1~100的所有数字，比`while`循环更紧凑：

```python
for number in range(1, 101):
    print(number)
```

只要能使用`for`循环，就不要使用`while`循环。

### 5.5.3 迭代字典
可以使用`for`语句遍历字典的键。

```python
d = {'x': 1, 'y': 2, 'z': 3}
for key in d:
    print(key, 'corresponds to', d[key])
```

也可以使用`keys()`方法来获取键。如果只需要值，可以使用`values()`。`items()`以元组形式返回键值对。`for`循环的优点之一是可以使用序列解包。

```python
for key, value in d.items():
    print(key, 'corresponds to', value)
```

注意：字典元素的顺序是不确定的。如果顺序很重要，可以将键或值存储在列表中，对其排序后再遍历。

### 5.5.4 一些迭代工具
Python提供了一些函数，在迭代序列（或其他可迭代对象）时非常有用。其中一些在`itertools`模块（第10章将介绍），还有一些内置函数也非常方便。

#### 并行迭代
`zip()`函数用于并行迭代两个或多个序列，返回一个元组的序列，其中第i个元组是由各序列的第i个元素组成的。

```python
>>> list(zip('abc', [1, 2, 3]))
[('a', 1), ('b', 2), ('c', 3)]
```

例如，有下面两个列表：

```python
names = ['anne', 'beth', 'george', 'damon']
ages = [12, 45, 32, 102]
```

如果要打印名字和对应的年龄，可以这样做：

```python
for i in range(len(names)):
    print(names[i], 'is', ages[i], 'years old')
```

使用`zip()`函数：

```python
for name, age in zip(names, ages):
    print(name, 'is', age, 'years old')
```

`zip()`可以处理不同长度的序列，当最短的序列用完后停止。

```python
>>> list(zip('ABCDE', range(100000000)))
[('A', 0), ('B', 1), ('C', 2), ('D', 3), ('E', 4)]
```

#### 带索引的迭代
`enumerate()`函数用于迭代索引-值对，可以通过`start`参数指定起始索引（默认为0）。

```python
>>> list(enumerate(['Spring', 'Summer', 'Fall', 'Winter']))
[(0, 'Spring'), (1, 'Summer'), (2, 'Fall'), (3, 'Winter')]
```

例如，要替换一个字符串列表中所有包含子串`'xxx'`的字符串，可以这样做：

```python
for i in range(len(strings)):
    if 'xxx' in strings[i]:
        strings[i] = '[censored]'
```

使用`enumerate()`函数：

```python
for i, string in enumerate(strings):
    if 'xxx' in string:
        strings[i] = '[censored]'
```

#### 反向和有序迭代
函数`reversed()`和`sorted()`类似于列表方法`reverse()`和`sort()`，但可用于任何序列或可迭代对象，且不是原地修改对象，而是返回反转和排序后的版本。

```python
>>> sorted([4, 3, 6, 8, 3])
[3, 3, 4, 6, 8]
>>> sorted('Hello, world!')
[' ', '!', ',', 'H', 'd', 'e', 'l', 'l', 'l', 'o', 'o', 'r', 'w']
>>> list(reversed('Hello, world!'))
['!', 'd', 'l', 'r', 'o', 'w', ' ', ',', 'o', 'l', 'l', 'e', 'H']
>>> ''.join(reversed('Hello, world!'))
'!dlrow ,olleH'
```

注意，`sorted()`返回一个列表，而`reversed()`返回一个迭代器。

与`list.sort()`一样，`sorted()`函数也支持`key`和`reverse`参数。

### 5.5.5 跳出循环
#### break
`break`语句用于结束（跳出）循环。例如，要找出100以内的最大平方数：

[break语句示例](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch05/break_statement_example.py)

#### continue
`continue`语句用于结束当前迭代，并跳到下一次迭代开头。即跳过剩余循环体，但不结束循环。

#### while True/break
假设要在用户根据提示输入单词时执行某种操作，并在没有提供单词时结束循环。一种方式如下：

```python
word = 'dummy'
while word:
  word = input('Please enter a word: ')
  # do something with the word:
  print('The word was', word)
```

代码正确工作，但是有些丑。哑值(dummy value)通常意味着做法不太对。下面尝试将其消除：

```python
word = input('Please enter a word: ')
while word:
  # do something with the word:
  print('The word was ', word)
  word = input('Please enter a word: ')
```

哑值消除了，但包含重复的代码，这样也不好。为了避免重复，可以使用`while True/break`习惯用法：

[while True/break示例](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch05/while_true_break_idiom_example.py)

`if/break`行自然地将循环分成两部分：第一部分负责初始化，第二部分在条件为真时使用第一部分初始化的数据。

注：这个示例可以使用赋值表达式进一步简化：

```python
while word := input('Please enter a word: '):
    print('The word was ', word)
```

### 5.5.6 循环中的else子句
循环语句可以添加`else`子句，只有当没有调用`break`（即循环正常结束）时才执行。继续前面`break`一节的示例：

[循环中的else子句示例](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch05/else_clause_in_loop_example.py)

## 5.6 推导式——轻量级循环
**列表推导式**(list comprehension)是一种从其他列表创建列表的方式。其工作原理非常简单，类似于`for`循环。

```python
>>> [x * x for x in range(10)]
[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
```

这个列表由`range(10)`中每个值的平方组成。类似于数学上的集合推导式 $ \lbrace x^2 \vert x \in \mathbb{Z} \land 0 \le x < 10 \rbrace $ 。

如果只需要能被3整除的平方数，可以添加一个`if`子句。

```python
>>> [x * x for x in range(10) if x % 3 == 0]
[0, 9, 36, 81]
```

也可以添加多个`for`。

```python
>>> [(x, y) for x in range(3) for y in range(3)]
[(0, 0), (0, 1), (0, 2), (1, 0), (1, 1), (1, 2), (2, 0), (2, 1), (2, 2)]
```

下面的两个`for`循环创建同样的列表：

```python
result = []
for x in range(3):
    for y in range(3)
        result.append((x, y))
```

使用多个`for`时也可以添加`if`子句。

```python
>>> girls = ['alice', 'bernice', 'clarice']
>>> boys = ['chris', 'arnold', 'bob']
>>> [b + '+' + g for b in boys for g in girls if b[0] == g[0]]
['chris+clarice', 'arnold+alice', 'bob+bernice']
```

这会将名字首字母相同的男孩和女孩配对。

使用圆括号代替方括号并不是“元组推导式”，而是**生成器**，详见9.7.1节。然而，可以使用花括号来创建**字典推导式**(dictionary comprehension)。

```python
>>> squares = {i: '{} squared is {}'.format(i, i**2) for i in range(10)}
>>> squares[8]
'8 squared is 64'
```

## 5.7 三人行
### 5.7.1 什么都不做
有时候什么都不用做。此时可以使用`pass`语句。

```python
>>> pass
>>>
```

在编写代码时，可以将其用作占位符。例如：

```python
if name == 'Ralph Auldus Melish':
    print('Welcome!')
elif name == 'Enid':
    # Not finished yet ...
    pass
elif name == 'Bill Gates':
    print('Access Denied')
```

### 5.7.2 使用del删除
一般来说，Python会删除那些不再使用的对象（因为没有任何变量或数据结构成员引用它）（即引用计数变为0），这被称为**垃圾收集**(garbage collection)。

另一种方式是使用`del`语句。它不仅会删除对象的引用，还会删除名字本身。

```python
>>> x = 1
>>> del x
>>> x
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'x' is not defined
```

然而，在下面的示例中，删除`x`不会影响`y`，也不会删除列表本身（因为变量`y`还引用该列表）。

```python
>>> x = y = ['Hello', 'Python']
>>> del x
>>> y
['Hello', 'Python']
```

`del`语句还可用于删除序列和字典元素，见2.3.2和4.2.2节。

### 5.7.3 使用exec和eval执行和求值字符串
有时候，你可能想动态地("on the fly")创建Python代码，并将其作为语句执行或作为表达式求值。这可能犹如黑暗魔法，一定要小心。

警告：本节介绍如何执行存储在字符串中的Python代码，这样做是一个严重的潜在安全漏洞。如果将用户提供的字符串作为代码执行，将无法控制代码的行为（[代码注入攻击](https://en.wikipedia.org/wiki/Code_injection)）。在网络应用程序（例如第15章将介绍的CGI脚本）中，这样做尤其危险。

#### exec
内置函数`exec()`用于执行("execute")一个字符串。

```python
>>> exec("print('Hello, world!')")
Hello, world!
```

然而，使用单个参数调用`exec()`绝非好事。还应该提供一个**命名空间**(namespace)/**作用域**(scope)——用于放置变量的地方。否则，代码将污染你的命名空间（即修改你的变量）。

```python
>>> from math import sqrt
>>> exec('sqrt = 1')
>>> sqrt(4)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'int' object is not callable
```

为此，需要添加第二个参数——用作命名空间的字典。

```python
>>> from math import sqrt
>>> scope = {}
>>> exec('sqrt = 1', scope)
>>> sqrt(4)
2.0
>>> scope['sqrt']
1
```

#### eval
内置函数`eval()`类似于`exec()`，用于求值("evaluate")Python表达式（字符串形式），并返回结果值。例如，可以使用如下代码来创建一个Python计算器：

```python
>>> eval(input('Enter an arithmetic expression: '))
Enter an arithmetic expression: 6 + 18 * 2
42
```

与`exec()`一样，也可以向`eval()`提供一个命名空间，虽然表达式通常不会像语句那样给变量重新赋值。
