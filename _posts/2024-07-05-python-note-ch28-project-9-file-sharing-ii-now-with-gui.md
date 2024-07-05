---
title: 《Python基础教程》笔记 第28章 项目9：文件共享2——GUI版本
date: 2024-07-05 20:03:43 +0800
categories: [Python, Beginning Python]
tags: [python, gui, tkinter]
---
这是个相对较小的项目，因为需要的大部分功能都已经在第27章写好了。在本章中，你将看到给已有的Python程序添加GUI多么容易。

## 28.1 问题描述
在这个项目中，我们将扩展第27章开发的文件共享系统，添加GUI客户端，让程序更容易使用。这个项目的第二个目标是展示具有足够高的模块化设计的程序非常容易扩展。

这个GUI客户端应该满足以下需求：
* 允许用户输入文件名，并将其提交给服务器的`fetch()`方法。
* 列出服务器的文件目录当前包含的文件。

## 28.2 有用的工具
除了第27章用到的工具外，还需要Python自带的Tkinter工具包。关于Tkinter的详细信息，参见第12章。

## 28.3 准备工作
开始这个项目前，应该准备好第27章的程序，以及一个可用的GUI工具包。

## 28.4 初次实现
客户端提供了一个界面，用户可以通过它来访问服务器的功能（`fetch()`方法）。下面来看看GUI相关的代码。

第27章的客户端是`cmd.Cmd`的子类，而本章的客户端是`tkinter.Frame`的子类。GUI相关的设置工作在名为`create_widgets()`的方法中完成，该方法在构造函数中被调用。它创建一个用于输入文件名的文本框(`Entry`)以及一个用于获取指定文件的按钮(`Button`)，按钮的动作被设置为`fetch_handler()`方法。这个事件处理器非常类似于第27章中的`do_fetch()`，它从`self.input`（文本框）获取查询，并在一条`try/except`语句中调用`self.server.fetch()`。

初次实现的源代码如代码清单28-1所示。

[代码清单28-1 简单的GUI客户端](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch28/simple_guiclient.py)

这个GUI客户端的工作原理与第27章中基于文本的客户端相同，运行方式也相同。

```shell
$ python simple_guiclient.py urls.txt directory http://servername.com:4242
```

下图展示了客户端的GUI。

![简单的GUI客户端](/assets/images/python-note-ch28-project-9-file-sharing-ii-now-with-gui/简单的GUI客户端.png)

这个实现能用，但只实现了部分功能——它还应该列出服务器的文件目录包含的文件。为此，必须对服务器(`Node`)本身进行扩展。

## 28.5 再次实现
第一个原型非常简单。它确实实现了文件共享系统的功能，但对用户不太友好。如果用户能够知道有哪些文件可用将会有很大帮助。再次实现将解决这一问题，完整的源代码如代码清单28-2所示。

要获取节点的文件列表，必须添加一个方法。扩展一个对象非常简单：可以通过继承。只需创建一个`Node`的子类`ListableNode`，并添加一个方法`list()`，它使用`os.listdir()`返回目录中的所有文件的列表。

```python
class ListableNode(Node):
    def list(self):
        return listdir(self.dirname)
```

为了访问这个服务器方法，在客户端中添加`update_list()`方法。

```python
def update_list(self):
    self.files.delete(0, tk.END)
    self.files.insert(tk.END, *self.server.list())
```

属性`self.files`指向一个列表框，这个列表框是在`create_widgets()`方法中添加的。在`create_widgets()`方法创建列表框时，以及每次调用`fetch_handler()`时，都会调用`update_list()`方法。

[代码清单28-2 最终的GUI客户端](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch28/guiclient.py)

现在已经有了一个带GUI的P2P文件共享程序，可以使用下面的命令运行：

```shell
$ python guiclient.py urls.txt directory http://servername.com:4242
```

下图展示了最终的GUI客户端。

![最终的GUI客户端](/assets/images/python-note-ch28-project-9-file-sharing-ii-now-with-gui/最终的GUI客户端.png)

## 28.6 进一步探索
第27章提出了一些扩展这个文件共享系统的想法，这里再列出一些：
* 让用户选择要获取的文件，而不是输入文件名。
* 添加一个状态栏，显示诸如“下载中”或“无法找到文件foo.txt”等消息。
* 想办法让节点能够共享“好友”。例如，将一个节点介绍给另一个时，它们都可以将自己已知的节点介绍给对方。另外，也可以让节点在关闭前将其已知节点都告诉所有的邻居。
* 在GUI中添加一个已知节点(URL)的列表，让用户能够添加新的URL并将其保存到URL文件中。
