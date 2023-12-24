---
title: 《Python基础教程》笔记 第2章 列表和元组
date: 2023-11-27 21:38:21 +0800
categories: [Python, Beginning Python]
tags: [python, sequence, slice, list, tuple, method, sort]
---
本章将介绍一个新概念：**数据结构**(data structure)。数据结构是以某种方式组织起来的数据元素的集合。在Python中，最基本的数据结构是**序列**(sequence)。

## 2.1 序列概述
Python有多种内置序列，本章重点讨论其中最常用的两种：**列表**(list)和**元组**(tuple)。字符串是另一种重要的序列，将在下一章讨论。

列表和元组的主要区别在于，列表是可以修改的，而元组不可以。

在需要处理一系列值时，序列很有用。例如，用序列表示数据库中一个人的信息，第一个元素是姓名，第二个元素是年龄。使用列表表示如下：

```python
>>> edward = ['Edward Gumby', 42]
```

列表的元素用方括号`[]`括起来，使用逗号分隔。

序列也可以包含其他的序列，因此可以创建一个人员的列表：

```python
>>> edward = ['Edward Gumby', 42]
>>> john = ['John Smith', 50]
>>> database = [edward, john]
>>> database
[['Edward Gumby', 42], ['John Smith', 50]]
```

注：Python有一个数据结构的基本概念，称为**容器**(container)。序列（例如列表和元组）、映射（例如字典）和集合是三类主要的容器。

## 2.2 通用序列操作
所有序列类型都支持一些共同的操作，包括**索引、切片、相加、相乘和成员资格检查**。另外，Python还提供了计算序列长度、找出最大和最小元素的内置函数。

注意：这里没有提到的一个重要操作是**迭代**，详见5.5节。

另见官方文档[Common Sequence Operations](https://docs.python.org/3/library/stdtypes.html#common-sequence-operations)。

### 2.2.1 索引
序列中的所有元素都有编号，**从0开始**递增。可以使用**索引**(indexing)来访问元素：`s[i]`是序列s的第i个元素。

```python
>>> greeting = 'Hello'
>>> greeting[0]
'H'
```

注意：字符串就是字符的序列。 索引0指向第一个元素，在这里是字母H。不同于其他一些语言，Python没有单独的字符类型。一个字符就是只包含一个元素的字符串。

**使用负数索引时，将从右往左数。** 最后一个元素的位置是-1。

```python
>>> greeting[-1]
'o'
```

注：
* `s[-i]`等价于`s[len(s) - i]` (`0 <= i < len(s)`)
* 如果序列长度为n，则索引的合法范围是`-n <= i < n`，否则将报错下标越界。

字符串字面值（以及其他序列字面值）可以直接索引。例如：

```python
>>> 'Hello'[1]
'e'
```

返回序列的函数调用也可以直接索引。例如：

```python
>>> fourth = input('Year: ')[3]
Year: 2005
>>> fourth
'5'
```

代码清单2-1包含的程序要求你输入年、月（1~12的数字）、日（1~31），然后使用相应的月份名等将日期打印出来。

[代码清单2-1 索引示例](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch02/indexing_example.py)

该程序的一个交互示例如下：

```
Year: 1974
Month (1-12): 8
Day (1-31): 16
August 16th, 1974
```

### 2.2.2 切片
可以使用**切片**(slicing)来访问特定**范围**内的元素。为此，需要使用两个索引，用冒号分隔：`s[i:j]`是序列s从i到j的切片，即下标满足i <= x < j的元素构成的子序列。

```python
>>> tag = '<a href="http://www.python.org">Python web site</a>'
>>> tag[9:30]
'http://www.python.org'
>>> tag[32:-4]
'Python web site'
```

切片用于提取序列的一部分。第一个索引是包含的（即切片的第一个元素），第二个是不包含的（即切片最后一个元素的下一个位置），即左闭右开区间：

```python
>>> numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
>>> numbers[3:6]
[4, 5, 6]
>>> numbers[0:1]
[1]
```

#### 绝妙的简写
假设要访问`numbers`的最后三个元素，可以显式指定：

```python
>>> numbers[7:10]
[8, 9, 10]
```

虽然索引10并不存在，但它是最后一个元素的下一个位置。

如果使用负数索引，则无法包含最后一个元素，因为不能用索引0表示“末尾的下一个位置”：

```python
>>> numbers[-3:-1]
[8, 9]
>>> numbers[-3:0]
[]
```

实际上，当切片的第一个索引位于第二个索引之后时，结果就是空序列。好在可以使用一种简写：**如果切片结束于序列末尾，可以省略第二个索引；如果开始于序列开头，可以省略第一个索引；如果两个都省略，则拷贝整个序列。**

![切片](/assets/images/python-note-ch02-lists-and-tuples/切片.png)

```python
>>> numbers[-3:]
[8, 9, 10]
>>> numbers[:3]
[1, 2, 3]
>>> numbers[:]
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

代码清单2-2包含的程序提示用户输入一个URL（假定形式为<http://www.somedomainname.com>），并从中提取域名。

[代码清单2-2 切片示例](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch02/slicing_example.py)

#### 更大的步长
切片的第三个参数是步长：`s[i:j:k]`是序列s从i到j、步长为k的切片，即下标满足x = i, i+k, i+2k...且i <= x < j的元素构成的子序列。步长默认为1。

```python
>>> numbers[0:10:1]
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
>>> numbers[0:10:2]
[1, 3, 5, 7, 9]
>>> numbers[3:6:3]
[4]
>>> numbers[::4]
[1, 5, 9]
```

步长不能为0，但可以为负数，即从右向左提取元素。

```python
>>> numbers[8:3:-1]
[9, 8, 7, 6, 5]
>>> numbers[10:0:-2]
[10, 8, 6, 4, 2]
>>> numbers[0:10:-2]
[]
>>> numbers[::-2]
[10, 8, 6, 4, 2]
>>> numbers[5::-2]
[6, 4, 2]
>>> numbers[:5:-2]
[10, 8]
>>> numbers[::0]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: slice step cannot be zero
```

当步长为负时，起始索引仍然是包含的，终止索引仍然是不包含的，起始索引必须大于终止索引。省略起始和终止索引时，Python会执行“正确的操作”：
* 当步长为正时，省略起始索引表示序列开头，省略终止索引表示序列结尾的下一个位置。
* 当步长为负时，省略起始索引表示序列结尾，省略终止索引表示序列开头的前一个位置。

注：与索引操作不同，切片的索引超过合法范围并不会导致下标越界
* 当步长为正时，如果切片的索引超过`len(s)`则等价于`len(s)`。例如，`numbers[7:20]`等价于`numbers[7:10]`。
* 当步长为负时，如果切片的索引超过`len(s)-1`则等价于`len(s)-1`。例如，`numbers[10:0:-2]`等价于`numbers[9:0:-2]`。

### 2.2.3 序列相加
可以使用`+`运算符来拼接序列。不能拼接不同类型的序列。

```python
>>> [1, 2, 3] + [4, 5, 6]
[1, 2, 3, 4, 5, 6]
>>> 'Hello, ' + 'world!'
'Hello, world!'
>>> [1, 2, 3] + 'world!'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: can only concatenate list (not "str") to list
```

### 2.2.4 乘法
将序列乘以数字x会创建一个新序列，将原序列重复x次。

```python
>>> 'python' * 5
'pythonpythonpythonpythonpython'
>>> [42] * 10
[42, 42, 42, 42, 42, 42, 42, 42, 42, 42]
>>> [1, 2, 3] * 3
[1, 2, 3, 1, 2, 3, 1, 2, 3]
```

注：序列乘法操作是**浅拷贝**，重复的元素将引用相同的对象。对于可变类型的元素，这可能会导致意外的结果（另见4.2.4节字典的`copy()`方法）。例如：

```python
>>> a = [0] * 3
>>> a
[0, 0, 0]
>>> a[0] = 42
>>> a
[42, 0, 0]
```

```python
>>> b = [[]] * 3
>>> b
[[], [], []]
>>> b[0].append(42)
>>> b
[[42], [42], [42]]
```

```python
>>> c = [[] for _ in range(3)]
>>> c
[[], [], []]
>>> c[0].append(42)
>>> c
[[42], [], []]
```

列表`a`、`b`和`c`在内存中如下图所示：

![序列乘法-变量a](/assets/images/python-note-ch02-lists-and-tuples/序列乘法-变量a.png)

![序列乘法-变量b](/assets/images/python-note-ch02-lists-and-tuples/序列乘法-变量b.png)

![序列乘法-变量c](/assets/images/python-note-ch02-lists-and-tuples/序列乘法-变量c.png)

#### None、空列表和初始化
空列表用两个方括号表示(`[]`)（无参数的`list()`也返回一个空列表）。如果要创建一个包含10个元素的列表，但没有任何有用的内容，可以使用`None`。在Python中，`None`表示“什么都没有”(nothing here)。

```python
>>> sequence = [None] * 10
>>> sequence
[None, None, None, None, None, None, None, None, None, None]
```

注：与C++/Java中的空指针不同，Python中的`None`是一个真实的对象，其类型是`NoneType`。此类型只有一个全局单例对象，即`None`。因此可以使用`is`运算符来判断一个对象是否为`None`。详见官方文档[The Null Object](https://docs.python.org/3/library/stdtypes.html#the-null-object)。

代码清单2-3包含的程序在屏幕上打印一个由字符组成的方框。方框位于屏幕中央，并根据用户输入的句子适配大小。

[代码清单2-3 序列乘法示例](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch02/sequence_multiplication_example.py)

### 2.2.5 成员资格
要检查一个值是否在序列中，可以使用`in`运算符。该运算符返回**布尔值**(Boolean value)，即`True`或`False`。例如：

```python
>>> permissions = 'rw'
>>> 'w' in permissions
True
>>> 'x' in permissions
False
>>> users = ['mlh', 'foo', 'bar']
>>> input('Enter your user name: ') in users
Enter your user name: mlh
True
>>> subject = '$$$ Get rich now!!! $$$'
>>> '$$$' in subject
True
```

注意：最后一个例子有些不同。一般而言，`in`运算符检查一个对象是否是一个序列（或其他容器）的成员（即元素）。但对于字符串来说，不仅可以检查一个字符是否包含在字符串中，还可以检查一个字符串是否是另一个的子串。例如：

```python
>>> 'P' in 'Python'
True
>>> 'Py' in 'Python'
True
```

注：其他序列不能用`in`检查子序列。例如：

```python
>>> [1, 2] in [1, 2, 3]
False
>>> [1, 2] in [[1, 2], [3]]
True
```

代码清单2-4所示的程序读取一个用户名和一个PIN码，并查询数据库（实际上是一个列表）。如果找到则打印字符串`'Access granted'`。

[代码清单2-4 序列成员资格示例](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch02/sequence_membership_example.py)

### 2.2.6 长度、最小值和最大值
内置函数`len()`返回序列包含的元素个数，`min()`和`max()`分别返回序列中最小和最大的元素。

```python
>>> numbers = [100, 34, 678]
>>> len(numbers)
3
>>> max(numbers)
678
>>> min(numbers)
34
>>> max(2, 3)
3
>>> min(9, 3, 2, 5)
2
```

在最后两个表达式中，调用`max()`和`min()`的参数并不是序列，而是直接将数字作为参数。

## 2.3 列表：Python的主力
本节主要讨论列表与元组和字符串的不同之处：列表是可变的。另外，列表有很多有用的、特有的**方法**。

### 2.3.1 list函数
由于字符串不能像列表一样被修改，因此有时从字符串创建列表会很有用。为此，可以使用`list()`函数。

注：`list`实际上是一个类，而不是函数。

```python
>>> list('Hello')
['H', 'e', 'l', 'l', 'o']
```

注意，`list()`适用于所有类型的序列，而不只是字符串。

注：要将字符列表转换为字符串，可以使用`''.join(somelist)`。详见3.4.3节。

### 2.3.2 基本的列表操作
可以对列表执行所有的标准序列操作。此外，本节将介绍修改列表的方式。

另见官方文档[Mutable Sequence Types](file:///D:/Python/Python311/Doc/html/library/stdtypes.html#mutable-sequence-types)。

#### 元素赋值
要给列表元素赋值，只需使用第1章介绍的赋值语句即可：`s[i] = x`将列表s的第i个元素赋值为x。

```python
>>> x = [1, 1, 1]
>>> x[1] = 2
>>> x
[1, 2, 1]
```

注意：不能给不存在的元素赋值，否则将报错“下标越界”。

#### 删除元素
要从列表中删除元素，只需使用`del`语句：`del s[i]`从列表s中删除第i个元素。

```python
>>> names = ['Alice', 'Beth', 'Cecil', 'Dee-Dee', 'Earl']
>>> del names[2]
>>> names
['Alice', 'Beth', 'Dee-Dee', 'Earl']
```

注：也可以通过切片删除一个范围内的元素：`del s[i:j]`删除切片`s[i:j]`对应的元素，`del s[i:j:k]`删除切片`s[i:j:k]`对应的元素。

#### 切片赋值
切片是一个非常强大的特性，而能够给切片赋值使其显得更加强大：`s[i:j] = t`将切片`s[i:j]`的元素替换为t的元素，`s[i:j:k] = t`将切片`s[i:j:k]`的元素替换为t的元素。

```python
>>> name = list('Perl')
>>> name
['P', 'e', 'r', 'l']
>>> name[2:] = list('ar')
>>> name
['P', 'e', 'a', 'r']
```

这样可以同时给多个元素赋值。使用切片赋值，可以将（步长为1的）切片替换为长度与其不同的序列。

```python
>>> name = list('Perl')
>>> name[1:] = list('ython')
>>> name
['P', 'y', 't', 'h', 'o', 'n']
```

切片赋值还可以在不替换原有元素的情况下**插入**元素。

```python
>>> numbers = [1, 5]
>>> numbers[1:1] = [2, 3, 4]
>>> numbers
[1, 2, 3, 4, 5]
```

在这里，“替换”了一个空切片，相当于插入了一个序列。可以通过相反的操作来删除切片。

```python
>>> numbers
[1, 2, 3, 4, 5]
>>> numbers[1:4] = []
>>> numbers
[1, 5]
```

`numbers[1:4] = []`等价于`del numbers[1:4]`。

下面尝试步长不为1、甚至是负数的切片赋值：

```python
>>> numbers = [1, 2, 3, 4, 5]
>>> numbers[::2] = [6, 8, 10]
>>> numbers
[6, 2, 8, 4, 10]
>>> numbers[::2] = [6, 8]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: attempt to assign sequence of size 2 to extended slice of size 3
>>> numbers[::2] = []
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: attempt to assign sequence of size 0 to extended slice of size 3
>>> numbers = [1, 2, 3, 4, 5]
>>> del numbers[::2]
>>> numbers
[2, 4]
```

```python
>>> numbers = [1, 2, 3, 4, 5]
>>> numbers[-2::-2] = [7, 9]
>>> numbers
[1, 9, 3, 7, 5]
>>> numbers[-2::-2] = [7, 9, 11]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: attempt to assign sequence of size 3 to extended slice of size 2
>>> numbers[-2::-2] = []
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: attempt to assign sequence of size 0 to extended slice of size 2
>>> numbers = [1, 2, 3, 4, 5]
>>> del numbers[-2::-2]
>>> numbers
[1, 3, 5]
```

可以看出：只有给步长为1的切片赋值时，新序列的长度可以与切片不同；否则新序列的长度必须与切片相同（但仍然可以使用`del`语句删除切片）。

### 2.3.3 列表方法
**方法**(method)是与对象（列表、数字、字符串等）紧密联系的函数。一般地，像这样调用方法：

```python
object.method(arguments)
```

第7章将详细介绍方法是什么。列表提供了一些用于查询或修改其内容的方法。

#### append
`append()`方法用于将一个对象附加到列表末尾。

```python
>>> lst = [1, 2, 3]
>>> lst.append(4)
>>> lst
[1, 2, 3, 4]
```

注意：`append()`等方法**原地**(in place)修改列表。这意味着它不是返回一个修改后的新列表，而是直接修改原列表。

#### clear
`clear()`方法清空列表内容。

```python
>>> lst = [1, 2, 3]
>>> lst.clear()
>>> lst
[]
```

这类似于切片赋值`lst[:] = []`。

#### copy
`copy()`方法拷贝一个列表。**普通赋值只是将另一个名字绑定到同一个对象**（即浅拷贝，见1.4节）。

```python
>>> a = [1, 2, 3]
>>> b = a
>>> b[1] = 4
>>> a
[1, 4, 3]
```

要让`a`和`b`是不同的列表，必须使用`copy()`方法。

```python
>>> a = [1, 2, 3]
>>> b = a.copy()
>>> b[1] = 4
>>> a
[1, 2, 3]
```

这类似于使用`a[:]`或`list(a)`。

注：
* 在上面两个示例中，列表`a`和`b`在内存中如下图所示：

![copy方法示例](/assets/images/python-note-ch02-lists-and-tuples/copy方法示例.png)

* `copy()`方法只深拷贝了列表对象，而列表元素仍然是浅拷贝。下面的例子可以证明这一点：

```python
>>> a = [[], [], []]
>>> b = a.copy()
>>> b[1].append(42)
>>> a
[[], [42], []]
```

#### count
`count()`方法统计一个元素在列表中出现的次数。

```python
>>> ['to', 'be', 'or', 'not', 'to', 'be'].count('to')
2
>>> x = [[1, 2], 1, 1, [2, 1, [1, 2]]]
>>> x.count(1)
2
>>> x.count([1, 2])
1
```

#### extend
`extend()`方法可以通过提供一个序列来一次性附加多个值。

```python
>>> a = [1, 2, 3]
>>> b = [4, 5, 6]
>>> a.extend(b)
>>> a
[1, 2, 3, 4, 5, 6]
```

这看起来类似于拼接(`a + b`)，但主要区别是`extend()`方法会修改原列表，而拼接操作返回一个新列表。

`a.extend(b)`等价于`a = a + b`、`a += b`或者`a[len(a):] = b`。

#### index
`index()`方法用于在列表中查找指定值第一次出现的索引，如果未找到则引发`ValueError`。

```python
>>> knights = ['We', 'are', 'the', 'knights', 'who', 'say', 'ni']
>>> knights.index('who')
4
>>> knights.index('herring')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: 'herring' is not in list
```

#### insert
`insert()`方法用于将一个对象插入到列表中。`s.insert(i, x)`等价于`s[i:i] = [x]`。

```python
>>> numbers = [1, 2, 3, 5, 6, 7]
>>> numbers.insert(3, 'four')
>>> numbers
[1, 2, 3, 'four', 5, 6, 7]
```

#### pop
`pop()`方法从列表中删除一个元素（默认为最后一个）并将其返回。

```python
>>> x = [1, 2, 3]
>>> x.pop()
3
>>> x
[1, 2]
>>> x.pop(0)
1
>>> x
[2]
```

使用`append()`和`pop()`方法，可以实现一种常见的数据结构——**栈**(stack)。栈就像一叠盘子，只能在顶部放盘子，也只能从顶部取走盘子，这个原则称为**后进先出**(last-in first-out, LIFO)。

注：如果需要先进先出(first-in first-out, FIFO)的队列(queue)，可以使用`insert(0, ...)`和`pop()`，或者`append()`和`pop(0)`。但更好的解决方案是使用`collections`模块中的`deque`，详见第10章。

#### remove
`remove()`方法用于删除**第一个**等于给定值的元素，如果未找到则引发`ValueError`。`s.remove(x)`等价于`del s[s.index(x)]`。

```python
>>> x = ['to', 'be', 'or', 'not', 'to', 'be']
>>> x.remove('be')
>>> x
['to', 'or', 'not', 'to', 'be']
>>> x.remove('bee')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: list.remove(x): x not in list
```

#### reverse
`reverse()`方法反转列表中的元素。

```python
>>> x = [1, 2, 3]
>>> x.reverse()
>>> x
[3, 2, 1]
```

注：如果要按逆序迭代一个序列，可以使用`reversed()`函数。该函数不修改列表，而是返回一个迭代器。

```python
>>> x = [1, 2, 3]
>>> list(reversed(x))
[3, 2, 1]
```

#### sort
`sort()`方法用于对列表原地排序。

注：从Python 2.3起，`sort()`方法使用的是稳定排序算法。

```python
>>> x = [4, 6, 2, 1, 7, 9]
>>> x.sort()
>>> x
[1, 2, 4, 6, 7, 9]
```

注意，`sort()`方法直接修改列表，没有返回值（实际上返回`None`）。如果需要排序后的列表副本并保留原列表不变，`y = x.sort()`这种写法是错误的（最终结果是`x`已排序，而`y`是`None`）。一种正确方法是先将`y`绑定到`x`的拷贝，然后对`y`进行排序：

```python
>>> x = [4, 6, 2, 1, 7, 9]
>>> y = x.copy()
>>> y.sort()
>>> x
[4, 6, 2, 1, 7, 9]
>>> y
[1, 2, 4, 6, 7, 9]
```

另一种方法是使用`sorted()`函数。

```python
>>> x = [4, 6, 2, 1, 7, 9]
>>> y = sorted(x)
>>> x
[4, 6, 2, 1, 7, 9]
>>> y
[1, 2, 4, 6, 7, 9]
```

该函数可用于任何序列，但总是返回一个列表。

```python
>>> sorted('Python')
['P', 'h', 'n', 'o', 't', 'y']
```

#### 高级排序
`sort()`方法接受两个可选参数：`key`和`reverse`。要使用这两个参数，必须按名称指定（叫做**关键字参数**，详见第6章）。`key`参数提供一个函数，用于为每个元素提供一个键，再按照这些键对元素进行排序（即比较`key(a)`和`key(b)`而不是直接比较`a`和`b`，仍然使用`<`比较）。例如，要根据长度对元素进行排序，使用函数`len()`作为`key`。

```python
>>> x = ['aardvark', 'abalone', 'acme', 'add', 'aerate']
>>> x.sort(key=len)
>>> x
['add', 'acme', 'aerate', 'abalone', 'aardvark']
```

另一个参数`reverse()`是一个布尔值，指出是否按逆序排序。

```python
>>> x = [4, 6, 2, 1, 7, 9]
>>> x.sort(reverse=True)
>>> x
[9, 7, 6, 4, 2, 1]
```

`key`和`reverse`参数也可用于`sorted()`函数。

更多关于排序的内容可参考[Sorting HOW TO](https://docs.python.org/3/howto/sorting.html)。

## 2.4 元组：不可变序列
与列表一样，元组也是序列，唯一的区别是元组**不能修改**（字符串也是如此）。元组语法很简单——只要将一些值用逗号分隔，就自动创建了一个元组。

```python
>>> 1, 2, 3
(1, 2, 3)
```

元组也可以用圆括号括起来。

```python
>>> (1, 2, 3)
(1, 2, 3)
```

空元组用两个不包含任何内容的圆括号表示（无参数的`tuple()`也返回空元组）。

```python
>>> ()
()
```

只包含一个值的元组有些特殊——必须加一个逗号。

```python
>>> 42
42
>>> (42)
42
>>> 42,
(42,)
>>> (42,)
(42,)
```

后两个示例是长度为1的元素，而前两个示例只是一个整数。逗号至关重要，一个逗号可以完全改变表达式的值。

```python
>>> 3 * (40 + 2)
126
>>> 3 * (40 + 2,)
(42, 42, 42)
```

`tuple()`函数与`list()`很像：接受一个序列作为参数，并将其转换为元组。如果参数已经是元组，则原样返回。

注：与`list`一样，`tuple`实际上是类型而不是函数。

```python
>>> tuple([1, 2, 3])
(1, 2, 3)
>>> tuple('abc')
('a', 'b', 'c')
>>> tuple((1, 2, 3))
(1, 2, 3)
```

元组并不复杂，除了创建和访问元素外，没有太多其他操作，使用方式与其他序列相同。

```python
>>> x = (1, 2, 3)
>>> x[1]
2
>>> x[0:2]
(1, 2)
```

元组的切片也是元组。

需要了解元组的两个重要原因：
* 元组可以用作映射的键和集合的成员，而列表不行。映射将在第4章介绍。
* 有些内置函数和方法返回元组。

一般来说，列表足以满足对序列的需求。
