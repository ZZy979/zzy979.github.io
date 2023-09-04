---
title: Python高效字符串拼接
date: 2019-07-07 22:46:50 +0800
categories: [Python]
tags: [python]
---
参考：[https://waymoot.org/home/python_string/](https://waymoot.org/home/python_string/)

Python中没有类似于C++的`ostringstream`或Java中的`StringBuilder`类的对象，若要在循环中向字符串末尾添加字符串只能用`+=`运算符：

```python
result = ''
for i in range(n):
    result += str(i)
```

但Python中的字符串是不可变类，每次拼接都会构造一个新的字符串，当字符串很长时性能就会很差。

高性能拼接方法：

1.列表+join方法：

```python
lst = []
for i in range(n):
    lst.append(str(i))
result = ''.join(lst)
```

2.列表推导式：

```python
result = ''.join([str(i) for i in range(n)])
```

3.生成器表达式：

```python
result = ''.join(str(i) for i in range(n))
```

性能对比：

```
当n=1000000时
直接拼接：3.25 s
列表+join方法：0.63 s
列表推导式：0.33 s
生成器表达式：0.37 s
```

测试代码：

```python
import time

n = 1000000
t = time.time()
result = ''
for i in range(n):
    result += str(i)
print(time.time() - t)

t = time.time()
lst = []
for i in range(n):
    lst.append(str(i))
result = ''.join(lst)
print(time.time() - t)

t = time.time()
result = ''.join([str(i) for i in range(n)])
print(time.time() - t)

t = time.time()
result = ''.join(str(i) for i in range(n))
print(time.time() - t)
```
