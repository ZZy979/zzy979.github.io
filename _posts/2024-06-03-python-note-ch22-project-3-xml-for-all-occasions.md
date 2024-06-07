---
title: 《Python基础教程》笔记 第22章 项目3：万能的XML
date: 2024-06-03 21:46:05 +0800
categories: [Python, Beginning Python]
tags: [python, xml, sax]
---
这个项目的目标是根据描述各种网页和目录的单个XML文件生成完整的网站。

有关XML的简洁描述，参见W3C网站的文章 "[XML in 10 points](https://www.w3.org/XML/1999/XML-in-10-points-19990327)" 。XML的详尽教程可以在W3Schools网站(<https://www.w3schools.com/xml/>)上找到。有关SAX的详细信息，参见官网(<http://www.saxproject.org/>)。

## 22.1 问题描述
在这个项目中，要解决的通用问题是解析（读取并处理）XML文件。鉴于XML几乎能用来表示任何信息，而你可以在解析时对数据做任何处理，因此应用场景是无限的（正如本章标题指出）。本章要解决的具体问题是根据一个XML文件生成完整的网站，这个文件描述了网站结构以及每个网页的基本内容。

下面确定这个项目的具体目标。
* 整个网站应该由单个XML文件描述，该文件包含有关各个网页和目录的信息。
* 程序应该根据需要创建目录和网页。
* 应该能够轻松地修改整个网站的设计，并使用新的设计重新生成所有网页。

## 22.2 有用的工具
Python有一些内置的XML支持。在这个项目中，需要一个可用的SAX (Simple API for XML)解析器。要确定是否已经有可用的SAX解析器，可尝试执行以下代码：

```python
>>> from xml.sax import make_parser
>>> parser = make_parser()
```

应该不会引发任何异常。

提示：有很多用于Python的XML工具。除了“标准”框架外，另一个很有趣的选择是ElementTree，包含在Python标准库的`xml.etree`包中。

## 22.3 准备工作
在编写处理XML文件的程序前，必须先设计XML格式。需要什么标签？应该包含哪些属性？每个标签应该放在哪里？要回答这些问题，首先要考虑你想用XML格式来描述什么。

主要的概念包括网站、目录、页面、名称、标题和内容。
* **网站**(web site)是顶级元素，包含所有的文件和目录。
* **目录**(directory)是文件和其他目录的容器。
* **页面**(page)是单个网页。
* 目录和网页都需要**名称**(name)。这些名称用作目录名和文件名，将出现在文件系统和相应的URL中。
* 每个网页都应该有**标题**(title)（不同于文件名）。
* 每个网页都包含一些**内容**(contents)。这里用普通的XHTML来表示内容。

总之，XML文档只包含一个`<website>`元素，它包含多个`<directory>`和`<page>`元素，每个`<directory>`元素可能包含更多`<page>`和`<directory>`。`<directory>`和`<page>`元素都包含`name`属性，表示其名称。另外，`<page>`元素还有`title`属性。`<page>`元素包含XHTML代码。代码清单22-1是一个示例文件。

[代码清单22-1 表示简单网站的XML文件](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch22/website.xml)

## 22.4 初次实现
这里使用的XML解析方法(SAX)需要编写一组事件处理器，并让XML解析器在读取XML文档时调用这些处理器（基本思想类似于第15章介绍的`HTMLParser`）。

注意：处理XML的两种常见方式是SAX和文档对象模型(Document Object Model, DOM)。SAX解析器读取XML文件并指出发现的内容（文本、标签和属性），每次只存储文档的一小部分。这让SAX简单、快速且占用内存较少。DOM采用的是另一种方法：创建一个表示整个文档的数据结构（**文档树**）。这种方法比较慢、需要更多内存，但在需要操作文档结构时很有用。

### 22.4.1 创建简单的内容处理器
使用SAX进行解析时，有多种可用的事件类型，但这里只使用三种：元素开始（遇到开始标签）、元素结束（遇到结束标签）和普通文本（字符）。要解析XML文件，使用`xml.sax`模块中的`parse()`函数。这个函数负责读取文件并生成事件，在生成事件时，它需要调用一些事件处理器。这些事件处理器将实现为**内容处理器**(content handler)对象的方法。为此，继承`xml.sax.handler`模块中的`ContentHandler`类，它实现了所有必要的事件处理器（没有任何效果），你只需覆盖需要的那些。

下面从一个极简的XML解析器开始（假设XML文件叫做website.xml）：

[测试解析器](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch22/test_handler.py)

事件处理器`startElement()`的参数是标签名及其属性（保存在类似于字典的对象中）。如果运行这个程序，将看到如下输出：

```
website []
page ['name', 'title']
h1 []
p []
ul []
li []
a ['href']
li []
a ['href']
li []
a ['href']
directory ['name']
page ['name', 'title']
h1 []
p []
page ['name', 'title']
h1 []
p []
page ['name', 'title']
h1 []
p []
```

其工作原理应该非常清晰。除了`startElement()`外，还将使用`endElement()`和`characters()`。

下面的示例使用这三个方法来构建网站文件中所有标题（`<h1>`元素）的列表。

[标题解析器](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch22/headline_handler.py)

注意，`HeadlineHandler`跟踪当前解析的文本是否位于一对`h1`标签内：当`startElement()`发现`h1`标签时将`self.in_headline`置为`True`，当`endElement()`发现`h1`标签时将`self.in_headline`置为`False`。`characters()`方法在解析器遇到文本时自动被调用，只要当前位于`h1`标签内，就将字符串（可能只是标签内文本的**一部分**）添加到`self.data`列表（注：代码清单15-2中的`chunks`属性使用了同样的方法）。合并这些文本片段、将其添加到`self.headlines`并将`self.in_headline`置为`False`的工作是由`endElement()`完成的。这种通用方法（使用布尔变量来指出当前是否在特定类型的标签内）在SAX编程中很常见。

运行这个程序将得到如下输出：

```
The following <h1> elements were found:
Welcome to My Home Page
Mr. Gumby's Shouting Page
Mr. Gumby's Sleeping Page
Mr. Gumby's Eating Page
```

### 22.4.2 创建HTML页面
现在已经准备好创建原型了。暂时先忽略目录，而专注于创建HTML页面。你需要创建事件处理器，使其执行以下操作：
* 在每个`page`元素的开头，使用给定的名称打开一个新文件，并写入合适的HTML头部(header)（包括给定的标题）。
* 在每个`page`元素的末尾，将合适的HTML尾部(footer)写入文件，并关闭文件。
* 在`page`元素内部，遍历所有的标签和字符，不做修改（将其原样写入文件）。
* 在`page`元素外部，忽略所有标签（例如`website`和`directory`）。

大部分内容都很简单。然而，有两个问题可能不那么显而易见。
* 你不能简单地“穿过”HTML标签（直接将其写入HTML文件），因为只给你提供了标签的名称和属性（即`page`元素中的HTML标签将会与其他XML标签一样被解析，而不是像文本那样直接作为字符串处理）。因此，你必须自己重建这些标签（使用尖括号等）（注：将标签名和属性重新构造成`<name attr="value">`的形式）。
* SAX本身无法告诉你当前是否在`page`元素内部，你必须自己跟踪这一点（就像`HeadlineHandler`示例中那样）。这里使用一个名为`passthrough`的布尔变量，并在进入和离开`page`元素时更新。

原型程序的代码如代码清单22-2所示。

[代码清单22-2 简单的页面创建脚本](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch22/pagemaker.py)

在要生成文件的目录中执行这个程序。注意，由于忽略了目录，生成的超链接并不能指向正确的文件（再次实现时将修复这一问题）。

使用代码清单22-1的文件website.xml，将得到4个HTML文件，其中index.html的内容如下：

```html
<html><head>
<title>Home Page</title>
</head><body>

    <h1>Welcome to My Home Page</h1>

    <p>Hi, there. My name is Mr. Gumby, and this is my home page. Here
    are some of my interests:</p>

    <ul>
      <li><a href="interests/shouting.html">Shouting</a></li>
      <li><a href="interests/sleeping.html">Sleeping</a></li>
      <li><a href="interests/eating.html">Eating</a></li>
    </ul>
  
</body></html>
```

下图是在浏览器中查看这个页面的结果。

![生成的网页](/assets/images/python-note-ch22-project-3-xml-for-all-occasions/生成的网页.png)

这个程序有两个明显的缺点：
* 使用`if`语句来处理各种事件类型。如果要处理的事件种类很多，`if`语句变得很长且难以理解。
* HTML代码是硬编码的(hardwired)（手动拼接），而它应该很容易替换。

## 22.5 再次实现
因为SAX机制较底层且基础，编写一个混入类来处理管理性细节（例如收集文本，管理布尔状态变量，将事件分派给自定义事件处理器等）通常很有帮助。

### 22.5.1 分派器混入类
与其在标准事件处理器（例如`startElement()`）中编写很长的`if`语句，不如只编写自定义的特定事件处理器（例如`startPage()`）并让它们自动被调用。可以在一个混入类中实现这个功能，然后将这个类和`ContentHandler`一起继承。

注意：**混入类**(mix-in class)是一种功能有限、旨在与其他更实质的类一起被继承的类。

你希望程序具有如下功能：
* 当使用名字`'foo'`调用`startElement()`时，它应该试图寻找叫做`startFoo()`的事件处理器，并使用给定的属性调用它。
* 类似地，如果使用`'foo'`调用`endElement()`，它应该尝试调用`endFoo()`。
* 如果没有找到相应的处理器，则应该调用方法`defaultStart()`或`defaultEnd()`。如果默认处理器也没有，就什么都不做。

关于参数，自定义处理器（例如`startFoo()`）无需将标签名作为参数，而默认处理器需要。另外，只有开始处理器需要将属性作为参数。

先来编写这个类最简单的部分。

```python
class Dispatcher:
    # ...
    def startElement(self, name, attrs):
        self.dispatch('start', name, attrs)
    def endElement(self, name):
        self.dispatch('end', name)
```

这里实现了基本的事件处理器，它们只是调用`dispatch()`方法，而`dispatch()`负责查找合适的处理器、创建参数元组并使用这些参数调用处理器。下面是`dispatch()`方法的代码：

```python
def dispatch(self, prefix, name, attrs=None):
    mname = prefix + name.capitalize()
    dname = 'default' + prefix.capitalize()
    method = getattr(self, mname, None)
    if callable(method):
        args = ()
    else:
        method = getattr(self, dname, None)
        args = name,
    if prefix == 'start':
        args += attrs,
    if callable(method):
        method(*args)
```

这个方法所做的工作如下：
1. 根据前缀（`'start'`或`'end'`）和标签名（例如`'page'`）构造处理器的方法名（例如`'startPage'`）。
2. 根据相同的前缀构造默认处理器的名称（例如`'defaultStart'`）。
3. 尝试使用`getattr()`获取处理器，用`None`作为默认值。
4. 如果结果是可调用的（即找到了对应的处理器），则将`args`赋值为空元组。
5. 否则，尝试使用`getattr()`获取默认处理器，并将`args`设置为只包含标签名的元组。
6. 如果要调用的是开始处理器，则将属性添加到参数元组中。
7. 如果处理器是可调用的（即找到了具体处理器或默认处理器），则使用正确的参数调用它。

这意味着现在可以像这面这样编写内容处理器：

```python
class TestHandler(Dispatcher, ContentHandler):
    def startPage(self, attrs):
        print('Beginning page', attrs['name'])
    def endPage(self):
        print('Ending page')
```

注：
* 这里的`dispatch()`方法使用了与第20章中的`Handler.callback()`同样的技术。
* `Dispatcher`类定义了`startElement()`等方法，但没有继承`ContentHandler`类。当具体的类（例如`TestHandler`）同时继承这两个类时，`Dispatcher.startElement()`将“混入”子类并覆盖`ContentHandler`的方法。注意，由于MRO，`Dispatcher`在超类列表中必须位于`ContentHandler`之前，详见[Python多继承的MRO和构造函数问题]({% post_url 2020-10-15-python-multi-inheritance-mro-constructor %})。

### 22.5.2 提取头部、尾部和默认处理方法
我们将编写专门用于写头部和尾部的方法，而不是直接在事件处理器中调用`self.out.write()`。这样就可以很容易地通过继承事件处理器来覆盖这些方法。

注：以下方法都是`WebsiteConstructor`类的，不是`Dispatcher`类的。

```python
def writeHeader(self, title):
    self.out.write('<html>\n  <head>\n    <title>')
    self.out.write(title)
    self.out.write('</title>\n  </head>\n  <body>\n')

def writeFooter(self):
    self.out.write('\n  </body>\n</html>\n')
```

在初次实现中，处理XHTML内容的代码还与处理器耦合太紧，现在它们将由`defaultStart()`和`defaultEnd()`处理。

```python
def defaultStart(self, name, attrs):
    if self.passthrough:
        self.out.write('<' + name)
        for key, val in attrs.items():
            self.out.write(' {}="{}"'.format(key, val))
        self.out.write('>')

def defaultEnd(self, name):
    if self.passthrough:
        self.out.write('</{}>'.format(name))
```

这些代码与前面相同，只是移到了单独的方法中（这通常是件好事）。

### 22.5.3 支持目录
为了创建目录，需要使用函数`os.makedirs()`，它在给定的路径中创建所有必要的目录。例如，`os.makedirs('foo/bar/baz')`在当前目录下创建目录foo，然后在foo中创建bar，最后在bar中创建baz。如果foo已存在，则只创建bar和baz；如果bar也存在，则只创建baz。然而，如果baz也已存在，将引发异常。为避免这一问题，设置关键字参数`exist_ok=True`。另一个有用的函数是`os.path.join()`，它使用正确的分隔符（在UNIX中为`/`，在Windows中为`\`）连接多个路径。

在处理过程中，将当前目录保存为目录名称列表，由变量`directory`引用。进入目录时，将其名称添加到列表；离开目录时，将其名称弹出（例如，foo/bar → `['foo', 'bar']`，离开bar目录 → `['foo']`，进入baz目录 → `['foo', 'baz']`）。

注：将目录保存为列表而不是字符串是为了便于修改。还可以使用`pathlib.Path`对象来操作路径，从而可以使用`p / 'foo'`和`p.parent`代替`append('foo')`和`pop()`。

可以定义一个函数来确保当前目录存在。

```python
def ensureDirectory(self):
    path = os.path.join(*self.directory)
    os.makedirs(path, exist_ok=True)
```

注意，将目录列表传递给`os.path.join()`时使用了参数解包（`*`运算符）（例如，`os.path.join(*['foo', 'bar']) = os.path.join('foo', 'bar') = 'foo/bar'`）。

可以通过参数将网站的根目录（例如public_html）传递给构造函数，如下所示：

```python
def __init__(self, directory):
    self.directory = [directory]
    self.ensureDirectory()
```

### 22.5.4 事件处理器
终于要实现事件处理器了。需要4个处理器：2个用于处理目录，2个用于处理页面。目录处理器只使用了`directory`列表和`ensureDirectory()`方法。

```python
def startDirectory(self, attrs):
    self.directory.append(attrs['name'])
    self.ensureDirectory()

def endDirectory(self):
    self.directory.pop()
```

页面处理器调用`writeHeader()`和`writeFooter()`方法、设置`passthrough`变量，以及打开和关闭与页面关联的文件。

```python
def startPage(self, attrs):
    filename = os.path.join(*self.directory + [attrs['name'] + '.html'])
    self.out = open(filename, 'w')
    self.writeHeader(attrs['title'])
    self.passthrough = True

def endPage(self):
    self.passthrough = False
    self.writeFooter()
    self.out.close()
```

`startPage()`的第一行与`ensureDirectory()`的第一行大致相同，只是添加了文件名。

注：参数解包运算符`*`的优先级**低于** `+`，即`f(*a + b)`等价于`f(*(a + b))`（这与来自C++的直觉相反）。因此`os.path.join(*['foo', 'bar'] + ['baz.html']) = os.path.join(*['foo', 'bar', 'baz.html']) = os.path.join('foo', 'bar', 'baz.html') = 'foo/bar/baz.html'`。

程序的完整代码如代码清单22-3所示。

[代码清单22-3 网站生成器](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch22/website.py)

程序将生成以下文件和目录：

```
public_html/
    index.html
    interests/
        shouting.html
        sleeping.html
        eating.html
```

## 22.6 进一步探索
现在已经有了一个基本程序，可以对其做哪些扩展呢？下面是一些建议：
* 创建一个新的`ContentHandler`，用于为网站生成目录或菜单（带有链接）。
* 在网页中添加导航帮助，告诉用户位于何处（哪个目录中）。
* 创建一个`WebsiteConstructor`的子类，覆盖`writeHeader()`和`writeFooter()`方法，以提供自定义设计。
* 创建另一个`ContentHandler`，根据XML文件创建单个网页。
* 创建一个`ContentHandler`，以某种方式（例如RSS）提供网站内容摘要。
* 研究其他XML转换工具，尤其是XML Transformations (XSLT)。
* 使用ReportLab的Platypus等工具根据XML文件创建一个或多个PDF文档。
* 实现通过Web界面编辑XML文件的功能（参见第25章）。
