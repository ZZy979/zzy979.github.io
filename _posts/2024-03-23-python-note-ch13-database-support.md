---
title: 《Python基础教程》笔记 第13章 数据库支持
date: 2024-03-23 13:26:34 +0800
categories: [Python, Beginning Python]
tags: [python, database, sqlite]
---
本章讨论Python数据库API（一种连接到SQL数据库的标准化方式），并演示如何使用该API来执行一些基本的SQL。

这里不会提供关系型数据库和SQL语言教程，可以阅读有关数据库（例如PostgreSQL、MySQL或本章使用的SQLite）的文档或其他教程（例如[SQLCourse](https://www.sqlcourse.com/)和[SQL Tutorial](https://www.w3schools.com/sql/)）。

本章使用的是简单数据库SQLite，但绝非唯一的选择。Python支持的数据库列表见[DatabaseInterfaces](https://wiki.python.org/moin/DatabaseInterfaces)。

本章的重点是低级数据库交互，但有一些高级库能够帮助你抽象复杂的细节（对象关系映射(object-relational mapper, ORM)），例如[SQLAlchemy](https://www.sqlalchemy.org/)和[SQLObject](https://sqlobject.org/)。

## 13.1 Python数据库API
前面提到，有各种SQL数据库可供选择，其中很多都有相应的Python客户端模块（有些数据库甚至有多个）。为了统一不同模块的接口，Python定义了标准数据库API (DB API)。这个API的当前版本(2.0)定义在[PEP 249](https://peps.python.org/pep-0249/)。

本节概述该API的基础。

### 13.1.1 全局变量
任何与DB API 2.0兼容的数据库模块都必须包含这三个全局变量，它们描述了模块特性。如果要让程序能够使用多种不同的数据库，比较现实的做法是检查这些变量，如果不能接受则显示合适的错误消息并退出。

| 变量名 | 描述 |
| --- | --- |
| `apilevel` | 使用的DB API版本 |
| `threadsafety` | 线程安全等级 |
| `paramstyle` | SQL查询中使用的参数（占位符）风格 |

### 13.1.2 异常
DB API定义了多种异常，让你能够细致地处理错误。

| 异常 | 超类 | 描述 |
| --- | --- | --- |
| `Warning` | `Exception` | 非致命错误 |
| `Error` | `Exception` | 所有错误的基类 |
| `InterfaceError` | `Error` | 与接口而不是数据库相关的错误 |
| `DatabaseError` | `Error` | 与数据库相关的错误 |
| `DataError` | `DatabaseError` | 与数据相关的问题，例如值超出范围 |
| `OperationalError` | `DatabaseError` | 数据库内部操作错误，例如意外断连 |
| `IntegrityError` | `DatabaseError` | 关系完整性被破坏，例如外键检查失败 |
| `InternalError` | `DatabaseError` | 数据库内部错误，例如游标无效 |
| `ProgrammingError` | `DatabaseError` | 用户编程错误，例如未找到表 |
| `NotSupportedError` | `DatabaseError` | 请求了不支持的特性，例如回滚 |

### 13.1.3 连接和游标
要使用底层数据库系统，必须先**连接**到它，为此使用函数`connect()`。该函数的参数取决于数据库。作为指南，DB API定义了下表所示的参数，推荐将这些参数作为关键字参数，并按表中的顺序排列。这些参数都应该是字符串。

| 参数名 | 描述 |
| --- | --- |
| `dsn` | 数据源名称（含义取决于数据库） |
| `user` | 用户名 |
| `password` | 密码 |
| `host` | 主机名 |
| `database` | 数据库名 |

注：该表格仅仅是“指南”，实际模块的定义取决于具体数据库。例如[sqlite3.connect()](https://docs.python.org/3/library/sqlite3.html#sqlite3.connect)、[pymysql.connect()](https://pymysql.readthedocs.io/en/latest/modules/connections.html)、[MySQLdb.connect()](https://mysqlclient.readthedocs.io/user_guide.html#functions-and-attributes)。

`connect()`函数返回一个**连接**(connection)对象，表示与数据库的当前会话。连接对象支持下表所示的方法。

| 方法名 | 描述 |
| --- | --- |
| `close()` | 关闭连接（之后连接对象及其游标将不可用） |
| `commit()` | 提交挂起的事务（如果不支持则什么都不做） |
| `rollback()` | 回滚挂起的事务（可能不可用） |
| `cursor()` | 返回连接的游标对象 |

`cursor()`方法返回**游标**(cursor)对象。游标用来执行SQL查询和查看结果。游标的方法如下表所示。

| 方法名 | 描述 |
| --- | --- |
| `callproc(name[, params])` | 调用数据库过程（可选） |
| `close()` | 关闭游标（之后游标不可用） |
| `execute(sql[, params])` | 执行SQL操作，可能带参数 |
| `executemany(sql, param_seq)` | 对序列中的每组参数执行SQL操作 |
| `fetchone()` | 获取结果集中的下一行，如果没有更多数据则返回`None` |
| `fetchmany([size])` | 获取结果集中的多行，默认大小为`arraysize`，如果没有更多数据则返回空序列 |
| `fetchall()` | 获取所有（剩余）的行 |
| `nextset()` | 跳到下一个可用的结果集（可选） |
| `setinputsizes(sizes)` | 用于为参数预定义内存区域 |
| `setoutputsize(size[, col])` | 为取回大量数据设置缓冲区大小 |

注：DB API的可选扩展还定义了`__iter__()`方法，从而游标对象是可迭代的。

游标的属性如下表所示。

| 属性名 | 描述 |
| --- | --- |
| `description` | 结果列的描述，只读 |
| `rowcount` | 结果中的行数，只读 |
| `arraysize` | `fetchmany()`返回的行数，默认为1 |

### 13.1.4 类型
对于插入到某些类型的列中的值，底层SQL数据库可能要求它们满足一定的条件。为了能够与底层SQL数据库正确地互操作，DB API定义了一些构造函数和常量（单例），用于提供特殊的类型和值，如下表所示。有些模块可能没有完全遵守。

| 名称 | 描述 |
| --- | --- |
| `Date(year, month, day)` | 创建包含日期值的对象 |
| `Time(hour, minute, second)` | 创建包含时间值的对象 |
| `Timestamp(year, month, day, hour, minute, second)` | 创建包含时间戳值的对象 |
| `DateFromTicks(ticks)` | 根据从纪元经过的秒数创建包含日期值的对象 |
| `TimeFromTicks(ticks)` | 根据从纪元经过的秒数创建包含时间值的对象 |
| `TimestampFromTicks(ticks)` | 根据从纪元经过的秒数创建包含时间戳值的对象 |
| `Binary(string)` | 创建包含二进制字符串值的对象 |
| `STRING` | 描述基于字符串的列（例如`CHAR`） |
| `BINARY` | 描述二进制列（例如`LONG`或`RAW`） |
| `NUMBER` | 描述数字列 |
| `DATETIME` | 描述日期/时间列 |
| `ROWID` | 描述行ID列 |

## 13.2 SQLite和PySQLite
为降低Python DB API的使用门槛，本章选择了一个名为SQLite的小型数据库引擎。它不需要作为独立的服务器运行，并且可以直接使用本地文件，而不需要集中式数据库存储机制。官方网站： <https://www.sqlite.org/> 。

在较新的Python版本中，标准库已经包含了一个SQLite包装——PySQLite，在`sqlite3`模块。

### 13.2.1 入门
要使用Python标准库中的SQLite，可以导入`sqlite3`模块。然后，就可以创建直接到数据库文件的连接，只需提供一个文件名（后缀为.db），如果不存在将自动创建。

```python
>>> import sqlite3
>>> conn = sqlite3.connect('somedatabase.db')
```

接下来可以从连接获得游标。

```python
>>> cursor = conn.cursor()
```

这个游标可用来执行SQL查询。如果修改了数据，务必提交所做的修改，这样才会真正保存到文件中。

```python
>>> conn.commit()
```

要关闭连接，使用`close()`方法。

```python
>>> conn.close()
```

注：另见[sqlite3](https://docs.python.org/3/library/sqlite3.html)模块 "Tutorial" 一节。

### 13.2.2 数据库应用程序示例
作为示例，本节将介绍如何创建一个小型的食品营养成分数据库。程序基于美国农业部(USDA)农业研究服务(<https://www.ars.usda.gov/>)提供的数据。在数据集[SR28](https://www.ars.usda.gov/northeast-area/beltsville-md-bhnrc/beltsville-human-nutrition-research-center/methods-and-application-of-food-composition-laboratory/mafcl-site-pages/sr11-sr28/)的页面上，下载文件[sr28abbr.zip](https://www.ars.usda.gov/ARSUserFiles/80400535/DATA/SR/sr28/dnload/sr28abbr.zip)。解压后，得到一个名为ABBREV.txt的文本文件和一个描述其内容的PDF文件。

在文件[ABBREV.txt](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch13/ABBREV.txt)中，每行都是一条数据记录，字段之间用脱字符(^)分隔。数字字段直接包含数字，而文本字段用波浪线(~)将字符串值“括起来”。下面是一个示例行。

```
~07278~^~HORMEL PILLOW ...~^47.84^243^30.99^ ... ^~1 serving~^^~~^0
```

将这样的行解析成字段很简单，只需使用`line.split('^')`。如果一个字段以波浪线开头，就知道它是一个字符串，可以使用`field.strip('~')`来获取其内容。对于数字字段，可以使用`float(field)`，除非字段为空。

本节接下来将开发一个程序，将这个文本文件中的数据转移到SQL数据库中，并让你能够执行一些有趣的查询。

#### 创建并填充数据库表
代码清单13-1所示的程序创建一个名为`food`的表和适当的字段，读取并解析文件ABBREV.txt，通过SQL `INSERT`语句将值插入数据库中。

[代码清单13-1 将数据导入数据库](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch13/importdata.py)

运行这个程序时（文件ABBREV.txt位于同一目录下），它将新建一个名为food.db的文件，其中包含数据库的所有数据。

#### 搜索并处理结果
使用数据库很简单：创建一个连接并从它获取一个游标，使用`execute()`方法执行SQL查询，并使用`fetchall()`等方法提取结果。代码清单13-2所示的程序通过命令行参数接受一个SQL `SELECT`语句的条件，并将返回的行打印出来。

[代码清单13-2 食品数据库查询程序](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch13/food_query.py)

可以在命令行中尝试运行这个程序：

```shell
$ python food_query.py "kcal <= 100 AND fiber >= 10 ORDER BY sugar" 
SELECT * FROM food WHERE kcal <= 100 AND fiber >= 10 ORDER BY sugar
id: 09216            
desc: ORANGE PEEL,RAW
water: 72.5          
kcal: 97.0           
protein: 1.5         
fat: 0.2
ash: 0.8
carbs: 25.0
fiber: 10.6
sugar: 0.0

id: 09156
desc: LEMON PEEL,RAW
water: 81.6
kcal: 47.0
protein: 1.5
fat: 0.3
ash: 0.6
carbs: 16.0
fiber: 10.6
sugar: 4.17

...
```

```shell
$ python food_query.py "id = '01105'" 
SELECT * FROM food WHERE id = '01105'
id: 01105
desc: MILK,CHOC BEV,HOT COCOA,HOMEMADE
water: 82.45
kcal: 77.0
protein: 3.52
fat: 2.34
ash: 0.65
carbs: 10.74
fiber: 1.0
sugar: 9.66
```

警告：这个程序从用户获取输入，并将其插入到SQL查询中。当用户不输入太不可思议的内容时，这没有问题。然而，利用这种输入偷偷地插入恶意SQL代码以破坏数据库是一种常见的计算机系统攻击方式，称为**SQL注入**。不要将你的数据库（或其他任何东西）暴露给原始用户输入。
