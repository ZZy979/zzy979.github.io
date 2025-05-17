---
title: Maven使用教程
date: 2025-05-15 14:14 +0800
categories: [Build Tools]
tags: [build tools, maven]
---
## 1.简介
Maven是一个软件项目管理工具，主要用于Java项目的自动构建和依赖管理。
* 官方网站：<https://maven.apache.org/>
* 官方文档：<https://maven.apache.org/guides/index.html>

## 2.安装
<https://maven.apache.org/install.html>

1. 首先，确保已经安装了JDK 8+。
2. 从[Maven下载页面](https://maven.apache.org/download.cgi)下载二进制包，并解压到任意目录。
3. 将解压目录中的bin目录添加到`PATH`环境变量。
4. 使用`mvn -v`命令验证安装是否成功：

```shell
$ mvn -v
Apache Maven 3.9.9 (8e8579a9e76f7d015ee5ec7bfcdc97d260186937)
Maven home: /usr/local/apache-maven-3.9.9
Java version: 1.8.0_412, vendor: Tencent, runtime: /usr/local/jdk1.8.0_412.jdk/Contents/Home/jre
Default locale: zh_CN, platform encoding: UTF-8
OS name: "mac os x", version: "14.5", arch: "aarch64", family: "mac"
```

另外，也可以使用包管理工具安装Maven。例如使用Homebrew：

```shell
brew install maven
```

或者使用APT：

```shell
sudo apt install maven
```

## 3.基本用法
官方入门教程：
* [Maven in 5 Minutes](https://maven.apache.org/guides/getting-started/maven-in-five-minutes.html)
* [Maven Getting Started Guide](https://maven.apache.org/guides/getting-started/index.html)

### 3.1 创建项目
使用Maven的[Archetype插件](https://maven.apache.org/archetype/maven-archetype-plugin/)可以从[模板](https://maven.apache.org/guides/introduction/introduction-to-archetypes.html)(Archetype)快速创建一个Maven项目：

```shell
mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=my-app -DarchetypeArtifactId=maven-archetype-quickstart -DarchetypeVersion=1.5 -DinteractiveMode=false
```

生成的项目具有如下的[标准目录结构](https://maven.apache.org/guides/introduction/introduction-to-the-standard-directory-layout.html)：

```
my-app/
  pom.xml
  src/
    main/
      java/
        com/
          mycompany/
            app/
              App.java
    test/
      java/
        com/
          mycompany/
            app/
              AppTest.java
```

* pom.xml：项目的核心配置文件，定义了依赖、插件、构建配置等
* src/main/java：项目的源代码目录
* src/test/java：项目的测试代码目录

### 3.2 编译项目
在项目根目录中执行以下命令来编译项目：

```shell
mvn compile
```

类文件会生成在target/classes目录中。

### 3.3 运行测试
执行以下命令来运行项目的单元测试：

```shell
mvn test
```

### 3.4 打包项目
将项目打包成JAR文件：

```shell
mvn package
```

打包后的JAR文件会生成在target目录中。可以通过以下命令运行打包的应用程序：

```shell
$ java -cp target/my-app-1.0-SNAPSHOT.jar com.mycompany.app.App
Hello World!
```

或者也可以使用[exec-maven-plugin](https://www.mojohaus.org/exec-maven-plugin/)插件运行：

```shell
mvn exec:java -Dexec.mainClass=com.mycompany.app.App
```

这样生成的JAR不包含依赖，不能直接使用`java -jar`命令执行。要生成包含依赖的JAR，可以使用[maven-assembly-plugin](https://maven.apache.org/plugins/maven-assembly-plugin/)插件，在pom.xml中添加以下配置：

```xml
<build>
    <plugins>
        <plugin>
            <artifactId>maven-assembly-plugin</artifactId>
            <configuration>
                <descriptorRefs>
                    <descriptorRef>jar-with-dependencies</descriptorRef>
                </descriptorRefs>
                <archive>
                    <manifest>
                        <mainClass>com.mycompany.app.App</mainClass>
                    </manifest>
                </archive>
            </configuration>
            <executions>
                <execution>
                    <id>make-assembly</id>
                    <phase>package</phase>
                    <goals>
                        <goal>single</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

其中，`<mainClass>`指定JAR文件的主类，`<execution>`将插件目标`assembly:single`绑定到`package`阶段。

现在执行`mvn package`命令，将在target目录中生成my-app-1.0-SNAPSHOT-jar-with-dependencies.jar，其中包含了当前项目和所有依赖的类文件（不包括`provided`依赖和排除的传递依赖）。可以直接使用`java -jar`命令来执行：

```shell
$ java -jar my-app-1.0-SNAPSHOT-jar-with-dependencies.jar
Hello World!
```

### 3.5 安装到本地仓库
将项目安装到本地仓库：

```shell
mvn install
```

本地仓库位于$HOME/.m2/repository目录，这个示例项目将被安装到其中的com/mycompany/app/my-app目录。其他项目可以依赖它：

```xml
<dependency>
    <groupId>com.mycompany.app</groupId>
    <artifactId>my-app</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

### 3.6 清理项目
删除生成的target目录：

```shell
mvn clean
```

### 3.7 资源文件
如果有资源文件，应该将其放在src/main/resources目录中，在打包时会与类文件一起打包到JAR中。测试资源应该放在src/test/resources目录中。

```
my-app/
  pom.xml
  src/
    main/
      java/
        com/mycompany/app/
          App.java
      resources/
        application.properties
    test/
      java/
        com/mycompany/app/
          AppTest.java
      resources/
        test.properties
```

在代码中，可以通过如下方式访问资源文件：

```java
InputStream is = getClass().getResourceAsStream("/application.properties");
```

### 3.8 添加依赖
在pom.xml文件的`<dependencies>`中添加项目所需的依赖。

例如，添加JUnit依赖：

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13.2</version>
    <scope>test</scope>
</dependency>
```

下次构建时，Maven会自动从中央仓库(<https://mvnrepository.com/>)下载所需的依赖库。

### 3.9 使用插件
Maven的核心是一个插件执行框架，大部分工作都是由**插件**(plugin)完成的。例如，`mvn compile`命令实际上执行了`compiler`插件的`compile`目标，因此等价于`mvn compiler:compile`。

可以通过在pom.xml文件的`<build>`中添加或配置插件来自定义项目的构建。例如，可以通过配置maven-compiler-plugin插件指定Java版本（等价于Java编译器的`-source`和`-target`选项）：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.13.0</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
        </plugin>
    </plugins>
    ...
</build>
```

或者也可以直接指定`maven.compiler.source`和`maven.compiler.target`属性：

```xml
<properties>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
</properties>
```

对于JDK 9+，推荐使用`release`参数（等价于Java编译器的`--release`选项）：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.13.0</version>
            <configuration>
                <release>17</release>
            </configuration>
        </plugin>
    </plugins>
    ...
</build>
```

或者直接指定`maven.compiler.release`属性：

```xml
<properties>
    <maven.compiler.release>17</maven.compiler.release>
</properties>
```

参见[Setting the -source and -target of the Java Compiler](https://maven.apache.org/plugins/maven-compiler-plugin/examples/set-compiler-source-and-target.html)和[Setting the --release of the Java Compiler](https://maven.apache.org/plugins/maven-compiler-plugin/examples/set-compiler-release.html)。

Maven支持的所有插件列表：<https://maven.apache.org/plugins/>

## 4.生命周期
<https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html>

Maven基于**构建生命周期**(build lifecycle)这一核心概念。有三种内置的生命周期：
* `default`负责项目的构建和部署
* `clean`负责项目清理
* `site`负责创建项目网站

### 4.1 阶段
每个生命周期都由一系列**阶段**(phase)组成。默认生命周期主要包括以下阶段：
* `validate`：验证项目是否正确，以及所有必要信息是否可用
* `compile`：编译项目的源代码
* `test`：使用合适的单元测试框架测试编译后的源代码
* `package`：将编译后的代码打包为可分发格式（例如JAR）
* `verify`：运行检查来验证包是否有效并符合质量标准
* `install`：将包安装到本地仓库，以便在本地其他项目中用作依赖
* `deploy`：在集成或发布环境中完成，将最终包上传到远程仓库，以便与其他开发者共享

生命周期阶段的完整列表参见[Lifecycle Reference](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#Lifecycle_Reference)。

这些生命周期阶段是**依次**执行的，在命令行中只需指定要执行的最后一个阶段。例如，命令

```shell
mvn verify
```

将依次执行`validate`、`compile`、`test`、`package`和`verify`等阶段。

也可以在一条命令中执行多个阶段。例如：

```shell
mvn clean deploy
```

### 4.2 插件目标
阶段负责构建生命周期中的一个特定步骤，但执行方式可能有所不同（例如`package`阶段的打包方式可能是JAR或WAR等）。这是通过声明与阶段绑定的插件目标实现的。

**插件目标**(plugin goal)表示一项特定的任务（比阶段更精细），其名称形如`plugin:goal`（例如前面用过的`archetype:generate`）。一个目标可能绑定到零个或多个阶段。如果阶段绑定了一个或多个目标，它将执行所有这些目标。

每种打包类型都包含一系列绑定到特定阶段的目标。例如，`jar`打包将以下目标绑定到默认生命周期的构建阶段：

| 阶段 | 目标 |
| --- | --- |
| `process-resources` | `resources:resources` |
| `compile` | `compiler:compile` |
| `process-test-resources` | `resources:testResources` |
| `test-compile` | `compiler:testCompile` |
| `test` | `surefire:test` |
| `package` | `jar:jar` |
| `install` | `install:install` |
| `deploy` | `deploy:deploy` |

阶段-目标绑定关系的完整列表参见[Built-in Lifecycle Bindings](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#Built-in_Lifecycle_Bindings)。

## 5.POM
POM表示**项目对象模型**(Project Object Model)，它是Maven项目的XML表示，保存在名为**pom.xml**的文件中。

POM的根元素是`<project>`，常用的子元素如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- The Basics -->
    <groupId>...</groupId>
    <artifactId>...</artifactId>
    <version>...</version>
    <packaging>...</packaging>
    <properties>...</properties>

    <!-- More Project Information -->
    <name>...</name>
    <description>...</description>
    <url>...</url>

    <!-- Dependencies -->
    <dependencyManagement>...</dependencyManagement>
    <dependencies>...</dependencies>
    <parent>...</parent>
    <modules>...</modules>

    <!-- Build Settings -->
    <build>...</build>
    <reporting>...</reporting>
</project>
```

完整结构参见[POM Reference](https://maven.apache.org/pom.html)和[Maven Model Reference](https://maven.apache.org/ref/current/maven-model/maven.html#class_dependency)。

### 5.1 Maven坐标
以下三个元素是必需的：
* **groupId**：项目的组织或公司名称（如`org.apache.maven`）。groupId不一定使用点符号，也不必与项目的包结构对应，不过这种做法是一个好习惯。
* **artifactId**：项目名称（如`maven-project`）。
* **version**：版本号（如`2.2.1`或`3.0-alpha-1`）。

`groupId:artifactId:version`唯一标识了一个Maven项目（就像坐标一样）。例如，JUnit 4.13.2版本的坐标为`junit:junit:4.13.2`。

### 5.2 打包
`<packaging>`指定项目的打包类型，可以是`pom`、`jar`（默认）、`maven-plugin`、`ejb`、`war`、`ear`或`rar`。

打包类型定义了生命周期阶段绑定的目标，参见[Plugin Bindings for default Lifecycle Reference](https://maven.apache.org/ref/current/maven-core/default-bindings.html)。

### 5.3 属性
**属性**(properties)是一种占位符（可理解为变量）。其值可以在POM中的任何地方使用`${x}`语法访问。或者也可以被插件用作默认值，例如：

```xml
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
</properties>
```

属性有五种不同的风格：
* `env.X`：返回Shell的环境变量。例如，`${env.PATH}`包含`PATH`环境变量。注意：Maven会将环境变量名称转换为全大写。
* `project.x`：返回POM中对应元素的值。例如，`<project><version>1.0</version></project>`可以通过`${project.version}`访问。
* `settings.x`：返回设置文件settings.xml中对应元素的值。例如，`<settings><offline>false</offline></settings>`可以通过`${settings.offline}`访问。
* Java系统属性：所有可通过`System.getProperties()`访问的属性，例如`${java.home}`。
* `x`：POM的`<properties>`中设置的属性。例如，`<properties><someVar>value</someVar></properties>`可以通过`${someVar}`访问。

## 6.依赖管理
<https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html>

依赖管理是Maven的核心特性之一。Maven通过pom.xml文件定义和管理项目的依赖。在构建时能够自动下载所需的依赖，并设置正确的类路径和库版本，从而解决了在依赖关系复杂的大型项目中存在的 "Jarmageddon" 和 "Jar Hell" 问题。

### 6.1 依赖声明
在pom.xml文件的`<dependencies>`中声明项目所需的依赖。例如：

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13.2</version>
    <scope>test</scope>
</dependency>
```

* groupId：依赖的组织或公司名称
* artifactId：依赖的项目名称
* version：依赖的版本号
* scope：依赖的作用域（详见下一节）

### 6.2 作用域
作用域定义了依赖在构建生命周期中的使用阶段（即添加到哪些阶段的类路径）。有6种作用域：
* `compile`：默认作用域，编译、测试和运行时都可用，具有传递性。
* `test`：仅在测试时可用（如JUnit），不具有传递性。
* `provided`：编译和测试时可用，运行时由JDK或容器提供（如Servlet API），不具有传递性。
* `runtime`：仅在测试和运行时可用（如JDBC驱动）。
* `system`：类似于`provided`，但需要显式指定本地JAR路径，而不会从仓库下载。
* `import`：用于从其他POM文件中导入依赖配置。

### 6.3 传递依赖
Maven会自动包含传递依赖，因此无需指定项目依赖所需的依赖库。Maven会根据依赖的作用域和传递性规则自动解析依赖树。

例如，如果A依赖B（`compile`作用域），B依赖C（`compile`作用域），那么A会自动依赖C；如果B依赖C（`test`作用域），那么A不会自动依赖C。

完整的传递依赖作用域规则参见[Dependency Scope](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#Dependency_Scope)。

当遇到同一个依赖的多个版本时，Maven会按照**最近优先原则**选择依赖版本（即选择依赖树中离当前项目最近的依赖的版本）。如果两个依赖版本的距离相同，则选择第一个声明的版本。

例如，在下面的依赖树中，会使用D 1.0，因为路径A -> E -> D更短。

```
A
├── B
│   └── C
│       └── D 2.0
└── E
    └── D 1.0
```

如果A添加了对D 2.0的依赖，则会使用D 2.0。

```
A
├── B
│   └── C
│       └── D 2.0
├── E
│   └── D 1.0
│
└── D 2.0
```

尽管传递依赖可以隐式包含所需的依赖项，但好的做法是明确指定源代码直接使用的依赖项。

### 6.4 依赖排除
可以使用`<exclusions>`排除引入的传递依赖。例如：

```xml
<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-common</artifactId>
    <version>3.3.1</version>
    <exclusions>
        <exclusion>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

其中groupId和artifactId可以使用通配符`*`。

### 6.5 依赖树分析
可以使用以下命令查看项目的依赖树：

```shell
mvn dependency:tree
```

输出示例：

```
[INFO] com.mycompany.app:my-app:jar:1.0-SNAPSHOT
[INFO] +- junit:junit:jar:4.13.2:test
[INFO] |  \- org.hamcrest:hamcrest-core:jar:1.3:test
[INFO] \- org.apache.commons:commons-lang3:jar:3.12.0:compile
```

## 7.项目继承和项目聚合
Maven的**项目继承**(project inheritance)允许子项目继承父项目的配置，包括依赖、插件、属性等。项目继承可以统一管理多个项目的公共配置，避免重复配置。

Maven还支持**项目聚合**(project aggregation)（也叫做**多模块项目**），允许将一个大型项目拆分为多个子模块，每个子模块可以独立构建，也可以作为一个整体进行构建。多模块项目能够提高代码的复用性和可维护性。

官方文档：
* 项目继承：[Project Inheritance](https://maven.apache.org/guides/introduction/introduction-to-the-pom.html#Project_Inheritance)、[Inheritance](https://maven.apache.org/pom.html#Inheritance)
* 项目聚合：[Project Aggregation](https://maven.apache.org/guides/introduction/introduction-to-the-pom.html#Project_Aggregation)、[Aggregation (or Multi-Module)](https://maven.apache.org/pom.html#Aggregation_.28or_Multi-Module.29)

### 7.1 项目结构
多模块项目的典型目录结构如下所示：

```
my-app/
  pom.xml
  my-module1/
    pom.xml
    src/main/java/
  my-module2/
    pom.xml
    src/main/java/
```

* my-app：父项目
* my-module1、my-module2：子模块，每个模块是一个独立的Maven项目

可以同时使用项目继承和项目聚合。也就是说，可以让子项目指定父项目，同时让父项目将这些子项目指定为**模块**(module)。为此，需要
* 将父POM的打包类型设置为`pom`。
* 在每个子POM中使用`<parent>`指定其父POM。
* 在父POM中使用`<modules>`指定子模块（子POM）。

### 7.2 父项目的POM
父项目的pom.xml文件需要定义以下内容：
* 打包类型必须是`pom`
* 使用`<modules>`声明所有子模块
* 使用`<dependencyManagement>`统一管理子模块的依赖版本
* 使用`<pluginManagement>`统一管理子模块的插件配置

示例：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.mycompany.app</groupId>
    <artifactId>my-app</artifactId>
    <version>1.0</version>
    <packaging>pom</packaging>

    <!-- 声明子模块 -->
    <modules>
        <module>my-module1</module>
        <module>my-module2</module>
    </modules>

    <!-- 定义公共属性 -->
    <properties>
        <java.verison>1.8</java.verison>
        <junit.version>4.13.2</junit.version>
    </properties>

    <!-- 定义公共依赖 -->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>${junit.version}</version>
                <scope>test</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <!-- 定义公共插件 -->
    <build>
        <pluginManagement>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <version>3.13.0</version>
                    <configuration>
                        <source>${java.version}</source>
                        <target>${java.version}</target>
                    </configuration>
                </plugin>
            </plugins>
        </pluginManagement>
    </build>
</project>
```

`<dependencyManagement>`与`<dependencies>`的区别：
* `<dependencies>`中声明的依赖会始终被所有子模块继承。
* `<dependencyManagement>`中声明的依赖仅当子模块也在`<dependencies>`中声明时才会被继承，但子模块只需指定groupId和artifactId，而无需指定版本号。这样做的好处是可以统一管理子模块的依赖版本，而且子模块可以只继承需要的依赖。

注：正如所有Java对象最终继承自`Object`一样，所有POM都继承自[Super POM](https://maven.apache.org/maven-model-builder/super-pom.html)。

### 7.3 子模块的POM
子模块的pom.xml文件需要定义以下内容：
* 使用`<parent>`声明父项目
* 子模块特有的配置（属性、依赖、插件等），可以覆盖父项目的配置

示例：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- 声明父项目 -->
    <parent>
        <groupId>com.mycompany.app</groupId>
        <artifactId>my-app</artifactId>
        <version>1.0</version>
    </parent>

    <artifactId>my-module1</artifactId>

    <dependencies>
        <!-- 从父项目继承的依赖 -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
        </dependency>

        <!-- 子模块特有的依赖 -->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>3.12.0</version>
        </dependency>
    </dependencies>
</project>
```

### 7.4 构建多模块项目
在父项目目录下执行以下命令，可以构建整个多模块项目：

```shell
mvn clean install
```

Maven会按照子模块的依赖关系依次构建每个子模块，并将构建结果安装到本地仓库。

### 7.5 模块之间的依赖
如果子模块之间存在依赖关系，可以在pom.xml中声明依赖。

例如，my-module2依赖my-module1，则在my-module2/pom.xml中声明：

```xml
<dependencies>
    <dependency>
        <groupId>com.mycompany.app</groupId>
        <artifactId>my-module1</artifactId>
        <version>1.0</version>
    </dependency>
</dependencies>
```

注意，子模块之间应该避免循环依赖。

## 8.示例
完整项目示例：<https://github.com/ZZy979/mvnex-examples>

## 参考
* [Maven by Example](https://books.sonatype.com/mvnex-book/reference/index.html)
