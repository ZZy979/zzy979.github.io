---
title: 《Python基础教程》笔记 第3章 使用字符串
date: 2023-12-11 21:51:08 +0800
categories: [Python, Beginning Python]
tags: [python, string, formatting]
render_with_liquid: false
---
本章将介绍如何使用字符串来格式化其他的值（比如用于打印），并大致了解字符串方法，例如分割、连接、搜索等。

## 3.1 基本字符串操作
所有的标准序列操作都适用于字符串。但是，字符串是不可变的，因此所有的元素和切片赋值都是非法的。

## 3.2 字符串格式化：精简版
将值格式化为字符串是一个重要的操作，需要考虑众多不同的需求。因此随着时间的流式，多种方法被添加到语言中。

历史上，主要使用**字符串格式化运算符** `%`。其行为类似于C语言中经典的`printf()`函数，左边是格式化字符串，右边是要格式化的值（可以是单个值、元组或字典）。

```python
>>> 'Hello, %s!' % 'world'
'Hello, world!'
>>> 'Hello, %s. %s enough for ya?' % ('world', 'Hot')
'Hello, world. Hot enough for ya?'
```

格式化字符串中的`%s`称为**转换说明符**(conversion specifier)。`s`意味着值将被格式化为字符串，如果不是字符串，则使用`str()`将其转换为字符串。其他的转换说明符例如`%d`、`%.3f`等。

详见官方文档[printf-style String Formatting](https://docs.python.org/3/library/stdtypes.html#printf-style-string-formatting)。

另一种方法是使用**模板字符串**，它使用类似于UNIX shell的语法。例如：

```python
>>> from string import Template
>>> tmpl = Template('Hello, $who! $what enough for ya?')
>>> tmpl.substitute(who='Mars', what='Dusty')
'Hello, Mars! Dusty enough for ya?'
```

详见官方文档[Template strings](https://docs.python.org/3/library/string.html#template-strings)。

编写新代码时，应选择使用**字符串方法`format()`**，它结合并扩展了早期方法的优点。每个替换字段都用花括号括起来，其中可能包含名称和格式化信息。

最简单的情况是，字段没有名字，或者名字是索引。

```python
>>> '{}, {} and {}.'.format('first', 'second', 'third')
'first, second and third.'
>>> '{0}, {1} and {2}.'.format('first', 'second', 'third')
'first, second and third.'
```

索引不需要按顺序排列。

```python
>>> '{3} {0} {2} {1} {3} {0}'.format('be', 'not', 'or', 'to')
'to be or not to be'
```

命名字段需要使用关键字参数。

```python
>>> from math import pi
>>> '{name} is approximately {value:.2f}.'.format(value=pi, name='π')
'π is approximately 3.14.'
```

关键字参数的顺序无关紧要。这里还指定了格式说明符`.2f`，用冒号与字段名隔开，意味着使用包含2位小数的浮点数格式。

最后，在Python 3.6中，如果变量与替换字段同名，还可以使用一种简写——**f-字符串**，用前缀`f`表示。

```python
>>> from math import e
>>> f"Euler's constant is roughly {e}."
"Euler's constant is roughly 2.718281828459045."
```

这等价于`"Euler's constant is roughly {e}.".format(e=e)`。

详见官方文档[str.format()](https://docs.python.org/3/library/stdtypes.html#str.format)和[f-strings](https://docs.python.org/3/reference/lexical_analysis.html#f-strings)。

## 3.3 字符串格式化：完整版
字符串格式化涉及的内容非常广泛，因此即使是“完整版”也无法探索所有的细节，而只是介绍主要组成部分。基本思想是对字符串调用`format()`方法，并提供要格式化的值。该字符串包含使用一种[“微型语言”](https://docs.python.org/3/library/string.html#format-specification-mini-language)指定的格式化信息。每个值都被插入到字符串的一个**替换字段**(replacement field)中，每个替换字段都用花括号括起来。要包含花括号本身，可以使用两个花括号（来转义），即`{{`和`}}`。

```python
>>> "{{ceci n'est pas une replacement field}}".format()
"{ceci n'est pas une replacement field}"
```

格式字符串最有趣的部分是替换字段，由以下部分组成，其中每个部分都是可选的：
* **字段名**(field name)：索引或标识符。指定要格式化并插入到该字段的值。
* **转换标志**(conversion flag)：一个感叹号，后面跟着单个字符。如果指定了转换标志，将不使用对象本身的格式化机制，而是使用指定的函数将对象转换为字符串，再做进一步的格式化。
* **格式说明符**(format specifier)：一个冒号，后面跟着一个（格式说明微型语言的）表达式。指定最终格式化的细节。

注：格式说明符的一般形式为`[[填充]对齐][符号][z][#][0][宽度][分组][.精度][类型]`，详见官方文档[Format String Syntax](https://docs.python.org/3/library/string.html#format-string-syntax)。

下面详细介绍其中的一些要素。

### 3.3.1 替换字段名
最简单的情况下，格式字符串中的字段和`format()`的参数都没有名字，字段和参数将按顺序配对。

```python
>>> '{} {} {} {}'.format(1, 2, 3, 4)
'1 2 3 4'
```

可以通过索引指定参数。

```python
>>> '{2} {1} {3} {0}'.format(1, 2, 3, 4)
'3 2 4 1'
```

还可以给参数指定名称，该名称用于替换字段。两种方法可以混合使用。

```python
>>> '{foo} {} {bar} {}'.format(1, 2, bar=4, foo=3)
'3 1 4 2'
>>> '{foo} {1} {bar} {0}'.format(1, 2, bar=4, foo=3)
'3 2 4 1'
```

然而，不能同时使用手动编号和自动编号。

```python
>>> '{} {1} {} {0}'.format(1, 2, 3, 4)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: cannot switch from automatic field numbering to manual field specification
```

替换字段还可以访问值的一部分（索引或属性）。例如：

```python
>>> fullname = ['Alfred', 'Smoketoomuch']
>>> 'Mr {name[1]}'.format(name=fullname)
'Mr Smoketoomuch'
>>> import math
>>> 'The {mod.__name__} module defines the value {mod.pi} for π'.format(mod=math)
'The math module defines the value 3.141592653589793 for π'
```

### 3.3.2 基本转换
指定了字段包含的值之后，就可以添加格式化指令。首先，可以提供一个**转换标志**。

```python
>>> print('{pi!s} {pi!r} {pi!a}'.format(pi='π'))
π 'π' '\u03c0'
```

标志`s`、`r`和`a`分别使用`str()`、`repr()`和`ascii()`进行转换。

还可以指定要转换的值的类型——更准确地说，是要将其视为哪种类型。例如，可以指定一个整数，但将其作为小数处理。为此，可以在格式说明符中使用`:f`。

```python
>>> 'The number is {num}'.format(num=42)
'The number is 42'
>>> 'The number is {num:f}'.format(num=42)
'The number is 42.000000'
```

`:b`将整数格式化为二进制数。

```python
>>> 'The number is {num:b}'.format(num=42)
'The number is 101010'
```

下表列出了不同类型的参数可用的类型说明符。

| 类型说明符 | 参数类型 | 含义 |
| --- | --- | :-- |
| `d` | 整数 | 十进制格式（整数的默认值） |
| `b` | 整数 | 二进制格式 |
| `c` | 整数 | Unicode字符 |
| `o` | 整数 | 八进制格式 |
| `x` | 整数 | 十六进制格式，小写形式 |
| `X` | 整数 | 十六进制格式，大写形式 |
| `e` | 浮点数 | 科学记数法（有效数字位数固定），使用`e`表示指数 |
| `E` | 浮点数 | 与`e`相同，但使用`E`表示指数 |
| `f` | 浮点数 | 定点表示法（小数位数固定） |
| `F` | 浮点数 | 与`f`相同，但对于特殊值（`nan`和`inf`）使用大写形式 |
| `g` | 浮点数 | 自动在定点表示法和科学记数法之间选择，并移除末尾的0（浮点数的默认值） |
| `G` | 浮点数 | 与`g`相同，但使用大写形式表示指数和特殊值 |
| `%` | 浮点数 | 百分数格式 |
| `s` | 字符串 | 字符串格式（字符串的默认值） |

注：
* 整数可以使用浮点数类型说明符，其他类型不能混用。
* 如果省略类型说明符，则根据参数类型选择对应的默认值。

### 3.3.3 宽度、精度和千分位分隔符
格式说明符可以指定宽度和精度：`[宽度][.精度][类型]`。

**宽度**(width)是一个整数，指定格式化后字段的最小宽度。如果大于实际宽度，则用空格（或指定的字符）填充；如果未指定，则由内容决定。

```python
>>> '{:10}'.format(3)
'         3'
>>> '{:10}'.format('Bob')
'Bob       '
```

可以看出，数字和字符串的对齐方式不同。对齐将在下一节介绍。

**精度**(precision)也是一个整数，但前面有一个句点。精度对于不同类型格式化的含义不同：
* 对于浮点数，对于`f`和`e`表示小数位数，对于`g`表示有效数字位数，默认为6。
* 对于字符串，表示最大字符数（如果小于字符串长度则截断）。
* 整数不允许使用。

例如：

```python
>>> from math import pi
>>> 'Pi day is {:.2f}'.format(pi)
'Pi day is 3.14'
>>> '{:.5}'.format('Guido van Rossum')
'Guido'
>>> '{:.2}'.format(12345)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: Precision not allowed in integer format specifier
```

```python
>>> '{:f}'.format(pi), '{:e}'.format(pi), '{:g}'.format(pi)
('3.141593', '3.141593e+00', '3.14159')
>>> x = 1.2345e6
>>> '{:f}'.format(x), '{:e}'.format(x), '{:g}'.format(x)
('1234500.000000', '1.234500e+06', '1.2345e+06')
>>> y = 1.234e-5
>>> '{:f}'.format(y), '{:e}'.format(y), '{:g}'.format(y)
('0.000012', '1.234000e-05', '1.234e-05')
```

宽度和精度可以组合。

```python
>>> '{:10f}'.format(pi)
'  3.141593'
>>> '{:10.2f}'.format(pi)
'      3.14'
>>> '{:.2f}'.format(pi)
'3.14'
```

可以使用逗号添加**千分位分隔符**（放在宽度和表示精度的句点之间）。

```python
>>> 'One googol is {:,}'.format(10**100)
'One googol is 10,000,000,000,000,000,000,000,000,000,000,000,000,000,000,000,000,000,000,000,000,000,000,000,000,000,000,000,000,000,000,000,000,000'
```

### 3.3.4 符号、对齐和用0填充
在宽度和精度之前，可以指定符号、对齐方式和填充字符：`[[填充]对齐][符号][#][0][宽度][.精度][类型]`。

“符号”可以是`+`、`-`或空格：
* `+`表示正数和负数都显示符号。
* `-`表示只有负数显示符号（默认）。
* 空格表示正数前添加一个空格，负数仍然使用负号。

总之，“符号”选项只影响正数的符号。

```python
>>> print('{0:-.2f}\n{1:-.2f}'.format(pi, -pi)) # Default
3.14
-3.14
>>> print('{0:+.2f}\n{1:+.2f}'.format(pi, -pi))
+3.14
-3.14
>>> print('{0: .2f}\n{1: .2f}'.format(pi, -pi))
 3.14
-3.14
```

“对齐”选项分别使用`<`、`>`和`^`表示左对齐、右对齐和居中对齐。

```python
>>> print('|{0:<10.2f}|\n|{0:^10.2f}|\n|{0:>10.2f}|'.format(pi))
|3.14      |
|   3.14   |
|      3.14|
```

注：
* 字符串默认左对齐，数字默认右对齐。
* 只有当手动指定了宽度时对齐选项才有意义。

对齐选项前还可以指定一个填充字符，默认为空格。

```python
>>> '{:$^15}'.format(' WIN BIG ')
'$$$ WIN BIG $$$'
```

还有一个对齐选项`=`，表示将填充字符放在符号和数字之间，该选项只对数字有效。

```python
>>> print('{0:10.2f}\n{1:10.2f}'.format(pi, -pi))
      3.14
     -3.14
>>> print('{0:=+10.2f}\n{1:=+10.2f}'.format(pi, -pi))
+     3.14
-     3.14
```

`0`表示数字用0填充(zero-padded)。

```python
>>> '{:010.2f}'.format(pi)
'0000003.14'
>>> '{:010.2f}'.format(-pi)
'-000003.14'
```

这等价于`'{:0=10.2f}'`。

`#`表示使用“备选形式”，细节随类型而异：
* 对于整数，使用二进制、八进制和十六进制格式时，分别添加前缀`0b`、`0o`、`0x`或`0X`。
* 对于浮点数，强制包含小数点（对于`g`，保留结尾的0）。

```python
>>> '{:b}'.format(42)
'101010'
>>> '{:#b}'.format(42)
'0b101010'
>>> '{:g}'.format(42)
'42'
>>> '{:#g}'.format(42)
'42.0000'
```

代码清单3-1所示的示例打印了格式化后的水果价格表。其中，对同一个字符串使用了两次格式化，第一次用于插入字段宽度作为第二次的格式说明符。因为宽度是由用户提供的，无法以硬编码的方式指定。

[代码清单3-1 字符串格式化示例](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch03/string_formatting_example.py)

注：格式字符串本身就支持两层嵌套，只需调用一次`format()`。无名字段对应的参数由左大括号的出现顺序决定。但为了清晰，最好使用索引或参数名。例如：

```python
>>> '{{:{}}}{{:>{}}}'.format(10, 10).format('Apples', 0.4)
'Apples           0.4'
>>> '{:{}}{:>{}}'.format('Apples', 10, 0.4, 10)
'Apples           0.4'
>>> '{0:{item_width}}{1:>{price_width}}'.format('Apples', 0.4, item_width=10, price_width=10)
'Apples           0.4'
```

## 3.4 字符串方法
与列表相比，字符串的方法要多得多。因此这里只介绍一些最有用的，完整列表参考官方文档[String Methods](https://docs.python.org/3/library/stdtypes.html#string-methods)。

注：除了字符串方法外，`string`模块还提供了一些有用的常量和函数，以及3.2节提到的模板字符串，详见官方文档[string模块](https://docs.python.org/3/library/string.html)。

### 3.4.1 center
`center()`方法通过在两边添加填充字符（默认为空格）让字符串居中。

```python
>>> 'The Middle by Jimmy Eat World'.center(39)
'     The Middle by Jimmy Eat World     '
>>> 'The Middle by Jimmy Eat World'.center(39, '*')
'*****The Middle by Jimmy Eat World*****'
```

注：`s.center(width, fill)`等价于`'{:{fill}^{width}s}'.format(s, width=width, fill=fill)`。

另见：`ljust()`、`rjust()`、`zfill()`。

### 3.4.2 find
`find()`方法在字符串中查找子串，如果找到则返回子串第一次出现的索引，否则返回-1。

```python
>>> 'With a moo-moo here, and a moo-moo there'.find('moo')
7
>>> title = "Monty Python's Flying Circus"
>>> title.find('Monty')
0
>>> title.find('Python')
6
>>> title.find('Flying')
15
>>> title.find('Zirquss')
-1
```

注：
* 如果只需要知道子串是否存在，则使用`in`运算符。
* `index()`类似于`find()`，但如果未找到则引发`ValueError`。

还可以指定搜索的起点和终点：`s.find(sub, start)`和`s.find(sub, start, end)`分别在`s[start:]`和`s[start:end]`中查找子串`sub`。

```python
>>> subject = '$$$ Get rich now!!! $$$'
>>> subject.find('$$$')
0
>>> subject.find('$$$', 1) # Only supplying the start
20
>>> subject.find('!!!')
16
>>> subject.find('!!!', 0, 16) # Supplying start and end
-1
```

注意，与切片一样，起点是包含的，终点是不包含的。

另见：`rfind()`、`index()`、`rindex()`、`count()`、`startswith()`、`endswith()`。

### 3.4.3 join
`join()`是一个非常重要的字符串方法，是`split()`的反向操作，使用当前字符串作为分隔符拼接序列中的字符串。

```python
>>> seq = ['1', '2', '3', '4', '5']
>>> '+'.join(seq) # Joining a list of strings
'1+2+3+4+5'
>>> dirs = '', 'usr', 'bin', 'env'
>>> '/'.join(dirs)
'/usr/bin/env'
>>> dirs = 'C:', 'Windows', 'System32'
>>> print('\\'.join(dirs))
C:\Windows\System32
>>> seq = [1, 2, 3, 4, 5]
>>> '+'.join(seq) # Trying to join a list of numbers
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: sequence item 0: expected str instance, int found
```

可以看到，`join()`的序列元素必须都是字符串。

另见：`split()`。

### 3.4.4 lower
`lower()`方法返回字符串的小写形式。

```python
>>> 'Trondheim Hammer Dance'.lower()
'trondheim hammer dance'
```

如果想要编写不区分大小写的代码，这将很有用。例如，检查列表中是否包含指定的用户名，不区分大小写：

```python
>>> name = 'Gumby'
>>> names = ['gumby', 'smith', 'jones']
>>> if name.lower() in names: print('Found it!')
...
Found it!
```

另见：`islower()`、`istitle()`、`isupper()`、`translate()`、`capitalize()`、`casefold()`、`swapcase()`、`title()`、`upper()`。

注：一个与`lower()`相关的方法是`title()`，它将字符串转换为词首大写(title case)形式——所有单词的首字母都大写，其他字母都小写。然而，它确定单词边界的方式（用非字母字符分割）可能导致不自然的结果。

```python
>>> "that's all folks".title()
"That'S All Folks"
```

另一种方法是使用`string`模块中的`capwords()`函数，该函数使用空格划分单词。

```python
>>> import string
>>> string.capwords("that's all, folks")
"That's All, Folks"
```

要实现真正正确的词首大写（例如冠词、连词、介词等小写），需要自己实现。

### 3.4.5 replace
`replace()`方法将指定子串都替换为另一个字符串，并返回替换后的结果。

```python
>>> 'This is a test'.replace('is', 'eez')
'Theez eez a test'
```

另见：`translate()`、`expandtabs()`。

### 3.4.6 split
`split()`是一个非常重要的字符串方法，是`join()`的反向操作，用于将字符串分割为字符串序列。

```python
>>> '1+2+3+4+5'.split('+')
['1', '2', '3', '4', '5']
>>> '/usr/bin/env'.split('/')
['', 'usr', 'bin', 'env']
>>> 'Using   the   default'.split()
['Using', 'the', 'default']
```

如果没有指定分隔符，则默认为连续的空白符（空格、制表符、换行符等）。

注：如果指定了分隔符，则连续的分隔符不会被合并，而是分割出空字符串，例如：

```python
>>> 'Using   the   default'.split(' ')
['Using', '', '', 'the', '', '', 'default']
```

另见：`join()`、`partition()`、`rpartition()`、`rsplit()`、`splitlines()`。

### 3.4.7 strip
`strip()`方法将字符串开头和末尾的空白符删除，并返回结果。

```python
>>> '    internal whitespace is kept    '.strip()
'internal whitespace is kept'
```

与`lower()`一样，需要将输入与存储的值进行比较时，`strip()`很有用。回到`lower()`一节的用户名示例，假设用户不小心多输入了一个空格：

```python
>>> names = ['gumby', 'smith', 'jones']
>>> name = 'gumby '
>>> if name in names: print('Found it!')
...
>>> if name.strip() in names: print('Found it!')
...
Found it!
```

还可以通过一个字符串参数指定要删除的字符。

```python
>>> '*** SPAM * for * everyone!!! ***'.strip(' *!')
'SPAM * for * everyone'
```

这个方法只删除开头和末尾的指定字符，因此中间的星号未被删除。

另见：`lstrip()`、`rstrip()`。

### 3.4.8 translate
`translate()`方法与`replace()`一样替换字符串的特定部分，但不同的是它只能进行单字符替换。其优势在于能够同时替换多个字符，并且比`replace()`更高效。

考虑一个简单的例子：假设要将一段英语文本转换为带有德国口音的版本。为此，需要将字符c替换为k、s替换为z。

在使用`translate()`之前必须创建一个**转换表**。转换表包含Unicode码点之间的转换关系，使用`str.maketrans()`方法创建。该方法接受两个参数：两个长度相同的字符串，第一个字符串中的每个字符替换为第二个字符串中相应位置的字符。对于这个例子：

```python
>>> table = str.maketrans('cs', 'kz')
>>> table
{99: 107, 115: 122}
>>> 'this is an incredible test'.translate(table)
'thiz iz an inkredible tezt'
```

第三个可选参数指定要删除的字符。例如，可以删除所有的空格：

```python
>>> table = str.maketrans('cs', 'kz', ' ')
>>> 'this is an incredible test'.translate(table)
'thizizaninkredibletezt'
```

另见：`replace()`、`lower()`。

### 3.4.9 isxxx
很多字符串方法都以`is`开头，用于判断字符串是否具有特定的性质（例如空白符、数字或大写字母）。如果字符串非空，且所有字符都具有特定的性质则返回`True`，否则返回`False`。

详见：`isalnum()`、`isalpha()`、`isdecimal()`、`isdigit()`、`isidentifier()`、`islower()`、`isnumeric()`、`isprintable()`、`isspace()`、`istitle()`、`isupper()`。
