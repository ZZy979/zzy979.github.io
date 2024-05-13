---
title: 《Python基础教程》笔记 第18章 程序打包
date: 2024-05-12 17:53:19 +0800
categories: [Python, Beginning Python]
tags: [python, setuptools, py2exe]
---
程序准备发布时，你可能想先将其打包。Setuptools和较旧的Distutils都是用于发布Python包的工具，让你能够用Python轻松地编写安装脚本。这些脚本可用于构建可发布的归档文件，供用户来编译和安装你编写的库。

## 18.1 Setuptools基础
Python打包用户指南(<https://packaging.python.org/>)和Setuptools官网(<https://setuptools.pypa.io/>)有很多相关的文档。使用Setuptools可以完成各种有用的任务，只需编写像代码清单18-1这样简单的脚本即可。（如果还没有Setuptools，可以使用pip安装）

[代码清单18-1 简单的Setuptools安装脚本](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch18/hello/setup.py)

将上面的脚本存储为setup.py（这是Setuptools脚本的通用约定），并确保在同一个目录下有一个名为hello.py的模块。

警告：运行安装脚本时，将在当前目录中创建新的文件和子目录，因此最好在一个新的目录中进行试验，以免覆盖既有的文件。

下面看看如何使用这个简单的脚本。像这样执行它：

```shell
$ python setup.py
usage: setup.py [global_opts] cmd1 [cmd1_opts] [cmd2 [cmd2_opts] ...]
   or: setup.py --help [cmd1 cmd2 ...]
   or: setup.py --help-commands
   or: setup.py cmd --help

error: no commands supplied
```

可以看到，要获得更多信息，可以使用开关`--help`或`--help-commands`。尝试执行`build`命令：

```shell
$ python setup.py build
running build
running build_py
creating build
creating build\lib
copying hello.py -> build\lib
```

Setuptools创建了一个名为build的目录，其中包含子目录lib，同时将hello.py拷贝到build/lib目录中。build目录相当于工作区，Setuptools在其中组装包（以及编译扩展等）。安装时只需运行`install`命令，如果需要，会自动运行`build`命令。

注意：在这个示例中，`install`命令会将hello.py复制到PYTHONPATH包含的某个系统特定的目录中（例如Python安装目录\Lib\site-packages）。可以使用`-n`开关：`python setup.py -n install`，这样将只进行演示而并不实际安装(dry run)。目前没有标准的`uninstall`命令，因此需要手动卸载模块。

说到这里，来尝试安装这个模块：

```shell
$ python setup.py install
...
Installed d:\python\python311\lib\site-packages\hello-1.0-py3.11.egg
Processing dependencies for Hello==1.0
Finished processing dependencies for Hello==1.0
```

之后就可以直接使用这个模块了：

```python
>>> import hello
Hello, world!
>>> hello
<module 'hello' from 'D:\\Python\\Python311\\Lib\\site-packages\\hello-1.0-py3.11.egg\\hello.py'>
```

这就是安装Python模块、包和扩展的标准机制。你只需提供一个小小的安装脚本。如你所见，在安装过程中，Setuptools创建了一个.egg文件，这是一个独立的Python包。

示例脚本只使用了Setuptools指令`py_modules`。如果要安装整个包，可以使用指令`packages`。你还可以设置很多其他的选项（18.3节将介绍其中一些，完整列表见[官方文档](https://setuptools.pypa.io/en/latest/references/keywords.html)）。

## 18.2 打包
编写完能够让用户安装模块的setup.py脚本后，就可以自己使用它来构建归档文件。你还可以创建Windows安装程序、RPM包、egg文件、wheel文件等（wheel将最终取代egg）。这里只介绍如何创建.tar.gz文件。

要创建源代码归档文件，可以使用命令`sdist`（表示 "source distribution" ）。

```shell
$ python setup.py sdist
...
creating Hello-1.0
creating Hello-1.0\Hello.egg-info
copying files to Hello-1.0...
copying hello.py -> Hello-1.0
copying setup.py -> Hello-1.0
copying Hello.egg-info\PKG-INFO -> Hello-1.0\Hello.egg-info
copying Hello.egg-info\SOURCES.txt -> Hello-1.0\Hello.egg-info
copying Hello.egg-info\dependency_links.txt -> Hello-1.0\Hello.egg-info
copying Hello.egg-info\top_level.txt -> Hello-1.0\Hello.egg-info
Writing Hello-1.0\setup.cfg
Creating tar archive
removing 'Hello-1.0' (and everything under it)
```

现在，除了build目录，应该还有一个dist目录，其中有一个名为Hello-1.0.tar.gz的压缩文件。你可以将其分发给其他人，对方可以解压缩并使用包含的setup.py脚本进行安装。可以使用`--formats`选项指定分发格式，使用`--help-formats`获得可用的格式列表。

## 18.3 编译扩展
第17章介绍了如何编写Python扩展。这些扩展编译起来有点麻烦。所幸，还可以使用Setuptools。假设源代码文件palindrome2.c（代码清单17-6）位于当前（空）目录，可以使用下面的setup.py脚本来编译（和安装）它：

[palindrome安装脚本](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch18/palindrome2/setup.py)

如果使用这个脚本运行`install`命令，将自动编译并安装扩展模块`palindrome`。这里没有指定一个模块名列表，而是将`ext_modules`参数指定为一个`Extension`实例列表。构造函数接受一个名称和一个相关文件列表（例如，可以指定头文件）。

如果只想就地编译扩展（得到.so或.pyd文件），可以使用以下命令：

```shell
python setup.py build_ext --inplace
```

注：这样可以避免手动编写那些复杂的编译命令。

现在来看最有趣的地方。如果你安装了SWIG，可以让Setuptools直接使用它！这非常简单，只需将源文件palindrome.c（代码清单17-3）和接口文件（代码清单17-5）添加到`Extension`实例的文件列表中即可。

[使用SWIG的palindrome安装脚本](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch18/palindrome/setup.py)

如果用刚才的命令(`build_ext`)运行这个脚本，也将得到一个.so或.pyd文件，但这次无需自己编写包装代码。注意，将这个扩展命名为`_palindrome`，因为SWIG将创建一个名为palindrome.py的包装器，通过这个名称导入C库。

## 18.4 使用py2exe创建可执行程序
py2exe是Setuptools的扩展（可通过pip安装），让你能够创建可执行的Windows程序（.exe文件）。这在你不想给用户增加单独安装Python解释器的负担时很有用。py2exe可用来创建带GUI的可执行文件。下面使用一个非常简单的示例：

```python
print('Hello, world!')
input('Press <enter>')
```

同样，创建一个只包含这个文件hello.py的空目录，然后创建setup.py：

[py2exe安装脚本](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch18/hello2/setup.py)

可以像这样运行这个脚本：

```shell
python setup.py py2exe
```

这将在子目录dist中创建一个控制台应用程序(hello.exe)。可以从命令行运行，或者双击运行。

有关py2exe的工作原理和高级用法，参见py2exe网站(<https://www.py2exe.org/>)。如果你使用的是macOS，你可能想了解一下py2app (<https://py2app.readthedocs.io/>)，它为macOS提供了类似的功能。

### 在PyPI注册包
要让别人能够使用pip安装你开发的包，必须在Python Package Index ([PyPI](https://pypi.org/))上注册。使用以下命令：

```shell
python setup.py register
```

这将打开一个菜单，让你能够登录或注册。注册包后，就可以使用`upload`命令将其上传到PyPI。例如，下面的命令上传一个源代码分发包：

```shell
python setup.py sdist upload
```
