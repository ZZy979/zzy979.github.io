---
title: 《Java核心技术》笔记 卷II 第8章 脚本、编译和注解处理
date: 2025-07-26 10:43:36 +0800
categories: [Java, Core Java]
tags: [java, script, compiler, annotation, bytecode]
---
本章将介绍三种用于处理代码的技术：脚本API使你可以调用诸如JavaScript和Groovy这样的脚本语言代码；当你希望在应用程序内部编译Java代码时，可以使用编译器API；注解处理器可以操作包含注解的Java源代码和类文件。

## 8.1 Java平台的脚本
脚本语言是一种通过在运行时解释程序文本，从而避免通常的编辑/编译/链接/运行循环的语言。脚本语言有许多优势：快速反馈、鼓励实验，可以改变正在运行的程序的行为，允许用户自定义。另一方面，大多数脚本语言缺乏对编写复杂应用有益的特性，例如强类型、封装和模块化。

因此，将脚本语言和传统语言的优势相结合是很有诱惑力的。脚本API使你可以在Java平台上实现这一点。它能够在Java程序中调用使用JavaScript、Groovy、Ruby，甚至更加奇异的Scheme和Haskell等语言编写的脚本。

### 8.1.1 获取脚本引擎
脚本引擎是可以执行用特定语言编写的脚本的库。当虚拟机启动时，它会发现可用的脚本引擎。为了枚举这些引擎，构造一个`ScriptEngineManager`并调用`getEngineFactories()`方法。可以查询每个引擎工厂支持的引擎名、MIME类型和文件扩展名。

| 引擎 | 名称 | MIME类型 | 扩展名 |
| --- | --- | --- | --- |
| Rhino (JavaScript) | rhino, Rhino,<br>JavaScript, javascript | application/javascript,<br>application/ecmascript,<br>text/javascript,<br>text/ecmascript | js |
| Groovy | groovy | 无 | groovy |
| Renjin | Renjin | text/x-R | R, r, S, s |

可以通过名称、MIME类型或扩展名获得脚本引擎。例如：

```java
var manager = new ScriptEngineManager();
ScriptEngine engine = manager.getEngineByName("javascript");
```

需要在类路径中提供实现脚本引擎的JAR文件。（Oracle JDK曾经包含一个JavaScript引擎(Nashorn)，但是在Java 15中被移除了。）

### 8.1.2 脚本计算和绑定
一旦有了引擎，就可以通过以下方法来调用脚本：

```java
Object result = engine.eval(scriptString);
```

如果脚本存储在文件中，也可以提供一个`Reader`。

可以在同一个引擎上多次调用脚本。如果脚本定义了变量、函数或类，大多数引擎都会保留这些定义供将来使用。例如：

```java
engine.eval("n = 1728");
Object result = engine.eval("n + 1"); // 1729
```

注释：要确定在多个线程中并发执行脚本是否安全，调用

```java
Object param = factory.getParameter("THREADING");
```

返回值是下列之一：
* `null`：并发执行不安全。
* `"MULTITHREADED"`：并发执行安全。一个线程的执行效果对另一个线程可能是可见的。
* `"THREAD-ISOLATED"`：除了满足`"MULTITHREADED"`，还会为每个线程维护不同的变量绑定。
* `"STATELESS"`：除了满足`"THREAD-ISOLATED"`，脚本不会修改变量绑定。

可以向引擎添加变量绑定。绑定由名字和关联的Java对象构成。例如：

```java
engine.put("k", 1728);
Object result = engine.eval("k + 1");
```

脚本代码从“引擎作用域”的绑定中读取`k`的定义。

大多数脚本语言都可以访问Java对象，通常比Java语法更简单。例如：

```java
engine.put("b", new JButton());
engine.eval("b.text = 'Ok'");
```

反过来，也可以获取由脚本语句绑定的变量：

```java
engine.eval("n = 1728");
Object result = engine.get("n");
```

除了引擎作用域，还有全局作用域。任何添加到`ScriptEngineManager`的绑定对所有引擎都是可见的。另外，还可以将绑定收集到一个`Bindings`类型的对象中，并将其传递给`eval()`方法：

```java
Bindings scope = engine.createBindings();
scope.put("b", new JButton());
engine.eval(scriptString, scope);
```

注释：如果需要除了引擎和全局作用域之外的其他作用域（例如Web容器可能需要请求和会话作用域），需要写一个实现`ScriptContext`接口的类。

### 8.1.3 重定向输入和输出
可以通过调用脚本上下文对象的`setReader()`和`setWriter()`方法来重定向脚本的标准输入和输出。例如

```java
var writer = new StringWriter();
engine.getContext().setWriter(new PrintWriter(writer, true));
```

这两个方法只会影响脚本引擎的标准输入和输出。例如，如果执行下面的JavaScript代码：

```javascript
println("Hello");
java.lang.System.out.println("World");
```

只有第一个输出会被重定向。

Rhino引擎没有标准输入源的概念，因此调用`setReader()`没有任何效果。

### 8.1.4 调用脚本函数和方法
许多脚本引擎可以在不计算脚本代码的情况下调用脚本语言的函数。提供这种功能的脚本引擎（如Rhino）实现了`Invocable`接口。

要调用一个函数，需要调用`invokeFunction()`方法并指定函数名和函数参数：

```java
// Define greet function in JavaScript
engine.eval("function greet(how, whom) { return how + ', ' + whom + '!' }");

// Call the function with arguments "Hello", "World"
result = ((Invocable) engine).invokeFunction("greet", "Hello", "World"); // "Hello, World!"
```

如果脚本语言是面向对象的，可以调用`invokeMethod()`：

```java
// Define Greeter class in JavaScript
engine.eval("function Greeter(how) { this.how = how }");
engine.eval("Greeter.prototype.welcome = "
    + " function(whom) { return this.how + ', ' + whom + '!' }");
  
// Construct an instance
Object yo = engine.eval("new Greeter('Yo')");

// Call the welcome method on the instance
result = ((Invocable) engine).invokeMethod(yo, "welcome", "World"); // "Yo, World!"
```

注释：Rhino不支持现代JavaScript类语法。

注释：即使脚本引擎没有实现`Invocable`接口，仍然可以用语言无关的方式调用方法。`ScriptEngineFactory`接口的`getMethodCallSyntax()`方法产生一个可以传递给`eval()`方法的字符串。但是，所有的方法参数都必须与名字绑定。（例如，`factory.getMethodCallSyntax("yo", "welcome", "whom")`返回`"yo.welcome(whom);"`）

可以更进一步，让脚本引擎实现一个Java接口。然后就可以用Java语法来调用脚本函数和方法。

细节取决于脚本引擎，但一般需要为接口中的每个方法提供一个函数。例如，考虑下面的Java接口：

```java
public interface Greeter {
    String welcome(String whom);
}
```

如果在Rhino中定义了具有相同名字的全局函数，就可以通过这个接口调用它：

```java
// Define welcome function in JavaScript
engine.eval("function welcome(whom) { return 'Hello, ' + whom + '!' }");

// Get a Java object and call a Java method
Greeter g = ((Invocable) engine).getInterface(Greeter.class);
result = g.welcome("World"); // "Hello, World!"
```

在面向对象的脚本语言中，可以通过匹配的Java接口来访问一个脚本类。例如，可以像这样用Java语法来调用JavaScript `Greeter`类的对象：

```java
Greeter g = ((Invocable) engine).getInterface(yo, Greeter.class);
result = g.welcome("World"); // "Yo, World!"
```

### 8.1.5 编译脚本
某些脚本引擎可以将脚本代码编译成中间格式，以便高效执行。这些引擎实现了`Compilable`接口。下面的示例展示了如何编译并计算包含在脚本文件中的代码：

```java
var reader = new FileReader("myscript.js");
CompiledScript script = null;
if (engine instanceof Compilable)
    script = ((Compilable) engine).compile(reader);
```

一旦脚本被编译，就可以执行它。

```java
if (script != null)
    script.eval();
else
    engine.eval(reader);
```

当然，只有需要重复执行时，编译脚本才有意义。

### 8.1.6 示例：用脚本处理GUI事件
为了演示脚本API，本节将编写一个示例程序，允许用户指定用自己选择的脚本语言编写的GUI事件处理器。

程序清单8-1中的程序可以向任意窗体类添加脚本。默认情况下，它会读取程序清单8-2中的`ButtonFrame`类，该类与卷I第10章中的（程序清单10-5）类似，但是有两个差异：
* 每个组件都设置了`name`属性。
* 没有事件处理器。

事件处理器是在属性文件中定义的，每个属性定义具有以下形式：

```
componentName.eventName = scriptCode
```

例如，如果选择使用JavaScript，就在js.properties文件中提供事件处理器：

```properties
yellowButton.action=panel.background = java.awt.Color.YELLOW
blueButton.action=panel.background = java.awt.Color.BLUE
redButton.action=panel.background = java.awt.Color.RED
```

示例代码还包括用于Groovy和R的属性文件。

该程序首先加载指定脚本语言的引擎（默认为JavaScript）。然后处理init.language脚本（如果存在），用于像R这样需要初始化的语言。接下来，递归遍历所有子组件，并将(name, object)绑定添加到一个映射，然后将绑定添加到引擎。最后，读取language.properties文件。对于每个属性，合成一个事件处理器代理，来执行脚本代码（参见卷I第6章 6.5节）。关键在于每个事件处理器都调用了`engine.eval()`。

[程序清单8-1 script/ScriptTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch08/script/ScriptTest.java)

[程序清单8-2 buttons1/ButtonFrame.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch08/buttons1/ButtonFrame.java)

如果使用Java 15或以上，类路径必须包含一个像Rhino (<https://rhino.github.io/>)这样的JavaScript引擎。像这样运行该程序：

```shell
java -classpath .:rhino-1.7.14.jar:rhino-engine-1.7.14.jar script.ScriptTest
```

对于Groovy (<https://www.groovy-lang.org/>)，使用

```shell
java -classpath .:$groovy/lib/\* script.ScriptTest groovy
```

其中$groovy是Groovy的安装目录。

对于R的Renjin实现，需要在类路径中包含Renjin Studio和脚本引擎的JAR文件，可以在 <https://www.renjin.org/downloads.html> 下载。

## 8.2 编译器API
有很多工具都需要编译Java代码，例如集成开发环境和Java Server Pages（JSP，嵌入了Java代码的网页）。

### 8.2.1 调用编译器
调用编译器非常简单，下面是一个示例：

```java
JavaCompiler compiler = ToolProvider.getSystemJavaCompiler();
OutputStream outStream = ...;
OutputStream errStream = ...;
int result = compiler.run(null, outStream, errStream, "-sourcepath", "src", "Test.java");
```

返回值为0表示编译成功。

编译器会将输出和错误消息发送到提供的流，如果为`null`则使用`System.out`和`System.err`。第一个参数是输入流，编译器不接受控制台输入，因此始终为`null`（`run()`方法继承自`Tool`接口，它允许工具读取输入）。`run()`方法的其余参数（选项或文件名）将传递给`javac`。

### 8.2.2 发起编译任务
可以使用`CompilationTask`对象对编译过程进行更多控制。例如，从字符串提供源码，在内存中捕获类文件，或者处理错误和警告消息。

可以像这样获得`CompilationTask`对象：

```java
JavaCompiler.CompilationTask task = compiler.getTask(
    errorWriter, // Uses System.err if null
    fileManager, // Uses the standard file manager if null
    diagnostics, // Uses System.err if null
    options, // null if no options
    classes, // For annotation processing; null if none
    sources);
```

例如：

```java
StandardJavaFileManager fileManager = compiler.getStandardFileManager(null, null, null);
Iterable<String> fileNames = List.of("File1.java", "File2.java");
Iterable<JavaFileObject> sources = fileManager.getJavaFileObjectsFromStrings(fileNames);
Iterable<String> options = List.of("-d", "bin");
JavaCompiler.CompilationTask task = compiler.getTask(null, null, null, options, null, sources);
```

注释：`classes`参数只用于注解处理。在这种情况下，还需要调用`task.processors()`。注解处理的示例参见8.6节。

`getTask()`方法返回任务对象，但是并不启动编译过程。`CompilationTask`类扩展了`Callable<Boolean>`，因此可以将其传递给`ExecutorService`以并行执行，也可以只进行异步调用：

```java
Boolean success = task.call();
```

### 8.2.3 捕获诊断信息
为了监听错误消息，需要安装一个`DiagnosticListener`。监听器在编译器报告警告或错误消息时会接收到一个`Diagnostic`对象。`DiagnosticCollector`类实现了该接口，它会收集所有诊断信息，可以在编译完成之后遍历这些信息。

```java
DiagnosticCollector<JavaFileObject> collector = new DiagnosticCollector<>();
compiler.getTask(null, fileManager, collector, null, null, sources).call();
for (Diagnostic<? extends JavaFileObject> d : collector.getDiagnostics()) {
    System.out.println(d);
}
```

`Diagnostic`对象包含有关问题位置的信息（包括文件名、行号和列号），以及人类可读的描述。

还可以在标准文件管理器上安装诊断监听器，以便捕获有关缺失文件的消息：

```java
StandardJavaFileManager fileManager = compiler.getStandardFileManager(diagnostics, null, null);
```

### 8.2.4 从内存中读取源文件
如果动态生成源代码，就可以从内存中编译，而无需将文件保存到磁盘。例如，使用下面的类持有源代码：

[compiler/StringSource.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch08/compiler/StringSource.java)

然后生成类的代码，并给编译器一个`StringSource`对象的列表：

```java
List<StringSource> sources = List.of(new StringSource(className1, class1CodeString), ...);
task = compiler.getTask(null, fileManager, diagnostics, null, null, sources);
```

### 8.2.5 将字节码写出到内存
如果动态地编译类，就无需将类文件保存到磁盘。可以将其保存到内存中并立刻加载。

首先，要有一个类来持有这些字节：

[compiler/ByteArrayClass.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch08/compiler/ByteArrayClass.java)

接下来，需要配置文件管理器使用该类作为输出：

```java
List<ByteArrayClass> classes = new ArrayList<>();
StandardJavaFileManager stdFileManager = compiler.getStandardFileManager(null, null, null);
JavaFileManager fileManager = new ForwardingJavaFileManager<JavaFileManager>(stdFileManager) {
    public JavaFileObject getJavaFileForOutput(
            Location location, String className, Kind kind, FileObject sibling) throws IOException {
        if (kind == Kind.CLASS) {
            ByteArrayClass outfile = new ByteArrayClass(className);
            classes.add(outfile);
            return outfile;
        }
        else
            return super.getJavaFileForOutput(location, className, kind, sibling);
    }
};
```

为了加载这个类，需要一个类加载器（参见第10章）：

[compiler/ByteArrayClassLoader.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch08/compiler/ByteArrayClassLoader.java)

编译完成后，用上面的类加载器调用`Class.forName()`方法：

```java
ByteArrayClassLoader loader = new ByteArrayClassLoader(classes);
Class<?> cl = Class.forName(className, true, loader);
```

（好麻烦……）

### 8.2.6 示例：动态Java代码生成
在实现动态网页的JSP技术中，可以将HTML与Java代码片段混合。例如：

```html
<p>The current date and time is <b><%= new java.util.Date() %></b>.</p>
```

JSP引擎动态地将Java代码编译成Servlet。在示例应用中，我们使用了一个更简单的示例：动态生成Swing代码。其基本思想是使用GUI构建器在窗体中放置组件，并在一个外部文件中指定组件的行为。程序清单8-4展示了一个非常简单的窗体类示例，程序清单8-5展示了按钮动作的代码。注意，窗体类的构造器调用了抽象方法`addEventHandlers()`。代码生成器将产生一个实现了该方法的子类，对于action.properties文件中的每一行添加一个动作监听器。我们将这个子类放在名为`x`的包中，不希望在程序其他地方使用。生成的代码具有如下形式：

```java
package x;

public class Frame extends SuperclassName {
    protected void addEventHandlers() {
        componentName1.addActionListener(event -> {
            // code for event handler1
        });
        // repeat for the other event handlers ...
    }
}
```

程序清单8-3中的`buildSource()`方法生成了这些代码，并将其放到`StringSource`对象中。该对象会被传递给Java编译器（`getTask()`方法的`sources`参数）。

我们使用了`ForwardingJavaFileManager`作为文件管理器（如前一节所述），它会为每个已编译的类构造一个`ByteArrayClass`对象，这些对象会捕获编译`x.Frame`类生成的类文件。

编译完成后，我们使用前一节中描述的`ByteArrayClassLoader`来加载窗体类，然后构造并显示应用程序的窗体。

```java
var loader = new ByteArrayClassLoader(classFileObjects);
var frame = (JFrame) loader.loadClass("x.Frame").getConstructor().newInstance();
frame.setVisible(true);
```

[程序清单8-3 compiler/CompilerTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch08/compiler/CompilerTest.java)

[程序清单8-4 buttons2/ButtonFrame.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch08/buttons2/ButtonFrame.java)

[程序清单8-5 buttons2/action.properties](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch08/buttons2/action.properties)

点击按钮时，背景色会改变。为了验证这些动作是动态编译的，修改action.properties文件中的一行，例如：

```properties
yellowButton=panel.setBackground(java.awt.Color.YELLOW); yellowButton.setEnabled(false);
```

再次运行程序，现在Yellow按钮点击后会被禁用。查看代码目录，会发现没有任何`x`包中的资源或类文件。

## 8.3 使用注解
**注解**(annotation)是插入到源代码中、可以使用其他工具进行处理的标签。这些工具可以在源代码级别上操作，也可以处理包含注解的类文件。

注解不会改变程序的编译方式。Java编译器对于包含和不包含注解的代码会生成相同的虚拟机指令。

为了利用注解，需要选择一个**处理工具**。在代码中插入处理工具可以理解的注解，然后用处理工具处理代码。

注解的使用范围很广泛，例如：
* 自动生成辅助文件，如部署描述符或bean信息类。
* 自动生成测试、日志、事务语义等代码。

### 8.3.1 注解简介
下面是一个简单的注解示例：

```java
public class MyClass {
    ...
    @Test
    public void checkRandomInsertions()
}
```

`@Test`用于注解`checkRandomInsertions()`方法。

在Java中，注解位于被注解项之前，名字以`@`开头。

`@Test`注解本身并不做任何事，需要工具支持才会有用。例如，JUnit测试工具(<https://junit.org/>)在测试一个类时会调用所有标注了`@Test`的方法。另一个工具可能会删除类文件中所有的测试方法，以免与程序一起发布。

注解可以包含**元素**，例如：

```java
@Test(timeout = 10000)
```

元素可以被读取注解的工具处理。

除了方法，还可以注解类、字段和局部变量。注解可以放在任何可以放置修饰符（如`public`或`static`）的地方。另外，还可以注解包、参数、类型参数和类型用法。

注解必须通过注解接口定义。接口的方法对应注解的元素。例如，JUnit的`Test`注解由以下接口定义：

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Test {
    long timeout() default 0L;
    ...
}
```

`@interface`声明创建了一个真实的Java接口。处理工具会接收到实现了该接口的对象，可以调用`timeout()`方法获取特定`Test`注解的`timeout`元素。

注解`Target`和`Retention`是元注解。它们注解了`Test`注解，将在8.5.2节详细讨论。

注释：对于有吸引力的注解用法，可以查看JCommander (<https://jcommander.org/>)和picocli (<https://picocli.info/>)。这些库将注解用于命令行参数的处理。

### 8.3.2 示例：注解事件处理器
在用户界面编程中，一件比较讨厌的事情是将监听器连接到事件源，例如：

```java
myButton.addActionListener(event -> doSomething());
```

本节设计了一个注解来进行反向连接（将事件源标注到监听器方法）。该注解定义在程序清单8-8中，用法如下：

```java
@ActionListenerFor(source = "myButton")
void doSomething() { ... }
```

程序清单8-7展示了卷I第10章的`ButtonFrame`类，但使用该注解重新实现了。

注解本身并不会做任何事情。现在需要一种机制来分析注解并安装动作监听器，这是`ActionListenerInstaller`类的职责。`ButtonFrame`构造器调用了这个类的静态方法`processAnnotations()`。该方法枚举给定对象（在示例程序中是`ButtonFrame`对象）的所有方法，对于每个方法，获得`ActionListenerFor`注解对象并处理它（如果有）。

```java
Class<?> cl = obj.getClass();
for (Method m : cl.getDeclaredMethods()) {
    ActionListenerFor a = m.getAnnotation(ActionListenerFor.class);
    if (a != null) ...
}
```

这里使用了`AnnotatedElement`接口的`getAnnotation()`方法。`Method`、`Constructor`、`Field`、`Class`和`Package`类都实现了这个接口。

可以通过调用注解对象的`source()`方法获取源字段的名字，然后查找匹配的字段（如`yellowButton`）。

```java
String fieldName = a.source();
Field f = cl.getDeclaredField(fieldName);
```

对于每个被注解的方法，构造了一个实现`ActionListener`接口的代理对象，其`actionPerformed()`方法调用被注解的方法（监听器）（关于代理详见卷I第6章 6.5节）。

[程序清单8-6 runtimeAnnotations/ActionListenerInstaller.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch08/runtimeAnnotations/ActionListenerInstaller.java)

[程序清单8-7 buttons3/ButtonFrame.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch08/buttons3/ButtonFrame.java)

[程序清单8-8 runtimeAnnotations/ActionListenerFor.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch08/runtimeAnnotations/ActionListenerFor.java)

在这个示例中，注解是在运行时处理的。也可以在源码级别处理：源代码生成器产生用于添加监听器的代码。或者，也可以在字节码级别处理：字节码编辑器将`addActionListener()`调用注入窗体构造器。

## 8.4 注解语法
### 8.4.1 注解接口
注解是由**注解接口**(annotation interface)定义的：

```java
modifiers @interface AnnotationName {
    elementDeclaration1
    elementDeclaration2
    ...
}
```

每个元素声明形如

```java
type elementName();
```

或者

```java
type elementName() default value;
```

例如，下面的注解有两个元素，`assignedTo`和`severity`：

```java
public @interface BugReport {
    String assignedTo() default "[none]";
    int severity();
}
```

所有注解接口都隐式地扩展了`java.lang.annotation.Annotation`接口。这个接口是常规接口，不是注解接口。**不能扩展注解接口**——换句话说，所有注解接口都直接扩展自`Annotation`。也从不会为注解接口提供实现类（虽然可以但无意义）。

注解接口的方法没有参数和`throws`子句，不能是`default`或`static`方法，也不能有类型参数。

注解元素的类型为下列之一：
* 基本类型
* `String`
* `Class`（具有可选的类型参数，例如`Class<? extends MyClass>`）
* 枚举类型
* 注解类型
* 上述类型的数组（多维数组是非法的）

例如：

```java
public @interface BugReport {
    enum Status { UNCONFIRMED, CONFIRMED, FIXED, NOTABUG };
    boolean showStopper() default false;
    String assignedTo() default "[none]";
    Class<?> testCase() default Void.class;
    Status status() default Status.UNCONFIRMED;
    Reference ref() default @Reference(); // an annotation type
    String[] reportedBy();
}
```

### 8.4.2 注解
注解具有以下格式：

```java
@AnnotationName(elementName1 = value1, elementName2 = value2, ...)
```

例如

```java
@BugReport(assignedTo = "Harry", severity = 10)
```

元素的顺序无关紧要。

如果未指定某个元素的值，则使用声明的默认值。例如：

```java
@BugReport(severity = 10)  // assignedTo is "[none]"
```

警告：默认值不是和注解一起存储的，而是动态计算的。例如，如果将`assignedTo`元素的默认值改为`"[]"`并重新编译`BugReport`接口，注解`@BugReport(severity = 10)`将使用新的默认值（即使它所在的类文件没有重新编译）。

有两个特殊的捷径可以简化注解。如果没有指定元素（因为注解没有任何元素，或者所有元素都使用默认值），就不需要使用圆括号。例如，`@BugReport`等价于`@BugReport(assignedTo = "[none]", severity = 0)`。这种注解称为**标记注解**(marker annotation)。

另一种捷径是**单值注解**(single-value annotation)。如果仅指定一个具有特殊名字`value`的元素，就可以省略元素名和等号。例如，假设将前一节的注解接口`ActionListenerFor`定义为

```java
public @interface ActionListenerFor {
    String value();
}
```

就可以将注解写成`@ActionListenerFor("yellowButton")`。

一个项可以有多个注解，例如：

```java
@Test
@BugReport(showStopper = true, reportedBy = "Joe")
public void checkRandomInsertions()
```

如果注解声明为可重复的（使用`@Repeatable`），就可以多次重复使用同一个注解：

```java
@BugReport(showStopper = true, reportedBy = "Joe")
@BugReport(reportedBy = {"Harry", "Carl"})
public void checkRandomInsertions()
```

注释：注解的所有元素值必须是编译时常量。

警告：注解元素永远不能设置为`null`，甚至不允许默认值为`null`。必须使用其他的默认值，例如`""`或者`Void.class`。

如果元素值是数组，需要将它的值用花括号括起来：

```java
@BugReport(reportedBy = {"Harry", "Carl"})
```

如果数组只有单个值，就可以省略花括号：

```java
@BugReport(reportedBy = "Joe") // OK, same as {"Joe"}
```

由于注解元素可以是另一个注解，你可以构建出任意复杂的注解。例如，

```java
@BugReport(ref = @Reference(id = "3352627"), ...)
```

注释：在注解中引入循环依赖是错误的。例如，`BugReport`具有注解类型`Reference`的元素，因此`Reference`不能有`BugReport`类型的元素。

### 8.4.3 注解声明
注解可以出现在许多地方，这些地方可分为两类：声明和类型用法。声明注解可以出现在下列声明处：
* 包
* 类（包括`enum`）
* 接口（包括注解接口）
* 方法
* 构造器
* 实例字段（包括`enum`常量）
* 局部变量
* 参数
* 类型参数

对于类和接口，将注解放在`class`或`interface`关键字前：

```java
@Entity public class User { ... }
```

对于变量和参数，将其放在类型前：

```java
@SuppressWarnings("unchecked") List<User> users = ...;
public User getUser(@Param("id") String userId)
```

泛型类或方法中的类型参数可以像这样注解：

```java
public class Cache<@Immutable V> { ... }
```

包在文件package-info.java中注解，该文件只包含包语句：

```java
/**
 * Package-level Javadoc
 */
@GPL(version = "3")
package com.horstmann.corejava;
import org.gnu.GPL;
```

注释：局部变量的注解只能在源码级别处理（因为在编译完后就会被丢弃）。类似地，包注解也不会在源码级别之外保留。

### 8.4.4 注解类型用法
**类型用法**(type use)注解可以出现在下列位置：
* 类型参数：`List<@NonNull String>`, `Comparator.<@NonNull String>reverseOrder()`
* 数组的任何位置：
  * `@NonNull String[][] words`（`words[i][j]`不为`null`）
  * `String @NonNull [][] words`（`words`不为`null`）
  * `String[] @NonNull [] words`（`words[i]`不为`null`）
* 超类和实现接口：`class Warning extends @Localized Message`
* 构造器调用：`new @Localized String(...)`
* 强制类型转换和`instanceof`检查：`(@Localized String) text`, `if (text instanceof @Localized String)`（这些注解仅供外部工具使用，对类型转换和`instanceof`检查的行为没有影响）
* 异常声明：`public String read() throws @Localized IOException`
* 通配符和类型边界：`List<@Localized ? extends Message>`, `List<? extends @Localized Message>`
* 方法和构造器引用：`@Localized Message::getText`

有些位置不能被注解：

```java
@NonNull String.class // ERROR: Cannot annotate class literal
import java.lang.@NonNull String; // ERROR: Cannot annotate import
```

可以将注解放在修饰符的前面或后面。习惯（但不是必需）的做法是：将类型用法注解放在修饰符之后，将声明注解放在修饰符之前。例如，

```java
private @NonNull String text; // Annotates the type use
@Id private String userId; // Annotates the variable
```

在注解记录组件（见卷I第4章 4.7节）时，注解也应用于生成的字段、getter方法和构造器参数。

注释：注解的作者需要指定注解可以出现在哪里。如果一个注解既可用于声明也可用于类型用法，则当它被用于变量声明时，该变量和类型用法都会被注解。例如：

```java
public User getUser(@NonNull String userId)
```

如果`@NonNull`可同时用于声明和类型用法，那么`userId`参数被注解了，并且参数类型是`@NonNull String`。

### 8.4.5 注解this
假设想要注解不会被方法修改的参数：

```java
public class Point {
    public boolean equals(@ReadOnly Object other) { ... }
}
```

那么处理这个注解的工具在看到调用`p.equals(q)`时就会推理出`q`没有被修改过。当该方法被调用时，`this`变量绑定到`p`。但是`this`从来没有声明过，因此无法注解它。

实际上，可以用一种很少使用的语法变体来声明它，这样就可以添加注解了：

```java
public class Point {
    public boolean equals(@ReadOnly Point this, @ReadOnly Object other) { ... }
}
```

第一个参数称为**接收器参数**(receiver parameter)，它必须命名为`this`，其类型是方法所属的类。

注释：不能为构造器提供接收器参数（内部类除外）。从概念上讲，构造器中的`this`引用在构造器没有执行完之前不是给定类型的对象。放在构造器上的注解描述了被构造对象的属性。

内部类构造器的接收器参数是外部类对象的引用。

```java
public class Sequence {
    private int from;
    private int to;

    class Iterator implements java.util.Iterator<Integer> {
        private int current;
        public Iterator(@ReadOnly Sequence Sequence.this) {
            this.current = Sequence.this.from;
        }
        ...
    }
    ...
}
```

## 8.5 标准注解
`java.lang`、`java.lang.annotation`和`javax.annotation`包中定义了许多注解接口。下表展示了这些标准注解。

| 注解接口 | 应用场合 | 目的 |
| --- | --- | --- |
| `Deprecated` | 所有 | 标记为已弃用 |
| `SuppressWarnings` | 除了包和注解 | 抑制给定类型的警告 |
| `SafeVarargs` | 方法和构造器 | 断言可变参数可以安全使用 |
| `Override` | 方法 | 检查该方法覆盖了超类方法 |
| `Serial` | 方法 | 检查该方法是正确的序列化方法 |
| `FunctionalInterface` | 方法 | 标记为函数式接口（具有单个抽象方法） |
| `Generated` | 所有 | 标记为由工具生成的源代码 |
| `Target` | 注解 | 指定该注解可应用的项 |
| `Retention` | 注解 | 指定该注解保留多久 |
| `Documented` | 注解 | 指定该注解应该包含在被注解项的文档中 |
| `Inherited` | 注解 | 指定当该注解应用于类时自动被子类继承 |
| `Repeatable` | 注解 | 指定该注解可以多次应用于同一个项 |

### 8.5.1 用于编译的注解
`@Deprecated`注解可以添加到任何不再鼓励使用的项。当使用已弃用的项时，编译器会发出警告。该注解与Javadoc标签`@deprecated`具有相同的作用，但注解会保留到运行时。

注释：JDK自带的`jdeprscan`工具可以扫描JAR文件中已弃用的元素。

`@SuppressWarnings`注解告诉编译器抑制特定类型的警告。例如，`@SuppressWarnings("unchecked")`。

`@Override`注解只能应用于方法。编译器会检查具有该注解的方法真正覆盖了一个超类方法。例如，如果声明

```java
public MyClass {
    @Override public boolean equals(MyClass other);
    ...
}
```

那么编译器会报错——该方法没有覆盖`Object`类的`equals()`方法，因为其参数类型是`Object`而不是`MyClass`。

`@Generated`注解旨在供代码生成工具使用，用于将生成的代码与程序员编写的代码区分开。每个注解必须包含表示代码生成器的唯一标识符，日期字符串（ISO 8601格式）和注释字符串是可选的。例如，

```java
@Generated("com.horstmann.beanproperty", "2008-01-04T12:08:56.235-0700");
```

### 8.5.2 元注解
**元注解**(meta-annotation)即“注解的注解”。

元注解`@Target`用于限制注解可应用的项。例如，

```java
@Target({ElementType.TYPE, ElementType.METHOD})
public @interface BugReport
```

下表列出了所有可能的取值。它们属于枚举类型`ElementType`。可以指定任意数量的值，用花括号括起来。

| 元素类型 | 应用场合 |
| --- | --- |
| `ANNOTATION_TYPE` | 注解类型声明 |
| `PACKAGE` | 包 |
| `TYPE` | 类（包括枚举）和接口（包括注解接口） |
| `METHOD` | 方法 |
| `CONSTRUCTOR` | 构造器 |
| `FIELD` | 字段（包括枚举常量） |
| `PARAMETER` | 方法或构造器参数 |
| `LOCAL_VARIABLE` | 局部变量 |
| `TYPE_PARAMETER` | 类型参数 |
| `TYPE_USE` | 类型用法 |

没有`@Target`限制的注解可应用于任何项。如果将注解应用于不允许的项会导致编译错误。

元注解`@Retention`指定注解保留多久，可以指定下表中的值之一（属于枚举类型`RetentionPolicy`），默认值是`CLASS`。

| 保留策略 | 描述 |
| --- | --- |
| `SOURCE` | 注解不包括在类文件中 |
| `CLASS` | 注解包括在类文件中，但虚拟机不加载 |
| `RUNTIME` | 注解包括在类文件中，并由虚拟机加载，可通过反射获得 |

在程序清单8-8中，`ActionListenerFor`注解声明为`RUNTIME`，因为使用了反射来处理注解。在下面两节中将会看到在源码和类文件级别处理注解的示例。

元注解`@Documented`为诸如Javadoc这样的文档工具提供了一些提示。Javadoc会在输出的文档中显示带有`@Documented`的注解，而其他注解不会包含在文档中。例如，假设将`ActionListenerFor`声明为

```java
@Documented
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface ActionListenerFor
```

现在每个被该注解标注的方法的文档就会显示该注解，如下图所示。

![文档化注解](/assets/images/java-note-v2ch08-scripting-compiling-and-annotation-processing/文档化注解.png)

如果一个注解是暂时性的（例如`@BugReport`），就不应该将其包含在文档中。

注释：将注解应用于自身是合法的。例如，`@Documented`本身被注解为`@Documented`。

元注解`@Inherited`只能应用于类的注解。如果一个类具有继承注解，那么它的所有子类都自动具有相同的注解。这使得创建与标记接口（如`Serializable`）具有相同作用的注解变得很容易。

例如，假设定义了一个继承注解`@Persistent`来指示一个类的对象可以存储到数据库中，那么该类的子类会自动被注解为持久性的。

```java
@Inherited @interface Persistent {}
@Persistent class Employee { ... }
class Manager extends Employee { ... } // also @Persistent
```

从Java 8起，将同一注解类型多次应用于一个项是合法的。为了向后兼容，可重复注解的实现者需要提供一个**容器注解**(container annotation)，将重复的注解包装到一个数组中。

下面定义了可重复注解`@TestCase`及其容器注解`@TestCases`：

```java
@Repeatable(TestCases.class)
@interface TestCase {
    String params();
    String expected();
}

@interface TestCases {
    TestCase[] value();
}
```

警告：处理可重复注解时必须小心。如果调用`getAnnotation()`来查找可重复注解，而该注解确实重复了多次，就会得到`null`（如果只出现一次则不为`null`）。这是因为重复的注解被包装到了容器注解中。在这种情况下，应该调用`getAnnotationsByType()`，该方法返回重复注解的数组。使用这个方法就不用操心容器注解了。

```java
@TestCase(params = "Hello", expected = "hello")
public String toLower(String s) { ... }

@TestCase(params = "Hello", expected = "HELLO")
@TestCase(params = "world", expected = "WORLD")
public String toUpper(String s) { ... }
```

```java
Method lower = cl.getMethod("toLower", String.class);
TestCase lowerTestCase = lower.getAnnotation(TestCase.class); // not null
TestCase[] lowerTestCases = lower.getAnnotationsByType(TestCase.class); // length = 1

Method upper = cl.getMethod("toUpper", String.class);
TestCase upperTestCase = upper.getAnnotation(TestCase.class); // null
TestCase[] upperTestCases = upper.getAnnotationsByType(TestCase.class); // length = 2
```

## 8.6 源码级注解处理
注解的另一种用法是自动处理源文件以产生源代码、配置文件、脚本或任何其他你想生成的东西。

### 8.6.1 注解处理器
注解处理已经集成到了Java编译器中。在编译时，可以通过运行以下命令来调用注解处理器：

```shell
javac -processor ProcessorClassName1,ProcessorClassName2,... sourceFiles
```

编译器会定位源文件中的注解。每个注解处理器依次执行，并得到它感兴趣的注解。如果某个注解处理器生成了新的源文件，则重复上述过程。一旦一轮处理没有产生更多源文件，就会编译所有源文件。

注释：注解处理器只能生成新的源文件，不能修改已有的源文件。

注解处理器需要实现`javax.annotation.processing.Processor`接口，一般是通过扩展`AbstractProcessor`类。需要指定处理器支持的注解，例如：

```java
@SupportedAnnotationTypes("com.horstmann.annotations.ToString")
@SupportedSourceVersion(SourceVersion.RELEASE_8)
public class ToStringAnnotationProcessor extends AbstractProcessor {
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment currentRound) {
        ...
    }
}
```

处理器可以声明具体注解类型、通配符（例如`"com.horstmann.*"`表示`com.horstmann`包及其子包中的所有注解），或者`"*"`（所有注解）。

`process()`方法每轮被调用一次，参数为这一轮在所有文件中发现的所有注解构成的集，以及包含有关当前处理轮次信息的对象。

### 8.6.2 语言模型API
使用**语言模型**(language model) API来分析源码级注解。与呈现类和方法的虚拟机表示的反射API不同，语言API可以根据Java语法规则分析Java程序（注：类似于编译器的语法分析）。

编译器会产生一颗树（语法分析树），其节点是实现了`javax.lang.model.element.Element`接口及其`TypeElement`、`VariableElement`和`ExecutableElement`等子接口的类的实例。这些节点类比于`Class`、`Field/Parameter`和`Method/Constructor`反射类。

本节不会详细介绍该API，以下是处理注解需要了解的要点：
* `RoundEnvironment`接口的`getElementsAnnotatedWith()`方法返回具有特定注解的所有元素构成的集。
* 源码级别上等价于`AnnotatedElement`接口的是`AnnotatedConstruct`。同样可以使用`getAnnotation()`和`getAnnotationsByType()`方法获得属于给定类型的注解或重复注解。
* `TypeElement`表示类或接口，`getEnclosedElements()`方法产生其字段和方法构成的列表。
* 调用`Element.getSimpleName()`或`TypeElement.getQualifiedName()`会产生一个`Name`对象，可以用`toString()`转换为字符串。

### 8.6.3 使用注解生成源码
作为示例，本节将使用注解来减少实现`toString()`方法时枯燥的工作量。

不能将这些方法放到原来的类中，因为注解处理器不能修改已有的类。因此，将所有自动生成的方法添加到工具类`ToStrings`中（这个类也是生成的）：

```java
public class ToStrings {
    public static String toString(Point obj) {
        // Generated code
    }
    public static String toString(Rectangle obj) {
        // Generated code
    }
    ...
    public static String toString(Object obj) {
        return Objects.toString(obj);
    }
}
```

我们不想使用反射，因此注解访问器方法而不是字段：

```java
@ToString
public class Rectangle {
    ...
    @ToString(includeName = false) public Point getTopLeft() { return topLeft; }
    @ToString public int getWidth() { return width; }
    @ToString public int getHeight() { return height; }
}
```

然后注解处理器应该生成下面的源代码（除了类名和字段名外都是“样板”代码）：

```java
public static String toString(Rectangle obj) {
    var result = new StringBuilder();
    result.append("Rectangle");
    result.append("[");
    result.append(toString(obj.getTopLeft()));
    result.append(",");
    result.append("width=");
    result.append(toString(obj.getWidth()));
    result.append(",");
    result.append("height=");
    result.append(toString(obj.getHeight()));
    result.append("]");
    return result.toString();
}
```

下面是为给定的类生成`toString()`方法的框架：

```java
private void writeToStringMethod(PrintWriter out, TypeElement te) {
    String className = te.getQualifiedName().toString();
    // Print method header and declaration of string builder
    ToString ann = te.getAnnotation(ToString.class);
    if (ann.includeName())
        // Print code to add class name
    for (Element c : te.getEnclosedElements()) {
        ann = c.getAnnotation(ToString.class);
        if (ann != null) {
            if (ann.includeName()) // Print code to add field name
            // Print code to append toString(obj.methodName())
        }
    }
    // Print code to return string
}
```

下面是注解处理器的`process()`方法的框架。它会创建工具类`ToStrings`的源文件，并为每个被注解的类创建一个静态`toString()`方法。

```java
public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment currentRound) {
    if (annotations.size() == 0) return true;
    try {
        JavaFileObject sourceFile = processingEnv.getFiler().createSourceFile(
            "com.horstmann.annotations.ToStrings");
        try (var out = new PrintWriter(sourceFile.openWriter())) {
            // Print code for package and class
            for (Element e : currentRound.getElementsAnnotatedWith(ToString.class))
                if (e instanceof TypeElement te)
                    writeToStringMethod(out, te);
            // Print code for toString(Object)
        }
    }
    catch (IOException e) {
        processingEnv.getMessager().printMessage(Kind.ERROR, ex.getMessage());
    }
    return true;
}
```

注意，`process()`方法在后续轮次中是用空列表调用的，然后它会立即返回，不会再次创建源文件。

[sourceAnnotations/ToStringAnnotationProcessor.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch08/sourceAnnotations/ToStringAnnotationProcessor.java)

[rect/SourceLevelAnnotationDemo.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch08/rect/SourceLevelAnnotationDemo.java)

首先编译注解处理器，然后编译并运行测试程序：

```shell
javac sourceAnnotations/ToStringAnnotationProcessor.java
javac -processor sourceAnnotations.ToStringAnnotationProcessor rect/*.java
java rect.SourceLevelAnnotationDemo
```

提示：要查看轮次，可以用`-XprintRounds`选项运行`javac`命令。

除了源文件，注解处理器还可以生成XML描述符、属性文件、shell脚本、HTML文档等。

注释：有人建议使用注解来自动生成getter和setter。例如：

```java
@Property private String title;
```

可以产生方法

```java
public String getTitle() { return title; }
public void setTitle(String title) { this.title = title; }
```

但是，这需要编辑源文件而不只是生成另一个文件，这超出了注解处理器的能力范围。注解旨在作为对代码项的**描述**，而不是添加或修改代码的指令。

## 8.7 字节码工程
除了运行时和源码级别，还可以在字节码级别处理注解。除非注解在源码级别被删除，否则就会出现在类文件中。类文件的格式相当复杂（参见[Java虚拟机规范](https://docs.oracle.com/javase/specs/jvms/se17/html/index.html)第4章），在没有特殊库的情况下处理类文件具有很大的挑战性。ASM库就是这种库之一，可以从 <https://asm.ow2.io/> 获得。下载asm-9.2.jar和asm-commons-9.2.jar并将其放到某个目录中（下面将其称为$asm）。

### 8.7.1 修改类文件
在本节中，使用ASM为注解的方法添加日志消息。如果一个方法具有注解

```java
@LogEntry(logger = loggerName)
```

则在方法开头添加以下语句的字节码：

```java
Logger.getLogger(loggerName).entering(className, methodName);
```

例如，如果对`Item`类的`hashCode()`方法添加了注解`@LogEntry(logger = "global")`，那么每当调用该方法时就会打印类似于下面的消息：

```
May 17, 2016 10:57:59 AM Item hashCode
FINER: ENTRY
```

为此，需要执行以下步骤：
1. 加载类文件中的字节码。
2. 定位所有方法。
3. 对于每个方法，检查它是否具有`LogEntry`注解。
4. 如果有，在方法开头添加下列指令的字节码：

```
ldc loggerName
invokestatic java/util/logging/Logger.getLogger:(Ljava/lang/String;)Ljava/util/logging/Logger;
ldc className
ldc methodName
invokevirtual java/util/logging/Logger.entering:(Ljava/lang/String;Ljava/lang/String;)V
```

这看起来很棘手，不过ASM使它变得相当简单。程序清单8-9中的程序可以编辑一个类文件，并在具有`LogEntry`注解的方法开头插入日志调用。例如，可以像这样向程序清单8-10中的Item.java添加日志指令（其中$asm是ASM库的安装目录）：

```shell
javac set/Item.java
javac -classpath .:$asm/\* bytecodeAnnotations/EntryLogger.java
java -classpath .:$asm/\* bytecodeAnnotations.EntryLogger set/Item.class
```

在修改`Item`类文件前后分别运行一下`javap -c set.Item`，可以看到在`hashCode()`、`equals()`和`compareTo()`方法开头插入的指令。

```
public int hashCode();
  Code:
     0: ldc           #42    // String com.horstmann
     2: invokestatic  #48    // Method java/util/logging/Logger.getLogger:(Ljava/lang/String;)Ljava/util/logging/Logger;
     5: ldc           #50    // String set.Item
     7: ldc           #67    // String hashCode
     9: invokevirtual #55    // Method java/util/logging/Logger.entering:(Ljava/lang/String;Ljava/lang/String;)V
    12: iconst_2
    13: anewarray     #4     // class java/lang/Object
    ...
```

程序清单8-11中的`SetTest`程序将`Item`对象插入到散列集中。当使用修改过的类文件来运行该程序时，将会看到日志消息：

```
May 17, 2016 10:57:59 AM Item hashCode
FINER: ENTRY
May 17, 2016 10:57:59 AM Item hashCode
FINER: ENTRY
May 17, 2016 10:57:59 AM Item hashCode
FINER: ENTRY
May 17, 2016 10:57:59 AM Item equals
FINER: ENTRY
[[description=Toaster, partNumber=1729], [description=Microwave, partNumber=4104]]
```

[程序清单8-9 bytecodeAnnotations/EntryLogger.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch08/bytecodeAnnotations/EntryLogger.java)

[程序清单8-10 set/Item.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch08/set/Item.java)

[程序清单8-11 set/SetTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch08/set/SetTest.java)

注意：必须按以下顺序编译和运行：
1. 编译`Item`类
2. 编译`EntryLogger`类
3. 运行`EntryLogger`类，修改Item.class
4. 编译`SetTest`
5. 运行`SetTest`程序

### 8.7.2 在加载时修改字节码
上一节介绍了编辑类文件的工具。一种有吸引力的替代方案是将字节码工程推迟到**加载时**，即类加载器加载类的时候。

Java **instrumentation API** 提供了一个安装字节码转换器的挂钩(hook)。转换器必须在程序的`main()`方法被调用前安装，可以通过定义一个**代理**(agent)（以某种方式监视程序的库）来满足这一要求。代理代码可以在`premain()`方法中执行初始化。

下面是构建代理所需的步骤：

1.实现一个具有以下方法的类。当代理加载时该方法会被调用。代理可以获取单个命令行参数，通过`arg`参数传递。`instr`参数可以用来安装各种挂钩。

```java
public static void premain(String arg, Instrumentation instr)
```

2.制作一个清单文件（例如EntryLoggingAgent.mf），设置`Premain-Class`。例如：

```
Premain-Class: bytecodeAnnotations.EntryLoggingAgent
```

3.将代理代码和清单打包成一个JAR文件：

```shell
javac -classpath .:$asm/\* bytecodeAnnotations/EntryLoggingAgent.java
jar cvfm EntryLoggingAgent.jar bytecodeAnnotations/EntryLoggingAgent.mf bytecodeAnnotations/Entry*.class
```

为了与代理一起启动Java程序，使用以下命令：

```shell
java -javaagent:AgentJARFile=agentArgument ...
```

例如，运行具有入口日志代理的`SetTest`程序：

```shell
javac set/SetTest.java
java -javaagent:EntryLoggingAgent.jar=set.Item -classpath .:$asm/\* set.SetTest
```

其中`set.Item`参数是代理应该修改的类名。

程序清单8-12展示了这个代理的代码。代理安装了一个类文件转换器，这个转换器首先检查类名是否与代理参数匹配。如果匹配，则使用上一节的`EntryLogger`类修改字节码。不过，修改后的字节码没有保存到文件，而是转换器将其返回以加载到虚拟机中（见下图）。换句话说，这项技术对字节码进行“即时”(just in time)修改。

![在加载时修改类](/assets/images/java-note-v2ch08-scripting-compiling-and-annotation-processing/在加载时修改类.png)

[程序清单8-12 bytecodeAnnotations/EntryLoggingAgent.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch08/bytecodeAnnotations/EntryLoggingAgent.java)
