---
title: Blade构建工具
date: 2022-01-20 02:05:17 +0800
categories: [Build Tools]
tags: [build tools, blade]
---
## 1.简介
Blade是腾讯开发的一个开源构建工具，旨在简化大型项目的构建，能够自动分析依赖，集成了编译、链接、测试、静态代码检查等功能，支持C/C++, Java, Python, Scala, protobuf等多种语言（主要面向C/C++）（借鉴自[Bazel](https://bazel.build/)）。

注意：构建(build)和编译(compile)不同——编译器负责将源代码转换为库文件或可执行文件；构建工具负责分析构建目标之间的依赖关系，并调用编译器来生成构建目标。

例如，自己的代码依赖A库，A库又依赖B库，如果手动编译则需要写复杂的编译和链接命令，当依赖库代码发生变化时还需要重新编译，构建工具旨在自动化这一过程。

* 项目主页：<https://github.com/chen3feng/blade-build>
* 官方文档：<https://github.com/chen3feng/blade-build/blob/master/doc/en/README.md>
* 用户手册：<https://github.com/chen3feng/blade-build/blob/master/doc/blade_user_manual.pdf>
* 介绍ppt：<https://github.com/chen3feng/blade-build/blob/master/doc/blade.pdf>

特性：
* 自动分析库之间的依赖关系
* 递归构建：当依赖库的源文件发生变化时会自动重新构建依赖库，而GNU Make无法实现递归构建
* 增量构建：未发生变化的依赖库不会重新构建，加快构建速度
* 提供了对protobuf和测试（使用gtest）的内置支持

## 2.依赖软件
Blade需要以下依赖：
* Linux或macOS操作系统
* Python 2.7
* Ninja 1.8+

构建特定语言所需的编译器：
* C/C++: GCC 4.0+
* Java: JDK 1.6+
* Scala: 2.10+

## 3.安装
### 3.1 安装Python
Linux或macOS系统默认已经安装了Python 2.7。

### 3.2 安装Ninja
[Ninja](https://ninja-build.org/)是一个小型构建系统，Blade将其作为底层构建工具使用。

下载地址：<https://github.com/ninja-build/ninja/releases>

解压后只有一个可执行文件ninja，将其放到PATH环境变量包含的某个目录下（例如/usr/local/bin），从而能够直接在命令行中直接执行ninja命令：

```bash
$ ninja --version
1.10.2
```

### 3.3 安装Blade
安装方式：下载源代码，执行install脚本

```bash
$ git clone https://github.com/chen3feng/blade-build.git
$ cd blade-build/
$ git checkout v2.0
$ ./install
```

执行完成后Blade将被安装在~/bin目录下，该目录也被添加到PATH环境变量，执行`source ~/.profile`命令或重启终端使其生效，此时应该能够在命令行中直接执行blade命令：

```bash
$ blade -h
usage: blade [-h] [--version] {build,run,test,clean,query,dump} ...

blade <subcommand> [options...] [targets...]
...
```

注意：不能删除blade-build目录，因为`blade`命令会用到其中的源代码。

## 4.简单示例
下面使用Blade创建一个Hello World项目。

官方文档：[Quick Start](https://github.com/chen3feng/blade-build/blob/master/doc/en/quick_start.md)

### 4.1 创建工作目录
首先创建项目根目录blade-demo，并在根目录下创建一个文件BLADE_ROOT和一个子目录quick-start：

```bash
mkdir blade-demo && cd blade-demo
touch BLADE_ROOT
mkdir quick-start
```

其中项目根目录blade-demo可以在任意位置，BLADE_ROOT文件用于标识项目根目录。

### 4.2 实现say库
在quick-start目录下创建say.h和say.cc两个文件，内容如下：

say.h

```cpp
#pragma once
#include <string>

// Say a message
void Say(const std::string& msg);
```

say.cc

```cpp
#include "quick-start/say.h"

#include <iostream>

void Say(const std::string& msg) {
    std::cout << msg << "!\n";
}
```

这两个文件组成了一个库(library)，可以将其编译为库文件供其他代码使用。创建一个BUILD文件来描述say库：

```
cc_library(
    name = 'say',
    srcs = 'say.cc',
    hdrs = 'say.h',
)
```

其中`cc_library`表示该构建目标是一个C++库，`srcs`为源文件，`hdrs`为公共接口头文件。

### 4.3 实现hello库
下面创建另一个库hello，并调用say库提供的函数。

在quick-start目录下创建hello.h和hello.cc两个文件，内容如下：

hello.h

```cpp
#pragma once
#include <string>

// Say hello to `to`
void Hello(const std::string& to);
```

hello.cc

```cpp
#include "quick-start/hello.h"

#include "quick-start/say.h"

void Hello(const std::string& to) {
    Say("Hello, " + to);
}
```

其中函数`Hello()`调用了say库提供的函数`Say()`，因此**hello库依赖say库**。

在BUILD文件中添加hello库的定义：

```
cc_library(
    name = 'hello',
    srcs = 'hello.cc',
    hdrs = 'hello.h',
    deps = ':say',
)
```

其中`deps`为该构建目标的依赖，`:say`表示当前BUILD文件中名为say的构建目标，即前面的say库。

### 4.4 实现hello_world程序
下面创建一个hello_world程序，在`main()`函数中调用hello库提供的函数来打印信息。

创建源文件hello_world.cc，内容如下：

```cpp
#include "quick-start/hello.h"

int main() {
    Hello("World");
    return 0;
}
```

在BUILD文件中添加hello_world的定义：

```
cc_binary(
    name = 'hello_world',
    srcs = 'hello_world.cc',
    deps = ':hello',
)
```

`cc_binary`表示该构建目标是一个可执行程序。

注意：依赖只需要添加`:hello`，而不需要添加`:say`，因为这是hello库的实现细节，在编译和链接过程中Blade会自动处理这样的传递依赖。但是，如果hello_world.cc中显式包含了say.h，则依赖中需要添加`:say`。

下面构建并运行hello_world程序：

```bash
$ cd quick-start
$ blade build :hello_world
$ blade run :hello_world
Hello, World!
```

`blade build`命令底层了调用g++编译器和ld链接器，可使用`--verbose`参数查看具体执行的命令，生成的库文件和可执行文件在build64_release目录下。

注：对于这个简单的示例，直接执行

```bash
g++ -o hello_world hello_world.cc hello.cc say.cc
```

即可完成编译，但是对于具有成百上千个源文件、包含很多模块的大型项目，使用构建工具就很有必要了。

完整的项目目录结构如下：

```
blade-demo/
    BLADE_ROOT
    quick-start/
        BUILD
        say.h
        say.cc
        hello.h
        hello.cc
        hello_world.cc
    build64_release/     # Blade自动创建
        quick-start/
            libsay.a     # say库
            libhello.a   # hello库
            hello_world  # 可执行程序
            ...
```

注：该示例只有quick-start一个子目录和一个BUILD文件。实际的项目会按模块将文件分为多个不同的子目录，每个子目录下都包含一个BUILD文件。

完整代码：<https://github.com/chen3feng/blade-build/blob/master/example/quick-start>

## 5.代码组织结构
Blade要求项目有一个显式的根目录，即BLADE_ROOT文件所在目录。根目录下是自己的模块子目录和第三方库目录，每个子目录下都有一个BUILD文件来声明该模块所包含的构建目标。

以下是一个示例目录结构：

```
my-project/
    BLADE_ROOT
    common/
        string/
            BUILD
            algorithm.h
            algorithm.cc
    foo/
        BUILD
        foo.h
        foo.cc
    thirdparty/
        gtest/
            BUILD
            gtest.h
            gtest.cc
            ...
```

注：这是Google推荐的源代码管理方式——将所有代码放在同一个仓库中，包括第三方库源代码。

Blade会自动将项目根目录添加到头文件搜索路径，因此`#include`包含头文件的相对路径是基于项目根目录的，例如`#include "quick-start/say.h"`包含的是blade-demo/quick-start/say.h。虽然对于相同目录下的头文件来说这样有些繁琐（写成`#include "say.h"`即可），但对于跨目录包含的头文件来说这样可以清楚地知道头文件所在位置。例如，在上面的目录结构中，要在foo.cc中包含algorithm.h，直接写`#include "common/string/algorithm.h"`即可，而不必写成`#include "../common/string/algorithm.h`。

## 6.BUILD文件
Blade通过名为BUILD（全部大写）的文件声明构建目标，构建目标可以是库、可执行文件、测试等，由若干源文件（和头文件）组成。

构建目标可以相互依赖。在BUILD文件中只需要声明目标的**直接**依赖，Blade将自动分析传递依赖关系，并调用编译器和链接器来生成构建目标，构建一个目标时会先构建其依赖的目标。

### 6.1 示例
假设common/string目录下定义了一些字符串辅助函数，并且依赖common/int目录下的int库，则common/string/BUILD文件如下：

```
cc_library(
    name = 'string',
    srcs = [
        'algorithm.cc',
        'concat.cc',
        'format.cc',
    ],
    hdrs = [
        'algorithm.h',
        'concat.h',
        'format.h',
    ],
    deps = [
        '//common/int:int',
    ],
)
```

其他构建目标通过`'//common/string:string'`引用该目标。

注：当`srcs`、`hdrs`或`deps`有多个时，可以使用列表`[]`（BUILD文件实际上就是Python源代码，声明构建目标就是函数调用）

### 6.2 风格建议
* 缩进4个空格
* 使用单引号而不是双引号
* 目标名称使用小写
* `srcs`中的文件按字母顺序排列
* `deps`先写当前目录下的依赖(`:name`)，再写其他目录下的依赖(`//path/to/dir:name`)，按字母顺序排列
* 当每行一个参数时，最后一个参数也以逗号结尾，从而减少当增加或删除参数时影响的行数
* 不同目标之间空一行，每个目标前添加注释，注释以`#`开头

### 6.3 构建目标
Blade支持多种语言，每种语言支持多种构建目标。以下是几种常用的构建目标的语法。

完整列表参考：[Write a BUILD file - Build Rules](https://github.com/chen3feng/blade-build/blob/master/doc/en/build_file.md#build-rules)

公共属性：
* `name`：字符串，指定构建目标的名称，和路径一起构成目标的唯一标识
* `srcs`：字符串列表，指定源文件，位于当前目录或当前目录的子目录下，可使用[glob](https://github.com/chen3feng/blade-build/blob/master/doc/en/functions.md#glob)函数
* `hdrs`：字符串列表，指定公共接口头文件
* `deps`：字符串列表，指定依赖目标，支持以下格式：
  * `//path/to/dir:name`：项目根目录下path/to/dir/BUILD文件中声明的名为name的目标
  * `:name`：当前BUILD文件中名为name的目标
  * `#name`：系统库，例如`#pthread`将添加链接选项`-lpthread`
* `visibility`：字符串列表，仅对列出的目标可见（格式见7.2节），可使用特殊值`'PUBLIC'`指定对所有目标可见
  * 在Blade 2中，**目标默认是私有的**，即只对当前目录下的目标可见

注：类型为字符串列表的属性如果只有一项则可省略中括号，例如`srcs = ['foo.cc']`等价于`srcs = 'foo.cc'`。

#### 6.3.1 C++
##### 6.3.1.1 cc_library
构建C++库文件。语法（仅包含了常用属性）：

```
cc_library(
    name = 'foo',
    srcs = ['foo.cc', 'bar.cc', ...],
    hdrs = ['foo.h', 'bar.h', ...],
    deps = [':name', '//path/to/dir:name', ...],
)
```

注：
* 公共接口头文件应声明在`hdrs`中，私有头文件应声明在`srcs`中。
* 如果通过`#include`包含了一个头文件，则应该将该头文件所属的库添加到`deps`中。
* 默认只生成静态链接库文件(.a)，如果在`blade build`时指定`--generate-dynamic`选项，或依赖该目标的`cc_binary`指定了`dynamic_link = True`，则生成动态链接库文件(.so)。

##### 6.3.1.2 cc_binary
构建C++可执行程序。语法：

```
cc_binary(
    name = 'foo',
    srcs = ['foo.cc', 'bar.cc', ...],
    deps = [':name', '//path/to/dir:name', ...],
    dynamic_link = False,
)
```

* `dynamic_link`：布尔值（默认为`False`），如果为`True`则使用动态链接，生成的可执行文件较小，但启动较慢
  * 注：如果使用动态链接，则生成的可执行文件必须使用`blade run`执行，或者在项目根目录下执行，否则会找不到库文件

##### 6.3.1.3 cc_test
构建C++单元测试，使用[GoogleTest](https://google.github.io/googletest/)测试框架，本质上就是自动链接了`gtest`和`gtest_main`的`cc_binary`。语法：

```
cc_test(
    name = 'foo_test',
    srcs = 'foo_test.cc',
    deps = [':foo', ...],
    testdata = testdata = ['data1.txt', '//path/to/data2.txt'],
)
```

* `testdata`：字符串列表，测试代码只能访问列表指定的文件（例如测试数据）

具体用法见8.1节。

#### 6.3.2 Protobuf
##### 6.3.2.1 proto_library
构建[protobuf](https://developers.google.com/protocol-buffers)库，使用protoc编译器。语法：

```
proto_library(
    name = 'foo_proto',
    srcs = 'foo.proto',
    deps = [':bar_proto', ...],
    target_languages = ['cpp', 'java', 'python'],
)
```

* `target_languages`：字符串列表，生成指定语言的源代码。默认只生成C++代码，当protobuf库被其他构建目标依赖，或者`blade build`指定了`--generate-*`选项时也会生成对应语言的代码。例如，如果被`java_library`目标依赖，或指定了`--generate-java`选项，则会生成Java代码，对应protoc编译器的`--java_out`选项。

具体示例见[Protocol Buffers入门教程]({% post_url 2022-04-26-protocol-buffers-tutorial %}) 3.1.7.1 (3)和4.1.2节。

#### 6.3.3 Java
（官方文档很不全，很多细节根本没有说明）

##### 6.3.3.1 maven_jar
表示Maven仓库中的一个jar文件。语法：

```
maven_jar(
  name = 'commons-lang3',
  id = 'org.apache.commons:commons-lang3:3.12.0',
  transitive = True
)
```

* `id`：字符串，指定Maven id，格式为groupId:artifactId:version，见[Maven Naming Conventions](https://maven.apache.org/guides/mini/guide-naming-conventions.html)
* `transitive`：布尔值（默认为`True`），指定该目标被打包进fat jar时是否包含传递依赖，不影响编译和测试分析传递依赖

直接构建该目标将什么都不做，只有构建其他依赖该目标的目标时才会下载jar文件。

##### 6.3.3.2 java_library
从Java源代码构建jar文件。语法：

```
java_library(
    name = 'Foo',
    srcs = ['Foo.java', 'Bar.java', ...],
    resources = ['resources/foo.conf', ...],
    deps = [':name', '//path/to/dir:name', ...],
    exported_deps = [':name', '//path/to/dir:name', ...],
    provided_deps = [':name', '//path/to/dir:name', ...],
)
```

* `resources`：字符串列表，指定要打包进jar的资源文件
* `deps`：字符串列表，指定依赖目标，**无传递性**
* `exported_deps`：字符串列表，指定导出依赖目标，**有传递性**
* `provided_deps`：字符串列表，指定**由运行环境提供**的依赖（例如Hadoop、Spark等）
  * 如果当前目标被`java_binary`、`java_test`、`java_fat_library`或`scala_fat_library`目标依赖或传递依赖，则`provided_deps`及其上游依赖不会被打包进上述目标中。

生成的jar文件名为{name}.jar，仅包含类文件，**不包含依赖**。`srcs`和`resources`支持[glob函数](https://github.com/chen3feng/blade-build/blob/master/doc/en/functions.md#glob)。三种依赖的区别详见6.3.3.6节。

##### 6.3.3.3 java_binary
从Java源代码构建可执行jar文件，包含依赖。语法：

```
java_binary(
    name = 'Foo',
    srcs = ['Foo.java', 'Bar.java', ...],
    resources = ['resources/foo.conf', ...],
    deps = [':name', '//path/to/dir:name', ...],
    main_class = 'foo.Foo',
    exclusions = ['org.slf4j:*:*', 'org.apache.hadoop:*:*', ...],
)
```

* `main_class`：字符串，指定程序入口类（全名）
* `exclusions`：字符串列表，指定要排除的依赖库，格式同Maven id，支持通配符`*`

生成的jar文件名为{name}.one.jar，包含类文件、依赖、传递依赖以及资源文件各自生成的jar。

[One-JAR](https://one-jar.sourceforge.net/)是一个用于将Java应用及其依赖打包为单个可执行jar文件的开源项目。构建`java_binary`目标需要提前下载[one-jar-boot-0.97.jar](https://sourceforge.net/projects/one-jar/files/one-jar/one-jar-0.97/one-jar-boot-0.97.jar/download)，并将其所在路径添加到BLADE_ROOT文件的`one_jar_boot_jar`配置中：

```
java_binary_config(
    one_jar_boot_jar = '/path/to/one-jar-boot-0.97.jar'
)
```

否则会报错 "Blade(error): Blade build tool java_onejar error: [Errno 2] No such file or directory: ''"。

##### 6.3.3.4 java_fat_library
从Java源代码构建fat jar文件。语法：

```
java_fat_library(
    name = 'Foo',
    srcs = ['Foo.java', 'Bar.java', ...],
    resources = ['resources/foo.conf', ...],
    deps = [':name', '//path/to/dir:name', ...],
    exclusions = ['org.slf4j:*:*', 'org.apache.hadoop:*:*', ...],
)
```

生成的jar文件名为{name}.fat.jar。

[Fat jar](https://stackoverflow.com/questions/11947037/what-is-an-uber-jar)（也叫uber jar）即包含依赖的jar，类似于Maven的[jar-with-dependencies](https://maven.apache.org/plugins/maven-assembly-plugin/descriptor-refs.html#jar-with-dependencies)。Fat jar与one-jar的区别是：fat jar将所有子jar的内容提取出来聚合成一个超级jar，而one-jar中每个子jar单独存在。

通过`exclusions`和上游`java_library`依赖的`provided_deps`可以排除最终打包到fat jar中的依赖库，可以减小fat jar文件的大小，避免与运行环境的依赖冲突。

##### 6.3.3.5 java_test
从Java源代码构建[JUnit](https://junit.org)测试jar文件。语法：

```
java_test(
    name = 'FooTest',
    srcs = ['FooTest.java', ...],
    resources = ['resources/foo.conf', ...],
    deps = [':Foo', ...],
    exclusions = ['org.slf4j:*:*', 'org.apache.hadoop:*:*', ...],
    testdata = testdata = ['data1.txt', '//path/to/data2.txt'],
)
```

具体用法见8.2节。

##### 6.3.3.6 Java的依赖处理
* 编译依赖 = 直接依赖 + 直接依赖的导出依赖 + 导出依赖的递归导出依赖 + Maven依赖的传递依赖
* 测试依赖 = 直接依赖 + 直接依赖的所有传递依赖 + Maven依赖的传递依赖
* 打包依赖 = 直接依赖 - provided依赖 + (直接依赖 - provided依赖)的打包依赖 - exclusions

##### 6.3.3.7 示例
下面用Java实现第4节中的Hello World示例。

首先在BLADE_ROOT中添加配置：

```
java_config(
    java_home = os.environ['JAVA_HOME'],
    source_encoding = 'utf-8',
)
```

Say.java

```java
package hello;

public class Say {
  public static void say(String msg) {
    System.out.println(msg);
  }
}
```

Hello.java

```java
package hello;

public class Hello {
  public static void hello(String to) {
    Say.say("Hello, " + to);
  }
}
```

HelloWorld.java

```java
package hello;

public class HelloWorld {
  public static void main(String[] args) {
    Hello.hello("world");
  }
}
```

BUILD

```
java_library(
    name = 'Say',
    srcs = 'Say.java',
)

java_library(
    name = 'Hello',
    srcs = 'Hello.java',
    deps = ':Say',
)

java_binary(
    name = 'HelloWorld',
    srcs = 'HelloWorld.java',
    deps = ':Hello',
    main_class = 'hello.HelloWorld',
)

java_fat_library(
    name = 'HelloWorldFat',
    srcs = 'HelloWorld.java',
    deps = ':Hello',
)
```

BLADE_ROOT中需要添加`one_jar_boot_jar`配置，如6.3.3.3节所述。

目录结构：

```
blade-demo/
    BLADE_ROOT
    java/
        hello/
            BUILD
            Say.java
            Hello.java
            HelloWorld.java
```

在blade-demo/java/hello目录下执行

```bash
blade build :HelloWorld :HelloWorldFat
```

则在blade-demo/build64_release/java/hello目录下会生成HelloWorld.one.jar和HelloWorldFat.fat.jar两个文件：

```bash
$ jar -tf HelloWorld.one.jar
META-INF/
META-INF/MANIFEST.MF
OneJar.class
com/simontuffs/onejar/JarClassLoader.class
...
main/HelloWorld.jar
lib/Hello.jar
lib/Say.jar

$ jar -tf HelloWorldFat.fat.jar
META-INF/
hello/
hello/HelloWorld.class
hello/Hello.class
hello/Say.class
META-INF/blade/JAR.LIST
META-INF/blade/MERGE-INFO
META-INF/MANIFEST.MF
```

要运行程序，可以使用`blade run`命令，直接执行one-jar，或者使用fat jar+手动指定类路径。可以在blade-demo/java/hello目录下执行

```bash
$ blade run :HelloWorld
...
Blade(info): Run '['.../blade-demo/build64_release/java/hello/HelloWorld']'
Hello, world
```

或者在blade-demo/build64_release/java/hello目录下执行

```bash
$ java -jar HelloWorld.one.jar 
Hello, world

$ java -cp Say.jar:Hello.jar:HelloWorld.jar hello.HelloWorld
Hello, world

$ java -cp HelloWorldFat.fat.jar hello.HelloWorld
Hello, world
```

注：
* `deps`不具有传递性：如果`HelloWorld`类直接使用了`Say`类，则必须将`:Say`也添加到目标`HelloWorld`的依赖中，或者将`:Say`添加到目标`Hello`的`exported_deps`中。
* 如果目标`Hello`的依赖`:Say`声明为`provided_deps`，则Say.class不会被打包到HelloWorldFat.fat.jar中，因此最后一种运行方式需要将Say.jar也添加到类路径。
* 该示例并没有采用[Maven标准目录结构](https://maven.apache.org/guides/introduction/introduction-to-the-standard-directory-layout.html)。尽管习惯上按照包名组织Java源代码的目录结构，但Java编译器并没有强制要求（但输出的.class文件的目录结构必须与包名一致）。

#### 6.3.4 Scala
Scala构建目标和Java构建目标基本一致，但没有`scala_binary`。

##### 6.3.4.1 scala_library
从Scala源代码构建jar文件。语法：

```
scala_library(
    name = 'Foo',
    srcs = ['Foo.scala', 'Bar.scala', ...],
    resources = ['resources/foo.conf', ...],
    deps = [':name', '//path/to/dir:name', ...],
    exported_deps = [':name', '//path/to/dir:name', ...],
    provided_deps = [':name', '//path/to/dir:name', ...],
)
```

##### 6.3.4.2 scala_fat_library
从Scala源代码构建fat jar文件。语法：

```
scala_fat_library(
    name = 'Foo',
    srcs = ['Foo.scala', 'Bar.scala', ...],
    resources = ['resources/foo.conf', ...],
    deps = [':name', '//path/to/dir:name', ...],
    exclusions = ['org.slf4j:*:*', 'org.apache.hadoop:*:*', ...],
)
```

##### 6.3.4.3 scala_test
从Scala源代码构建[ScalaTest](https://www.scalatest.org)测试jar文件。语法：

```
scala_test(
    name = 'FooTest',
    srcs = ['FooTest.scala', ...],
    resources = ['resources/foo.conf', ...],
    deps = [':Foo', ...],
    exclusions = ['org.slf4j:*:*', 'org.apache.hadoop:*:*', ...],
    testdata = testdata = ['data1.txt', '//path/to/data2.txt'],
)
```

具体用法见8.3节。

##### 6.3.4.4 示例
下面用Scala实现第4节中的Hello World示例。

首先在BLADE_ROOT中添加配置：

```
scala_config(
    scala_home = os.environ['SCALA_HOME'],
    source_encoding = 'utf-8',
)
```

Say.scala

```scala
package hello

object Say {
    def say(msg: String): Unit = println(msg)
}
```

Hello.scala

```scala
package hello

object Hello {
  def hello(to: String): Unit = Say.say("Hello, " + to)
}
```

HelloWorld.scala

```scala
package hello

object HelloWorld {
  def main(args: Array[String]): Unit = Hello.hello("world")
}
```

BUILD

```
scala_library(
    name = 'Say',
    srcs = 'Say.scala',
)

scala_library(
    name = 'Hello',
    srcs = 'Hello.scala',
    deps = ':Say',
)

scala_fat_library(
    name = 'HelloWorldFat',
    srcs = 'HelloWorld.scala',
    deps = ':Hello',
)
```

目录结构：

```
blade-demo/
    BLADE_ROOT
    scala/
        hello/
            BUILD
            Say.scala
            Hello.scala
            HelloWorld.scala
```

在blade-demo/scala/hello目录下执行

```bash
blade build :HelloWorldFat
```

则在blade-demo/build64_release/scala/hello目录下会生成HelloWorldFat.fat.jar文件：

```bash
$ jar tf HelloWorldFat.fat.jar
hello/HelloWorld.class
hello/HelloWorld$.class
hello/Hello.class
hello/Hello$.class
hello/Say.class
hello/Say$.class
META-INF/blade/JAR.LIST
META-INF/blade/MERGE-INFO
META-INF/MANIFEST.MF
```

要运行程序，在blade-demo/build64_release/scala/hello目录下执行

```bash
$ scala -cp HelloWorldFat.fat.jar hello.HelloWorld
Hello, world
```

注：如果用java命令运行会报错 "Exception in thread "main" java.lang.NoClassDefFoundError: scala/collection/mutable/StringBuilder"，因为缺少Scala标准库。

#### 6.3.5 Python
Python是解释型语言，因此Python的构建规则不需要执行任何编译操作，只生成一个包含源代码位置的文件。

##### 6.3.5.1 py_library
构建Python库。语法：

```
py_library(
    name = 'foo',
    srcs = ['foo.py', 'bar.py', ...]
    deps = [':name', '//path/to/dir:name', ...],
    base = '//path/to/base',
)
```

* `base`：字符串，指定导入模块起始路径，默认为项目根目录

##### 6.3.5.2 py_binary
构建Python可执行程序。语法：

```
py_binary(
    name = 'foo',
    srcs = ['foo.py', 'bar.py', ...]
    deps = [':name', '//path/to/dir:name', ...],
    base = '//path/to/base',
    main = 'foo.py',
)
```

* `main`：字符串，当`srcs`包含多个文件时指定程序入口文件

##### 6.3.5.3 py_test
构建Python测试程序。语法：

```
py_test(
    name = 'foo_test',
    srcs = ['foo_test.py', ...],
    deps = [':name', '//path/to/dir:name', ...],
    base = '//path/to/base',
    main = 'foo_test.py',
    testdata = testdata = ['data1.txt', '//path/to/data2.txt'],
)
```

具体用法见8.4节。

##### 6.3.5.4 示例
下面用Python实现第4节中的Hello World示例。

say.py

```python
def say(msg):
    print(msg)
```

hello.py

```python
from python.hello import say

def hello(to):
    say.say('Hello, ' + to)
```

hello_world.py

```python
from python.hello import hello

def main():
    hello.hello('world')

if __name__ == '__main__':
    main()
```

BUILD

```
py_library(
    name = 'say',
    srcs = 'say.py',
)

py_library(
    name = 'hello',
    srcs = 'hello.py',
    deps = ':say',
)

py_binary(
    name = 'hello_world',
    srcs = 'hello_world.py',
    deps = ':hello',
)
```

目录结构：

```
blade-demo/
    BLADE_ROOT
    python/
        hello/
            BUILD
            say.py
            hello.py
            hello_world.py
```

其中`import`路径相对于根目录blade-demo，因此`python.hello`对应python/hello目录。

在blade-demo/python/hello目录下执行

```bash
blade build :hello_world
```

则在blade-demo/build64_release/python/hello目录下会生成三个.pylib文件和一个可执行文件hello_world：

```bash
$ ls
hello.pylib  hello_world  hello_world.pylib  say.pylib  ...

$ cat hello_world.pylib
{'srcs': [('python/hello/hello_world.py', 'df892f737d6f06060254cb903c597286')], 'base_dir': ''}

$ head -3 hello_world
#!/bin/sh

PYTHONPATH="$0:$PYTHONPATH" exec python -m "python.hello.hello_world" "$@"
```

可以看出，.pylib文件只是一个记录了源代码位置和文件md5的JSON文件，可执行文件hello_world就是一个Shell脚本。

要运行程序，可以在blade-demo/python/hello目录下执行

```bash
$ blade run :hello_world
...
Blade(info): Run '['.../blade-demo/build64_release/python/hello/hello_world']'
Hello, world
```

或者在blade-demo/build64_release/python/hello目录下执行

```bash
$ ./hello_world
Hello, world
```

或者直接在blade-demo目录下执行（不需要Blade）

```bash
$ python3 -m python.hello.hello_world
Hello, world
```

#### 6.3.6 文件打包
从源代码目录和构建目录打包文件。语法：

```
package(
    name = 'foo_package',
    type = 'tgz',
    shell = True,
    srcs = [
        ('$(location //path/to/src:name)', 'path/to/dst/name'),
        ('//path/to/src/file', 'path/to/dst/file'),
    ]
)
```

* `type`：字符串，指定压缩文件后缀名，可以是zip、tar、tar.gz、tgz、tar.bz2、tbz
* `shell`：布尔值，如果为`True`则使用Shell创建压缩文件
* `srcs`：二元组(src, dst)的列表，指定要打包的内容，其中src可以是源代码目录中的文件（例如配置文件）或构建目标的产物（例如可执行程序），dst是压缩文件中的相对路径，src支持的格式如下：
  * `$(location //path/to/src:name)`：目标`//path/to/src:name`的构建产物
  * `//path/to/src/file`：源代码目录中的文件，如果以`//`开头则表示相对于项目根目录，否则相对于当前目录；file可以是单个文件，也可以是整个目录

`package`规则在构建时默认不执行，除非`blade build`显式指定该目标，或指定了`--generate-package`选项。

例如：在第4节Hello World项目的基础上增加一个配置文件conf/hello_world.conf和一个数据文件data.txt：

```
blade-demo/
    BLADE_ROOT
    quick-start/
        BUILD
        say.h
        say.cc
        hello.h
        hello.cc
        hello_world.cc
        data.txt
        conf/
            hello_world.conf
```

在BUILD文件中增加一个`package`目标：

```
package(
    name = 'hello_world_package',
    type = 'tar',
    shell = True,
    srcs = [
        ('$(location //quick-start:hello_world)', 'bin/hello_world'),
        ('$(location //quick-start:hello)', 'lib/libhello.a'),
        ('//quick-start/conf', 'conf'),
        ('data.txt', 'data/foo.txt'),
    ]
)
```

在quick-start目录下执行

```bash
$ blade build :hello_world_package
```

将在blade-demo/build64_release/quick-start目录下生成hello_world_package.tar：

```bash
$ alt  # 等价于cd ../build64_release/quick-start
$ pwd
.../blade-demo/build64_release/quick-start
$ tar -tf hello_world_package.tar
conf/hello_world.conf
data/foo.txt
bin/hello_world
lib/libhello.a
```

#### 6.3.7 自定义构建规则
语法：

```
gen_rule(
    name = 'foo',
    srcs = ['bar', 'baz', ...],
    deps = ['//path/to/dir:name', ...],
    outs = ['foo'],
    cmd = 'shell command to generate foo',
    cmd_name = 'FOO',
)
```

* `srcs`：字符串列表（可选），指定构建目标的输入文件
* `outs`：字符串列表（必需），指定构建目标的输出文件
* `cmd`：字符串（必需），生成输出文件的Shell命令，可使用以下变量：
  * `$SRCS`：空格分隔的输入文件列表，相对于项目根目录
  * `$OUTS`	：空格分隔的输出文件列表，相对于项目根目录
  * `$FIRST_SRC`：第一个输入文件
  * `$FIRST_OUT`：第一个输出文件
  * `$SRC_DIR`：输入文件所在目录
  * `$OUT_DIR`：输出文件所在目录
  * `$BUILD_DIR`：根输出目录
* `cmd_name`：字符串（可选），命令名称的简写，默认为`COMMAND`

执行命令的工作目录是项目根目录。如果最后没有生成`outs`指定的文件则报错。

例如，在子目录foo中有a.txt和b.txt两个文件：

```
blade-demo/
    BLADE_ROOT
    foo/
        BUILD
        a.txt
        b.txt
```

a.txt和b.txt的内容分别为“123”和“abc”，BUILD文件包含一个自定义规则——通过拼接a.txt和b.txt生成c.txt：

```
gen_rule(
    name = 'c',
    srcs = ['a.txt', 'b.txt'],
    outs = ['c.txt'],
    cmd = 'cat $SRCS > $OUTS',
    cmd_name = 'CAT',
)
```

在foo目录下执行

```bash
$ blade build :c
Blade(info): Building...
[1/1] CAT //foo:c
Blade(info): Build success.
Blade(info): Cost time 0.199s

$ alt
$ pwd
.../blade-demo/build64_release/foo
$ cat c.txt
123
abc
```

在这个示例中，各变量的值如下：
```
$SRCS = "foo/a.txt foo/b.txt"
$OUTS = "build64_release/foo/c.txt"
$FIRST_SRC = "foo/a.txt"
$FIRST_OUT = "build64_release/foo/c.txt"
$SRC_DIR = "foo"
$OUT_DIR = "build64_release/foo"
$BUILD_DIR = "build64_release"
```

## 7.命令行参考
官方文档：[Command Line](https://github.com/chen3feng/blade-build/blob/master/doc/en/command_line.md)

Blade命令行语法：

```bash
blade <subcommand> [options...] [targets...]
```

使用`blade --help`可以查看所有的子命令。

### 7.1 子命令
* `build`：构建指定的目标
* `run`：构建并运行指定的目标
* `test`：构建指定的目标并运行测试
* `clean`：删除指定目标的构建产物
* `query`：分析指定的目标依赖或被依赖的目标
* `dump`：打印指定目标的内部信息

使用`blade <subcommand> --help`可以查看子命令的帮助信息。

### 7.2 构建目标模式
子命令需要指定一个或多个构建目标参数，称为目标模式(target pattern)，支持以下语法：
* `path:name`：path目录下名为name的目标
* `:name`：当前目录下名为name的目标
* `path:*`或`path`：path目录下的所有目标，不包括子目录
* `path/...`：path目录及其子目录下的所有目标

如果path以`//`开头则表示从项目根目录开始的路径，否则表示基于当前目录的相对路径。

如果没有指定目标则表示当前目录下的所有目标，不包括子目录。

### 7.3 示例

```bash
## 构建当前目录下的所有目标，不包括子目录
blade build

## 构建当前目录及其子目录下的所有目标
blade build ...

## 构建当前目录下名为hello的目标
blade build :hello

## 构建项目根目录/common/string目录下名为string的目标
blade build //common/string:string

## 构建当前目录/string目录下名为string的目标
blade build string:string

## 构建项目根目录/common及其子目录下的所有目标
blade build //common/...
```

## 8.测试
官方文档：[Testing Support](https://github.com/chen3feng/blade-build/blob/master/doc/en/test.md)

Blade为每种语言都提供了测试支持，分别使用不同的测试框架（但官方文档缺少详细的说明）。

### 8.1 C++ - GoogleTest
C++测试使用`cc_test`目标来定义，底层使用[GoogleTest](https://google.github.io/googletest/)框架。

#### 8.1.1 安装和配置GoogleTest
在Blade中使用`cc_test`之前，首先要安装GoogleTest，并在BLADE_ROOT中指定`gtest`和`gtest_main`库的位置，有两种方式。

方法一：参考[googletest README - Standalone CMake Project](https://github.com/google/googletest/tree/main/googletest#standalone-cmake-project)单独安装GoogleTest，安装完成后头文件将被拷贝到/usr/local/include目录，库文件将被拷贝到/usr/local/lib目录。

之后在BLADE_ROOT中添加：

```
cc_test_config(
    gtest_libs = ['#gtest', '#pthread'],
    gtest_main_libs = '#gtest_main',
)
```

其中，`#gtest`相当于链接选项`-lgtest`，`#gtest_main`相当于链接选项`-lgtest_main`。由于`gtest`库依赖pthread库，因此还需要添加`#pthread`。

方法二：将GoogleTest源代码包含在项目源代码中，并为其编写BUILD文件（需要了解GoogleTest库的构建细节和专业知识）。例如，将源代码放在thirdparty/gtest目录下，BUILD文件包含`gtest`和`gtest_main`两个构建目标。之后在BLADE_ROOT中添加：

```
cc_test_config(
    gtest_libs = 'thirdparty/gtest:gtest',
    gtest_main_libs = 'thirdparty/gtest:gtest_main',
)
```

#### 8.1.2 示例
在项目根目录下创建一个test_demo目录，并实现一个计算阶乘的函数作为被测函数。

factorial.h

```cpp
#pragma once

// Returns the factorial of n
long long factorial(int n);
```

factorial.cc

```cpp
#include "factorial.h"

long long factorial(int n) {
    return n == 0 ? 1 : n * factorial(n - 1);
}
```

下面为`factorial()`函数编写测试代码：

factorial_test.cc

```cpp
#include <gtest/gtest.h>

#include "factorial.h"

// Tests factorial of 0.
TEST(FactorialTest, HandlesZeroInput) {
    EXPECT_EQ(factorial(0), 1);
}

// Tests factorial of positive numbers.
TEST(FactorialTest, HandlesPositiveInput) {
    EXPECT_EQ(factorial(1), 1);
    EXPECT_EQ(factorial(2), 2);
    EXPECT_EQ(factorial(3), 6);
    EXPECT_EQ(factorial(8), 40320);
}
```

BUILD

```
cc_library(
    name = 'factorial',
    srcs = 'factorial.cc',
    hdrs = 'factorial.h',
)

cc_test(
    name = 'factorial_test',
    srcs = 'factorial_test.cc',
    deps = ':factorial',
)
```

目录结构：

```
blade-demo/
    BLADE_ROOT
    test_demo/
        BUILD
        factorial.h
        factorial.cc
        factorial_test.cc
```

要运行测试，在test_demo目录下执行

```bash
$ blade test :factorial_test

[==========] Running 2 tests from 1 test suite.
[----------] Global test environment set-up.
[----------] 2 tests from FactorialTest
[ RUN      ] FactorialTest.HandlesZeroInput
[       OK ] FactorialTest.HandlesZeroInput (0 ms)
[ RUN      ] FactorialTest.HandlesPositiveInput
[       OK ] FactorialTest.HandlesPositiveInput (0 ms)
[----------] 2 tests from FactorialTest (0 ms total)

[----------] Global test environment tear-down
[==========] 2 tests from 1 test suite ran. (1 ms total)
[  PASSED  ] 2 tests.
Blade(info): [1/0/1] Test //test_demo:factorial_test finished : SUCCESS
```

GoogleTest的用法参考：[GoogleTest使用教程]({% post_url 2022-10-06-googletest-tutorial %})

### 8.2 Java - JUnit
Java测试使用`java_test`目标来定义，底层使用[JUnit](https://junit.org/)框架。

#### 8.2.1 配置JUnit
虽然BLADE_ROOT支持`java_test_config.junit_libs`配置，但经过测试，该配置并不起作用，因此每个`java_test`必须显式地依赖JUnit库。可以在thirdparty目录下使用`maven_jar`声明JUnit库：

thirdparty/junit/BUILD

```
maven_jar(
  name = 'junit4',
  id = 'junit:junit:4.13.2',
)
```

#### 8.2.2 示例
下面使用Java实现阶乘的示例。

Factorial.java

```java
package com.example.java;

public class Factorial {
    public static long factorial(int n) {
        return n == 0 ? 1 : n * factorial(n - 1);
    }
}
```

FactorialTest.java

```java
package com.example.java;

import org.junit.Test;

import static org.junit.Assert.*;

public class FactorialTest {

    @Test
    public void handlesZeroInput() {
        assertEquals(1, Factorial.factorial(0));
    }

    @Test
    public void handlesPositiveInput() {
        assertEquals(1, Factorial.factorial(1));
        assertEquals(2, Factorial.factorial(2));
        assertEquals(6, Factorial.factorial(3));
        assertEquals(40320, Factorial.factorial(8));
    }

}
```

BUILD

```
java_library(
    name = 'FactorialJava',
    srcs = 'Factorial.java',
)

java_test(
    name = 'FactorialJavaTest',
    srcs = 'FactorialTest.java',
    deps = [
        ':FactorialJava',
        '//thirdparty/junit:junit4',
    ],
)
```

要运行测试，在test_demo目录下执行

```bash
$ blade test :FactorialJavaTest

JUnit version 4.13.2
..
Time: 0.018

OK (2 tests)

Blade(info): [1/0/1] Test //test_demo:FactorialJavaTest finished : SUCCESS
```

### 8.3 Scala - ScalaTest
Scala测试使用`scala_test`目标来定义，底层使用[ScalaTest](https://www.scalatest.org/)框架。

#### 8.3.1 配置ScalaTest
手动下载ScalaTest jar文件：

```bash
mvn dependency:get -Dartifact=org.scalatest:scalatest-app_2.13:3.2.15
```

其中 "2.13" 是Scala版本，"3.2.15" 是ScalaTest版本。

在thirdparty目录下声明ScalaTest及其依赖的scala-xml库：

thirdparty/scalatest/BUILD

```
java_library(
    name = 'scalatest_2.13',
    prebuilt = True,
    binary_jar = '/home/zzy/.m2/repository/org/scalatest/scalatest-app_2.13/3.2.15/scalatest-app_2.13-3.2.15.jar',
)
```

thirdparty/scala/BUILD

```
maven_jar(
    name = 'scala-xml_2.13',
    id = 'org.scala-lang.modules:scala-xml_2.13:2.1.0',
)
```

之后在BLADE_ROOT中添加：

```
scala_test_config(
    scalatest_libs = [
        '//thirdparty/scala:scala-xml_2.13',
        '//thirdparty/scalatest:scalatest_2.13',
    ],
)
```

#### 8.3.2 示例
下面使用Scala实现阶乘的示例。

Factorial.scala

```scala
package com.example.scala

object Factorial {
  def factorial(n: Int): Long = if (n == 0) 1 else n * factorial(n - 1)
}
```

FactorialTest.scala

```scala
package com.example.scala

import org.scalatest.funsuite.AnyFunSuite

class FactorialTest extends AnyFunSuite {
  test("handlesZeroInput") {
    assertResult(1)(Factorial.factorial(0))
  }

  test("handlesPositiveInput") {
    assertResult(1)(Factorial.factorial(1))
    assertResult(2)(Factorial.factorial(2))
    assertResult(6)(Factorial.factorial(3))
    assertResult(40320)(Factorial.factorial(8))
  }
}
```

BUILD

```
scala_library(
    name = 'FactorialScala',
    srcs = 'Factorial.scala',
)

scala_test(
    name = 'FactorialScalaTest',
    srcs = 'FactorialTest.scala',
    deps = ':FactorialScala',
)
```

要运行测试，在test_demo目录下执行

```bash
$ blade test :FactorialScalaTest

Run starting. Expected test count is: 2
FactorialTest:
- handlesZeroInput
- handlesPositiveInput
Run completed in 384 milliseconds.
Total number of tests run: 2
Suites: completed 1, aborted 0
Tests: succeeded 2, failed 0, canceled 0, ignored 0, pending 0
All tests passed.
Blade(info): [1/0/1] Test //test_demo:FactorialScalaTest finished : SUCCESS
```

注意：
* 必须配置`java_config.java_home`和`scala_config.scala_home`，否则运行Scala测试会报错（Blade自动生成的测试命令中包含 "java" 和 "scala"，但如果没有配置`java_home`和`scala_home`，Blade会将项目根目录拼接到这两个命令前面，见[builtin_tools.py](https://github.com/chen3feng/blade-build/blob/master/src/blade/builtin_tools.py) `generate_scala_test()`）。
* ScalaTest库不能通过`maven_jar`声明，因为其打包方式是bundle，Blade解析依赖时会报错 "Unknown packaging: bundle"（普通的Maven项目可以通过在pom.xml文件中添加maven-bundle-plugin插件来解决，但Blade不支持添加Maven插件）。

### 8.4 Python - unittest
Python测试使用`py_test`目标来定义，底层使用标准库自带的[unittest](https://docs.python.org/3/library/unittest.html)模块。

#### 8.4.1 示例
下面使用Python实现阶乘的示例。

factorial.py

```python
def factorial(n):
    return 1 if n == 0 else n * factorial(n - 1)
```

factorial_test.py

```python
import unittest

from test_demo.factorial import factorial


class FactorialTest(unittest.TestCase):

    def test_handles_zero_input(self):
        self.assertEqual(factorial(0), 1)
    
    def test_handles_positive_input(self):
        self.assertEqual(factorial(1), 1)
        self.assertEqual(factorial(2), 2)
        self.assertEqual(factorial(3), 6)
        self.assertEqual(factorial(8), 40320)


if __name__ == '__main__':
    unittest.main()
```

BUILD

```
py_library(
    name = 'factorial_py',
    srcs = 'factorial.py',
)

py_test(
    name = 'factorial_py_test',
    srcs = 'factorial_test.py',
    deps = ':factorial_py',
)
```

要运行测试，在test_demo目录下执行

```bash
$ blade test :factorial_py_test

..
----------------------------------------------------------------------
Ran 2 tests in 0.004s

OK
Blade(info): [1/0/1] Test //test_demo:factorial_py_test finished : SUCCESS
```

也可以直接在项目根目录下运行

```bash
$ python3 -m unittest test_demo.factorial_test
..
----------------------------------------------------------------------
Ran 2 tests in 0.002s

OK
```

## 9.配置文件
官方文档：[Configuration file](https://github.com/chen3feng/blade-build/blob/master/doc/en/config.md)

## 10.IDE支持
详见：[Blade项目的IDE支持]({% post_url 2023-04-25-blade-project-ide-support %})
