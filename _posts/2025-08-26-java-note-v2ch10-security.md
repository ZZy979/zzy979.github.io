---
title: 《Java核心技术》笔记 卷II 第10章 安全
date: 2025-08-26 23:16:43 +0800
categories: [Java, Core Java]
tags: [java, class loader, authentication, jaas, message digest, sha1, encryption, aes, rsa]
---
安全一直是Java的设计者和使用者所关心的一个主要问题。安全机制是Java不可分割的一部分。安全架构由三部分组成：
* 语言和虚拟机设计特性（数组边界检查、无未经检查的类型转换、无指针算术等）。
* **安全管理器**(security manager)：用于控制代码能够执行的操作（如文件访问、网络访问等）。
* 代码签名：代码作者可以使用标准加密算法来认证Java代码，用户能够确定代码作者，以及代码在签名后没有被修改过。

第一部分已经获得了巨大的成功，但是另外两部分并不那么成功。安全管理器很复杂、受攻击面也很大，从Java 17起被弃用了。代码签名架构也随着applet和Java Web Start这两种机制的消亡而被抛弃了。

## 10.1 类加载器
Java编译器将源代码转换为虚拟机指令。这些指令存储在类文件(.class)中。**类加载器**(class loader)是负责加载类的对象。下面几节将介绍虚拟机是如何加载这些类文件的。

### 10.1.1 类加载过程
虚拟机只加载程序执行所需要的类文件。下面是启动程序时虚拟机执行的步骤：
1. 加载主类的类文件。
2. 加载主类的字段或超类的类文件（如果有）。加载一个类依赖的所有类的过程称为类的**解析**(resolving)。
3. 执行主类的`main()`方法。
4. 加载`main()`方法或其调用的方法用到的类。

每个Java程序至少有三个类加载器：
* **引导类加载器**(bootstrap class loader)
* **平台类加载器**(platform class loader)
* **系统类加载器**(system class loader)（有时也称为应用类加载器）

引导类加载器负责加载`java.base`等JDK模块中的平台类。引导类加载器没有对应的`ClassLoader`对象。例如，`StringBuilder.class.getClassLoader()`返回`null`。

在Java 9之前，Java平台类位于文件rt.jar中。如今，Java平台是模块化的，每个平台模块包含在一个JMOD文件中（参见第9章）。平台类加载器会加载引导类加载器没有加载的所有Java平台类（如`java.sql`模块）。

系统类加载器会从类路径和模块路径加载应用程序类。

注释：在Java 9之前，扩展类加载器(extension class loader)会从jre/lib/ext目录加载“标准扩展”，而“[认可标准覆盖](https://docs.oracle.com/javase/8/docs/technotes/guides/standards/)”机制提供了一种用较新版本覆盖某些平台类（包括CORBA和XML实现）的方式。这两种机制都已被移除。

### 10.1.2 类加载器的层次结构
类加载器具有父/子关系。除了引导类加载器，每个类加载器都有一个父加载器。类加载器会首先通过父加载器加载给定的类，如果失败则自己加载。例如，当使用系统加载器加载`StringBuilder`类时，它首先询问平台类加载器，该加载器又询问引导类加载器。引导类加载器会找到并加载这个类，因此无须另外两个类加载器做更多搜索。

有些程序具有插件架构，其中代码的某些部分是作为可选插件打包的。如果插件被打包为JAR文件，可以直接用`URLClassLoader`加载插件类。

```java
var url = new URL("file:///path/to/plugin.jar");
var pluginLoader = new URLClassLoader(new URL[] {url});
Class<?> cl = pluginLoader.loadClass("mypackage.MyClass");
```

由于在`URLClassLoader`构造器中没有指定父加载器，因此其父加载器就是系统类加载器。下图展示了其层次结构。

![类加载器层次结构](/assets/images/java-note-v2ch10-security/类加载器层次结构.png)

警告：在Java 9之前，系统类加载器是`URLClassLoader`类的实例，但现在不再是了。

大多数时候，你不必操心类加载器的层次结构。但是，偶尔也会需要指定类加载器。例如：你的应用代码包含一个辅助方法，它调用了`Class.forName(className)`；该方法是从一个插件类中调用的；`className`指定的是插件JAR中的类。辅助方法的类是由系统类加载器加载的，这也是`Class.forName()`所使用的类加载器，而插件JAR中的类对其不可见。这种现象称为**类加载器反转**(classloader inversion)。

要解决这个问题，辅助方法需要使用正确的类加载器：可以要求将类加载器作为参数传递，或者要求将正确的类加载器设置为当前线程的上下文类加载器。许多框架都使用这种策略（例如JAXP和JNDI）。

每个线程都有一个类加载器的引用，称为**上下文类加载器**(context class loader)。主线程的上下文类加载器是系统类加载器。当新线程创建时，其上下文类加载器默认设置为父线程的上下文类加载器，也可以通过调用`Thread.setContextClassLoader()`方法设置。

辅助方法可以获取上下文类加载器：

```java
Thread t = Thread.currentThread();
ClassLoader loader = t.getContextClassLoader();
Class<?> cl = loader.loadClass(className);
```

提示：如果你编写了一个按名字来加载类的方法，让调用者在显式传递类加载器和使用上下文类加载器之间进行选择是一种好的做法。不要直接使用方法所属类的类加载器。

### 10.1.3 将类加载器用作命名空间
令人惊讶的是，在同一个虚拟机中可以有两个类名和包名都相同的类。实际上，类是由全名**和类加载器**确定的。这项技术在加载来自多个来源的代码时很有用。例如，应用服务器对于每个应用使用单独的类加载器（如下图所示）。

![两个类加载器加载两个同名的类](/assets/images/java-note-v2ch10-security/两个类加载器加载两个同名的类.png)

### 10.1.4 编写自己的类加载器
你可以编写自己的用于特殊目的的类加载器，这使你可以在将字节码传递给虚拟机之前执行自定义检查。

要编写自己的类加载器，只需扩展`ClassLoader`类，并覆盖`findClass()`方法。`loadClass()`方法负责将类的加载委托给父加载器，只有当该类尚未加载且父加载器无法加载该类时，才会调用`findClass()`方法。

该方法的实现必须做到以下几点：
1. 从本地文件系统或其他来源加载类的字节码。
2. 调用`ClassLoader`超类的`defineClass()`方法将字节码提供给虚拟机。

在程序清单10-1中实现了一个类加载器，用于加载加密的类文件。该程序要求用户输入主类的名字和密钥，然后使用一个特殊的类加载器来解密指定的类，最后调用其`main()`方法（如下图所示）。

![ClassLoaderTest程序](/assets/images/java-note-v2ch10-security/ClassLoaderTest程序.png)

![ClassLoaderTest程序2](/assets/images/java-note-v2ch10-security/ClassLoaderTest程序2.png)

为了简单起见，这里采用了古老的Caesar算法对类文件进行加密。我们的Caesar算法使用的密钥是1~255之间的数字。加密时，只需将密钥与每个字节相加，然后对256取模。程序清单10-2中的`Caesar`程序用于进行加密。

为了避免混淆，加密的类文件使用不同的扩展名.caesar。解密时，类加载器只需将每个字节减去密钥。在代码中有四个类文件，都是使用3这个密钥值加密的。需要使用`ClassLoaderTest`程序中的自定义类加载器进行解密。

[程序清单10-1 classLoader/ClassLoaderTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch10/classLoader/ClassLoaderTest.java)

[程序清单10-2 classLoader/Caesar.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch10/classLoader/Caesar.java)

加密类文件有许多实际用途（当然前提是使用更强的加密算法）。没有解密密钥，类文件就毫无用处——既不能由标准虚拟机执行，也不能轻易被反汇编。这意味着可以使用自定义类加载器来认证用户，或者确保程序在运行前已付费。加密只是自定义类加载器的用途之一，例如还可以将类文件存储到数据库。

### 10.1.5 字节码校验
当类加载器将新加载的类的字节码提供给虚拟机时，这些字节码首先由**校验器**(verifier)检查，以确保指令不会执行明显有破坏性的操作。除了系统类外，所有的类都要校验。

下面是校验器执行的一些检查：
* 变量在使用前已初始化
* 方法调用与对象类型匹配
* 没有违反访问私有数据和方法的规则
* 对局部变量的访问落在运行时栈内
* 运行时栈没有溢出

如果这些检查中任何一项失败，就认为该类已损坏，且不会加载。

Java编译器生成的类文件总是可以通过校验。然而，类文件使用的字节码格式是有详细文档的，对于具有汇编编程经验和十六进制编辑器的人来说，手动地创建一个包含合法但不安全的指令的类文件是很容易的事情。校验器是为了防范恶意篡改的类文件，而不是检查类文件是否由编译器产生。

下面的例子展示了如何创建一个修改过的类文件。程序清单10-3是一个简单的程序，调用`fun()`方法计算1+2并显示结果。

```java
public static int fun() {
    int m;
    int n;
    m = 1;
    n = 2;
    int r = m + n;
    return r;
}
```

为了创建一个不良的类文件，首先运行`javap`程序查看`fun()`方法的指令。命令

```shell
javap -c verifier.VerifierTest
```

会以助记符(mnemonic)格式显示类文件中的字节码：

```
public static int fun();
  Code:
     0: iconst_1
     1: istore_0
     2: iconst_2
     3: istore_1
     4: iload_0
     5: iload_1
     6: iadd
     7: istore_2
     8: iload_2
     9: ireturn
```

使用一个十六进制编辑器将指令3从`istore_1`改为`istore_0`。也就是说，局部变量0（即`m`）被初始化了两次，而局部变量1（即`n`）根本没有初始化。为此需要知道这些指令的十六进制值，这可以从[Java虚拟机规范 6.5节](https://docs.oracle.com/javase/specs/jvms/se17/html/jvms-6.html#jvms-6.5)找到。

```
0: iconst_1 04
1: istore_0 3B
2: iconst_2 05
3: istore_1 3C
4: iload_0  1A
5: iload_1  1B
6: iadd     60
7: istore_2 3D
8: iload_2  1C
9: ireturn  AC
```

下图展示了使用十六进制编辑器（IntelliJ IDEA插件[BinEd](https://plugins.jetbrains.com/plugin/9339-bined--binary-hex-editor)）打开类文件VerifierTest.class，其中`fun()`方法的字节码高亮显示。

![使用十六进制编辑器修改字节码](/assets/images/java-note-v2ch10-security/使用十六进制编辑器修改字节码.png)

将其中的`3C`改为`3B`并保存类文件。然后尝试运行`VerifierTest`程序，将会看到错误消息：

```
Error: Unable to initialize main class verifier.VerifierTest
Caused by: java.lang.VerifyError: Bad local variable type
Exception Details:
  Location:
    verifier/VerifierTest.fun()I @5: iload_1
  Reason:
    Type top (current frame, locals[1]) is not assignable to integer
  Current Frame:
    bci: @5
    flags: { }
    locals: { integer }
    stack: { integer }
  Bytecode:
    0000000: 043b 053b 1a1b 603d 1cac
```

虚拟机检测到了我们的修改。现在用`-noverify`（或`-Xverify:none`）选项来运行程序：

```shell
$ java -noverify verifier.VerifierTest
1 + 2 == 15102330
```

`fun()`方法返回了一个看似随机的值。实际上，这是2加上碰巧存储在变量`n`中的值（未初始化）得到的结果。

## 10.2 用户认证
Java API提供了一个叫做Java认证和授权服务(Java Authentication and
Authorization Service, JAAS)的框架，它提供了对平台提供和自定义认证机制的访问。

### 10.2.1 JAAS框架
JAAS框架包含两部分：“认证”部分涉及确定用户的身份，“授权”部分与已弃用的安全管理器相关，这里不再讨论。

JAAS是一个可插拔的API，将Java应用与实现认证的特定技术分离开。它支持UNIX登录、Windows登录、Kerberos认证和基于证书的认证等。

下面是登录代码的基本框架：

```java
try {
    System.setSecurityManager(new SecurityManager());
    var context = new LoginContext("Login1"); // defined in JAAS configuration file
    context.login();
    // get the authenticated Subject
    Subject subject = context.getSubject();
    ...
    context.logout();
}
catch (LoginException e) { // thrown if login was not successful
    e.printStackTrace();
}
```

其中`subject`是指已被认证的个体。

`LoginContext`构造器的字符串参数`"Login1"`引用了JAAS配置文件中同名的条目。下面是一个示例配置文件：

```
Login1 {
  com.sun.security.auth.module.UnixLoginModule required;
  com.whizzbang.auth.module.RetinaScanModule sufficient;
};

Login2 {
  ...
};
```

JDK在`com.sun.security.auth.module`包中提供了以下登录模块：`UnixLoginModule`, `NTLoginModule`, `Krb5LoginModule`, `JndiLoginModule`, `KeyStoreLoginModule`。

登录策略由一系列登录模块组成，每个模块被标记为`required`, `sufficient`, `requisite`或`optional`。这些关键字的含义由以下算法描述：
1. 依次执行各模块，直到某个`sufficient`模块成功，某个`requisite`模块失败，或者到达列表末尾。
2. 如果所有`required`和`requisite`模块都成功，或者没有执行过这两类模块，但至少有一个`sufficient`或`optional`模块成功，则认证成功。

登录对**主体**(`Subject`)进行认证，主体可以有多个**特征**(`Principal`)。特征描述了主体的某些属性，如用户名、组ID或角色。`UnixPrincipal`描述了UNIX登录名，`UnixNumericGroupPrincipal`可以检测用户是否属于某个UNIX用户组。

程序清单10-4中的程序展示了当前已登录用户的身份。像这样运行该程序：

```shell
java -Djava.security.auth.login.config=auth/jaas.config auth.AuthTest
```

程序清单10-5展示了登录配置。在Windows上，需要将jaas.config中的`UnixLoginModule`改为`NTLoginModule`。

[程序清单10-4 auth/AuthTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch10/auth/AuthTest.java)

[程序清单10-5 auth/jaas.config](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch10/auth/jaas.config)

### 10.2.2 JAAS登录模块
本节将通过一个JAAS示例介绍如何实现自己的登录模块，以及如何实现基于角色的认证。

如果将登录信息存储在数据库中，那么提供自己的登录模块就很有用。

登录模块的工作之一是填充被认证主体的特征集。如果登录模块支持角色，就添加描述角色的`Principal`对象。Java库并没有提供这样的类，所以我们写了自己的类（见程序清单10-6）。该类仅仅存储了一个描述/值对，例如`role=admin`。

我们的登录模块会在包含如下行的文本文件中查找用户、密码和角色（在实际的登录模块中，可能会将这些信息存储在数据库中）：

```
harry|secret|admin
carl|guessme|HR
```

程序清单10-7是`SimpleLoginModule`的代码。`checkLogin()`方法检查输入的用户名和密码是否匹配上述文件中的记录。如果匹配，则添加两个`SimplePrincipal`对象到主体的特征集中：

```java
Set<Principal> principals = subject.getPrincipals();
principals.add(new SimplePrincipal("username", username));
principals.add(new SimplePrincipal("role", role));
```

`initialize()`方法接收以下参数：
* 被认证的`Subject`
* 获取登录信息的handler
* `sharedState`映射，可用于登录模块之间的通信
* `options`映射，包含登录配置中设置的键值对

例如，如下配置登录模块：

```java
SimpleLoginModule required pwfile="jaas/password.txt";
```

则`options`映射包含`pwfile`设置。

登录模块不收集用户名和密码，这是handler的工作。这种分离允许你使用相同的登录模块，而不用关心登录信息是来自GUI对话框、控制台输入还是配置文件。

handler是在构造`LoginContext`时指定的，例如：

```java
var context = new LoginContext("Login1", new DialogCallbackHandler());
```

`DialogCallbackHandler`会弹出一个简单的GUI对话框来获取用户名和密码，而`TextCallbackHandler`从控制台获取这些信息。

但是，在示例程序中是通过自己编写的GUI来获取用户名和密码（如下图所示）。

![自定义登录模块](/assets/images/java-note-v2ch10-security/自定义登录模块.png)

我们创建了一个简单的handler，仅仅存储并返回这些信息（见程序清单10-8）。该handler只有一个`handle()`方法，用于处理`Callback`对象数组。

```java
public void handle(Callback[] callbacks) {
    for (Callback callback : callbacks) {
        if (callback instanceof NameCallback) ...
        else if (callback instanceof PasswordCallback) ...
        else ...
    }
}
```

登录模块提供认证需要的callback数组：

```java
var nameCall = new NameCallback("username: ");
var passCall = new PasswordCallback("password: ", false);
callbackHandler.handle(new Callback[] { nameCall, passCall });
```

程序清单10-9中的程序显示一个窗体，用于输入登录信息。如果登录成功，就显示所有principal。像这样运行该程序：

```shell
java -Djava.security.auth.login.config=jaas/jaas.config jaas.JAASTest
```

[程序清单10-6 jaas/SimplePrincipal.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch10/jaas/SimplePrincipal.java)

[程序清单10-7 jaas/SimpleLoginModule.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch10/jaas/SimpleLoginModule.java)

[程序清单10-8 jaas/SimpleCallbackHandler.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch10/jaas/SimpleCallbackHandler.java)

[程序清单10-9 jaas/JAASTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch10/jaas/JAASTest.java)

[程序清单10-10 jaas/jaas.config](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch10/jaas/jaas.config)

注释：可以支持更复杂的两阶段协议：只有登录配置中的所有模块都认证成功，登录才会被**提交**。详见登录模块开发指南： <https://docs.oracle.com/javase/8/docs/technotes/guides/security/jaas/JAASLMDevGuide.html> 。

## 10.3 数字签名
在过去的50年里，数学家和计算机学家已经开发出了复杂的算法，用于确保数据的完整性和创建电子签名。`java.security`包包含许多这类算法的实现。在下面几节中，将介绍消息摘要是如何检测数据文件中的更改，以及数字签名是如何证明签名者的身份的。

### 10.3.1 消息摘要
**消息摘要**(message digest)是数据块的数字指纹。例如，SHA-1 (Secure Hash Algorithm #1)可以将任意长度的数据块压缩为160位（20字节）的序列。人们希望任何两条不同的消息都不会有相同的SHA-1指纹。当然，这是不可能是，因为只有2<sup>160</sup>个SHA-1指纹。但是2<sup>160</sup>太大了，碰撞的概率微乎其微。

消息摘要有两个基本属性：
* 如果数据的1位或几位发生改变，那么消息摘要也（几乎）一定会改变。
* （几乎）不可能构造出与原消息具有相同签名的假消息。

人们已经设计出大量用于计算消息摘要的算法，其中最著名的是SHA-1和MD5。Java支持SHA-2和SHA-3算法集。`MessageDigest`类是用于创建指纹算法对象的工厂，其静态方法`getInstance()`返回一个扩展了`MessageDigest`的类的对象。

例如，可以这样获取一个计算SHA-1指纹的对象：

```java
MessageDigest alg = MessageDigest.getInstance("SHA-1");
```

之后，反复调用`update()`方法提供消息的所有字节：

```java
InputStream in = ...;
int ch;
while ((ch = in.read()) != -1)
    alg.update((byte) ch);
```

或者也可以一次提供整个字节数组：

```java
byte[] bytes = ...;
alg.update(bytes);
```

最后调用`digest()`方法执行计算，并以字节数组的形式返回消息摘要：

```java
byte[] hash = alg.digest();
```

程序清单10-11中的程序计算了一个消息摘要，可以在命令行指定文件和算法：

```shell
java hash.Digest hash/input.txt SHA-1
```

如果没有提供命令行参数，则提示输入文件和算法名。

[程序清单10-11 hash/Digest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch10/hash/Digest.java)

### 10.3.2 消息签名
如果消息及其指纹是分开传送的，接收者就可以检查消息是否被篡改过。但是，如果消息和指纹都被截获，那么修改消息然后重新计算指纹就是一件很容易的事。毕竟，消息摘要算法是公开的，不需要密钥。在这种情况下，接收者永远不会知道消息已被篡改。**数字签名**(digital signature)解决了这个问题。

为了理解数字签名的工作原理，需要解释几个来自**公钥密码学**(public key cryptography)（也称为非对称加密）领域的几个概念。公钥密码学基于**公钥**(public key)和**私钥**(private key)这两个概念。其思想是：你可以将公钥告诉任何人，但是只有自己持有私钥，不要泄露给任何人。两个密钥通过数学关系进行匹配。在现实中，几乎不可能用一个密钥推算出另一个。

假设Alice给Bob发送一条消息，Bob想知道该消息是否来自Alice而不是冒名顶替者。Alice编写消息，并用她的私钥对消息摘要进行**签名**(sign)。Bob获得了她的公钥，然后用公钥对签名进行**验证**(verify)。如果验证通过，Bob就可以确信两个事实：
* 原消息没有被篡改过。
* 该消息是由Alice（与Bob用于校验的公钥相匹配的私钥的持有者）签名的。

可以看到私钥的安全性非常重要。如果有人偷了Alice的私钥，小偷就可以假扮她来发送消息。

![公钥签名交换](/assets/images/java-note-v2ch10-security/公钥签名交换.png)

### 10.3.3 校验签名
JDK自带了`keytool`程序（在$jdk/bin目录中），是用于生成和管理证书(certificate)的命令行工具。该程序负责管理**密钥库**(keystore)——证书和私钥/公钥对的数据库。密钥库中的每一项都有一个**别名**(alias)。Alice可以像这样创建一个密钥库alice.certs，并生成一个别名为alice的密钥对：

```shell
keytool -genkeypair -keystore alice.certs -keyalg dsa -alias alice
```

当新建或打开密钥库时，系统会提示输入口令。当生成密钥时，系统会提示输入以下信息：

```
Enter keystore password: secret
Re-enter new password: secret
What is your first and last name?
  [Unknown]:  Alice Lee
What is the name of your organizational unit?
  [Unknown]:  Engineering
What is the name of your organization?
  [Unknown]:  ACME Software
What is the name of your City or Locality?
  [Unknown]:  San Francisco
What is the name of your State or Province?
  [Unknown]:  CA
What is the two-letter country code for this unit?
  [Unknown]:  US
Is CN=Alice Lee, OU=Engineering, O=ACME Software, L=San Francisco, ST=CA, C=US correct?
  [no]:  yes

Generating 2,048 bit RSA key pair and self-signed certificate (SHA256withRSA) with a validity of 90 days
        for: CN=Alice Lee, OU=Engineering, O=ACME Software, L=San Francisco, ST=CA, C=US
```

假设Alice想把她的公钥提供给Bob，她需要导出一个证书文件：

```shell
keytool -exportcert -keystore alice.certs -alias alice -file alice.cer
```

现在，Alice可以将证书发送给Bob。当Bob收到证书时，他可以将其打印出来：

```shell
keytool -printcert -file alice.cer
```

打印结果如下：

```
Owner: CN=Alice Lee, OU=Engineering, O=ACME Software, L=San Francisco, ST=CA, C=US
Issuer: CN=Alice Lee, OU=Engineering, O=ACME Software, L=San Francisco, ST=CA, C=US
Serial number: 470835ce
Valid from: Sat Oct 06 18:26:38 PDT 2007 until: Fri Jan 04 17:26:38 PST 2008
Certificate fingerprints:
         MD5: BC:18:15:27:85:69:48:B1:5A:C3:0B:1C:C6:11:B7:81
         SHA1: 31:0A:A0:B8:C2:8B:3B:B6:85:7C:EF:C0:57:E5:94:95:61:47:6D:34
Signature algorithm name: SHA1withDSA
Version: 3
```

Bob可以向Alice验证证书的指纹。

注释：有些证书发行者将证书指纹公布在他们的网站上。例如，要检查密钥库$jdk/lib/security/cacerts中的DigiCert公司的证书，可以使用`-list`选项（将$jdk替换为JDK安装目录）：

```shell
keytool -list -v -keystore $jdk/lib/security/cacerts
```

该密钥库的口令是changeit。其中一个证书是：

```
Owner: CN=DigiCert Assured ID Root G3, OU=www.digicert.com, O=DigiCert Inc, C=US
Issuer: CN=DigiCert Assured ID Root G3, OU=www.digicert.com, O=DigiCert Inc, C=US
Serial number: ba15afa1ddfa0b54944afcd24a06cec
Valid from: Thu Aug 01 14:00:00 CEST 2013 until: Fri Jan 15 13:00:00 CET 2038
Certificate fingerprints:
         SHA1: F5:17:A2:4F:9A:48:C6:C9:F8:A2:00:26:9F:DC:0F:48:2C:AB:30:89
         SHA256: 7E:37:CB:8B:4C:47:09:0C:AB:36:55:1B:A6:F4:5D:B8:40:68:0F:BA:16:6A:95:2D:B1:00:71:7F:43:05:3F:C2
```

可以通过访问网站 <https://knowledge.digicert.com/general-information/digicert-trusted-root-authority-certificates> 来核实证书的有效性。

一旦Bob信任该证书，就可以将其导入他的密钥库中：

```shell
keytool -importcert -keystore bob.certs -alias alice -file alice.cer
```

警告：绝对不要将不完全信任的证书导入密钥库。一旦添加，任何使用密钥库的程序都会认为该证书可以用来验证签名。

现在Alice就可以给Bob发送签过名的文档了。`jarsigner`工具用于签名和验证JAR文件。Alice只需将待签名的文档添加到JAR文件中：

```shell
jar cvf document.jar document.txt
```

然后将签名添加到JAR文件，指定要使用的密钥库、JAR文件和密钥的别名：

```shell
jarsigner -keystore alice.certs document.jar alice
```

当Bob收到JAR文件时，可以使用`-verify`选项进行验证：

```shell
jarsigner -verify -keystore bob.certs document.jar
```

Bob不需要指定密钥别名。`jarsigner`程序会在密钥库中查找与签名匹配的证书。

如果JAR文件没有损坏且签名匹配，`jarsigner`程序就会打印 "jar verified" 。否则，程序将显示错误消息。

### 10.3.4 认证问题
任何人都可以生成一对公钥和私钥，用私钥对消息签名，然后把签过名的消息和公钥发送给你。你仍然不知道消息是谁写的。这种确定发送者身份的问题称为**认证问题**(authentication problem)。

注：认证问题的本质在于**如何确定发送者的公钥是可信的**。只要确定了公钥是可信的，并且消息能用该公钥验证通过，就能确定消息是可信的。

解决认证问题的通常做法很简单。假设陌生人和你有一个你们都信任的共同的熟人（可信中间人），陌生人将包含公钥的磁盘交给熟人，然后熟人将磁盘交给你，这样就能确定陌生人的身份（即公钥）是可信的（见下图）。

![通过可信中间人进行认证](/assets/images/java-note-v2ch10-security/通过可信中间人进行认证.png)

事实上，熟人可以使用他的私钥对陌生人的公钥文件进行签名（见下图）。当你得到公钥文件时，可以验证熟人的签名。由于你信任他，因此你相信他在添加签名之前确实核实了陌生人的身份（即陌生人的公钥是可信的）。

![通过可信中间人的签名进行认证](/assets/images/java-note-v2ch10-security/通过可信中间人的签名进行认证.png)

然而，你们之间可能没有共同的熟人。信任模型通常假设有一家我们都信任的公司，例如DigiCert、GlobalSign和Entrust等公司提供验证服务。

### 10.3.5 证书签名
在10.3.3节中，已经看到了Alice如何使用自签名的证书向Bob分发公钥。但是，Bob需要通过验证Alice的指纹以确保这个证书是有效的。

假设Alice想要给Cindy发送一条签过名的消息，但是Cindy不想为验证大量的签名指纹而费心。假设Cindy信任ACME软件公司的信息资源部，该部门运营着**证书颁发机构**(certificate authority, CA)。ACME的每个人在其密钥库中都有CA的公钥，这是由负责详细核查密钥指纹的系统管理员安装的。CA对ACME员工的密钥进行签名。当他们安装彼此的密钥时，密钥库将隐式地信任这些密钥，因为它们是由可信密钥签名的。

下面展示了如何模仿这个过程。首先创建一个密钥库acmesoft.certs，生成一个密钥对并导出公钥：

```shell
keytool -genkeypair -keystore acmesoft.certs -alias acmeroot
keytool -exportcert -keystore acmesoft.certs -alias acmeroot -file acmeroot.cer
```

公钥被导出到一个自签名的证书acmeroot.cer。然后将其添加到每个员工的密钥库中：

```shell
keytool -importcert -keystore cindy.certs -alias acmeroot -file acmeroot.cer
```

如果Alice要发送消息给Cindy（或者ACME公司的其他任何人），她需要将自己的证书提交给信息资源部并签名。遗憾的是，`keytool`程序并没有这个功能。因此，本书代码提供了一个[CertificateSigner](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch10/CertificateSigner.java)类来弥补这一空白（注：来自[第8版书中代码](https://horstmann.com/corejava/corejava8.zip)，在第9版中被删除）。CA管理员将验证Alice的身份，并像这样生成签名的证书：

```shell
java CertificateSigner -keystore acmesoft.certs -alias acmeroot \
    -infile alice.cer -outfile alice_signedby_acmeroot.cer
```

`CertificateSigner`程序必须拥有ACME公司密钥库的访问权限，并且CA管理员必须知道密钥库的口令。显然这是一项敏感操作。

Alice将文件alice_signedby_acmeroot.cer交给Cindy（或者ACME公司的其他任何人）。该文件包含Alice的公钥和ACME公司的声明，证明该密钥确实属于Alice。

现在，Cindy可以将签名的证书导入到她的密钥库中：

```shell
keytool -importcert -keystore cindy.certs -alias alice -file alice_signedby_acmeroot.cer
```

密钥库会验证该密钥是由密钥库中已有的可信根密钥签名过的，Cindy就不必验证证书指纹了。一旦Cindy添加了根证书和经常给她发送文档的人的证书后，她就再也不用担心密钥库了。

### 10.3.6 证书请求
在前一节中，我们用密钥库和证书签名工具模拟了一个CA。但是，大多数CA都运行更加复杂的软件来管理证书，并且使用的证书格式也略有不同。本节将展示与这些软件包进行交互需要增加的步骤。

我们以OpenSSL软件包为例。许多Linux和macOS系统都预装了这个软件，对于Windows系统可以在 <https://www.openssl.org/> 下载。

为了创建一个CA，需要运行`CA`脚本。其确切位置取决于操作系统。在Ubuntu上，运行

```shell
/usr/lib/ssl/misc/CA.pl -newca
```

这个脚本会在当前目录中创建一个子目录demoCA，该目录包含一个根密钥对，并存储了证书和证书吊销列表。

你希望将这个公钥导入所有员工的Java密钥库中，但是它的格式是隐私增强型邮件(Privacy Enhanced Mail, PEM)，而不是密钥库容易接受的DER格式。将文件demoCA/cacert.pem复制成acmeroot.pem。在文本编辑器中打开这个文件，并删除`-----BEGIN CERTIFICATE-----`行之前以及`-----END CERTIFICATE-----`行之后的所有内容。

现在可以按照通常的方式将acmeroot.pem导入到密钥库中：

```shell
keytool -importcert -keystore cindy.certs -alias acmeroot -file acmeroot.pem
```

（`keytool`居然不能自动执行这个编辑操作）

要对Alice的公钥签名，首先生成一个**证书请求**(certificate request)，它包含PEM格式的证书：

```shell
keytool -certreq -keystore alice.store -alias alice -file alice.pem
```

要对证书签名，运行

```shell
openssl ca -in alice.pem -out alice_signedby_acmeroot.pem
```

与前面一样，将alice_signedby_acmeroot.pem中证书开始/结束标记之外的内容删除。然后将其导入到密钥库中：

```shell
keytool -importcert -keystore cindy.certs -alias alice -file alice_signedby_acmeroot.pem
```

可以使用相同的步骤用CA提供的密钥对证书进行签名。

### 10.3.7 代码签名
认证技术的一个常见用途是对可执行程序进行签名。如果从网上下载一个程序，你自然会担心该程序可能带来的危害（例如感染了病毒）。如果知道代码从何而来，并且没有被篡改该过，那么放心程度就会高得多。

假设你在上网时遇到一个网站，该网站提示运行来自不明提供商的applet（能在HTML页面上运行的Java小程序），前提是你为它授予权限（如下图所示）。这样的程序是由Java运行时环境信任的证书颁发机构颁发的“软件开发者”证书进行签名的。弹出的对话框显示了软件开发者和证书颁发者。现在你需要决定是否对该程序授权。

![启动签过名的applet.png](/assets/images/java-note-v2ch10-security/启动签过名的applet.png)

假设你已经了解以下情况：
1. Thawte公司将一个证书卖给了软件开发者。
2. 程序确实是用该证书签名的，并且在传输过程中没有被篡改。
3. 该证书确实是由Thawte签名的。

当然，这些信息都不能告诉你代码是否可以安全运行。如果你只知道供应商的名字以及Thawte公司卖给了他们一个软件开发者证书，你就能信任该供应商吗？这种方式显然没什么意义。

对于内联网部署，证书更可信。管理员可以在本地机器上安装策略文件和证书，这样在运行可信代码时无需任何用户交互。但是，随着安全管理器的弃用，这种方式已经不再可行了。

## 10.4 加密
除了认证，安全性的第二个重要方面是**加密**(encryption)。认证仅仅确保信息没有被篡改，信息本身是明文可见的。相比之下，信息被加密后是不可见的，只能用匹配的密钥进行解密。认证对于代码签名已经足够了。但是，当传输机密信息（例如信用卡号和其他个人数据）时，就有必要加密了。

Java提供了出色的加密支持，已经成为标准库的一部分。

### 10.4.1 对称加密
Java加密扩展（`javax.crypto`包）包含一个`Cipher`类，它是所有加密算法的超类。通过调用静态方法`getInstance()`获得一个`Cipher`对象：

```java
Cipher cipher = Cipher.getInstance(algorithmName);
```

算法名称是一个字符串，例如`"AES"`或`"DES/CBC/PKCS5Padding"`，完整列表参见`Cipher`类的API文档和[Java安全标准算法名称规范](https://docs.oracle.com/en/java/javase/17/docs/specs/security/standard-names.html#cipher-algorithm-names)。可以通过第二个参数指定[提供者](https://docs.oracle.com/en/java/javase/17/security/oracle-providers.html)，默认为`"SunJCE"`。

数据加密标准(Data Encryption Standard, DES)是一种密钥长度为56位的古老的分组加密(block cipher)，现在已经过时，因为可以用暴力法破解。更好的选择是其后续版本——高级加密标准(Advanced Encryption Standard, AES)。AES算法的详细描述参见 <https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.197.pdf> 。

注：加密和解密使用同一个密钥的算法称为**对称加密**(symmetric cipher)，如DES、AES等。使用公钥加密、私钥解密的算法称为非对称加密，如RSA、DSA等。

一旦获得了一个`Cipher`对象，就可以通过设置模式和密钥来对其初始化：

```java
int mode = ...;
Key key = ...;
cipher.init(mode, key);
```

模式是`ENCRYPT_MODE`, `DECRYPT_MODE`, `WRAP_MODE`, `UNWRAP_MODE`之一（定义在`Cipher`类中）。wrap和unwrap模式用一个密钥对另一个密钥进行加密，例子参见10.4.4节。

现在可以反复调用`update()`方法来加密数据块：

```java
int blockSize = cipher.getBlockSize();
var inBytes = new byte[blockSize];
// read inBytes
int outputSize = cipher.getOutputSize(blockSize);
var outBytes = new byte[outputSize];
int outLength = cipher.update(inBytes, 0, outputSize, outBytes);
// write outBytes
```

完成后，必须调用一次`doFinal()`方法。如果还有最后一个输入数据块（小于`blockSize`字节），则调用

```java
outBytes = cipher.doFinal(inBytes, 0, inLength);
```

如果所有输入数据都已加密，则调用

```java
outBytes = cipher.doFinal();
```

调用`doFinal()`对于最后一块进行填充(padding)是必需的（补齐到`blockSize`字节）。一种常用的填充方案是RSA Security公司的公钥加密标准(Public Key Cryptography Standard, PKCS) #5 (<https://www.ietf.org/rfc/rfc2898.txt>)。在该方案中，最后一个数据块不是用0进行填充，而是用等于填充字节数的值进行填充。换句话说，如果块大小为8字节，`L`是最后一个（不完整的）数据块，则按以下方式进行填充：

```
L0 L1 L2 L3 L4 L5 L6 01    if length(L) = 7
L0 L1 L2 L3 L4 L5 02 02    if length(L) = 6
L0 L1 L2 L3 L4 03 03 03    if length(L) = 5
...
L0 07 07 07 07 07 07 07    if length(L) = 1
```

如果输入数据的长度能被8整除（即没有不完整的块），则将下面的块附加到输入数据后面，然后进行加密。

```
08 08 08 08 08 08 08 08
```

在解密后，明文的最后一个字节就是要丢弃的填充字符数量。

### 10.4.2 密钥生成
为了加密，需要生成一个密钥(key)。每种加密算法有不同的密钥格式，你需要确保密钥是随机生成的。遵循以下步骤：
1. 为加密算法获取`KeyGenerator`。
2. 初始化生成器。如果算法的块长度是可变的，还需要指定期望的块长度。
3. 调用`generateKey()`方法。

例如，可以如下生成一个AES密钥：

```java
KeyGenerator keygen = KeyGenerator.getInstance("AES");
var random = new SecureRandom(); // see below
keygen.init(random);
Key key = keygen.generateKey();
```

或者，可以从固定的原始数据生成密钥：

```java
byte[] keyData = ...; // 16 bytes for AES
var key = new SecretKeySpec(keyData, "AES");
```

生成密钥时，确保使用**真随机数**。常规的`Random`类不够随机，`SecureRandom`类生成的随机数远比`Random`类生成的更安全。仍然需要提供一个种子：

```java
var random = new SecureRandom();
var b = new byte[20];
// fill with truly random bits
random.setSeed(b);
```

本节的示例程序将应用AES加密（程序清单10-12）。为了使用该程序，首先需要生成一个密钥：

```shell
java aes.AESTest -genkey secret.key
```

密钥保存在secret.key文件中。

然后使用以下命令进行加密：

```shell
java aes.AESTest -encrypt plaintextFile encryptedFile secret.key
```

使用以下命令进行解密：

```shell
java aes.AESTest -decrypt encryptedFile decryptedFile secret.key
```

该程序很简单。`-genkey`选项生成一个新的密钥，并将其序列化到给定的文件。`-encrypt`和`-decrypt`选项都调用`crypt()`方法，而该方法调用`Cipher`对象的`update()`和`doFinal()`方法。

[程序清单10-12 aes/AESTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch10/aes/AESTest.java)

[程序清单10-13 aes/Util.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch10/aes/Util.java)

### 10.4.3 加密流
Java加密扩展提供了一组便捷的流类，用于对数据自动加密或解密。例如，可以使用`CipherOutputStream`将数据加密后写入文件：

```java
Cipher cipher = ...;
cipher.init(Cipher.ENCRYPT_MODE, key);
var out = new CipherOutputStream(new FileOutputStream(outputFileName), cipher);
var bytes = new byte[BLOCKSIZE];
int inLength = getData(bytes); // get data from data source
while (inLength != -1) {
    out.write(bytes, 0, inLength);
    inLength = getData(bytes); // get more data from data source
}
out.flush();
```

类似地，可以使用`CipherInputStream`从文件读取并解密数据：

```java
Cipher cipher = ...;
cipher.init(Cipher.DECRYPT_MODE, key);
var in = new CipherInputStream(new FileInputStream(inputFileName), cipher);
var bytes = new byte[BLOCKSIZE];
int inLength = in.read(bytes);
while (inLength != -1) {
    putData(bytes, inLength); // put data to destination
    inLength = in.read(bytes);
}
```

加密流类能够自动调用`update()`和`doFinal()`方法。

### 10.4.4 公钥加密
前面看到的AES加密是一种对称加密，即加密和解密使用相同的密钥。对称加密的致命缺点是密钥分发。如果Alice给Bob发送加密消息，Bob需要使用与Alice相同的密钥。如果Alice修改了密钥，她需要通过安全渠道给Bob发送新的密钥。但是也许她并没有安全渠道——这正是她对消息进行加密的原因。

**公钥加密**(public key cryptography)（也称为非对称加密）解决了这个问题。在公钥加密中，Bob有一个密钥对，由公钥和匹配的私钥组成。Bob可以在任何地方发布公钥，但必需严格保守私钥。Alice只需使用公钥对她发送给Bob的消息进行加密即可。

注：假设n个人之间互相通信，如果使用对称加密，每个人需要管理n-1个密钥，总共有n(n-1)/2个密钥，管理难度大。而使用非对称加密，每个人只需管理自己的密钥对，总共只有n个密钥对。

实际上，所有已知的公钥加密算法都比对称加密算法（如DES或AES）**慢得多**。使用公钥加密对大量的信息进行加密是不切实际的。但是，通过将公钥加密与快速对称加密相结合可以很容易地克服这个问题：
1. Alice生成一个随机的对称密钥(k<sub>1</sub>)，用它加密明文(x → k<sub>1</sub>(x))。
2. Alice用Bob的公钥(k<sub>2</sub>)加密对称密钥(k<sub>1</sub> → k<sub>2</sub>(k<sub>1</sub>))。
3. Alice将加密后的对称密钥和加密后的明文发送给Bob。
4. Bob用他的私钥解密对称密钥(k<sub>2</sub>(k<sub>1</sub>) → k<sub>1</sub>)。
5. Bob用对称密钥解密消息(k<sub>1</sub>(x) → x)。

除了Bob，没有人能解密对称密钥，因为只有Bob有私钥。这样，昂贵的公钥加密只应用于少量数据（对称密钥）。

最常用的公钥加密算法是由Ron Rivest、Adi Shamir和Leonard Adleman发明的RSA算法（发明人姓氏首字母缩写）。现在该算法已经公开。

为了使用RSA算法，需要用`KeyPairGenerator`生成一个公钥/私钥对：

```java
KeyPairGenerator pairgen = KeyPairGenerator.getInstance("RSA");
var random = new SecureRandom();
pairgen.initialize(KEYSIZE, random);
KeyPair keyPair = pairgen.generateKeyPair();
Key publicKey = keyPair.getPublic();
Key privateKey = keyPair.getPrivate();
```

程序清单10-14中的程序有三个选项。`-genkey`选项生成一个密钥对。

```shell
java rsa.RSATest -genkey public.key private.key
```

`-encrypt`选项生成一个AES密钥并使用公钥对其包装(wrap)，然后生成一个文件，包含包装过的密钥长度和字节以及使用AES密钥加密的明文。

```shell
java rsa.RSATest -encrypt plaintextFile encryptedFile public.key
```

`-decrypt`选项解密上述文件。

```shell
java rsa.RSATest -decrypt encryptedFile decryptedFile private.key
```

[程序清单10-14 rsa/RSATest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch10/rsa/RSATest.java)
