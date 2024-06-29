---
title: 《Python基础教程》笔记 第26章 项目7：自建公告板
date: 2024-06-24 22:43:03 +0800
categories: [Python, Beginning Python]
tags: [python, cgi, database]
---
本章将实现一个基于Web的论坛。

## 26.1 问题描述
在这个项目中，你将创建一个通过Web发布和回复消息的简单系统，可以作为论坛(discussion forum)使用。

本章介绍的技术不仅可用于开发独立的论坛，还可用于实现更通用的协作系统，例如问题跟踪(issue-tracking)系统、带评论功能的博客等。将CGI和数据库结合使用功能强大且用途广泛。

具体地说，最终的系统应该满足以下需求：
* 显示当前所有消息的主题。
* 支持消息主题帖(threading)（在消息下方缩进显示回复）。
* 用户能够查看现有的消息。
* 用户能够回复现有的消息。

除了这些功能需求外，如果系统能达到以下目标就更好了：非常稳定，能够处理大量的消息，避免两个用户同时写入同一个文件等问题。为了实现这样的健壮性，可以使用数据库服务器，而不是自己编写文件处理代码。

## 26.2 有用的工具
除了第15章讨论的CGI外，还需要一个SQL数据库，这在第13章讨论过。可以使用第13章中的单机数据库SQLite，也可以使用其他系统，例如下面这两种优秀的免费数据库：
* PostgreSQL (<https://www.postgresql.org/>)
* MySQL (<https://www.mysql.com/>)

本章使用的是SQLite，但只需对代码稍作修改就可以使用其他SQL数据库。

首先，确保你能够访问SQL数据库服务器。另外，还需要能够与服务器交互的Python模块。这种模块大都支持第13章详细讨论过的Python DB API。每种数据库对应的模块如下表所示。

| SQL数据库 | Python模块 |
| --- | --- |
| SQLite | [sqlite3](https://docs.python.org/3/library/sqlite3.html) |
| MySQL | [PyMySQL](https://pymysql.readthedocs.io/)或[MySQLdb](https://mysqlclient.readthedocs.io/index.html) |
| PostgreSQL | [Psycopg](https://www.psycopg.org/) |

## 26.3 准备工作
在程序使用数据库之前，必须先创建它。

数据库的结构取决于要解决的问题。这个项目的数据库只有一张表，每行对应一条消息。每条消息都有唯一的ID（整数）、主题、发布者以及一些文本（正文）。另外，因为你希望能够以层次方式显示消息，每条消息还应存储它所回复消息的引用(ID)。在PostgreSQL中的`CREATE TABLE`命令如代码清单26-1所示。

[代码清单26-1 在PostgreSQL中创建数据库](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch26/create_table_pg.sql)

代码清单26-2是这个命令的MySQL版本（注：需要先使用`CREATE DATABASE`创建数据库）。

[代码清单26-2 在MySQL中创建数据库](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch26/create_table_mysql.sql)

代码清单26-3是用于SQLite的命令。

[代码清单26-3 在SQLite中创建数据库](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch26/create_table_sqlite.sql)

这个SQL语句创建的新表包含以下5个字段：
* `id`：用于标识消息。
* `subject`：包含消息主题的字符串。
* `sender`：包含发送者姓名、Email地址等信息的字符串。
* `reply_to`：如果这条消息回复了另一条消息，这个字段将包含那条消息的`id`，否则为空。
* `text`：包含消息正文的字符串。

注：SQLite没有类似于MySQL的命令行工具，因此必须使用Python代码来执行创建表的命令。例如：

[执行SQL命令](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch26/execute_sql.py)

```shell
$ python execute_sql.py messages.db create_table_sqlite.sql
```

## 26.4 初次实现
第一个原型的功能很有限，只有一个使用数据库功能的脚本。代码的CGI部分与第25章很相似，另外还应该回顾一下15.4.2节。

Python DB API的工作原理参见第13章。下面是一个简单的测试，获取数据库中的所有消息（数据库当前为空，因此结果也为空）：

```python
>>> import sqlite3
>>> conn = sqlite3.connect('messages.db')
>>> curs = conn.cursor()
>>> curs.execute('SELECT * FROM messages')
<sqlite3.Cursor object at 0x000002D1F09A5EC0>
>>> curs.fetchall()
[]
```

由于还没有实现Web接口，因此要测试这个数据库，必须手动输入消息。为此，可以使用管理工具（`mysql`或`psql`），也可以在Python解释器中使用数据库模块。

下面是添加消息的代码，以便于测试：

[添加消息](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch26/addmessage.py)

试着添加几条消息并在Python解释器中查看数据库。下面编写访问数据库的CGI脚本，如代码清单26-4所示。唯一的新知识是格式化代码，用于获得主题帖外观（在消息右下方显示回复）。

[代码清单26-4 公告板主页](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch26/cgi-bin/simple_main.cgi)

其基本工作原理如下：
1. 对于每条消息，获取其`reply_to`字段。如果是`None`（不是回复），则将其添加到顶级消息列表。否则，将其添加到子消息列表`children[parent_id]`。
2. 对于每条顶级消息，调用`format()`函数，打印消息的主题。如果有子消息（回复），就打印起始标签`<blockquote>`，对每条子消息递归地调用`format()`，再打印结束标签`</blockquote>`。

注：
* `curs.description`包含查询结果的字段信息，每个字段对应一个元组，其中第一个元素是字段名。详见[Cursor attributes](https://peps.python.org/pep-0249/#cursor-attributes)。
* PyMySQL和Psycopg本身就支持以字典形式返回结果行（前者使用[DictCursor](https://pymysql.readthedocs.io/en/latest/modules/cursors.html#pymysql.cursors.DictCursor)，后者使用[dict_row](https://www.psycopg.org/psycopg3/docs/api/rows.html#psycopg.rows.dict_row)），而SQLite必须自己转换。

原型的运行结果如下图所示。

![原型运行结果-主页](/assets/images/python-note-ch26-project-7-your-own-bulletin-board/原型运行结果-主页.png)

## 26.5 再次实现
初次实现的功能很有限，用户甚至无法发布消息。本节将对这个简单的系统进行扩展，增加一些对提供的参数的检查（例如检查`reply_to`是否是数字、是否提供了必要的参数）。你应该意识到，让这类系统健壮且对用户友好是一项艰巨的任务。如果打算使用这个系统，就应该在这些问题上做足工作。

组织Web程序的一种简单的方式是，对用户执行的每项操作都用一个脚本来实现。对于这个系统，这意味着需要以下脚本：
* main.cgi：显示所有消息的主题（层次方式），并链接到消息本身。
* view.cgi：显示一条消息，包括让用户能够回复的链接。
* edit.cgi：显示可编辑的表单，提交按钮链接到save脚本。
* save.cgi：从edit脚本接收消息的相关信息，并通过在数据库表中插入一行来保存这条消息。

下面来分别编写这些脚本。

### 26.5.1 编写main脚本
脚本main.cgi很像第一个原型中的simple_main.cgi，主要区别在于加入了链接。每个主题链接到对应的消息（链接到view.cgi），同时在页面底部添加了让用户能够发布新消息的链接（链接到edit.cgi）。

如代码清单26-5所示。下面这行打印每条消息的链接：

```python
print('<p><a href="view.cgi?id={id:d}">{subject}</a></p>'.format_map(row))
```

这将创建链接`view.cgi?id=xxx`。这种语法(`?key=val`)是向CGI脚本传递参数的方式。用户点击链接时，将使用正确的`id`参数跳转到view.cgi。 "Post message" 直接链接到edit.cgi。

[代码清单26-5 公告板主页](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch26/cgi-bin/main.cgi)

### 26.5.2 编写view脚本
脚本view.cgi使用提供的参数`id`从数据库获取一条消息，再使用得到的值来生成一个简单的HTML页面。这个页面包含一个返回主页(main.cgi)的链接，还包含一个到edit.cgi的链接，并将`reply_to`参数设置为`id`，以确保新消息是对当前消息的回复。脚本view.cgi的代码如代码清单26-6所示。

[代码清单26-6 消息查看器](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch26/cgi-bin/view.cgi)

警告：应该避免将不信任的文本直接插入用作SQL查询的字符串中，因为这样的代码很容易遭受SQL注入攻击。相反，应使用Python DB API占位符机制，并向`curs.execute()`提供一个额外的参数元组。详细信息参见 <https://bobby-tables.com/> 。

### 26.5.3 编写edit脚本
脚本edit.cgi实际上承担了双重职责：既用于编辑新消息，也用于编辑回复。二者的差别并不大：如果在CGI请求中提供了`reply_to`参数（表示编辑回复），就将其存储在表单中一个**隐藏的**输入框中。在Web表单中，隐藏的输入框用于临时存储信息。它们不展示给用户，但它们的值仍然被传递给表单的`action`属性指定的CGI脚本。另外，回复的主题默认设置为 "Re: parentsubject" （除非主题已经以 "Re:" 开头）。

代码清单26-7展示了edit.cgi脚本的源代码。

[代码清单26-7 消息编辑器](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch26/cgi-bin/edit.cgi)

### 26.5.4 编写save脚本
下面来编写最后一个脚本。脚本save.cgi（从edit.cgi生成的表单那里）接收一条消息的相关信息，并将其存储到数据库中。这意味着使用SQL `INSERT`命令，同时由于对数据库做了修改，必须调用`conn.commit()`，这样脚本终止时所做的改动才不会丢失。

代码清单26-8展示了save.cgi脚本的源代码。

[代码清单26-8 保存脚本](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch26/cgi-bin/save.cgi)

### 26.5.5 尝试使用
项目目录结构如下：

```
project/
    cgi-bin/
        main.cgi
        view.cgi
        edit.cgi
        save.cgi
    testdata/
        messages.sql
    create_table_sqlite.sql
    execute_sql.py
    messages.db    # 自动创建
```

运行步骤：

（1）创建数据库

```shell
$ python execute_sql.py messages.db create_table_sqlite.sql
```

（2）导入测试数据（可选）

```shell
$ python execute_sql.py messages.db testdata/messages.sql
```

（3）启动服务器

```shell
$ python -m http.server --cgi
```

（4）在Web浏览器中访问主页 <http://localhost:8000/cgi-bin/simple_main.cgi>

要发布消息，首先打开主页，点击 "Post message" 链接。在编辑页面填写表单，点击Save按钮，将显示消息 "Message Saved" 。点击 "Back to the main page" 链接回到主页，列表中应该包含刚才发布的消息。

要查看消息，只需点击其主题。在查看页面，点击 "Reply" 链接，将再次打开编辑页面，但设置了默认主题。同样，输入一些文本，点击Save按钮，回到主页。你的回复应该显示在原来主题的下方。

主页、消息查看器和消息编辑器页面如下图所示。

![主页](/assets/images/python-note-ch26-project-7-your-own-bulletin-board/主页.png)

![消息查看器](/assets/images/python-note-ch26-project-7-your-own-bulletin-board/消息查看器.png)

![消息编辑器](/assets/images/python-note-ch26-project-7-your-own-bulletin-board/消息编辑器.png)

## 26.6 进一步探索
现在你有能力使用可靠而高效的存储开发强大的大型Web应用了，但值得深入探究的方面还有很多。
* 可以为你喜欢的巨蟒剧团剧目数据库编写一个Web前端。
* 如果你想改进本章的系统，应该考虑抽象。可以创建一个实用模块，其中包含用于打印网页标准头部和尾部的函数，这样就无需在每个脚本中编写同样的HTML内容了。另外，添加一个能够处理密码的用户数据库，或者将创建连接的代码抽象出来或许比较有帮助。
* 如果你希望存储不需要专用的服务器，可以使用SQLite，也可以使用非SQL数据库（例如MongoDB (<https://www.mongodb.com/>)），还可以使用专用的文件格式（例如HDF5 (<https://www.h5py.org/>)）。
