---
title: 【Python】模拟对象模块unittest.mock
date: 2021-12-25 14:10:11 +0800
categories: [Python]
tags: [python, unit test, mock]
---
模拟对象(mock object)用于在单元测试中将系统的一部分替换为虚假对象，从而方便验证这些对象如何被使用

标准库提供了`unittest.mock`模块，其核心是`Mock`, `MagicMock`两个类以及`patch()`函数

## Mock类
访问`Mock`对象的任何属性或方法时，它都会记录访问细节（例如方法的调用实参），如果不存在则会创建；另外`Mock`对象是可调用的，即可以当作函数来调用，每次调用时都会记录实参，并返回指定的值，之后可以验证是否按照指定的实参进行调用

### return_value参数：指定返回值
```python
>>> from unittest.mock import Mock
>>> mock = Mock(return_value=8)
>>> mock(3, 4, 5, key='value')
8
>>> mock.assert_called_with(3, 4, 5, key='value')
```

### side_effect参数：指定对Mock对象进行调用时执行的操作
* 如果是函数，则以相同的实参调用该函数，并返回函数的返回值
* 如果是异常（类或实例），则抛出该异常
* 如果是可迭代对象，则每次返回下一个元素，或抛出异常

```python
>>> def f(x):
...     return x + 1
... 
>>> mock = Mock(side_effect=f)
>>> mock(1), mock(2), mock(3)
(2, 3, 4)

>>> mock = Mock(side_effect=KeyError('foo'))
>>> mock()
Traceback (most recent call last):
  ...
KeyError: 'foo'

>>> mock = Mock(side_effect=[1, 2, 3])
>>> mock(), mock(), mock()
(1, 2, 3)
>>> mock()
Traceback (most recent call last):
  ...
StopIteration
```

总结：Mock对象调用的返回结果
* 返回定值：使用return_value参数
* 依次返回一些值：使用可迭代对象作为side_effect参数
* 调用函数：使用函数作为side_effect参数
* 抛出异常：使用异常作为side_effect参数

### 属性访问/方法调用
实际上，`Mock`对象创建的属性也是`Mock`对象，而所谓方法调用就是对创建的`Mock`对象进行调用

```python
>>> mock = Mock()
>>> mock.a
<Mock name='mock.a' id='2523126865008'>
>>> mock.f
<Mock name='mock.f' id='2523126864480'>
>>> mock.f()
<Mock name='mock.f()' id='2523126865248'>
>>> mock.f(1, 2, 3)
<Mock name='mock.f()' id='2523126865248'>
>>> mock.f.return_value = 8
>>> mock.f()
8
>>> mock.f(4, 5, 6)
8
>>> mock.f.call_args_list
[call(), call(1, 2, 3), call(), call(4, 5, 6)]
>>> mock.f.assert_called_with(1, 2, 3)
Traceback (most recent call last):
  ...
AssertionError: expected call not found.
Expected: f(1, 2, 3)
Actual: f(4, 5, 6)
>>> mock.f.assert_any_call(1, 2, 3)
```

### 内置属性

| 属性 | 说明 |
| --- | --- |
| return_value | 指定返回值 |
| side_effect | 指定被调用时执行的动作 |
| called | 是否被调用过 |
| call_count	 | 被调用次数 |
| call_args | 最后一次被调用的实参 |
| call_args_list | 每次被调用的实参 |

```python
>>> from unittest.mock import Mock
>>> mock = Mock(return_value=None)
>>> mock.called
False
>>> mock()
>>> mock(1, 2)
>>> mock(3, 4, a=5, b=6)
>>> mock.called
True
>>> mock.call_count
3
>>> mock.call_args
call(3, 4, a=5, b=6)
>>> mock.call_args.args
(3, 4)
>>> mock.call_args.kwargs
{'a': 5, 'b': 6}
>>> mock.call_args_list
[call(), call(1, 2), call(3, 4, a=5, b=6)]
```

### 断言方法

| 方法 | 断言 |
| --- | --- |
| assert_called() | 至少被调用过一次 |
| assert_not_called() | 没有被调用过 |
| assert_called_once() | 恰好被调用过一次 |
| assert_called_with(*args, **kwargs) | 最后一次调用的实参与给定的匹配 |
| assert_called_once_with(*args, **kwargs) | 恰好被调用过一次，且实参与给定的匹配 |
| assert_any_call(*args, **kwargs) | 任何一次被调用的实参与给定的匹配 |

## MagicMock类
`MagicMock`类是`Mock`类的子类，只是预先将一些魔法方法创建为`Mock`对象

```python
>>> mock = MagicMock()
>>> type(mock.__str__)
<class 'unittest.mock.MagicMock'>
>>> mock.__str__.return_value = 'foobarbaz'
>>> str(mock)
'foobarbaz'
>>> mock.__str__.assert_called()
```

使用普通Mock类的等价做法是
```python
>>> mock = Mock()
>>> mock.__str__ = Mock(return_value='foobarbaz')
```

## patch()函数
`patch()`函数可用作函数装饰器、类装饰器或上下文管理器，用于将指定的类替换为一个`Mock`对象，退出作用域时恢复

其参数是一个字符串，表示一个类的完整名称`'package.module.ClassName'`，替换后创建出的所有该类的对象都将是`Mock`对象（Python中实例化一个对象就是对类进行调用）
* 如果用作函数装饰器，则创建的`Mock`对象将被传递给被装饰函数的第一个参数
* 如果用作类装饰器，则作用于该类的所有以test开头的方法
* 如果用作上下文管理器，则创建的`Mock`对象将作为上下文管理器的返回值

假设模块`foo`中有一个类`C`：

```python
class C:

    def __init__(self, n):
        self.n = n

    def f(self, x):
        return self.n * x
```

一个正常的测试函数如下：

```python
>>> import foo
>>> def test():
...     c = foo.C(2)
...     print(c.f(4))
... 
>>> test()
8
```

使用`patch()`替换`C`类后结果如下：

```python
>>> from unittest.mock import patch
>>> @patch('foo.C')
... def test(mock_C):
...     assert foo.C is mock_C
...     print(mock_C)
...     c = mock_C(2)
...     print(c)
...     print(c.f(4))
... 
>>> test()
<MagicMock name='C' id='2294047189600'>
<MagicMock name='C()' id='2294047153840'>
<MagicMock name='C().f()' id='2294047615824'>
>>> foo.C
<class 'foo.C'>
```

注意：`test()`函数的第一个参数将被自动赋值为创建的`Mock`对象，因此实际调用时不需要传入参数

使用上下文管理器的等价做法如下：

```python
>>> with patch('foo.C') as mock_C:
...     assert foo.C is mock_C
...     print(mock_C)
...     c = mock_C(2)
...     print(c)
...     print(c.f(4))
... 
<MagicMock name='C' id='2294047657072'>
<MagicMock name='C()' id='2294047686768'>
<MagicMock name='C().f()' id='2294047731088'>
```

内部实现原理大致如下

进入`patch()`的作用域时：

```python
## target == 'foo.C'
target, attribute = target.rsplit('.', 1)
target = __import__(target)
original = getattr(target, attribute)
setattr(target, attribute, Mock())
```

离开`patch()`的作用域时：

```python
setattr(target, attribute, original)
```

了解实现原理后就会发现，`patch()`所做的事只不过是先导入指定的模块，再将该模块指定的**属性**设置为一个`Mock`对象，而模块的属性可以是该模块包含的类、函数或变量，因此`patch()`不仅可以用于模块中的类，可也以用于模块中的函数和变量
