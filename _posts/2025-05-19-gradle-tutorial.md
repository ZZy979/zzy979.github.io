---
title: Gradle使用教程
date: 2025-05-19 13:24 +0800
categories: [Build Tools]
tags: [build tools, gradle]
---
## 1.简介
Gradle是一个基于Groovy或Kotlin DSL的构建自动化工具，主要用于Java、Kotlin、Android等项目的构建、依赖管理和任务自动化。
* 官方网站：<https://gradle.org/>
* 官方文档：<https://docs.gradle.org/current/userguide/userguide.html>

## 2.安装
<https://gradle.org/install/>

1. 首先，需要安装JDK 8+。
2. 从[Gradle下载页面](https://gradle.org/releases/)下载二进制包并解压。
3. 将解压目录中的bin目录添加到`PATH`环境变量。
4. 验证安装是否成功：

```shell
$ gradle -v

------------------------------------------------------------
Gradle 8.14
------------------------------------------------------------
```

也可以使用包管理工具安装Gradle。例如：

```shell
brew install gradle
```

注：
* 如果要构建已有的Gradle项目，并且项目使用了Gradle Wrapper，则可以直接使用项目根目录下的gradlew或gradlew.bat，无需安装Gradle。
* 从Gradle 9.0起将需要JDK 17+。

## 3.基本用法
官方入门教程：
* [Beginner Tutorial](https://docs.gradle.org/current/userguide/part1_gradle_init.html)
* [Learning the Basics](https://docs.gradle.org/current/userguide/gradle_basics.html)

### 3.1 创建项目
使用`gradle init`命令可以快速创建一个Gradle项目（参见[Build Init插件](https://docs.gradle.org/current/userguide/build_init_plugin.html)）：

```shell
mkdir my-app
cd my-app
gradle init --type java-application  --dsl groovy --java-version 17
```

对于其他提示（如测试框架）选择默认即可。

生成的项目目录结构如下：

```
my-app/
  app/
    build.gradle
    src/
      main/
        java/
          org/
            example/
              App.java
        resources/
      test/
        java/
          org/
            example/
              AppTest.java
        resources/
  gradle/
    libs.versions.toml
    wrapper/
      gradle-wrapper.jar
      gradle-wrapper.properties
  gradle.properties
  gradlew
  gradlew.bat
  settings.gradle
```

* app：Java应用
  * build.gradle：应用的构建脚本，定义了依赖、插件和任务
  * src：应用的源代码
* gradle/libs.versions.toml：用于集中定义依赖版本
* gradle/wrapper：Gradle Wrapper的JAR和配置文件
* gradlew和gradlew.bat：Linux和Windows的Gradle Wrapper脚本，用于在没有安装Gradle的情况下执行构建
* settings.gradle：项目的设置文件，定义了子项目（模块）

有些项目在根目录中也包含build.gradle文件，但不推荐这样做。

### 3.2 编译项目
在项目根目录中执行以下命令来编译项目：

```shell
gradle build
```

类文件会生成在app/build/classes目录中。

### 3.3 运行测试
执行以下命令来运行项目的单元测试：

```shell
gradle test
```

### 3.4 打包项目
将项目打包成JAR文件：

```shell
gradle jar
```

打包后的JAR文件会生成在app/build/libs目录中。

### 3.5 运行程序
可以通过以下命令运行打包的应用程序：

```shell
$ java -cp app/build/libs/app.jar org.example.App
Hello World!
```

也可以使用[Application插件](https://docs.gradle.org/current/userguide/application_plugin.html)提供的`run`任务：

```shell
gradle run
```

### 3.6 清理项目
删除生成的build目录：

```shell
gradle clean
```

### 3.7 添加依赖
在build.gradle文件的`dependencies`块中添加项目所需的依赖。

例如，添加JUnit依赖：

```groovy
dependencies {
    testImplementation 'junit:junit:4.13.2'
}
```

下次构建时，Gradle会自动从`repositories`指定的仓库下载所需的依赖库。默认情况下只有Maven中央仓库：

```groovy
repositories {
    mavenCentral()
}
```

### 3.8 使用插件
Gradle可以通过插件扩展功能。在build.gradle文件的`plugins`块中指定项目使用的插件。

例如，使用`java`插件来支持Java项目：

```groovy
plugins {
    id 'java'
}
```

`application`插件用于创建可运行的应用程序。

Gradle支持的核心插件列表参见[Gradle Plugin Reference](https://docs.gradle.org/current/userguide/plugin_reference.html)。

### 3.9 自定义任务
在build.gradle文件中可以自定义任务。例如：

```groovy
tasks.register('hello') {
    doLast {
        println 'Hello, Gradle!'
    }
}
```

```shell
$ gradle hello
> Task :app:hello
Hello, Gradle!
```

参见[Implementing Custom Tasks](https://docs.gradle.org/current/userguide/implementing_custom_tasks.html)。

列出所有可用的任务：

```shell
gradle tasks
```

### 3.10 使用Gradle Wrapper
[Gradle Wrapper](https://docs.gradle.org/current/userguide/gradle_wrapper.html)是一个名为gradlew或gradlew.bat的脚本，用于在没有安装Gradle的情况下执行构建，必要时会自动下载指定版本的Gradle。这是推荐的执行Gradle构建的方式。

使用以下命令在项目中生成Gradle Wrapper：

```shell
gradle wrapper
```

之后就可以使用Gradle Wrapper来运行构建，只需执行`./gradlew`而不是`gradle`命令：

```shell
./gradlew build
```

在Windows中需要使用gradlew.bat：

```shell
gradlew.bat build
```

## 4.依赖管理
<https://docs.gradle.org/current/userguide/getting_started_dep_man.html>

依赖管理机制是Gradle的核心功能之一。

### 4.1 依赖声明
在build.gradle文件中，通过`dependencies`块声明项目所需的依赖。依赖的格式为`group:name:version`（类似于Maven坐标）：
* `group`：依赖的组织或公司名称
* `name`：依赖的项目名称
* `version`：依赖的版本号

例如：

```groovy
dependencies {
    implementation 'org.apache.commons:commons-lang3:3.12.0'
    testImplementation 'junit:junit:4.13.2'
}
```

DSL语法参考：[Gradle Build Language Reference](https://docs.gradle.org/current/dsl/index.html)

### 4.2 依赖配置
依赖配置定义了依赖的作用域。常见的配置包括：
* `api`：编译和运行时可用，并且会暴露给其他模块。
* `implementation`：编译和运行时可用，但不会暴露给其他模块。
* `compileOnly`：仅在编译时可用，不会打包到最终产物中。
* `runtimeOnly`：仅在运行时可用，不会参与编译。
* `testImplementation`：编译和运行测试时可用。
* `testCompileOnly`：仅在测试编译时可用。
* `testRuntimeOnly`：仅在测试运行时可用。

### 4.3 传递依赖
Gradle会自动包含传递依赖。即如果项目A依赖项目B，项目B依赖项目C，那么项目A会自动依赖项目C。

当产生依赖版本冲突（即直接或传递依赖同一个库的不同版本）时，默认情况下Gradle会选择**最高**的版本。

### 4.4 依赖排除
可以使用`exclude`排除引入的传递依赖。例如：

```groovy
dependencies {
    implementation('org.apache.hadoop:hadoop-common:3.3.1') {
        exclude group: 'org.slf4j', module: 'slf4j-log4j12'
    }
}
```

### 4.5 依赖树分析
可以使用以下命令查看项目的依赖树：

```shell
gradle :app:dependencies
```

输出示例：

```
compileClasspath - Compile classpath for source set 'main'.
\--- org.apache.commons:commons-lang3:3.12.0

...

testCompileClasspath - Compile classpath for source set 'test'.
+--- org.apache.commons:commons-lang3:3.12.0
\--- junit:junit:4.13.2
     \--- org.hamcrest:hamcrest-core:1.3
```

### 4.6 仓库
在build.gradle文件的`repositories`块指定Gradle下载依赖的仓库：

```groovy
repositories {
    mavenCentral()
}
```

* `mavenCentral()`：[Maven中央仓库](https://repo1.maven.org/maven2/)
* `google()`：[Google Maven仓库](https://maven.google.com/)，用于Android项目
* `mavenLocal()`：本地Maven仓库(~/.m2/repository)
* 自定义仓库：公司或团队内部搭建的仓库，例如

```groovy
repositories {
    maven {
        url = "http://repo.mycompany.com/maven2"
    }
}
```

## 5.多项目构建
<https://docs.gradle.org/current/userguide/multi_project_builds.html>

Gradle支持多项目构建(multi-project build)，允许将一个大型项目拆分为多个子项目，以支持模块化和代码复用。

### 5.1 项目结构
多项目构建由一个根项目以及一个或多个子项目构成。典型的目录结构如下：

```
my-project/
  settings.gradle
  build.gradle
  app/
    build.gradle
  core/
    build.gradle
  util/
    build.gradle
```

* settings.gradle：根项目设置，声明子项目
* build.gradle：根项目构建逻辑（可选）
* app、core、util：子项目，每个都是一个独立的Gradle项目

### 5.2 根项目配置
在根项目的settings.gradle文件中声明所有子项目：

```groovy
rootProject.name = 'my-project'
include 'app', 'core', 'util'
```

在根项目的build.gradle文件中，可以定义所有子项目的公共配置，如插件、依赖、属性等。

例如：

```groovy
allprojects {
    group = 'com.example'
    version = '1.0-SNAPSHOT'

    repositories {
        mavenCentral()
    }
}

subprojects {
    apply plugin: 'java'

    dependencies {
        testImplementation 'junit:junit:4.13.2'
    }
}
```

### 5.3 子项目配置
子项目会自动继承根项目的配置，子项目的build.gradle文件定义子项目特有的配置。

例如core/build.gradle：

```groovy
dependencies {
    implementation 'org.apache.commons:commons-lang3:3.12.0'
}
```

### 5.4 子项目之间的依赖
如果子项目之间存在依赖关系，可以在build.gradle中声明依赖。

例如，app依赖core，则在app/build.gradle中声明：

```groovy
dependencies {
    implementation project(':core')
}
```

子项目之间应该避免循环依赖。

### 5.5 构建多项目
在项目根目录中执行以下命令，可以构建整个多项目：

```shell
gradle build
```

Gradle会按照子项目的依赖关系依次构建每个子项目。

对于大型多项目构建，可以使用`--parallel`选项并行构建子项目：

```shell
gradle build --parallel
```
