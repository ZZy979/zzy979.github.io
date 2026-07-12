---
title: PyQt基础教程
date: 2021-06-17 19:54 +0800
categories: [Qt]
tags: [qt, pyqt, gui]
---
## 1.简介
Qt是一个功能强大的跨平台C++图形用户界面(GUI)框架，广泛用于开发桌面、移动和嵌入式系统的应用程序，支持多种操作系统，并提供了丰富的工具和模块来简化开发过程。
* 官方网站：<https://www.qt.io/>
* 官方文档：<https://doc.qt.io/>
* 下载地址：<https://www.qt.io/development/download-qt-installer-oss>

Qt的Python工具包对Qt库进行了封装，可以使用Python开发Qt应用程序。主要有两个Python工具包：PySide和PyQt。

### PySide
PySide是Qt官方的Qt for Python项目，提供了Qt库的Python API。
* 官方文档：<https://doc.qt.io/qtforpython-6/>
* 官方教程：<https://doc.qt.io/qtforpython-6/tutorials/index.html>
* 示例：<https://doc.qt.io/qtforpython-6/examples/index.html>

安装：

```shell
pip install PySide6
```

PySide6对应Qt6，PySide2对应Qt5，PySide对应Qt4。

验证安装成功：

```python
import PySide6.QtCore

# Prints PySide6 version
print(PySide6.__version__)

# Prints the Qt version used to compile PySide6
print(PySide6.QtCore.__version__)
```

### PyQt
PyQt是Riverbank Computing公司开发的Qt Python工具包，比PySide出现得早，API与PySide基本相同。二者的区别详见[Differences Between PySide and PyQt](https://wiki.qt.io/Differences_Between_PySide_and_PyQt)。
* 官方网站：<https://www.riverbankcomputing.com/software/pyqt/>
* 官方文档：<https://www.riverbankcomputing.com/static/Docs/PyQt6/>

安装：

```shell
pip install PyQt6
```

PyQt6对应Qt6，PyQt5对应Qt5，PyQt4对应Qt4。

## 2.入门
<https://doc.qt.io/qtforpython-6/gettingstarted.html>

Qt提供了两种构建UI的方式：
* Qt Widgets：**命令式**编程和设计方式，自Qt诞生以来就存在，稳定可靠。
* Qt Quick：**声明式**编程和设计方式，能够通过用简单元素描述来创建流畅的UI。

### 2.1 使用Qt Widgets创建第一个Qt应用
下面开发一个用多种语言打印 "Hello World" 的简单应用程序。

（1）创建一个名为hello_world.py的文件，并添加以下导入：

```python
import random
import sys

from PySide6 import QtWidgets, QtCore
```

`PySide6`模块的子模块提供了对Qt API的访问。在这里导入了子模块`QtCore`和`QtWidgets`。

注：如果选择PyQt6，需要将模块名从`PySide6`改为`PyQt6`。

（2）定义一个名为`MyWidget`的类，该类扩展`QWidget`，包含一个按钮和一个标签：

```python
class MyWidget(QtWidgets.QWidget):

    def __init__(self):
        super().__init__()

        self.hello = ["Hello World", "Hallo Welt", "Hei maailma", "Hola Mundo", "Привет мир", "你好世界"]

        self.button = QtWidgets.QPushButton("Click me!")
        self.text = QtWidgets.QLabel("Hello World", alignment=QtCore.Qt.AlignCenter)

        self.layout = QtWidgets.QVBoxLayout(self)
        self.layout.addWidget(self.text)
        self.layout.addWidget(self.button)

        self.button.clicked.connect(self.magic)

    @QtCore.Slot()
    def magic(self):
        self.text.setText(random.choice(self.hello))
```

当点击按钮时，会调用`magic()`函数，从`hello`列表中随机选择一项并更新标签文本。

（3）添加main函数，实例化`MyWidget`并显示：

```python
if __name__ == '__main__':
    app = QtWidgets.QApplication([])

    widget = MyWidget()
    widget.resize(300, 200)
    widget.show()

    sys.exit(app.exec())
```

完整代码：[hello_world.py](https://github.com/ZZy979/pyqt-tutorial/blob/main/getting_started/hello_world.py)

使用以下命令运行该示例：

```shell
python hello_world.py
```

点击底部按钮，会看到不同语言的问候语。

![Hello World application in Qt Widgets](/assets/images/pyqt-tutorial/hello_widgets.png)

### 2.2 使用Qt Quick创建第一个Qt应用
下面使用Qt Quick实现同样的功能。

UI可以用QML语言描述：

```
import QtQuick
import QtQuick.Controls
import QtQuick.Layouts

Window {
    width: 300
    height: 200
    visible: true
    title: "Hello World"

    readonly property list<string> texts: [
        "Hello World", "Hallo Welt", "Hei maailma", "Hola Mundo", "Привет мир", "你好世界"
    ]

    function setText() {
        var i = Math.round(Math.random() * 5)
        text.text = texts[i]
    }

    ColumnLayout {
        anchors.fill:  parent

        Text {
            id: text
            text: "Hello World"
            Layout.alignment: Qt.AlignHCenter
        }
        Button {
            text: "Click me"
            Layout.alignment: Qt.AlignHCenter
            onClicked: setText()
        }
    }
}
```

将以上代码放在名为Example的目录中的Main.qml文件中，另外用一个名为qmldir的文件描述QML模块：

```
module Example
Main 254.0 Main.qml
```

注：QML完整语法参见文档[QML Reference](https://doc.qt.io/qt-6/qmlreference.html)。

最后创建一个hello_world_quick.py文件（与Example目录并列），在main函数中实例化`QQmlApplicationEngine`并加载QML：

```python
import sys

from PySide6.QtGui import QGuiApplication
from PySide6.QtQml import QQmlApplicationEngine

if __name__ == "__main__":
    app = QGuiApplication(sys.argv)
    engine = QQmlApplicationEngine()
    engine.addImportPath(sys.path[0])
    engine.loadFromModule("Example", "Main")
    if not engine.rootObjects():
        sys.exit(-1)
    exit_code = app.exec()
    del engine
    sys.exit(exit_code)
```

完整代码：
* [hello_world_quick.py](https://github.com/ZZy979/pyqt-tutorial/blob/main/getting_started/hello_world_quick.py)
* [Example/Main.qml](https://github.com/ZZy979/pyqt-tutorial/blob/main/getting_started/Example/Main.qml)
* [Example/qmldir](https://github.com/ZZy979/pyqt-tutorial/blob/main/getting_started/Example/qmldir)

使用以下命令运行该示例：

```shell
python hello_world_quick.py
```

![Hello World application in Qt Quick](/assets/images/pyqt-tutorial/hello_quick.png)

## 3.Qt Widgets基础教程
<https://doc.qt.io/qtforpython-6/tutorials/index.html>

Qt内置构件的完整列表参见文档[Qt Widgets](https://doc.qt.io/qt-6/qtwidgets-index.html)和[Widgets Classes](https://doc.qt.io/qt-6/widget-classes.html)。

### 3.1 构件
<https://doc.qt.io/qtforpython-6/tutorials/basictutorial/widgets.html>

下面是PySide6的简单Hello World示例：

```python
import sys

from PySide6.QtWidgets import QApplication, QLabel

app = QApplication(sys.argv)
label = QLabel("Hello World!")
label.show()
app.exec()
```

[hello_world.py](https://github.com/ZZy979/pyqt-tutorial/blob/main/basic_tutorial/hello_world.py)

* 从`PySide6.QtWidgets`模块导入需要的类。
* 创建`QApplication`实例，可以传递命令行参数。
* `QLabel`是一个可以展示文本（纯文本或HTML）和图像的**构件**(widget)，创建后调用其`show()`方法。
* 最后调用`app.exec()`进入Qt主循环。

![Simple Widget](/assets/images/pyqt-tutorial/widgets.png)

### 3.2 按钮
<https://doc.qt.io/qtforpython-6/tutorials/basictutorial/clickablebutton.html>

本教程介绍如何处理信号和槽。**信号**(signal)和**槽**(slot)是一项Qt的特性，允许图形构件与其他构件或Python代码进行通信。本示例将创建一个按钮，每次点击时都会向控制台发送消息 "Button clicked, Hello!" 。

首先导入必要的模块和类：

```python
import sys
from PySide6.QtCore import Slot
from PySide6.QtWidgets import QApplication, QPushButton
```

创建一个向控制台打印消息的函数：

```python
@Slot()
def say_hello():
    print("Button clicked, Hello!")
```

`@Slot()`是一个装饰器，用于将函数标识为槽。

创建`QApplication`：

```python
# Create the Qt Application
app = QApplication(sys.argv)
```

接下来创建一个按钮，这是一个`QPushButton`实例，传递一个字符串给构造函数作为按钮标签：

```python
# Create a button
button = QPushButton("Click me")
```

之后需要将按钮**连接**到`say_hello()`函数。`QPushButton`有一个预定义的信号`clicked`，在按钮被点击时触发。将该信号连接到`say_hello()`函数：

```python
# Connect the button to the function
button.clicked.connect(say_hello)
```

最后显示按钮并启动Qt主循环：

```python
# Show the button
button.show()
# Run the main Qt loop
app.exec()
```

完整代码：[button.py](https://github.com/ZZy979/pyqt-tutorial/blob/main/basic_tutorial/button.py)

![Clickable Button Example](/assets/images/pyqt-tutorial/clickablebutton.png)

信号和槽的详细介绍参见文档[Signals and Slots](https://doc.qt.io/qtforpython-6/tutorials/basictutorial/signals_and_slots.html)。

### 3.3 对话框
<https://doc.qt.io/qtforpython-6/tutorials/basictutorial/dialog.html>

本教程介绍如何创建一个包含基本构件的简单对话框，让用户在输入框中输入姓名，点击按钮后显示问候语。

首先创建并显示一个简单的对话框：

```python
import sys

from PySide6.QtWidgets import QDialog, QApplication


class Form(QDialog):

    def __init__(self, parent=None):
        super().__init__(parent)
        self.setWindowTitle("My Form")


if __name__ == "__main__":
    # Create the Qt Application
    app = QApplication(sys.argv)
    # Create and show the form
    form = Form()
    form.show()
    # Run the main Qt loop
    sys.exit(app.exec())
```

可以创建任何构件的子类。在这个示例中，通过继承`QDialog`创建了一个自定义对话框，称为`Form`。在main函数中创建了一个`Form`对象，显示一个空白对话框。

![Empty Dialog](/assets/images/pyqt-tutorial/empty_dialog.png)

#### 创建构件
下面创建两个构件：一个`QLineEdit`，用户在这里输入姓名；一个`QPushButton`，点击时打印问候语。将以下代码添加到`Form`的`__init__()`方法：

```python
# Create widgets
self.edit = QLineEdit("Write my name here..")
self.button = QPushButton("Show Greetings")
```

#### 创建布局
Qt支持**布局**(layout)，可以帮助组织应用程序中的构件。在这里使用`QVBoxLayout`来垂直排列构件。将以下代码添加到`__init__()`方法创建构件的代码之后：

```python
# Create layout and add widgets
layout = QVBoxLayout(self)
layout.addWidget(self.edit)
layout.addWidget(self.button)
```

#### 创建函数并连接到按钮
向`Form`类添加一个方法：

```python
# Greets the user
def greetings(self):
    print(f"Hello {self.edit.text()}")
```

使用`QLineEdit.text()`方法获取文本。

最后将按钮连接到`Form.greetings()`方法，向`__init__()`方法添加以下代码：

```python
# Add button signal to greetings slot
self.button.clicked.connect(self.greetings)
```

完整代码：[dialog.py](https://github.com/ZZy979/pyqt-tutorial/blob/main/basic_tutorial/dialog.py)

执行代码，输入名字并点击按钮，会在控制台显示问候语。

![Simple Dialog Example](/assets/images/pyqt-tutorial/dialog.png)

### 3.4 表格
<https://doc.qt.io/qtforpython-6/tutorials/basictutorial/tablewidget.html>

如果要显示表格数据，可以使用`QTableWidget`。还可以创建数据模型并使用`QTableView`显示。

注：关于Qt中的Model/View架构详见文档[Model/View Programming](https://doc.qt.io/qt-6/model-view-programming.html)。

1.导入类：

```python
import sys
from PySide6.QtGui import QColor
from PySide6.QtWidgets import QApplication, QTableWidget, QTableWidgetItem
```

2.创建一个简单的数据模型，包含不同颜色的名字和十六进制代码列表：

```python
colors = [
    ("Red", "#FF0000"),
    ("Green", "#00FF00"),
    ("Blue", "#0000FF"),
    ("Black", "#000000"),
    ("White", "#FFFFFF"),
    ("Electric Green", "#41CD52"),
    ("Dark Blue", "#222840"),
    ("Yellow", "#F9E56d")
]
```

3.定义一个将十六进制代码转换为RGB的函数：

```python
def get_rgb_from_hex(code):
    code_hex = code.replace("#", "")
    rgb = tuple(int(code_hex[i:i + 2], 16) for i in (0, 2, 4))
    return QColor.fromRgb(rgb[0], rgb[1], rgb[2])
```

4.实例化`QApplication`：

```python
app = QApplication()
```

5.创建`QTableWidget`，设置行数等于颜色的个数，列数等于每个颜色的成员数+1（用于显示颜色），使用`setHorizontalHeaderLabels()`设置列名：

```python
table = QTableWidget()
table.setRowCount(len(colors))
table.setColumnCount(len(colors[0]) + 1)
table.setHorizontalHeaderLabels(["Name", "Hex Code", "Color"])
```

6.遍历`colors`，对于每个单元格创建一个`QTableWidgetItems`实例，使用(x, y)坐标将其添加到表格：

```python
for i, (name, code) in enumerate(colors):
    item_name = QTableWidgetItem(name)
    item_code = QTableWidgetItem(code)
    item_color = QTableWidgetItem()
    item_color.setBackground(get_rgb_from_hex(code))
    table.setItem(i, 0, item_name)
    table.setItem(i, 1, item_code)
    table.setItem(i, 2, item_color)
```

7.显示表格并执行`QApplication`：

```python
table.show()
sys.exit(app.exec())
```

完整代码：[table.py](https://github.com/ZZy979/pyqt-tutorial/blob/main/basic_tutorial/table.py)

最终程序如下图所示：

![QTableWidget example](/assets/images/pyqt-tutorial/tablewidget.png)

### 3.5 树
<https://doc.qt.io/qtforpython-6/tutorials/basictutorial/treewidget.html>

如果要显示树形数据，可以使用`QTreeWidget`。还可以创建数据模型并使用`QTreeView`显示。

1.导入类：

```python
import sys
from PySide6.QtWidgets import QApplication, QTreeWidget, QTreeWidgetItem
```

2.定义一个包含项目结构的字典，以树形结构显示信息，其中包含每个项目的文件：

```python
data = {
    "Project A": ["file_a.py", "file_a.txt", "something.xls"],
    "Project B": ["file_b.csv", "photo.jpg"],
    "Project C": []
}
```

3.实例化`QApplication`：

```python
app = QApplication()
```

4.创建`QTreeWidget`，包含两列，一列是项目/文件名称，另一列是文件类型。使用`setHeaderLabels()`设置列名：

```python
tree = QTreeWidget()
tree.setColumnCount(2)
tree.setHeaderLabels(["Name", "Type"])
```

5.遍历`data`，对于每个节点创建一个`QTreeWidgetItem`实例，并添加到对应的父节点。只对文件提取扩展名，并将其添加到第二列。

```python
items = []
for key, values in data.items():
    item = QTreeWidgetItem([key])
    for value in values:
        ext = value.split(".")[-1].upper()
        child = QTreeWidgetItem([value, ext])
        item.addChild(child)
    items.append(item)

tree.insertTopLevelItems(0, items)
```

6.显示树并执行`QApplication`：

```python
tree.show()
sys.exit(app.exec())
```

完整代码：[tree.py](https://github.com/ZZy979/pyqt-tutorial/blob/main/basic_tutorial/tree.py)

最终程序如下图所示：

![QTreeWidget example](/assets/images/pyqt-tutorial/treewidget.png)
