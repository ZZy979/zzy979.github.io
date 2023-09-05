---
title: Python导入模块报错问题的分析
date: 2020-11-05 20:22:04 +0800
categories: [Python]
tags: [python, relative import, absolute import]
---
在Python代码中导入自定义模块时经常遇到报错问题，并且在PyCharm和命令行中会有不同的表现。本文通过实例分析两种常见的导入错误出现的原因及解决方法。

Python版本：3.8

## 1.相对导入报错
假设有如下的项目目录结构：

```
import-error-demo/
  config.py
  foo/
    __init__.py
    bar.py
    baz.py
    qux.py
```

其中import-error-demo是项目根目录，config.py的内容为

```python
A = 1
B = 2
```

### 情况1：导入父级模块
在bar.py中通过相对导入使用config.py中的变量`A`，代码如下：

```python
from ..config import A


def f():
    return 2 * A


if __name__ == '__main__':
    print(f())
```

（1）使用PyCharm执行bar.py：

```
Traceback (most recent call last):
  File "D:/PyCharm/projects/import-error-demo/foo/bar.py", line 1, in <module>
    from ..config import A
ImportError: attempted relative import with no known parent package
```

（2）在项目根目录下执行 `python foo\bar.py`：

```
Traceback (most recent call last):
  File "foo\bar.py", line 1, in <module>
    from ..config import A
ImportError: attempted relative import with no known parent package
```

（3）在项目根目录下执行 `python -m foo.bar`：

```
Traceback (most recent call last):
  File "D:\Python\Python38\lib\runpy.py", line 194, in _run_module_as_main
    return _run_code(code, main_globals, None,
  File "D:\Python\Python38\lib\runpy.py", line 87, in _run_code
    exec(code, run_globals)
  File "D:\PyCharm\projects\import-error-demo\foo\bar.py", line 1, in <module>
    from ..config import A
ValueError: attempted relative import beyond top-level package
```

### 情况2：导入同级模块
假设baz.py定义了一个函数`f`：

```python
def f():
    return 8
```

在qux.py中通过相对导入使用该函数，代码如下：

```python
from .baz import f


def g():
    return f() + 1


if __name__ == '__main__':
    print(g())
```

（1）使用PyCharm执行qux.py：

```
Traceback (most recent call last):
  File "D:/PyCharm/projects/import-error-demo/foo/qux.py", line 1, in <module>
    from .baz import f
ImportError: attempted relative import with no known parent package
```

（2）在项目根目录下执行 `python foo\qux.py`：

```
Traceback (most recent call last):
  File "foo\qux.py", line 1, in <module>
    from .baz import f
ImportError: attempted relative import with no known parent package
```

（3）在项目根目录下执行 `python -m foo.qux`：正常输出结果`9`

（4）在foo目录下执行 `python -m qux`：

```
Traceback (most recent call last):
  File "D:\Python\Python38\lib\runpy.py", line 194, in _run_module_as_main
    return _run_code(code, main_globals, None,
  File "D:\Python\Python38\lib\runpy.py", line 87, in _run_code
    exec(code, run_globals)
  File "D:\PyCharm\projects\import-error-demo\foo\qux.py", line 1, in <module>
    from .baz import f
ImportError: attempted relative import with no known parent package
```

### 原因分析
情况1的（1）（2）和情况2的（1）（2）（4）报错信息都是无法找到父级包。

关于该问题，[PEP 328](https://www.python.org/dev/peps/pep-0328/)的Relative Imports and __name__一节有相关介绍：

> Relative imports use a module's `__name__` attribute to determine that module's position in the package hierarchy. If the module's name does not contain any package information (e.g. it is set to `'__main__'`) then relative imports are resolved as if the module were a top level module, regardless of where the module is actually located on the file system.

“相对导入使用模块的`__name__`属性来确定该模块在包层次结构中的位置。如果模块的名称不包含任何包信息（例如`'__main__'`），则相对导入将被视为该模块是顶级模块来进行解析，而不管模块实际位于文件系统上的什么位置。”

通过断点查看每种情况下启动模块的`__name__`和`__package__`属性：

情况1：

| 运行方式 | `__name__` | `__package__` |
| ----- | ----- | ----- |
| （1） | `'__main__'` | `None` |
| （2） | `'__main__'` | `None` |
| （3） | `'__main__'` | `'foo'` |

情况2：

| 运行方式 | `__name__` | `__package__` |
| ----- | ----- | ----- |
| （1） | `'__main__'` | `None` |
| （2） | `'__main__'` | `None` |
| （3） | `'__main__'` | `'foo'` |
| （4） | `'__main__'` | `''` |

从这些结果中可以解释以上现象出现的原因：
* 情况1的（1）（2）和情况2的（1）（2）：`__package__`为`None`，即相对导入不知道该模块所在的包，因此无法找到父级包
* 情况1的（3）：`__name__`为`'__main__'`，根据PEP 328的说明，相对导入认为模块`bar`是顶级模块，因此导入父级模块`config`时报错“顶级包之外的相对导入”而不是“找不到父级包”
* 情况2的（3）：`__package__`为`'foo'`，即相对导入知道模块`qux`所在的包是`foo`，导入同级模块`baz`就是导入`foo.baz`，因此能正常运行
* 情况2的（4）：`__package__`为空字符串，即相对导入认为模块`qux`是顶级模块而不知道所在的包，因此导入同级模块时报错“找不到父级包”

因此，**启动模块不要使用相对导入**

尝试在项目根目录下创建run.py并将函数`bar.f()`和`qux.g()`的调用都移至run.py：

```python
if __name__ == '__main__':
    from foo import qux
    print('qux.g() ->', qux.g())
    from foo import bar
    print('bar.f() ->', bar.f())
```

此时无论是使用PyCharm执行run.py、执行`python run.py`还是执行`python -m run`，`qux.g()`的调用都会成功，而模块`bar`的相对导入都会报错“顶级包之外的相对导入”：

```
qux.g() -> 9
Traceback (most recent call last):
  File "run.py", line 4, in <module>
    from foo import bar
  File "D:\PyCharm\projects\import-error-demo\foo\bar.py", line 1, in <module>
    from ..config import A
ValueError: attempted relative import beyond top-level package
```

分别查看三个模块的`__name__`和`__package__`属性：

| 模块 | `__name__` | `__package__` |
| ----- | ----- | ----- |
| `run` | `'__main__'` | `None` |
| `qux` | `'foo.qux'` | `'foo'` |
| `bar` | `'foo.bar'` | `'foo'` |

这是因为在bar.py中执行`from ..config import A`时，首先会查找该模块所在包`foo`的父级包，但`foo`位于项目根目录，而项目根目录并不是一个Python包，因此就会报上面的错

### 解决方法
在项目根目录下创建一个pkg目录作为顶级包：

```
import-error-demo/
  run1.py
  pkg/
    __init__.py
    config.py
    run2.py
    foo/
      __init__.py
      bar.py
      baz.py
      qux.py
```

run.py复制为两个，分别位于项目根目录和pkg目录，内容均为

```python
from pkg.foo import bar, qux

if __name__ == '__main__':
    print('qux.g() ->', qux.g())
    print('bar.f() ->', bar.f())
```

#### run1.py：正常
使用PyCharm执行run1.py、在项目根目录下执行`python run1.py`或`python -m run1`都能得到正确结果：

```
qux.g() -> 9
bar.f() -> 2
```

#### run2.py：找不到模块

（1）使用PyCharm执行run2.py和在项目根目录下执行`python -m pkg.run2`都能得到正确结果：

```
qux.g() -> 9
bar.f() -> 2
```

（2）在项目根目录下执行`python pkg\run2.py`则会报错找不到模块：

```
Traceback (most recent call last):
  File "pkg\run2.py", line 1, in <module>
    from pkg.foo import qux, bar
ModuleNotFoundError: No module named 'pkg'
```

这里的原因在下一节中解释。

## 2.找不到模块
**Python在 [sys.path](https://docs.python.org/3/library/sys.html#sys.path) 指定的路径列表中搜索模块**

该列表从[PYTHONPATH](https://docs.python.org/3/using/cmdline.html#envvar-PYTHONPATH)环境变量初始化，并自动添加了Python标准库和第三方库目录。

因此所有找不到模块的问题根本原因都是模块所在路径不在`sys.path`列表中。

不同情况下，`sys.path`会被自动添加不同的目录：

（1）PyCharm会自动将运行配置中的**工作目录**（就是启动脚本所在的目录）和**项目根目录**这两项添加到`sys.path`开头
![PyCharm运行配置](/assets/images/python-import-error-analysis/PyCharm运行配置.png)

（2）在命令行中执行脚本文件（即`python xxx.py`）时，**脚本文件所在目录**会被自动添加到`sys.path`开头（见 [Python命令行参数](https://docs.python.org/3/using/cmdline.html#interface-options) 的`<script>`参数）

（3）在命令行中执行模块（即`python -m xxx`）时，**命令行的当前目录**会被自动添加到`sys.path`开头（见 [Python命令行参数](https://docs.python.org/3/using/cmdline.html#interface-options) 的`-m`选项）

打印出各种情况下的`sys.path`：

#### run2.py
（1）使用PyCharm运行run2.py：

```
# 工作目录（启动脚本所在目录）
D:\PyCharm\projects\import-error-demo\pkg
# 项目根目录
D:\PyCharm\projects\import-error-demo
# PyCharm插件
D:\PyCharm\plugins\python\helpers\pycharm_display
# =====Python=====
D:\Python\Python38\python38.zip
D:\Python\Python38\DLLs
D:\Python\Python38\lib  # 标准库
D:\Python\Python38
D:\Python\Python38\lib\site-packages  # 第三方库
# PyCharm插件
D:\PyCharm\plugins\python\helpers\pycharm_matplotlib_backend
```

（2）在项目根目录下执行`python pkg\run2.py`：

```
# 脚本文件所在目录
D:\PyCharm\projects\import-error-demo\pkg
# =====Python=====
D:\Python\Python38\python38.zip
D:\Python\Python38\DLLs
D:\Python\Python38\lib
D:\Python\Python38
D:\Python\Python38\lib\site-packages
```

（3）在项目根目录下执行`python -m pkg.run2`（省略了Python本身的相关路径，下同）：

```
# 当前目录
D:\PyCharm\projects\import-error-demo
...
```

#### run1.py
（1）使用PyCharm运行run1.py：

```
# 工作目录
D:\PyCharm\projects\import-error-demo
# 项目根目录
D:\PyCharm\projects\import-error-demo
...
```

（2）在项目根目录下执行`python run1.py`：

```
# 脚本文件所在目录
D:\PyCharm\projects\import-error-demo
...
```

（3）在项目根目录下执行`python -m run1`：

```
# 当前目录
D:\PyCharm\projects\import-error-demo
...
```

从这些结果中不难分析出原因：顶级包`pkg`所在目录是D:\PyCharm\projects\import-error-demo，因此只要这个目录在`sys.path`中就能找到模块`pkg`，否则就会报错
* run1.py的三种情况和run2.py的（1）（3）都包含了该目录，因此能够找到模块`pkg`
* run2.py的（2）不包含该目录，因此报错找不到模块`pkg`

## 3.绝对导入
PyCharm的导入自动补全会使用绝对导入而不是相对导入
例如在qux.py中导入`baz.f`时，PyCharm的自动补全提示如下：

![PyCharm导入自动补全](/assets/images/python-import-error-analysis/PyCharm导入自动补全.png)

插入的导入语句为`from pkg.foo.baz import f`；同理，在bar.py中导入`config.A`时自动插入的导入语句为`from pkg.config import A`

将bar.py和qux.py中的相对导入都改为绝对导入，只考虑项目根目录下的run1.py（重命名为run.py）

使用PyCharm执行run.py、在项目根目录下执行`python run.py`或`python -m run`都能得到正确结果：

```
qux.g() -> 9
bar.f() -> 2
```

原因和上面分析的一样，顶级包`pkg`所在目录在`sys.path`中，因此Python能够找到模块`pkg`

## 4.src目录
如果项目根目录下有一个src目录，所有代码都放在该目录下：

```
import-error-demo/
  data/
  src/
    run.py
    pkg/
      __init__.py
      config.py
      foo/
        __init__.py
        bar.py
        baz.py
        qux.py
```

此时导入自定义模块时可能会报错找不到模块，需要将src目录标记为源代码根目录，PyCharm以源代码根目录为解析导入的起点（见 [PyCharm项目结构](https://www.jetbrains.com/help/pycharm/configuring-project-structure.html) ）

### 解决方法
（1）在src目录上点击右键→Mark Directory as→Sources Root

![标记src目录为源代码根目录](/assets/images/python-import-error-analysis/标记src目录为源代码根目录.png)

（2）打开PyCharm设置→Build, Execution, Deployment→Console→Python Console，勾选"Add source roots to PYTHONPATH"

![将源代码根目录添加到PYTHONPATH](/assets/images/python-import-error-analysis/将源代码根目录添加到PYTHONPATH.png)

## 5.总结
* Python的模块搜索路径列表是`sys.path`，报错找不到模块的原因一定是模块所在路径不在该列表中
* 比较好的项目目录结构：将所有模块放在一个顶级包下（可确保相对导入不会出错）；启动模块（如run.py）放在与顶级包同级目录下，不要使用相对导入
* 如果顶级包不是在项目根目录下，则要将顶级包所在目录（如src）设置为源代码根目录

## 参考博客
* <https://blog.csdn.net/nigelyq/article/details/78930330>
* <https://blog.csdn.net/ZeropointS/article/details/88353300>
* <https://blog.csdn.net/qq_30622831/article/details/80978118>
* <https://blog.csdn.net/weixin_35684521/article/details/81953199>
* Python项目中的模块如何正确相互调用可以参考这个Demo项目：<https://github.com/pfllo/demo-python-project>

## 参考文档
* [PEP 328 - Imports: Multi-Line and Absolute/Relative](https://www.python.org/dev/peps/pep-0328/)
* [PEP 366 - Main module explicit relative imports](https://www.python.org/dev/peps/pep-0366/)
* [Python命令行参数](https://docs.python.org/3/using/cmdline.html#interface-options)
* [模块属性__package__](https://docs.python.org/3/reference/import.html#__package__)
* [Package Relative Imports](https://docs.python.org/3/reference/import.html#package-relative-imports)
* [sys.path](https://docs.python.org/3/library/sys.html#sys.path)
* [PYTHONPATH环境变量](https://docs.python.org/3/using/cmdline.html#envvar-PYTHONPATH)
* [PyCharm项目结构](https://www.jetbrains.com/help/pycharm/configuring-project-structure.html)
