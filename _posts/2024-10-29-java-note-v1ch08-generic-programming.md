---
title: 《Java核心技术》笔记 第8章 泛型编程
date: 2024-10-29 21:58:25 +0800
categories: [Java, Core Java]
tags: [java, generic programming, type erasure, bridge method, wildcard type, reflection]
---
泛型类和泛型方法有类型参数，这使得它们可以准确地描述用特定类型实例化时会发生什么。在有泛型类之前，程序员必须使用`Object`编写适用于多种类型的代码，这既繁琐又不安全。

在本章中，你将了解泛型编程的优势及挑战。

## 8.1 为何使用泛型编程
**泛型编程**(generic programming)意味着编写可用于多种不同类型的对象的代码。`ArrayList`就是一个例子（见5.3节）。

### 8.1.1 类型参数的好处
在Java增加泛型类之前，泛型编程是用继承实现的。`ArrayList`类只维护一个`Object`数组：

```java
// before generic classes
public class ArrayList {
    private Object[] elementData;
    ...
    public Object get(int i) { ... }
    public void add(Object o) { ... }
}
```

这种方法有两个问题。获取一个值时必须进行强制类型转换：

```java
ArrayList files = new ArrayList();
...
String filename = (String) files.get(0);
```

此外，没有错误检查。可以添加任何类的值：

```java
files.add(new File("..."));
```

这个调用编译和运行都不会出错。但在其他地方将`get()`的结果强制转换为`String`就会产生一个错误。

泛型提供了一个更好的解决方案：**类型参数**(type parameter)（或**类型变量**(type variable)）。现在`ArrayList`类有一个类型参数表示元素类型：

```java
var files = new ArrayList<String>();
```

这使得代码具有更好的可读性。

注释：如果使用明确的类型而不是`var`声明一个变量，可以通过使用“菱形”语法省略构造器中的类型参数：

```java
ArrayList<String> files = new ArrayList<>();
```

省略的类型可以从变量的类型推断得出。Java 9扩展了菱形语法的使用范围。例如，现在可以对匿名子类使用菱形语法：

```java
ArrayList<String> passwords = new ArrayList<>() { // diamond OK in Java 9
    public String get(int n) { return super.get(n).replaceAll(".", "*"); }
};
```

编译器也可以充分利用这个类型信息。调用`get()`时不需要强制类型转换，编译器知道返回类型是`String`：

```java
String filename = files.get(0);
```

编译器还知道`add()`方法有一个`String`类型的参数，这比`Object`参数安全得多，插入错误类型的对象将导致编译错误：

```java
files.add(new File("...")); // error
```

这就是泛型的魅力所在：让你的程序更易读、更安全。

### 8.1.2 谁想成为泛型程序员
泛型编程可以分为三个能力水平。基本水平是，仅仅使用泛型类（如`ArrayList`）而不考虑其工作原理。大多数应用程序员都会停留在这一水平，直到出现了问题。不过，当混合使用不同的泛型类，或者与对类型参数一无所知的遗留代码交互时，可能会遇到令人困惑的错误消息。那时你就需要对Java泛型有足够的了解，才能系统地解决问题，而不是胡乱地猜测。当然，最终你可能希望实现自己的泛型类和泛型方法。

## 8.2 定义简单泛型类
**泛型类**(generic class)是有一个或多个类型变量的类。本章使用一个简单的`Pair`类作为示例。下面是泛型`Pair`类的代码：

[pair/Pair.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch08/pair/Pair.java)

`Pair`类引入了一个类型变量`T`，用尖括号`<>`括起来，放在类名后。泛型类可以有多个类型变量。例如，可以定义`first`和`second`字段使用不同类型的`Pair`类：

```java
public class Pair<T, U> { ... }
```

类型变量在整个类定义中用于指定方法的返回类型以及字段和局部变量的类型。例如：

```java
private T first; // uses the type variable
```

注释：常见的做法是使用简短的大写字母作为类型变量。Java标准库使用`E`表示集合的元素类型，`K`和`V`表示映射的键和值类型，`T`（以及相邻的`U`和`S`）表示任意类型。

可以用具体的类型替换类型参数来**实例化**(instantiate)泛型类。例如`Pair<String>`，可以将结果**想象**成一个普通类：

```java
public class Pair {
    private String first;
    private String second;

    public Pair() { ... }
    public Pair(String first, String second) { ... }

    public String getFirst() { ... }
    public String getSecond() { ... }

    public void setFirst(String newValue) { ... }
    public void setSecond(String newValue) { ... }
}
```

换句话说，泛型类相当于普通类的工厂（模板）。

程序清单8-1中的程序使用了`Pair`类。

[程序清单8-1 pair1/PairTest1.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch08/pair1/PairTest1.java)

C++注释：从表面上看，Java的泛型类类似于C++的模板类，唯一明显的不同是Java没有特殊的`template`关键字。然而，在本章中你将会看到，这两种机制有着本质的区别。

## 8.3 泛型方法
除了泛型类，还可以定义带有类型变量的**泛型方法**(generic Method)。

```java
class ArrayAlg {
    public static <T> T getMiddle(T... a) {
        return a[a.length / 2];
    }
}
```

注意，类型变量放在修饰符之后、返回类型之前。

泛型方法可以定义在普通类中，也可以定义在泛型类中。

当调用一个泛型方法时，可以把具体类型用尖括号括起来，放在方法名之前：

```java
String middle = ArrayAlg.<String>getMiddle("John", "Q.", "Public");
```

在大多数情况下，方法调用中可以省略类型参数，编译器可以推断出类型参数：

```java
String middle = ArrayAlg.getMiddle("John", "Q.", "Public");
```

编译器偶尔也会出错。考虑这个示例：

```java
double middle = ArrayAlg.getMiddle(3.14, 1729, 0);
```

错误消息以晦涩的方式指出：解释这行代码有两种方式，而且都是合法的。简单地说，编译器会把参数自动装箱为一个`Double`和两个`Integer`对象，然后尝试寻找这些类的公共超类型。实际上找到两个：`Number`类和`Comparable`接口（因此无法自动推断类型参数`T`）。解决方法是将所有的参数都写为`double`：

```java
double middle = ArrayAlg.getMiddle(3.14, 1729.0, 0.0);
```

注：即使在调用时显式指定类型参数为`Double`，编译器仍然会报错“类型不匹配”，因为`int`无法转换为`Double`。

提示：如果想知道编译器对一个泛型方法调用推断出哪种类型，可以使用这个窍门：故意引入一个错误，然后研究所得到的错误消息。例如，考虑调用`ArrayAlg.getMiddle("Hello", 0, null)`。将结果赋给一个`JButton`，这肯定是不对的，将会得到错误报告：

```
error: incompatible types: inferred type does not conform to upper bound(s)
        JButton middle = ArrayAlg.getMiddle("Hello", 0, null);
                                           ^
    inferred: INT#1
    upper bound(s): JButton,Object
  where INT#1,INT#2 are intersection types:
    INT#1 extends Object,Serializable,Comparable<? extends INT#2>,Constable,ConstantDesc
    INT#2 extends Object,Serializable,Comparable<?>,Constable,ConstantDesc
```

也就是说，可以将结果赋给`Object`、`Serializable`、`Comparable`、`Constable`或`ConstantDesc`。

## 8.4 类型变量的边界
有时，泛型类或方法需要对类型变量加以约束。下面是一个典型的例子，我们要计算数组中的最小元素：

```java
class ArrayAlg {
    public static <T> T min(T[] a) { // almost correct
    if (a == null || a.length == 0) return null;
    T smallest = a[0];
    for (int i = 1; i < a.length; i++)
        if (smallest.compareTo(a[i]) > 0) smallest = a[i];
        return smallest;
    }
}
```

但是这里有一个问题。变量`smallest`的类型为`T`，这意味着它可以是任意类的对象。如何知道类型`T`有`compareTo()`方法呢？

解决方法是将`T`限制为实现了`Comparable`接口的类。可以通过对类型变量`T`设置**边界**(bound)来实现这一点：

```java
public static <T extends Comparable> T min(T[] a) ...
```

注释：实际上，`Comparable`接口本身就是一个泛型类型。目前先忽略这一复杂性以及编译器产生的警告。8.8.2节会讨论如何在`Comparable`接口中适当地使用类型参数。

现在，泛型方法`min()`只能使用实现了`Comparable`接口的类的数组调用，否则将产生编译错误。

C++注释：在早期版本的C++中，不能对模板参数的类型加以限制。为此，C++20引入了概念。详见[C++20之概念]({% post_url 2024-03-24-cpp20-concept %})。

在这里为什么使用关键字`extends`而不是`implements`？毕竟`Comparable`是一个接口。记法`<T extends BoundingType>`表示`T`应该是边界类型的**子类型**(subtype)。`T`和边界类型都可以是类，也可以是接口。选择关键字`extends`的原因是它更接近子类型的概念。

一个类型变量可以有多个边界，用`&`分隔。例如：`T extends Comparable & Serializable`。

和继承一样，边界类型可以有任意多个接口，但至多有一个是类。如果有一个类作为边界，它必须是列表中的第一个。

注：边界只能限定子类型，不能限定超类型，但通配符可以，见8.8.2节。

在程序清单8-2中，我们把`minmax()`重写为泛型方法。

[程序清单8-2 pair2/PairTest2.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch08/pair2/PairTest2.java)

## 8.5 泛型代码和虚拟机
**虚拟机没有泛型类型对象**——所有对象都属于普通类。在下面几节中，你会看到编译器如何“擦除”类型参数，以及这个过程对Java程序员有什么影响。

### 8.5.1 类型擦除
每当定义一个泛型类型，都会自动提供一个相应的**原始类型**(raw type)，这个原始类型的名字就是去掉类型参数后的泛型类型名，这叫做**类型擦除**(type erasure)。类型变量会被擦除(erased)，并**替换为其边界类型**（无边界的类型变量替换为`Object`）。

例如，`Pair<T>`的原始类型如下所示：

```java
public class Pair {
    private Object first;
    private Object second;

    public Pair() { ... }
    public Pair(Object first, Object second) { ... }

    public Object getFirst() { ... }
    public Object getSecond() { ... }

    public void setFirst(Object newValue) { ... }
    public void setSecond(Object newValue) { ... }
}
```

因为`T`是一个无边界的类型变量，所以直接替换为`Object`。其结果是一个普通类，就像Java中引入泛型之前实现的类一样。

注：如果使用第5章的`ReflectionTest`程序查看`Pair`类，输出的是原始类型，而`javap`工具输出的是泛型类，如下所示。

```shell
$ cd out/v1ch08
$ java -cp .:../v1ch05 reflection.ReflectionTest pair.Pair        
public class pair.Pair
{
    public pair.Pair();
    public pair.Pair(java.lang.Object, java.lang.Object);

    public void setFirst(java.lang.Object);
    public void setSecond(java.lang.Object);
    public java.lang.Object getFirst();
    public java.lang.Object getSecond();

    private java.lang.Object first;
    private java.lang.Object second;
}

$ javap -private pair.Pair 
Compiled from "Pair.java"
public class pair.Pair<T> {
  private T first;
  private T second;
  public pair.Pair();
  public pair.Pair(T, T);
  public T getFirst();
  public T getSecond();
  public void setFirst(T);
  public void setSecond(T);
}
```

在程序中可能包含不同类型的`Pair`，例如`Pair<String>`或`Pair<LocalDate>`，但类型擦除会把它们都变成原始的`Pair`类型。

C++注释：就这点而言，Java泛型与C++模板有很大的区别。C++会为每个模板实例化生成不同的类型，这一现象称为“模板代码膨胀”。Java不受这个问题的困扰（注：但受另外一大堆问题的困扰，见8.6节）。

注：
* C++：Code specialization，例如`vector<int>`和`vector<string>`是两个完全不同的类。
* Java：Code sharing，例如`Pair<String>`和`Pair<LocalDate>`都被擦除为`Pair`。

原始类型用第一个边界来替换类型变量，如果没有给定边界则替换为`Object`。例如，`<T extends Comparable & Serializable>`将替换为`Comparable`。

注释：如果将边界切换为`<T extends Serializable & Comparable>`，原始类型就会将`T`替换为`Serializable`，编译器会在必要时插入到`Comparable`的强制类型转换。为了提高效率，应该将标记接口（即没有方法的接口）放在边界列表的末尾。

### 8.5.2 翻译泛型表达式
当调用泛型方法时，如果返回类型被擦除，编译器会插入强制类型转换。例如，考虑以下语句

```java
Pair<Employee> buddies = ...;
Employee buddy = buddies.getFirst();
```

`getFirst()`擦除后的返回类型是`Object`，编译器会自动插入到`Employee`的强制类型转换：

```java
Pair buddies = ...;
Employee buddy = (Employee) buddies.getFirst();
```

访问泛型字段时也会插入强制类型转换。

### 8.5.3 翻译泛型方法
泛型方法也会发生类型擦除。例如，下面的泛型方法

```java
public static <T extends Comparable> T min(T[] a)
```

并不是（像C++一样）整个一组方法，类型擦除之后只剩下一个方法：

```java
public static Comparable min(Comparable[] a)
```

方法的擦除带来了几个复杂问题。考虑这个示例：

```java
class DateInterval extends Pair<LocalDate> {
    public void setSecond(LocalDate second) {
        if (second.compareTo(getFirst()) >= 0)
            super.setSecond(second);
    }
    ...
}
```

日期区间是一对`LocalDate`对象，通过覆盖`setSecond()`方法来确保第二个值不小于第一个值。考虑下面的语句序列：

```java
var interval = new DateInterval(...);
Pair<LocalDate> pair = interval; // OK--assignment to superclass
pair.setSecond(aDate);
```

我们期望`setSecond()`的调用具有多态性，这里应该调用`DateInterval.setSecond()`。然而，擦除后的超类方法是`setSecond(Object)`，`DateInterval.setSecond()` **并未覆盖**超类方法。问题在于**类型擦除与多态发生了冲突**。为了解决这个问题，编译器会在`DateInterval`类中生成一个**桥方法**(bridge method)：

```java
public void setSecond(Object second) { setSecond((LocalDate) second); }
```

桥方法用于覆盖超类的`setSecond(Object)`方法并调用`DateInterval.setSecond(LocalDate)`方法。要了解为什么这样可行，下面仔细分析语句`pair.setSecond(aDate)`的执行过程。

变量`pair`声明为类型`Pair<LocalDate>`，这个类型只有一个名为`setSecond`的方法，即`setSecond(Object)`。虚拟机在`pair`引用的对象上调用这个方法。这个对象是`DateInterval`类型，因此会调用`DateInterval.setSecond(Object)`方法（由于多态性）。这个方法是合成的桥方法，它会调用`DateInterval.setSecond(LocalDate)`，这正是我们想要的。

![桥方法](/assets/images/java-note-v1ch08-generic-programming/桥方法.png)

桥方法可能会变得更奇怪。假设`DateInterval`类也覆盖了`getSecond()`方法：

```java
class DateInterval extends Pair<LocalDate> {
    public LocalDate getSecond() { return super.getSecond(); }
    ...
}
```

实际上，`DateInterval`类会有两个`getSecond()`方法：

```java
LocalDate getSecond() // defined in DateInterval
Object getSecond() // overrides the method defined in Pair to call the first method
```

你不能编写这样的Java代码（两个同名方法有相同的参数类型是不合法的）。但是，在虚拟机中，由参数类型**和返回类型**共同指定一个方法。因此，编译器可以为两个只有返回类型不同的方法生成字节码，虚拟机能够正确地处理这种情况。

[bridgeMethod/BridgeMethodTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch08/bridgeMethod/BridgeMethodTest.java)

注：栈轨迹并不会体现桥方法。但`javap`工具的输出会包含桥方法，如下所示。

```java
public class DateInterval extends pair.Pair<java.time.LocalDate> {
  public DateInterval(java.time.LocalDate, java.time.LocalDate);
  public void setSecond(java.time.LocalDate);
  public java.time.LocalDate getSecond();
  public void setSecond(java.lang.Object);
  public java.lang.Object getSecond();
}
```

注释：桥方法不仅限于泛型类型。第5章已经讲过，覆盖超类方法时可以指定一个更严格的返回类型。例如6.1.9节中的`Employee.clone()`。实际上，`Employee`类有两个`clone()`方法：

```java
Employee clone() // defined in Employee
Object clone() // synthesized bridge method, overrides Object.clone
```

合成的桥方法会调用新定义的方法。

总之，对于Java泛型的翻译，需要记住以下几点：
* 虚拟机中没有泛型，只有普通类和方法。
* 所有的类型变量都会替换为其边界类型。
* 编译器会合成桥方法来保持多态。
* 编译器会在必要时插入强制类型转换来保持类型安全。

### 8.5.4 调用遗留代码
设计Java泛型时，一个主要目标是允许泛型代码和遗留代码之间能够互操作。下面看一个遗留代码的具体示例（注：下面的示例来自5.3.3节）。

假设有以下遗留类：

```java
public class EmployeeDB {
    public void update(ArrayList list) { ... }
    public ArrayList find(String query) { ... }
}
```

可以将一个类型化的数组列表传递给`update()`方法，而无需强制类型转换：

```java
ArrayList<Employee> staff = ...;
employeeDB.update(staff);
```

警告：尽管编译器没有给出错误或警告，但是这样调用并不安全。`update()`方法可能会在数组列表中添加不是`Employee`类型的元素，访问这些元素时就会出现`ClassCastException`异常。实际上，这种行为与Java中引入泛型之前是一样的。

相反，将一个原始数组列表赋给类型化的数组列表时，会得到一个警告：

```java
ArrayList<Employee> result = employeeDB.find(query); // yields warning
```

注释：要看到警告的文字信息，需要添加编译选项`-Xlint:unchecked`。

使用强制类型转换并不能避免出现警告。

在这种情形下并不能做什么。在与遗留代码交互时，要研究编译器的警告，确保这些警告不太严重（不存在违反类型规则的现象）。一旦确保没有问题，可以用`@SuppressWarnings("unchecked")`注解来标记变量或整个方法，如下所示：

```java
@SuppressWarnings("unchecked")
ArrayList<Employee> result = employeeDB.find(query); // yields warning
```

## 8.6 限制与局限性
在下面几节中，将讨论使用Java泛型时需要考虑的一些限制。大多数限制都是类型擦除的后果（万恶之源！）。

### 8.6.1 不能用基本类型替换类型参数
**不能用基本类型替换类型参数。** 因此，没有`Pair<double>`，只有`Pair<Double>`。原因是类型擦除之后，`Pair`类有`Object`类型的字段，不能存储`double`值。

### 8.6.2 运行时类型查询只适用于原始类型
虚拟机中的对象总是有一个特定的非泛型类型。因此，类型查询只适用于原始类型。例如：

```java
Pair a = new Pair<Integer>(3, 4);
if (a instanceof Pair<String>) // compile-time error
```

只能测试`a`是否是一个任意类型的`Pair`。

注：无论`a`的类型是`Pair`或`Pair<Integer>`，这个测试都会产生编译错误，但可以测试`a instanceof Pair`。

强制类型转换也是如此：

```java
Pair<String> b = (Pair<String>) a; // warning
```

注：如果`a`的类型是`Pair<Integer>`，则这个转换会导致编译错误。

同样的道理，`getClass()`方法总是返回原始类型。例如：

```java
Pair<String> stringPair = ...;
Pair<Employee> employeePair = ...;
if (stringPair.getClass() == employeePair.getClass()) // they are equal
```

这个比较的结果是true，因为两个`getClass()`调用都返回`Pair.class`（注：`Pair<String>.class`是不合法的）。

### 8.6.3 不能创建泛型类型的数组
不能创建泛型类型的数组。例如：

```java
var table = new Pair<String>[10]; // ERROR
```

因为擦除之后，`table`的类型是`Pair[]`。可以把它转换为`Object[]`：

```java
Object[] objarray = table;
```

数组会记住它的元素类型，如果试图存储类型不正确的元素就会抛出`ArrayStoreException`（见5.1.5节警告）：

```java
objarray[0] = "Hello"; // ERROR--component type is Pair
```

但是擦除会使这种机制对泛型类型无效（只能记住原始类型）。赋值

```java
objarray[0] = new Pair<Employee>();
```

尽管能够通过数组存储的检查（因为`Pair<Employee>`确实是`Pair`的子类），但仍然会导致类型错误。出于这个原因，不允许创建类型化参数的数组。

注意，只是不允许创建这些数组。仍然可以声明`Pair<String>[]`类型的变量，但是不能用`new Pair<String>[10]`或`{}`初始化（注：可以使用`new Pair[10]`初始化）。

注释：可以创建通配符类型的数组，然后进行强制类型转换：

```java
var table = (Pair<String>[]) new Pair<?>[10];
```

结果是不安全的。如果在`table[0]`中存储一个`Pair<Employee>`，然后对`table[0].getFirst()`调用一个`String`方法，会得到一个`ClassCastException`异常。

```java
Pair a = new Pair<Employee>(...);
table[0] = a;
String s = table[0].getFirst().toLowerCase(); // ClassCastException
```

提示：如果需要收集泛型类型对象，直接使用`ArrayList`：`ArrayList<Pair<String>>`很安全也很高效。

### 8.6.4 变参警告
在上一节中已经看到，Java不支持泛型类型的数组。这一节讨论一个相关的问题：向参数个数可变的方法传递泛型类型的实例。

考虑下面这个简单的方法：

```java
public static <T> void addAll(Collection<T> coll, T... ts) {
    for (T t : ts) coll.add(t);
}
```

回忆一下，参数`ts`实际上是一个包含提供的所有参数的数组（见5.5节）。考虑以下调用：

```java
Collection<Pair<String>> table = ...;
Pair<String> pair1 = ...;
Pair<String> pair2 = ...;
addAll(table, pair1, pair2);
```

为了调用这个方法，虚拟机必须创建一个`Pair<String>`数组，这就违反了规则。不过，对于这种情况规则有所放松，你只会得到一个警告而不是错误。

可以用两种方式来抑制这个警告。可以为包含`addAll()`调用的方法添加注解`@SuppressWarnings("unchecked")`。或者从Java 7起，还可以为`addAll()`方法本身添加注解`@SafeVarargs`。

对于任何只读取参数数组元素的方法，都可以使用`@SafeVarargs`注解（注：例如`Arrays.asList()`方法）。这个注解只能用于`static`、`final`或（从Java 9起）`private`的构造器和方法。任何其他方法都可能被覆盖，使这个注解失去意义。

注释：可以使用`@SafeVarargs`注解来打破对于创建泛型类型数组的限制：

```java
@SafeVarargs static <E> E[] array(E... array) { return array; }
```

现在可以调用

```java
Pair<String>[] table = array(pair1, pair2);
```

这看起来很方便，但是隐藏着危险。以下代码

```java
Object[] objarray = table;
objarray[0] = new Pair<Employee>();
```

能顺利运行而不会出现`ArrayStoreException`（因为数组存储只会检查擦除后的类型），但在使用`table[0]`时会在别处得到一个异常(`ClassCastException`)。

注：变参的类型除了可以是类型变量本身(`T...`)，还可以是泛型类型（如`Pair<T>...`）。例如：

```java
public static <T> void print(Pair<T>... pairs) {
    for (Pair<T> p : pairs)
        System.out.println(p.getFirst() + "," + p.getSecond());
}
```

甚至可以用混合的类型调用这个方法：

```java
print(new Pair<>("a", "b"), new Pair<>(1, 2));
```

### 8.6.5 不能实例化类型变量
不能在类似`new T(args)`的表达式中使用类型变量。例如，下面的`Pair<T>`构造器是非法的：

```java
public Pair() { first = new T(); second = new T(); } // ERROR
```

类型擦除会将`T`变成`Object`，而你肯定不是希望调用`new Object()`。

在Java 8之后，最好的解决方法是让调用者提供一个构造器引用（见6.2.5节）。例如：

```java
Pair<String> p = Pair.makePair(String::new);
```

静态方法`makePair()`接收一个`Supplier<T>`：

```java
public static <T> Pair<T> makePair(Supplier<T> constr) {
    return new Pair<>(constr.get(), constr.get());
}
```

注：可以在泛型类`Pair<T>`中定义静态方法，但不能直接引用类型变量`T`，除非其本身是泛型方法。在调用时，必须通过原始类型`Pair`调用，不能通过参数化类型（如`Pair<String>`）调用。

比较传统的解决方法是通过反射来构造泛型对象：让调用者提供一个`Class`对象并调用`Constructor.newInstance()`方法。

```java
public static <T> Pair<T> makePair(Class<T> cl) {
    try {
        return new Pair<>(cl.getConstructor().newInstance(), cl.getConstructor().newInstance());
    }
    catch (Exception e) { return null; }
}
```

这个方法可以如下调用：

```java
Pair<String> p = Pair.makePair(String.class);
```

注意，不能调用

```java
first = T.class.getConstructor().newInstance(); // ERROR
```

表达式`T.class`是不合法的，因为它会擦除为`Object.class`。

[genericAlgorithms/Pair.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch08/genericAlgorithms/Pair.java)

### 8.6.6 不能构造泛型数组
就像不能用类型变量实例化对象一样，也不能实例化数组：`new T[n]`是非法的。不过原因有所不同——数组会记住元素类型，用来监控虚拟机中的数组存储，这个类型会被擦除（与8.6.3节讨论的问题类似）。例如，考虑下面的例子：

```java
public static <T extends Comparable> T[] minmax(T... a) {
    T[] result = new T[2]; // ERROR
    ...
}
```

类型擦除会让这个方法总是构造`Comparable[2]`数组。

如果数组仅用作类的私有实例字段，可以将数组的元素类型声明为擦除后的类型并使用强制类型转换。例如，`ArrayList`类可以如下实现：

```java
public class ArrayList<E> {
    private Object[] elements;
    ...
    @SuppressWarnings("unchecked") public E get(int n) { return (E) elements[n]; }
    public void set(int n, E e) { elements[n] = e; } // no cast needed
}
```

这个技术并不适用于`minmax()`方法。假设实现如下：

```java
public static <T extends Comparable> T[] minmax(T... a) {
    var result = new Comparable[2]; // array of erased type
    ...
    return (T[]) result; // compiles with warning
}
```

这里的强制类型转换`T[]`是一个彻头彻尾的谎言（擦除后就是`Comparable[]`）。以下调用

```java
String[] names = ArrayAlg.minmax("Tom", "Dick", "Harry");
```

编译时不会有任何警告。但在方法返回后将`Comparable[]`强制转换为`String[]`时，将会发生`ClassCastException`。

[limitations/NoGenericArray.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch08/limitations/NoGenericArray.java)

注：由于类型擦除，`minmax()`方法的返回类型实际上就是`Comparable[]`。如5.9.6节所述，通过`new String[n]`创建的数组可以临时赋给`Comparable[]`变量，之后再转换成`String[]`。但一开始就是`new Comparable[n]`创建的数组永远不能转换成`String[]`。如果将`names`的类型改为`Comparable[]`或`Object[]`就不会有异常。

在这种情况下，最好让调用者提供一个数组构造器引用：

```java
String[] names = ArrayAlg.minmax(String[]::new, "Tom", "Dick", "Harry");
```

`minmax()`方法使用这个参数创建正确类型的数组：

```java
public static <T extends Comparable> T[] minmax(IntFunction<T[]> constr, T... a) {
    T[] result = constr.apply(2);
    ...
    return result;
}
```

比较老式的方法是使用反射并调用`Array.newInstance()`（见5.9.6节）：

```java
public static <T extends Comparable> T[] minmax(T... a) {
    var result = (T[]) Array.newInstance(a.getClass().getComponentType(), 2);
    ...
    return result;
}
```

注：泛型版本的`Arrays.copyOf()`方法就采用了这种方法。

[genericAlgorithms/GenericAlgorithms.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch08/genericAlgorithms/GenericAlgorithms.java)

`ArrayList`类的`toArray()`方法就没有这么幸运了。它需要生成一个`T[]`数组，但没有元素类型（因为`ArrayList`的底层数组本身就是`Object[]`）。因此，有两个变体：

```java
Object[] toArray()
T[] toArray(T[] a)
```

第二个变体接收一个数组参数。如果数组`a`足够大，就使用这个数组；否则，用`a`的元素类型创建一个足够大的新数组。

注：Java 11增加了该方法的第三种形式：

```java
T[] toArray(IntFunction<T[]> generator)
```

这样就可以使用数组构造器引用：

```java
ArrayList<String> x = ...;
String[] y = x.toArray(String[]::new);
```

### 8.6.7 泛型类的静态上下文中类型变量无效
不能在静态字段或方法中引用类型变量。例如，下面这个实现单例模式的聪明主意行不通：

```java
public class Singleton<T> {
    private static T singleInstance; // ERROR

    public static T getSingleInstance() { // ERROR
        if (singleInstance == null) // construct new instance of T
        return singleInstance;
    }
}
```

如果这样可行，程序就可以声明`Singleton<Random>`以共享一个随机数生成器，声明`Singleton<JFileChooser>`以共享一个文件选择器对话框。但是这样是行不通的。类型擦除之后，只有一个`Singleton`类，也只有一个`singleInstance`字段。因此，带有类型变量的静态字段和方法是非法的（即使允许也达不到预期的效果）。

### 8.6.8 不能抛出或捕获泛型类的实例
不能抛出也不能捕获泛型类的对象。实际上，泛型类扩展`Throwable`甚至都是不合法的。例如：

```java
public class Problem<T> extends Exception { ... } // ERROR--can't extend Throwable
```

不能在`catch`子句中使用类型变量。例如：

```java
public static <T extends Throwable> void doWork(Class<T> t) {
    try {
        // do work
    }
    catch (T e) { // ERROR--can't catch type variable
        Logger.global.info(...);
    }
}
```

不过，在`throws`声明中使用类型变量是可以的。例如：

```java
public static <T extends Throwable> void doWork(T t) throws T { // OK
    try {
        // do work
    }
    catch (Throwable realCause) {
        t.initCause(realCause);
        throw t;
    }
}
```

### 8.6.9 可以打破对检查型异常的检查
Java异常处理的一个基本原则是，必须为所有检查型异常提供处理器。不过可以利用泛型来打破这个规则。考虑以下方法：

```java
@SuppressWarnings("unchecked")
static <T extends Throwable> void throwAs(Throwable t) throws T {
    throw (T) t;
}
```

假设这个方法定义在接口`Task`中。如果有一个检查型异常`e`，并调用`Task.<RuntimeException>throwAs(e)`，编译器就会认为`e`变成了一个非检查型异常。以下代码会把所有异常都转换为编译器所认为的非检查型异常：

```java
try {
    // do work
}
catch (Exception e) {
    Task.<RuntimeException>throwAs(e);
}
```

下面使用这个技术解决一个棘手的问题。要在一个线程中运行代码，需要把代码放在一个实现了`Runnable`接口的类的`run()`方法中。但是这个方法不允许抛出检查型异常。我们提供一个从`Task`到`Runnable`的适配器，它的`run()`方法允许抛出任意的异常：

```java
interface Task {
    void run() throws Exception;

    @SuppressWarnings("unchecked")
    static <T extends Throwable> void throwAs(Throwable t) throws T {
        throw (T) t;
    }

    static Runnable asRunnable(Task task) {
        return () -> {
            try {
                task.run();
            }
            catch (Exception e) {
                Task.<RuntimeException>throwAs(e);
            }
        };
    }
}
```

例如，以下程序运行了一个将会抛出检查型异常的线程：

```java
public class Test {
    public static void main(String[] args) {
        var thread = new Thread(Task.asRunnable(() -> {
            Thread.sleep(1000);
            System.out.println("Hello, World!");
            throw new Exception("Check this out!");
        }));
        thread.start();
    }
}
```

[limitations/DefeatCheckedExceptionChecking.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch08/limitations/DefeatCheckedExceptionChecking.java)

这有什么意义呢？正常情况下，你必须捕获`Runnable.run()`方法中的所有检查型异常。不过在这里，我们直接抛出异常，并欺骗编译器相信这不是检查型异常。

通过使用泛型类、擦除和`@SuppressWarnings`注解，我们就能够打破Java类型系统的一个基本限制。（？？？）

### 8.6.10 注意擦除后的冲突
不允许创建当泛型类型被擦除后会引发冲突的条件。下面是一个示例。假设为`Pair`类增加一个`equals()`方法：

```java
public class Pair<T> {
    ...
    public boolean equals(T value) { return first.equals(value) && second.equals(value); }
}
```

考虑`Pair<String>`，我们以为它有两个`equals()`方法：

```java
boolean equals(String) // defined in Pair<T>
boolean equals(Object) // inherited from Object
```

但是直觉把我们引入歧途——方法`boolean equals(T)`擦除后就是`boolean equals(Object)`，这会与`Object.equals()`方法发生冲突。解决方法是重新命名引发冲突的方法。

泛型规范还指出了另一个规则：“一个类不能同时作为同一个接口的两个不同参数化接口的子类型。”例如，下面的代码是非法的：

```java
class Employee implements Comparable<Employee> { ... }
class Manager extends Employee implements Comparable<Manager> { ... } // ERROR
```

其原因非常微妙：合成的桥方法将会产生冲突。实现了`Comparable<X>`的类会获得一个桥方法（见8.5.3节）：

```java
public int compareTo(Object other) { return compareTo((X) other); }
```

对于不同的类型`X`不能有两个这样的方法。

`_(:з」∠)_`

### 小结
对于泛型类`C<T>`：
* `C<int>`× `C<Integer>`√
* `a instanceof C<String>`× `a instanceof C`√
* `new T(args)`× `new C<String>(args)`√ `new C<T>(args)`√
* `new T[n]`× `new C<String>[n]`× `new C<T>[n]`× `new C[n]`√
* `T...`√ `C<String>...`√ `C<T>...`√
* `T.class`× `C<String>.class`× `C<T>.class`× `C.class`√
* `static T field;`×
* `extends Exception`× `catch(T e)`×

## 8.7 泛型类型的继承规则
使用泛型类时，需要了解有关继承的一些规则。

首先，考虑一个类和一个子类，例如`Employee`和`Manager`。`Pair<Manager>` **不是** `Pair<Employee>`的子类型。例如：

```java
Pair<Employee> buddies = new Pair<Manager>(ceo, cfo); // illegal
```

一般地，**无论`S`与`T`有什么关系，`C<S>`与`C<T>`都没有任何关系**（如下图所示）。

![Pair类之间没有继承关系](/assets/images/java-note-v1ch08-generic-programming/Pair类之间没有继承关系.png)

这一限制看起来很严格，但对于类型安全来说是必要的。假设允许将`Pair<Manager>`转换为`Pair<Employee>`。考虑以下代码：

```java
var managerBuddies = new Pair<Manager>(ceo, cfo);
Pair<Employee> employeeBuddies = managerBuddies; // illegal, but suppose it wasn't
Employee lowlyEmployee = ...;
employeeBuddies.setFirst(lowlyEmployee);
```

现在能够将CFO和一个底层员工组成一对，这对于`Pair<Manager>`来说应该是不可能的。

注释：这是泛型类型与数组之间的重要区别：可以将一个`Manager[]`数组赋给一个`Employee[]`类型的变量。不过，数组有特别的保护，上述赋值会导致`ArrayStoreException`。

另外，**总是可以将参数化类型转换为原始类型**。例如，`Pair<Employee>`是原始类型`Pair`的一个子类型。在与遗留代码交互时，这个转换是必要的。

然而，转换成原始类型可能会导致类型错误（如8.5.4节所述）。考虑这个例子：

```java
var managerBuddies = new Pair<Manager>(ceo, cfo);
Pair rawBuddies = managerBuddies; // OK
rawBuddies.setFirst(new File("...")); // only a compile-time warning
```

当通过`getFirst()`获取对象并赋给`Manager`变量时，就会抛出`ClassCastException`。这与Java引入泛型之前是一样的，只是失去了泛型编程提供的附加安全性。

最后，**泛型类可以实现或扩展其他泛型类型**。例如，`ArrayList<T>`类实现了`List<T>`接口。这意味着`ArrayList<Manager>`可以转换为`List<Manager>`。但是，如前面所见，`ArrayList<Manager>`并不是`ArrayList<Employee>`或`List<Employee>`。下图展示了这些关系。

![泛型列表类型之间的子类型关系](/assets/images/java-note-v1ch08-generic-programming/泛型列表类型之间的子类型关系.png)

## 8.8 通配符类型
死板的泛型类型系统使用起来令人很不愉快。Java设计者发明了一种巧妙（但仍然安全）的**通配符类型**(wildcard type)。

### 8.8.1 通配符概念
在通配符类型中，允许类型参数变化。

**子类型边界**(subtype bound)通配符`? extends T`限制为`T`或其子类型。例如，通配符类型`Pair<? extends Employee>`表示类型参数是`Employee`的子类的任何泛型`Pair`类，例如`Pair<Manager>`，但不能是`Pair<String>`。

假设要编写一个打印员工对的方法，如下所示：

```java
public static void printBuddies(Pair<Employee> p) {
    Employee first = p.getFirst();
    Employee second = p.getSecond();
    System.out.println(first.getName() + " and " + second.getName() + " are buddies.");
}
```

正如上一节讲到的，不能将`Pair<Manager>`传递给这个方法，这一点很受限。解决方法很简单——使用通配符类型：

```java
public static void printBuddies(Pair<? extends Employee> p)
```

注：这个例子也可以用类型变量边界实现，但边界不能限定超类型，而通配符可以（见下一节）。

`Pair<Manager>`是`Pair<? extends Employee>`的子类型，如下图所示。

![使用通配符的子类型关系](/assets/images/java-note-v1ch08-generic-programming/使用通配符的子类型关系.png)

再次考虑8.7节中的例子。使用通配符不会引起破坏：

```java
var managerBuddies = new Pair<Manager>(ceo, cfo);
Pair<? extends Employee> wildcardBuddies = managerBuddies; // OK
wildcardBuddies.setFirst(lowlyEmployee); // compile-time error
```

调用`setFirst()`是一个编译错误。要了解原因，仔细看一下类型`Pair<? extends Employee>`，其方法如下（不是真正的Java语法，只是便于理解）：

```java
? extends Employee getFirst()
void setFirst(? extends Employee)
```

调用`setFirst()`方法是不可能的！编译器只知道其参数是某个扩展了`Employee`的类型，但无法知道具体类型。编译器不能保证实参是形参的子类，因此必须拒绝所有实参（除了`null`）（注：因为Java中不存在一种类型是所有类型的子类型，但存在一个这样的特殊值`null`）。

仍然可以调用`getFirst()`方法。其返回类型是`Employee`的某个子类型。编译器不知道具体类型，但可以保证将其赋给`Employee`变量是安全的。

这就是有边界通配符背后的核心思想。现在已经有办法区分安全的访问器方法和不安全的修改器方法了。

### 8.8.2 通配符的超类边界
通配符边界与类型变量边界十分类似，但是还有一个额外的能力——可以指定**超类型边界**(supertype bound)通配符：`? super T`限制为`T`或其超类型。

这与子类型通配符的行为正好相反：可以为方法提供参数，但不能使用返回值。例如，`Pair<? super Manager>`的方法可以描述如下：

```java
? super Manager getFirst()
void setFirst(? super Manager)
```

`setFirst()`的参数是`Manager`的某个超类型（例如`Object`、`Employee`或`Manager`）。因此可以传递`Manager`或其子类的对象。

相反，如果调用`getFirst()`，不能保证返回对象的类型，只能把它赋给一个`Object`。

`Pair<Employee>`和`Pair<Object>`是`Pair<? super Manager>`的子类型，如下图所示。

![使用通配符的超类型关系](/assets/images/java-note-v1ch08-generic-programming/使用通配符的超类型关系.png)

下面是一个典型的示例。假设有一个`Manager`数组，并且想把奖金最高和最低的经理放在一个`Pair`对象中。结果的类型应该是`Pair<? super Manager>`：

```java
public static void minmaxBonus(Manager[] a, Pair<? super Manager> result) {
    if (a.length == 0) return;
    Manager min = a[0];
    Manager max = a[0];
    for (int i = 1; i < a.length; i++) {
        if (min.getBonus() > a[i].getBonus()) min = a[i];
        if (max.getBonus() < a[i].getBonus()) max = a[i];
    }
    result.setFirst(min);
    result.setSecond(max);
}
```

直观地讲，带有超类型边界的通配符允许写入泛型对象，带有子类型边界的通配符允许读取泛型对象。

下面是超类型边界的另一种应用。`Comparable`接口本身是一个泛型类型，类型变量指示了`compareTo()`方法的参数类型（在此之前，参数是一个`Object`，方法的实现中必须使用强制类型转换）。现在，可以将程序清单8-2中的`minmax()`方法声明为

```java
public static <T extends Comparable<T>> Pair<T> minmax(T[] a)
```

这样看起来比只使用`T extends Comparable`更严密，而且对于很多类都适用。例如，对于一个`String`数组，`T`就是`String`，而`String`是`Comparable<String>`的子类型。但是，处理`LocalDate`数组时会遇到一个问题。`LocalDate`实现了`ChronoLocalDate`，而`ChronoLocalDate`扩展了`Comparable<ChronoLocalDate>`，因此`LocalDate`实现的是`Comparable<ChronoLocalDate>`而不是`Comparable<LocalDate>`。

在这种情况下，可以利用超类型边界通配符来解决：

```java
public static <T extends Comparable<? super T>> Pair<T> minmax(T[] a)
```

现在，`compareTo()`方法的形式为`int compareTo(? super T)`，无论如何都可以安全地传递一个`T`类型的对象。

注释：超类型边界的另一个常见用法是作为函数式接口的参数类型。例如，`Collection`接口有一个方法`default boolean removeIf(Predicate<? super E> filter)`，用于删除所有满足给定条件的元素。例如，可以删除有奇数散列码的员工：

```java
ArrayList<Employee> staff = ...;
Predicate<Object> oddHashCode = obj -> obj.hashCode() % 2 != 0;
staff.removeIf(oddHashCode);
```

你希望能够传入一个`Predicate<Object>`而不只是`Predicate<Employee>`。超类型边界通配符使之成为可能。

### 8.8.3 无边界通配符
还可以使用无边界的通配符，例如`Pair<?>`。乍一看，这好像与原始的`Pair`类型一样。实际上，二者有很大的不同。`Pair<?>`类型有以下方法：

```java
? getFirst()
void setFirst(?)
```

`getFirst()`的返回值只能赋给一个`Object`。`setFirst()`方法不能调用，甚至不能用`Object`调用（`null`除外）。`Pair<?>`和`Pair`的本质区别在于：可以用任意`Object`调用原始`Pair`类的`setFirst()`方法。

为什么要使用这样“懦弱”的类型？它对于很简单的操作很有用。例如，下面的方法测试一个`Pair`是否包含`null`引用，它不需要实际类型：

```java
public static boolean hasNulls(Pair<?> p) {
    return p.getFirst() == null || p.getSecond() == null;
}
```

也可以通过将`hasNulls()`改为泛型方法避免使用通配符类型：

```java
public static <T> boolean hasNulls(Pair<T> p)
```

但是，使用通配符类型的版本可读性更好。

### 8.8.4 通配符捕获
下面编写一个方法来交换`Pair`的元素：

```java
public static void swap(Pair<?> p)
```

通配符不是类型变量，因此不能使用`?`作为类型。也就是说，下面的代码是非法的：

```java
? tmp = p.getFirst(); // ERROR
p.setFirst(p.getSecond());
p.setSecond(tmp);
```

这是一个问题，因为在交换时必须临时保存第一个元素。可以写一个辅助方法`swapHelper()`，如下所示：

```java
public static <T> void swapHelper(Pair<T> p) {
    T tmp = p.getFirst();
    p.setFirst(p.getSecond());
    p.setSecond(tmp);
}
```

注意，`swapHelper()`是泛型方法，而`swap()`不是——它有一个固定的`Pair<?>`类型的参数。

现在让`swap()`调用`swapHelper()`：

```java
public static void swap(Pair<?> p) { swapHelper(p); }
```

在这种情况下，`swapHelper()`方法的类型参数`T` **捕获了通配符**。

当然，这里并不是一定要使用通配符。也可以直接把`swap()`实现为一个没有通配符的泛型方法（即`swapHelper()`）。不过，考虑下面这个例子，通配符类型很自然地出现在计算中间：

```java
public static void maxminBonus(Manager[] a, Pair<? super Manager> result) {
    minmaxBonus(a, result);
    PairAlg.swapHelper(result); // OK--swapHelper captures wildcard type
}
```

在这里，通配符捕获机制是不可避免的。

通配符捕获只在非常有限的情况下是合法的。编译器必须能够保证通配符表示单个、确定的类型。例如，`ArrayList<Pair<T>>`中的`T`无法捕获`ArrayList<Pair<?>>`中的通配符——数组列表可能包含两个`Pair<?>`，其`?`分别有不同的类型。

程序清单8-3中的测试程序将前几节讨论的各种方法综合在一起，以便了解具体用法。

[程序清单8-3 pair3/PairTest3.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch08/pair3/PairTest3.java)

### 小结
与通配符类型赋值有关的所有规则本质上都是**替换原则**：只能将对象赋给相同类型或超类的变量。假设T<sub>x</sub>表示x的类型，`<`表示继承关系：`B < A`表示类`B`扩展类`A`。那么替换原则可以表述为：如果将对象x赋给变量v，则必须满足T<sub>x</sub> ≤ T<sub>v</sub>。

（1）如果`B` ≤ `A`，则`C<B>` < `C<? extends A>` < `C<?>`，`C<A>` < `C<? super B>` < `C<?>`

（2）传递参数时，是将实参赋给形参，要求T<sub>实参</sub> ≤ T<sub>形参</sub>

| 形参类型 | 实参类型 | 要求 |
| --- | --- | --- |
| `? extends T` | 无法调用 | T<sub>实参</sub> ≤ T<sub>形参</sub> ≤ T |
| `? super T` | `T`或其子类 | T<sub>实参</sub> ≤ T ≤ T<sub>形参</sub> |
| `?` | 无法调用 | T<sub>实参</sub> ≤ T<sub>形参</sub> |

（3）获取返回值时，是将返回值赋给目标变量，要求T<sub>返回值</sub> ≤ T<sub>变量</sub>

| 返回值类型 | 变量类型 | 要求 |
| --- | --- | --- |
| `? extends T` | `T`或其超类 | T<sub>返回值</sub> ≤ T ≤ T<sub>变量</sub> |
| `? super T` | `Object` | T ≤ T<sub>返回值</sub> ≤ T<sub>变量</sub> |
| `?` | `Object` | T<sub>返回值</sub> ≤ T<sub>变量</sub> |

## 8.9 反射和泛型
反射允许你在运行时分析任意对象。然而，如果对象是泛型类的实例，则得不到多少关于类型参数的信息，因为它们已经被擦除了。在下面几节中，你将学习利用反射可以获得泛型类的哪些信息。

### 8.9.1 泛型Class类
`Class`类是泛型类。例如，`String.class`实际上是`Class<String>`类的（唯一）对象。

类型参数十分有用，因为它允许`Class<T>`的方法有更具体的返回类型。

`newInstance()`方法返回这个类的一个实例，由无参数构造器获得。其返回类型为`T`，这样就省去了强制类型转换。

`getConstructor()`和`getDeclaredConstructor()`方法返回一个`Constructor<T>`对象。`Constructor`类也已经变成泛型，使得其 `newInstance()`方法有正确的返回类型。

### 8.9.2 使用Class\<T\>参数进行类型匹配
匹配泛型方法中`Class<T>`参数的类型变量有时会很有用。下面是一个经典的示例：

```java
public static <T> Pair<T> makePair(Class<T> c) throws InstantiationException, IllegalAccessException {
    return new Pair<>(c.newInstance(), c.newInstance());
}
```

如果调用`makePair(Employee.class)`，`Employee.class`是`Class<Employee>`类型的对象。`makePair()`方法的类型参数`T`将匹配`Employee`，编译器可以推断出这个方法返回一个`Pair<Employee>`。

### 8.9.3 虚拟机中的泛型类型信息
Java泛型的显著特性之一是擦除虚拟机中的泛型类型。令人惊讶的是，被擦除的类仍然保留着原先泛型的一些微弱记忆。例如，原始`Pair`类知道它源于泛型类`Pair<T>`，尽管一个`Pair`类型的对象无法区分它是构造为`Pair<String>`还是`Pair<Employee>`。

类似地，考虑以下方法：

```java
public static Comparable min(Comparable[] a)
```

这是擦除以下泛型方法得到的：

```java
public static <T extends Comparable<? super T>> T min(T[] a)
```

可以使用反射API确定：
* 这个泛型方法有一个名为`T`的类型参数
* 这个类型参数有一个子类型边界，其自身也是泛型类型
* 这个边界类型有一个通配符参数
* 这个通配符参数有一个超类型边界
* 这个泛型方法有一个泛型数组参数

换句话说，你可以重建实现者声明的泛型类和方法的所有内容。但是，你不会知道对于特定的对象或方法调用会如何解析类型参数。

为了描述泛型类型声明，使用`java.lang.reflect`包中的`Type`接口。这个接口有以下子类型：
* `Class`类，描述具体类型
* `TypeVariable`接口，描述类型变量（如`T extends Comparable<? super T>`）
* `WildcardType`接口，描述通配符（如`? super T`）
* `ParameterizedType`接口，描述泛型类或接口类型（如`Comparable<? super T>`）
* `GenericArrayType`接口，描述泛型数组（如`T[]`）

下图展示了继承层次结构。

![Type接口及其后代](/assets/images/java-note-v1ch08-generic-programming/Type接口及其后代.png)

程序清单8-4使用泛型反射API来打印给定类的有关信息。

[程序清单8-4 genericReflection/GenericReflectionTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch08/genericReflection/GenericReflectionTest.java)

如果用`Pair`类运行，将会得到：

```
class pair.Pair<T> extends java.lang.Object
public T getFirst()
public T getSecond()
public void setFirst(T)
public void setSecond(T)
```

如果用`PairTest2`程序中的`ArrayAlg`类运行，将会显示以下方法：

```
public static <T extends java.lang.Comparable> pair.Pair<T> minmax(T[])
```

### 8.9.4 类型字面值
有时，你希望由值的类型决定程序的行为。例如，在持久化存储机制中，你可能希望用户指定一种方法来保存特定类的对象。通常的实现方法是将`Class`对象与动作关联。然而，对于泛型类，擦除会带来问题。例如，既然`ArrayList<Integer>`和`ArrayList<String>`都擦除为同一个原始类型`ArrayList`，如何让它们有不同的动作呢？

注：如8.6.2节所述，类型查询只适用于原始类型，因此也不能使用`if (a instanceof ArrayList<String>)`。

这里有一个技巧，在某些情况下可以解决这个问题。可以捕获`Type`接口（上一节介绍过）的实例。如下构造一个匿名子类：

```java
var type = new TypeLiteral<ArrayList<Integer>>(){}; // note the {}
```

`TypeLiteral`构造器会捕获泛型超类型：

```java
class TypeLiteral {
    public TypeLiteral() {
        Type parentType = getClass().getGenericSuperclass();
        if (parentType instanceof ParameterizedType paramType)
            type = paramType.getActualTypeArguments()[0];
        else
            throw new UnsupportedOperationException("Construct as new TypeLiteral<...>(){}");
    }
    ...
}
```

注：对于上面的例子，`getClass()`是匿名子类，`parentType`是`TypeLiteral<ArrayList<Integer>>`，`type`是`ArrayList<Integer>`。

我们无法从一个对象得到泛型类型——已经被擦除。不过，正如在上一节看到的，字段、方法参数和超类的泛型类型还留存在虚拟机中。

CDI和Guice等注入框架(injection framework)就使用类型字面值来控制泛型类型的注入。程序清单8-5给出了一个更简单的例子。给定一个对象，枚举它的字段（其泛型类型是可获得的），并查找关联的格式化动作：通过空格分隔值来格式化`ArrayList<Integer>`，通过将字符连接成字符串来格式化`ArrayList<Character>`，所有其他数组列表都由`ArrayList.toString()`格式化。

[程序清单8-5 genericReflection/TypeLiterals.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch08/genericReflection/TypeLiterals.java)

关于对Java泛型所有知识的全面讨论，可以查看Angelika Langer提供的常见问题列表：<https://angelikalanger.com/GenericsFAQ/JavaGenericsFAQ.html>
