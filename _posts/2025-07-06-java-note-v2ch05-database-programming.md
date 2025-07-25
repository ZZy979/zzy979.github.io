---
title: 《Java核心技术》笔记 卷II 第5章 数据库编程
date: 2025-07-06 20:39:51 +0800
categories: [Java, Core Java]
tags: [java, database, jdbc, derby, transaction]
---
在本章中，将阐述JDBC背后的关键思想，并介绍SQL——关系型数据库业界标准的结构化查询语言。本章还将提供足够的细节和示例，使你能在常见的编程场景中使用JDBC。

注释：根据Oracle的声明，JDBC是一个注册了商标的术语，而并非Java Database Connectivity的首字母缩写。

## 5.1 JDBC的设计
Java设计者最初希望通过扩展Java使程序只使用“纯”Java就能与任何数据库进行通信。但是，他们很快就意识到这是一项不可能完成的任务：业界存在太多不同的数据库，使用的协议也各不相同。

所有数据库供应商都一致认为，如果Java能够为SQL访问提供一套纯Java API，同时提供一个驱动管理器以允许第三方驱动程序连接到特定的数据库，这将非常有用。数据库供应商可以提供自己的驱动程序，将其插入到驱动管理器。

这种组织方式遵循了微软公司非常成功的ODBC模式，ODBC为数据库访问提供了C语言接口。JDBC和ODBC都基于同一个思想：根据API编写的程序与驱动管理器通信，而驱动管理器使用驱动与实际的数据库通信。这意味着大多数程序员只需使用JDBC API即可。

### 5.1.1 JDBC驱动类型
JDBC规范将驱动分为以下几类：
* **第1类驱动**将JDBC翻译成ODBC，依赖ODBC驱动与数据库进行通信。
* **第2类驱动**由部分Java和部分本地代码编写，与数据库的客户端API通信。
* **第3类驱动**是纯Java客户端库，使用一种数据库无关的协议将数据库请求发送给服务器组件，该组件再将请求翻译成特定数据库的协议。
* **第4类驱动**是纯Java库，将JDBC请求直接翻译成特定数据库的协议。

注释：JDBC规范可以在 <https://jcp.org/aboutJava/communityprocess/mrel/jsr221/index3.html> 获得。

总之，JDBC的最终目标是：
* 程序员可以用Java语言编写应用程序，以使用标准SQL语句（甚至是SQL的专用扩展）访问任何数据库，同时仍然遵循Java语言的相关约定。
* 数据库和工具供应商可以提供底层驱动，这样就可以优化其特定数据库的驱动。

### 5.1.2 JDBC的典型用法
在传统的客户端/服务器模型中，客户端具有GUI，数据库在服务器，如下图所示。在这种模型中，JDBC驱动部署在客户端。

![传统的客户端-服务器应用](/assets/images/java-note-v2ch05-database-programming/传统的客户端-服务器应用.png)

如今，三层模型更加常见。客户端不直接调用数据库，而是调用服务器上的中间层（通常是通过HTTP），由中间层完成数据库查询（通过JDBC），如下图所示。这种模型的优点是将可视化表示（位于客户端）与业务逻辑（位于中间层）和原始数据（位于数据库）分离。因此，可以从不同的客户端（如Java桌面应用、浏览器或移动App）访问相同的数据和业务规则。

![三层结构的应用](/assets/images/java-note-v2ch05-database-programming/三层结构的应用.png)

## 5.2 结构化查询语言
**结构化查询语言**(Structured Query Language, SQL)是基本上所有现代关系型数据库的命令语言。JDBC使你可以使用SQL与数据库进行通信。可以将JDBC看作用于将SQL语句传递给数据库的API。

可以将（关系型）数据库想象成一组有行和列的**表**(table)。每一列都有列名，每一行都由多个列（**字段**）组成。

下面考虑描述经典计算机书籍的数据库，由以下四张表组成：作者、图书、图书作者和出版社。

`Authors`表

| Author_ID | Name | Fname |
| --- | --- | --- |
| ALEX | Alexander | Christopher |
| BROO | Brooks | Frederick P. |
| ... | ... | ... |

`Books`表

| Title | ISBN | Publisher_ID | Price |
| --- | --- | --- | --- |
| A Guide to the SQL Standard | 0-201-96426-0 | 0201 | 47.95 |
| A Pattern Language: Towns, Buildings, Construction | 0-19-501919-9 | 019 | 65.00 |
| ... | ... | ... | ... |

`BooksAuthors`表

| ISBN | Author_ID | Seq_No |
| --- | --- | --- |
| 0-201-96426-0 | DATE | 1 |
| 0-201-96426-0 | DARW | 2 |
| 0-19-501919-9 | ALEX | 1 |
| ... | ... | ... |

`Publishers`表

| Publisher_ID | Name | URL |
| --- | --- | --- |
| 0201 | Addison-Wesley | www.aw-bc.com |
| 0407 | John Wiley & Sons | www.wiley.com |
| 019 | Oxford University Press | www.oup.co.uk |
| ... | ... | ... |

本节将介绍如何编写查询语句。按照惯例，SQL关键字全部使用大写字母，但不是必需的。

可以使用下面的`SELECT`语句查询`Books`表中的所有行（的所有列）：

```sql
SELECT * FROM Books
```

在`SELECT`语句中，`FROM`子句是必需的，用于指定要查询的表名。

可以选择所需要的列：

```sql
SELECT ISBN, Price, Title
FROM Books
```

可以使用`WHERE`子句来过滤行：

```sql
SELECT ISBN, Price, Title
FROM Books
WHERE Price <= 29.95
```

注意，与Java语言不同，SQL使用`=`和`<>`而不是`==`和`!=`。

`WHERE`子句也可以使用`LIKE`运算符进行模糊查询，`%`表示0个或多个字符，`_`表示单个字符（而不是通常的`*`和`?`）。例如：

```sql
SELECT ISBN, Price, Title
FROM Books
WHERE Title NOT LIKE '%n_x%'
```

查询书名不包含诸如Unix或Linux等单词的图书。

注意，字符串用单引号括起来，而不是双引号。字符串中的单引号则需要用一对单引号表示。例如，

```sql
SELECT Title
FROM Books
WHERE Title LIKE '%''%'
```

查询所有包含单引号的书名。

可以从多个表查询数据（称为**连接**(join)）：

```sql
SELECT * FROM Books, Publishers
```

如果没有`WHERE`子句，这个查询的意义就不大，它会罗列两个表中记录的**所有组合**（笛卡尔积）。而我们只对图书与出版社相匹配的数据感兴趣：

```sql
SELECT * FROM Books, Publishers
WHERE Books.Publisher_Id = Publishers.Publisher_Id
```

当查询涉及多个表时，相同的列名可能会出现在两个不同的表中。当出现歧义时，可以在列名前添加它所属的表名作为前缀，例如`Books.Publisher_Id`。

可以使用`UPDATE`语句来修改数据库中的数据。例如，假设要将所有书名包含 "C++" 的图书降价$5：

```sql
UPDATE Books
SET Price = Price - 5.00
WHERE Title LIKE '%C++%'
```

要删除数据，使用`DELETE`语句：

```sql
DELETE FROM Books
WHERE Title LIKE '%C++%'
```

`GROUP BY`子句和聚合函数（如求和、平均值、最大/最小值等）不在此讨论。

可以使用`INSERT`语句向表中插入数据：

```sql
INSERT INTO Books
VALUES ('A Guide to the SQL Standard', '0-201-96426-0', '0201', 47.95)
```

在查询、修改和插入数据之前，必需先创建表。使用`CREATE TABLE`语句创建一张新表，并指定每一列的名字和数据类型。例如：

```sql
CREATE TABLE Books (
  Title CHAR(60),
  ISBN CHAR(13),
  Publisher_Id CHAR(6),
  Price DECIMAL(10,2)
)
```

下表展示了最常见的SQL数据类型。

| 数据类型 | 描述 |
| --- | --- |
| `INTEGER`, `INT` | 32位整数 |
| `SMALLINT` | 16位整数 |
| `NUMERIC(m,n)`, `DECIMAL(m,n)`, `DEC(m,n)` | 定点十进制数，共m位数字，小数点后n位数字 |
| `FLOAT(n)` | 精度为n位二进制数的浮点数 |
| `REAL` | 32位浮点数 |
| `DOUBLE` | 64位浮点数 |
| `CHARACTER(n)`, `CHAR(n)` | 长度固定为n的字符串 |
| `VARCHAR(n)` | 最大长度为n的变长字符串 |
| `BOOLEAN` | 布尔值 |
| `DATE` | 日期 |
| `TIME` | 时间 |
| `TIMESTAMP` | 时间戳 |
| `BLOB` | 二进制大对象 |
| `CLOB` | 字符大对象 |

## 5.3 JDBC配置
你需要有一个可以获得其JDBC驱动的数据库。有很多不错的选择，例如[IBM DB2](https://www.ibm.com/products/db2)、[Microsoft SQL Server](https://www.microsoft.com/en-us/sql-server/)、[MySQL](https://www.mysql.com/)、[Oracle](https://www.oracle.com/database/)和[PostgreSQL](https://www.postgresql.org/)。

你还需要创建一个数据库用于实验，假设将其命名为`COREJAVA`。

如果是第一次接触数据库，建议使用Apache Derby，可以从 <https://db.apache.org/derby/derby_downloads.html> 下载。

注：
* 对于Java 17可选择[10.16.1.1版本](https://db.apache.org/derby/releases/release-10_16_1_1.cgi)，从下载页面下载文件db-derby-10.16.1.1-lib.zip（或.tar.gz）。
* 解压目录中的lib目录包含Derby的所有JAR文件。每个JAR都是一个模块（参见卷II第9章），对应关系如下表所示：

| JAR | 模块 |
| --- | --- |
| derby.jar | `org.apache.derby.engine` |
| derbyclient.jar | `org.apache.derby.client` |
| derbynet.jar | `org.apache.derby.server` |
| derbyoptionaltools.jar | `org.apache.derby.optionaltools` |
| derbyrun.jar | `org.apache.derby.runner` |
| derbyshared.jar | `org.apache.derby.commons` |
| derbytools.jar | `org.apache.derby.tools` |

依赖关系如下图所示（模块名省略了前缀`org.apache.derby`）。

![Derby模块依赖关系](/assets/images/java-note-v2ch05-database-programming/Derby模块依赖关系.png)

### 5.3.1 数据库URL
在连接到数据库时，必需使用各种与数据库相关的参数，例如主机名、端口号、用户名、密码和数据库名等。

JDBC使用一种与普通URL类似的语法来描述数据源，一般语法为：

```
jdbc:subprotocol:parameters
```

其中`subprotocol`用于选择具体驱动，`parameters`的格式取决于所使用的子协议。

例如：

```
jdbc:derby://localhost:1527/COREJAVA;create=true
jdbc:postgresql:COREJAVA
```

上面的JDBC URL分别指定了名为`COREJAVA`的Derby数据库和PostgreSQL数据库。

### 5.3.2 驱动JAR文件
你需要获得你所使用的数据库的驱动JAR文件。如果使用Derby，则需要derbyclient.jar。对于其他数据库，需要寻找适当的驱动。例如，PostgreSQL使用[pgJDBC](https://jdbc.postgresql.org/)，MySQL使用[Connector/J](https://dev.mysql.com/downloads/connector/j/)。另外也可以在Maven上搜索[JDBC Drivers](https://mvnrepository.com/open-source/jdbc-drivers)。

在运行访问数据库的程序时，需要将驱动JAR文件添加到类路径（编译时不需要）。

```shell
java -classpath driverPath:. ProgramName
```

### 5.3.3 启动数据库
在连接之前需要先启动数据库服务器，细节取决于所使用的数据库。

对于Derby数据库，步骤如下：

1.打开一个shell窗口，并切换到要存放数据库文件的目录。

2.找到文件derbyrun.jar，它位于Derby解压目录（下面用$derby表示）中的lib目录。

3.运行命令

```shell
java -jar $derby/lib/derbyrun.jar server start
```

服务器进程会一直运行，直到通过`shutdown`命令手动关闭。

4.创建一个文件ij.properties，内容如下：

```properties
ij.driver=org.apache.derby.jdbc.ClientDriver
ij.protocol=jdbc:derby://localhost:1527/
ij.database=COREJAVA;create=true
```

在另一个shell窗口中，执行以下命令来运行Derby的交互式脚本工具（称为`ij`）：

```shell
java -jar $derby/lib/derbyrun.jar ij -p ij.properties
```

现在可以执行SQL命令，例如：

```sql
ij> CREATE TABLE Greetings (Message CHAR(20));
0 rows inserted/updated/deleted
ij> INSERT INTO Greetings VALUES ('Hello, World!');
1 row inserted/updated/deleted
ij> SELECT * FROM Greetings;
MESSAGE
--------------------
Hello, World!

1 row selected
ij> DROP TABLE Greetings;
0 rows inserted/updated/deleted
ij> EXIT;
```

输入`EXIT;`退出。

5.使用完数据库后，用下面的命令关闭服务器：

```shell
java -jar $derby/lib/derbyrun.jar server shutdown
```

### 5.3.4 注册驱动类
许多JDBC JAR文件（如Derby驱动）会自动注册驱动类，在这种情况下可以跳过本节的手动注册步骤。如果JAR文件包含META-INF/services/java.sql.Driver文件，就可以自动注册驱动类（参见卷I第6章 6.4节）。可以解压缩JAR文件（或使用`jar -tf`命令）来检查。

如果驱动JAR文件不支持自动注册，就需要找出你使用的JDBC驱动类名（如`org.postgresql.Driver`）。

使用`DriverManager`注册驱动有两种方式。一种方式是在Java程序中加载驱动类，例如：

```java
Class.forName("org.postgresql.Driver"); // force loading of driver class
```

这条语句将导致驱动类被加载，从而执行注册驱动的静态初始化器。

或者使用命令行参数来指定`jdbc.drivers`属性，例如：

```shell
java -Djdbc.drivers=org.postgresql.Driver ProgramName
```

也可以通过以下调用设置系统属性：

```java
System.setProperty("jdbc.drivers", "org.postgresql.Driver");
```

还可以提供多个驱动（例如在一个程序中同时连接PostgreSQL和Derby数据库），用冒号分隔，例如：

```
org.postgresql.Driver:org.apache.derby.jdbc.ClientDriver
```

### 5.3.5 连接到数据库
在Java程序中，可以像这样打开一个数据库连接：

```java
String url = "jdbc:postgresql:COREJAVA";
String username = "dbuser";
String password = "secret";
Connection conn = DriverManager.getConnection(url, username, password);
```

驱动管理器会遍历已注册的驱动，以查找一个支持URL中指定的子协议的驱动。

注释：默认情况下，Derby允许使用任意用户名连接，并且不检查密码。它会为每个用户生成一组独立的表。默认用户名是app。

程序清单5-1给出了完整的测试程序。它从名为database.properties的文件中加载连接参数并连接到数据库。

[程序清单5-1 test/TestDB.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch05/test/TestDB.java)

要运行这个测试程序，先按照前面描述的方式启动数据库，然后像这样启动程序：

```shell
java -classpath .:$derby/lib/derbyclient.jar test.TestDB
```

注：
* Derby的数据文件存储在**系统目录**下与数据库同名的目录中，系统目录默认为当前工作目录，可以通过系统属性`derby.system.home`指定。详见文档[Derby Developer's Guide](https://db.apache.org/derby/docs/10.11/devguide/index.html) "Defining the system directory" 和 "The database directory" 两节。
* Derby支持**嵌入式驱动**，数据库引擎不是运行在单独的进程中，而是在与应用程序相同的JVM中，因此无需单独启动和关闭。详见官方教程[Embedded Derby](https://db.apache.org/derby/papers/DerbyTut/embedded_intro.html)。为了在这个示例程序中使用嵌入式驱动，需要
1. 使用驱动类`org.apache.derby.jdbc.EmbeddedDriver`。
2. 在URL中省略主机和端口号。
3. 将derby.jar和derbytools.jar添加到类路径，而不需要derbyclient.jar。

最终的database.properties文件如下：

```properties
jdbc.drivers=org.apache.derby.jdbc.EmbeddedDriver
jdbc.url=jdbc:derby:COREJAVA;create=true
jdbc.username=dbuser
jdbc.password=secret
```

通过以下命令运行程序：

```shell
java -classpath .:$derby/lib/derby.jar:$derby/lib/derbytools.jar test.TestDB
```

提示：调试JDBC相关问题的一种方法是启用JDBC跟踪。调用`DriverManager.setLogWriter()`方法将跟踪信息发送给一个`PrintWriter`，输出将包含JDBC活动的详细列表。大多数JDBC驱动实现都提供了额外的跟踪机制。例如，对于Derby，可以在URL中添加`traceFile`选项：`jdbc:derby://localhost:1527/COREJAVA;create=true;traceFile=trace.out`。

## 5.4 使用JDBC语句
### 5.4.1 执行SQL语句
为了执行SQL语句，首先需要创建一个`Statement`对象：

```java
Statement stat = conn.createStatement();
```

接着，把要执行的语句放到字符串中，例如：

```java
String command = "UPDATE Books"
    + " SET Price = Price - 5.00"
    + " WHERE Title NOT LIKE '%Introduction%'";
```

然后调用`Statement`接口的`executeUpdate()`方法：

```java
stat.executeUpdate(command);
```

该方法返回SQL语句影响的行数。

`executeUpdate()`方法既可以执行诸如`INSERT`、`UPDATE`和`DELETE`之类的数据修改操作，也可以执行诸如`CREATE TABLE`和`DROP TABLE`之类的数据定义语句。但是，执行`SELECT`查询必须使用`executeQuery()`方法。另外还有一个`execute()`方法可以执行任意的SQL语句，通常只用于用户提供的交互式查询。

为了获取查询结果，使用`executeQuery()`方法返回的`ResultSet`对象遍历结果，每次一行。

```java
ResultSet rs = stat.executeQuery("SELECT * FROM Books");
```

分析结果集的基本循环如下：

```java
while (rs.next()) {
    // look at a row of the result set
}
```

警告：`ResultSet`接口与`Iterator`接口稍有不同。`ResultSet`的迭代器被初始化为在第一行**之前**的位置，必须先调用一次`next()`方法将它移动到第一行。另外，它没有`hasNext()`方法，需要不断地调用`next()`，直到返回`false`。

结果集中行的顺序是完全随机的，除非使用`ORDER BY`子句指定排序。

查看每一行时，可以使用访问器方法获取字段的内容：

```java
String isbn = rs.getString(1);
double price = rs.getDouble("Price");
```

不同类型有不同的访问器`getXxx()`。每个访问器都有两种形式，一种接受整型参数（列号），另一种接受字符串参数（列名）。例如，`rs.getString(1)`返回当前行中第一列的字符串值，`rs.getDouble("Price")`返回名为`Price`的列的浮点值。使用列号更高效一些，但是使用列名可以使代码更易于阅读和维护。

警告：与数组索引不同，数据库的列号从1开始。

当访问器方法的类型和列的类型不一致时，会进行合理的类型转换。例如，调用`rs.getString("Price")`会将`Price`列的浮点值转换成字符串。

### 5.4.2 管理连接、语句和结果集
每个`Connection`对象可以创建一个或多个`Statement`对象。同一个语句对象可以用于多个不相关的查询。但是，一个语句最多只能有一个打开的结果集。如果需要执行多个查询，且同时分析结果，则需要多个语句对象。每个连接的语句数量是有限的。使用`conn.getMetaData().getMaxStatements()`获取JDBC驱动支持同时打开的语句数量。

在实际中，通常不需要多个结果集。让数据库执行组合查询（如join）并分析单个结果集比Java程序遍历多个结果集要高效得多。

确保在一个语句对象上触发新的查询之前处理完之前的结果集，因为之前查询的结果集会被自动关闭。使用完`ResultSet`、`Statement`或`Connection`对象后，最好立即调用`close()`方法，因为这些对象都使用了大型数据结构，会占用数据库服务器有限的资源。`Statement`对象的`close()`方法会关闭关联的结果集，`Connection`对象的`close()`方法会关闭连接的所有语句。反过来，可以在`Statement`上调用`closeOnCompletion()`方法，一旦其所有结果集都被关闭，该语句就会自动关闭。

为了确保连接对象不会保持打开状态，可以使用带资源的`try`语句：

```java
try (Connection conn = ...) {
    Statement stat = conn.createStatement();
    ResultSet result = stat.executeQuery(queryString);
    // process query result
}
```

### 5.4.3 分析SQL异常
每个`SQLException`都有一个异常链，可以通过`getNextException()`方法获取。这个异常链与每个异常都有的“原因”链不同（参见卷I第7章 7.2.3节）。`SQLException`实现了`Iterable<Throwable>`接口，其`iterator()`方法返回一个遍历这两个异常链的迭代器。可以直接使用for each循环：

```java
for (Throwable t : sqlException) {
    // do something with t
}
```

`SQLException`的`getSQLState()`方法返回符合X/Open或SQL:2003标准的字符串，`getErrorCode()`返回与具体数据库相关的错误码。

SQL异常的继承层次结构参见API文档。

另外，数据库驱动可能将非致命问题报告为警告。`SQLWarning`是`SQLException`的子类，可以调用上述两个方法来获得有关警告的更多信息。与SQL异常类似，警告也是链式的。要获取所有警告，使用以下循环：

```java
SQLWarning w = stat.getWarning();
while (w != null) {
    // do something with w
    w = w.nextWarning();
}
```

### 5.4.4 填充数据库
目前数据库中还没有数据，因此需要填充数据库。一种简单的方法是使用一组SQL语句来创建表并插入数据，语句之间用`;`分隔，保存在.sql文件中。例如：

```sql
CREATE TABLE Publishers (Publisher_Id CHAR(6), Name CHAR(30), URL CHAR(80));
INSERT INTO Publishers VALUES ('0201', 'Addison-Wesley', 'www.awbc.com');
INSERT INTO Publishers VALUES ('0471', 'John Wiley & Sons', 'www.wiley.com');
...
```

程序清单5-2是读取并执行SQL文件的程序代码。运行本章剩余的示例之前，使用该程序填充数据库。确认数据库服务器正在运行，然后像这样运行该程序：

```shell
java -classpath driverPath:. exec.ExecSQL Books.sql
java -classpath driverPath:. exec.ExecSQL Authors.sql
java -classpath driverPath:. exec.ExecSQL Publishers.sql
java -classpath driverPath:. exec.ExecSQL BooksAuthors.sql
```

[程序清单5-2 exec/ExecSQL.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch05/exec/ExecSQL.java)

注释：数据库可能也包含直接读取SQL文件的工具。例如，对于Derby，可以运行

```shell
java -jar $derby/lib/derbyrun.jar ij -p ij.properties Books.sql
```

## 5.5 执行查询
在本节中，将编写一个对`COREJAVA`数据库执行查询的程序。

### 5.5.1 预备语句
在这个程序中使用了**预备语句**(prepared statement)。考虑下面按出版社名称查询图书的SQL：

```sql
SELECT Books.Price, Books.Title
FROM Books, Publishers
WHERE Books.Publisher_Id = Publishers.Publisher_Id AND Publishers.Name = publisherName
```

我们可以**预备**一个带有宿主变量（占位符，如`publisherName`）的查询语句并多次使用，每次为该变量填入不同的字符串，而不是每次都构建一个单独的语句。这种技术提升了性能（数据库只进行一次查询计划）。

在预备语句中，每个变量都用`?`表示。例如：

```java
String publisherQuery = "SELECT Books.Price, Books.Title"
    + " FROM Books, Publishers"
    + " WHERE Books.Publisher_Id = Publishers.Publisher_Id AND Publishers.Name = ?";
PreparedStatement stat = conn.prepareStatement(publisherQuery);
```

在执行预备语句之前，必须先使用`setXxx()`方法将变量绑定到实际值，不同类型有不同的`set`方法。例如：

```java
stat.setString(1, publisher);
```

第一个参数是变量的位置（从1开始），第二个参数是赋予变量的值。

如果复用一个已经执行过的预备语句，所有变量绑定的值都不会改变，除非使用`set`方法改变或调用`clearParameters()`方法。这意味着下一次查询时只需要设置改变的变量即可。

一旦所有变量都绑定了值，就可以执行预备语句了：

```java
ResultSet rs = stat.executeQuery();
```

提示：通过拼接字符串手动构建查询非常繁琐（必须转义特殊字符），而且存在潜在的危险（SQL注入攻击）。因此，只要查询涉及变量就应该使用预备语句。

`UPDATE`语句应该调用`executeUpdate()`方法，因为它不返回结果集。该方法的返回值是修改的行数。

注释：很多数据库会自动缓存预备语句。因此，不必担心调用`prepareStatement()`的开销。

示例程序支持四种操作：
* 按作者姓名、出版社名称或二者一起查询图书。第三种查询需要连接4张表：

```sql
SELECT Books.Price, Books.Title
FROM Books, BooksAuthors, Authors, Publishers
WHERE Authors.Author_Id = BooksAuthors.Author_Id AND BooksAuthors.ISBN = Books.ISBN
AND Books.Publisher_Id = Publishers.Publisher_Id AND Authors.Name = ?
AND Publishers.Name = ?
```

* 按出版社名称修改图书价格。需要使用嵌套子查询获得出版社名称对应的id：

```sql
UPDATE Books
SET Price = Price + ?
WHERE Books.Publisher_Id = (SELECT Publisher_Id FROM Publishers WHERE Name = ?)
```

程序清单5-3给出了完整的程序代码。

[程序清单5-3 query/QueryTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch05/query/QueryTest.java)

### 5.5.2 读写LOB
除了数字、字符串和日期，很多数据库还可以存储**大对象**(large object)，例如图片。在SQL中，二进制大对象称为BLOB，字符型大对象称为CLOB。

要读取LOB，需要在`ResultSet`上调用`getBlob()`或`getClob()`方法，得到`Blob`或`Clob`类型的对象。要从`Blob`获得二进制数据，调用`getBytes()`或`getBinaryStream()`。例如，如果有一张保存图书封面图的表，可以像这样获取图像：

```java
PreparedStatement stat = conn.prepareStatement("SELECT Cover FROM BookCovers WHERE ISBN=?");
...
stat.set(1, isbn);
try (ResultSet result = stat.executeQuery()) {
    if (result.next()) {
        Blob coverBlob = result.getBlob(1);
        Image coverImage = ImageIO.read(coverBlob.getBinaryStream());
    }
}
```

对于`Clob`对象，可以通过调用`getSubString()`或`getCharacterStream()`方法获取字符数据。

要将LOB存入数据库，需要在`Connection`对象上调用`createBlob()`或`createClob()`，然后获得一个LOB的输出流或writer，写出数据，并将该对象存储到数据库。例如，可以像这样存储一张图像：

```java
Blob coverBlob = connection.createBlob();
int offset = 0;
OutputStream out = coverBlob.setBinaryStream(offset);
ImageIO.write(coverImage, "PNG", out);
PreparedStatement stat = conn.prepareStatement("INSERT INTO Cover VALUES (?, ?)");
stat.set(1, isbn);
stat.set(2, coverBlob);
stat.executeUpdate();
```

### 5.5.3 SQL转义
数据库通常支持“转义”语法，但使用与数据库相关的语法变体。

转义主要用于日期时间字面值、调用标量函数、调用存储过程、外连接和`LIKE`子句中的转义字符。

日期或时间字面值使用[ISO 8601](https://www.cl.cam.ac.uk/~mgk25/iso-time.html)格式指定，驱动会将其翻译成特定数据库的格式。使用`d`、`t`、`ts`来表示`DATE`、`TIME`和`TIMESTAMP`值：

```sql
{d '2008-01-24'}
{t '23:59:59'}
{ts '2008-01-24 23:59:59.999'}
```

**标量函数**(scalar function)是返回单个值的函数。许多函数在数据库中广泛使用，但名称存在差异。JDBC规范提供了标准名称，驱动将其翻译为特定数据库的名称。要调用函数，像这样使用`fn`指定标准函数名和参数：

```sql
{fn left(?, 20)}
{fn user()}
```

在[JDBC规范](https://download.oracle.com/otn-pub/jcp/jdbc-4_3-mrel3-eval-spec/jdbc4.3-fr-spec.pdf)附录C可以找到支持的函数名的完整列表。

两个表的**外连接**(outer join)不要求每个表中的所有行都根据连接条件进行匹配。例如，查询

```sql
SELECT * FROM {oj Books LEFT OUTER JOIN Publishers ON Books.Publisher_Id = Publisher.Publisher_Id}
```

结果中包含在`Publishers`表中没有匹配的图书，其`Publisher_Id`为`NULL`。使用`RIGHT OUTER JOIN`将包含没有匹配图书的出版社，而使用`FULL OUTER JOIN`二者都包含。由于并非所有数据库对于这些连接都使用标准写法，因此需要转义语法。

字符`_`和`%`在`LIKE`子句中具有特殊含义，用于匹配单个字符或字符序列。没有标准方式按字面意思使用它们。如果想要查询所有包含下划线的字符串，需要使用`escape`：

```sql
WHERE Title LIKE '%!_%' {escape '!'}
```

这里将`!`定义为转义字符，`!_`组合表示字面上的下划线（类似于Java字符串常量中`\"`表示双引号）。

**存储过程**(stored procedure)是在数据库中执行的用特定数据库的语言编写的过程。要调用存储过程，需要使用`call`。如果存储过程没有参数则不需要加括号，使用`=`捕获返回值。

```sql
{call PROC1(?, ?)}
{call PROC2}
{call ? = PROC3(?)}
```

在Java中需要使用`CallableStatement`接口执行存储过程，设置所有输入参数，并指定输出的类型。

```java
CallableStatement stat = conn.prepareCall("{call PROC4(?, ?)}")
stat.setInt(1, id);
stat.registerOutParameter(2, java.sql.Types.VARCHAR);
stat.execute();
String name = stat.getString(2);
```

### 5.5.4 多结果集
一个查询有可能会返回多个结果集（例如执行存储过程，或者使用允许在单个查询中提交多个`SELECT`语句的数据库）。下面是获取所有结果集的方法：
1. 使用`execute()`方法执行SQL语句。
2. 获取第一个结果集或更新计数。
3. 反复调用`getMoreResults()`方法移动到下一个结果集，直到没有更多结果集。

```java
boolean isResult = stat.execute(command);
boolean done = false;
while (!done) {
    if (isResult) {
        ResultSet result = stat.getResultSet();
        // do something with result
    }
    else {
        int updateCount = stat.getUpdateCount();
        if (updateCount >= 0)
            // do something with updateCount
        else
            done = true;
    }
    if (!done) isResult = stat.getMoreResults();
}
```

### 5.5.5 获取自动生成的键
大多数数据库都支持某种对行自动编号的机制，但是不同数据库的机制差异很大。这些自动编号通常用作主键。尽管JDBC没有提供生成键的方案，但它提供了获取自动生成键的方式。当你向表中插入一个新行并自动生成了键时，可以用以下代码获取这个键：

```java
stat.executeUpdate(insertStatement, Statement.RETURN_GENERATED_KEYS);
ResultSet rs = stat.getGeneratedKeys();
if (rs.next()) {
    int key = rs.getInt(1);
    ...
}
```

## 5.6 可滚动和可更新结果集
`ResultSet`接口的`next()`方法可以遍历结果集中的行，这对于只需要分析数据的程序已经足够了。但是，对于可视化数据显示，你通常希望用户能够在结果集中前后移动并编辑内容（如下图所示）。这就需要**可滚动**(scrollable)结果集和**可更新**(updatable)结果集。下面几节将讨论这些功能。

![可视化数据显示](/assets/images/java-note-v2ch05-database-programming/可视化数据显示.png)

### 5.6.1 可滚动结果集
默认情况下，结果集是不可滚动、不可更新的。为了从查询获得可滚动结果集，必须使用以下方法获得一个不同的语句对象：

```java
Statement stat = conn.createStatement(type, concurrency);
```

对于预备语句，使用

```java
PreparedStatement stat = conn.prepareStatement(command, type, concurrency);
```

结果集类型`type`可能的值如下表所示：

| 值 | 解释 |
| --- | --- |
| `TYPE_FORWARD_ONLY` | 结果集不可滚动（默认） |
| `TYPE_SCROLL_INSENSITIVE` | 结果集可滚动，但对数据库改变不敏感 |
| `TYPE_SCROLL_SENSITIVE` | 结果集可滚动，且对数据库改变敏感 |

并发类型`concurrency`可能的值如下表所示：

| 值 | 解释 |
| --- | --- |
| `CONCUR_READ_ONLY` | 结果集不能用于更新数据库（默认） |
| `CONCUR_UPDATABLE` | 结果集可用于更新数据库 |

例如，如果只想滚动遍历结果集，而不想编辑其数据，则使用

```java
Statement stat = conn.createStatement(ResultSet.TYPE_SCROLL_INSENSITIVE, ResultSet.CONCUR_READ_ONLY);
```

现在，调用`executeQuery()`返回的所有结果集都是可滚动的。可滚动结果集有一个指示当前位置的**游标**(cursor)。

注释：并非所有的数据库驱动都支持可滚动和可更新结果集（可通过`DatabaseMetaData`接口的`supportsResultSetType()`和`supportsResultSetConcurrency()`方法获知特定的驱动支持哪些类型和并发模式）。即使数据库支持所有模式，某个特定的查询也可能无法生成具有所有要求的属性的结果集（例如，复杂查询的结果集可能是不可更新的）。在这种情况下，`executeQuery()`方法会将一个`SQLWarning`添加到连接对象（参见5.4.3节）。也可以使用`ResultSet`接口的`getType()`和`getConcurrency()`方法得到结果集实际支持的模式。如果触发了一个不支持的操作（比如对不可滚动的结果集调用`previous()`方法），操作将抛出`SQLException`。

滚动结果集非常简单。使用`previous()`向后滚动（上一行）。如果游标位于实际的行，则该方法返回`true`；如果位于第一行之前则返回`false`。

可以使用`relative(n)`将游标向前或向后移动任意行。如果`n`为正数则向前移动，如果`n`为负数则向后移动，如果`n`为0则调用无效。如果试图将游标移动到结果集之外，则游标会被设置在最后一行之后或第一行之前（取决于`n`的符号），此时该方法返回`false`。如果游标位于实际的行，则该方法返回`true`。

注：`next()`等价于`relative(1)`，`previous()`等价于`relative(-1)`。

![可滚动结果集的游标](/assets/images/java-note-v2ch05-database-programming/可滚动结果集的游标.png)

或者，可以调用`absolute(n)`将游标设置到特定的行号上。

要得到当前行号，调用

```java
int currentRow = rs.getRow();
```

结果集中第一行的行号为1。如果返回值为0，那么游标当前不在任何行上——要么位于第一行之前，要么位于最后一行之后。

便捷方法`first()`、`last()`、`beforeFirst()`和`afterLast()`分别将游标移动到第一行、最后一行、第一行之前和最后一行之后。`isFirst()`、`isLast()`、`isBeforeFirst()`和`isAfterLast()`方法测试游标是否位于这些特殊位置。

### 5.6.2 可更新结果集
如果希望编辑结果集中的数据，并且将改变自动保存到数据库，就需要使用可更新结果集。可更新结果集不一定是可滚动的，但如果将数据展示给用户去编辑，那么通常也希望是可滚动的。

要获得可更新结果集，需要像这样创建语句：

```java
Statement stat = conn.createStatement(ResultSet.TYPE_SCROLL_INSENSITIVE, ResultSet.CONCUR_UPDATABLE);
```

这样，调用`executeQuery()`返回的结果集就是可更新的。

注释：并非所有查询都会返回可更新结果集。如果查询涉及多表连接，结果就可能是不可更新的。但如果查询只涉及一个表，或者按主键连接多个表，结果集就应当是可更新的。可以调用`ResultSet`接口的`getConcurrency()`方法来确定。

例如，假设想提高某些图书的价格，但没有可用于`UPDATE`语句的简单标准，那么可以遍历所有图书并根据任意的条件更新价格。

```java
String query = "SELECT * FROM Books";
ResultSet rs = stat.executeQuery(query);
while (rs.next()) {
    if (...) {
        double increase = ...;
        double price = rs.getDouble("Price");
        rs.updateDouble("Price", price + increase);
        rs.updateRow(); // make sure to call updateRow after updating fields
    }
}
```

每种类型都有对应的`updateXxx()`方法，指定列名或列号（与`getXxx()`方法一样），然后是字段的新值。

注释：`updateXxx()`方法的第一个参数指的是结果集中的列号，而不是数据库中的列号。

`updateXxx()`方法只改变结果集中行的值，而非数据库中的值。当更新完行中的字段后，必须调用`updateRow()`方法将更新发送到数据库。如果没有调用`updateRow()`就将游标移动到其他行，那么对该行的更新将被丢弃。还可以调用`cancelRowUpdates()`方法来取消对当前行的更新。

前面的例子展示了如何更新现有的行。如果想向数据库添加一个新行，首先使用`moveToInsertRow()`方法将游标移动到一个特殊位置，称为**插入行**(insert row)。然后调用`updateXxx()`方法构建新行。最后调用`insertRow()`方法将新行发送到数据库。完成插入操作后，调用`moveToCurrentRow()`将游标移动回之前的位置。下面是一个示例：

```java
rs.moveToInsertRow();
rs.updateString("Title", title);
rs.updateString("ISBN", isbn);
rs.updateString("Publisher_Id", pubid);
rs.updateDouble("Price", price);
rs.insertRow();
rs.moveToCurrentRow();
```

在插入行中没有指定的列将被设置为`NULL`。但是，如果这个列有`NOT NULL`约束，那么会抛出异常，这一行也不会插入。

最后，可以使用`deleteRow()`方法删除游标指向的行。该方法会立即将行从结果集和数据库中删除。

警告：如果不小心，就可能会用可更新结果集编写出效率极低的代码。执行`UPDATE`语句要比一边遍历一边修改数据高效得多。

## 5.7 行集
可滚动结果集虽然功能强大，但有一个重要的缺陷：在整个用户交互过程中必须与数据库保持连接。这可能会长时间占用数据库连接资源。在这种情况下，可以使用**行集**(row set)。`RowSet`接口扩展了`ResultSet`，但行集不需要与数据库保持连接。

### 5.7.1 构造行集
以下接口扩展了`RowSet`接口：
* `CachedRowSet`允许在断开连接的状态下执行操作。
* `WebRowSet`是可以保存到XML文件的缓存行集。
* `FilteredRowSet`和`JoinRowSet`接口支持对行集的轻量级操作，它们等价于SQL的`SELECT`和`JOIN`操作。
* `JdbcRowSet`是`ResultSet`的薄封装，添加了来自`RowSet`接口的方法。

要获得一个缓存行集，调用

```java
RowSetFactory factory = RowSetProvider.newFactory();
CachedRowSet crs = factory.createCachedRowSet();
```

获得其他类型的行集也有类似的方法。

### 5.7.2 缓存行集
缓存行集包含结果集中的所有数据，并且在关闭数据库连接后仍然可以使用。在程序清单5-4的示例程序中将看到，这大大简化了交互式应用程序的实现。

甚至可以修改缓存行集中的数据。当然，这些修改不会立即反映到数据库中。需要发起一个显式的请求，`CachedRowSet`会重新连接到数据库，并执行SQL语句来写入修改。

可以使用结果集来填充缓存行集：

```java
ResultSet result = ...;
RowSetFactory factory = RowSetProvider.newFactory();
CachedRowSet crs = factory.createCachedRowSet();
crs.populate(result);
conn.close(); // now OK to close the database connection
```

或者，也可以让`CachedRowSet`对象自动建立连接。首先设置连接参数：

```java
crs.setURL("jdbc:derby://localhost:1527/COREJAVA");
crs.setUsername("dbuser");
crs.setPassword("secret");
```

然后设置查询语句和变量：

```java
crs.setCommand("SELECT * FROM Books WHERE Publisher_ID = ?");
crs.setString(1, publisherId);
```

最后，使用查询结果填充行集：

```java
crs.execute();
```

这个调用会建立数据库连接，执行查询，填充行集，并断开连接。

如果查询结果非常大，你可能不想将其全部放入行集。在这种情况下，可以指定页大小：

```java
crs.setPageSize(20);
```

现在只会获得20行。调用`crs.nextPage()`获得下一页。

可以使用与结果集相同的方法来查看和修改行集。如果修改了行集的内容，必须通过调用`crs.acceptChanges(conn)`或`crs.acceptChanges()`将其写回到数据库。只有在行集配置了连接参数（如URL、用户名和密码）时，第二个调用才有效。

在5.6.2节中曾经介绍过，并非所有的结果集都是可更新的。类似地，包含复杂查询结果的行集无法将修改写回到数据库。

警告：如果使用结果集来填充行集，那么行集无法知道要更新的表名，需要调用`setTableName()`来设置表名。

另一个导致复杂性的情况是：在填充了行集之后，数据库发生了改变。显然这可能导致数据不一致。参考实现会检查行集中的原始值与数据库中的当前值是否一致。如果一致，则替换为编辑后的值；否则将抛出`SyncProviderException`，不写入任何修改。其他实现可能采用不同的同步策略。

## 5.8 元数据
除了填充、查询和更新数据库表，JDBC还可以提供关于数据库及其表结构的信息。例如，某个数据库的所有表，或者某个表的列名和类型。对于编写数据库工具的程序员来说，这些结构信息是极其有用的。

在SQL中，描述数据库或其组成部分的数据称为**元数据**(metadata)（区别于数据库中存储的实际数据）。可以获得三种元数据：关于数据库的、关于结果集的以及关于预备语句参数的。

要获得关于数据库的元数据，从连接对象获取一个`DatabaseMetaData`类型的对象。

```java
DatabaseMetaData meta = conn.getMetaData();
```

现在可以获得元数据了。例如

```java
ResultSet mrs = meta.getTables(null, null, null, new String[] {"TABLE"});
```

返回一个包含数据库中所有表信息的结果集。每行对应一张表，第三列是表名（其他列参见`getTables()`方法的API文档）。下面的循环获取所有表名：

```java
while (mrs.next())
    tableNames.addItem(mrs.getString(3));
```

`ResultSetMetaData`接口提供关于结果集的元数据。可以获取结果集的列数以及每列的名字、类型和字段宽度。下面是一个典型的循环：

```java
ResultSet rs = stat.executeQuery("SELECT * FROM " + tableName);
ResultSetMetaData meta = rs.getMetaData();
for (int i = 1; i <= meta.getColumnCount(); i++) {
    String columnName = meta.getColumnLabel(i);
    int columnWidth = meta.getColumnDisplaySize(i);
    ...
}
```

本节将编写一个简单的数据库工具。程序清单5-4中的程序使用元数据来浏览数据库中的所有表。从顶部的组合框选择一张表，窗体中间就会显示出该表的字段名以及第一行的值，如下图所示。点击Next和Previous按钮可以滚动浏览表中的行。还可以删除或编辑行的内容，点击Save按钮将修改保存到数据库。

![ViewDB应用程序](/assets/images/java-note-v2ch05-database-programming/ViewDB应用程序.png)

[程序清单5-4 view/ViewDB.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch05/view/ViewDB.java)

注释：很多数据库都自带了更复杂的查看和编辑表的工具。如果你的数据库没有，可以使用[DBeaver](https://dbeaver.io/)或[SQuirreL](https://squirrel-sql.sourceforge.io/)。

## 5.9 事务
可以将一组语句组成**事务**(transaction)。当所有语句都成功完成后，可以**提交**(commit)事务；如果其中一个发生错误，则可以**回滚**(roll back)，就像没有执行过任何语句一样（即事务具有**原子性**）。

使用事务的主要原因是保证数据库**完整性**(integrity)。例如，假设需要从一个银行账户向另一个账户转账。如果系统在从第一个账户取款之后、向另一个账户存款之前发生故障，则必须撤销取款操作。

### 5.9.1 使用JDBC事务
默认情况下，数据连接处于**自动提交模式**(autocommit mode)，即每条SQL语句在执行后都会立即提交到数据库。一旦提交，就无法进行回滚。为了使用事务，需要关闭这个模式：

```java
conn.setAutoCommit(false);
```

按照通常的方式创建并执行任意多次语句：

```java
Statement stat = conn.createStatement();
stat.executeUpdate(command1);
stat.executeUpdate(command2);
stat.executeUpdate(command3);
...
```

如果所有语句成功执行，则调用`commit()`方法：

```java
conn.commit();
```

如果发生了错误，则调用

```java
conn.rollback();
```

这样，自上次提交之后的所有语句都会自动撤销。通常，当事务被`SQLException`中断时执行回滚。

### 5.9.2 保存点
有些数据库和驱动支持使用**保存点**(save point)更细粒度地控制回滚操作。创建一个保存点，稍后可以返回这个点，而不必放弃整个事务。例如，

```java
Statement stat = conn.createStatement(); // start transaction; rollback() goes here
stat.executeUpdate(command1);
Savepoint svpt = conn.setSavepoint(); // set savepoint; rollback(svpt) goes here
stat.executeUpdate(command2);
if (...) conn.rollback(svpt); // undo effect of command2
...
conn.commit();
```

当不再需要保存点时，应该将其释放：

```java
conn.releaseSavepoint(svpt);
```

### 5.9.3 批量更新
使用**批量更新**(batch update)可以将一系列语句作为一批提交。

注释：使用`DatabaseMetaData`接口的`supportsBatchUpdates()`方法查看你的数据库是否支持该特性。

同一批中的语句可以是`INSERT`、`UPDATE`和`DELETE`等操作，也可以是数据定义语句（如`CREATE TABLE`和`DROP TABLE`）。如果在其中添加`SELECT`语句会抛出异常。

为了执行批量更新，应该调用语句对象的`addBatch()`方法，而不是`executeUpdate()`：

```java
String command = "CREATE TABLE ..."
stat.addBatch(command);

while (...) {
    command = "INSERT INTO ... VALUES (" + ... + ")";
    stat.addBatch(command);
}
```

最后，提交整个批次：

```java
int[] counts = stat.executeBatch();
```

`executeBatch()`方法返回所有提交的语句影响行数构成的数组。

为了在批量模式下正确地处理错误，应该将批量更新视为单个事务。如果在执行过程中失败，则回滚到批量更新开始前的状态。

```java
boolean autoCommit = conn.getAutoCommit();
conn.setAutoCommit(false);
Statement stat = conn.createStatement();
...
// keep calling stat.addBatch(...);
...
stat.executeBatch();
conn.commit();
conn.setAutoCommit(autoCommit);
```

### 5.9.4 高级SQL类型
下表列出了JDBC支持的SQL数据类型及其对应的Java数据类型。

| SQL数据类型 | Java数据类型 |
| --- | --- |
| `INTEGER`, `INT` | `int` |
| `SMALLINT` | `short` |
| `NUMERIC(m,n)`, `DECIMAL(m,n)`, `DEC(m,n)` | `java.math.BigDecimal` |
| `FLOAT(n)` | `double` |
| `REAL` | `float` |
| `DOUBLE` | `double` |
| `CHARACTER(n)`, `CHAR(n)` | `String` |
| `VARCHAR(n)`, `LONG VARCHAR` | `String` |
| `BOOLEAN` | `boolean` |
| `DATE` | `java.sql.Date` |
| `TIME` | `java.sql.Time` |
| `TIMESTAMP` | `java.sql.Timestamp` |
| `BLOB` | `java.sql.Blob` |
| `CLOB` | `java.sql.Clob` |
| `ARRAY` | `java.sql.Array` |
| `ROWID` | `java.sql.RowId` |
| `NCHAR(n)`, `NVARCHAR(n)`, `LONG NVARCHAR` | `String` |
| `NCLOB` | `java.sql.NClob` |
| `SQLXML` | `java.sql.SQLXML` |

## 5.10 Web和企业应用中的连接管理
前面几节介绍的使用database.properties文件设置的简单数据库连接适用于小型测试程序，但无法扩展到大型应用程序。

在Web或企业环境中部署JDBC应用时，数据库连接管理是与Java命名和目录接口(JNDI)集成的。可以将数据源属性存储在一个目录中，以便于集中管理。在这种环境中，可以使用以下代码建立数据库连接：

```java
var jndiContext = new InitialContext();
var source = (DataSource) jndiContext.lookup("java:comp/env/jdbc/corejava");
Connection conn = source.getConnection();
```

注意，不再使用`DriverManager`，而是由JNDI服务定位**数据源**(data source)。

注释：在Java EE容器中，甚至不必编程实现JNDI查找。只需在`DataSource`字段上使用`@Resource`注解，当应用加载时将设置数据源引用：

```java
@Resource(name="jdbc/corejava")
private DataSource source;
```

当然，必须在某个地方配置数据源。如果数据库程序在Servlet容器（如Apache Tomcat）或应用服务器（如GlassFish）中执行，则需要将数据库配置信息（包括JNDI名字、JDBC URL、用户名和密码）放在配置文件中，或者在管理员GUI中设置。

另一个问题涉及建立数据库连接的开销。示例程序使用了两种策略来获取数据库连接：程序清单5-3中的`QueryTest`程序在程序开头建立连接，在程序结尾关闭；程序清单5-4中的`ViewDB`程序在每次需要时都打开一个新连接。但是，这两种方式都不令人满意。数据库连接是有限的资源。如果用户离开应用一段时间，连接就不应该保持打开状态。反过来，每次查询都获取一个连接并在之后关闭的代价也非常高。

上述问题的解决方案是使用**连接池**(connection pool)，这意味着数据库连接保存在一个队列中并重复使用。JDBC规范为连接池实现者提供了钩子(hook)，但JDK本身并未实现，JDBC驱动通常也不包含。相反，Web容器和应用服务器的供应商通常会提供连接池实现。

连接池的使用对程序员来说是完全透明的。通过获取数据源并调用`getConnection()`方法来获取连接池中的连接。使用完连接后调用`close()`方法。这不会关闭物理连接，而是告诉连接池已经使用完。
