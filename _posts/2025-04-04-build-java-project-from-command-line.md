---
title: 从命令行构建Java项目
date: 2025-04-04 20:48:31 +0800
categories: [Java]
tags: [java, class path, jar file, module]
---
在实际中，通常会使用IDE（如IntelliJ IDEA或Eclipse）和构建工具（如Maven或Gradle）来构建Java项目。本文将介绍如何使用命令行构建，以便了解IDE和构建工具的底层实现原理。

## 1.单个源文件
首先考虑最简单的单个源文件场景。假设有一个源文件HelloWorld.java：

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, world!");
    }
}
```

从命令行进入该文件所在目录，并执行以下命令：

```shell
# 编译
javac HelloWorld.java
# 运行
java HelloWorld
```

`javac`命令是Java编译器，将源文件编译成类文件(.class)。`java`命令启动Java虚拟机，并执行指定的类的`main()`方法。

注：从Java 11起，对于单个源文件的程序，可以直接执行`java HelloWorld.java`（见[JEP 330](https://openjdk.org/jeps/330)）。

### 使用包
上面的`HelloWorld`类在无名包中。假设将其放到`com.example.hello`包中：

```java
package com.example.hello;

public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, world!");
    }
}
```

源代码应该放到与包名匹配的子目录中(com/example/hello)，编译器会将类文件也放到相同的目录结构中，如下所示。

```
basedir/
  com/
    example/
      hello/
        HelloWorld.java
        HelloWorld.class
```

从基目录编译并运行这个程序：

```shell
javac com/example/hello/HelloWorld.java
java com.example.hello.HelloWorld
```

## 2.多个源文件
实际的项目通常由多个包和源文件组成。假设`com.example.greet`包中的`Greeter`类提供了获取问候语的方法：

```java
package com.example.greet;

public class Greeter {
    public String greet(String name) {
        return "Hello, " + name + "!";
    }
}
```

`com.example.hello`包中的`HelloWorld`类使用`Greeter`类打印问候语：

```java
package com.example.hello;

import com.example.greet.Greeter;

public class HelloWorld {
    public static void main(String[] args) {
        Greeter greeter = new Greeter();
        System.out.println(greeter.greet("world"));
    }
}
```

目录结构如下：

```
basedir/
  com/
    example/
      greet/
        Greeter.java
        Greeter.class
      hello/
        HelloWorld.java
        HelloWorld.class
```

编译和运行方式与上一节相同：

```shell
javac com/example/hello/HelloWorld.java
java com.example.hello.HelloWorld
```

编译器会自动地查找文件com/example/greet/Greeter.java并进行编译（当然，如果愿意，也可以在编译命令中把所有源文件都列出）。

### 指定输出目录
上面的构建方式直接将类文件生成到源代码目录下。为了便于打包和部署，最好将类文件放到单独的目录中。为此，将源文件移至src目录，使用build目录存放类文件，如下所示。

```
basedir/
  src/
    com/
      example/
        greet/
          Greeter.java
        hello/
          HelloWorld.java
  build/
    com/
      example/
        greet/
          Greeter.class
        hello/
          HelloWorld.class
```

为了编译并运行这个程序，在基目录中执行

```shell
javac -d build src/com/example/hello/HelloWorld.java src/com/example/greet/Greeter.java
java -cp build com.example.hello.HelloWorld
```

其中，`java`命令的`-cp`选项指定**类路径**(class path)，即Java虚拟机搜索类文件的路径集合。类路径可以包含类文件输出目录、当前目录(.)或JAR文件。如果有多项，在Linux中用`:`分隔，在Windows中用`;`分隔。如果未指定，则默认为当前目录。

注意，在这种情况下，必须在编译命令中列出所有源文件，否则编译器会报错“找不到符号Greeter”。对于大型项目，列出所有源文件会很繁琐。此时可以使用`find`命令和通配符：

```shell
javac -d build $(find src -name *.java)
```

另一种方式是像之前一样让编译器自动查找Greeter.java，为此需要将源代码目录添加到编译器的类路径：

```shell
javac -d build -cp src src/com/example/hello/HelloWorld.java
```

注：当编译器遇到未知的类时（例如在编译HelloWorld.java时遇到`Greeter`类），会在`-cp`选项指定的类路径中搜索类文件（如Greeter.class）。如果未找到，则在`-sourcepath`选项指定的路径（如果未指定则使用类路径）中搜索源文件（如Greeter.java），并与当前源文件一起编译（除非在编译模块，见第5节）。如果二者都未找到则报错。

## 3.多个项目
为了便于管理，有时会将大型项目拆分成多个子项目，子项目之间存在依赖关系。

继续考虑上一节中的例子。假设`Greeter`和`HelloWorld`类分别在两个单独的项目greeter-proj和hello-proj中，后者依赖前者，源代码目录结构如下：

```
basedir/
  greeter-proj/
    src/
      com/example/greet/
        Greeter.java
  hello-proj/
    src/
      com/example/hello/
        HelloWorld.java
```

在这个例子中，有两种构建方式。

（1）两个项目共用输出目录。在编译时，通过`javac`命令的`-cp`选项将两个项目的源代码目录添加到类路径。这样两个项目的类文件将输出到同一个目录。

在基目录中执行

```shell
javac -d build -cp greeter-proj/src:hello-proj/src hello-proj/src/com/example/hello/HelloWorld.java
java -cp build com.example.hello.HelloWorld
```

目录结构如下：

```
basedir/
  greeter-proj/
    src/
      com/example/greet/
        Greeter.java
  hello-proj/
    src/
      com/example/hello/
        HelloWorld.java
  build/
    com/example/greet/
      Greeter.class
    com/example/hello/
      HelloWorld.class
```

这种方式的类路径设置比较简单，适用于小型项目。但是，每次都会执行全量构建，即构建项目的所有依赖和传递依赖。这在大型项目中将非常低效。

（2）每个项目有各自的输出目录。在这种情况下，必须先编译greeter-proj：

```shell
javac -d greeter-proj/build greeter-proj/src/com/example/greet/Greeter.java
```

然后编译hello-proj，将greeter-proj的输出目录添加到类路径：

```shell
javac -d hello-proj/build -cp greeter-proj/build hello-proj/src/com/example/hello/HelloWorld.java
```

最后运行程序，将两个项目的输出目录都添加到类路径：

```shell
java -cp greeter-proj/build:hello-proj/build com.example.hello.HelloWorld
```

目录结构如下：

```
basedir/
  greeter-proj/
    src/
      com/example/greet/
        Greeter.java
    build/
      com/example/greet/
        Greeter.class
  hello-proj/
    src/
      com/example/hello/
        HelloWorld.java
    build/
      com/example/hello/
        HelloWorld.class
```

这种方式的优点是可以执行增量构建：如果某个依赖的源代码没有发生变化，则无需重新编译。如果多个项目依赖同一个项目，被依赖的项目只需要编译一次。但是，当依赖关系非常复杂时，手动编译将非常困难，因为需要添加大量的类路径（包括所有依赖和传递依赖）。另外，逐个比较源文件和类文件的更新时间戳来判断是否更新也非常耗时。因此，大型项目通常使用Maven、Gradle等构建工具来自动化构建过程。

## 4.JAR文件
JAR文件用于将类文件打包成单一的文件，以便于发布和部署。

使用`jar`命令创建JAR文件（语法类似于UNIX的`tar`命令）：

```shell
jar cf greeter-proj.jar -C greeter-proj/build .
```

该命令将greeter-proj输出目录中的类文件打包成greeter-proj.jar。可以用如下命令查看JAR文件的内容：

```shell
$ jar tf greeter-proj.jar 
META-INF/
META-INF/MANIFEST.MF
com/
com/example/
com/example/greet/
com/example/greet/Greeter.class
```

可以看到，其中的目录结构与输出目录完全相同，另外添加了清单文件META-INF/MANIFEST.MF。

用同样的方式创建hello-proj.jar。之后将两个JAR文件添加到类路径并运行程序：

```shell
java -cp greeter-proj.jar:hello-proj.jar com.example.hello.HelloWorld
```

依赖第三方库时，也可以使用同样的方法。

另外，也可以通过`jar`命令的`e`选项指定主类，从而创建可执行JAR：

```shell
jar cfe hello.jar com.example.hello.HelloWorld -C greeter-proj/build . -C hello-proj/build .
```

之后使用`java -jar`命令执行JAR文件：

```shell
java -jar hello.jar
```

注意：使用`-jar`时，不能指定其他的类路径，因此必须将所有依赖的类文件放在同一个JAR文件中。

## 5.模块
为了更好地维护大型项目，Java 9引入了Java平台模块系统。与传统的类路径方式相比，它具有强封装、可靠的依赖管理等特性。详见[《Java核心技术》笔记 卷II 第9章]({% post_url 2025-03-24-java-note-v2ch09-the-java-platform-module-system %})。

下面在第3节示例的基础上，将两个项目改造为模块。首先，用`greeter.mod`模块代替greeter-proj。`Greeter`类改为接口，属于该模块的公共API：

```java
package com.example.greet;

public interface Greeter {
    static Greeter newInstance() {
        return new com.example.greet.internal.GreeterImpl();
    }

    String greet(String name);
}
```

实现类`GreeterImpl`在`com.example.greet.internal`包中，属于该模块的内部实现：

```java
package com.example.greet.internal;

import com.example.greet.Greeter;

public class GreeterImpl implements Greeter {
    @Override
    public String greet(String name) {
        return "Hello, " + name + "!";
    }
}
```

模块声明放在模块根目录下名为module-info.java的文件中：

```java
module greeter.mod {
    exports com.example.greet;
}
```

其中，`exports`关键字用于声明导出的包。其他模块只能访问该模块导出的包，而不能访问未导出的包（如`com.example.greet.internal`）。

`hello.mod`模块代替hello-proj。`HelloWorld`类改为使用`Greeter.newInstance()`方法，而不是直接创建`Greeter`实例：

```java
package com.example.hello;

import com.example.greet.Greeter;

public class HelloWorld {
    public static void main(String[] args) {
        Greeter greeter = Greeter.newInstance();
        System.out.println(greeter.greet("modular world"));
    }
}
```

`hello.mod`模块需要使用`requires`关键字声明依赖`greeter.mod`模块：

```java
module hello.mod {
    requires greeter.mod;
}
```

这两个模块的目录结构如下：

```
basedir/
  greeter.mod/
    src/
      module-info.java
      com/example/greet/
        Greeter.java
        internal/
          GreeterImpl.java
    build/
  hello.mod/
    src/
      module-info.java
      com/example/hello/
        HelloWorld.java
    build/
```

这两个模块的构建方式与第3节类似。首先编译`greeter.mod`模块：

```shell
javac -d greeter.mod/build greeter.mod/src/module-info.java \
  greeter.mod/src/com/example/greet/Greeter.java \
  greeter.mod/src/com/example/greet/internal/GreeterImpl.java
```

编译模块时，需要将module-info.java添加到源文件列表。注意，由于`GreeterImpl`不属于该模块导出的包，因此必须显式列出对应源文件，而不能用第2节结尾的方式让编译器自动查找。或者，使用`find`命令：

```shell
javac -d greeter.mod/build $(find greeter.mod/src -name *.java)
```

然后编译`hello.mod`模块，使用`-p`选项将`greeter.mod`的输出目录添加到**模块路径**(module path)：

```shell
javac -d hello.mod/build -p greeter.mod/build $(find hello.mod/src -name *.java)
```

最后，为了运行这个程序，需要将两个模块的输出目录都添加到模块路径，并使用`-m`选项以`modulename/classname`的形式指定主类：

```shell
java -p greeter.mod/build:hello.mod/build -m hello.mod/com.example.hello.HelloWorld
```

### 模块化的JAR
模块也可以打包成JAR文件，其中module-info.class在根目录。这样的JAR文件称为模块化的JAR。

创建模块化的JAR文件的方式与常规JAR文件完全相同：

```shell
jar cf greeter.mod.jar -C greeter.mod/build .
jar cf hello.mod.jar -C hello.mod/build .
```

运行程序时，将JAR文件添加到模块路径，并使用`-m`选项指定模块和主类：

```shell
java -p greeter.mod.jar:hello.mod.jar -m hello.mod/com.example.hello.HelloWorld
```
