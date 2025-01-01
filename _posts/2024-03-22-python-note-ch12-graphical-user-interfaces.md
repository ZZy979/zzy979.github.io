---
title: 《Python基础教程》笔记 第12章 图形用户界面
date: 2024-03-22 16:52:41 +0800
categories: [Python, Beginning Python]
tags: [python, gui, tkinter, event handling]
---
本章将介绍**图形用户界面**(graphical user interface, GUI)的基本知识——包含按钮、文本框等控件的窗口。Tkinter是事实上的Python标准GUI工具包，包含在Python标准安装中。

本章简要地介绍Tkinter的用法。详见文档[tkinter模块](https://docs.python.org/3/library/tkinter.html)和[TkDocs](https://tkdocs.com/)。

## 12.1 创建示例GUI应用程序
下面介绍如何创建一个简单的GUI应用程序。你的任务是编写一个简单的程序，让用户能够编辑文本文件。这个微型文本编辑器的需求如下：
* 能够打开指定的文本文件
* 能够编辑文本文件
* 能够保存文本文件
* 能够退出

编写GUI程序时，绘制其用户界面草图通常很有帮助。下图是一个可满足文本编辑器需求的简单布局。

![文本编辑器草图](/assets/images/python-note-ch12-graphical-user-interfaces/文本编辑器草图.png)

### 12.1.1 初探
首先，必须导入`tkinter`。

```python
>>> from tkinter import *
```

下面创建一个充当主窗口的顶级组件（**控件**(widget)）。为此，实例化一个`Tk`对象。

```python
>>> top = Tk()
```

此时将出现一个窗口，如下图所示。在常规程序中，应该在这里调用函数`mainloop()`进入Tkinter **主事件循环**，但在交互式解释器中不需要这样做。

![空窗口](/assets/images/python-note-ch12-graphical-user-interfaces/空窗口.png)

有很多可用的控件。例如，要创建按钮，实例化`Button`类。

```python
>>> btn = Button()
```

现在这个按钮还不可见——需要使用**布局管理器**(layout manager)（也叫**几何管理器**(geometry manager)）来告诉Tkinter将它放在什么地方。我们将使用**pack**管理器，最简单的情况只需调用`pack()`方法。

```python
>>> btn.pack()
```

控件有各种属性，可用于修改其外观和行为。可以像字典项一样访问属性，因此要给按钮指定文本，只需使用一条赋值语句。

```python
>>> btn['text'] = 'Click me!'
```

至此，应该有如下窗口：

![带按钮的窗口](/assets/images/python-note-ch12-graphical-user-interfaces/带按钮的窗口.png)

给按钮添加行为也非常简单。

```python
>>> def clicked():
...     print('I was clicked!')
...
>>> btn['command'] = clicked
```

现在如果点击按钮，将看到消息被打印出来。

可以使用`config()`方法而不是单独的赋值来同时设置多个属性。

```python
>>> btn.config(text='Click me!', command=clicked)
```

也可以使用控件的构造函数来配置。

```python
>>> Button(text='Click me too!', command=clicked).pack()
```

### 12.1.2 布局
对控件调用`pack()`方法时，将把控件放在其父控件（**主控件**）中。要指定主控件，可以使用构造函数的第一个可选参数；如果没有指定，则使用顶级主窗口。例如：

```python
Label(text="I'm in the first window!").pack()
second = Toplevel()
Label(second, text="I'm in the second window!").pack()
```

`Toplevel`类表示主窗口之外的顶级窗口，`Label`就是文本标签。

没有任何参数时，`pack`从窗口顶部开始将控件堆叠成一列，并水平居中。例如：

```python
for i in range(10):
    Button(text=i).pack()
```

![一列按钮](/assets/images/python-note-ch12-graphical-user-interfaces/一列按钮.png)

可以调整控件的位置和拉伸方式，详见`help(Pack.config)`。

还有其他的布局管理器`grid`和`place`。要使用它们，对控件调用对应方法。`grid`将控件放在不可见的表格单元格中，`place`让你能够手工放置控件，详见`help(Grid.config)`和`help(Place.config)`。

为避免麻烦，在一个容器（例如窗口）中应该只使用一种布局管理器。

### 12.1.3 事件处理
可以通过设置`command`属性给按钮指定动作(action)。这是一种特殊的**事件处理**(event handling)，Tkinter还提供了一种更通用的机制：`bind()`方法。要让控件对特定的事件进行处理，可以对其调用该方法，并指定事件名称和要使用的函数。例如：

```python
>>> from tkinter import *
>>> top = Tk()
>>> def callback(event):
...     print(event.x, event.y)
...
>>> top.bind('<Button-1>', callback)
'2105343784128callback'
```

其中，`<Button-1>`是鼠标左键单击的事件名称。我们将其绑定到`callback()`函数。每次点击`top`窗口内部时，都将调用这个函数，并向其传递一个事件对象，其属性取决于事件类型。详见`help(Tk.bind)`。

### 12.1.4 最终程序
至此，我们大致具备了编写文本编辑器程序所需的知识。最终程序如代码清单12-1所示。

[代码清单12-1 简单文本编辑器](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch12/text_editor.py)

![简单文本编辑器](/assets/images/python-note-ch12-graphical-user-interfaces/简单文本编辑器.png)

## 12.2 使用其他GUI工具包
大部分GUI工具包的基本要素都大致相同。遗憾的是，当学习使用新包时，必须花时间了解让你能够实现目标的细节。因此，你应该花时间来决定使用哪个包，再深入研究其文档并开始编写代码。

参考：
* [Graphic User Interface FAQ](https://docs.python.org/3/faq/gui.html)
* [GUI Programming in Python](https://wiki.python.org/moin/GuiProgramming)
