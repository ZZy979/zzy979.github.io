---
title: 《Java核心技术》笔记 卷II 第9章 Java平台模块系统
date: 2025-03-24 21:28:48 +0800
categories: [Java, Core Java]
tags: [java, module, jar file]
---
Java 9引入了[Java平台模块系统](https://openjdk.org/projects/jigsaw/spec/)。它是为了模块化大型Java代码库而设计的。如果愿意，也可以使用这个系统来模块化自己的应用。

本章将展示如何声明和使用Java平台模块。还将介绍如何迁移你的应用程序，使其与Java平台和第三方模块一起工作。

## 9.1 模块概念
面对规模巨大、盘根错节的代码，Java平台设计者认为他们需要一种能够提供更多控制的结构化机制。他们发现现有的模块系统（例如OSGi）都不适用于他们的问题。因此，他们设计了一个新的系统，称为**Java平台模块系统**(Java Platform Module System)，现在成了Java语言和虚拟机的一部分。这个系统已经成功地用于将Java API模块化。

一个Java平台**模块**(module)包括：
* 包的集合
* （可选）资源文件和其他文件（例如本地库）
* 模块中可访问的包的列表
* 该模块依赖的模块列表

Java平台在编译时和虚拟机中强制执行封装(encapsulation)和依赖(dependencies)。

与传统的通过类路径使用JAR文件的方式，Java平台模块系统有两个优点：
1. 强封装：可以控制哪些包是可访问的，无需操心维护那些不打算公共使用的代码。
2. 可靠配置：可以避免常见的类路径问题，例如重复或缺少类。

有一些Java平台模块系统无法解决的问题，例如模块的版本管理。不支持指定依赖模块的版本，或者在同一个程序中使用一个模块的多个版本。

## 9.2 命名模块
**模块是包的集合。** 模块中的包名无须彼此相关。例如，[`java.sql`模块](https://docs.oracle.com/en/java/javase/17/docs/api/java.sql/module-summary.html)包含`java.sql`和`javax.sql`这两个包。模块名和包名相同是完全可行的。

与包名一样，模块名由字母、数字、下划线和句点组成。而且，模块之间也没有层次关系。例如，模块`java.sql`和`java.sql.rowset`是无关的。

创建供他人使用的模块时，重要的是确保其名字是全局唯一的。大多数模块名都遵循“反向域名”惯例，就像包名一样（见卷I第4章 4.8.1节）。最简单的方法是以模块提供的顶级包来命名模块。例如，SLF4J日志库有一个[`org.slf4j`模块](https://github.com/qos-ch/slf4j/blob/master/slf4j-api/src/main/java9/module-info.java)，其中包含`org.slf4j`、`org.slf4j.spi`、`org.slf4j.event`和`org.slf4j.helpers`包。这个惯例可以防止模块中的包名产生冲突。因为一个包只能放到一个模块中，如果模块名是唯一的，并且包名以模块名开头，那么包名也就是唯一的。

注释：模块名只用于模块声明中。在Java源文件中永远不会使用模块名，而应该按照一贯的方式使用包名。

## 9.3 模块化的 "Hello, World!" 程序
下面将传统的 "Hello, World!" 程序放到一个模块中。首先，需要将这个类放到一个包中——**无名包不能包含在模块中**。

```java
package com.horstmann.hello;

public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, Modular World!");
    }
}
```

为了创建包含这个包的模块`v2ch09.hellomod`，需要添加一个模块声明。模块声明放在名为**module-info.java**的文件中，位于基目录中（即com目录所在目录）。按照惯例，基目录的名字与模块名相同。目录结构如下所示：

```
v2ch09.hellomod/
  module-info.java
  com/
    horstmann/
      hello/
        HelloWorld.java
```

module-info.java文件的内容如下：

```java
module v2ch09.hellomod {
}
```

这个模块声明是空的，因为该模块没有向其他人提供任何东西，也不依赖任何东西。

[v2ch09.hellomod](https://github.com/ZZy979/Core-Java-code/tree/main/v2ch09/v2ch09.hellomod)

（在v2ch09.hellomod目录的上层目录中）使用以下命令编译：

```shell
javac v2ch09.hellomod/module-info.java v2ch09.hellomod/com/horstmann/hello/HelloWorld.java
```

module-info.java文件被编译成类文件module-info.class，其中包含二进制形式的模块定义。

为了运行这个程序，需要用`--module-path`或`-p`选项指定**模块路径**(module path)（类似于类路径，但包含的是模块），还需要用`--module`或`-m`选项以`modulename/classname`的形式指定主类。

```shell
$ java -p v2ch09.hellomod -m v2ch09.hellomod/com.horstmann.hello.HelloWorld
Hello, Modular World!
```

注释：在编译这个模块时，会得到一条警告： "warning: [module] module name component v2ch09 should avoid terminal digits" 。这条警告意在建议程序员不要给模块名添加版本号。可以忽略这个警告，或者用注解来抑制它：

```java
@SuppressWarnings("module")
module v2ch09.hellomod {
}
```

注：
* 在Java 9引入模块系统之前，IntelliJ IDEA已经有模块的概念，见文档[Modules](https://www.jetbrains.com/help/idea/creating-and-managing-modules.html)。对Java模块的支持参见[Support for Java 9 Modules in IntelliJ IDEA 2017.1](https://blog.jetbrains.com/idea/2017/03/support-for-java-9-modules-in-intellij-idea-2017-1/)和[Java 9 and IntelliJ IDEA](https://blog.jetbrains.com/idea/2017/09/java-9-and-intellij-idea/)。
* 在IntelliJ IDEA中创建Java模块时，应该将模块基目录（即module-info.java文件所在目录）设置为源代码根目录(Sources Root)。

## 9.4 依赖模块
下面创建一个新的模块`v2ch09.requiremod`，其中使用`JOptionPane`显示消息 "Hello, Modular World!" ：

```java
package com.horstmann.hello;

import javax.swing.JOptionPane;

public class HelloWorld {
    public static void main(String[] args) {
        JOptionPane.showMessageDialog(null, "Hello, Modular World!");
    }
}
```

现在，编译会报错：

```
error: package javax.swing is not visible
  (package javax.swing is declared in module java.desktop, but module v2ch09.requiremod does not read it)
```

这是因为JDK已经被模块化，`javax.swing`包现在包含在`java.desktop`模块中。我们的模块需要用`requires`声明它依赖这个模块：

```java
module v2ch09.requiremod {
    requires java.desktop;
}
```

[v2ch09.requiremod](https://github.com/ZZy979/Core-Java-code/tree/main/v2ch09/v2ch09.requiremod)

模块系统的设计目标之一就是模块需要明确其依赖，因此虚拟机可以确保在启动程序之前所有依赖都被满足。

在上一节中，不需要显式依赖，因为只用到了`java.lang`包（`String`和`System`类），它包含在**默认依赖的`java.base`模块**中。

注意，`v2ch09.requiremod`模块只列出了它自己依赖的模块`java.desktop`。`java.desktop`模块本身又依赖另外三个模块：`java.datatransfer`、`java.prefs`和`java.xml`。下图展示了**模块图**(module graph)，其中节点是模块，边是依赖关系。

![v2ch09.requiremod的模块图](/assets/images/java-note-v2ch09-the-java-platform-module-system/v2ch09.requiremod的模块图.png)

在模块图中不能有环，即一个模块不能直接或间接依赖自身。

**模块的依赖关系不具有传递性(transitive)。** 例如，`java.desktop`依赖`java.prefs`，而`java.prefs`依赖`java.xml`，但是这并不会赋予`java.desktop`使用`java.xml`模块中的包的权力。一般来说，这种行为是我们想要的，因为这使得依赖明确。但是在9.11节中将看到，在某些情况下可以放松这条限制。

注释：按照Java平台模块系统的用语，模块M会在下列情况下**读入**(read)模块N：
* M依赖N
* M传递依赖N
* N是M或`java.base`

## 9.5 导出包
依赖一个模块并不意味着可以使用这个模块中的所有包。模块可以用`exports`关键字来声明哪些包是可访问的。例如，下面是`java.xml`模块声明的一部分：

```java
module java.xml {
    exports javax.xml;
    exports javax.xml.catalog;
    exports javax.xml.datatype;
    exports javax.xml.namespace;
    exports javax.xml.parsers;
    ...
}
```

这个模块导出了很多包，其他没有导出的包是隐藏的（例如`jdk.xml.internal`）。

当包被导出时，其中的`public`和`protected`类和接口，及其`public`和`protected`成员可以在模块外部访问（`protected`类型和成员仍然只能在子类中访问）。

**没有导出的包在其模块外部是不可访问的。** 这与Java 9引入模块之前有很大不同。在过去，可以使用任何包中的公有类。如今，不能再访问Java API中未导出的包。

下面在一个简单场景中使用导出机制。我们将编写一个`com.horstmann.greet`模块，它导出了一个包`com.horstmann.greet`，还有一个未导出的包`com.horstmann.greet.internal`。

公有的`Greeter`接口在第一个包中：

```java
package com.horstmann.greet;

public interface Greeter {
    static Greeter newInstance() {
        return new com.horstmann.greet.internal.GreeterImpl();
    }

    String greet(String subject);
}
```

第二个包有一个实现了该接口的类`GreeterImpl`（这个类是公有的，因为需要在第一个包中访问）：

```java
package com.horstmann.greet.internal;

import com.horstmann.greet.Greeter;

public class GreeterImpl implements Greeter {
    @Override
    public String greet(String subject) {
        return "Hello, " + subject + "!";
    }
}
```

该模块包含两个包，但是只导出第一个包：

```java
module com.horstmann.greet {
    exports com.horstmann.greet;
}
```

[com.horstmann.greet](https://github.com/ZZy979/Core-Java-code/tree/main/v2ch09/com.horstmann.greet)

应用程序放在另一个模块`v2ch09.exportedpkg`中，它依赖`com.horstmann.greet`模块：

```java
module v2ch09.exportedpkg {
    requires com.horstmann.greet;
}
```

程序使用`Greeter`来获取问候语：

```java
package com.horstmann.hello;

import com.horstmann.greet.Greeter;

public class HelloWorld {
    public static void main(String[] args) {
        Greeter greeter = Greeter.newInstance();
        System.out.println(greeter.greet("Modular World"));
    }
}
```

[v2ch09.exportedpkg](https://github.com/ZZy979/Core-Java-code/tree/main/v2ch09/v2ch09.exportedpkg)

下面是这两个模块的源文件目录结构：

```
com.horstmann.greet/
  module-info.java
  com/
    horstmann/
      greet/
        Greeter.java
        internal/
          GreeterImpl.java

v2ch09.exportedpkg/
  module-info.java
  com/
    horstmann/
      hello/
        HelloWorld.java
```

为了构建这个应用程序，首先要编译`com.horstmann.greet`模块：

```shell
javac com.horstmann.greet/module-info.java \
  com.horstmann.greet/com/horstmann/greet/Greeter.java \
  com.horstmann.greet/com/horstmann/greet/internal/GreeterImpl.java
```

然后编译应用程序模块，使用`-p`选项将第一个模块添加到模块路径：

```shell
javac -p com.horstmann.greet v2ch09.exportedpkg/module-info.java \
  v2ch09.exportedpkg/com/horstmann/hello/HelloWorld.java
```

最后，将两个模块都添加到模块路径来运行这个程序：

```shell
$ java -p v2ch09.exportedpkg:com.horstmann.greet -m v2ch09.exportedpkg/com.horstmann.hello.HelloWorld 
Hello, Modular World!
```

警告：模块不提供作用域。不同模块中不能有两个同名的包，即使是未导出的包也是如此。

注：如果要用IntelliJ IDEA来构建这个应用程序，首先通过菜单File → New → Module创建这两个模块（如果已创建模块目录则选择Module from Existing Sources）。然后，在模块目录上点击右键，选择Mark Directory as → Sources Root。最后，打开菜单 File → Project Structure，在`v2ch09.exportedpkg`模块的依赖中添加`com.horstmann.greet`模块（如下图所示）。

![添加IntelliJ IDEA模块依赖](/assets/images/java-note-v2ch09-the-java-platform-module-system/添加IntelliJ IDEA模块依赖.png)

## 9.6 模块化的JAR
到目前为止，我们直接将模块编译到了源代码目录中。显然，这不能用于部署。可以通过将所有类文件放置在一个JAR文件中来部署模块，其中module-info.class在根目录。这样的JAR文件称为**模块化的**(modular) JAR。

要创建模块化的JAR文件，只需以通常的方式使用`jar`工具。最好用`-d`选项来编译，从而将类文件放在单独的目录中。然后使用`jar`命令的`-C`选项指定该目录。

```shell
javac -d modules/com.horstmann.greet $(find com.horstmann.greet -name *.java)
jar -cvf com.horstmann.greet.jar -C modules/com.horstmann.greet .
```

如果使用像Maven、Ant或Gradle这样的构建工具，只需按照往常的方式来构建JAR文件。只要包含了module-info.class，就可以得到模块化的JAR。

然后，在模块路径中包含模块化的JAR，该模块就会被加载。

与常规JAR文件一样，可以在模块化的JAR中指定主类：

```shell
javac -p com.horstmann.greet.jar -d modules/v2ch09.exportedpkg $(find v2ch09.exportedpkg -name *.java)
jar -cvf v2ch09.exportedpkg.jar -e com.horstmann.hello.HelloWorld -C modules/v2ch09.exportedpkg .
```

启动程序时，使用`-m`选项指定包含主类的模块：

```shell
java -p com.horstmann.greet.jar:v2ch09.exportedpkg.jar -m v2ch09.exportedpkg
```

创建JAR文件时，可以使用`--module-version`选项指定版本号，并在JAR文件名中添加`@`和版本号：

```shell
jar -cvf com.horstmann.greet@1.0.jar --module-version 1.0 -C com.horstmann.greet .
```

如9.1节所述，Java平台模块系统并不会使用版本号来解析模块，但是可以通过其他工具和框架来查询版本号。

注释：可以通过反射API找到版本号。在上面的例子中：

```java
Optional<String> version = Greeter.class.getModule().getDescriptor().rawVersion();
    // contains the version string "1.0"
```

注释：对于模块，等价于类加载器的是**层**(layer)。Java平台模块系统会将JDK模块和应用程序模块加载到**启动层**(boot layer)。程序可以使用层API加载其他模块。

提示：如果想要将模块加载到JShell中，需要将JAR包含在模块路径中，并使用`--add-modules`选项：

```shell
jshell --module-path com.horstmann.greet@1.0.jar --add-modules com.horstmann.greet
```

## 9.7 模块和反射式访问
在过去，总是可以通过使用反射来克服讨厌的访问限制。正如在卷I第5章（5.9.5节）中看到的，反射可以访问任何类的私有成员。

然而，在模块化的程序中，这不再成立。如果类在一个模块中，通过反射访问非公有成员将会失败。但是，有许多使用反射式访问的库。典型的例子包括自动将对象持久化到数据库的对象-关系映射器(object-relational mappers, ORM)（如[JPA](https://spring.io/projects/spring-data-jpa)），以及在对象和XML或JSON等格式之间转换的库（如[JAXB](https://javaee.github.io/jaxb-v2/)和[JSON-B](https://javaee.github.io/jsonb-spec/)）。

如果使用这种库，并且还想使用模块，那么就必须格外小心。为了演示这个问题，我们将卷I第5章中的`ObjectAnalyzer`类放到`com.horstmann.util`模块中，这个类的`toString()`方法使用反射来打印对象的字段。单独的`v2ch09.openpkg`模块包含一个简单的`Country`类：

```java
package com.horstmann.places;

public class Country {
    private String name;
    private double area;

    public Country(String name, double area) {
        this.name = name;
        this.area = area;
    }
    // ...
}
```

下面的程序演示了如何分析一个`Country`对象：

```java
package com.horstmann.places;

import com.horstmann.util.ObjectAnalyzer;

public class Demo {
    public static void main(String[] args) throws ReflectiveOperationException {
        var belgium = new Country("Belgium", 30510);
        var analyzer = new ObjectAnalyzer();
        System.out.println(analyzer.toString(belgium));
    }
}
```

[com.horstmann.util](https://github.com/ZZy979/Core-Java-code/tree/main/v2ch09/com.horstmann.util)

[v2ch09.openpkg](https://github.com/ZZy979/Core-Java-code/tree/main/v2ch09/v2ch09.openpkg)

现在编译两个模块和`Demo`程序：

```shell
javac com.horstmann.util/module-info.java com.horstmann.util/com/horstmann/util/ObjectAnalyzer.java
javac -p com.horstmann.util v2ch09.openpkg/module-info.java v2ch09.openpkg/com/horstmann/places/*.java
java -p v2ch09.openpkg:com.horstmann.util -m v2ch09.openpkg/com.horstmann.places.Demo
```

程序会抛出异常：

```
Exception in thread "main" java.lang.reflect.InaccessibleObjectException: Unable to make field private java.lang.String com.horstmann.places.Country.name accessible: module v2ch09.openpkg does not "opens com.horstmann.places" to module com.horstmann.util
```

当然，理论上，违反封装并访问对象的私有成员是错误的。但是像ORM或XML/JSON绑定这样的机制非常常见，因此模块系统必须顾及它们。

模块可以使用`opens`关键字**开放**一个包，从而允许反射式访问给定包中的类的所有实例。

```java
module v2ch09.openpkg {
    requires com.horstmann.util;
    opens com.horstmann.places;
}
```

这样`ObjectAnalyzer`就可以正确地工作了。程序输出如下：

```
com.horstmann.places.Country[name=Belgium,area=30510.0][]
```

模块可以声明为开放的(`open`)，例如：

```java
open module v2ch09.openpkg {
    requires com.horstmann.util;
}
```

开放模块允许在运行时访问其所有包，就像所有包都用`exports`和`opens`声明一样。但是，只有显式导出的包才能在编译时访问。开放模块将模块系统的编译时安全性与经典的放任的运行时行为相结合。

回忆一下卷I第5章（5.9.3节），JAR文件除了类文件和清单，还可以包含资源。资源可以使用`Class.getResourceAsStream()`方法加载，现在还可以使用`Module.getResourceAsStream()`方法。如果资源位于模块的某个包的目录中，那么这个包必须是对调用者所属模块开放的。其他目录中的资源以及类文件和清单可以被任何人读取。

作为一个更实际的例子，我们使用JSON-B规范将`Country`对象转换为JSON。为了使用JSON-B的Yasson实现，需要从[Maven Central Repository](https://mvnrepository.com/)下载以下JAR文件（这些JAR都是模块化的）：
* [jakarta.json-api-2.0.1.jar](https://repo1.maven.org/maven2/jakarta/json/jakarta.json-api/2.0.1/jakarta.json-api-2.0.1.jar)
* [jakarta.json.bind-api-2.0.0.jar](https://repo1.maven.org/maven2/jakarta/json/bind/jakarta.json.bind-api/2.0.0/jakarta.json.bind-api-2.0.0.jar)
* [jakarta.json-2.0.1-module.jar](https://repo1.maven.org/maven2/org/glassfish/jakarta.json/2.0.1/jakarta.json-2.0.1-module.jar)
* [yasson-2.0.3.jar](https://repo1.maven.org/maven2/org/eclipse/yasson/2.0.3/yasson-2.0.3.jar)

将这些JAR文件添加到模块路径，然后运行`v2ch09.openpkg2`模块的`Demo`程序。只有当`com.horstmann.places`包对`org.eclipse.yasson`模块开放时，才能成功转换为JSON。

[v2ch09.openpkg2](https://github.com/ZZy979/Core-Java-code/tree/main/v2ch09/v2ch09.openpkg2)

```shell
javac -p jakarta.json-api-2.0.1.jar:jakarta.json.bind-api-2.0.0.jar:jakarta.json-2.0.1-module.jar:yasson-2.0.3.jar \
  v2ch09.openpkg2/module-info.java v2ch09.openpkg2/com/horstmann/places/*.java
java -p jakarta.json-api-2.0.1.jar:jakarta.json.bind-api-2.0.0.jar:jakarta.json-2.0.1-module.jar:yasson-2.0.3.jar:v2ch09.openpkg2 \
  -m v2ch09.openpkg2/com.horstmann.places.Demo
```

程序输出如下：

```
{"area":30510.0,"name":"Belgium"}
```

注：如果将`Country`的字段声明为`public`，程序会抛出另一个异常，如下所示。将`opens`语句后面的限定删除可以修复这一问题。

```
Exception in thread "main" jakarta.json.bind.JsonbException: Error accessing field 'area' declared in 'class com.horstmann.places.Country'
        at org.eclipse.yasson@2.0.3/org.eclipse.yasson.internal.model.PropertyModel.createReadHandle(PropertyModel.java:550)
        at org.eclipse.yasson@2.0.3/org.eclipse.yasson.internal.model.PropertyModel.<init>(PropertyModel.java:167)
        at org.eclipse.yasson@2.0.3/org.eclipse.yasson.internal.ClassParser.lambda$parseProperties$0(ClassParser.java:70)
        ...
        at org.eclipse.yasson@2.0.3/org.eclipse.yasson.internal.Marshaller.marshall(Marshaller.java:101)
        at org.eclipse.yasson@2.0.3/org.eclipse.yasson.internal.JsonBinding.toJson(JsonBinding.java:126)
        at v2ch09.openpkg2/com.horstmann.places.Demo.main(Demo.java:16)
Caused by: java.lang.IllegalAccessException: access to public member failed: com.horstmann.places.Country.area/double/getField, from public Lookup
        at java.base/java.lang.invoke.MemberName.makeAccessException(MemberName.java:955)
        at java.base/java.lang.invoke.MethodHandles$Lookup.checkAccess(MethodHandles.java:3882)
        at java.base/java.lang.invoke.MethodHandles$Lookup.checkField(MethodHandles.java:3832)
        ...
```

### 小结
模块系统提供的安全性：
* 编译时：模块只能访问依赖(`requires`)的模块中导出(`exports`)的包。
* 运行时：只有模块中开放(`opens`)的包允许反射式访问。

## 9.8 自动模块
如果从全新的项目开始，其中所有代码都由自己编写，那么你可以设计模块、声明模块依赖，并将应用程序打包成模块化的JAR文件。

然而，这是极其罕见的场景。几乎所有项目都依赖第三方库。当然，你可以等到所有库的提供者都将其转换成模块，然后模块化自己的代码。但如果不想等，Java平台模块系统提供了两种机制来跨越模块化前后之间的鸿沟：自动模块和无名模块。

如果是为了迁移，可以通过**把任何JAR文件置于模块路径而不是类路径上**，从而将其转换成一个模块。模块路径上非模块化（即没有module-info.class）的JAR文件叫做**自动模块**(automatic module)。自动模块具有以下属性：
* 隐式地依赖所有其他模块。
* 其所有包都是导出、开放的。
* 如果清单文件(META-INF/MANIFEST.MF)中具有键为`Automatic-Module-Name`的条目，其值将变为模块名；否则，从JAR文件名获得模块名：删除结尾的版本号，并将非字母数字字符替换为句点。

前两条规则表明，自动模块中的包的行为和在类路径上一样。使用模块路径的原因是使其他模块可以依赖该模块。

例如，假设要实现一个处理CSV文件的模块，使用[Apache Commons CSV](https://commons.apache.org/proper/commons-csv/)库。如果将[commons-csv-1.9.0.jar](https://repo1.maven.org/maven2/org/apache/commons/commons-csv/1.9.0/commons-csv-1.9.0.jar)添加到模块路径，那么你的模块就可以引用该模块，其名字是`commons.csv`（如果维护者使用`org.apache.commons.csv`作为模块名会更好）。

注释：在将第三方JAR放到模块路径之前，先检查它们是否是模块化的。如果不是，仍然可以将其转换成自动模块，但是要准备好以后更新模块名。

`v2ch9.automod`模块包含一个读取国家数据CSV文件[countries.csv](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch09/countries.csv)的简单程序：

[v2ch09.automod](https://github.com/ZZy979/Core-Java-code/tree/main/v2ch09/v2ch09.automod)

下面是编译和运行该程序的命令：

```shell
javac -p commons-csv-1.9.0.jar v2ch09.automod/module-info.java \
  v2ch09.automod/com/horstmann/places/CSVDemo.java
java -p v2ch09.automod:commons-csv-1.9.0.jar -m v2ch09.automod/com.horstmann.places.CSVDemo
```

## 9.9 无名模块
任何不在模块路径中的类都是**无名模块**(unnamed module)的一部分。与自动模块一样，无名模块可以访问所有其他模块，并且其所有包都是导出、开放的。

不过，显式模块不能访问无名模块。**显式模块**(explicit module)是模块路径上有module-info.class的模块（如模块化的JAR）。换句话说，显式模块可以避免“[JAR地狱](https://dzone.com/articles/what-is-jar-hell)”（指传统的JAR文件+类路径方式存在的传递依赖、遮蔽、版本冲突等问题）。

例如，考虑前一节的程序。假设将commons-csv-1.9.0.jar放到类路径而不是模块路径上：

```shell
java --module-path v2ch09.automod --class-path commons-csv-1.9.0.jar -m v2ch09.automod/com.horstmann.places.CSVDemo
```

现在程序将无法启动：

```shell
Error occurred during initialization of boot layer
java.lang.module.FindException: Module commons.csv not found, required by v2ch09.automod
```

因此，迁移到Java平台模块系统必须是一个自底向上的过程：
1. JDK自身被模块化。
2. 之后，库被模块化（要么使用自动模块，要么将其转换为显式模块）。
3. 一旦应用程序依赖的所有库都被模块化，就可以将应用程序的代码转换为一个模块。

注释：自动模块可以读取无名模块，因此其依赖关系可以放在类路径中。

## 9.10 用于迁移的命令行标志
从Java 11起，编译时封装是严格强制执行的。但是，在Java 16之前，运行时反射式访问是允许的。从Java 16起，运行时反射式访问也是强制的（即必须使用`opens`）。为了给用户时间来应对这种变化，Java 9到16的`java`命令有一个`--illegal-access=mode`标志，其中`mode`有4种可能的设置：
* `permit`：允许非法访问（相当于每个模块中的每个包都是开放的），并在第一次非法访问时打印警告消息。这是Java 9的默认行为。
* `warn`：等同于`permit`，但是对每次非法访问都打印警告消息。
* `debug`：等同于`warn`，但是对每次非法访问都打印警告消息和栈轨迹。
* `deny`：拒绝所有非法访问。这是Java 16的默认行为。

这个标志在Java 17中已经不可用了。

`--add-exports`和`--add-opens`标志允许指定模块导出和开放的包（覆盖模块声明），其参数格式为`module/package=target-module`，表示将指定的模块和包导出/开放到目标模块。目标模块可以用`ALL-UNNAMED`表示无名模块。

例如，假设一个遗留的应用程序使用了内部API `CachedRowSetImpl`（在`java.sql.rowset`模块未导出的`com.sun.rowset`包中）。最好的解决方案是修改实现。但假设你不能访问源代码，此时可以用`--add-exports`标志启动程序：

```shell
java --add-exports java.sql.rowset/com.sun.rowset=ALL-UNNAMED -jar MyApp.jar
```

无名模块内部的反射是可以的，但是反射式访问JDK类的非公有成员不再可行了。例如，有些动态生成Java类的库会通过反射来调用受保护的`ClassLoader.defineClass()`方法。如果应用程序使用了这样的库，就需要在启动时添加标志`--add-opens java.base/java.lang=ALL-UNNAMED`。（注：5.9.5节中的`ObjectAnalyzerTest`程序就使用了这个标志）

遗留程序的命令行选项可能多得吓人。为了更好地管理多个选项，可以将它们放到一个或多个用`@`前缀指定的文件中。例如，

```shell
java @options1 @options2 -jar MyProg.java
```

选项文件有一些语法规则：
* 用空格、制表符或换行来分隔选项。
* 用双引号将包含空格的参数括起来，例如`"Program Files"`。
* 以`\`结尾的行会与下一行合并。
* 反斜杠必须转义，例如`C:\\Users\\Fred`。
* 注释以`#`开头。

## 9.11 传递和静态依赖
在9.4节中已经看到了`requires`语句的基本形式。本节将介绍偶尔会用到的两种变体。

在某些情况下，对于模块的用户来说声明所有依赖的模块可能会很繁琐。例如，`java.desktop`模块依赖三个模块：`java.prefs`、`java.datatransfer`和`java.xml`。其中，`java.prefs`模块只在内部使用，但是后两个模块中的类出现在了公共API中，例如：

```java
java.awt.datatransfer.Clipboard java.awt.Toolkit.getSystemClipboard()
java.beans.XMLDecoder(org.xml.sax.InputSource is)
```

这不应该是`java.desktop`模块的用户应该考虑的问题。因此，`java.desktop`模块使用`transitive`修饰符声明依赖：

```java
module java.desktop {
    requires java.prefs;
    requires transitive java.datatransfer;
    requires transitive java.xml;
    ...
}
```

任何依赖`java.desktop`的模块都会自动地依赖这两个模块。

`requires transitive`语句的一种很有吸引力的用法是**聚合模块**(aggregator module)，即没有任何包、只有传递依赖的模块。`java.se`就是一个这样的模块，它传递依赖了所有JDK模块。对细粒度模块依赖不感兴趣的程序员可以直接依赖`java.se`，这样就会得到Java SE平台的所有模块。

最后，还有一种不常见的`requires static`变体，它声明一个模块必须在编译时出现，但在运行时是可选的。有两个用例：
1. 访问在不同模块中声明的编译时处理的注解。
2. 如果位于不同模块中的类可用就使用它，否则执行其他操作。例如：

```java
try {
    new oracle.jdbc.driver.OracleDriver();
    ...
}
catch (NoClassDefFoundError er) {
    // Do something else
}
```

## 9.12 限定导出和开放
本节将介绍`exports`和`opens`语句的一种变体，使用关键字`to`将其作用域缩窄到一组指定的模块。例如，`java.base`模块声明包含语句

```java
exports sun.net to
    java.net.http,
    jdk.naming.dns;
```

这样的语句称为**限定导出**(qualified export)。只有列出的模块可以访问导出的包，而其他模块不能。

类似地，可以将`opens`语句限制到特定模块。例如，在9.7节中，可以使用如下的限定`opens`语句：

```java
module v2ch09.openpkg {
    requires com.horstmann.util;
    opens com.horstmann.places to com.horstmann.util;
}
```

现在，`com.horstmann.places`包只对`com.horstmann.util`模块开放。

## 9.13 服务加载
`ServiceLoader`类（参见卷I第6章 6.4节）提供了一种用于将服务接口与实现匹配的轻量级机制。Java平台模块系统使这种机制更易于使用。

服务有一个接口以及一个或多个可能的实现。下面是一个简单的接口示例：

```java
public interface GreeterService {
    String greet(String subject);
    Locale getLocale();
}
```

一个或多个模块提供了实现，例如：

```java
public class FrenchGreeter implements GreeterService {
    public String greet(String subject) { return "Bonjour " + subject; }
    public Locale getLocale() { return Locale.FRENCH; }
}
```

服务的用户在所有提供的实现中选择一个。

```java
ServiceLoader<GreeterService> greeterLoader = ServiceLoader.load(GreeterService.class);
GreeterService chosenGreeter;
for (GreeterService greeter : greeterLoader) {
    if (...) {
        chosenGreeter = greeter;
    }
}
```

在过去，实现是通过将文本文件放到包含实现类的JAR文件的META-INF/services目录中来提供的（例如卷I第6章中的[serviceLoader.Cipher](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch06/META-INF/services/serviceLoader.Cipher)）。模块系统提供了一种更好的方式。提供服务实现的模块添加一条`provides`语句，列出服务接口（可能定义在任何模块中）和实现类（必须是该模块的一部分）。下面是来自`jdk.security.auth`模块的一个例子：

```java
module jdk.security.auth {
    ...
    provides javax.security.auth.spi.LoginModule with
        com.sun.security.auth.module.Krb5LoginModule,
        com.sun.security.auth.module.UnixLoginModule,
        com.sun.security.auth.module.JndiLoginModule,
        com.sun.security.auth.module.KeyStoreLoginModule,
        com.sun.security.auth.module.LdapLoginModule,
        com.sun.security.auth.module.NTLoginModule;
}
```

这等价于META-INF/services中的文件。

消费（使用该服务的）模块包含一条`uses`语句：

```java
module java.base {
    ...
    uses javax.security.auth.spi.LoginModule;
}
```

当消费模块中的代码调用`ServiceLoader.load(ServiceInterface.class)`时，匹配的实现类将被加载，尽管它们可能在未导出的包中。

在我们的示例中，`com.horstmann.greetsvc`模块在`com.horstmann.greetsvc.internal`包中为德语和法语问候者提供了实现：

```java
module com.horstmann.greetsvc {
    exports com.horstmann.greetsvc;

    provides com.horstmann.greetsvc.GreeterService with
        com.horstmann.greetsvc.internal.FrenchGreeter,
        com.horstmann.greetsvc.internal.GermanGreeterFactory;
}
```

[com.horstmann.greetsvc](https://github.com/ZZy979/Core-Java-code/tree/main/v2ch09/com.horstmann.greetsvc)

`v2ch09.useservice`模块会消费该服务。使用`ServiceLoader`工具挑选与期望语言匹配的服务：

```java
package com.horstmann.hello;

import com.horstmann.greetsvc.GreeterService;

import java.util.ServiceLoader;

public class HelloWorld {
    public static void main(String[] args) {
        ServiceLoader<GreeterService> greeterLoader = ServiceLoader.load(GreeterService.class);
        String desiredLanguage = args.length > 0 ? args[0] : "de";
        GreeterService chosenGreeter = null;
        for (GreeterService greeter : greeterLoader) {
            if (greeter.getLocale().getLanguage().equals(desiredLanguage))
                chosenGreeter = greeter;
        }
        if (chosenGreeter == null)
            System.out.println("No suitable greeter.");
        else
            System.out.println(chosenGreeter.greet("Modular World"));
    }
}
```

该模块声明依赖服务模块并使用`GreeterService`：

```java
module v2ch09.useservice {
    requires com.horstmann.greetsvc;
    uses com.horstmann.greetsvc.GreeterService;
}
```

由于`provides`和`uses`声明，消费模块允许访问私有（未导出的）实现类。

[v2ch09.useservice](https://github.com/ZZy979/Core-Java-code/tree/main/v2ch09/v2ch09.useservice)

为了构建并运行该程序，首先编译服务模块：

```shell
javac com.horstmann.greetsvc/module-info.java \
  com.horstmann.greetsvc/com/horstmann/greetsvc/GreeterService.java \
  com.horstmann.greetsvc/com/horstmann/greetsvc/internal/*.java
```

然后编译并运行消费模块：

```shell
javac -p com.horstmann.greetsvc v2ch09.useservice/module-info.java \
  v2ch09.useservice/com/horstmann/hello/HelloWorld.java
java -p com.horstmann.greetsvc:v2ch09.useservice -m v2ch09.useservice/com.horstmann.hello.HelloWorld
```

## 9.14 使用模块的工具
[`jdeps`工具](https://docs.oracle.com/en/java/javase/17/docs/specs/man/jdeps.html)可以分析一组给定的JAR文件之间的依赖关系。例如，假设你想模块化[JUnit 4](https://junit.org/junit4/)。下载[junit-4.12.jar](https://repo1.maven.org/maven2/junit/junit/4.12/junit-4.12.jar)及其依赖的[hamcrest-core-1.3.jar](https://repo1.maven.org/maven2/org/hamcrest/hamcrest-core/1.3/hamcrest-core-1.3.jar)，之后运行

```shell
jdeps -s junit-4.12.jar hamcrest-core-1.3.jar
```

`-s`标志生成总结性的输出：

```
hamcrest-core-1.3.jar -> java.base
junit-4.12.jar -> hamcrest-core-1.3.jar
junit-4.12.jar -> java.base
junit-4.12.jar -> java.management
```

这告诉你模块图：

![JUnit的模块图](/assets/images/java-note-v2ch09-the-java-platform-module-system/JUnit的模块图.png)

如果省略`-s`标志，将得到模块总结，后面跟着包到依赖的包和模块的映射。如果添加`-v`标志，则会列出类到依赖的包和模块的映射。

`--generate-module-info`选项会对每个分析的模块生成module-info.java文件：

```shell
jdeps --generate-module-info /tmp/junit junit-4.12.jar hamcrest-core-1.3.jar
```

注释：还有一个`-dotoutput`选项可以用[DOT语言](https://graphviz.org/doc/info/lang.html)生成用于描述图的图形化输出。假设已经安装了[dot工具](https://graphviz.org/)，运行以下命令：

```shell
jdeps -s -dotoutput /tmp/junit junit-4.12.jar hamcrest-core-1.3.jar
dot -Tpng /tmp/junit/summary.dot > /tmp/junit/summary.png
```

就会得到下面的summary.png：

![summary.png](/assets/images/java-note-v2ch09-the-java-platform-module-system/summary.png)

[`jlink`工具](https://docs.oracle.com/en/java/javase/17/docs/specs/man/jlink.html)用于生成无需单独的Java运行时环境即可执行的应用程序。生成的镜像比整个JDK要小得多。可以指定要包含的模块和输出目录：

```shell
jlink --module-path com.horstmann.greet.jar:v2ch09.exportedpkg.jar:$JAVA_HOME/jmods \
  --add-modules v2ch09.exportedpkg --output /tmp/hello
```

输出目录有一个子目录bin，其中包含可执行文件java。如果运行

```shell
bin/java -m v2ch09.exportedpkg
```

该模块的主类的`main()`方法就会被调用。

`jlink`的关键是它将运行应用程序所需的最小模块集打包在一起。可以列出所有模块：

```shell
bin/java --list-modules
```

在这个例子中，输出如下：

```
v2ch09.exportedpkg
com.horstmann.greet
java.base@17.0.12
```

所有模块都包含在**运行时镜像**(runtime image)文件lib/modules中。在我的计算机上，这个文件只有24.3 MB，而所有JDK模块($jdk/lib/modules)有119 MB。

这可以作为打包应用程序的工具的基础。你仍然需要生成针对多平台的文件集，以及应用程序的启动脚本。

注释：可以使用`jimage`命令来查看运行时镜像。

最后，[`jmod`工具](https://docs.oracle.com/en/java/javase/17/docs/specs/man/jmod.html)用于构建和检查与JDK一起包含的模块文件(.jmod)。查看$jdk/jmods目录，会发现每个模块都有一个扩展名为jmod的文件。而不再有rt.jar文件。

与JAR文件一样，模块文件也包含类文件。此外，还可以包含本地代码库、命令、头文件、配置文件和法律声明。模块文件使用ZIP格式，可以用任何ZIP工具查看其内容。

与JAR文件不同，模块文件只在链接（即生成运行时镜像）时才有用。你不需要生成模块文件，除非想要将二进制文件（如本地代码库）与模块捆绑在一起。
