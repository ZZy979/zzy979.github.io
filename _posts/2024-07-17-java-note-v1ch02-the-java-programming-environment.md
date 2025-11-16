---
title: 《Java核心技术》笔记 卷I 第2章 Java编程环境
date: 2024-07-17 22:39:06 +0800
categories: [Java, Core Java]
tags: [java, hello world, jshell]
---
本章主要介绍如何安装Java开发工具包(JDK)以及如何编译和运行Java程序。

## 2.1 安装Java开发工具包
过去，Oracle公司提供了最新、最完整的JDK版本。如今，很多不同的公司都为Linux、macOS和Windows提供了最新的[OpenJDK](https://openjdk.org/)构建版本，有些公司的许可证条件比Oracle更宽松。

| 发行版 | 官方网站 |
| --- | --- |
| Oracle JDK | [链接](https://www.oracle.com/java/technologies/downloads/) |
| Oracle OpenJDK | [链接](https://jdk.java.net/) |
| Adoptium OpenJDK (Eclipse Temurin) | [链接](https://adoptium.net/) |
| Azul Zulu OpenJDK | [链接](https://www.azul.com/downloads/?package=jdk) |
| Liberica JDK | [链接](https://bell-sw.com/libericajdk/) |
| Microsoft Build of OpenJDK | [链接](https://www.microsoft.com/openjdk) |
| Amazon Corretto | [链接](https://aws.amazon.com/corretto/) |
| IBM Semeru Runtimes | [链接](https://developer.ibm.com/languages/java/semeru-runtimes/downloads/) |
| Alibaba Dragonwell JDK | [链接](https://dragonwell-jdk.io/) |
| Tencent Kona | [链接](https://github.com/Tencent/TencentKona-17) |
| SapMachine | [链接](https://sap.github.io/SapMachine/) |
| Red Hat OpenJDK | [链接](https://developers.redhat.com/products/openjdk/overview) |

注：如何选择JDK可参考[Which Version of JDK Should I Use?](https://whichjdk.com/)。

### 2.1.1 下载JDK
可以从Oracle网站 <https://www.oracle.com/java/technologies/downloads/> 下载Java开发工具包，也可以选择其他提供商。应该使用Java SE 17 (LTS) JDK。下表总结了在下载网站上可能遇到的术语和缩略语。

| 术语 | 缩写 | 解释 |
| --- | --- | --- |
| Java Development Kit（Java开发工具包） | JDK | 编写Java程序的程序员使用的软件 |
| Java Runtime Environment（Java运行时环境） | JRE | 用于运行Java程序的软件，不带开发工具 |
| Standard Edition（标准版） | SE | 用于桌面或简单服务器应用的Java平台 |
| Micro Edition（微型版） | ME | 用于小型设备的Java平台 |
| OpenJDK | - | Java SE的一个免费开源实现 |
| Hotspot | - | Oracle开发的Java虚拟机(JVM) |
| OpenJ9 | - | IBM开发的另一个Java虚拟机 |
| Long Term Support（长期支持版本） | LTS | 很多年都支持的版本，不同于每6个月发布的展示新特性的版本 |

注意：
* 你需要的是JDK而不是JRE。
* 在Windows上，下载.exe或.msi；在Linux和macOS上，下载.tar.gz。

### 2.1.2 设置JDK
下载JDK之后，需要安装，并记住安装位置。
* 在Windows上，启动安装程序。最好不要接受默认安装位置（例如C:\Program Files\Java\jdk-17），因为路径名中有空格。可以选择其他位置（例如D:\Java\jdk-17）。
* 在Linux或macOS上，只需要把.tar.gz文件解压到某个位置（例如/usr/local/jdk-17）。

在本书中，安装目录用 *$jdk* 表示（注意是包含bin子目录的那个目录）。

安装JDK后，还需要将$jdk/bin目录（包含可执行文件java.exe和javac.exe）添加到PATH环境变量——系统查找可执行文件时所遍历的目录列表。

* 在Linux或macOS上，在~/.bashrc或~/.bash_profile文件的最后增加两行：

```shell
export JAVA_HOME=$jdk
export PATH=$PATH:$JAVA_HOME/bin
```

* 在Windows上，右键点击此电脑 → 属性 → 高级系统设置 → 环境变量。在系统变量中新建一个名为JAVA_HOME的变量，值为$jdk目录。之后编辑Path变量，添加一行 "%JAVA_HOME%\bin" 。如下图所示。

![Windows中设置JAVA_HOME环境变量](/assets/images/java-note-v1ch02-the-java-programming-environment/Windows中设置JAVA_HOME环境变量.png)

![Windows中设置Path环境变量](/assets/images/java-note-v1ch02-the-java-programming-environment/Windows中设置Path环境变量.png)

可以如下测试设置是否正确。打开一个终端窗口，输入

```shell
javac --version
```

然后按回车键。应该能看到显示以下信息：

```
javac 17.0.12
```

### 2.1.3 安装源代码和文档
标准库源文件以压缩文件$jdk/lib/src.zip的形式随JDK发布。解压缩这个文件以访问源代码。

文档包含在一个独立于JDK的压缩文件中，可以从 <https://www.oracle.com/java/technologies/javase-jdk17-doc-downloads.html> 下载。解压后在浏览器中打开index.html即可。

注：在线文档
* [Java SE文档](https://docs.oracle.com/en/java/javase/)
  * [Java SE 17文档](https://docs.oracle.com/en/java/javase/17/index.html)
  * [JDK 17 API文档](https://docs.oracle.com/en/java/javase/17/docs/api/index.html)
  * [Java语言新特性](https://docs.oracle.com/en/java/javase/17/language/java-language-changes.html)
* [JDK命令行工具文档](https://docs.oracle.com/en/java/javase/17/docs/specs/man/index.html)
  * [`javac`命令文档](https://docs.oracle.com/en/java/javase/17/docs/specs/man/javac.html)
  * [`java`命令文档](https://docs.oracle.com/en/java/javase/17/docs/specs/man/java.html)
* [Java语言和虚拟机规范](https://docs.oracle.com/javase/specs/index.html)
  * [Java SE 17语言规范](https://docs.oracle.com/javase/specs/jls/se17/html/index.html)
  * [Java SE 17虚拟机规范](https://docs.oracle.com/javase/specs/jvms/se17/html/index.html)
* [Java教程](https://docs.oracle.com/javase/tutorial/)

还需要本书的程序示例，可以从 <https://horstmann.com/corejava/> 下载。这些程序打包在一个压缩文件corejava.zip中，将其解压缩，得到corejava目录。

## 2.2 使用命令行工具
首先介绍如何从命令行编译并运行Java程序。
1. 打开一个终端窗口。
2. 进入corejava/v1ch02/Welcome目录（corejava是上一节安装本书示例源代码的目录）。
3. 输入以下命令：

```shell
javac Welcome.java
java Welcome
```

然后，应该会在终端窗口中看到如下图所示的输出。

Windows:

![编译并运行Welcome-Windows.java](/assets/images/java-note-v1ch02-the-java-programming-environment/编译并运行Welcome.java-Windows.png)

Linux:

![编译并运行Welcome.java-Linux](/assets/images/java-note-v1ch02-the-java-programming-environment/编译并运行Welcome.java-Linux.png)

你已经编译并运行了第一个Java程序。那么刚才都发生了什么？`javac`程序是Java编译器，它将文件Welcome.java编译成Welcome.class。`java`程序启动Java虚拟机，执行类文件(.class)中的字节码。

Welcome程序非常简单，它只是向终端输出了一条消息，如代码清单2-1所示。下一章将解释它是如何工作的。

[程序清单2-1 Welcome/Welcome.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch02/Welcome/Welcome.java)

一定要注意以下几点：
* 如果手动输入源程序，一定要确保大小写正确。例如，类名为`Welcome`，而不是`welcome`或`WELCOME`。
* 编译器需要一个**文件名**(Welcome.java)；而运行程序时需要指定**类名**(Welcome)，不带扩展名.java或.class。
* 如果看到诸如“ 'javac' 不是内部或外部命令，也不是可运行的程序或批处理文件”或 "javac: command not found" 之类的消息，就要返回去检查安装是否有问题，特别是可执行路径的设置。
* 如果`javac`报错无法找到文件Welcome.java，就应该检查当前目录中是否存在这个文件。在Linux中，使用命令`ls`；在Windows中，使用命令`dir`。
* 如果运行程序之后，收到关于`java.lang.NoClassDefFoundError`的错误消息，就应该仔细检查出问题的类名。

提示：在 <https://docs.oracle.com/javase/tutorial/getStarted/cupojava/> 上有一个很好的教程，其中更详细地介绍了初学者容易犯的一些错误。

注释：如果只有一个源文件，可以跳过`javac`命令（即可以直接执行`java Welcome.java`）。这个Java 11新特性是为了用于Shell脚本（以 "shebang" 行`#!/path/to/java`开头）或简单的学生程序。

注：`javac`和`java`命令的输出默认使用系统本地语言。要改为英文，需要将系统属性`user.language`设置为`en`：

```shell
javac -J-Duser.language=en Welcome.java
java -Duser.language=en Welcome
```

另外也可以设置环境变量JAVA_TOOL_OPTIONS：

```shell
export JAVA_TOOL_OPTIONS=-Duser.language=en
javac Welcome.java
java Welcome
```

这个Welcome程序没有太大意思。接下来再尝试一个图形化应用。这个程序是一个简单的图像文件查看器，可以加载并显示图像。与之前一样，从命令行编译和运行这个程序。
1. 打开一个终端窗口。
2. 切换到目录corejava/v1ch02/ImageViewer。
3. 输入以下命令：

```shell
javac ImageViewer.java
java ImageViewer
```

会弹出一个新的程序窗口（ImageViewer应用）。选择File → Open，打开一个图像文件（在同一个目录下有几个示例文件），然后会显示这个图像，如下图所示。要关闭程序，可以点击标题栏上的关闭按钮，或者从菜单选择File → Exit。

![运行ImageViewer应用](/assets/images/java-note-v1ch02-the-java-programming-environment/运行ImageViewer应用.png)

可以简单浏览源代码（程序清单2-2）。这个程序比第一个程序长多了，但只要想一想用C或C++编写类似的应用所需要的代码量，就不会觉得它复杂了。当然，如今编写带图形用户界面的桌面应用并不常见。不过，如果你感兴趣，可以在第10章学习更多详细内容。

[程序清单2-2 ImageViewer/ImageViewer.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch02/ImageViewer/ImageViewer.java)

## 2.3 使用集成开发环境
在上一节中，你已经了解了如何从命令行编译和运行一个Java程序。这是一个很有用的技能，不过对于大多数日常工作来说，还是应当使用集成开发环境(integrated development environment, IDE)。这些环境非常强大，也很方便。有很多优秀的免费IDE可供选择，如Eclipse、IntelliJ IDEA（社区版）和NetBeans。

### Eclipse
首先从 <https://www.eclipse.org/downloads/packages/installer> 下载Eclipse。

注意：从2024-06 (4.32)版本开始，Eclipse需要Java 21+。要使用Java 17，必须选择2024-03 (4.31)及以前的版本，可以从历史版本列表 <https://www.eclipse.org/downloads/packages/release> 下载。

运行安装程序，并选择 "Eclipse IDE for Java Developers" 。

![安装Eclipse-选择安装类型](/assets/images/java-note-v1ch02-the-java-programming-environment/安装Eclipse-选择安装类型.png)

安装程序会自动检测系统中已安装的JDK。选择安装位置，之后点击Install按钮即可开始安装。

![安装Eclipse-选择安装位置](/assets/images/java-note-v1ch02-the-java-programming-environment/安装Eclipse-选择安装位置.png)

下面是用Eclipse编写程序的步骤。

1.启动Eclipse之后，从菜单选择File → New → Project。

2.从向导对话框中选择 "Java Project" ，如下图所示。

![Eclipse中的新建项目对话框](/assets/images/java-note-v1ch02-the-java-programming-environment/Eclipse中的新建项目对话框.png)

3.点击Next按钮。取消勾选 "Use default location" 。点击Browse，选择corejava/v1ch02/Welcome目录。取消勾选 "Create module-info.java file" 。如下图所示。

![配置Eclipse项目](/assets/images/java-note-v1ch02-the-java-programming-environment/配置Eclipse项目.png)

4.点击Finish按钮，项目创建完成。

5.在左侧窗格中找到Welcome.java并双击，应该看到包含程序代码的窗口，如下图所示。

![使用Eclipse编辑源文件](/assets/images/java-note-v1ch02-the-java-programming-environment/使用Eclipse编辑源文件.png)

6.用鼠标右键单击左侧窗格中的项目名(Welcome)，选择Run As → Java Application。程序输出会显示在控制台窗格中。

### IntelliJ IDEA
IntelliJ IDEA可以从 <https://www.jetbrains.com.cn/idea/download/> 下载（社区版是免费的）。创建Java项目参考官方文档[Create your first Java application](https://www.jetbrains.com.cn/en-us/help/idea/2024.1/creating-and-running-your-first-java-application.html)。

注意：如果像上一节那样将corejava/v1ch02/Welcome目录作为项目打开，则需要为每个示例程序都创建一个项目，很繁琐。而如果直接打开corejava目录，IntelliJ IDEA可能无法识别项目目录结构，导致代码自动补全和跳转等功能不可用。为了解决这一问题，可以将每个示例目录标记为源代码根目录。例如，在corejava/v1ch02/Welcome目录上点击鼠标右键，选择Mark Directory as → Sources Root。对于包含包声明的示例（例如corejava/v1ch05/arrays/CopyOfTest.java），需要将包所在目录（例如corejava/v1ch05）标记为源代码根目录。参见官方文档[Folder categories](https://www.jetbrains.com.cn/en-us/help/idea/2024.1/content-roots.html#folder-categories)。

![使用IntelliJ IDEA编辑源文件](/assets/images/java-note-v1ch02-the-java-programming-environment/使用IntelliJ IDEA编辑源文件.png)

注：上述方法在卷II第9章会遇到问题，因为无法创建多个module-info.java文件。一种好的方法是将每章的子目录分别当作一个[模块](https://www.jetbrains.com/help/idea/creating-and-managing-modules.html)（见[提交59513c2](https://github.com/ZZy979/Core-Java-code/commit/59513c258cdb18af6d8f8b851a92099f2a80cea1)），卷II第9章中的每个Java模块同时也是一个IntelliJ IDEA模块。

## 2.4 JShell
Java 9引入了另一种使用Java的方法——JShell。JShell程序提供了一个“读取-求值-打印循环”(Read-Evaluate-Print Loop, REPL)（类似于Python交互式解释器和Scala REPL）。输入一个Java表达式，JShell会求值，打印结果，并等待下一个输入。

要启动JShell，只需在终端窗口中输入`jshell`，如下图所示。

![运行JShell](/assets/images/java-note-v1ch02-the-java-programming-environment/运行JShell.png)

JShell首先显示问候语，后面是一个提示符：

```
|  Welcome to JShell -- Version 17.0.12
|  For an introduction type: /help intro

jshell>
```

现在输入一个表达式，例如

```java
"Core Java".length()
```

JShell会回应一个结果——在这里是字符串`"Core Java"`中的字符个数。

```
$1 ==> 9
```

输出中的`$1`表示这个结果可用于进一步的计算。例如，如果输入

```java
5 * $1 - 3
```

结果为

```
$2 ==> 42
```

如果需要多次使用一个变量，可以给它指定一个名字。例如，

```java
jshell> int answer = 6 * 7
answer ==> 42
```

另一个有用的特性是Tab补全。输入

```java
Math.
```

然后按Tab键，会得到`Math`类的所有方法的列表。

```
jshell> Math.
E                 IEEEremainder(    PI                abs(
absExact(         acos(             addExact(         asin(
atan(             atan2(            cbrt(             ceil(
class             copySign(         cos(              cosh(
decrementExact(   exp(              expm1(            floor(
floorDiv(         floorMod(         fma(              getExponent(
hypot(            incrementExact(   log(              log10(
log1p(            max(              min(              multiplyExact(
multiplyFull(     multiplyHigh(     negateExact(      nextAfter(
nextDown(         nextUp(           pow(              random()
rint(             round(            scalb(            signum(
sin(              sinh(             sqrt(             subtractExact(
tan(              tanh(             toDegrees(        toIntExact(
toRadians(        ulp(
jshell> Math.
```

现在输入`l`，然后再按一次Tab键。方法名会补全为`log`，然后得到一个比较短的列表：

```
jshell> Math.l
log(     log10(   log1p(
jshell> Math.log
```

接下来你可以手动输入其余的代码：

```java
jshell> Math.log10(0.001)
$3 ==> -3.0
```

要重复一个命令，可以连续按↑键，直到找到想要重新运行或编辑的行。可以用←和→键移动光标，然后增加和删除字符，完成后再按回车键。例如，将0.001替换为1000，然后按回车键：

```java
jshell> Math.log10(1000)
$4 ==> 3.0
```

JShell让Java语言和库的学习变得轻松而有趣，而无需启动一个庞大的开发环境，不会再让你为`public static void main`而困扰。

输入`/help intro`查看帮助，输入`/exit`退出JShell。
