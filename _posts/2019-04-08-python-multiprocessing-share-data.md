---
title: Python多进程共享数据的问题
date: 2019-04-08 00:13:33 +0800
categories: [Python]
tags: [python, multiprocessing]
---
Python的多进程共享数据在Windows和Linux下会产生不同的结果，分别测试直接访问全局变量、传递参数和使用Manager三种情况。

Python版本：3.7.0

## 1.直接访问全局变量

```python
import random
from multiprocessing.pool import Pool


class A:
    def __init__(self, n):
        self.data = list(range(n))

    def get_data(self, index):
        return self.data[index]


data = []
print('global: id(data) = {}'.format(hex(id(data))))
a = A(0)
print('global: id(a) = {}'.format(hex(id(a))))


def f(i):
    print('in f({}): id(data) = {}'.format(i, hex(id(data))))
    return data[i]


def g(x):
    data.append(x)
    print('in g({}): id(data) = {}, data = {}'.format(x, hex(id(data)), data))


def h(i):
    print('in h({}): id(a) = {}'.format(i, hex(id(a))))
    return a.get_data(i)


if __name__ == '__main__':
    data = list(range(10))
    print('main: id(data) = {}, data = {}'.format(hex(id(data)), data))

    print('***多进程从列表读数据***')
    args = [random.randrange(10) for i in range(5)]
    try:
        with Pool() as pool:
            results = pool.map(f, args)
        print(results)
    except IndexError as e:
        print('***IndexError: {}***'.format(e))

    print('***多进程向列表写数据***')
    print('main, before: id(data) = {}, data = {}'.format(hex(id(data)), data))
    args = [random.randrange(100, 200) for i in range(5)]
    with Pool() as pool:
        pool.map(g, args)
    print('main, after: id(data) = {}, data = {}'.format(hex(id(data)), data))

    a = A(10)
    print('main: id(a) = {}'.format(hex(id(a))))

    print('***多进程访问类对象***')
    args = [random.randrange(10) for i in range(5)]
    try:
        with Pool() as pool:
            results = pool.map(h, args)
        print(results)
    except IndexError as e:
        print('***IndexError: {}***'.format(e))
```

Windows下运行结果：

```
global: id(data) = 0x20178826408
global: id(a) = 0x201784c2f28
main: id(data) = 0x2017876b108, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
***多进程从列表读数据***
global: id(data) = 0x18159d938c8
global: id(a) = 0x18159d92828
global: id(data) = 0x2add9e44748
global: id(a) = 0x2add9e42828
global: id(data) = 0x1a6865847c8
global: id(a) = 0x1a686582860
in f(3): id(data) = 0x18159d938c8
in f(6): id(data) = 0x18159d938c8
in f(6): id(data) = 0x18159d938c8
in f(9): id(data) = 0x18159d938c8
in f(4): id(data) = 0x18159d938c8
global: id(data) = 0x1be59732908
global: id(a) = 0x1be59733748
global: id(data) = 0x1c05a9147c8
global: id(a) = 0x1c05a912828
global: id(data) = 0x1f2288146c8
global: id(a) = 0x1f228812860
global: id(data) = 0x1fb75394788
global: id(a) = 0x1fb753927f0
global: id(data) = 0x1ea386e4808
global: id(a) = 0x1ea386e27b8
***IndexError: list index out of range***
***多进程向列表写数据***
main, before: id(data) = 0x2017876b108, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
global: id(data) = 0x23af89d4888
global: id(a) = 0x23af89d27f0
global: id(data) = 0x1d8c8ca4788
global: id(a) = 0x1d8c8ca2828
global: id(data) = 0x2295d9a4748
global: id(a) = 0x2295d9a2828
in g(196): id(data) = 0x23af89d4888, data = [196]
in g(104): id(data) = 0x23af89d4888, data = [196, 104]
in g(110): id(data) = 0x23af89d4888, data = [196, 104, 110]
in g(154): id(data) = 0x23af89d4888, data = [196, 104, 110, 154]
in g(158): id(data) = 0x23af89d4888, data = [196, 104, 110, 154, 158]
global: id(data) = 0x1d3fb8c3908
global: id(a) = 0x1d3fb8c27b8
global: id(data) = 0x126e17038c8
global: id(a) = 0x126e17027f0
main, after: id(data) = 0x2017876b108, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
main: id(a) = 0x20178920518
***多进程访问类对象***
global: id(data) = 0x1fc92b03948
global: id(a) = 0x1fc92b027f0
global: id(data) = 0x1e549384848
global: id(a) = 0x1e5493827b8
in h(8): id(a) = 0x1fc92b027f0
in h(4): id(a) = 0x1fc92b027f0
in h(7): id(a) = 0x1fc92b027f0
in h(4): id(a) = 0x1fc92b027f0
in h(1): id(a) = 0x1fc92b027f0
global: id(data) = 0x11bdcf44808
global: id(a) = 0x11bdcf427b8
global: id(data) = 0x203b2b34808
global: id(a) = 0x203b2b32828
global: id(data) = 0x160090d4808
global: id(a) = 0x160090d2828
global: id(data) = 0x24defac4788
global: id(a) = 0x24defac2828
global: id(data) = 0x1c5e9254848
global: id(a) = 0x1c5e9252780
***IndexError: list index out of range***
```

Linux下运行结果：

```
global: id(data) = 0x7f040823f148
global: id(a) = 0x7f040a0b7240
main: id(data) = 0x7f0408b1fb48, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
***多进程从列表读数据***
in f(5): id(data) = 0x7f0408b1fb48
in f(4): id(data) = 0x7f0408b1fb48
in f(5): id(data) = 0x7f0408b1fb48
in f(6): id(data) = 0x7f0408b1fb48
in f(5): id(data) = 0x7f0408b1fb48
[5, 4, 5, 6, 5]
***多进程向列表写数据***
main, before: id(data) = 0x7f0408b1fb48, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
in g(117): id(data) = 0x7f0408b1fb48, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 117]
in g(149): id(data) = 0x7f0408b1fb48, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 149]
in g(191): id(data) = 0x7f0408b1fb48, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 191]
in g(130): id(data) = 0x7f0408b1fb48, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 130]
in g(176): id(data) = 0x7f0408b1fb48, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 176]
main, after: id(data) = 0x7f0408b1fb48, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
main: id(a) = 0x7f040a0b71d0
***多进程访问类对象***
in h(3): id(a) = 0x7f040a0b71d0
in h(4): id(a) = 0x7f040a0b71d0
in h(9): id(a) = 0x7f040a0b71d0
in h(7): id(a) = 0x7f040a0b71d0
in h(5): id(a) = 0x7f040a0b71d0
[3, 4, 9, 7, 5]
```

可以看到：Windows下子进程无法正确读写主进程的全局变量，且全局变量定义部分被多次执行；Linux下子进程可以读主进程的全局变量且访问的是同一对象，但无法修改。

## 2.传递参数

```python
import random
from multiprocessing.pool import Pool


class A:
    def __init__(self, n):
        self.data = list(range(n))

    def get_data(self, index):
        return self.data[index]


def f(data, i):
    print('in f(data, {}): id(data) = {}'.format(i, hex(id(data))))
    return data[i]


def g(data, x):
    data.append(x)
    print('in g(data, {}): id(data) = {}, data = {}'.format(x, hex(id(data)), data))


def h(a, i):
    print('in h(a, {}): id(a) = {}'.format(i, hex(id(a))))
    return a.get_data(i)


if __name__ == '__main__':
    data = list(range(10))
    print('main: id(data) = {}, data = {}'.format(hex(id(data)), data))

    print('***多进程从列表读数据***')
    args = [(data, random.randrange(10)) for i in range(5)]
    try:
        with Pool() as pool:
            results = pool.starmap(f, args)
        print(results)
    except IndexError as e:
        print('***IndexError: {}***'.format(e))

    print('***多进程向列表写数据***')
    print('main, before: id(data) = {}, data = {}'.format(hex(id(data)), data))
    args = [(data, random.randrange(100, 200)) for i in range(5)]
    with Pool() as pool:
        pool.starmap(g, args)
    print('main, after: id(data) = {}, data = {}'.format(hex(id(data)), data))

    a = A(10)
    print('main: id(a) = {}'.format(hex(id(a))))

    print('***多进程访问类对象***')
    args = [(a, random.randrange(10)) for i in range(5)]
    try:
        with Pool() as pool:
            results = pool.starmap(h, args)
        print(results)
    except IndexError as e:
        print('***IndexError: {}***'.format(e))
```

Windows下运行结果：

```
main: id(data) = 0x21787d36d48, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
***多进程从列表读数据***
in f(data, 4): id(data) = 0x1be55d22608
in f(data, 5): id(data) = 0x1be55d22608
in f(data, 0): id(data) = 0x1be55d22608
in f(data, 1): id(data) = 0x1be55d22608
in f(data, 2): id(data) = 0x1be55d22608
[4, 5, 0, 1, 2]
***多进程向列表写数据***
main, before: id(data) = 0x21787d36d48, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
in g(data, 118): id(data) = 0x2c8c63916c8, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 118]
in g(data, 119): id(data) = 0x2c8c63916c8, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 119]
in g(data, 107): id(data) = 0x2c8c63916c8, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 107]
in g(data, 124): id(data) = 0x2c8c63916c8, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 124]
in g(data, 115): id(data) = 0x2c8c63916c8, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 115]
main, after: id(data) = 0x21787d36d48, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
main: id(a) = 0x21787e34080
***多进程访问类对象***
in h(a, 3): id(a) = 0x21038dcf550
in h(a, 1): id(a) = 0x21038d79b70
in h(a, 8): id(a) = 0x21038dcf550
in h(a, 8): id(a) = 0x21038d79b70
in h(a, 6): id(a) = 0x21038dcf550
[3, 1, 8, 8, 6]
```

Linux下运行结果：

```
main: id(data) = 0x7f9bff8cab48, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
***多进程从列表读数据***
in f(data, 5): id(data) = 0x7f9bff8c3b08
in f(data, 2): id(data) = 0x7f9bff8c3b08
in f(data, 0): id(data) = 0x7f9bff8c3b08
in f(data, 0): id(data) = 0x7f9bff8c3b08
in f(data, 9): id(data) = 0x7f9bff8c3b08
[5, 2, 0, 0, 9]
***多进程向列表写数据***
main, before: id(data) = 0x7f9bff8cab48, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
in g(data, 190): id(data) = 0x7f9bff8c3b08, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 190]
in g(data, 108): id(data) = 0x7f9bff8c3b08, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 108]
in g(data, 140): id(data) = 0x7f9bff8c3b08, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 140]
in g(data, 129): id(data) = 0x7f9bff8c3b08, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 129]
in g(data, 112): id(data) = 0x7f9bff8c3b08, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 112]
main, after: id(data) = 0x7f9bff8cab48, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
main: id(a) = 0x7f9bfdaab320
***多进程访问类对象***
in h(a, 4): id(a) = 0x7f9bfda9e668
in h(a, 8): id(a) = 0x7f9bfda9e780
in h(a, 7): id(a) = 0x7f9bfda9e898
in h(a, 1): id(a) = 0x7f9bfda9e9b0
in h(a, 1): id(a) = 0x7f9bfda9eac8
[4, 8, 7, 1, 1]
```

可以看到，Windows和Linux下子进程读取通过参数传入的主进程全局变量时，访问的参数与主进程全局变量并非同一对象，因此也无法修改；而通过参数传入自定义类对象时，不同子进程的参数都是不同的对象，即每个子线程都持有一份不同的拷贝，如果该全局变量占用很大内存则多次拷贝可能导致内存超限。

## 3.使用Manager

```python
import random
from multiprocessing import Manager
from multiprocessing.managers import BaseManager
from multiprocessing.pool import Pool


class A:
    def __init__(self, n):
        self.data = list(range(n))

    def get_data(self, index):
        return self.data[index]


class MyManager(BaseManager):
    pass


MyManager.register('A', A)


def f(data, i):
    print('in f(data, {}): id(data) = {}'.format(i, hex(id(data))))
    return data[i]


def g(data, x):
    data.append(x)
    print('in g(data, {}): id(data) = {}, data = {}'.format(x, hex(id(data)), data))


def h(a, i):
    print('in h(a, {}): id(a) = {}'.format(i, hex(id(a))))
    return a.get_data(i)


if __name__ == '__main__':
    with Manager() as manager:
        data = manager.list(range(10))
        print('main: id(data) = {}, data = {}'.format(hex(id(data)), data))

        print('***多进程从列表读数据***')
        args = [(data, random.randrange(10)) for i in range(5)]
        try:
            with Pool() as pool:
                results = pool.starmap(f, args)
            print(results)
        except IndexError as e:
            print('***IndexError: {}***'.format(e))

        print('***多进程向列表写数据***')
        print('main, before: id(data) = {}, data = {}'.format(hex(id(data)), data))
        args = [(data, random.randrange(100, 200)) for i in range(5)]
        with Pool() as pool:
            pool.starmap(g, args)
        print('main, after: id(data) = {}, data = {}'.format(hex(id(data)), data))

    with MyManager() as manager:
        a = manager.A(10)
        print('main: id(a) = {}'.format(hex(id(a))))

        print('***多进程访问类对象***')
        args = [(a, random.randrange(10)) for i in range(5)]
        try:
            with Pool() as pool:
                results = pool.starmap(h, args)
            print(results)
        except IndexError as e:
            print('***IndexError: {}***'.format(e))
```

Windows下运行结果：

```
main: id(data) = 0x213fb6fd518, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
***多进程从列表读数据***
in f(data, 5): id(data) = 0x1c9d74b2cf8
in f(data, 7): id(data) = 0x1c9d74f2cc0
in f(data, 8): id(data) = 0x21ba9277940
in f(data, 3): id(data) = 0x1c9d74f2b70
in f(data, 6): id(data) = 0x21ba92c7ba8
[5, 7, 8, 3, 6]
***多进程向列表写数据***
main, before: id(data) = 0x213fb6fd518, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
in g(data, 129): id(data) = 0x158f6643d30, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 129]
in g(data, 141): id(data) = 0x158f6682cf8, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 129, 141, 179]
in g(data, 179): id(data) = 0x1c86745da58, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 129, 141, 179]
in g(data, 102): id(data) = 0x158f6682ba8, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 129, 141, 179, 102]
in g(data, 192): id(data) = 0x1c867453c50, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 129, 141, 179, 102, 192]
main, after: id(data) = 0x213fb6fd518, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 129, 141, 179, 102, 192]
main: id(a) = 0x213fba53390
***多进程访问类对象***
in h(a, 3): id(a) = 0x1c7cd102d68
in h(a, 9): id(a) = 0x1c7cd102d68
in h(a, 0): id(a) = 0x205a9904cc0
in h(a, 9): id(a) = 0x1c7cd102c88
in h(a, 1): id(a) = 0x205a9904cc0
[3, 9, 0, 9, 1]
```

Linux下运行结果：

```
main: id(data) = 0x7f26e44b26a0, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
***多进程从列表读数据***
in f(data, 3): id(data) = 0x7f26e44d0780
in f(data, 6): id(data) = 0x7f26e44d0978
in f(data, 0): id(data) = 0x7f26e44d0a90
in f(data, 2): id(data) = 0x7f26e44d0860
in f(data, 5): id(data) = 0x7f26e44d0ba8
[3, 2, 6, 0, 5]
***多进程向列表写数据***
main, before: id(data) = 0x7f26e44b26a0, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
in g(data, 167): id(data) = 0x7f26e2c6c160, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 167]
in g(data, 128): id(data) = 0x7f26e2c6b5c0, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 167, 128, 180]
in g(data, 180): id(data) = 0x7f26e2c6b278, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 167, 128, 180, 103]
in g(data, 103): id(data) = 0x7f26e2c6b390, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 167, 128, 180, 103]
in g(data, 166): id(data) = 0x7f26e2c6b6d8, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 167, 128, 180, 103, 166]
main, after: id(data) = 0x7f26e44b26a0, data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 167, 128, 180, 103, 166]
main: id(a) = 0x7f26e44dc080
***多进程访问类对象***
in h(a, 1): id(a) = 0x7f26e44b2c18
in h(a, 3): id(a) = 0x7f26e44b2c18
in h(a, 6): id(a) = 0x7f26e44b2c18
in h(a, 8): id(a) = 0x7f26e44b2c18
in h(a, 2): id(a) = 0x7f26e44b2c18
[1, 3, 2, 6, 8]
```

可以看到，Windows和Linux下子进程均可读取和修改通过参数传入的主进程manager.list变量，但每个子进程访问的不是同一个对象，说明也执行了拷贝操作；子进程也均可以读取通过参数传入的自定义的manager.A变量，但Windows下不同子进程的参数不是同一对象，而Linux下不同子进程的参数是同一对象（但和主进程的manager.A变量不是同一对象）。
