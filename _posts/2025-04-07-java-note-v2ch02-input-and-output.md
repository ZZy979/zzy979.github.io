---
title: 《Java核心技术》笔记 卷II 第2章 输入和输出
date: 2025-04-07 21:57:54 +0800
categories: [Java, Core Java]
tags: [java, io, file access, character encoding, unicode, serialization, regular expression]
---
本章将介绍用于输入和输出的Java API。你将学习如何访问文件和目录，以及如何以二进制和文本格式来读写数据。本章还会介绍对象序列化机制。最后将讨论正则表达式。

## 2.1 输入/输出流
在Java API中，可以从其中读取字节序列的对象称为**输入流**(input stream)，可以向其中写入字节序列的对象称为**输出流**(output stream)。这些字节序列的来源和目的地可以是文件、网络连接，甚至是内存块。抽象类`InputStream`和`OutputStream`是输入/输出(I/O)类层次结构的基础。

注释：输入/输出流与上一章介绍的“流”没有任何关系。

面向字节的输入/输出流不便于处理以Unicode形式存储的信息。因此，一个继承自`Reader`和`Writer`类的单独的层次结构提供了用于处理Unicode字符的类。这些类拥有的读写操作是基于`char`值（即UTF-16码元）而不是`byte`值的。

### 2.1.1 读写字节
`InputStream`类有一个抽象方法：

```java
abstract int read()
```

这个方法读取并返回一个字节，如果遇到输入结尾则返回-1。例如，在`FileInputStream`类中，该方法从文件读取一个字节。而`System.in`（`InputStream`的一个子类的预定义对象）从标准输入（即控制台或重定向的文件）中读取信息。

`InputStream`类还有一些非抽象的方法，例如：
* `read(b)` 将字节读取到数组`b`中，返回读取的字节数
* `readAllBytes()` 读取所有字节，返回`byte[]`
* `readNBytes(n)` 读取至多n个字节，返回`byte[]`
* `skip(n)` 跳过至多n个字节

类似地，`OutputStream`类定义了抽象方法

```java
abstract void write(int b)
```

这个方法将一个字节写入输出。还有一个接受`byte[]`的`write()`方法，可以一次写入多个字节。

`InputStream`类的`transferTo()`方法将所有字节从输入流传输到输出流：`in.transferTo(out)`。

`read()`和`write()`方法都将**阻塞**，直到字节真正被读入或写出。这意味着如果输入流不能被立即访问（例如由于网络连接繁忙），当前线程就会阻塞。这使得其他线程有机会去做有用的工作。

`InputStream`类的`available()`方法可用于检查当前可读取的字节数（估计值）。这意味着像下面这样的代码不可能阻塞：

```java
int bytesAvailable = in.available();
if (bytesAvailable > 0) {
    var data = new byte[bytesAvailable];
    in.read(data);
}
```

当你完成对输入/输出流的读写时，应该调用`close()`方法来关闭它，这会释放掉有限的操作系统资源。关闭输出流还会冲刷缓冲区：所有被临时置于缓冲区中（以便用更大的包传递）的字节都将被送出。特别是，如果不关闭文件输出流，那么最后一个包中的字节可能永远得不到传递。也可以调用`flush()`方法手动冲刷缓冲区。

应用程序员很少使用原始的`read()`和`write()`方法。你感兴趣的数据可能包含数字、字符串和对象，而不是原始字节。为此，可以使用`InputStream`和`OutputStream`的众多子类之一。

### 2.1.2 完整的流家族
与C语言只有一个`FILE*`类型不同，Java有60多种不同的输入/输出流类型（如图2.1和2.2所示）。

![输入和输出流层次结构](/assets/images/java-note-v2ch02-input-and-output/输入和输出流层次结构.png)
_图2.1 输入和输出流层次结构_

![Reader和Writer层次结构](/assets/images/java-note-v2ch02-input-and-output/Reader和Writer层次结构.png)
_图2.2 Reader和Writer层次结构_

按照使用方法来划分，形成了处理字节和字符的两个单独的类层次结构。`InputStream`和`OutputStream`类可以读写字节，这两个类是图2.1所示的层次结构的基础。例如，`DataInputStream`和`DataOutputStream`可以以二进制格式读写所有的基本Java类型。`ZipInputStream`和`ZipOutputStream`可以读写ZIP压缩文件。

另一方面，对于Unicode文本，可以使用`Reader`和`Writer`的子类，如图2.2所示。这两个类的基本方法如下：

```java
abstract int read()
abstract void write(int c)
```

`read()`方法读取一个UTF-16码元（0~65536之间的整数），在到达文件结尾时返回-1（所以返回类型不能为`char`）。`write()`方法写入一个UTF-16码元（只使用低16位）。（关于Unicode和码元参见卷I第3章 3.3.4和3.6.6节）

另外还有四个接口：`Closeable`、`Flushable`、`Readable`和`Appendable`（如下图所示）。

![Closeable、Flushable、Readable和Appendable接口](/assets/images/java-note-v2ch02-input-and-output/Closeable、Flushable、Readable和Appendable接口.png)

前两个接口非常简单，分别有以下方法：

```java
void close() throws IOException
void flush()
```

`InputStream`、`OutputStream`、`Reader`和`Writer`都实现了`Closeable`接口。

注释：`java.io.Closeable`接口扩展了`java.lang.AutoCloseable`接口。因此，对任何`Closeable`都可以使用带资源的`try`语句（见卷I第7章 7.2.5节）。为什么有两个接口？因为`Closeable.close()`方法只抛出`IOException`，而`AutoCloseable.close()`方法可以抛出任何异常。

`OutputStream`和`Writer`还实现了`Flushable`接口。

`Readable`接口只有一个方法：

```java
int read(CharBuffer cb)
```

`CharBuffer`类具有顺序和随机读写访问的方法，它表示内存缓冲区或内存映射文件（详见2.5.2节）。

`Appendable`接口有两个用于添加单个字符和字符序列的方法：

```java
Appendable append(char c)
Appendable append(CharSequence s)
```

在输入/输出流中，只有`Writer`和`PrintStream`实现了`Appendable`。

`CharSequence`接口描述了一个`char`值序列。`String`、`CharBuffer`、`StringBuilder`和`StringBuffer`都实现了它。

### 2.1.3 组合输入/输出流过滤器
`FileInputStream`是用于读取文件的输入流，需要向构造器提供文件名或完整路径。例如：

```java
var fin = new FileInputStream("employee.dat");
```

提示：`java.io`中的所有类都将相对路径解释为从**工作目录**(working directory)开始。可以通过调用`System.getProperty("user.dir")`来获得这个目录。（注：另见卷I第3章 3.7.3节第二个注释）

警告：由于反斜杠在Java字符串中是转义字符，对于Windows路径名要使用`\\`（例如`C:\\Windows\\win.ini`）。

文件输入流只支持读取字节：

```java
byte b = (byte) fin.read();
```

在下一节将会看到，`DataInputStream`可以读取数值类型：

```java
DataInputStream din = ...;
double x = din.readDouble();
```

但是，正如`FileInputStream`没有读取数值类型的方法，`DataInputStream`没有从文件读取数据的方法。

Java使用了一种巧妙的机制来分离这两种职责。有些输入流（如`FileInputStream`）可以从文件或其他位置获取字节；而其他输入流（如`DataInputStream`）可以将字节组装为更有用的数据类型。Java程序员必须对二者进行组合。

例如，为了能够从文件中读取数字，需要先创建一个`FileInputStream`，然后将其传递给`DataInputStream`的构造器：

```java
var fin = new FileInputStream("employee.dat");
var din = new DataInputStream(fin);
double x = din.readDouble();
```

再次查看图2.1，可以看到`FilterInputStream`和`FilterOutputStream`类。它们的子类用于向处理字节的输入/输出流添加额外的功能。（注：这些“过滤器流”的共同点是构造器接受一个输入/输出流参数，其读写方法会调用底层流的方法。）

可以通过嵌套过滤器来添加多种功能。例如，默认情况下，输入流是无缓冲的(unbuffered)。也就是说，每次调用`read()`都会请求操作系统分发下一个字节。而一次请求一个数据块并将其存储在缓冲区中会更高效（注：缓冲区(buffer)就是内存中的一个字节数组）。如果想对文件使用缓冲和数据输入，就需要使用如下的构造器序列：

```java
var din = new DataInputStream(
    new BufferedInputStream(
        new FileInputStream("employee.dat")));
```

注意，我们把`DataInputStream`放在构造器链的最后，因为我们希望使用数据输入方法，并且希望这些方法使用带缓冲机制的`read()`方法。

在读取输入时，经常需要预览下一个字节是否是期望的值（如果不是则放回）。为此，Java提供了`PushbackInputStream`。

```java
var pbin = new PushbackInputStream(
    new BufferedInputStream(
        new FileInputStream("employee.dat")));
```

现在就可以大胆地读取下一个字节，如果不是想要的值就将其放回：

```java
int b = pbin.read();
if (b != '<') pbin.unread(b);
```

注：`PushbackInputStream`自带一个缓冲区，但仅用于实现放回功能。如果需要缓冲机制，仍然需要嵌套`BufferedInputStream`。

然而，`PushbackInputStream`只有`read()`和`unread()`方法。如果希望能够预览/放回并且读取数值，那么既需要放回输入流又需要数据输入流。

```java
var pbin = new PushbackInputStream(
    new BufferedInputStream(
        new FileInputStream("employee.dat")));
var din = new DataInputStream(pbin);
```

在其他语言的I/O库中，诸如缓冲和预览等细节都是自动处理的。相比之下，在Java中需要借助于组合流过滤器来实现，有点麻烦。但是，这种能力带来了极大的灵活性。例如，可以如下从ZIP压缩文件中读取数字：

```java
var zin = new ZipInputStream(new FileInputStream("employee.zip"));
var din = new DataInputStream(zin);
```

关于ZIP文件详见2.2.3节。

## 文本输入和输出
在保存数据时，可以选择二进制或文本格式。例如，整数1234用二进制格式存储为字节序列`00 00 04 D2`（大端序十六进制表示法），而用文本格式存储为字符串`"1234"`。尽管二进制I/O快速且高效，但不容易被人类阅读。本节先讨论文本I/O，2.2节将讨论二进制I/O。

在存储文本字符串时，需要考虑字符编码。例如，字符串 "José" 的UTF-16编码为`4a 00 6f 00 73 00 e9 00`（每个字符占2字节，对应Java字符串常量`"\u004a\u006f\u0073\u00e9"`），而UTF-8编码为`4a 6f 73 c3 a9`（前3个字符占1字节，字符é占2字节）。

注：可以使用`String.getBytes()`方法得到字符串在指定编码下的字节数组。

`OutputStreamWriter`类使用指定的字符编码将字节输出流转换为字符`Writer`（写入的字符被编码为字节发送到底层输出流）。相反，`InputStreamReader`将字节输入流转换为字符`Reader`。

例如，可以如下创建一个reader，从控制台读取按键并将其转换为Unicode：

```java
var in = new InputStreamReader(System.in);
```

这个reader会使用默认字符编码(`Charset.defaultCharset()`)，取决于操作系统和区域设置。应该始终在构造器中指定具体的字符编码，例如：

```java
var in = new InputStreamReader(new FileInputStream("data.txt"), StandardCharsets.UTF_8);
```

关于字符编码详见2.1.8节。

`Reader`和`Writer`类只有读写单个字符的基本方法。与输入/输出流一样，可以使用子类来处理字符串和数字。

### 2.1.5 如何写文本输出
对于文本输出，使用`PrintWriter`。这个类具有与`System.out`相同的`print()`、`println()`和`printf()`方法，可以用这些方法来打印数字、字符、布尔值、字符串和对象。

为了打印到文件，使用文件名和字符编码构造一个`PrintWriter`对象：

```java
var out = new PrintWriter("employee.txt", StandardCharsets.UTF_8);
String name = "Harry Hacker";
double salary = 75000;
out.print(name);
out.print(' ');
out.println(salary);
// writes "Harry Hacker 75000.0" to employee.txt
```

注：还有一个`FileWriter`类，但是只有写入单个字符或字符串的`write()`方法，而没有这些`print`方法。

`println()`方法会在输出行中添加适当的换行符，即通过调用`System.getProperty("line.separator")`获得的字符串（Windows中是`"\r\n"`，UNIX中是`"\n"`）。

如果writer设置为**自动冲刷模式**(auto-flush mode)，那么每次调用`println()`都会冲刷缓冲区（`PrintWriter`总是带缓冲的）。默认情况下禁用自动冲刷，可以使用以下构造器来开启：

```java
var out = new PrintWriter(
    new FileOutputStream("employee.txt"),
    true, // auto-flush
    StandardCharsets.UTF_8);
```

这些`print`方法不会抛出异常。可以调用`checkError()`方法来检查是否出现了错误。

注释：`System.out`的类型是`PrintStream`而不是`PrintWriter`。

### 2.1.6 如何读取文本输入
读取文本输入最简单的方法就是使用卷I第3章 3.7.1节介绍过的`Scanner`类。可以从任何输入流构造`Scanner`对象。

或者，也可以像下面这样将短小的文本文件读入一个字符串中：

```java
String content = Files.readString(path, charset);
```

如果希望将文件读取为行的序列，则调用

```java
List<String> lines = Files.readAllLines(path, charset);
```

如果文件很大，可以作为`Stream<String>`惰性处理行：

```java
try (Stream<String> lines = Files.lines(path, charset)) {
    ...
}
```

还可以使用`Scanner`来读取**单词**(token)——由分隔符分隔的字符串。默认的分隔符是空白符，可以修改为任意的正则表达式。例如，

```java
Scanner in = ...;
in.useDelimiter("\\PL+");
```

使用任何非Unicode字母作为分隔符（注：正则表达式`\PL`匹配单个非Unicode字母），因此这个scanner接受仅由Unicode字母组成的单词。调用`next()`方法产生下一个单词：

```java
while (in.hasNext()) {
    String word = in.next();
    ...
}
```

或者，可以如下获得包含所有单词的流：

```java
Stream<String> words = in.tokens();
```

在Java的早期版本中，处理文本输入的唯一方式是`BufferedReader`类。其`readLine()`方法产生一行文本，如果没有更多输入则返回`null`。典型的输入循环如下：

```java
InputStream inputStream = ...;
try (var in = new BufferedReader(new InputStreamReader(inputStream, charset))) {
    String line;
    while ((line = in.readLine()) != null) {
        // do something with line
    }
}
```

如今，`BufferedReader`类又有了一个产生`Stream<String>`的`lines()`方法，但没有用于读取数字的方法。

### 2.1.7 以文本格式保存对象
本节的示例程序将一个`Employee`数组存储为文本文件，然后再读入。每条记录存储为单独的一行，实例字段之间用分隔符`|`分隔（假定要存储的字符串不包含`|`）。下面是一个示例：

```
Carl Cracker|75000.0|1987-12-15
Harry Hacker|50000.0|1989-10-01
Tony Tester|40000.0|1990-03-15
```

写出记录很简单。使用`PrintWriter`类，直接写出所有字段，后面跟着一个`|`，最后一个字段后面跟着换行符。

```java
public static void writeEmployee(PrintWriter out, Employee e) {
    out.println(e.getName() + "|" + e.getSalary() + "|" + e.getHireDay());
}
```

为了读取记录，使用`Scanner`每次读取一行并分割所有字段。

```java
public static Employee readEmployee(Scanner in) {
    String line = in.nextLine();
    String[] tokens = line.split("\\|");
    String name = tokens[0];
    double salary = Double.parseDouble(tokens[1]);
    LocalDate hireDate = LocalDate.parse(tokens[2]);
    int year = hireDate.getYear();
    int month = hireDate.getMonthValue();
    int day = hireDate.getDayOfMonth();
    return new Employee(name, salary, year, month, day);
}
```

静态方法`writeData()`先写出数组长度，然后写出每条记录。相反，静态方法`readData()`先读入数组长度，然后读入每条记录。注意，`nextInt()`不会读入结尾的换行符，因此要调用一次`nextLine()`将换行符读走（见卷I 3.7.1节注）（注：如果一行只有一个整数，也可以使用`Integer.parseInt(in.nextLine().trim())`）。

完整的程序如程序清单2-1所示。

[程序清单2-1 textFile/TextFileTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch02/textFile/TextFileTest.java)

### 2.1.8 字符编码
Java对于字符使用[Unicode标准](https://www.unicode.org/standard/standard.html)。每个字符都有一个**码点**，表示为21位的整数(U+0000~U+10FFFF)（参见[Unicode Character Table](https://symbl.cc/en/unicode-table/)）。

有多种不同的**字符编码**(character encoding)/**字符集**(character set)，也就是将Unicode码点转换为字节的方法。最常见的编码是UTF-8（由[RFC 2279](https://www.ietf.org/rfc/rfc2279.txt)定义），将每个Unicode码点编码为1~4个**字节**（8位整数，对应Java的`byte`），如下表所示。UTF-8的优点是传统的ASCII字符集（包含了英语中使用的所有字符）中的每个字符只占用一字节。

| 字符范围 | UTF-8编码 |
| --- | --- |
| 0~7F | **0**<font color="#FF0000">a<sub>6</sub>a<sub>5</sub>a<sub>4</sub>a<sub>3</sub>a<sub>2</sub>a<sub>1</sub>a<sub>0</sub></font> |
| 80~7FF | **110**<font color="#00A000">a<sub>10</sub>a<sub>9</sub>a<sub>8</sub>a<sub>7</sub>a<sub>6</sub></font>&emsp;**10**<font color="#FF0000">a<sub>5</sub>a<sub>4</sub>a<sub>3</sub>a<sub>2</sub>a<sub>1</sub>a<sub>0</sub></font> |
| 800~FFFF | **1110**<font color="#0000FF">a<sub>15</sub>a<sub>14</sub>a<sub>13</sub>a<sub>12</sub></font>&emsp;**10**<font color="#00A000">a<sub>11</sub>a<sub>10</sub>a<sub>9</sub>a<sub>8</sub>a<sub>7</sub>a<sub>6</sub></font>&emsp;**10**<font color="#FF0000">a<sub>5</sub>a<sub>4</sub>a<sub>3</sub>a<sub>2</sub>a<sub>1</sub>a<sub>0</sub></font> |
| 10000~10FFFF | **11110**<font color="#FF00FF">a<sub>20</sub>a<sub>19</sub>a<sub>18</sub></font>&emsp;**10**<font color="#0000FF">a<sub>17</sub>a<sub>16</sub>a<sub>15</sub>a<sub>14</sub>a<sub>13</sub>a<sub>12</sub></font>&emsp;**10**<font color="#00A000">a<sub>11</sub>a<sub>10</sub>a<sub>9</sub>a<sub>8</sub>a<sub>7</sub>a<sub>6</sub></font>&emsp;**10**<font color="#FF0000">a<sub>5</sub>a<sub>4</sub>a<sub>3</sub>a<sub>2</sub>a<sub>1</sub>a<sub>0</sub></font> |

注：在上表中，加粗的位是固定的，其余a<sub>i</sub>表示码点的第i位。例如，英文字母 'A' 、希腊字母 'π' 、汉字 '你' 和数学符号 '𝕆' 的UTF-8编码如下：

| 字符 | 码点 | UTF-8编码 |
| --- | --- | --- |
| A | U+0041<br>0<font color="#FF0000">1000001</font> | 41<br>**0**<font color="#FF0000">1000001</font> |
| π | U+03C0<br>00000<font color="#00A000">011&emsp;11</font><font color="#FF0000">000000</font> | CF 80<br>**110**<font color="#00A000">01111</font>&emsp;**10**<font color="#FF0000">000000</font> |
| 你 | U+4F60<br><font color="#0000FF">0100</font><font color="#00A000">1111&emsp;01</font><font color="#FF0000">100000</font> | E4 BD A0<br>**1110**<font color="#0000FF">0100</font>&emsp;**10**<font color="#00A000">111101</font>&emsp;**10**<font color="#FF0000">100000</font> |
| 𝕆 | U+1D546<br><font color="#FF00FF">000</font><font color="#0000FF">01&emsp;1101</font><font color="#00A000">0101&emsp;01</font><font color="#FF0000">000110</font> | F0 9D 95 86<br>**11110**<font color="#FF00FF">000</font>&emsp;**10**<font color="#0000FF">011101</font>&emsp;**10**<font color="#00A000">010101</font>&emsp;**10**<font color="#FF0000">000110</font> |

另一种常见的编码是UTF-16（由[RFC 2781](https://www.ietf.org/rfc/rfc2781.txt)定义），将每个Unicode码点编码为1个或2个16位的值（对应Java的`char`），如下表所示。这是Java字符串使用的编码方式。实际上，有两种形式的UTF-16，分别称为**大端序**(big-endian, BE)和**小端序**(little-endian, LE)。考虑16位值0x2122，在大端序中，高位字节在前：21 22；在小端序中则相反：22 21。为了指示使用的字节顺序，文件可以以**字节顺序标记**(byte order mark, BOM)开头，即16位值0xFEFF。读取文件时可以使用这个值来确定字节顺序（如果读到FE FF则表示大端序，如果读到FF FE则表示小端序），然后丢弃它。

| 字符范围 | UTF-16编码 |
| --- | --- |
| 0~FFFF | a<sub>15</sub>a<sub>14</sub>a<sub>13</sub>a<sub>12</sub>a<sub>11</sub>a<sub>10</sub>a<sub>9</sub>a<sub>8</sub>&emsp;a<sub>7</sub>a<sub>6</sub>a<sub>5</sub>a<sub>4</sub>a<sub>3</sub>a<sub>2</sub>a<sub>1</sub>a<sub>0</sub> |
| 10000~10FFFF | **110110**<font color="#FF00FF">b<sub>19</sub>b<sub>18</sub>&emsp;b<sub>17</sub>b<sub>16</sub></font><font color="#0000FF">a<sub>15</sub>a<sub>14</sub>a<sub>13</sub>a<sub>12</sub>a<sub>11</sub>a<sub>10</sub></font>&emsp;**110111**<font color="#00A000">a<sub>9</sub>a<sub>8</sub></font>&emsp;<font color="#FF0000">a<sub>7</sub>a<sub>6</sub>a<sub>5</sub>a<sub>4</sub>a<sub>3</sub>a<sub>2</sub>a<sub>1</sub>a<sub>0</sub></font><br>其中b<sub>19</sub>b<sub>18</sub>b<sub>17</sub>b<sub>16</sub> = a<sub>20</sub>a<sub>19</sub>a<sub>18</sub>a<sub>17</sub>a<sub>16</sub> - 1 |

注：上面UTF-8示例中四个字符的UTF-16BE编码如下：

| 字符 | 码点 | UTF-16BE编码 |
| --- | --- | --- |
| A | U+0041<br>00000000&emsp;01000001 | 00 41<br>00000000&emsp;01000001 |
| π | U+03C0<br>00000011&emsp;11000000 | 03 C0<br>00000011&emsp;11000000 |
| 你 | U+4F60<br>01001111&emsp;01100000 | 4F 60<br>01001111&emsp;01100000 |
| 𝕆 | U+1D546<br><font color="#FF00FF">00001</font>&emsp;<font color="#0000FF">110101</font><font color="#00A000">01</font>&emsp;<font color="#FF0000">01000110</font> | D8 35 DD 46<br>**110110**<font color="#FF00FF">00&emsp;00</font><font color="#0000FF">110101</font>&emsp;**110111**<font color="#00A000">01</font>&emsp;<font color="#FF0000">01000110</font> |

UTF-32最简单，字符的Unicode码点本身就是其编码，但每个字符需要占用4字节。

除了UTF编码，还有一些编码方式只覆盖了适用于特定用户人群的字符范围。例如，ISO-8859-1（也叫latin1）是一种单字节编码，包含了西欧语言中使用的带重音符号的字符（如é）。Shift_JIS是一种用于日文字符的变长编码。GBK用于编码汉字。大量这样的编码方式仍被广泛使用。

注：IANA字符集数据库(<https://www.iana.org/assignments/character-sets/character-sets.xhtml>)包含所有字符集的完整列表。

没有可靠的方法可以从字节流中自动检测字符编码。有些API方法使用默认字符集（即计算机操作系统首选的字符编码），这可能与字节源使用的编码不同。因此，**应该始终明确指定编码方式**。例如，在读取网页时，应该检查`Content-Type`标头。

注释：平台默认字符集由静态方法`Charset.defaultCharset()`以及系统属性`native.encoding`返回。静态方法`Charset.availableCharsets()`以名称到`Charset`对象映射的形式返回所有可用的字符集。

警告：Java的Oracle实现有一个用于覆盖平台默认字符集的系统属性`file.encoding`。这并非官方支持的属性，并且Java库的Oracle实现并未以一致的方式处理它。因此你不应该设置该属性。

`StandardCharsets`类具有`Charset`类型的静态变量，用于表示所有Java虚拟机都必须支持的字符编码（例如`StandardCharsets.UTF_8`）。

要获得指定名称的`Charset`对象，使用静态方法`forName()`：

```java
Charset shiftJIS = Charset.forName("Shift_JIS");
```

在读或写文本时，使用`Charset`对象指定字符集。例如，可以如下将字节数组按UTF-8编码转换为字符串：

```java
var str = new String(bytes, StandardCharsets.UTF_8);
```

提示：从Java 10起，`java.io`包中的所有方法都允许用`Charset`对象或字符串名称来指定字符编码。最好选择`StandardCharsets`常量以避免拼写错误。

警告：如果不指定编码，有些方法（如`String(byte[])`构造器）会使用平台默认编码，而其他方法（如`Files.readAllLines()`）会使用UTF-8。

注释：从Java 17以后，UTF-8将成为JDK的默认编码，不管平台默认编码是什么。在此之前，需要明确指定字符编码。

## 2.2 读写二进制数据
文本格式对于测试和调试而言很方便，因为它是人类可读的，但是它并不像二进制格式那样高效。下面几节将介绍如何用二进制数据进行输入和输出。

### 2.2.1 DataInput和DataOutput接口
`DataOutput`接口定义了用于以二进制格式写数字、字符、布尔值和字符串的方法。例如，`writeInt()`将一个`int`写出为4字节，`writeDouble()`将一个`double`写出为8字节。其结果并非人类可读的，但是给定类型的每个值使用的空间都是相同的，而且将其读回也比解析文本更快。

注释：C和C++在内存中存储数字的字节顺序取决于使用的处理器。例如，对于整数1234（即十六进制的4D2），用大端序存储为`00 00 04 D2`，而小端序为`D2 04 00 00`。在Java中，**所有值都是按大端序写出的**。这使得Java数据文件与平台无关。

`writeUTF()`方法使用修改版的UTF-8写出字符串（详见`DataInput`接口API文档）。与标准UTF-8编码相比，修改版对编码大于0xFFFF的字符的处理有所不同，这是为了向后兼容在Unicode还没有超过16位时构建的虚拟机。由于没有其他地方会使用这种修改版的UTF-8，你应该只在写出用于Java虚拟机的字符串时（例如生成字节码的程序）才使用`writeUTF()`方法。对于其他目的，使用`writeChars()`方法。

为了读回数据，使用`DataInput`接口定义的相应方法，如`readInt()`、`readDouble()`、`readUTF()`等。

`DataInputStream`类实现了`DataInput`接口。为了从文件读取二进制数据，需要将`DataInputStream`与`FileInputStream`组合：

```java
var in = new DataInputStream(new FileInputStream("employee.dat"));
```

类似地，要写出二进制数据，使用实现了`DataOutput`接口的`DataOutputStream`类：

```java
var out = new DataOutputStream(new FileOutputStream("employee.dat"));
```

### 2.2.2 随机访问文件
`RandomAccessFile`类可以在文件中的任何位置读写数据。磁盘文件是可随机访问的，但是与网络套接字通信的输入/输出流不是。可以打开一个随机访问文件用于只读或读写，使用字符串`"r"`（只读）或`"rw"`（读写）作为构造器的第二个参数来指定模式：

```java
var in = new RandomAccessFile("employee.dat", "r");
var inOut = new RandomAccessFile("employee.dat", "rw");
```

随机访问文件有一个**文件指针**(file pointer)，指示下一个将被读或写的字节的位置。可以用`seek()`方法将文件指针设置到文件内的任意字节位置，其参数是一个0到文件长度（单位为字节）之间的`long`整数。`getFilePointer()`方法返回文件指针的当前位置。

`RandomAccessFile`类同时实现了`DataInput`和`DataOutput`接口，因此可以使用上一节讨论的`read`/`write`方法来读写随机访问文件。

下面编写一个以二进制格式将`Employee`记录存储到随机访问文件中的程序。每条记录都有相同的大小，这样可以很容易地读写任意一条记录。假设想读取第3条记录，只需将文件指针置于第3条记录的开始字节并读取。

```java
long n = 3;
in.seek((n - 1) * RECORD_SIZE);
var e = readData(in);
```

类似地，可以修改（覆盖）第n条记录：

```java
in.seek((n - 1) * RECORD_SIZE);
writeData(out, e);
```

`length()`方法返回文件的字节数，记录条数等于文件长度除以每条记录的大小。

```java
long nbytes = in.length(); // length in bytes
int nrecords = (int) (nbytes / RECORD_SIZE);
```

在二进制格式中，整数和浮点值都具有固定大小，但是处理字符串有些麻烦。我们提供了两个辅助方法来读写固定长度的字符串。

`writeFixedString()`方法写出字符串开头指定个数的字符，如果过长则截断，过短则用0补齐。

```java
public static void writeFixedString(String s, int size, DataOutput out) throws IOException {
    for (int i = 0; i < size; i++) {
        char ch = 0;
        if (i < s.length()) ch = s.charAt(i);
        out.writeChar(ch);
    }
}
```

`readFixedString()`方法从输入流读取字符，直到读入`size`个字符或者遇到0，然后跳过输入字段中剩余的0。为了提高效率，这个类使用了`StringBuilder`类来读入字符串。

```java
public static String readFixedString(int size, DataInput in) throws IOException {
    var b = new StringBuilder(size);
    int i = 0;
    var done = false;
    while (!done && i < size) {
        char ch = in.readChar();
        i++;
        if (ch == 0) done = true;
        else b.append(ch);
    }
    in.skipBytes(2 * (size - i));
    return b.toString();
}
```

我们将这两个方法放到辅助类`DataIO`中。

为了写出一条`Employee`记录，直接以二进制写出所有字段：

```java
DataIO.writeFixedString(e.getName(), NAME_SIZE, out);
out.writeDouble(e.getSalary());
LocalDate hireDay = e.getHireDay();
out.writeInt(hireDay.getYear());
out.writeInt(hireDay.getMonthValue());
out.writeInt(hireDay.getDayOfMonth());
```

读回数据也很简单：

```java
String name = DataIO.readFixedString(NAME_SIZE, in);
double salary = in.readDouble();
int y = in.readInt();
int m = in.readInt();
int d = in.readInt();
```

每条记录的大小为100字节：
* `name`：40字符 = 80字节
* `salary`：1个`double` = 8字节
* `hireDay`：3个`int` = 12字节

![Employee记录结构](/assets/images/java-note-v2ch02-input-and-output/Employee记录结构.png)

程序清单2-2所示的程序将三条记录写入一个数据文件中，然后以逆序将其从文件中读回。

[程序清单2-2 randomAccess/RandomAccessTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch02/randomAccess/RandomAccessTest.java)

### 2.2.3 ZIP文件
ZIP文件以压缩格式存储一个或多个文件。在Java中，可以使用`ZipInputStream`来读取ZIP文件。为了访问ZIP文件中的**条目**（即文件），调用`getNextEntry()`方法得到下一个条目（`ZipEntry`类型的对象），调用`read()`读取当前条目的数据（到一个`byte[]`中），然后调用`closeEntry()`并读取下一个条目。读取最后一个条目后关闭输入流。下面是读取ZIP文件的典型代码：

```java
var zin = new ZipInputStream(new FileInputStream(zipname));
boolean done = false;
while (!done) {
    ZipEntry entry = zin.getNextEntry();
    if (entry == null)
        done = true;
    else {
        // read the contents of zin with read()
        zin.closeEntry();
    }
}
zin.close();
```

[zip/ZipTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch02/zip/ZipTest.java)

注：
* `ZipEntry`对象只包含条目本身的信息（如文件名、压缩方法、数据大小等），而不包含数据。需要使用`ZipInputStream.read()`方法读取当前条目的数据。
* `ZipInputStream`只能按顺序读取所有条目，`java.util.zip.ZipFile`类支持读取指定名称的条目。

要写出ZIP文件，使用`ZipOutputStream`。对于每个条目，创建一个`ZipEntry`对象并指定文件名。然后调用`putNextEntry()`方法，并使用`write()`写出文件数据。完成后调用`closeEntry()`。代码框架如下：

```java
var fout = new FileOutputStream("test.zip");
var zout = new ZipOutputStream(fout);
for (all files) {
    var ze = new ZipEntry(filename);
    zout.putNextEntry(ze);
    // send data to zout with write()
    zout.closeEntry();
}
zout.close();
```

注释：JAR文件（已在卷I第4章中讨论过）就是带有清单的ZIP文件。可以使用`JarInputStream`和`JarOutputStream`类来读写清单条目。

ZIP输入流是体现流抽象的强大之处的一个很好的例子。当读取以压缩格式存储的数据时，不必担心边读取边解压缩的问题。此外，ZIP流的字节源不必是文件，也可以是网络连接。

注释：2.4.8节将展示如何使用Java 7的`FileSystem`类来访问ZIP文件。

## 2.3 对象输入/输出流与序列化
如果需要存储相同类型的数据，使用固定长度的记录格式是一个不错的选择。但是，在面向对象程序中创建的对象很少全都具有相同的类型。例如，可能有一个数组名义上是`Employee`数组，但实际上包含诸如`Manager`这样的子类实例。

Java语言支持一种称为**对象序列化**(object serialization)的非常通用的机制，可以将任何对象写到输出流，并在之后将其读回。

### 2.3.1 保存和加载可序列化对象
为了保存对象数据，首先需要打开一个`ObjectOutputStream`对象：

```java
var out = new ObjectOutputStream(new FileOutputStream("employee.dat"));
```

为了保存对象，只需使用`ObjectOutputStream`类的`writeObject()`方法，如下所示：

```java
var harry = new Employee("Harry Hacker", 50000, 1989, 10, 1);
var boss = new Manager("Carl Cracker", 80000, 1987, 12, 15);
out.writeObject(harry);
out.writeObject(boss);
```

为了将对象读回，首先获得一个`ObjectInputStream`对象：

```java
var in = new ObjectInputStream(new FileInputStream("employee.dat"));
```

然后使用`readObject()`方法按写出对象的顺序读取它们：

```java
var harry = (Employee) in.readObject();
var boss = (Manager) in.readObject();
```

通过对象输入/输出流读写的类必须实现`Serializable`接口：

```java
class Employee implements Serializable { ... }
```

`Serializable`接口没有任何方法（即标记接口，参见卷I第6章 6.1.9节），因此不需要对类做任何改动。

注释：只能使用`readObject()`/`writeObject()`方法读写**对象**。对于基本类型，使用诸如`readInt()`/`writeInt()`这样的方法（对象输入/输出流实现了`ObjectInput`/`ObjectOutput`接口，而这两个接口分别继承了`DataInput`/`DataOutput`接口）。

在幕后，`ObjectOutputStream`查看对象的所有字段并存储其内容。

但是，有一种重要的情况需要考虑：一个对象被多个对象共享（作为实例字段）。为了说明这个问题，对`Manager`类稍作修改。假设每个经理都有一个秘书：

```java
class Manager extends Employee {
    private Employee secretary;
    ...
}
```

现在每个`Manager`对象都包含一个`Employee`对象的引用。当然，两个经理可能共用一个秘书，如下面的代码和下图所示：

```java
var harry = new Employee("Harry Hacker", ...);
var carl = new Manager("Carl Cracker", ...);
carl.setSecretary(harry);
var tony = new Manager("Tony Tester", ...);
tony.setSecretary(harry);
```

![两个经理共用一个秘书](/assets/images/java-note-v2ch02-input-and-output/两个经理共用一个秘书.png)

保存这样的对象网络（即对象之间的引用关系构成的图结构）是一个挑战。当然，我们不能保存和恢复对象的内存地址，因为对象被重新加载时可能占据完全不同的内存地址。

每个对象都用一个**序列号**(serial number)保存——这就是这种机制称为“对象序列化”的原因。其算法如下：
1. 对于遇到的每个对象引用关联一个序列号。
2. 如果第一次遇到某个对象引用，则将对象数据保存到输出流。
3. 如果对象之前已经被保存过，则只记录序列号。

![对象序列化示例](/assets/images/java-note-v2ch02-input-and-output/对象序列化示例.png)

在读回对象时，过程是反过来的：
1. 对于输入流中第一次遇到的对象，构造它，并使用流中的数据来初始化，然后记录序列号和对象引用之间的关联。
2. 当遇到序列号标记时，直接获取序列号关联的对象引用。

注释：在本章中，我们使用序列化将对象集合保存到磁盘文件并原样读回。序列化的另一种非常重要的应用是通过网络将对象集合传输到另一台计算机。通过**用序列号代替内存地址**，序列化允许将对象集合从一台机器传送到另一台机器（注：前提是这两台机器上有相同的类文件）。

警告：对象流包含所有被序列化对象的类、超类和字段的名字。对于内部类，有些名字由编译器合成，不同编译器的命名约定可能不同。如果发生了这种情况，反序列化就会失败。因此，对于序列化内部类需要小心。静态内部类（包括内部枚举和记录）可以安全地序列化。

程序清单2-3是保存和重新加载`Employee`和`Manager`对象网络的程序。

[程序清单2-3 serial/ObjectStreamTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch02/serial/ObjectStreamTest.java)

### 2.3.2 理解对象序列化文件格式
对象序列化以特定的文件格式保存对象数据。具体细节参见[Java对象序列化规范](https://docs.oracle.com/en/java/javase/17/docs/specs/serialization/)中的[Object Serialization Stream Protocol](https://docs.oracle.com/en/java/javase/17/docs/specs/serialization/protocol.html)一节。

### 2.3.3 修改默认序列化机制
有些实例字段不应该被序列化，例如存储文件句柄或窗口句柄的值，这些值只对本地方法有意义。可以将这些字段用关键字`transient`（瞬时的）标记来防止它们被序列化。如果字段属于不可序列化的类，也需要将其标记成`transient`。这种字段在序列化时会被跳过。

注释：如果一个可序列化的类的超类是不可序列化的，那么超类必须有一个无参构造器，否则反序列化会失败。例如：

```java
class Person // Not serializable
class Employee extends Person implements Serializable
```

当反序列化一个`Employee`对象时，它的实例字段从对象输入流中读取，但是超类的实例字段只能由`Person`的默认构造器设置（因此无法还原之前的值）。

序列化机制提供了为单个类自定义读写行为的方式。可序列化的类可以定义如下方法：

```java
@Serial private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException
@Serial private void writeObject(ObjectOutputStream out) throws IOException
```

之后，实例字段就不再会被自动序列化，而是调用这些方法。这两个方法只需要读写当前类的数据，不关心超类数据和其他类信息。

注意，这两个方法并不属于某个接口，因此不能使用`@Override`注解让编译器检查方法声明。`@Serial`注解意在为序列化方法做相同的检查。但是到Java 17为止，`javac`编译器都还没有做这样的检查（未来可能会做）。IntelliJ IDEA会使用这个注解。

下面是一个典型的自定义序列化示例。`java.awt.geom`包中的许多类（如`Point2D.Double`）都是不可序列化的。现在假设你想序列化一个`LabeledPoint`类：

```java
public class LabeledPoint implements Serializable {
    private String label;
    private transient Point2D.Double point;
    ...
}
```

在`writeObject()`方法中，首先通过调用对象输出流的`defaultWriteObject()`方法写出对象描述符和`label`字段，然后使用`writeDouble()`方法写出点的坐标。

```java
@Serial
private void writeObject(ObjectOutputStream out) throws IOException {
    out.defaultWriteObject();
    out.writeDouble(point.getX());
    out.writeDouble(point.getY());
}
```

疑问：调用`defaultWriteObject()`方法时并没有将`this`作为参数，那么对象输出流是如何访问当前对象的实例字段的？

实际上，在一开始调用`out.writeObject(labeledPoint)`时，对象输出流会将当前写出的对象保存在`curContext`字段中，然后通过反射调用对象的`writeObject()`方法（如果有）。之后`defaultWriteObject()`方法会从`curContext`字段获取当前对象。相关代码位置：

```
ObjectOutputStream.writeObject(obj)
  writeObject0()
    writeOrdinaryObject()
      writeSerialData()
        curContext = new SerialCallbackContext(obj, ...)  // 保存当前对象
        ObjectStreamClass.invokeWriteObject()
          LabeledPoint.writeObject()  // 通过反射调用
            ObjectOutputStream.defaultWriteObject()
              curObj = curContext.getObj()  // 获取当前对象
              defaultWriteFields(curObj, ...)
```

在`readObject()`方法中，过程是相反的：

```java
@Serial
private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {
    in.defaultReadObject();
    double x = in.readDouble();
    double y = in.readDouble();
    point = new Point2D.Double(x, y);
}
```

[serializationTweaks/LabeledPoint.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch02/serializationTweaks/LabeledPoint.java)

另一个例子是`HashSet`类，它提供了自己的`readObject()`和`writeObject()`方法。`writeObject()`方法直接保存容量、大小、负载因子和元素，而不是保存散列表的内部结构。`readObject()`方法读回这些信息，构造一个新的散列表并插入元素。

警告：就像构造器一样，`readObject()`方法会操作部分初始化的对象。如果在该方法中调用了被子类覆盖的非`final`方法，就可能访问到未初始化的数据。

注释：如果可序列化的类定义了字段

```java
@Serial private static final ObjectStreamField[] serialPersistentFields;
```

那么序列化就会使用这些字段描述符，而不是所有非静态、非瞬时字段。

除了让序列化机制来保存和恢复对象数据，类还可以定义它自己的机制。为此，这个类必须实现`Externalizable`接口（扩展了`Serializable`），这需要定义两个方法：

```java
public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException;
public void writeExternal(ObjectOutput out) throws IOException;
```

与`readObject()`和`writeObject()`方法不同，这些方法负责保存和恢复**整个对象**（包括超类数据），你可以加密数据或者使用比序列化格式更高效的格式。

对于可外部化对象，在写出对象时，序列化机制只在输出流中记录对象所属的类，然后调用`writeExternal()`方法；在读取对象时，对象输入流使用无参构造器创建一个对象，然后调用`readExternal()`方法。

注：对象输出流的`writeObject()`方法内部会判断：如果对象是可外部化的，则调用其`writeExternal()`方法（如果类有自定义的`writeObject()`方法将被忽略）。否则，按常规方式写出可序列化对象（使用类自定义的`writeObject()`方法或默认方式）。

在下面的示例中，`LabeledPixel`类扩展了可序列化的`Point`类，但是它接管了这个类及其超类的序列化。

```java
public class LabeledPixel extends Point implements Externalizable {
    private String label;

    public LabeledPixel() {} // required for externalizable class

    public void writeExternal(ObjectOutput out) throws IOException {
        out.writeInt((int) getX());
        out.writeInt((int) getY());
        out.writeUTF(label);
    }

    public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
        int x = in.readInt();
        int y = in.readInt();
        setLocation(x, y);
        label = in.readUTF();
    }
    ...
}
```

[serializationTweaks/LabeledPixel.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch02/serializationTweaks/LabeledPixel.java)

小结：自定义序列化机制的两种方式
* 定义`readObject()`/`writeObject()`方法：只负责读写当前类的数据。
* 实现`Externalizable`接口并定义`readExternal()`/`writeExternal()`方法：负责读写整个对象。

警告：`readObject()`和`writeObject()`方法是私有的，只能被序列化机制调用。与此不同的是，`readExternal()`和`writeExternal()`方法是公有的。特别是，`readExternal()`潜在地允许修改现有对象的状态。

注释：你不能自定义枚举和记录的序列化。即使在其中定义了`readObject()`/`writeObject()`或`readExternal()`/`writeExternal()`方法，它们也不会被用于序列化。

### 2.3.4 序列化单例和“枚举”类型
在序列化和反序列化ID很重要（即需要通过`==`判断相等）的对象时必须特别注意。例如单例(singleton)或具有有限数量实例的类。

如果使用Java的`enum`结构，就不必担心序列化，它能够正常工作。但是，假设你在维护遗留代码，其中包含下面这种“枚举”类型：

```java
public class Orientation {
    public static final Orientation HORIZONTAL = new Orientation(1);
    public static final Orientation VERTICAL = new Orientation(2);

    private int value;

    private Orientation(int v) {
        value = v;
    }
}
```

在Java语言添加`enum`之前，这种用法很常见。注意其构造器是私有的，除了`Orientation.HORIZONTAL`和`Orientation.VERTICAL`之外不可能创建其他实例。因此可以使用`==`运算符来测试相等：

```java
if (orientation == Orientation.HORIZONTAL) ...
```

对于这种类型，默认的序列化机制是不适用的。假设写出一个`Orientation`类型的值，并再次读回：

```java
Orientation original = Orientation.HORIZONTAL;
ObjectOutputStream out = ...;
out.write(original);
out.close();
ObjectInputStream in = ...;
var saved = (Orientation) in.read();
```

现在，测试`if (saved == Orientation.HORIZONTAL)`将失败。因为`saved`是一个全新的对象，它与两个预定义常量都不相等。即使构造器是私有的，序列化机制也能创建新对象！

可以使用另一种特殊的序列化方法`readResolve()`来解决这个问题。如果定义了该方法，它将在对象反序列化之后被调用，返回的对象将作为`readObject()`返回值。在我们的例子中，`Orientation.readResolve()`方法检查`value`字段并返回对应的枚举常量：

```java
@Serial
private Object readResolve() throws ObjectStreamException {
    if (value == 1) return Orientation.HORIZONTAL;
    else if (value == 2) return Orientation.VERTICAL;
    else throw new ObjectStreamException(); // this shouldn't happen
}
```

提示：对于单例类，也必须当心不要让反序列化产生第二个实例。使用枚举可以避免这个问题：

```java
public enum MySingleton {
    INSTANCE;
    MySingleton() { ... } // automatically private
    ...
}
```

枚举会被正确反序列化，因此可以确保只有一个`MySingleton.INSTANCE`。

有时，`readResolve()`方法与另一个特殊方法`writeReplace()`结合使用。该方法返回一个对象，**代替**当前对象写入对象流。这个替代对象必须是可序列化或可外部化的（不必是相同类型）。当读取对象流时，替代对象会被反序列化，其`readResolve()`方法会被调用来重新创建原始对象。

注释：`readObject()`/`writeObject()`方法必须是私有的，`readExternal()`/`writeExternal()`方法必须是公有的，而`readResolve()`/`writeReplace()`方法可以有任意的访问修饰符。

下面的示例展示了这种机制。

```java
public class ColoredPoint implements Serializable {
    private Color color;
    private Point location;

    public ColoredPoint(Color color, int x, int y) {
        this.color = color;
        this.location = new Point(x, y);
    }

    @Serial
    private Object writeReplace() throws ObjectStreamException {
        return new Ser(color.getRGB(), location.x, location.y);
    }

    private record Ser(int rgba, int x, int y) implements Serializable {
        @Serial
        private Object readResolve() throws ObjectStreamException {
            return new ColoredPoint(new Color(rgba), x, y);
        }
    }
}
```

[serializationTweaks/ColoredPoint.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch02/serializationTweaks/ColoredPoint.java)

`LocalDate`以及`java.time`包中的其他类都使用了这种策略。

### 2.3.5 版本管理
如果使用序列化来保存对象，就需要考虑在类演化（增删改字段）时会有什么问题。

Java序列化机制使用一个`static final long serialVersionUID`字段来标识类的序列化版本号。在反序列化时，会将对象流中的版本号与相应类的`serialVersionUID`进行比较，如果不一致则抛出`InvalidClassException`（见`ObjectStreamClass.initNonProxy()`方法），从而确保序列化和反序列化的兼容性。

类可以通过指定`serialVersionUID`字段来表明它与早期版本**兼容**。如果没有指定，就会根据类名、超类、方法和字段等信息使用复杂的算法计算出一个散列值（见`ObjectStreamClass.computeDefaultSUID()`方法）。可以使用`serialver`程序（在$jdk/bin目录中）获得这个数字（注：如果类文件不在当前目录中，需要通过`-classpath`选项指定类路径）：

```shell
$ serialver serial.Employee
serial.Employee:    private static final long serialVersionUID = 8367346051156850807L;
```

这个类的所有后续版本都必须将`serialVersionUID`常量定义为与最初版本相同。

```java
// version 1.1
public class Employee implements Serializable {
    @Serial public static final long serialVersionUID = 8367346051156850807L;
    ...
}
```

版本号匹配并不意味着反序列化一定会成功。如果只有类的方法发生了变化，那么读取新对象数据不会有任何问题。但是如果实例字段改变了（例如增删字段或改变类型），就可能会有问题。在这种情况下，对象输入流会尽力将序列化的对象转换为这个类的当前版本：
* 如果两个字段名字匹配但类型不同，则对象是不兼容的。
* 如果序列化的对象具有当前版本所没有的字段，则忽略这些字段。
* 如果当前版本具有序列化的对象所没有的字段，则将这些字段设置为默认值（对象为`null`，数字为0，布尔值为`false`）（同默认字段初始化，见卷I第4章 4.6.2节）。

下面是一个示例。假设已经使用`Employee`类的最初版本(1.0)在磁盘上保存了一些员工记录。现在通过添加一个`department`实例字段将`Employee`类修改为2.0版本。下图展示了将1.0对象读入使用2.0类的程序，`department`字段被设置为`null`。

![读取具有较少字段的对象](/assets/images/java-note-v2ch02-input-and-output/读取具有较少字段的对象.png)

下图展示了相反的情况：使用1.0类的程序读取2.0对象，额外的`department`字段被忽略。

![读取具有较多字段的对象](/assets/images/java-note-v2ch02-input-and-output/读取具有较多字段的对象.png)

这样处理是否安全要视情况而定。丢弃字段是无害的，因为接收者仍然拥有所有它知道如何处理的数据。但是将字段设置为`null`可能不那么安全。类设计者有责任在`readObject()`方法中实现额外的代码以修正版本不兼容问题，或者确保方法足够健壮能够处理`null`数据。

提示：在将`serialVersionUID`字段添加到类之前，需要问问自己为什么要让类可序列化。如果序列化只是用于短期持久化（例如应用服务器中的分布式方法调用），那么就不需要关心版本机制。如果扩展了一个可序列化的类，但不想持久化它的实例，那么同样无需关心。如果IDE给出警告，可以修改偏好设置，或者添加注解`@SuppressWarnings("serial")`。这样做比添加以后可能忘记修改的`serialVersionUID`更安全。

注释：枚举和记录会忽略`serialVersionUID`字段。枚举的版本号始终为0，记录在反序列化时不需要匹配版本号。

注释：在类演化过程中也可能添加超类，那么使用新版本的程序读取的对象流中超类的实例字段可能并未被设置。默认情况下，这些字段会被设置为默认值（0或`null`），这可能会使超类处于不安全的状态。超类可以通过定义一个初始化方法来防范此问题：

```java
@Serial private void readObjectNoData() throws ObjectStreamException
```

该方法应该设置与无参构造器相同的状态，或者抛出`InvalidObjectException`。它只会在读取的对象流包含的子类实例缺失超类数据的异常情况下被调用。

### 2.3.6 使用序列化进行克隆
序列化机制有一种有趣的用法：它提供了一种克隆对象的简单方法，只要类是可序列化的即可。只需将对象序列化到输出流，然后再读回。得到的新对象是原对象的深拷贝。你不必将对象写出到文件，可以使用`ByteArrayOutputStream`将数据保存到字节数组中。

如程序清单2-4所示，要想“免费”得到`clone()`方法，只需继承`SerialCloneable`类。应当注意的是，尽管这个方法很灵巧，但它比显式构造新对象并拷贝/克隆实例字段的克隆方法要慢得多。

[程序清单2-4 serialClone/SerialCloneTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch02/serialClone/SerialCloneTest.java)

### 2.3.7 反序列化和安全
在反序列化过程中，对象的创建没有调用该类的任何构造器，即使类有无参构造器也不会使用。字段值是直接用对象输入流中的值设置的。

注释：对于记录，反序列化会调用标准构造器，并传递来自对象输入流的各字段的值（因此，记录中的循环引用将无法恢复）。

绕过构造器存在安全风险。攻击者可以构造一些字节，描述（正常通过构造器）不可能构造出来的无效对象。例如，假设`Employee`构造器会在薪水为负数时抛出异常，我们自然就会认为没有任何`Employee`对象的薪水为负数。但是，正如在前面的小节中看到的，查看序列化对象的字节并修改其内容并不困难。通过这种方式，就可以构造出表示薪水为负数的员工的字节，然后将其反序列化。

为了解决这种问题，可序列化的类可以实现`ObjectInputValidation`接口并定义`validateObject()`方法来检查其对象是否被正确地反序列化。例如，`Employee`类可以检查薪水不能为负：

```java
public void validateObject() throws InvalidObjectException {
    if (salary < 0)
        throw new InvalidObjectException("salary < 0");
}
```

遗憾的是，这个方法不会被自动调用。为了调用它，还必须提供以下方法：

```java
@Serial
private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {
    in.registerValidation(this, 0);
    in.defaultReadObject();
}
```

当这个对象及其所有依赖对象都被加载之后，`validateObject()`方法才会被调用。第二个参数用于指定优先级，具有较高优先级的验证请求会先被执行。

[serializationTweaks/Employee.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch02/serializationTweaks/Employee.java)

[serializationTweaks/ObjectStreamTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch02/serializationTweaks/ObjectStreamTest.java)

还有其他的安全风险。对手可以创建十分消耗资源足以让虚拟机崩溃的数据结构。更阴险的是，由于类路径上的任何类都可以被反序列化，黑客一直在狡猾地拼凑“小工具链”(gadget chains)，并以诸如`Runtime.exec()`这样的方法结束（反序列化漏洞）。任何通过网络连接从不可信的来源接收序列化数据的应用程序都很容易遭受此类攻击。

应该避免对来自不可信来源的数据进行反序列化。JEP 290和415提供了一种序列化过滤器(serialization filter)机制，用来加固应用程序。过滤器可以看到被反序列化的类的名字和若干指标（流大小、数组大小、引用总数、最长引用链等）。基于这些数据决定是否终止反序列化。

在最简单的形式中，只需提供一个描述允许和禁止反序列化的类的模式。例如，如果像下面这样运行序列化示例程序

```shell
java -Djdk.serialFilter='serial.*;java.**;!*' serial.ObjectStreamTest
```

过滤器允许`serial`包中的所有类以及包名以`java`开头的所有类进行反序列化，其他类都禁止。

你还可以实现自己的过滤器，详见 <https://docs.oracle.com/en/java/javase/17/core/serialization-filtering1.html> 。

## 2.4 操作文件
`Path`接口和`Files`类封装了使用文件系统所需的功能。它们是在Java 7中添加的，使用起来比自从JDK 1.0就有的`File`类要方便得多。输入/输出流关心的是文件内容，而这里讨论的类关心的是文件在磁盘上的存储。

### 2.4.1 Path
`Path`表示一个文件系统路径，即一个目录名序列，其后还可以跟着一个文件名。路径的第一个部件可以是**根部件**(root component)，例如`/`或`C:\`。以根部件开始的路径是**绝对路径**，否则是**相对路径**。例如，下面构造了一个类UNIX文件系统中的绝对路径和相对路径：

```java
Path absolute = Path.of("/home", "harry"); // "/home/harry"
Path relative = Path.of("myprog", "conf", "user.properties"); // "myprog/conf/user.properties"
```

静态方法`Path.of()`接受一个或多个字符串，并将它们用默认文件系统的路径分隔符（UNIX是`/`，Windows是`\`）连接起来，然后解析结果，返回一个`Path`对象。如果结果不是给定文件系统中的合法路径（例如包含非法字符）则抛出`InvalidPathException`。

注：
* 对于Open JDK，在Windows上实现类为[WindowsPath](https://github.com/openjdk/jdk/blob/master/src/java.base/windows/classes/sun/nio/fs/WindowsPath.java)，在UNIX上实现类为[UnixPath](https://github.com/openjdk/jdk/blob/master/src/java.base/unix/classes/sun/nio/fs/UnixPath.java)。
* 在Java 8以前接口不允许有静态方法，因此只能使用`Paths.get()`方法创建`Path`对象。从Java 11起，`Path`接口提供了等价的`of()`方法，这样就不再需要`Paths`类了。

`of()`方法也可以接受包含多个部件的单个字符串。例如，可以如下从配置文件中读取路径：

```java
String baseDir = props.getProperty("base.dir");
  // May be a string such as "/opt/myprog" or "c:\Program Files\myprog"
Path basePath = Path.of(baseDir); // OK that baseDir has separators
```

注释：`Path`不必对应某个真实存在的文件或目录，它仅仅是一个抽象的路径。

组合或解析(resolve)路径是非常常见的。调用`p.resolve(q)`按照以下规则返回一个路径：如果`q`是绝对路径，则结果就是`q`；否则，结果是“`p`后面跟着`q`”（即拼接）。例如：

```java
Path basePath = Path.of("/opt/myapp");
Path workRelative = Path.of("work");
Path workPath = basePath.resolve(workRelative); // "/opt/myapp/work"
```

`resolve()`方法也可以接受一个字符串而不是路径：

```java
Path workPath = basePath.resolve("work");
```

`p.resolveSibling(q)`生成`p`的兄弟路径（大致等价于`p.getParent().resolve(q)`）。例如：

```java
Path tempPath = workPath.resolveSibling("temp"); // "/opt/myapp/temp"
```

与`resolve()`相反的是`relativize()`：如果`r = p.relativize(q)`，则`p.resolve(r) = q`。例如：

```java
Path p = Path.of("/home/harry");
Path q = Path.of("/home/fred/input.txt");
Path r = p.relativize(q); // "../fred/input.txt"
Path q2 = p.resolve(r).normalize(); // "/home/fred/input.txt"
```

`normalize()`方法用于规范化路径，删除所有的`.`（当前目录）和`..`（父目录）部件。例如，规范化路径`/home/harry/../fred/./input.txt`将生成`/home/fred/input.txt`。

`toAbsolutePath()`方法生成给定路径的绝对路径（基于当前工作目录解析的结果，大致等价于`Path.of(".").resolve(p)`）。

`Path`接口有许多有用的方法来拆分路径。下面的代码示例展示了一些最有用的方法：

```java
Path p = Path.of("/home", "fred", "myprog.properties");
Path parent = p.getParent(); // "/home/fred"
Path file = p.getFileName(); // "myprog.properties"
Path root = p.getRoot(); // "/"
```

在卷I第3章中已经看到，可以用`Path`对象构造`Scanner`：

```java
var in = new Scanner(Path.of("/home/fred/input.txt"));
```

注释：偶尔你可能需要与使用`File`类的遗留API交互。`Path`接口有一个`toFile()`方法，`File`类有一个`toPath()`方法。

### 2.4.2 读写文件
`Files`类可以快速完成常见的文件操作。例如，可以很容易地将文件的全部内容读入字节数组、字符串、行列表或行构成的流：

```java
byte[] bytes = Files.readAllBytes(path);
String content = Files.readString(path, charset);
List<String> lines = Files.readAllLines(path, charset);
Stream<String> lineStream = Files.lines(path, charset);
```

相反，对于写出：

```java
Files.write(path, bytes);
Files.writeString(path, content, charset);
Path.write(path, lines, charset);
```

为了向给定文件追加内容，使用

```java
Files.write(path, content, charset, StandardOpenOption.APPEND);
```

以下调用返回两个文件内容不同的第一个的字节位置：

```java
long pos = Files.mismatch(path1, path2);
```

为了探测文件的内容类型(MIME)（例如text/html或image/png），调用

```java
String mimeType = Files.probeContentType(path);
```

下面的方法使你可以与使用了输入/输出流或reader/writer的API进行交互：

```java
InputStream in = Files.newInputStream(path);
OutputStream out = Files.newOutputStream(path);
Reader in = Files.newBufferedReader(path, charset);
Writer out = Files.newBufferedWriter(path, charset);
```

### 2.4.3 创建文件和目录
要创建一个新目录，调用`Files.createDirectory(path)`，除了最后一个部件外的其他部件都必须已存在。要同时创建中间目录，使用`Files.createDirectories(path)`。

可以使用`Files.createFile(path)`创建一个空文件，如果文件已存在则抛出异常。检查和创建文件是原子性的。

`createTempFile()`和`createTempDirectory()`方法用于创建临时文件或目录。例如，以下调用可能会返回一个像`/tmp/1234405522364837194.txt`这样的路径。

```java
Path tmpFile = Files.createTempFile(null, ".txt");
```

在创建文件或目录时可以指定属性，例如拥有者或权限。细节取决于文件系统。

### 2.4.4 复制、移动和删除文件
要将文件从一个位置复制到另一个位置，只需调用`Files.copy(fromPath, toPath)`。

要移动文件（即复制并删除原文件），调用`Files.move(fromPath, toPath)`。

注：这两个方法也可用于复制和移动目录。但`copy()`方法只复制目录本身，不复制其中的文件和子目录；`move()`方法会移动整个目录。

如果目标路径已存在，复制或移动将会失败。如果想覆盖已存在的文件，则使用`REPLACE_EXISTING`选项。如果想复制所有的文件属性，则使用`COPY_ATTRIBUTES`选项。可以如下同时指定这两个选项：

```java
Files.copy(fromPath, toPath, StandardCopyOption.REPLACE_EXISTING, StandardCopyOption.COPY_ATTRIBUTES);
```

可以使用`ATOMIC_MOVE`选项指定移动操作是原子性的，这样就可以保证要么移动成功完成，要么文件保持在原来的位置。

```java
Files.move(fromPath, toPath, StandardCopyOption.ATOMIC_MOVE);
```

还可以将输入流复制到`Path`，或者将`Path`复制到输出流：

```java
Files.copy(inputStream, toPath);
Files.copy(fromPath, outputStream);
```

要删除文件，只需调用`Files.delete(path)`。如果文件不存在，这个方法会抛出异常。可以使用以下方法，仅当文件存在时才删除：

```java
boolean deleted = Files.deleteIfExists(path);
```

删除方法也可以用于删除空目录。

文件操作标准选项的完整列表参见`StandardOpenOption`和`StandardCopyOption`的API文档。

### 2.4.5 获取文件信息
下面的静态方法检查路径的某个属性，并返回一个布尔值：
* `exists()`
* `isHidden()`
* `isReadable()`, `isWritable()`, `isExecutable()`
* `isRegularFile()`, `isDirectory()`, `isSymbolicLink()`

`size()`方法返回文件的字节数。

`getOwner()`方法返回文件的拥有者（表示为`UserPrincipal`对象）。

`BasicFileAttributes`接口封装了基本文件属性，包括
* 创建时间、上次访问时间、上次修改时间（表示为`FileTime`对象）
* 文件是常规文件、目录、符号链接，或三者都不是
* 文件大小
* 文件主键（特定于文件系统，可能唯一标识文件，也可能不是）

要获得这些属性，调用

```java
BasicFileAttributes attributes = Files.readAttributes(path, BasicFileAttributes.class);
```

如果文件系统兼容POSIX，那么可以获得`PosixFileAttributes`实例：

```java
PosixFileAttributes attributes = Files.readAttributes(path, PosixFileAttributes.class);
```

然后就可以查询文件所属的用户组以及拥有者权限、组权限和其他用户权限（详见UNIX文件权限）。

### 2.4.6 访问目录项
静态方法`Files.list()`返回一个由目录中的项（文件和子目录）构成的`Stream<Path>`。目录是被惰性读取的，这使得处理具有大量项的目录更高效。读取目录涉及需要关闭的系统资源，因此应该使用`try`块：

```java
try (Stream<Path> entries = Files.list(pathToDirectory)) {
    ...
}
```

`list()`方法不会进入子目录。要递归处理所有子目录，应该使用`Files.walk()`方法。

```java
try (Stream<Path> entries = Files.walk(pathToRoot)) {
    // Contains all descendants, visited in depth-first order
}
```

可以通过`walk()`方法的第二个参数限制访问的深度（即最多访问几层子目录）。`walk()`方法还有一个`FileVisitOption...`类型的变长参数，但只有一种选项：`FOLLOW_LINKS`跟踪符号链接。

注释：如果要根据路径或文件属性过滤目录项，则应该使用`find()`方法而不是`walk()`。这样做的优势是高效。

这段代码使用了`Files.walk()`方法将一个目录复制到另一个目录：

```java
Files.walk(source).forEach(p -> {
    try {
        Path q = target.resolve(source.relativize(p));
        if (Files.isDirectory(p))
            Files.createDirectory(q);
        else
            Files.copy(p, q);
    }
    catch (IOException e) {
        throw new UncheckedIOException(ex);
    }
});
```

遗憾的是，你无法很容易地使用`Files.walk()`来删除目录树，因为在删除父目录之前必须先删除子目录。下一节将展示如何克服此问题。

### 2.4.7 使用目录流
`Files.newDirectoryStream()`方法生成一个`DirectoryStream<Path>`对象，遍历目录中的所有项。注意，`DirectoryStream`不是`Stream`的子接口，而是专门用于目录遍历的接口。它扩展了`Iterable`接口，因此可以在for each循环中使用目录流。使用模式如下：

```java
try (DirectoryStream<Path> entries = Files.newDirectoryStream(dir)) {
    for (Path entry : entries)
        // Process entries
}
```

访问目录项没有特定的顺序。

注：该方法和`list()`一样，也不进入子目录。

可以用glob模式来过滤文件：`Files.newDirectoryStream(dir, "*.java")`。下表展示了所有的glob模式。

| 模式 | 描述 | 示例 |
| --- | --- | --- |
| `*` | 匹配零个或多个字符，不可跨目录边界 | `*.java`匹配当前目录中的所有Java文件 |
| `**` | 匹配零个或多个字符，可跨目录边界 | `src/main/java/**/*.java`匹配src/main/java目录及其所有子目录中的Java文件 |
| `?` | 匹配一个字符 | `????.java`匹配所有4个字符（不包括扩展名）的Java文件 |
| `[]` | 匹配一组字符中的一个，可以使用连字符(`-`)和取反(`!`) | `Test[0-9A-F].java`匹配Testx.java，其中x是一个十六进制数字 |
| `{}` | 匹配逗号分隔的多个子模式之一 | `*.{java,class}`匹配所有Java文件和类文件 |
| `\` | 转义上述特殊字符以及`\` | `*\**`匹配所有文件名包含 * 的文件 |

警告：如果在Windows上使用glob语法，必须对反斜杠转义两次：一次为glob语法转义，一次为Java字符串转义。例如`Files.newDirectoryStream(dir, "C:\\\\")`。

如果想递归访问所有子目录，则应该调用`walkFileTree()`方法并提供一个实现了`FileVisitor<Path>`接口的对象。该接口具有以下方法：
* `FileVisitResult preVisitDirectory(Path dir, BasicFileAttributes attrs)` 在目录被访问前调用
* `FileVisitResult postVisitDirectory(Path dir, IOException exc)` 在目录被访问后调用
* `FileVisitResult visitFile(Path file, BasicFileAttributes attrs)` 访问文件时调用
* `FileVisitResult visitFileFailed(Path file, IOException exc)` 访问文件失败时调用

对于每种情况，都可以通过返回值指定接下来希望执行的操作：
* `CONTINUE`：继续访问下一个文件
* `SKIP_SUBTREE`：继续访问，但是跳过这个目录中的项
* `SKIP_SIBLINGS`：继续访问，但是跳过这个文件的兄弟（同一目录下的其他文件）
* `TERMINATE`：终止访问

注释：`FileVisitor`是一个泛型接口，但是不太可能会使用除`FileVisitor<Path>`之外的实例化。

`SimpleFileVisitor`类实现了`FileVisitor`接口，其所有方法直接返回`CONTINUE`或者重新抛出异常。可以扩展这个类并只实现感兴趣的方法。例如，下面的代码打印出给定目录的所有子目录：

```java
Files.walkFileTree(Path.of("/"), new SimpleFileVisitor<Path>() {
    public FileVisitResult preVisitDirectory(Path path, BasicFileAttributes attrs) throws IOException {
        System.out.println(path);
        return FileVisitResult.CONTINUE;
    }

    public FileVisitResult postVisitDirectory(Path dir, IOException e) {
        return FileVisitResult.CONTINUE;
    }

    public FileVisitResult visitFileFailed(Path path, IOException e) throws IOException {
        return FileVisitResult.SKIP_SUBTREE;
    }
});
```

[findDirectories/FindDirectories.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch02/findDirectories/FindDirectories.java)

下面是删除目录树的完整代码：

```java
// Delete the directory tree starting at root
Files.walkFileTree(root, new SimpleFileVisitor<Path>() {
    public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
        Files.delete(file);
        return FileVisitResult.CONTINUE;
    }

    public FileVisitResult postVisitDirectory(Path dir, IOException e) throws IOException {
        if (e != null) throw e;
        Files.delete(dir);
        return FileVisitResult.CONTINUE;
    }
});
```

### 2.4.8 ZIP文件系统
`Path`类在默认文件系统（用户本地磁盘）中查找路径。也可以有其他文件系统，其中最有用的之一是**ZIP文件系统**。调用

```java
FileSystem fs = FileSystems.newFileSystem(Path.of(zipname));
```

建立了一个文件系统，包含给定ZIP文档中的所有文件。

可以使用`Files.copy()`方法从ZIP文档中复制出文件：

```java
Files.copy(fs.getPath(sourceName), targetPath);
```

其中`fs.getPath()`可以类比为任意文件系统的`Path.of()`。

要列出ZIP文档中的所有文件，可以使用`Files.walkFileTree()`：

```java
FileSystem fs = FileSystems.newFileSystem(Path.of(zipname));
Files.walkFileTree(fs.getPath("/"), new SimpleFileVisitor<Path>() {
    public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
        System.out.println(file);
        return FileVisitResult.CONTINUE;
    }
});
```

这比2.2.3节中描述的API更好用。

[zip/ZipTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch02/zip/ZipTest.java)

## 2.5 内存映射文件
大多数操作系统都可以利用虚拟内存将文件或其一部分“映射”到内存，即**内存映射文件**(memory-mapped file)。然后这个文件就可以像内存数组一样访问，这比传统的文件操作快得多。

### 2.5.1 内存映射文件性能
本节末尾给出了一个分别使用传统文件和内存映射文件计算CRC32校验和的程序。在同一台机器上，对$jdk/lib目录中49 MB的src.zip文件用不同方法计算校验和，计时数据如下表所示。

| 方法 | 时间 |
| --- | --- |
| 普通输入流 | 110 s |
| 带缓冲输入流 | 9.9 s |
| 随机访问文件 | 102 s |
| 内存映射文件 | 7.2 s |

当然，精确值在不同机器上会有很大不同。但很明显，与随机访问相比，性能提升是巨大的。另一方面，对于中等大小文件的顺序读取则没有必要使用内存映射文件。

为了使用内存映射文件，首先获得一个文件**通道**(channel)。通道是对磁盘文件的抽象，允许访问内存映射、文件加锁和文件间快速数据传输等操作系统特性。

```java
FileChannel channel = FileChannel.open(path, options);
```

然后，调用`FileChannel`类的`map()`方法获得一个`ByteBuffer`，并指定想要映射的文件区域和**映射模式**(mapping mode)。支持三种模式：
* `READ_ONLY`：缓冲区是只读的，尝试写入会导致`ReadOnlyBufferException`。
* `READ_WRITE`：缓冲区是可写的，修改会在某个时刻写回到文件。注意，其他映射同一个文件的程序可能不会立即看到这些修改。多个程序同时进行文件映射的具体行为取决于操作系统。
* `PRIVATE`：缓冲区是可写的，但是任何修改都是这个缓冲区私有的，不会写回到文件。

例如：

```java
MappedByteBuffer buffer = channel.map(FileChannel.MapMode.READ_ONLY, 0, channel.size());
```

一旦有了缓冲区，就可以使用`ByteBuffer`类和`Buffer`超类的方法读写数据了。

缓冲区支持顺序和随机访问。例如，可以如下顺序遍历缓冲区中的所有字节：

```java
while (buffer.hasRemaining()) {
    byte b = buffer.get();
    ...
}
```

或者，可以使用随机访问：

```java
for (int i = 0; i < buffer.limit(); i++) {
    byte b = buffer.get(i);
    ...
}
```

还可以使用`get(byte[])`方法读取字节数组，`getInt()`、`getDouble()`等方法用于读取存储为二进制的基本类型值。前面（2.2.1节）已经提到，Java对二进制数据使用大端序。如果需要以小端序处理二进制文件，只需调用

```java
buffer.order(ByteOrder.LITTLE_ENDIAN);
```

要查询缓冲区当前的字节顺序，调用

```java
ByteOrder b = buffer.order();
```

要向缓冲区写入数据，使用`put()`、`putInt()`、`putDouble()`等方法。在某个时刻，以及当通道关闭时，这些改动会被写回到文件。

程序清单2-5用于计算文件的32位循环冗余校验和(CRC32)。这个校验和经常用于判断文件是否已损坏。

[程序清单2-5 memoryMap/MemoryMapTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch02/memoryMap/MemoryMapTest.java)

像这样运行程序：

```shell
java memoryMap.MemoryMapTest $JAVA_HOME/lib/src.zip
```

在实际中，将会以更大的块读取和更新数据，而不是每次一个字节，这样速度差异就没有这么大了（参见[memoryMap/MemoryMapTest2.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch02/memoryMap/MemoryMapTest2.java)）。

注：在个人计算机上测试结果如下

| 方法 | 单字节读取 | 按块读取(1024 B) |
| --- | --- | --- |
| 普通输入流 | 130.19 s | 0.18 s |
| 带缓冲输入流 | 0.35 s | 0.04 s |
| 随机访问文件 | 126.70 s | 0.16 s |
| 内存映射文件 | 0.12 s | 0.02 s |

### 2.5.2 缓冲区数据结构
缓冲区是由相同类型的值构成的数组。`Buffer`类是一个抽象类，具有`ByteBuffer`、`CharBuffer`、`DoubleBuffer`、`FloatBuffer`、`IntBuffer`、`LongBuffer`和`ShortBuffer`等子类。在实际中，最常用的是`ByteBuffer`和`CharBuffer`。

注释：`StringBuffer`类与这些缓冲区没有关系。

缓冲区具有以下属性，结构如下图所示。
* **容量**(capacity)：包含的元素个数，永远不会改变。
* **界限**(limit)：第一个不应该读或写的元素的索引。
* **位置**(position)：下一个要读或写的元素的索引。
* **标记**(mark)：可选，用于重复读或写操作。

这些值满足条件：0 ≤ 标记 ≤ 位置 ≤ 界限 ≤ 容量。

![缓冲区](/assets/images/java-note-v2ch02-input-and-output/缓冲区.png)

缓冲区的主要方法如下：
* `capacity()` 返回缓冲区的容量
* `limit()`, `limit(n)` 返回/设置缓冲区的界限
* `position()`, `position(n)` 返回/设置缓冲区的位置
* `clear()` 位置=0、界限=容量，使缓冲区准备好写入
* `flip()` 界限=位置、位置=0，使缓冲区准备好读取
* `rewind()` 位置=0、界限不变，使缓冲区准备好重新读取相同的值
* `mark()` 标记=位置
* `reset()` 位置=标记，从而允许再次读或写标记的部分
* `remaining()` 返回剩余值的数量（界限-位置）

缓冲区的主要用途是“写，然后读”循环。开始时，位置等于0，界限等于容量。不断地调用`put()`将值添加到缓冲区。当写完数据或达到容量时，就该切换到读操作了。调用`flip()`将界限设置为当前位置、位置置为0。当`remaining()`方法（返回界限-容量）返回正数时，不断地调用`get()`。当读完缓冲区中的所有值时，调用`clear()`将位置置为0、界限置为容量，使缓冲区准备好下一次写循环。

```java
Buffer buffer = ...;
while (/* cycle not ended */) {
    while (/* more data to write */) {
        var data = ...;
        buffer.put(data);
    }
    buffer.flip();
    while (buffer.hasRemaining()) {
        var data = buffer.get();
        // process data
    }
    buffer.clear();
}
```

要得到一个缓冲区，可以使用`allocate()`或`wrap()`等静态方法。然后可以用来自通道的数据填充缓冲区，或者将其内容写入通道。

```java
ByteBuffer buffer = ByteBuffer.allocate(RECORD_SIZE);
channel.read(buffer);
channel.position(newpos);
buffer.flip();
channel.write(buffer);
```

这种方法可以替代随机访问文件。

[randomAccess2/RandomAccessTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch02/randomAccess2/RandomAccessTest.java)

## 2.6 文件加锁
当多个同时执行的程序需要修改同一个文件时，这些程序需要以某种方式进行通信，否则文件很容易被损坏。**文件锁**可以解决这个问题，它可以控制对文件或文件内字节范围的访问。

注：卷I第12章讨论的是同一个进程内的**线程间同步**，这里的文件锁是一种**进程间同步**。

例如，假设应用程序将用户偏好保存在配置文件中。如果用户启动了这个应用的两个实例，这两个实例就有可能同时写这个配置文件。在这种情况下，第一个实例应该锁定文件。当第二个实例发现文件被锁定时，它可以决定等到文件解锁，或者直接跳过写操作。

要锁定一个文件，调用`FileChannel`类的`lock()`或`tryLock()`方法：

```java
FileChannel = FileChannel.open(path);
FileLock lock = channel.lock(); // or channel.tryLock()
```

`lock()`会阻塞直到锁可用；`tryLock()`会立即返回，如果锁不可用则返回`null`。这个文件将保持锁定状态，直到通道被关闭，或者对锁调用了`release()`方法。

也可以使用以下方法锁定文件的一部分：

```java
FileLock lock(long start, long size, boolean shared)
FileLock tryLock(long start, long size, boolean shared)
```

如果`shared`为`false`则是互斥锁，当前进程可以读写锁定的文件；否则是共享锁，允许多个进程读取文件，并阻止任何进程获得互斥锁。并非所有的操作系统都支持共享锁，即使请求的是共享锁也可能得到互斥锁。可以调用`FileLock`类的`isShared()`方法查询锁类型。

注释：如果锁定了文件的尾部，而文件随后增长并超过了锁定部分，那么额外的区域是未锁定的。要锁定所有字节，使用`Long.MAX_VALUE`作为`size`参数。

要确保在操作完成时释放锁，最好使用带资源的`try`语句：

```java
try (FileLock lock = channel.lock()) {
    // access the locked file or segment
}
```

要记住，文件加锁是依赖于操作系统的。下面是需要注意的几点：
* 在某些系统中，文件加锁仅仅是建议性的(advisory)。如果一个程序未能得到锁，它可能仍然能写入另一个程序已锁定的文件。
* 在某些系统中，不能在锁定文件的同时将其映射到内存。
* 文件锁是由整个Java虚拟机持有的。同一个虚拟机上的并发任务不能获得同一个文件上重叠的锁，否则将抛出`OverlappingFileLockException`。
* 在某些系统中，关闭通道会释放底层文件上的所有锁。因此应该避免在同一个锁定的文件上使用多个通道。
* 锁定网络文件系统上的文件是高度依赖于系统的，应该尽量避免。

## 2.7 正则表达式
**正则表达式**(regular expression)用于指定字符串模式。如果需要查找与特定模式匹配的字符串，就可以使用正则表达式。

下面几节将介绍Java API使用的正则表达式语法，以及如何使用正则表达式。

### 2.7.1 正则表达式语法
下面从一个简单的例子开始。正则表达式`[Jj]ava.+`匹配任何满足以下条件的字符串：
* 第一个字母是J或j
* 接下来三个字母是ava
* 其余部分是一个或多个任意字符

例如，字符串 "javanese" 匹配这个正则表达式，但是 "Core Java" 不匹配。

你需要知道语法才能理解正则表达式的含义。幸运的是，对于大多数情况，几个简单的语法结构就足够了。

在正则表达式中，字符表示其自身，除非是保留字符：

```
. * + ? { | ( ) [ \ ^ $
```

例如，正则表达式`Java`只能匹配字符串 "Java" 。

符号`.`匹配任意单个字符。例如，`.a.a`可以匹配 "Java" 和 "data" 。

符号`*`表示前面的结构可以重复0次或多次，`+`表示重复1次或多次，`?`表示前面的结构是可选的（重复0次或1次），`{n}`表示重复n次。例如，`be+s?`可以匹配 "be" 、 "bee" 和 "bees" 。

`|`表示匹配左右两个模式之一。例如，`beef|woof`匹配 "beef" 或 "woof" 。

**字符类**(character class)是`[]`括起来的一组候选字符，例如`[Jj]`。在字符类内部，`-`表示范围（Unicode值落在两个边界之间的所有字符），例如`[0-9]`或`[A-Za-z]`。但是，如果`-`是第一个或最后一个字符则表示其自身。如果`^`是字符类中的第一个字符则表示补集（除了指定字符之外的所有字符），例如`[^0-9]`。

有许多**预定义字符类**，例如`\d`（数字）和`\p{Sc}`（Unicode货币符号）。见下（2）（7）表格。

字符`^`和`$`匹配行的开头和结尾。

如果需要保留字符的字面意思，则需要转义，即在前面添加反斜杠。在字符类内部，只需要转义`[`和`\`，但是要小心`] - ^`的位置。例如，`[]^-]`是只包含这三个字符的类。

或者，可以用`\Q`和`\E`把字符串括起来。例如，`\(\$0\.99\)`和`\Q($0.99)\E`都匹配字符串 "($0.99)" 。

提示：也可以调用`Pattern.quote()`进行转义。这个方法直接把字符串用`\Q`和`\E`括起来，但是会处理字符串包含`\E`的特殊情况。

下面的表格列出了常用的正则表达式语法，完整语法参见[`Pattern`类API文档](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/regex/Pattern.html)。

其他相关文档：
* [Lesson: Regular Expressions - The Java Tutorials](https://docs.oracle.com/javase/tutorial/essential/regex/index.html)
* [Regular-Expressions.info](https://www.regular-expressions.info/)

（1）字符

| 表达式 | 描述 | 示例 |
| --- | --- | --- |
| 除保留字符之外的字符 | 字符本身 | `J` |
| `.` | 任意字符（不匹配行终止符，除非设置了`DOTALL`标志） | |
| `\x{p}` | 具有给定十六进制Unicode码点的字符 | `\x{1D546}` |
| `\0o`, `\0oo`, `\0ooo`, `\xhh`, `\uhhhh` | 具有给定八进制或十六进制UTF-16码元的字符 | `\uFEFF` |
| `\a`, `\e`, `\f`, `\n`, `\r`, `\t`, `\\` | 响铃符、转义符、换页符、换行符、回车符、制表符、反斜杠 | |

（2）字符类

| 表达式 | 描述 | 示例 |
| --- | --- | --- |
| <code>[C<sub>1</sub>C<sub>2</sub>...]</code><br>其中C<sub>i</sub>是字符、范围或字符类 | 任何由C<sub>1</sub>, C<sub>2</sub>, ...表示的字符 | `[0-9abc]` |
| `[^...]` | 字符类的补集 | `[^\d\s]` |
| `[...&&...]` | 字符类的交集 | `[\p{L}&&[^A-Za-z]]` |
| `\p{...}`, `\P{...}`<br>名称为单个字母可以省略花括号 | 预定义字符类；它的补集 | `\p{L}`或`\pL`匹配一个Unicode字母 |
| `\d`, `\D` | 数字(`[0-9]`)；非数字 | `\d+`匹配数字序列 |
| `\w`, `\W` | 单词字符(`[a-zA-Z0-9_]`)；非单词字符 | |
| `\s`, `\S` | 空白符(`[ \n\r\t\f\xB]`)；非空白符 | |
| `\h`, `\H` | 水平空白符；非水平空白符 | |
| `\v`, `\V` | 垂直空白符；非垂直空白符 | |

（3）序列和选择

| 表达式 | 描述 | 示例 |
| --- | --- | --- |
| `XY` | `X`后面跟着`Y` | `[1-9][0-9]*`匹配没有前导零的正整数 |
| `X|Y` | `X`或`Y` | `http|ftp` |

（4）分组

| 表达式 | 描述 | 示例 |
| --- | --- | --- |
| `(X)` | 捕获`X`的匹配 | `'([^']*)'`捕获单引号括起来的文本 |
| `\n` | 第n组 | `(['"]).*\1`匹配`'Fred'`和`"Fred"`，但不匹配`"Fred'` |
| `(?<name>X)` | 使用给定的名字捕获`X`的匹配 | `(?<id>[A-Za-z0-9]+)` |
| `\k<name>` | 具有给定名字的组 | `\k<id>` |
| `(?:X)` | 使用分组但不捕获`X`的匹配 | `(?:http|ftp)://(.*)`，其中`.*`是分组1 |

（5）量词

| 表达式 | 描述 | 示例 |
| --- | --- | --- |
| `X?` | 可选的`X` | `\+?`匹配可选的+号 |
| `X*` | `X`重复0次或多次 | |
| `X+` | `X`重复1次或多次 | `[1-9][0-9]+`匹配≥10的整数 |
| `X{n}`, `X{n,}`, `X{m,n}` | `X`重复n次、至少n次、m到n次 | `[0-7]{1,3}`匹配1到3位八进制数 |
| `Q?`，其中`Q`是量词表达式 | 懒惰量词(reluctant quantifier)，尝试尽可能短的匹配 | `<.+?>`匹配最短的XML标签 |
| `Q+`，其中`Q`是量词表达式 | 占有量词(possessive quantifier)，在不回溯的情况下获取最长匹配 | |

贪婪、懒惰和占有量词的区别参见：
* [Quantifiers - The Java Tutorials](https://docs.oracle.com/javase/tutorial/essential/regex/quant.html)
* [Greedy vs. Reluctant vs. Possessive Qualifiers](https://stackoverflow.com/questions/5319840/greedy-vs-reluctant-vs-possessive-qualifiers)

例如，对于输入字符串 "xfooxxxxxxfoo"
* `.*foo`（贪婪量词）：一个匹配，<font color="#FF0000">xfooxxxxxx</font>foo（其中红色表示量词匹配的部分）
* `.*?foo`（懒惰量词）：两个匹配，<font color="#FF0000">x</font>foo和<font color="#FF0000">xxxxxx</font>foo
* `.*+foo`（占有量词）：无匹配，因为`.*+`匹配了整个字符串且不会回溯，`foo`无法匹配成功

（6）边界匹配

| 表达式 | 描述 | 示例 |
| --- | --- | --- |
| `^`, `$` | 行的开头和结尾 | `^Java$`匹配行 "Java"  |
| `\A`, `\Z`, `\z` | 输入的开头、输入的结尾、输入的绝对结尾 | |
| `\b`, `\B` | 单词边界；非单词边界 | `\bJava\b`匹配单词Java |
| `\R` | Unicode行分隔符 | |
| `\G` | 前一个匹配的结尾 | |

（7）预定义字符类（用于`\p{...}`）

| 名字 | 描述 | 示例 |
| --- | --- | --- |
| `posixClass` | posixClass是`Lower`, `Upper`, `ASCII`, `Alpha`, `Digit`, `Alnum`,<br>`Punct`, `Graph`, `Print`, `Blank`, `Cntrl`, `XDigit`, `Space`之一 | `\p{Digit}` |
| `IsScript`, `sc=Script`, `script=Script` | Script是`UnicodeScript.forName()`可接受的脚本名称 | `\p{sc=Hiragana}` |
| `InBlock`, `blk=Block`, `block=Block` | Block是`UnicodeBlock.forName()`可接受的块名称 | `\p{blk=Mongolian}` |
| `Category`, `IsCategory`, `gc=Category`,<br>`general_category=Category` | Category是[Unicode通用分类](https://www.unicode.org/reports/tr44/#General_Category_Values)的单字母或双字母名称 | `\p{L}` |
| `IsProperty` | Property是`Alphabetic`, `Ideographic`, `Letter`, `Lowercase`, `Uppercase`,<br>`Titlecase`, `Punctuation`, `Control`, `White_Space`, `Digit`, `Hex_Digit`,<br>`Join_Control`, `Noncharacter_Code_Point`, `Assigned`之一 | `\p{IsAlphabetic}` |
| `javaMethod` | 调用`Character.isMethod()`方法（不能是已弃用的） | `\p{javaLowerCase}` |

参见：
* [POSIX Bracket Expressions](https://www.regular-expressions.info/posixbrackets.html)
* [Unicode Characters and Properties](https://www.regular-expressions.info/unicode.html)

### 2.7.2 匹配整个字符串
有两种方式使用正则表达式：
* 检查某个字符串是否与正则表达式匹配
* 找出字符串中所有与正则表达式匹配的部分

对于第一种情况，可以使用静态方法`Pattern.matches()`：

```java
String regex = "\\d+";
String input = ...;
if (Pattern.matches(regex, input)) {
    ...
}
```

如果需要多次使用同一个正则表达式，那么更高效的方式是编译它，然后为每个输入创建一个`Matcher`：

```java
Pattern pattern = Pattern.compile(regex);
Matcher matcher = pattern.matcher(input);
if (matcher.matches()) {
    ...
}
```

如果想要匹配集合或流中的字符串，可以使用`asPredicate()`方法将模式转换为谓词（等价于`s -> matcher(s).find()`）：

```java
Stream<String> strings = ...;
List<String> result = strings.filter(pattern.asPredicate()).toList();
```

结果中的字符串都**包含**正则表达式的匹配（即匹配某个子串）。如果要匹配整个字符串，则使用`pattern.asMatchPredicate()`（等价于`s -> matcher(s).matches()`）。

### 2.7.3 找出字符串中的所有匹配
正则表达式的另一种常见用法是找出输入中的所有匹配。可以使用以下循环：

```java
String input = ...;
Matcher matcher = pattern.matcher(input);
while (matcher.find()) {
    String match = matcher.group();
    int matchStart = matcher.start();
    int matchEnd = matcher.end();
    ...
}
```

通过这种方式，可以依次处理每个匹配：获得匹配的字符串以及在输入字符串中的位置。

更优雅地，可以调用`results()`方法获得一个`Stream<MatchResult>`。`MatchResult`接口有`group()`、`start()`和`end()`方法（实际上`Matcher`类实现了这个接口）。可以像这样获得所有匹配的列表：

```java
List<String> matches = pattern.matcher(input)
    .results()
    .map(Matcher::group)
    .toList();
```

如果数据在文件中，那么可以使用`Scanner.findAll()`方法得到一个`Stream<MatchResult>`，而无需先将内容读取到一个字符串中。可以传递一个`Pattern`对象或模式字符串：

```java
Scanner in = new Scanner(path, "UTF_8");
Stream<String> words = in.findAll("\\pL+")
    .map(MatchResult::group);
```

程序清单2-6使用了这种机制。程序查找一个网页中的所有超链接(hypertext reference, href)并打印出来。要运行这个程序，需要在命令行提供一个URL，例如：

```shell
java match.HrefMatch https://horstmann.com
```

[程序清单2-6 match/HrefMatch.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch02/match/HrefMatch.java)

### 2.7.4 分组
使用**分组**(group)来提取匹配的各个部分是很常见的。例如，假设发票上的一行包含商品名称、货币和价格：

```
Blackwell Toaster    USD29.95
```

使用下面的正则表达式匹配：

```
(\w+(\s+\w+)*)\s+([A-Z]{3})([0-9.]*)
```

可以使用`group(n)`提取第n个分组匹配的字符串：

```java
String contents = matcher.group(n);
```

注：`group()`等价于`group(0)`。调用`group()`或`group(n)`之前必须先调用`matches()`或`find()`。

使用`start()`和`end()`方法可以获得分组在输入中的位置。

分组按左括号的位置排序，索引从1开始。分组0是整个输入。在上面的例子中，有4个分组：

| 分组索引 | 开始 | 结束 | 字符串 |
| --- | --- | --- | --- |
| 0 | 0 | 29 | "Blackwell Toaster&nbsp;&nbsp;&nbsp;&nbsp;USD29.95" |
| 1 | 0 | 17 | "Blackwell Toaster" |
| 2 | 9 | 17 | "&nbsp;Toaster" |
| 3 | 21 | 24 | "USD" |
| 4 | 24 | 29 | "29.95" |

在这个例子中，可以像这样提取商品名称、货币和价格：

```java
if (matcher.matches()) {
    String item = matcher.group(1);
    String currency = matcher.group(3);
    String price = matcher.group(4);
    // ...
}
```

注：下面是使用[正则表达式测试工具](https://regex101.com/)的测试结果：

![正则表达式测试结果](/assets/images/java-note-v2ch02-input-and-output/正则表达式测试结果.png)

![正则表达式测试结果-分组](/assets/images/java-note-v2ch02-input-and-output/正则表达式测试结果-分组.png)

我们对分组2并不感兴趣，它只是为了将`\s+\w+`作为整体重复。为了更清楚，可以使用非捕获分组：

```
(\w+(?:\s+\w+)*)\s+([A-Z]{3})([0-9.]*)
```

或者可以按名字捕获：

```
(?<item>\w+(\s+\w+)*)\s+(?<currency>[A-Z]{3})(?<price>[0-9.]*)
```

这样就可以按名字获取分组：

```java
String item = matcher.group("item");
```

警告：按名字获取分组只能用于`Matcher`，不能用于`MatchResult`。

注释：当重复中包含分组时（例如上面例子中的`(\s+\w+)*`），无法获得其所有匹配。`group()`方法只会返回最后一个匹配，这几乎没有什么用处。你需要用另一个分组捕获整个表达式。

提示：可以在正则表达式中添加注释。注释以`#`开头，直到该行末尾。这特别适合文本块：

```java
var regex = """
([1-9]|1[0-2]) # hours
:([0-5][0-9]) # minutes
[ap]m""";
```

程序清单2-7中的程序提示输入一个模式和待匹配的字符串，然后打印出输入是否与模式匹配。如果输入匹配并且模式包含分组，则用括号打印出分组边界，例如 "((11):(59))am" 。

[程序清单2-7 regex/RegexTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch02/regex/RegexTest.java)

### 2.7.5 用匹配分割
`Pattern.split()`方法将输入用模式的匹配分割，返回字符串数组。

```java
String input = "1, 2, 3";
Pattern commas = Pattern.compile("\\s*,\\s*");
String[] tokens = commas.split(input); // ["1", "2", "3"]
```

如果结果很长，可以惰性地获取：

```java
Stream<String> tokens = commas.splitAsStream(input);
```

也可以直接使用`String.split()`方法：

```java
String[] tokens = input.split("\\s*,\\s*");
```

注：`s.split(p)`大致等价于`Pattern.compile(p).split(s)`。

如果输入在文件中，可以使用`Scanner`：

```java
Scanner in = new Scanner(path, "UTF_8");
in.useDelimiter("\\s*,\\s*");
Stream<String> tokens = in.tokens();
```

### 2.7.6 替换匹配
如果想要将正则表达式的所有匹配替换为某个字符串，可以使用`Matcher`类的`replaceAll()`方法：

```java
Matcher matcher = commas.matcher(input);
String result = matcher.replaceAll(",");
  // Normalizes the commas: "1,  2 ,3" -> "1,2,3"
```

或者也可以使用`String`类的`replaceAll()`方法。

```java
String result = input.replaceAll("\\s*,\\s*", ",");
```

替换字符串可以包含分组编号`$n`或名字`${name}`，它们会被替换为对应分组捕获的内容。

```java
String result = "3:45".replaceAll(
    "(\\d{1,2}):(?<minutes>\\d{2})",
    "$1 hours and ${minutes} minutes");
  // Sets result to "3 hours and 45 minutes"
```

需要使用`\`来转义替换字符串中的`$`和`\`，或者可以使用便捷方法`Matcher.quoteReplacement()`：

```java
matcher.replaceAll(Matcher.quoteReplacement(str))
```

如果想要执行更复杂的操作，可以提供一个替换函数，该函数接受一个`MatchResult`，返回一个字符串。例如，下面的代码将所有至少具有4个字母的单词替换为其大写形式：

```java
String result = Pattern.compile("\\pL{4,}")
    .matcher("Mary had a little lamb")
    .replaceAll(m -> m.group().toUpperCase());
  // Yields "MARY had a LITTLE LAMB"
```

`replaceFirst()`方法只替换第一个匹配。

### 2.7.7 标志
**标志**(flag)可以改变正则表达式的行为。可以在编译模式时指定标志：

```java
Pattern pattern = Pattern.compile(regex,
    Pattern.CASE_INSENSITIVE | Pattern.UNICODE_CHARACTER_CLASS);
```

或者在模式内指定：

```java
String regex = "(?iU:expression)";
```

支持的标志如下表所示。

| `Pattern`类常量 | 标志字符 | 含义 |
| --- | --- | --- |
| `CASE_INSENSITIVE` | `i` | 匹配时不区分大小写 |
| `UNICODE_CASE` | `u` | 与`i`组合使用时，用Unicode字母大小写来匹配 |
| `UNICODE_CHARACTER_CLASS` | `U` | 选择Unicode而不是POSIX字符类，蕴含了`u` |
| `MULTILINE` | `m` | `^`和`$`匹配行的开头和结尾，而不是整个输入的 |
| `UNIX_LINES` | `d` | 在多行模式中匹配`^`和`$`时，只有`'\n'`是行终止符 |
| `DOTALL` | `s` | `.`匹配所有字符，包括行终止符 |
| `COMMENTS` | `x` | 允许空白符和注释 |
| `LITERAL` | | 按字面意思解释模式，必须精确匹配（大小写除外） |
| `CANON_EQ` | | 考虑Unicode字符的规范等价性（例如u¨匹配ü） |
