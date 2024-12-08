---
title: 《Python基础教程》笔记 第4章 字典：当索引行不通时
date: 2023-12-20 20:58:19 +0800
categories: [Python, Beginning Python]
tags: [python, map]
---
当需要通过编号来访问值时，列表很有用。本章将介绍一种可以通过名称来访问值的数据结构，称为**映射**(mapping)。Python中唯一的内置映射类型是**字典**。

## 4.1 字典的用途
**字典**(dictionary)（不管是现实中的还是Python中的）的结构让你能很容易地查找特定的单词（键）从而找到其定义（值）。

在很多情况下，字典都比列表更合适。例如，如果想创建一个小型数据库，在其中存储一些人的名字和对应的电话号码。一种方法是创建两个列表：

```python
>>> names = ['Alice', 'Beth', 'Cecil', 'Dee-Dee', 'Earl']
>>> numbers = ['2341', '9102', '3158', '0142', '5551']
```

可以像这样查找Cecil的电话号码：

```python
>>> numbers[names.index('Cecil')]
'3158'
```

这样虽然可行，但不太实用。你希望能够像下面这样做：

```python
>>> phonebook['Cecil']
'3158'
```

如果`phonebook`是字典就可以这样做。

## 4.2 创建和使用字典
字典像这样表示：

```python
phonebook = {'Alice': '2341', 'Beth': '9102', 'Cecil': '3158'}
```

字典由**键**(key)和对应的**值**(value)构成的键值对组成，键值对也称为**项**(item)。在上面的例子中，名字是键，电话号码是值。每个键与其值之间用冒号(`:`)分隔，项之间用逗号分隔，整个字典用花括号括起来。空字典用两个花括号表示：`{}`。

注意，在映射中，键必须是唯一的，而值不需要是唯一的。

### 4.2.1 dict函数
可以使用`dict()`函数从其他映射或键值对序列创建字典。

注：与`list`、`tuple`和`str`一样，`dict`并不是真正的函数，而是一个类。

```python
>>> items = [('name', 'Gumby'), ('age', 42)]
>>> d = dict(items)
>>> d
{'name': 'Gumby', 'age': 42}
>>> d['name']
'Gumby'
```

也可以使用关键字参数，例如：

```python
>>> dict(name='Gumby', age=42)
{'name': 'Gumby', 'age': 42}
```

还可以使用一个映射作为`dict()`的参数，这将创建一个包含相同项的字典，另外也可以使用字典的`copy()`方法。无参数的`dict()`返回一个空字典。

### 4.2.2 基本字典操作
字典的基本操作与序列有些类似：
* `len(d)` 返回`d`中项（键值对）的数量
* `d[k]` 返回键`k`关联的值，如果不存在则引发`KeyError`
* `d[k] = v` 将值`v`关联到键`k`
* `del d[k]` 删除键为`k`的项，如果不存在则引发`KeyError`
* `k in d` 检查`d`是否包含键`k`

然而，字典和列表有一些重要的区别：
* **键类型**：字典的键可以是任何**不可变类型**，例如整数、浮点数、字符串或元组；而列表的索引必须是整数。
* **自动添加**：即便是字典中原本没有的键，也可以给它赋值，这将创建一个新项；而不能给列表范围之外的索引赋值。
* **成员资格**：表达式`k in d`（`d`是字典）查找的是键而不是值，而表达式`v in l`（`l`是列表）查找的是值而不是索引。

注：字典的成员资格检查比列表更高效，前者是O(1)的，后者是O(n)的。

```python
>>> x = []
>>> x[42] = 'Foobar'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IndexError: list assignment index out of range
>>> x = {}
>>> x[42] = 'Foobar'
>>> x
{42: 'Foobar'}
```

代码清单4-1展示了电话簿示例的代码。

[代码清单4-1 字典示例](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch04/dictionary_example.py)

### 4.2.3 使用字典的字符串格式化
字符串的`format_map()`方法通过一个映射来指定要格式化的值。

```python
>>> phonebook = {'Beth': '9102', 'Alice': '2341', 'Cecil': '3258'}
>>> "Cecil's phone number is {Cecil}.".format_map(phonebook)
"Cecil's phone number is 3258."
```

这种字符串格式化在模板系统（在这里是HTML）中很有用。

```python
>>> template = '''<html>
... <head><title>{title}</title></head>
... <body>
... <h1>{title}</h1>
... <p>{text}</p>
... </body>'''
>>> data = {'title': 'My Home Page', 'text': 'Welcome to my home page!'}
>>> print(template.format_map(data))
<html>
<head><title>My Home Page</title></head>
<body>
<h1>My Home Page</h1>
<p>Welcome to my home page!</p>
</body>
```

### 4.2.4 字典方法
完整列表参考官方文档[Mapping Types — dict](https://docs.python.org/3/library/stdtypes.html#typesmapping)。

#### clear
`clear()`方法删除字典中所有的项。这是一个原地操作。

```python
>>> d = {'age': 42, 'name': 'Gumby'}
>>> d.clear()
>>> d
{}
```

#### copy
`copy()`方法返回一个具有相同键值对的新字典（**浅拷贝**，因为值本身没有没拷贝）。

```python
>>> x = {'username': 'admin', 'machines': ['foo', 'bar', 'baz']}
>>> y = x.copy()
>>> y['username'] = 'mlh'
>>> y['machines'].remove('bar')
>>> y
{'username': 'mlh', 'machines': ['foo', 'baz']}
>>> x
{'username': 'admin', 'machines': ['foo', 'baz']}
```

可以看到，当**替换**副本中的值时，原对象不受影响。然而，如果**修改**副本中的值，原对象也将发生变化。

注：如1.4节所述，Python变量相当于C++的指针变量，“替换”=改变指针指向，“修改”=通过指针修改对象。在上面的示例中，字典`x`和`y`在内存中如下图所示：

![copy方法示例](/assets/images/python-note-ch04-dictionaries-when-indices-wont-do/copy方法示例.png)

避免这个问题的一种方法是使用**深拷贝**。为此，使用`copy`模块中的`deepcopy()`函数。

```python
>>> from copy import deepcopy
>>> d = {'names': ['Alfred', 'Bertrand']}
>>> c = d.copy()
>>> dc = deepcopy(d)
>>> d['names'].append('Clive')
>>> c
{'names': ['Alfred', 'Bertrand', 'Clive']}
>>> dc
{'names': ['Alfred', 'Bertrand']}
```

#### fromkeys
`fromkeys()`方法使用给定的键创建一个新字典，每个键对应指定的值（默认为`None`）。

注：该方法是类方法，可以直接对类型`dict`调用。

```python
>>> dict.fromkeys(['name', 'age'])
{'name': None, 'age': None}
>>> dict.fromkeys(['name', 'age'], '(unknown)')
{'name': '(unknown)', 'age': '(unknown)'}
```

注意：该方法创建的字典的所有键都关联到**同一个**值对象，当值是可变类型时会导致意外的结果。例如：

```python
>>> d = dict.fromkeys(['foo', 'bar', 'baz'], [])
>>> d
{'foo': [], 'bar': [], 'baz': []}
>>> d['foo'].append(42)
>>> d
{'foo': [42], 'bar': [42], 'baz': [42]}
```

为避免这一问题，应使用字典推导式（详见5.6节）。

```python
>>> d = {k: [] for k in ['foo', 'bar', 'baz']}
>>> d
{'foo': [], 'bar': [], 'baz': []}
>>> d['foo'].append(42)
>>> d
{'foo': [42], 'bar': [], 'baz': []}
```

#### get
通常，如果试图访问字典中没有的项，将引发`KeyError`。

```python
>>> d = {}
>>> d['name']
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'name'
```

使用`get()`方法访问不存在的键时，不会引发异常，而是返回`None`或者指定的“默认”值。

```python
>>> print(d.get('name'))
None
>>> d.get('name', 'N/A')
'N/A'
```

如果键存在，则`get()`与普通的字典查询一样。

```python
>>> d['name'] = 'Eric'
>>> d.get('name')
'Eric'
```

代码清单4-2是代码清单4-1所示程序的修改版本，使用`get()`方法访问“数据库”。

[代码清单4-2 字典方法示例](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch04/dictionary_method_example.py)

#### items
`items()`方法返回一个包含所有字典项的列表，其中每一项的形式为`(key, value)`。字典项没有特定的顺序。

```python
>>> d = {'title': 'Python Web Site', 'url': 'http://www.python.org', 'spam': 0}
>>> d.items()
dict_items([('title', 'Python Web Site'), ('url', 'http://www.python.org'), ('spam', 0)])
```

返回值是一种叫做**字典视图**(dictionary view)的特殊类型。字典视图可用于迭代（详见第5章），还可以确定长度和成员资格检查。

```python
>>> it = d.items()
>>> len(it)
3
>>> ('spam', 0) in it
True
```

视图的一个优点是**没有拷贝**，它们始终反映底层字典。

```python
>>> d['spam'] = 1
>>> ('spam', 0) in it
False
>>> d['spam'] = 0
>>> ('spam', 0) in it
True
```

如果需要将字典项拷贝到列表，可以自己实现。

```python
>>> list(d.items())
[('title', 'Python Web Site'), ('url', 'http://www.python.org'), ('spam', 0)]
```

#### keys
`key()`方法返回字典的键的视图。

#### pop
`pop()`方法用于获取给定的键对应的值，并将该键值对删除。如果键不存在则返回默认值，如果未指定默认值则引发`KeyError`。

```python
>>> d = {'x': 1, 'y': 2}
>>> d.pop('x')
1
>>> d
{'y': 2}
>>> d.pop('z')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'z'
>>> d.pop('z', 3)
3
```

#### popitem
`popitem()`删除并返回一个字典项，如果字典为空则引发`KeyError`。

注：在早期版本中，该方法弹出随机的项。从3.7版本开始，该方法按后进先出(LIFO)的顺序返回项。

```python
>>> d = {'url': 'http://www.python.org', 'spam': 0, 'title': 'Python Web Site'}
>>> d.popitem()
('title', 'Python Web Site')
>>> d
{'url': 'http://www.python.org', 'spam': 0}
```

注：如果希望`popitem()`遵循可预测的顺序，使用`collections`模块的`OrderedDict`类。

#### setdefault
`setdefault()`方法返回与给定的键关联的值。如果键不存在，则将键对应的值设置为给定值（默认为`None`），并返回这个值。

```python
>>> d = {}
>>> d.setdefault('name', 'N/A')
'N/A'
>>> d
{'name': 'N/A'}
>>> d['name'] = 'Gumby'
>>> d.setdefault('name', 'N/A')
'Gumby'
>>> d
{'name': 'Gumby'}
>>> print(d.setdefault('age'))
None
>>> d
{'name': 'Gumby', 'age': None}
```

注：该方法大致等价于

```python
def setdefault(self, key, default=None):
    if key not in self:
        self[key] = default
    return self[key]
```

#### update
`update()`方法使用另一个字典的项来更新该字典。参数提供的字典中的项将被添加到该字典中，覆盖具有相同键的项。

```python
>>> d = {
...     'title': 'Python Web Site',
...     'url': 'http://www.python.org',
...     'changed': 'Mar 14 22:09:15 MET 2016'
... }
>>> x = {'title': 'Python Language Website'}
>>> d.update(x)
>>> d
{'title': 'Python Language Website', 'url': 'http://www.python.org', 'changed': 'Mar 14 22:09:15 MET 2016'}
```

`update()`方法与`dict()`函数一样，参数可以是一个映射、一个键值对序列或关键字参数。

注：Python 3.9引入了一个新的字典运算符`|=`，`d |= other`等价于`d.update(other)`。

#### values
`values()`方法返回字典的值的视图。与`keys()`不同，`values()`返回的视图可能包含重复值。

```python
>>> d = {1: 1, 2: 2, 3: 3, 4: 1}
>>> d.values()
dict_values([1, 2, 3, 1])
```
