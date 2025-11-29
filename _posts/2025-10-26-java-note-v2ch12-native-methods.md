---
title: 《Java核心技术》笔记 卷II 第12章 本地方法
date: 2025-10-26 11:09:43 +0800
categories: [Java, Core Java]
tags: [java, native method, jni]
---
虽然“100%纯Java”的解决方案原则上是不错的，但某些情况下你也会想要编写（或使用）其他语言的代码。这种代码通常称为**本地代码**(native code)。

建议只有在必要时才使用本地代码，特别是在以下三种情况下：
* 你的应用需要访问通过Java平台无法访问的系统特性或设备。
* 你已经有大量测试过和调试过的用另一种语言编写的代码，并且知道如何将其移植到所有需要的目标平台。
* 你通过基准测试发现Java代码比用另一种语言编写的等价代码要慢得多。

Java平台有一个用于和本地C代码互操作的API，称为**Java本地接口**(Java Native Interface, JNI)。本章将讨论JNI编程。

注：JNI规范：<https://docs.oracle.com/en/java/javase/17/docs/specs/jni/index.html>

C++注释：也可以使用C++而不是C来编写本地方法。但是，JNI并不支持Java类和C++类之间的任何映射。

在Java和本地代码之间提供绑定层在一定程度上显得乏味冗长。Java 17（作为预览特性）提供了一个可以访问“外部”函数和内存的API，比JNI要方便许多。

## 12.1 从Java程序中调用C函数
假设你有一个C函数，并且不想费事用Java重新实现它，可以在Java中通过**本地方法**(native method)调用该函数。

Java使用关键字`native`表示本地方法。当然，本地方法不包含任何Java代码，方法声明后面紧跟着一个分号（看起来类似于抽象方法声明）。例如：

```java
class HelloNative {
    public static native void greeting();
}
```

[程序清单12-1 helloNative/HelloNative.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch12/helloNative/HelloNative.java)

本地方法可以是静态的也可以是非静态的。

实际上你可以编译这个类，但如果在程序中使用它，虚拟机会报错`UnsatisfiedLinkError`——无法找到`greeting()`方法。为了实现本地方法，需要编写相应的C函数。必需**完全**按照Java虚拟机期望的方式命名该函数。规则如下：
1. 使用完整的Java方法名（包名+类名+方法名），例如`HelloNative.greeting`或`com.horstmann.HelloNative.greeting`。
2. 将所有`.`替换为下划线，并加上`Java_`前缀。例如`Java_HelloNative_greeting`或`Java_com_horstmann_HelloNative_greeting`。
3. 如果类名包含非ASCII字母或数字的字符，将其替换为`_0xxxx`（其中`xxxx`是字符Unicode码点的四位十六进制数）。

注释：如果重载了本地方法，必须在名字后添加两个下划线，后面跟着编码的参数类型。例如，如果有两个本地方法`greeting()`和`greeting(int)`，那么第一个叫做`Java_HelloNative_greeting__`，第二个叫做`Java_HelloNative_greeting__I`。

实际上，没人会手工完成这些操作。而应该运行`javac -h`，并指定放置头文件的目录：

```shell
javac -h . HelloNative.java
```

这条命令在当前目录中创建了头文件HelloNative.h，如程序清单12-2所示。

注：旧版本JDK使用`javah`工具生成头文件，在Java 9中被删除。

[程序清单12-2 helloNative/HelloNative.h](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch12/helloNative/HelloNative.h)

这个文件包含函数`Java_HelloNative_greeting()`的声明（宏`JNIEXPORT`和`JNICALL`定义在头文件jni.h中）。

```c
JNIEXPORT void JNICALL Java_HelloNative_greeting(JNIEnv *, jclass);
```

现在将函数原型复制到源文件中，并实现该函数，如程序清单12-3所示。

[程序清单12-3 helloNative/HelloNative.c](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch12/helloNative/HelloNative.c)

C++注释：你可以使用C++实现本地方法，但是必须将实现函数声明为`extern "C"`（阻止C++编译器“改编”(mangling)函数名）。例如：

```cpp
extern "C"
JNIEXPORT void JNICALL Java_HelloNative_greeting(JNIEnv* env, jclass cl) {
    cout << "Hello, Native World!" << endl;
}
```

将本地C代码编译成动态加载库，细节取决于编译器。例如，对于Linux上的GNU C编译器，使用以下命令：

```shell
gcc -fPIC -I jdk/include -I jdk/include/linux -shared -o libHelloNative.so HelloNative.c
```

对于macOS上的Clang编译器，命令是：

```shell
gcc -dynamiclib -I jdk/include -I jdk/include/darwin -o libHelloNative.dylib HelloNative.c
```

对于Windows上的MSVC编译器，命令是：

```shell
cl /I jdk\include /I jdk\include\win32 /LD HelloNative.c /FeHelloNative.dll
```

其中`jdk`是JDK安装目录（通常是`JAVA_HOME`环境变量指向的目录）。

提示：如果要从命令行中使用MSVC编译器，首先要运行批处理文件vcvars32.bat或vcvarsall.bat，该文件设置了编译器需要的路径和环境变量。可以在C:\Program Files\Microsoft Visual Studio 14.0\Common7\Tools目录或类似位置找到该文件，详见[Visual Studio文档](https://learn.microsoft.com/en-us/cpp/build/building-on-the-command-line?view=msvc-170)。

也可以使用Cygwin编程环境，可以从 <https://www.cygwin.com/> 免费获取。它包含GNU C编译器和用于Windows上UNIX风格编程的库。对于Cygwin，使用以下命令：

```shell
gcc -mno-cygwin -D __int64="long long" -I jdk/include/ -I jdk/include/win32 \
  -shared -Wl,--add-stdcall-alias -shared -o HelloNative.dll HelloNative.c
```

最后，在Java程序中添加一个`System.loadLibrary()`方法的调用。为了确保虚拟机在第一次使用该类之前就加载这个库，需要使用静态初始化块，如程序清单12-4所示。

[程序清单12-4 helloNative/HelloNativeTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch12/helloNative/HelloNativeTest.java)

注：
* `System.loadLibrary("foo")`在不同平台上查找的库文件名也不同：
  * Linux: `libfoo.so`
  * macOS: `libfoo.dylib`
  * Windows: `foo.dll`
* 这个映射操作由本地方法`System.mapLibraryName()`完成，实现代码见[System.c](https://github.com/openjdk/jdk/blob/jdk-17-ga/src/java.base/share/native/libjava/System.c#L293)。库文件名的前缀和后缀由`JNI_LIB_PREFIX`和`JNI_LIB_SUFFIX`这两个宏指定，其定义在[posix/jvm_md.h](https://github.com/openjdk/jdk/blob/jdk-17-ga/src/hotspot/os/posix/include/jvm_md.h#L47)和[windows/jvm_md.h](https://github.com/openjdk/jdk/blob/jdk-17-ga/src/hotspot/os/windows/include/jvm_md.h#L49)。

下图给出了本地代码处理的总结。

![处理本地代码](/assets/images/java-note-v2ch12-native-methods/处理本地代码.png)

编译并运行该程序，终端窗口会显示消息 "Hello, Native World!" 。

注释：在Linux上，必须把当前目录添加到库路径。可以设置`LD_LIBRARY_PATH`环境变量：

```shell
export LD_LIBRARY_PATH=.:$LD_LIBRARY_PATH
```

或者设置`java.library.path`系统属性：

```shell
java -Djava.library.path=. HelloNativeTest
```

总之，按照以下步骤将本地方法链接到Java程序：
1. 在Java类中声明本地方法。
2. 运行`javac -h`生成头文件。
3. 用C实现本地方法。
4. 将C代码编译成动态加载库（共享库）。
5. 在Java程序中加载该库。

注释：有些本地代码的共享库必须先执行初始化。可以把初始化代码放到`JNI_OnLoad()`函数中，该函数返回它需要的虚拟机最低版本（如`JNI_VERSION_1_2`）。类似地，当虚拟机关闭时，将会调用`JNI_OnUnload()`函数。其原型是：

```c
jint JNI_OnLoad(JavaVM* vm, void* reserved);
void JNI_OnUnload(JavaVM* vm, void* reserved);
```

## 12.2 数值参数和返回值
C语言`int`类型的大小是平台相关的，在一些平台上是16位，在另一些平台上是32位。而Java的`int`类型始终是32位。因此，JNI定义了`jint`、`jlong`等类型。

下表显示了Java类型和C类型的对应关系。

| Java类型 | C类型 | 字节 |
| --- | --- | --- |
| `boolean` | `jboolean` | 1 |
| `byte` | `jbyte` | 1 |
| `char` | `jchar` | 2 |
| `short` | `jshort` | 2 |
| `int` | `jint` | 4 |
| `long` | `jlong` | 8 |
| `float` | `jfloat` | 4  |
| `double` | `jdouble` | 8 |

在头文件jni.h中，这些类型使用`typedef`声明为目标平台上的等价类型。该头文件还定义了常量`JNI_FALSE = 0`和`JNI_TRUE = 1`。

Java 5之前没有与C语言的`printf()`类似的函数。在下面的示例中，假设你使用古老版本的JDK，并在本地方法中调用C语言的`printf()`函数来实现同样的功能。

程序清单12-5中的类`Printf1`使用本地方法来打印给定字段宽度和精度的浮点数。

[程序清单12-5 printf1/Printf1.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch12/printf1/Printf1.java)

注意，用C实现该方法时，所有`int`和`double`参数都变成了`jint`和`jdouble`，如程序清单12-6所示。

[程序清单12-6 printf1/Printf1.c](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch12/printf1/Printf1.c)

该函数在变量`fmt`中组装了格式字符串`"%w.pf"`，然后调用`printf()`，返回打印的字符个数。

程序清单12-7给出了测试程序`Printf1Test`。

[程序清单12-7 printf1/Printf1Test.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch12/printf1/Printf1Test.java)

## 12.3 字符串参数
接下来考虑如何把字符串传入、传出本地方法。字符串在这两种语言中很不一样：Java字符串是UTF-16码元的序列，而C字符串是以null结尾的字节序列。JNI有两组操作字符串的函数：一组把Java字符串转换成“修改版UTF-8”字节序列，另一组将其转换成UTF-16值（即`jchar`）的数组。（注：码元的概念参见卷I第3章 3.3.4节，“修改版UTF-8”参见卷II第2章 2.2.1节）

如果C代码已经使用了Unicode，那么可以使用第二组转换函数。另外，如果字符串都仅限于ASCII字符，就可以使用“修改版UTF-8”转换函数。

带有`String`参数的本地方法实际上会接收一个`jstring`类型的值，而具有`String`返回值的本地方法必须返回一个`jstring`类型的值。JNI函数将读取并构造`jstring`对象。例如，`NewStringUTF()`函数从`char`数组创建一个新的`jstring`对象：

```c
JNIEXPORT jstring JNICALL Java_HelloNative_getGreeting(JNIEnv* env, jclass cl) {
    jstring jstr;
    char greeting[] = "Hello, Native World\n";
    jstr = (*env)->NewStringUTF(env, greeting);
    return jstr;
}
```

所有对JNI函数的调用都使用了`env`指针，该指针是每个本地方法的第一个参数。`env`是指向函数指针表的指针（见下图）。因此，必须在每个JNI调用前面加上`(*env)->`。

![env指针](/assets/images/java-note-v2ch12-native-methods/env指针.png)

C++注释：在C++中访问JNI函数要简单一些。C++版本的`JNIEnv`类有内联成员函数负责帮你查找函数指针。例如，可以这样调用`NewStringUTF()`函数：

```cpp
jstr = env->NewStringUTF(greeting);
```

注意，这里省略了参数列表中的`env`指针。

要读取`jstring`对象的内容，使用`GetStringUTFChars()`函数。该函数返回指向“修改版UTF-8”字符的`const jbyte*`指针。

虚拟机必须知道你何时使用完字符串，以便进行垃圾回收。因此，你必须调用`ReleaseStringUTFChars()`函数。

最后，`GetStringUTFLength()`函数返回字符串的“修改版UTF-8”编码所需的字符个数。

注释：JNI字符串操作函数的完整列表参见[JNI规范 - String Operations](https://docs.oracle.com/en/java/javase/17/docs/specs/jni/functions.html#string-operations)。

下面使用这些函数来编写一个调用C语言的`sprintf()`函数的类，如程序清单12-8所示。

[程序清单12-8 printf2/Printf2Test.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch12/printf2/Printf2Test.java)

程序清单12-9展示了具有本地方法`sprint()`的类。

[程序清单12-9 printf2/Printf2.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch12/printf2/Printf2.java)

格式化浮点数的C函数原型如下：

```c
JNIEXPORT jstring JNICALL Java_Printf2_sprint(JNIEnv* env, jclass cl, jstring format, jdouble x)
```

程序清单12-10给出了C语言实现代码。

[程序清单12-10 printf2/Printf2.c](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch12/printf2/Printf2.c)

在这个函数中，如果格式字符串不是`%w.pc`的形式（其中`c`是`e`, `E`, `f`, `g`或`G`之一），则不对数字进行格式化。

## 12.4 访问字段
到目前为止看到的所有本地方法都是静态方法。下面考虑操作对象的本地方法。

### 12.4.1 访问实例字段
为了了解如何从本地方法中访问实例字段，我们将重新实现卷I第4章中的`raiseSalary()`方法。其Java代码如下：

```java
public void raiseSalary(double byPercent) {
    salary *= 1 + byPercent / 100;
}
```

下面将其重写为本地方法。与前面的示例不同，这不是静态方法，其原型如下：

```c
JNIEXPORT void JNICALL Java_Employee_raiseSalary(JNIEnv *, jobject, jdouble);
```

注意第二个参数不再是`jclass`类型，而是`jobject`类型。实际上，它等价于`this`引用。静态方法得到的是类的引用，而非静态方法得到的是隐式参数`this`的引用。

为了避免虚拟机暴露其内部数据布局，JNI要求程序员通过调用特殊的JNI函数来获取和设置实例字段的值。在这个例子中，需要使用`GetDoubleField()`和`SetDoubleField()`函数，因为`salary`是`double`类型。对于其他字段类型，有`GetIntField`/`SetIntField`, `GetObjectField`/`SetObjectField`等函数。（注：访问对象字段的JNI函数完整列表参见[JNI规范 - Accessing Fields of Objects](https://docs.oracle.com/en/java/javase/17/docs/specs/jni/functions.html#accessing-fields-of-objects)）

一般语法是：

```c
x = (*env)->GetXxxField(env, this_obj, field_id);
(*env)->SetXxxField(env, this_obj, field_id, x);
```

其中，`field_id`是一个特殊类型`jfieldID`的值，标识结构中的一个字段，`Xxx`代表Java数据类型（`Object`、`Int`、`Double`等）。为了获得字段ID，必须先获得一个表示类的值，有两种方式。`GetObjectClass()`函数返回任意对象的类。例如：

```c
jclass class_Employee = (*env)->GetObjectClass(env, this_obj);
```

`FindClass()`函数允许你以字符串形式指定类名（奇怪的是，用`/`而不是`.`作为包名分隔符）：

```c
jclass class_String = (*env)->FindClass(env, "java/lang/String");
```

之后使用`GetFieldID()`函数来获得字段ID，必须提供字段的名字和**签名**（其类型的编码）。例如，下面是获得`salary`字段ID的代码：

```c
jfieldID id_salary = (*env)->GetFieldID(env, class_Employee, "salary", "D");
```

字符串`"D"`表示`double`类型。下一节将介绍编码签名的完整规则。

汇总起来，下面的代码以本地方法的形式重新实现了`raiseSalary()`方法：

```c
JNIEXPORT void JNICALL Java_Employee_raiseSalary(JNIEnv* env, jobject this_obj, jdouble byPercent) {
    /* get the class */
    jclass class_Employee = (*env)->GetObjectClass(env, this_obj);

    /* get the field ID */
    jfieldID id_salary = (*env)->GetFieldID(env, class_Employee, "salary", "D");

    /* get the field value */
    jdouble salary = (*env)->GetDoubleField(env, this_obj, id_salary);

    salary *= 1 + byPercent / 100;

    /* set the field value */
    (*env)->SetDoubleField(env, this_obj, id_salary, salary);
}
```

警告：类引用只在本地方法返回之前有效。**不要**在代码中缓存`GetObjectClass()`的返回值供后续方法调用重复使用，必须在每次执行本地方法时都调用`GetObjectClass()`。如果无法接受，可以调用`NewGlobalRef()`来锁定该引用：

```c
static jclass class_X = 0;
static jfieldID id_a;
...
if (class_X == 0) {
    jclass cx = (*env)->GetObjectClass(env, obj);
    class_X = (*env)->NewGlobalRef(env, cx);
    id_a = (*env)->GetFieldID(env, class_X, "a", "...");
}
```

现在可以在后续调用中使用类引用和字段ID了。当使用结束时，务必调用`(*env)->DeleteGlobalRef(env, class_X);`

程序清单12-11和12-12给出了测试程序和`Employee`类的Java代码。程序清单12-13包含本地方法`raiseSalary()`的C代码。

[程序清单12-11 employee/EmployeeTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch12/employee/EmployeeTest.java)

[程序清单12-12 employee/Employee.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch12/employee/Employee.java)

[程序清单12-13 employee/Employee.c](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch12/employee/Employee.c)

### 12.4.2 访问静态字段
访问静态字段要使用`GetStaticFieldID()`和`GetStaticXxxField`/`SetStaticXxxField`函数。它们几乎与非静态的情形相同，只有两个区别：
* 由于没有对象，必须使用`FindClass()`而不是`GetObjectClass()`来获得类引用。
* 访问字段时必须提供类而不是实例对象。

例如，可以像这样得到`System.out`的引用：

```c
/* get the class */
jclass class_System = (*env)->FindClass(env, "java/lang/System");

/* get the field ID */
jfieldID id_out = (*env)->GetStaticFieldID(env, class_System, "out", "Ljava/io/PrintStream;");

/* get the field value */
jobject obj_out = (*env)->GetStaticObjectField(env, class_System, id_out);
```

## 12.5 编码签名
为了访问实例字段和调用Java方法，需要了解数据类型和方法签名的编码规则。

| 类型签名 | Java类型 |
| --- | --- |
| `Z` | `boolean` |
| `B` | `byte` |
| `C` | `char` |
| `S` | `short` |
| `I` | `int` |
| `J` | `long` |
| `F` | `float` |
| `D` | `double` |
| `V` | `void` |
| `Lclassname;` | 类 |
| `[type` | 数组 |
| `(arg-types) ret-type` | 方法 |

注：参见[JNI规范 - Type Signatures](https://docs.oracle.com/en/java/javase/17/docs/specs/jni/types.html#type-signatures)。

例如，`String[]`编码为`[Ljava/lang/String;`，`float[][]`编码为`[[F`。

方法签名在括号中列出参数类型，然后是返回类型。例如，接收两个整数、返回一个整数的方法签名编码为`(II)I`。12.3节中的`sprint()`方法的签名编码为`(Ljava/lang/String;D)Ljava/lang/String;`。

注意，`;`是类名的结束符，而不是参数之间的分隔符，参数之间没有分隔符。例如，构造器`Employee(String, double, java.util.Date)`的签名编码为`(Ljava/lang/String;DLjava/util/Date;)V`。

在签名中，必须用`/`而不是`.`来分隔包和类名。结尾的`V`表示返回类型为`void`。即使在Java中不指定构造器的返回类型，在签名中也需要添加`V`。

提示：可以使用`javap -s`命令从类文件中生成方法签名。例如，运行

```shell
javap -s -private Employee
```

将得到以下输出，显示所有字段和方法的签名：

```
Compiled from "Employee.java"
public class Employee {
  private java.lang.String name;
    descriptor: Ljava/lang/String;
  private double salary;
    descriptor: D
  public native void raiseSalary(double);
    descriptor: (D)V

  public Employee(java.lang.String, double);
    descriptor: (Ljava/lang/String;D)V

  public void print();
    descriptor: ()V

  static {};
    descriptor: ()V
}
```

## 12.6 调用Java方法
Java方法当然可以调用C函数，这正是本地方法的用途。反过来，有时需要在本地方法中调用Java方法。首先介绍如何调用实例方法，然后是静态方法。

### 12.6.1 实例方法
作为示例，我们给`Printf`类添加一个类似于C函数`fprintf()`的方法，能够向任意的`PrintWriter`对象打印字符串。下面是Java中本地方法的定义：

```java
class Printf3 {
    public native static void fprint(PrintWriter out, String s, double x);
    ...
}
```

首先把要打印的字符串组装成一个`String`对象，就像在已实现的`sprint()`方法中一样。然后调用`PrintWriter`类的`print()`方法。

可以使用以下函数从C中调用任何Java方法：

```c
(*env)->CallXxxMethod(env, implicit parameter, methodID, explicit parameters)
```

其中`Xxx`是方法的返回类型（`Void`、`Int`、`Object`等）。就像访问字段需要字段ID一样，调用方法需要方法ID。为了获得方法ID，需要调用JNI函数`GetMethodID()`，并提供类、方法名和方法签名。（注：调用实例方法的JNI函数完整列表参见[JNI规范 - Calling Instance Methods](https://docs.oracle.com/en/java/javase/17/docs/specs/jni/functions.html#calling-instance-methods)）

在这个例子中，我们想要获得`PrintWriter`类的`print()`方法的ID。该方法有多个重载，因此必须提供描述参数和返回类型的字符串（签名编码）。例如，我们想使用`void  print(String)`，其签名编码为`"(Ljava/lang/String;)V"`。

下面是进行方法调用的完整代码：

```c
/* get the class */
class_PrintWriter = (*env)->GetObjectClass(env, out);

/* get the method ID */
id_print = (*env)->GetMethodID(env, class_PrintWriter, "print", "(Ljava/lang/String;)V");

/* call the method */
(*env)->CallVoidMethod(env, out, id_print, str);
```

程序清单12-14和12-15给出了测试程序和`Printf3`类的Java代码。程序清单12-16包含本地方法`fprint()`的C代码。

[程序清单12-14 printf3/Printf3Test.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch12/printf3/Printf3Test.java)

[程序清单12-15 printf3/Printf3.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch12/printf3/Printf3.java)

[程序清单12-16 printf3/Printf3.c](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch12/printf3/Printf3.c)

注释：数值型的方法ID和字段ID在概念上类似于反射API中的`Method`和`Field`对象。可以使用以下函数在两者间进行转换：

```c
jobject ToReflectedMethod(JNIEnv* env, jclass cls, jmethodID methodID);  // returns Method object
jmethodID FromReflectedMethod(JNIEnv* env, jobject method);
jobject ToReflectedField(JNIEnv* env, jclass cls, jfieldID fieldID);  // returns Field object
jfieldID FromReflectedField(JNIEnv* env, jobject field);
```

### 12.6.2 静态方法
在本地方法中调用静态方法与调用实例方法类似。有两个区别：
* 要使用`GetStaticMethodID()`和`CallStaticXxxMethod()`函数。
* 调用方法时要提供类对象，而不是隐式参数对象。

例如，假设要在本地方法中调用静态方法`System.getProperty("java.class.path")`。首先，要找到`System`类的对象：

```c
jclass class_System = (*env)->FindClass(env, "java/lang/System");
```

接着，获取静态方法`getProperty()`的ID：

```c
jmethodID id_getProperty = (*env)->GetStaticMethodID(env, class_System, "getProperty",
    "(Ljava/lang/String;)Ljava/lang/String;");
```

最后，进行调用：

```c
jobject obj_ret = (*env)->CallStaticObjectMethod(env, class_System, id_getProperty,
    (*env)->NewStringUTF(env, "java.class.path"));
```

该方法的返回值是`jobject`类型。如果想把它当作字符串操作，必须将其强制转换为`jstring`：

```c
jstring str_ret = (jstring) obj_ret;
```

C++注释：在C中，`jstring`和`jclass`以及后面要介绍的数组类型都是与`jobject`等价的类型。因此在C中，前面例子中的类型转换并不是严格必需的。但是在C++中，这些类型被定义为拥有正确继承层次关系的“哑类”(dummy class)的指针。例如，在C++中将`jstring`不经转换赋给`jobject`是合法的，但是将`jobject`赋给`jstring`需要强制类型转换。

注：
* 在jni.h中可以看到这些类型的定义如下。

```cpp
#ifdef __cplusplus

class _jobject {};
class _jclass : public _jobject {};
class _jstring : public _jobject {};
class _jarray : public _jobject {};
class _jbooleanArray : public _jarray {};
...

typedef _jobject *jobject;
typedef _jclass *jclass;
typedef _jstring *jstring;
typedef _jarray *jarray;
typedef _jbooleanArray *jbooleanArray;
...

#else

struct _jobject;

typedef struct _jobject *jobject;
typedef jobject jclass;
typedef jobject jstring;
typedef jobject jarray;
typedef jarray jbooleanArray;
...

#endif
```

* `_jobject`等“哑类”没有实际作用，`jobject`是指向JVM内部对象结构的指针（可以理解为C/C++中的`void*`）。必须使用JNI函数来与这个对象进行交互，而不能直接操作它指向的内存。

### 12.6.3 构造器
本地方法可以通过调用构造器来创建新的Java对象。通过`NewObject()`函数来调用构造器：

```c
jobject obj_new = (*env)->NewObject(env, class, methodID, construction parameters);
```

可以通过`GetMethodID()`函数获取构造器的方法ID，将方法名指定为`"<init>"`，并指定构造器的签名编码（返回类型为`void`）。例如，本地方法可以像这样创建一个`FileOutputStream`对象：

```c
const char[] fileName = "...";
jstring str_fileName = (*env)->NewStringUTF(env, fileName);
jclass class_FileOutputStream = (*env)->FindClass(env, "java/io/FileOutputStream");
jmethodID id_FileOutputStream
    = (*env)->GetMethodID(env, class_FileOutputStream, "<init>", "(Ljava/lang/String;)V");
jobject obj_stream
    = (*env)->NewObject(env, class_FileOutputStream, id_FileOutputStream, str_fileName);
```

### 12.6.4 另一种方法调用
`CallNonvirtualXxxMethod()`函数接收隐式参数、方法ID、类对象（必需对应隐式参数的超类）和显式参数。该函数调用指定类中的方法版本，绕过常规的动态绑定（多态）机制。

所有调用函数都有带后缀 "A" 和 "V" 的版本，分别用数组和`va_list`（定义在C头文件stdarg.h中）接收显式参数。

## 12.7 访问数组元素
Java的所有数组类型都有对应的C类型，如下表所示。

| Java类型 | C类型 |
| --- | --- |
| `boolean[]` | `jbooleanArray` |
| `byte[]` | `jbyteArray` |
| `char[]` | `jcharArray` |
| `short[]` | `jshortArray` |
| `int[]` | `jintArray` |
| `long[]` | `jlongArray` |
| `float[]` | `jfloatArray` |
| `double[]` | `jdoubleArray` |
| `Object[]` | `jobjectArray` |

C++注释：在C中，所有数组类型实际上都是`jobject`的别名。而在C++中，它们被安排在如下图所示的继承层次结构中。`jarray`类型表示泛型数组。

![数组类型的继承层次结构](/assets/images/java-note-v2ch12-native-methods/数组类型的继承层次结构.png)

`GetArrayLength()`函数返回数组的长度。

```c
jarray array = ...;
jsize length = (*env)->GetArrayLength(env, array);
```

访问数组元素的方式取决于数组存储的是对象还是基本类型的值。使用`GetObjectArrayElement`/`SetObjectArrayElement`函数访问对象数组的元素。

```c
jobjectArray array = ...;
int i, j;
jobject x = (*env)->GetObjectArrayElement(env, array, i);
(*env)->SetObjectArrayElement(env, array, j, x);
```

这种方式虽然简单，但明显很低效。有时希望能够直接访问数组元素，特别是在进行向量和矩阵计算时。

`GetXxxArrayElements()`函数返回指向数组起始元素的指针（其中`Xxx`必须是基本类型，即不是`Object`）。与普通字符串一样，必须记得调用`ReleaseXxxArrayElements()`函数通知虚拟机不再需要该指针。这样就可以直接读写数组元素了。但是，由于指针**可能会指向一个副本**，因此只有在调用相应的`ReleaseXxxArrayElements()`函数后，才能保证所做的改变反映在原数组中。

注释：通过把一个指向`jboolean`变量的指针作为第三个参数传递给`GetXxxArrayElements()`，可以查看数组是不是副本。如果是副本，则该变量被置为`JNI_TRUE`。如果对这个信息不感兴趣，传递空指针即可。

下面的示例将一个`double`数组中的所有元素乘以一个常量。首先获取首元素的指针`a`，并用`a[i]`访问各个元素。

```c
jdoubleArray array_a = ...;
double scaleFactor = ...;
double* a = (*env)->GetDoubleArrayElements(env, array_a, NULL);
jsize i;
for (i = 0; i < (*env)->GetArrayLength(env, array_a); i++)
    a[i] *= scaleFactor;
(*env)->ReleaseDoubleArrayElements(env, array_a, a, 0);
```

[array/ArrayAlgTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch12/array/ArrayAlgTest.java)

[array/ArrayAlg.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch12/array/ArrayAlg.java)

[array/ArrayAlg.c](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch12/array/ArrayAlg.c)

要访问一个大数组中的少数几个元素，使用`GetXxxArrayRegion`/`SetXxxArrayRegion`函数，它将一个范围内的元素从Java数组复制到C数组或反过来。

可以用`NewXxxArray()`函数创建新的Java数组。要创建新的对象数组，需要指定长度、元素类型和**所有**元素的初始值（通常是`NULL`）。下面是一个例子：

```c
jclass class_Employee = (*env)->FindClass(env, "Employee");
jobjectArray array_e = (*env)->NewObjectArray(env, 100, class_Employee, NULL);
```

创建基本类型的数组更简单，只需提供数组长度。数组用0填充。

```c
jdoubleArray array_d = (*env)->NewDoubleArray(env, 100);
```

## 12.8 错误处理
本地方法对Java程序来说是一个重大的安全风险。当本地方法遇到它无法处理的问题时，应该将问题报告给Java虚拟机。

在这种情况下应该抛出异常。但是C语言没有异常，而必须调用`Throw()`或`ThrowNew()`函数来创建新的异常对象。当本地方法退出时，Java虚拟机就会抛出该异常。

要使用`Throw()`函数，需要先调用`NewObject()`来创建一个`Throwable`子类的对象。例如，下面的代码创建了一个`EOFException`对象，然后将它抛出：

```c
jclass class_EOFException = (*env)->FindClass(env, "java/io/EOFException");
jmethodID id_EOFException = (*env)->GetMethodID(env, class_EOFException, "<init>", "()V");
    /* ID of no-argument constructor */
jthrowable obj_exc = (*env)->NewObject(env, class_EOFException, id_EOFException);
(*env)->Throw(env, obj_exc);
```

调用`ThrowNew()`通常会更加方便，该函数使用给定的类和错误消息构造异常对象并抛出。

```c
(*env)->ThrowNew(env, class_EOFException, "Unexpected end of file");
```

这两个函数都只是**发布**(post)异常，不会中断本地方法的执行。只有当方法返回时，Java虚拟机才会抛出异常。因此，每次调用`Throw()`或`ThrowNew()`之后应该紧跟着`return`语句。

C++注释：如果用C++实现本地方法，要确保本地方法不会抛出C++异常。

通常，本地代码不需要考虑捕获Java异常。但是，当本地方法调用Java方法时，该方法可能会抛出异常。另外，有些JNI函数也会抛出异常。例如，如果索引越界，`SetObjectArrayElement()`会抛出`ArrayIndexOutOfBoundsException`。在这类情况下，本地方法应该调用`ExceptionOccurred()`函数来确认是否有异常抛出：

```c
jthrowable obj_exc = (*env)->ExceptionOccurred(env);
```

如果有则返回当前异常对象的引用，否则返回`NULL`。如果只想检查是否有异常抛出，而不需要获得异常对象的引用，则使用

```c
jboolean occurred = (*env)->ExceptionCheck(env);
```

通常，有异常出现时本地方法应该直接返回，以便Java虚拟机将其传播到Java代码。但是，本地方法也可以分析异常对象以确定能否处理该异常。如果能，则必须调用`ExceptionClear()`函数来关闭该异常。

在下一个示例中，我们为本地方法`fprint()`实现了错误处理：
* 如果格式字符串是`NULL`，则抛出`NullPointerException`。
* 如果格式字符串不包含适合打印`double`的`%`说明符，则抛出`IllegalArgumentException`。
* 如果调用`malloc()`失败，则抛出`OutOfMemoryError`。

最后，为了说明如何在本地方法调用Java方法时检查异常，我们将结果字符串逐个字符发送到`PrintWriter`，并在每次发送后调用`ExceptionOccurred()`。程序清单12-17给出了本地方法的代码，程序清单12-18展示了包含本地方法的类的定义。注意，在调用`PrintWriter.print()`出现异常时，本地方法要先释放缓冲区`cstr`。程序清单12-19中的测试程序说明了当格式字符串不合法时，本地方法如何抛出异常。

[程序清单12-17 printf4/Printf4.c](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch12/printf4/Printf4.c)

[程序清单12-18 printf4/Printf4.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch12/printf4/Printf4.java)

[程序清单12-19 printf4/Printf4Test.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch12/printf4/Printf4Test.java)

## 12.9 使用调用API
到目前为止，讨论的都是在Java程序中调用C代码。相反，假设你想在C或C++程序中调用Java代码。**调用API** (invocation API)使你能够把Java虚拟机嵌入到C或C++程序中。下面是初始化虚拟机所需的基本代码：

```c
JavaVMOption options[1];
JavaVMInitArgs vm_args;
JavaVM *jvm;
JNIEnv *env;

options[0].optionString = "-Djava.class.path=.";

memset(&vm_args, 0, sizeof(vm_args));
vm_args.version = JNI_VERSION_1_2;
vm_args.nOptions = 1;
vm_args.options = options;

JNI_CreateJavaVM(&jvm, (void**) &env, &vm_args);
```

调用`JNI_CreateJavaVM()`将创建虚拟机，并使指针`jvm`指向虚拟机，`env`指向执行环境。

可以给虚拟机提供任意数量的选项，只需增加`options`数组的大小和`vm_args.nOptions`的值。例如，

```c
options[i].optionString = "-Djava.compiler=NONE";
```

停用即时(just-in-time)编译器。

提示：当陷入麻烦时（例如不能初始化JVM或者不能加载类），可以打开JNI调试模式。添加一个选项`-verbose:jni`，你将看到一系列指示JVM初始化进度的消息。如果没有看到你的类被加载，请检查路径和类路径设置。

一旦设置好虚拟机，就可以像前面介绍的那样调用Java方法了。只需按照通常的方式使用`env`指针即可。

只有在调用invocation API中的其他函数时才需要`jvm`指针。目前只有四个这样的函数。最重要的一个是终止虚拟机的函数：

```c
(*jvm)->DestroyJavaVM(jvm);
```

遗憾的是，在Windows上，动态链接到jdk/bin/server/jvm.dll库中的`JNI_CreateJavaVM()`函数变得很困难，因为Vista改变了链接规则，而Oracle仍然依赖旧版本的C运行时库。示例程序通过手动加载库解决了这个问题。这与`java`程序所使用的方式一样（参见[java_md.c](https://github.com/openjdk/jdk/blob/jdk-17-ga/src/java.base/windows/native/libjli/java_md.c#L372)中的`LoadJavaVM()`函数）。

程序清单12-20中的C程序设置了虚拟机，并调用了`Welcome`类的`main()`方法（运行测试程序之前先编译Welcome.java）。

[程序清单12-20 invocation/InvocationTest.c](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch12/invocation/InvocationTest.c)

[invocation/Welcome.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch12/invocation/Welcome.java)

要在Linux上编译该程序，使用命令

```shell
gcc -I jdk/include -I jdk/include/linux -o InvocationTest InvocationTest.c -L jdk/lib/server -ljvm
```

在Windows上使用MSVC编译器时，使用命令

```shell
cl /D_WINDOWS /I jdk\include /I jdk\include\win32 InvocationTest.c jdk\lib\jvm.lib advapi32.lib
```

使用Cygwin时，使用命令

```shell
gcc -D_WINDOWS -mno-cygwin -D __int64="long long" -I jdk\include -I jdk\include\win32 \
    -I C:\cygwin\usr\include\w32api -o InvocationTest InvocationTest.c
```

在Linux/UNIX上运行该程序之前，需要将JDK共享库目录添加到环境变量`LD_LIBRARY_PATH`：

```shell
export LD_LIBRARY_PATH=jdk/lib/server:$LD_LIBRARY_PATH
```

## 12.10 完整示例：访问Windows注册表
在本节中，将介绍一个完整的例子，使用Java来包装用于操作Windows注册表的C API子集。

### 12.10.1 Windows注册表概述
Windows注册表(registry)是一个存放Windows操作系统和应用程序的配置信息的数据仓库。

不建议使用注册表来存储Java程序的配置参数。Java首选项(preferences) API是一个更好的解决方案（详见卷I第10章）。我们使用注册表只是为了说明如何将重要的本地API包装成Java类。

查看注册表的主要工具是**注册表编辑器**。打开CMD命令行（或者按Win+R打开“运行”对话框）然后输入`regedit`。下图展示了注册表编辑器。

![注册表编辑器](/assets/images/java-note-v2ch12-native-methods/注册表编辑器.png)

左边是树形结构排列的注册表键。注意，每个键都以`HKEY`节点开始，如

```
HKEY_CLASSES_ROOT
HKEY_CURRENT_USER
HKEY_LOCAL_MACHINE
...
```

右边是与特定键关联的名/值对。例如，如果安装了Java 17，那么键`HKEY_LOCAL_MACHINE\Software\JavaSoft\JDK`就包含`CurrentVersion="17.0.12"`这样的名/值对。在这里，值是字符串。值也可以是整数或字节数组。

### 12.10.2 访问注册表的Java接口
我们创建了一个从Java代码访问注册表的简单接口，然后用本地代码实现了这个接口。为了简单起见，该接口只允许少数几个注册表操作，省略了添加、删除和枚举键等重要操作（添加这些注册表API函数是很容易的）。

使用该接口可以
* 枚举某个键中存储的所有名字
* 读取某个名字对应的值
* 设置某个名字对应的值

程序清单12-21是封装注册表键的Java类：

```java
public class Win32RegKey {
    public Win32RegKey(int theRoot, String thePath) { ... }
    public Enumeration names() { ... }
    public native Object getValue(String name);
    public native void setValue(String name, Object value);

    public static final int HKEY_CLASSES_ROOT = 0x80000000;
    public static final int HKEY_CURRENT_USER = 0x80000001;
    public static final int HKEY_LOCAL_MACHINE = 0x80000002;
    ...
}
```

`names()`方法返回指定的键存储的所有名字的枚举（注：`Enumeration`接口是`Iterator`的前身，见卷I第9章 9.7.2节）。`getValue()`方法返回的对象是字符串、整数或字节数组。`setValue()`方法的`value`参数也必须是上述三种类型之一。

### 12.10.3 以本地方法实现注册表访问函数
有两个问题使得这些本地方法比之前的例子更加复杂。`getValue()`和`setValue()`方法处理的是`Object`类型，它可能是`String`、`Integer`或`byte[]`之一。枚举对象需要存储连续的`hasMoreElements()`和`nextElement()`调用之间的状态。

程序清单12-22包含实现本地方法的C代码。首先看`getValue()`方法，该方法执行以下步骤：
1. 打开注册表键。为了读取它们的值，注册表API要求这些键是开放的。
2. 查询与名字关联的值的类型和大小。
3. 把数据读到缓冲区。
4. 如果类型是`REG_SZ`（字符串），调用`NewStringUTF()`来创建一个新的字符串。
5. 如果类型是`REG_DWORD`（32位整数），调用`Integer`构造器。
6. 如果类型是`REG_BINARY`，调用`NewByteArray()`来创建一个新的字节数组，并调用`SetByteArrayRegion()`将数据复制到字节数组中。
7. 如果不是以上类型或调用API函数时出现错误，则抛出异常并释放当前获取的所有资源。
8. 关闭键，并返回创建的对象。

这个例子很好地说明了如何生成不同类型的Java对象。

在`getValue()`方法中，处理泛化的返回类型并不困难，`jstring`、`jobject`或`jarray`引用都可以直接作为`jobject`返回。但是，`setValue()`方法接收一个`Object`引用，必须确定其确切类型。为此可以查询`value`对象的类，找出`String`、`Integer`和`byte[]`类的引用，并使用`IsAssignableFrom()`函数比较它们。

如果`class1`和`class2`是两个类引用，那么调用

```c
(*env)->IsAssignableFrom(env, class1, class2)
```

当`class1`和`class2`是同一个类或`class1`是`class2`的子类（即`class1`的引用可以赋给`class2`的变量）时，返回`JNI_TRUE`。例如，当

```c
(*env)->IsAssignableFrom(env, (*env)->GetObjectClass(env, value), (*env)->FindClass(env, "[B"))
```

为true时，就知道`value`是一个字节数组。

下面是`setValue()`方法的步骤：
1. 打开注册表键用于写入。
2. 找出要写入的值的类型。
3. 如果类型是`String`，调用`GetStringUTFChars()`获得指向字符的指针。
4. 如果类型是`Integer`，调用`intValue()`方法获得包装器中存储的整数。
5. 如果类型是`byte[]`，调用`GetByteArrayElements()`获得指向字节的指针。
6. 把数据和长度传递给注册表。
7. 关闭键。
8. 如果类型是`String`或`byte[]`，释放指向数据的指针。

最后介绍枚举键的本地方法。这些方法属于`Win32RegKeyNameEnumeration`类（见程序清单12-21）。当枚举过程开始时，必须打开键。在枚举过程中，必须将键的句柄保存在枚举对象中。键的句柄是`HKEY`类型（32位整数），因此可以存放在Java整数中。它存放在枚举类的`hkey`字段中。当枚举开始时，使用`SetIntField()`初始化该字段，后续调用使用`GetIntField()`读取其值。

枚举对象还存放了另外三个数据。当枚举开始时，可以从注册表查询名字的数量和最长名字长度（用于分配保存名字的C字符数组）。这些值存放在枚举对象的`count`和`maxsize`字段中。最后，`index`字段指示当前名字的索引：初始化为-1，当其他字段初始化后置为0，每次枚举之后加1。

下面是枚举类本地方法的步骤。`hasMoreElements()`方法很简单：
1. 获取`index`和`count`字段。
2. 如果`index`是-1，调用`startNameEnumeration()`函数：打开键，查询数量和最大长度，并初始化`hkey`, `count`, `maxsize`和`index`字段。
3. 如果`index`小于`count`则返回`JNI_TRUE`，否则返回`JNI_FALSE`。

`nextElement()`方法要复杂一些：
1. 获取`index`和`count`字段。
2. 如果`index`是-1，调用`startNameEnumeration()`函数。
3. 如果`index`大于等于`count`，则抛出`NoSuchElementException`。
4. 从注册表读取下一个名字。
5. 递增`index`。
6. 如果`index`等于`count`，则关闭键。

在编译前，先生成头文件：

```shell
javac -h . Win32RegKey.java
```

MSVC编译器的命令为

```shell
cl /I jdk\include /I jdk\include\win32 /LD Win32RegKey.c advapi32.lib /FeWin32RegKey.dll
```

对于Cygwin，使用

```shell
gcc -mno-cygwin -D __int64="long long" -I jdk\include -I jdk\include\win32 \
    -I C:\cygwin\usr\include\w32api -shared -Wl,--add-stdcallalias -o Win32RegKey.dll Win32RegKey.c
```

由于注册表API是Windows特有的，所以这个程序不能在其他操作系统上运行。

程序清单12-23给出了测试程序。我们在键`HKEY_LOCAL_MACHINE\Software\JavaSoft\JDK`中添加了三个名/值对：一个字符串、一个整数、一个字节数组。然后枚举该键的所有名字并获取其值，程序应该打印

```
Default user=Harry Hacker
Lucky number=13
Small primes=2 3 5 7 11
```

![修改后的注册表](/assets/images/java-note-v2ch12-native-methods/修改后的注册表.png)

虽然在该键中添加这些名/值对不会有什么害处，但是在运行该程序后你可能还是想使用注册表编辑器删除它们（√）。

[程序清单12-21 win32reg/Win32RegKey.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch12/win32reg/Win32RegKey.java)

[程序清单12-22 win32reg/Win32RegKey.c](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch12/win32reg/Win32RegKey.c)

[程序清单12-23 win32reg/Win32RegKeyTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch12/win32reg/Win32RegKeyTest.java)

注：
* 必须用管理员权限运行该程序，否则写入注册表会失败。
* 为了使用MSVC编译器和头文件windows.h，需要从 <https://visualstudio.microsoft.com/downloads/> 下载Visual Studio IDE，或者从 <https://visualstudio.microsoft.com/visual-cpp-build-tools/> 下载Visual Studio生成工具。

安装时，勾选“使用C++的桌面开发”、“MSVC”和“Windows SDK”，如下图所示。

![安装Visual Studio生成工具](/assets/images/java-note-v2ch12-native-methods/安装Visual Studio生成工具.png)

安装完成后，运行开始菜单中的 "Visual Studio 2022 → x64 Native Tools Command Prompt for VS 2022" 。这个命令行工具设置了必要的环境变量，例如：
* 可执行程序路径`PATH`（可以直接使用编译器cl.exe和链接器link.exe）
* 头文件路径`INCLUDE`（包括windows.h所在目录C:\Program Files (x86)\Windows Kits\10\Include\10.0.26100.0\um，无需使用`/I`选项手动设置）
* 库路径`LIB`（包括AdvAPI32.lib所在目录C:\Program Files (x86)\Windows Kits\10\Lib\10.0.26100.0\um\x64）

## 12.11 外部函数：展望未来
使用JNI时，必须编写C代码来访问Java数据结构，调用需要的C函数，然后将结果转换回Java。然后还需要将C代码链接到特定平台的库。在“巴拿马项目”(Project Panama)中开发了一个用于访问“外部”(foreign)函数和内存的API，使你可以用Java来编写这类代码。

在Java 17中，这个API还是预览特性，在最终版发布前细节还会有所变化。程序清单12-24中的程序是这个API一个非常简单的演示。

[程序清单12-24 panama/PanamaDemo.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch12/panama/PanamaDemo.java)

这个程序调用了`printf()`函数，不需要任何额外的C代码。该函数是通过Java方法句柄(`MethodHandle`)调用的（这个类通常用于调用Java函数）。

为了访问C函数，需要先创建一个`CLinker`对象。给定函数地址和类型描述，`CLinker`对象就可以产生C函数的Java方法句柄。这个API的细节还在不断变化中。

这个API包含将Java对象转换为可以传递给C函数的内存块的方法。在示例程序中，需要向`printf()`函数提供一个`char*`，其字符来自Java `String`。`CLinker.toCString()`方法提供这种转换。

内存是在一个`ResourceScope`中分配的，以便管理其释放。这里使用了**受限作用域**(confined scope)。它是可自动关闭的，`close()`方法会释放所有已分配的内存。

可以看到，这个API比JNI简单、方便得多。示例程序只展示了这个API的皮毛，还有许多用于读、写和管理内存块的方法。还可以将Java函数作为回调传递给C函数。

为了编译这个示例程序，需要使用以下选项：

```shell
javac --enable-preview --source 17 --add-modules jdk.incubator.foreign panama/PanamaDemo.java
```

通常会将Java和C之间的桥接代码放到一个单独的模块中。但是，在这个示例中，本地访问发生在无名模块中（注：参见9.9节）。下面是运行该程序的命令：

```shell
java --add-modules jdk.incubator.foreign --enable-native-access=ALL-UNNAMED panama.PanamaDemo
```

目前，你仍然需要使用JNI作为与本地代码交互的稳定API。随着外部函数和内存API的成熟，它将提供一个JNI的卓越替代品。
