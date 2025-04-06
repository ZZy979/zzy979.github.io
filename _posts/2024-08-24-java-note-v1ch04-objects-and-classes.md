---
title: 《Java核心技术》笔记 卷I 第4章 对象和类
date: 2024-08-24 23:12:13 +0800
categories: [Java, Core Java]
tags: [java, object-oriented programming, class, instance field, method, date and time, constructor, "null", this pointer, static method, factory method, overloading, initialization, random number, record, package, import statement, class path, jar file]
---
本章主要介绍：
* 面向对象程序设计入门；
* 如何创建Java标准库中类的对象；
* 如何编写自己的类。

## 4.1 面向对象程序设计概述
**面向对象程序设计**(object-oriented programming, OOP)是当今主流的程序设计范型，它取代了20世纪70年代的过程式程序设计(procedural programming)技术。由于Java是面向对象的，你必须熟悉OOP才能够很好地使用Java。

面向对象的程序是由对象组成的，每个对象都有对用户公开的特定功能和隐藏的实现。传统的结构化程序设计通过设计一系列的过程（即算法）来求解问题。

### 4.1.1 类
**类**(class)指定了如何构造对象。可以将类想象为饼干模具，将对象想象为饼干。由类**构造**(construct)对象的过程称为创建类的**实例**(instance)。正如前面看到的，用Java编写的所有代码都在某个类中。

**封装**(encapsulation)是处理对象的一个关键概念。从形式上看，封装就是将数据和行为组合在一个对象中，并对使用者隐藏实现细节。对象中的数据称为**实例字段**(instance field)，操作数据的过程称为**方法**(method)。每个特定对象（类的实例）的实例字段都有特定的值，这些值的集合就是这个对象的当前**状态**(state)。只要调用对象的方法，其状态就有可能发生改变。

实现封装的关键在于，绝对不能让其他类的方法访问这个类的实例字段。程序只能通过对象的方法与对象数据进行交互。这意味着一个类完全可以改变存储数据的方式，只要仍旧使用同样的方法操作数据，其他对象就不会知道也不用关心。

OOP的另一个原则会让自定义Java类变得更容易：可以通过**扩展**(extend)其他类来构建新类。事实上，Java有一个“宇宙级超类”`Object`，所有其他类都扩展自这个类。

扩展一个已有的类时，新类将具有被扩展类的全部方法和字段。只需提供适用于新类的方法和实例字段。通过扩展一个类得到另一个类的概念称为**继承**(inheritance)，详细内容参见下一章。

### 4.1.2 对象
要使用OOP，一定要清楚对象的三个关键特征：
* 对象的**行为**(behavior)：可以对这个对象做哪些操作，或者可以调用哪些方法？
* 对象的**状态**(state)：调用这些方法时，对象会如何响应？
* 对象的**标识**(identity)：如何区分具有相同行为和状态的不同对象？

### 4.1.3 识别类
在传统的过程式程序中，从顶部的`main()`函数开始编写程序。设计面向对象的系统时，没有所谓的“顶部”。答案是：首先识别类，然后为每个类添加方法。

识别类的一个简单经验法则是在分析问题的过程中寻找名词，而方法对应动词。

例如，在订单处理系统中，有这样一些名词：商品(item)、订单(order)、收货地址(shipping address)、付款(payment)、账户(account)。从这些名词可以得到类`Item`、`Order`等。

接下来寻找动词。商品可以**添加**到订单中，订单可以**发货**或**取消**，订单**支付**货款。对于每个动词，例如“添加”、“发货”、“取消”或者“支付”，要识别出负责执行动作的对象。例如，商品被添加到订单中，`add()`应该是`Order`类的一个方法，它接受一个`Item`对象作为参数（即`order.add(item)`）。

### 4.1.4 类之间的关系
类之间最常见的关系有
* 依赖("use-a")
* 聚合("has-a")
* 继承("is-a")

**依赖**(dependence)是最明显、最普遍的关系。例如，`Order`类需要使用`Account`类来检查信用状态。如果一个类的方法使用或操作另一个类的对象，就说一个类依赖于另一个类。

应当尽可能减少相互依赖的类。如果类`A`不知道类`B`的存在，就无需关心`B`的任何改变。用软件工程的术语来说，就是最小化类之间的耦合(coupling)。

**聚合**(aggregation)/**关联**(association)意味着类`A`的对象包含类`B`的对象。例如，`Order`对象包含一些`Item`对象。

**继承**(inheritance)表示一个特殊的类和一个一般的类之间的关系。例如，`RushOrder`类继承`Order`类。特殊的`RushOrder`类包含用于优先处理的特殊方法以及计算运费的不同方法，而其他的方法（例如添加商品、生成账单等）都是从`Order`类继承来的。一般而言，如果类`D`继承（扩展）了类`C`，就会继承类`C`的方法，另外还会有一些额外的功能（下一章将详细讨论）。

很多程序员使用UML（Unified Modeling Language，统一建模语言）绘制**类图**(class diagrams)，用来描述类之间的关系。下图就是这样一个例子。

![类图](/assets/images/java-note-v1ch04-objects-and-classes/类图.png)

下面是UML中最常见的箭头样式。

![表达类关系的UML符号](/assets/images/java-note-v1ch04-objects-and-classes/表达类关系的UML符号.png)

## 4.2 使用预定义类
### 4.2.1 对象和对象变量
要使用对象，首先必须构造对象并指定其初始状态，然后调用对象的方法。

在Java中，使用**构造器**(constructor)构造新实例。构造器是一种特殊方法，用来构造并初始化对象。下面来看一个例子。Java标准库包含一个`Date`类，其对象描述一个时间点，例如1999年12月31日23:59:59 GMT。

构造器与类同名。因此`Date`类的构造器叫做`Date`。要构造一个对象，需要在构造器前面加上`new`运算符。例如：`new Date()`，这个表达式构造了一个新的`Date`对象，并初始化为当前的日期和时间。可以将这个对象传递给一个方法：

```java
System.out.println(new Date());
```

`Date`类有一个`toString()`方法，返回日期的字符串表示。可以像这样调用新构造的`Date`对象的`toString()`方法：

```java
String s = new Date().toString();
```

通常，构造的对象需要多次使用。只需将对象存放在一个变量中：

```java
Date rightNow = new Date();
```

下图显示了对象变量`rightNow`，它引用了新构造的对象。

![创建一个新对象](/assets/images/java-note-v1ch04-objects-and-classes/创建一个新对象.png)

对象和对象变量之间有一个重要区别。例如，语句

```java
Date startTime; // startTime doesn't refer to any object
```

定义了一个对象变量`startTime`，它可以引用`Date`类型的对象。但是一定要认识到：变量`startTime` **不是一个对象**。实际上它还没有引用任何对象，因此不能调用任何`Date`方法，否则将导致编译错误。

必须先初始化`startTime`变量。可以让它引用一个新构造的对象：

```java
startTime = new Date();
```

也可以让它引用一个已有的对象：


```java
startTime = rightNow;
```

现在，这两个变量**引用同一个对象**（见下图）。

![引用同一个对象的变量](/assets/images/java-note-v1ch04-objects-and-classes/引用同一个对象的变量.png)

重要的是要认识到：对象变量实际并不包含一个对象，而只是**引用**一个对象。

在Java中，任何对象变量的值都是指向存储在其他地方的某个对象的引用。`new`运算符的返回值也是一个引用。

可以显式地将对象变量设置为`null`，表示它目前没有引用任何对象。4.3.6节将更详细地讨论`null`。

C++注释：很多人错误地认为Java的对象变量相当于C++的引用。然而，C++中没有`null`引用，而且引用不能赋值。**应当把Java的对象变量看作C++的对象指针**。例如：

```java
Date rightNow; // Java
```

实际上等同于

```cpp
Date* rightNow; // C++
```

一旦建立了这种关联，一切就都清楚了。

如果把一个变量拷贝到另一个变量，两个变量就引用同一个对象——它们是同一个对象的指针。Java的`null`引用等同于C++的`nullptr`。

所有的Java对象都在堆上。当一个对象包含另一个对象变量时，它只是包含另一个对象的指针。

注：C++的值语义保证了对象不会为`null`，但带来了拷贝开销问题，指针也很容易出错。Java的对象引用解决了拷贝开销问题，但又需要检查`null`引用。

### 4.2.2 Java库中的LocalDate类
`Date`类的实例有一个状态——一个特定的时间点。时间是用距离一个固定时间点的毫秒数（可正可负）表示的，这个时间点就是所谓的**纪元**(epoch)，即1970年1月1日00:00:00 UTC。

但是，`Date`类对于处理人类记录日期的日历信息并不是很有用。类库设计者决定将保存时间与给时间点命名分开。因此，Java标准库分别包含了两个类：表示时间点的`Date`类，以及用日历表示法表示日期的`LocalDate`类。Java 8引入了另外一些类来处理日期和时间的不同方面，参见卷II第6章。

将时间与日历分开是一种很好的面向对象设计。通常，最好使用不同的类表示不同的概念。

不要使用构造器来构造`LocalDate`类的对象。而应当使用静态**工厂方法**(factory method)代表你调用构造器。表达式`LocalDate.now()`构造一个新对象，表示当前日期。可以指定年、月、日来构造一个表示特定日期的对象：`LocalDate.of(1999, 12, 31)`。

可以用`getYear()`、`getMonthValue()`和`getDayOfMonth()`方法得到年、月和日：

```java
LocalDate newYearsEve = LocalDate.of(1999, 12, 31);
int year = newYearsEve.getYear(); // 1999
int month = newYearsEve.getMonthValue(); // 12
int day = newYearsEve.getDayOfMonth(); // 31
```

`plusDays()`方法返回距当前对象指定天数的新的`LocalDate`：

```java
LocalDate aThousandDaysLater = newYearsEve.plusDays(1000);
year = aThousandDaysLater.getYear(); // 2002
month = aThousandDaysLater.getMonthValue(); // 09
day = aThousandDaysLater.getDayOfMonth(); // 26
```

### 4.2.3 修改器和访问器方法
在调用`newYearsEve.plusDays(1000)`之后，`newYearsEve`并没有改变。`plusDays()`方法会返回一个新的`LocalDate`对象，原来的对象不做任何改动。即`plusDays()`方法没有**修改**(mutate)调用这个方法的对象。

Java库的较早版本有另一个处理日历的类`GregorianCalendar`。可以如下为这个类表示的日期增加1000天：

```java
GregorianCalendar someDay = new GregorianCalendar(1999, 11, 31);
  // odd feature of that class: month numbers go from 0 to 11
someDay.add(Calendar.DAY_OF_MONTH, 1000);
```

与`LocalDate.plusDays()`方法不同，调用这个方法后，`someDay`对象的状态会改变：

```java
year = someDay.get(Calendar.YEAR); // 2002
month = someDay.get(Calendar.MONTH) + 1; // 09
day = someDay.get(Calendar.DAY_OF_MONTH); // 26
```

会修改对象的方法称为**修改器方法**(mutator method)，例如`GregorianCalendar.add()`。相反，只访问对象而不修改对象的方法称为**访问器方法**(accessor methods)，例如`LocalDate.getYear()`和`GregorianCalendar.get()`。

C++注释：在C++中带有`const`后缀的方法是访问器方法；没有声明为`const`的方法默认为修改器方法。但在Java中，访问器与修改器在语法上没有特殊区别。

下面看一个应用`LocalDate`类的程序。这个程序显示当前月的日历，像这样：

```
Mon Tue Wed Thu Fri Sat Sun
                          1
  2   3   4   5   6   7   8
  9  10  11  12  13  14  15
 16  17  18  19  20  21  22 
 23  24  25  26* 27  28  29
 30
```

当前日期用星号(*)标记。这个程序需要知道如何计算某月份的天数以及一个给定日期是星期几。程序清单4-1给出了完整的程序。

[程序清单4-1 CalendarTest/CalendarTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch04/CalendarTest/CalendarTest.java)

## 4.3 自定义类
现在来学习如何编写更复杂的应用所需要的类。这些类通常没有`main()`方法，而有自己的实例字段和方法。要构建一个完整的程序，需要组合多个类，其中只有一个类有`main()`方法。

### 4.3.1 Employee类
在Java中，最简单的类定义形式为

```java
class ClassName  {
    field1
    field2
    ...
    constructor1
    constructor2
    ...
    method1
    method2
    ...
}
```

下面看一个非常简单的`Employee`类，在编写薪资管理系统时可能会用到：

```java
class Employee {
    // instance fields
    private String name;
    private double salary;
    private LocalDate hireDay;

    // constructor
    public Employee(String n, double s, int year, int month, int day) {
        name = n;
        salary = s;
        hireDay = LocalDate.of(year, month, day);
    }

    // a method
    public String getName() {
        return name;
    }

    // more methods
    ...
}
```

这个类的实现细节分别在稍后的几节中介绍。首先来看程序清单4-2，这个程序展示了`Employee`类的实际使用。

[程序清单4-2 EmployeeTest/EmployeeTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch04/EmployeeTest/EmployeeTest.java)

在这个程序中，构造了一个`Employee`数组，并填入了3个`Employee`对象。接下来，使用`Employee`类的`raiseSalary()`方法将每个员工的薪水提高5%。最后，调用`getName()`、`getSalary()`和`getHireDay()`方法打印每个员工的信息。

注意，这个示例程序包含**两个**类：`Employee`类和带有`public`访问修饰符的`EmployeeTest`类。`EmployeeTest`类包含`main()`方法，其中包含上面描述的代码。

源文件名是EmployeeTest.java，这是因为文件名必须与`public`类的名字匹配。一个源文件中只能有一个公有类，但可以有任意数目的非公有类。

注：源文件中的非公有类只能是包访问（不带修饰符），只有内部类可以用`protected`和`private`修饰。

接下来，使用命令`javac EmployeeTest.java`编译这个源文件，编译器将在目录中创建两个类文件：EmployeeTest.class和Employee.class。

然后启动这个程序：`java EmployeeTest`

### 4.3.2 使用多个源文件
在程序清单4-2中，一个源文件包含了两个类。许多程序员习惯将每个类放在一个单独的源文件中。例如，将`Employee`类放在文件Employee.java中，将`EmployeeTest`类放在EmployeeTest.java中。

如果这样组织文件，有两种编译程序的方法。可以使用通配符调用Java编译器：

```shell
javac Employee*.java
```

这样，所有与通配符匹配的源文件都将被编译成类文件。或者直接使用以下命令：

```shell
javac EmployeeTest.java
```

第二种方式并没有显式地编译Employee.java。然而，当Java编译器发现EmployeeTest.java中使用了`Employee`类时，它会查找名为Employee.class的文件。如果没有找到，就会自动搜索Employee.java并编译这个文件。另外，如果Employee.java的版本（更新时间）比已有的Employee.class文件更新，Java编译器就会**自动地**重新编译这个文件。

注释：如果熟悉UNIX的make工具，可以认为Java编译器内置了make功能。

### 4.3.3 剖析Employee类
下面对`Employee`类进行剖析。这个类包含一个构造器和4个方法：

```java
public Employee(String n, double s, int year, int month, int day)
public String getName()
public double getSalary()
public LocalDate getHireDay()
public void raiseSalary(double byPercent)
```

这个类的所有方法都被标记为`public`。关键字`public`意味着任何类的方法都可以调用这些方法。

`Employee`类有3个实例字段，用来存放要操作的数据。

```java
private String name;
private double salary;
private LocalDate hireDay;
```

关键字`private`确保只有`Employee`类本身的方法能够访问这些实例字段，其他类的方法都不能读写这些字段。

注释：可以用`public`标记实例字段，但这是一种很不好的做法。`public`实例字段允许程序的任何部分对其进行读取或修改，这就完全破坏了封装。因此，强烈建议将所有实例字段标记为`private`。

注意，有两个实例字段本身就是对象（的引用）：`name`是`String`对象，`hireDay`是`LocalDate`对象。这十分常见：类经常包含类类型的实例字段。

### 4.3.4 从构造器开始
下面看`Employee`类的构造器：

```java
public Employee(String n, double s, int year, int month, int day) {
    name = n;
    salary = s;
    hireDay = LocalDate.of(year, month, day);
}
```

可以看到，构造器与类同名，没有返回值。构造`Employee`类的对象时，构造器将被调用，以初始化实例字段。

构造器只能用`new`运算符来调用。不能对一个已经存在的对象调用构造器来重新设置实例字段。例如，

```java
james.Employee("James Bond", 250000, 1950, 1, 1) // ERROR
```

将产生编译错误。

C++注释：Java构造器的工作方式与C++相同。但是要记住，所有Java对象都是在堆中构造的，构造器必须和`new`结合使用：

```cpp
Employee number007("James Bond", 100000, 1950, 1, 1); // C++, not Java
```

警告：不要定义与实例字段同名的局部变量，否则会**遮蔽**(shadow)同名的实例字段。例如：

```java
public class Main {
    public static void main(String[] args) {
        A a = new A(888);
        a.f(); // prints "123", not "888"
    }
}

class A {
    private int x;

    public A(int x) {
        this.x = x;
    }

    public void f() {
        int x = 123; // shadows this.x
        System.out.println(x);
    }
}
```

注：一种解决方法是在访问实例字段时添加`this.`前缀，详见4.3.7节。

### 4.3.5 用var声明局部变量
从Java 10起，如果可以从初始值推导出变量的类型，就可以用`var`关键字声明局部变量，而无需指定类型。例如：

```java
var harry = new Employee("Harry Hacker", 50000, 1989, 10, 1);
```

这样可以避免重复写类型名`Employee`。

注意`var`关键字只能用于方法中的局部变量。参数和字段的类型必须声明。

### 4.3.6 使用null引用
对象变量包含一个对象的引用，或者特殊值`null`（表示没有引用任何对象）。听上去这是一种处理特殊情况的便捷机制。但是**使用`null`值时要非常小心！**

如果对`null`值调用方法，就会产生`NullPointerException`异常。

```java
LocalDate rightNow = null;
String s = rightNow.toString(); // NullPointerException
```

这是一个很严重的错误。如果没有捕获异常，程序就会终止。正常情况下，程序并不捕获这种异常，而是依赖程序员一开始就不要造成异常（因为这种错误是完全可以避免的）。

注释：当程序因`NullPointerException`异常终止时，栈轨迹会显示问题出现在哪一行代码中。从Java 17开始，错误消息会包含有`null`值的变量或方法名。例如，在以下调用中：

```java
String s = e.getHireDay().toString();
```

错误消息会告诉你是`e`为`null`还是`getHireDay()`返回了`null`。

定义一个类时，最好清楚地知道哪些字段可能为`null`。在我们的例子中，我们不希望`name`或`hireDay`字段为`null`（不用担心`salary`字段，因为它是基本类型，不可能是`null`）。

`hireDay`字段保证不是`null`，因为它初始化为一个新的`LocalDate`对象。但是，如果调用构造器时参数`n`是`null`，`name`就是`null`。

对此有两种解决方法。“宽容的”方法是把`null`参数转换为一个适当的非`null`值：

```java
if (n == null) name = "unknown";
else name = n;
```

`Objects`类为此提供了一个便利方法：

```java
name = Objects.requireNonNullElse(n, "unknown");
```

“严格的”方法是拒绝`null`参数：

```java
name = Objects.requireNonNull(n, "The name cannot be null");
```

如果`n`为`null`，就会产生`NullPointerException`异常。乍一看这种方法好像不太有用，不过有两个好处：
1. 异常报告会提供问题的描述。
2. 异常报告会准确地指出问题所在的位置。否则`NullPointerException`异常可能会发生在其他地方（例如使用`name`字段的其他方法），而很难追踪到真正导致问题的构造器参数。

注释：如果接受一个对象引用作为构造器参数，就要问问自己：是不是真的希望表示一个可有可无的值。如果不是，那么“严格的”方法更合适。

### 4.3.7 隐式参数和显式参数
方法会操作对象并访问其实例字段。例如，`Employee.raiseSalary()`将调用这个方法的对象的`salary`字段设置为一个新值。调用`number007.raiseSalary(5)`将`number007.salary`字段的值增加5%。

`raiseSalary()`方法有两个参数。第一个参数称为**隐式参数**(implicit parameter)，是出现在方法名前的`Employee`类型的对象（如`number007`）。第二个参数是位于方法名后面括号中的数值（如`5`），是一个**显式参数**(explicit parameter)。

在每个方法中，关键字`this`表示隐式参数。如果愿意，可以如下改写`raiseSalary()`方法：

```java
public void raiseSalary(double byPercent)  {
    double raise = this.salary * byPercent / 100;
    this.salary += raise;
}
```

C++注释：在C++中，通常在类外定义方法。在Java中，所有的方法都在类内定义。

### 4.3.8 封装的优点
最后再仔细看一下非常简单的`getName()`、`getSalary()`和`getHireDay()`方法。这些都是典型的访问器方法。由于它们只返回实例字段的值，因此又称为**字段访问器**(field accessor)。

如果将`name`、`salary`和`hireDay`字段标记为公有，而不是编写独立的访问器方法，不是更容易些吗？不过，`name`字段是只读的，一旦在构造器中设置，就没有办法修改它。这样可以保证`name`字段不会受到破坏。虽然`salary`字段不是只读的，但它只能用`raiseSalary()`方法修改。特别是如果这个值出现错误，只需要调试这一个方法就可以了。

如果想要获得或设置实例字段的值，那么需要提供三项内容：
* 一个私有的实例字段
* 一个公有的访问器方法
* 一个公有的修改器方法

这样做要比提供一个简单的公有实例字段复杂些，但有很多明显的好处。

首先，可以改变内部实现，而不影响该类方法之外的任何代码。例如，如果将存储姓名的字段改为`firstName`和`lastName`，那么`getName()`方法可以改为返回`firstName + " " + lastName`。

另外，修改器方法可以执行错误检查，而直接对字段赋值的代码不会这么做。例如，`setSalary()`方法可以检查薪水不会小于0。

警告：注意不要编写返回可变对象引用的访问器方法。如果`Employee`的`hireDay`字段类型是`Date`：

```java
class Employee {
    private Date hireDay;
    ...
    public Date getHireDay() {
        return hireDay; // BAD
    }
    ...
}
```

与`LocalDate`不同，`Date`类有修改器方法`setTime()`，可以设置毫秒数。`Date`对象是可变的，这一点破坏了封装性！考虑下面这段流氓代码：

```java
Employee harry = ...;
Date d = harry.getHireDay();
double tenYearsInMilliseconds = 10 * 365.25 * 24 * 60 * 60 * 1000;
d.setTime(d.getTime() - (long) tenYearsInMilliseconds);
// let's give Harry ten years of added seniority
```

出错的原因很微妙：`d`和`harry.hireDay`引用同一个对象（见下图）。对`d`调用修改器方法会自动改变这个`Employee`对象的私有状态！

![返回可变实例字段的引用](/assets/images/java-note-v1ch04-objects-and-classes/返回可变实例字段的引用.png)

如果需要返回一个可变对象的引用，首先应该对它进行克隆。下面是修正后的代码：

```java
public Date getHireDay()  {
    return (Date) hireDay.clone(); // OK
}
```

### 4.3.9 基于类的访问权限
一个类的方法可以访问**这个类的所有对象**的私有数据。例如，考虑比较两个员工的`equals()`方法：

```java
class Employee {
    ...
    public boolean equals(Employee other) {
        return name.equals(other.name);
    }
}
```

这个方法不仅访问了隐式参数的私有字段，还访问了`other`的私有字段。这是合法的。

### 4.3.10 私有方法
实现一个类时，应该将所有实例字段都设置为私有的。尽管大多数方法都是公有的，但私有方法在某些情况下是有用的。有时，可能希望将一个计算代码分解成若干个独立的辅助方法，这些辅助方法不应该成为公有接口的一部分。最好将这样的方法实现为私有的。

在Java中，要实现私有方法，只需将关键字`public`改为`private`即可。

只要方法是私有的，类的设计者就可以确信它不会在别处使用，所以可以将其删除。如果方法是公有的，就不能简单地将其删除，因为可能会有其他代码依赖它。

### 4.3.11 final实例字段
可以将实例字段定义为`final`。这样的字段必须在构造器中初始化（否则编译器会报错），之后不能再修改这个字段。例如：

```java
class Employee {
    private final String name;
    ...
}
```

`final`修饰符对于基本类型或者**不可变类**类型的字段尤其有用。而对于可变类，`final`修饰符可能会令人困惑。例如，考虑以下字段

```java
private final StringBuilder evaluations;
```

在构造器中初始化为

```java
evaluations = new StringBuilder();
```

`final`关键字只意味着`evaluations`变量不会再引用另一个`StringBuilder`对象，但这个对象可以修改：

```java
public void giveGoldStar() {
    evaluations.append(LocalDate.now() + ": Gold star!\n");
}
```

注：`final`变量也存在同样的问题。

## 4.4 静态字段和方法
下面讨论一下`static`修饰符的含义。

### 4.4.1 静态字段
如果将一个字段定义为`static`，这个字段就是**静态字段**(static field)。可以认为静态字段属于类，而不是单个对象。例如，假设要为每个员工分配唯一的id，可以为`Employee`类添加实例字段`id`和静态字段`nextId`：

```java
class Employee {
    private static int nextId = 1;
    private int id;
    ...
}
```

每个`Employee`对象都有自己的`id`字段，但这个类的所有实例共享一个`nextId`字段。即使没有`Employee`对象，静态字段`nextId`也存在。

在构造器中，为新`Employee`对象分配下一个可用的id，然后将其加1：

```java
id = nextId;
nextId++;
```

这等价于

```java
this.id = Employee.nextId;
Employee.nextId++;
```

### 4.4.2 静态常量
静态变量使用得比较少，但静态常量却很常用。例如，`Math`类定义了静态常量`PI`：

```java
public class Math {
    public static final double PI = 3.14159265358979323846;
    ...
}
```

在程序中可以用`Math.PI`来访问这个常量。

另一个已经多次使用的静态常量是`System.out`。

前面曾多次提到，最好不要有公有字段，因为谁都可以修改。不过，公有常量（即`public final`字段）没问题。

### 4.4.3 静态方法
静态方法是不操作对象的方法。例如，`Math.pow()`就是一个静态方法，它并不使用任何`Math`对象来完成计算。

可以认为静态方法是没有隐式参数(`this`)的方法。静态方法不能访问实例字段，但可以访问静态字段。下面是一个示例：

```java
public static int advanceId() {
    int r = nextId; // obtain next available id
    nextId++;
    return r;
}
```

注：也可以直接`return nextId++;`

可以通过类名调用这个方法：

```java
int n = Employee.advanceId();
```

注释：使用对象调用静态方法也是合法的（例如`harry.advanceId()`），但不建议这样做。

在下面两种情况下使用静态方法：
* 方法不需要访问实例字段，所需参数都通过显式参数提供（例如`Math.pow()`）。
* 方法只需要访问类的静态字段（例如`Employee.advanceId()`）。

C++注释：“静态”(static)一词有一段不寻常的历史。起初，C语言引入关键字`static`是为了表示退出一个块后依然存在的局部变量。在这种情况下，“静态”一词是有意义的：变量一直保留，当再次进入这个块时它仍然存在。随后，`static`在C语言中有了第二种含义：表示不能从其他文件访问的全局变量和函数。复用关键字`static`只是为了避免引入一个新的关键字。最后，C++第三次复用了这个关键字，与之前的含义完全无关：表示属于类而不是任何特定对象的变量和函数。这个含义与Java中的相同。

### 4.4.4 工厂方法
静态方法还有另外一种常见的用途。`LocalDate`和`NumberFormat`等类使用静态**工厂方法**(factory method)来构造对象。你已经见过工厂方法`LocalDate.now()`和`LocalDate.of()`。可以使用`NumberFormat`类的工厂方法得到不同样式的格式化对象：

```java
NumberFormat currencyFormatter = NumberFormat.getCurrencyInstance();
NumberFormat percentFormatter = NumberFormat.getPercentInstance();
double x = 0.1;
System.out.println(currencyFormatter.format(x)); // prints $0.10
System.out.println(percentFormatter.format(x)); // prints 10%
```

为什么`NumberFormat`类不使用构造器来创建对象呢？有两个原因：
* 无法命名构造器。构造器的名字必须与类名相同，但是我们想用两个不同的名字分别得到货币格式和百分比格式的实例。
* 使用构造器时，无法改变所构造对象的类型，而工厂方法可以返回子类对象。`NumberFormat`的工厂方法实际上返回`DecimalFormat`类的对象，这是继承`NumberFormat`的子类。

### 4.4.5 main方法
`main()`方法也是一个静态方法。

```java
public class Application {
    public static void main(String[] args) {
        // construct objects here
        ...
    }
}
```

提示：每个类都可以有一个`main()`方法。这是为类添加演示代码的一个方便的技巧。例如，可以在`Employee`类中添加`main()`方法：

```java
class Employee {
    ...
    public static void main(String[] args) { // unit test
        var e = new Employee("Romeo", 50000, 2003, 3, 31);
        e.raiseSalary(10);
        System.out.println(e.getName() + " " + e.getSalary());
    }
}
```

要看`Employee`类的演示，只需执行`java Employee`。如果`Employee`类是一个更大的应用的一部分，可以使用命令`java Application`运行这个应用，`Employee`类的`main()`方法就不会被执行。

程序清单4-3中的程序包含了`Employee`类的一个简单版本，其中有一个静态字段`nextId`和一个静态方法`advanceId()`。

[程序清单4-3 StaticTest/StaticTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch04/StaticTest/StaticTest.java)

注意，`Employee`类也有一个`main()`方法用于单元测试。尝试运行`java Employee`和`java StaticTest`执行两个`main()`方法。

## 4.5 方法参数
首先来回顾在编程语言中描述如何将参数传递给方法（或函数）的术语。**按值调用**(call by value)表示方法接收的是调用者提供的值，而**按引用调用**(call by reference)表示方法接收的是调用者提供的变量的**位置**。(location)。因此，方法可以**修改**按引用传递的变量的值，而不能修改按值传递的变量的值。

**★Java总是采用按值调用。** 这意味着方法不能修改传递给它的参数变量的内容。

例如，假设一个方法试图将参数值变为3倍：

```java
public static void tripleValue(double x) { // doesn't work
    x = 3 * x;
}
```

调用这个方法：

```java
double percent = 10;
tripleValue(percent);
```

不过，这并不起作用。调用这个方法之后，`percent`的值仍然是10。具体过程如下：
1. `x`初始化为`percent`值的拷贝，即10。
2. `x`乘以3后等于30，但`percent`仍然是10（见下图）。
3. 方法结束后，参数变量`x`不再使用。

![修改参数变量无效](/assets/images/java-note-v1ch04-objects-and-classes/修改参数变量无效.png)

不过，有两种类型的方法参数：基本类型（数字、布尔值）和对象引用。已经看到，方法不可能修改基本类型的参数，而对象参数就不同了。例如，可以很容易地实现一个方法将一个员工的工资增至3倍：

```java
public static void tripleSalary(Employee x) { // works 
    x.raiseSalary(200);
}
```

当调用

```java
var harry = new Employee("Harry", 50000);
tripleSalary(harry);
```

具体过程如下：
1. `x`初始化为`harry`值（对象引用）的拷贝，与`harry`引用同一个对象。
2. 对这个对象调用`raiseSalary()`方法，`x`和`harry`同时引用的那个`Employee`对象的工资提高了200%。
3. 方法结束后，参数变量`x`不再使用。但对象变量`harry`继续引用那个工资增至3倍的员工对象（见下图）。

![修改参数引用的对象有效](/assets/images/java-note-v1ch04-objects-and-classes/修改参数引用的对象有效.png)

可以看到，实现改变对象参数状态的方法是完全可以的，实际上也相当常见。原因很简单：方法得到的是对象引用的副本，原来的引用和这个副本都引用同一个对象。

有些程序员认为Java对于对象采用的是按引用调用，这是错误的。由于这种误解很常见，下面给出一个反例来详细说明。

尝试编写一个交换两个`Employee`对象的方法：

```java
void swap(Employee x, Employee y) { // doesn't work
    Employee temp = x;
    x = y;
    y = temp;
}
```

如果Java对于对象采用的是按引用调用，调用下面的方法后`a`应该引用Bob，`b`应该引用Alice。

```java
var a = new Employee("Alice", ...);
var b = new Employee("Bob", ...);
swap(a, b);
```

但是，这个方法并没有改变存储在变量`a`和`b`中的对象引用。`swap()`方法的参数`x`和`y`初始化为这两个对象引用的**副本**，这个方法交换的是这两个副本。最终，白费力气。方法结束后，参数变量`x`和`y`被丢弃了，变量`a`和`b`仍然引用这个方法调用之前所引用的对象（如下图所示）。

![交换参数变量无效](/assets/images/java-note-v1ch04-objects-and-classes/交换参数变量无效.png)

这说明Java对于对象采用的不是按引用调用，而是**对象引用是按值传递的**。

下面总结了Java中的方法参数：
* 方法不能修改基本类型的参数。
* 方法可以改变对象参数的状态。
* 方法不能让对象参数引用新的对象。

程序清单4-4中的程序展示了这几点。

[程序清单4-4 ParamTest/ParamTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch04/ParamTest/ParamTest.java)

## 4.6 对象构造
### 4.6.1 重载
类可以有多个构造器。例如：

```java
var messages = new StringBuilder();
var todoList = new StringBuilder("To do:\n");
```

这种功能叫做**重载**(overloading)。如果多个方法有相同的名字、不同的参数，便产生了重载。调用方法时，编译器必须通过将形参类型与实参类型进行匹配来选出正确的方法（这个过程称为**重载解析**(overloading resolution)）。如果没有匹配的方法，或者有多个匹配的方法（有歧义），就会产生编译错误。例如：

```java
void f(int x, double y) {...}
void f(double x, int y) {...}

f(1);        // Cannot resolve method call
f(1, "2.5"); // Cannot resolve method call
f(1, 2);     // Ambiguous method call
```

注释：Java允许重载任何方法，而不只是构造器。要完整地描述一个方法，需要指定方法名和参数类型，这叫做方法的**签名**(signature)。返回类型不是方法签名的一部分。也就是说，不能有两个名字和参数类型都相同但返回类型不同的方法。

### 4.6.2 默认字段初始化
如果在构造器中没有显式地设置一个字段，就会自动设置为默认值：数值为0，布尔值为`false`，对象引用为`null`。

注释：这是字段与局部变量的一个重要区别。方法中的局部变量必须显式地初始化，而未初始化的字段会自动初始化为默认值。

### 4.6.3 无参数的构造器
很多类都包含无参数的构造器，会将创建的对象的状态设置为适当的默认值。例如，下面是`Employee`类的一个无参数构造器：

```java
public Employee() {
    name = "";
    salary = 0;
    hireDay = LocalDate.now();
}
```

如果编写的类没有构造器，编译器就会提供一个无参数构造器。这个构造器将所有实例字段设置为默认值（如上一节所述）。（注：这样的构造器等价于`public ClassName() {}`）

如果类提供了至少一个构造器，但没有提供无参数构造器，那么构造对象时不提供任何参数(`new ClassName()`)就是不合法的。如果需要，就必须提供无参数构造器。

### 4.6.4 显式字段初始化
可以在类定义中直接给字段赋值。例如：

```java
class Employee {
    private String name = "";
    ...
}
```

这个赋值在执行构造器之前完成。如果一个类的所有构造器都需要把某个实例字段设置为同一个值，这种语法尤其有用。

初始值不一定是常量，也可以是方法调用。例如，对于程序清单4-3中的`Employee`：

```java
class Employee {
    private static int nextId;
    private int id = advanceId();
    ...
    private static int advanceId() {...}
    ...
}
```

### 4.6.5 参数名
在编写很简单的构造器时，可能很难想出参数名。通常选择单个字母的参数名：

```java
public Employee(String n, double s) {
    name = n;
    salary = s;
}
```

但缺点是只有阅读代码才能知道参数`n`和`s`的含义。

有些程序员在每个参数前加上前缀 "a" ：

```java
public Employee(String aName, double aSalary) {
    name = aName;
    salary = aSalary;
}
```

还有一种常用的技巧：参数与实例字段同名，用`this`访问实例字段。例如：

```java
public Employee(String name, double salary) {
    this.name = name;
    this.salary = salary;
}
```

### 4.6.6 调用另一个构造器
关键字`this`引用方法的隐式参数。不过，这个关键字还有另一种含义。如果**构造器的第一条语句**形如`this(...)`，这个构造器将调用同一个类的另一个构造器。下面是一个典型的例子：

```java
public Employee(double s) {
    this("Employee #" + nextId, s); // calls Employee(String, double)
    nextId++;
}
```

当调用`new Employee(60000)`时，`Employee(double)`构造器将调用`Employee(String, double)`构造器。

这样只需要写一次公共构造代码。

### 4.6.7 初始化块
前面已经介绍过两种初始化实例字段的方法：在构造器中设置值，在声明中赋值。

Java还有第三种机制，称为**初始化块**(initialization block)。类声明可以包含任意的代码块。构造这个类的对象时，这些块就会执行。例如：

```java
class Employee {
    private static int nextId;
    private int id;
    private String name;
    private double salary;

    // object initialization block
    {
        id = nextId;
        nextId++;
    }

    public Employee(String n, double s) {
        name = n;
        salary = s;
    }

    public Employee() {
        name = "";
        salary = 0;
    }
    ...
}
```

在这个示例中，无论使用哪个构造器构造对象，`id`字段都会在对象初始化块中初始化。首先运行初始化块，然后才执行构造器的主体部分。

这种机制不是必需的，也不常见。通常直接将初始化代码放在构造器中。

初始化实例字段有多种途径，下面是调用构造器时的具体处理步骤：
1. 如果构造器的第一行调用了另一个构造器，则用提供的参数执行另一个构造器。
2. 否则，
   * 所有实例字段初始化为其默认值（`0`、`false`或`null`）。
   * 按照在类声明中出现的顺序，执行所有字段初始化和初始化块。
3. 执行构造器的主体。

注：例如，对于下面的类`A`：

```java
class A {
    private int x = f();

    {
        System.out.println("init block");
        x = 0;
    }

    public A(int x) {
        System.out.println("A(int)");
        this.x = x;
    }

    public A() {
        this(123);
        System.out.println("A()");
    }

    public static int f() {
        System.out.println("f()");
        return 42;
    }

    public int getX() {
        return x;
    }
}

```

调用`new A(888)`将输出

```
f()
init block
A(int)
```

调用`new A()`将输出

```
f()
init block
A(int)
A()
```

当然，最好精心地组织初始化代码，以便其他程序员能够轻松地理解。

要初始化静态字段，可以提供初始值或者使用静态初始化块。前面已经介绍过第一种机制：

```java
private static int nextId = 1;
```

静态初始化块用关键字`static`标记。例如，我们希望员工id从一个小于10000的随机整数开始。

```java
private static Random generator = new Random();

// static initialization block
static {
    nextId = generator.nextInt(10000);
}
```

静态初始化发生在类首次加载时。与实例字段一样，除非将静态字段显式地设置成其他值，否则为默认值。所有的静态字段初始化以及静态初始化块都按照类声明中出现的顺序执行。

这个例子使用了`Random`类来生成随机数。从Java 17开始，`java.util.random`包提供了考虑多种因素的强算法的实现，参见API文档。

程序清单4-5中的程序展示了本节讨论的很多特性：重载构造器、调用另一个构造器、无参数构造器、对象初始化块、静态初始化块、实例字段初始化。

[程序清单4-5 ConstructorTest/ConstructorTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch04/ConstructorTest/ConstructorTest.java)

### 4.6.8 对象析构和finalize方法
有些面向对象的编程语言（特别是C++）有显式的析构器(destructor)方法，其中放置当对象不再使用时需要的清理代码。在析构器中，最常见的操作是回收分配给对象的内存。由于Java有自动垃圾收集，不需要人工回收内存，所以Java不支持析构器。

当然，某些对象使用了除内存之外的资源，例如文件。在这种情况下，当资源不再需要时，将其回收就十分重要。

如果资源一旦使用完就需要立即关闭，应当提供一个`close()`方法来完成必要的清理工作。可以在对象使用完时调用这个`close()`方法。第7章将介绍如何确保自动调用这个方法。

如果可以等到虚拟机退出，可以使用`Runtime.addShutdownHook()`方法添加一个“关闭钩”(shutdown hook)。从Java 9起，可以使用`Cleaner`类注册一个当对象不再可达时要执行的动作。在实际中这些情况很少见。这两种方法的细节参见API文档。

警告：不要使用`finalize()`方法来完成清理。这个方法原本是要在垃圾收集器清理对象之前调用。但是，你并不能知道这个方法到底什么时候调用，而且该方法已经被废弃。

## 4.7 记录
有时，数据就只是数据，而面向对象程序设计提供的数据隐藏有些碍事。考虑描述平面上一个点的`Point`类，有x和y坐标。当然，可以创建一个类：

```java
class Point {
    private final double x;
    private final double y;

    public Point(double x, double y) { this.x = x; this.y = y; }
    public getX() { return x; }
    public getY() { return y; }
    public String toString() { return "Point[x=%d, y=%d]".formatted(x, y); }
    // More methods ...
}
```

但是，隐藏`x`和`y`，然后通过getter方法获得这些值，这种做法确实有好处吗？

为了更简洁地定义这种类，Java 14引入了预览特性“记录”，最终版本在Java 16中发布。

### 4.7.1 记录概念
**记录**(record)是一种特殊形式的类，其状态不可变，且公共可读。可以如下将`Point`定义为一个记录：

```java
record Point(double x, double y) {}
```

其结果是具有以下实例字段、构造器和访问器方法的类：

```java
class Point {
    private final double x;
    private final double y;

    public Point(double x, double y) { this.x = x; this.y = y; }
    public double x() { return x; }
    public double y() { return y; }
}
```

注：记录类似于Scala的`case`类。

在Java中，记录的实例字段称为**组件**(component)。

注意，访问器方法名为`x()`和`y()`，而不是`getX()`和`getY()`。

```java
var p = new Point(3, 4);
System.out.println(p.x() + " " + p.y());
```

除了字段访问器方法，每个记录还自动定义了3个方法：`toString()`、`equals()`和`hashCode()`。下一章会详细介绍这些方法。

可以为记录添加自己的方法：

```java
record Point(double x, double y) {
    public double distanceFromOrigin() { return Math.hypot(x, y); }
}
```

与类一样，记录可以有静态字段和方法：

```java
record Point(double x, double y) {
    public static Point ORIGIN = new Point(0, 0);
    public static double distance(Point p, Point q) {
        return Math.hypot(p.x - q.x, p.y - q.y);
    }
    ...
}
```

但是，不能为记录添加实例字段：

```java
record Point(double x, double y) {
    private double r; // ERROR
    ...
}
```

警告：记录的实例字段自动为`final`。不过，它们可能是可变对象的引用：

```java
record PointInTime(double x, double y, Date when) {}
```

这样，记录实例将是可变的：

```java
var pt = new PointInTime(0, 0, new Date());
pt.when().setTime(0);
```

如果希望记录实例是不可变的，字段就不要使用可变的类型。

提示：对于完全由一组变量表示的不可变数据，使用记录而不是类。如果数据是可变的，或者表示方式可能随时间改变，则使用类。记录更易读、更高效，而且在并发程序中更安全。

### 4.7.2 构造器：标准、自定义和简洁
自动定义的设置所有实例字段的构造器称为**标准构造器**(canonical constructor)。

还可以定义另外的**自定义构造器**(custom constructor)。这种构造器的第一条语句必须调用另一个构造器，所以最终会调用标准构造器。例如：

```java
record Point(double x, double y) {
    public Point() { this(0, 0); }
}
```

如果标准构造器需要完成额外的工作，那么可以提供自己的实现：

```java
record Range(int from, int to) {
    public Range(int from, int to) {
        if (from <= to) {
            this.from = from;
            this.to = to;
        }
        else {
            this.from = to;
            this.to = from;
        }
    }
}
```

不过，实现标准构造器时，建议使用**简洁**(compact)形式：

```java
record Range(int from, int to) {
    public Range { // Compact form
        if (from > to) { // Swap the bounds
            int temp = from;
            from = to;
            to = temp;
        }
    }
}
```

不用指定参数列表，主体是标准构造器的“前奏”。不能在简洁构造器中读取或修改实例字段。

[程序清单4-6 RecordTest/RecordTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch04/RecordTest/RecordTest.java)

## 4.8 包
Java允许使用**包**(package)将类组织起来。借助包可以方便地组织你的代码，并将自己的代码与其他人提供的代码库分开。下面将介绍如何使用和创建包。

### 4.8.1 包名
使用包的主要原因是确保类名的唯一性。假设两个程序员不约而同地提供了`Employee`类，只要他们将自己的类放置在不同的包中，就不会产生冲突。事实上，为了保证包名的绝对唯一性，可以**使用因特网域名的逆序形式作为包名**，对于不同的项目使用不同的子包。例如，考虑本书作者的域名[horstmann.com](https://horstmann.com/)，按逆序写就转换为包名`com.horstmann`。然后可以追加一个项目名，如`com.horstmann.corejava`。如果再把`Employee`类放在这个包中，那么这个类的“完全限定”名就是`com.horstmann.corejava.Employee`。

注释：从编译器的角度来看，嵌套的包之间没有任何关系。例如，`java.util`包与`java.util.jar`包毫无关系，每个包都是独立的类集合。

### 4.8.2 类的导入
一个类可以使用所属包中的所有类，以及其他包中的公有类。

所属包中的其他类可以直接使用（不需要导入）。有两种方式访问另一个包中的公有类。第一种方式是使用**完全限定名**(fully qualified name)，也就是包名后面跟着类名。例如：

```java
java.time.LocalDate today = java.time.LocalDate.now();
```

这显然很繁琐。更简单、更常用的方式是使用`import`语句。`import`语句的目的是提供一种简写形式来引用包中的类。一旦添加了`import`语句，就不必再写类的全名了。

可以导入一个特定的类或者整个包。`import`语句应该位于源文件的顶部（但位于`package`语句的后面）。例如，可以使用下面的语句导入`java.time`包中所有的类：

```java
import java.time.*;
```

然后就可以使用

```java
LocalDate today = LocalDate.now();
```

而不需要包前缀。也可以导入一个包中特定的类：

```java
import java.time.LocalDate;
```

但是，需要注意只能使用星号(`*`)导入一个包，而不能使用`import java.*`或`import java.*.*`导入以`java`为前缀的所有包。

注：`java.lang`包会被默认导入。

大多数情况下，只需导入需要的包，不用过多关注。但在发生命名冲突时就要注意包了。例如，`java.util`和`java.sql`包都有`Date`类。假设在程序中导入了这两个包：

```java
import java.util.*;
import java.sql.*;
```

此时如果使用`Date`类，就会出现编译错误：

```java
Date today; // ERROR--java.util.Date or java.sql.Date?
```

因为编译器无法确定你想使用的是哪个`Date`类。可以增加一个特定的`import`语句来解决这个问题：

```java
import java.util.*;
import java.sql.*;
import java.util.Date;
```

如果这两个`Date`类都需要使用，就使用完全限定名：

```java
var startTime = new java.util.Date();
var today = new java.sql.Date(...);
```

C++注释：C++程序员有时会将`import`与`#include`弄混。实际上，这两者并没有共同之处。在C++中，必须使用`#include`包含来包含外部头文件中的声明。而在Java中，通过给出完整的类名可以完全不使用`import`语句。`import`语句唯一的好处是简捷，可以使用更简短的名字来引用一个类。

在C++中，与包机制类似的是命名空间。可以认为Java中的`package`和`import`语句类似于C++中的`namespace`和`using`指令。

### 4.8.3 静态导入
有一种形式的`import`语句允许导入类的静态字段和方法。例如，在源文件顶部添加以下语句：

```java
import static java.lang.Math.*;
```

就可以使用`sqrt(pow(x, 2) + pow(y, 2))`而不是`Math.sqrt(Math.pow(x, 2) + Math.pow(y, 2))`。

还可以导入特定的方法或字段：

```java
import static java.lang.System.out;
```

### 4.8.4 将类加入包中
要将类放入包中，需要在源文件开头使用`package`语句指定包名。例如，程序清单4-8中的文件Employee.java开头是这样的：

```java
package com.horstmann.corejava;

public class Employee {
    ...
}
```

如果源文件中没有`package`语句，那么这个源文件中的类就属于**无名包**(unnamed package)（也叫做**默认包**）。到目前为止，所有示例中的类都在无名包中。

必须将源文件放到与完整包名匹配的子目录中。例如，`com.horstmann.corejava`包中的所有源文件应该位于子目录com/horstmann/corejava中。编译器将类文件也放在相同的目录结构中。

程序清单4-7和4-8中的程序分别放在两个包中：`PackageTest`类属于无名包，`Employee`类属于`com.horstmann.corejava`包。因此，目录结构如下所示：

```
basedir/
    PackageTest.java
    PackageTest.class
    com/
        horstmann/
            corejava/
                Employee.java
                Employee.class
```

要编译这个程序，只需切换到基目录并运行以下命令

```shell
javac PackageTest.java
```

编译器会自动地查找文件com/horstmann/corejava/Employee.java并进行编译。

下面看一个更加实际的例子。在这里没有使用无名包，而是将类分别放在不同的包中（`com.horstmann.corejava`和`com.mycompany`）：

```
basedir/
    com/
        horstmann/
            corejava/
                Employee.java
                Employee.class
        mycompany/
            PayrollApp.java
            PayrollApp.class
```

在这种情况下，仍然要从基目录编译和运行类：

```shell
javac com/mycompany/PayrollApp.java
java com.mycompany.PayrollApp
```

警告：编译器在编译源文件时**不检查**目录结构。例如，假设一个源文件开头有`package com.mycompany;`，即使这个源文件不在子目录com/mycompany下也可以编译。如果它不依赖于其他包，就可以通过编译。但是，最终的程序将无法运行，除非先将所有类文件移到正确的位置。如果包与目录不匹配，虚拟机就找不到类（注：因此在上面的示例中，不能在basedir/com/mycompany目录下运行`java PayrollApp`）。

[程序清单4-7 PackageTest/PackageTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch04/PackageTest/PackageTest.java)

[程序清单4-8 PackageTest/com/horstmann/corejava/Employee.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch04/PackageTest/com/horstmann/corejava/Employee.java)

### 4.8.5 包访问
前面已经见过访问修饰符`public`和`private`。标记为`public`的实体可以被任何类使用，标记为`private`的实体只能被定义它们的类使用。如果没有指定访问修饰符，这个实体（类、方法或字段）可以被**同一个包**中的所有方法访问，即**包访问**(package access)。

考虑程序清单4-2，这个程序没有将`Employee`定义为公有类，因此只有同一个包（这里是无名包）中的其他类（例如`EmployeeTest`）可以访问这个类。对于类来说，这种默认方式是合理的。而对于变量来说就不适宜了，显然这会破坏封装性。例如`java.awt.Window`类的`warningString`字段。

这可能会成为一个问题。默认情况下，包不是封闭的实体。也就是说，任何人都可以向包中添加更多的类。恶意或无知的程序员就可以利用包访问添加能修改这些字段的代码。

从1.2版本开始，JDK的实现者修改了类加载器，明确地禁止加载包名以`java.`开头的用户自定义类。当然，用户自定义类无法从这种保护中受益。另一种机制是让JAR文件将包声明为**密封的**(sealed)，以防止第三方修改，但这种机制已经过时。现在应当使用模块来封装包。卷II的第9章将详细讨论模块。

### 4.8.6 类路径
在前面已经看到，类存储在文件系统的子目录中，路径必须与包名匹配。

另外，类文件也可以存储在JAR文件中。一个JAR文件包含多个压缩格式的类文件和子目录。在程序中用到第三方库（例如Maven依赖库）时，通常会得到一个或多个需要包含的JAR文件。第11章将介绍如何创建自己的JAR文件。

提示：JAR文件使用ZIP格式组织文件和子目录。可以使用任何ZIP工具查看JAR文件。

**类路径**(class path)是Java编译器(`javac`)和虚拟机(`java`)搜索类文件的路径的集合（类似于Python的模块搜索路径`sys.path`）。类路径可以包含：
* 类文件的基目录（例如/home/user/classdir或C:\classdir）
* 当前目录(.)
* JAR文件（例如/home/user/archives/archive.jar或C:\archives\archive.jar）

在UNIX中，类路径的各项之间用冒号(:)分隔：

```
/home/user/classdir:.:/home/user/archives/archive.jar
```

在Windows中，则用分号(;)分隔：

```
C:\classdir;.;C:\archives\archive.jar
```

从Java 6开始，可以为JAR文件目录指定通配符(*)，目录中的所有.jar文件（不包括.class文件）都包含在类路径中。在UNIX中，`*`必须转义以防止shell扩展。例如`/home/user/classdir:.:/home/user/archives/'*'`或者`C:\classdir;.;C:\archives\*`。

Java标准库的类会被自动搜索，所以不必显式地包含在类路径中。

警告：Java编译器总是在当前目录中查找文件，但Java虚拟机只有在类路径包含 "." 目录时才查看当前目录。如果没有设置类路径，那么没有问题，因为默认的类路径就是 "." 目录。但是如果设置了类路径却忘记包含 "." 目录，那么程序可以通过编译，但不能运行。

类路径所列出的目录和JAR文件是搜索类的起始点。考虑一个类路径示例：

```
/home/user/classdir:.:/home/user/archives/archive.jar
```

假设Java虚拟机要搜索`com.horstmann.corejava.Employee`类的类文件。首先要查看Java标准库。显然在那里找不到，所以转而查看类路径，会查找以下文件：
* /home/user/classdir/com/horstmann/corejava/Employee.class
* 当前目录中的com/horstmann/corejava/Employee.class
* /home/user/archives/archive.jar中的com/horstmann/corejava/Employee.class

编译器查找文件要比虚拟机复杂得多。如果引用了一个类而没有指定这个类的包，那么编译器首先需要查找包含这个类的包。它会查询所有的`import`指令作为可能的来源。例如，假设源文件包含指令

```java
import java.util.*;
import com.horstmann.corejava.*;
```

并且源代码引用了`Employee`类。编译器将尝试查找`java.lang.Employee`（因为总是会默认导入`java.lang`包）、`java.util.Employee`、`com.horstmann.corejava.Employee`和当前包中的`Employee`。它会在类路径**所有位置**中搜索以上**每个类**。如果找到了一个以上的类，就会产生编译错误（完全限定类名必须是唯一的，所以`import`语句的次序并不重要）。

编译器的任务不止这些。它还要查看源文件是否比类文件新。如果是这样，那么源文件就会自动地重新编译。由于只能导入其他包中的公有类，一个源文件只能包含一个公有类，并且文件名和公有类名必须匹配。因此，编译器很容易定位公有类的源文件。不过，还可以从当前包中导入非公有类，这些类可能定义在与类名不同的源文件中。因此，如果从当前包中导入一个类，编译器就要搜索当前包中的所有源文件，以确定哪个源文件定义了这个类。

注：
* 具体来说，在上面的例子中，编译器需要在下表列出的位置查找`Employee`类。必须在恰好一个位置找到，否则将报错找不到类或有冲突。假设真实位置是/home/user/classdir/com/horstmann/corejava/Employee.class，则对应打√的单元格。

| 类名 | 标准库 | /home/user/classdir | 当前目录 | /home/user/archives/archive.jar |
| --- | --- | --- | --- | --- |
| `java.lang.Employee` | | | | |
| `java.util.Employee` | | | | |
| `com.horstmann.corejava.Employee` | | √ | | |
| 当前包`Employee` | | | | |

* 导入整个包的`import`语句数量决定了上表的行数，类路径包含的路径数量决定了列数。因此，如果导入特定的类而不是整个包，编译器就不需要查找类的包名，相当于上表只有一行。

### 4.8.7 设置类路径
最好使用`-classpath`（或`-cp`，或Java 9以后`--class-path`）选项指定类路径：

```shell
java -classpath /home/user/classdir:.:/home/user/archives/archive.jar MyProg
```

整个命令必须写在一行中。将这种长命令放在shell脚本或批处理文件中是个不错的主意。

另一种方法是通过`CLASSPATH`环境变量来设置类路径。在bash中，命令如下：

```shell
export CLASSPATH=/home/user/classdir:.:/home/user/archives/archive.jar
```

在Windows CMD中，命令如下：

```shell
set CLASSPATH=C:\classdir;.;C:\archives\archive.jar
```

直到退出shell，类路径设置均有效。

注：另见[《从命令行构建Java项目》]({% post_url 2025-04-04-build-java-project-from-command-line %})。

## 4.9 JAR文件
在将应用程序打包时，你希望向用户提供单一的文件，而不是一个包含大量类文件的目录。Java归档(Java archive, JAR)文件就是为此目的而设计的。JAR文件既可以包含类文件，也可以包含诸如图像和声音等其他类型的文件。另外，JAR文件是使用ZIP格式压缩的。

### 4.9.1 创建JAR文件
可以使用`jar`工具制作JAR文件（在默认的JDK安装中，这个工具位于$jdk/bin目录）。最常用的创建新的JAR文件命令语法如下：

```shell
jar cvf jarFileName file1 file2 ...
```

例如：

```shell
jar cvf CalculatorClasses.jar *.class icon.gif
```

使用`t`选项列出JAR文件的内容：

```shell
jar tf MyArchive.jar
```

使用`u`选项更新已有的JAR文件（添加或替换文件）：

```shell
jar uf MyArchive.jar newfile...
```

`jar`命令的选项类似于UNIX `tar`命令，完整列表参见文档：<https://docs.oracle.com/en/java/javase/17/docs/specs/man/jar.html>

### 4.9.2 清单文件
除了类文件和其他资源，每个JAR文件还包含一个**清单**(manifest)文件，用于描述归档文件的特殊特性。

清单文件名为MANIFEST.MF，位于JAR文件的一个特殊的META-INF子目录中。清单文件包含多个条目(entry)，这些条目被分组为多个节(section)，节与节之间用空行分开。第一节称为主节，作用于整个JAR文件。随后的条目可以指定命名实体（例如单个文件、包或URL）的属性，它们都必须以一个`Name`条目开始。例如：

```
Manifest-Version: 1.0
Created-By: My Company

Name: Woozle.class
SHA1-Digest: 265da9c51a9ea05ded00206478c802235774eeee

Name: com/mycompany/mypkg/
Sealed: true
```

警告：清单文件的最后一行必须以换行符结束，否则将无法正确地读取清单文件。

要创建一个包含清单的JAR文件，使用`m`选项指定清单文件名。例如：

```shell
jar cfm MyArchive.jar manifest.mf com/mycompany/mypkg/*.class
```

要更新已有JAR文件的清单，使用`u`和`m`选项指定要增加的部分（自动与已有清单文件合并）：

```shell
jar ufm MyArchive.jar manifest-additions.mf
```

注释：有关JAR和清单文件格式的更多信息参见 <https://docs.oracle.com/en/java/javase/17/docs/specs/jar/jar.html> 。

注：关于在JAR文件中加载资源文件，详见5.9.3节。

### 4.9.3 可执行JAR文件
可以使用`jar`命令的`e`选项指定程序的入口点，即通常调用`java`执行程序时指定的主类：

```shell
jar cvfe MyProgram.jar com.mycompany.mypkg.MainAppClass com/mycompany/mypkg/*.class
```

或者在清单文件中指定程序的主类：

```
Main-Class: com.mycompany.mypkg.MainAppClass
```

不要添加.class扩展名。

无论使用哪种方法，用户可以通过下面的命令来启动程序：

```shell
java -jar MyProgram.jar
```

注：
* `java -jar`命令会自动将JAR文件添加到类路径。
* 如果清单文件没有指定主类，就不能使用`-jar`选项执行JAR文件。这种情况下，可以将JAR文件添加到类路径，并在`java`命令中指定主类：

```shell
java -cp MyProgram.jar com.mycompany.mypkg.MainAppClass
```

### 4.9.4 多版本JAR文件
随着模块和包强封装的引入，有些之前可以访问的内部API不再可用。这可能要求库提供者为不同Java版本发布不同的代码。为此，Java 9引入了**多版本JAR**(multi-release JAR)。

为了保证向后兼容，特定于版本的类文件放在META-INF/versions目录中：

```
MyProgram.jar
    Application.class
    BuildingBlocks.class
    Util.class
    META-INF/
        MANIFEST.MF（包含一行Multi-Release: true）
        versions/
            9/
                Application.class
                BuildingBlocks.class
            10/
                BuildingBlocks.class
```

假设`Application`类使用了`CssParser`类，那么遗留版本（根目录下）的Application.class文件可以使用`com.sun.javafx.css.CssParser`，而Java 9版本使用`javafx.css.CssParser`（因为JavaFX的API发生了变化）。

Java 8完全不知道META-INF/versions目录，只会加载遗留的类。而Java 9读取这个JAR文件时，则会使用新版本。

要添加不同版本的类文件，可以使用`--release`选项：

```shell
jar uf MyProgram.jar --release 9 Application.class
```

要从头构建一个多版本JAR文件，首先使用`javac`编译器的`--release`选项面向不同JDK版本编译，并使用`-d`选项指定输出目录；之后使用`jar`命令的`-C`选项，对于每个版本切换到不同的类文件目录：

```shell
javac -d bin/8 --release 8 Application.java
javac -d bin/9 --release 9 Application.java
jar cf MyProgram.jar -C bin/8 . --release 9 -C bin/9 Application.class
```

`--release`选项是Java 9新增的。在旧版本中，需要使用`-source`、`-target`和`-bootclasspath`选项。

多版本JAR并不是指库版本(version)，而是JDK版本(release)。多版本JAR的唯一目的是使你的某个特定版本的程序或库能够用于多个JDK版本（例如MyProgram-1.0.jar既支持Java 8也支持Java 9）。如果增加了功能或者改变了API，就应当提供一个新版本的JAR（例如MyProgram-1.1.jar）。

### 4.9.5 关于命令行选项的说明
JDK的命令行选项传统上使用单个短横线加多字母选项名，例如：

```shell
java -jar ...
javac -Xlint:unchecked -classpath ...
```

但`jar`命令是个例外，遵循经典的`tar`命令选项格式，没有短横线：

```shell
jar cvf ...
```

注：`jar`命令也可以加短横线，例如`jar -cvf ...`

从Java 9开始，Java工具开始转向一种更常用的选项格式：多字母选项前加两个短横线，常用的选项有单字母缩写。例如，调用Linux `ls`命令可以带有“人类可读”选项：`ls --human-readable`或`ls -h`。

从Java 9起，可以使用`--version`而不是`-version`、`--class-path`而不是`-classpath`，`--module-path`选项有缩写`-p`。

详细内容可以参见JEP 293：<https://openjdk.org/jeps/293>。作者还提出要标准化选项参数。带`--`的多字母选项的参数用空格或=分隔：

```shell
javac --class-path /home/user/classdir ...
javac --class-path=/home/user/classdir ...
```

单字母选项的参数可以用空格分隔，或者直接跟在选项后面：

```shell
javac -p moduledir ...
javac -pmoduledir ...
```

警告：后一种方式目前不能使用，而且通常也不是一个好主意。

`jar`命令无参数的单字母选项可以组合在一起：

```shell
jar -cvf MyProgram.jar -e mypackage.MyProgram */*.class
```

警告：目前其他命令不能使用这种方式，这肯定会带来混淆。假设`javac`有一个`-c`选项，那么`java -cp`是指`-c -p`还是`-cp`？

可以使用`jar`命令的长选项，单字母选项也可以不组合。因此下面两条命令与上面的等价：

```shell
jar --create --verbose --file MyProgram.jar --main-class mypackage.MyProgram */*.class
jar -c -v -f MyProgram.jar -e mypackage.MyProgram */*.class
```

## 4.10 文档注释
JDK包含一个很有用的工具，叫做`javadoc`，它可以由源文件生成HTML文档。事实上，Java在线API文档就是通过对Java标准库源代码运行`javadoc`生成的。这是一种很好的方法，因为可以将代码和注释放在一个地方。

### 4.10.1 注释的插入
`javadoc`工具从下面几项中提取信息：
* 模块
* 包
* 公有类和接口
* 公有和受保护字段
* 公有和受保护构造器和方法

可以（而且应该）为以上各个特性编写注释。注释放置在所描述特性的上方，以`/**`开始，以`*/`结束。

文档注释包含**标记**(tag)，后面跟着**自由格式文本**(free-form text)。标记以`@`开始，如`@since`或`@param`。

自由格式文本的第一句应该是概要说明。`javadoc`工具自动地将这些句子提取出来生成概要页。

在自由格式文本中，可以使用HTML元素，例如用于强调的`<em>...</em>`、用于着重强调的`<strong>...</strong>`、用于列表的`<ul>`和`<li>`以及用于包含图像的`<img .../>`。要输入等宽代码，使用`{@code ...}`而不是`<code>...</code>`，这样就不需要转义代码中的`<`字符了。

注释：如果文档注释中包含到其他文件（例如图像）的链接，应该将这些文件放到源代码目录下的doc-files子目录中。`javadoc`工具会将doc-files目录复制到文档目录中。在链接中需要使用这个目录，例如`<img src="doc-files/uml.png" alt="UML diagram"/>`。

### 4.10.2 类注释
类注释必须放在`import`语句之后、`class`定义之前。下面是一个类注释的例子：

```java
/**
* A {@code Card} object represents a playing card, such
* as "Queen of Hearts". A card has a suit (Diamond, Heart,
* Spade or Club) and a value (1 = Ace, 2 ... 10, 11 = Jack,
* 12 = Queen, 13 = King)
*/
public class Card {
    ...
}
```

在IntelliJ IDEA中的渲染效果如下：

![类注释](/assets/images/java-note-v1ch04-objects-and-classes/类注释.png)

注释：没有必要在每一行的前面都添加`*`。不过，大部分IDE都会自动添加`*`。

### 4.10.3 方法注释
方法注释必须放在所描述的方法之前。除了通用标记之外，还可以使用以下标记：
* `@param variable description` 描述一个参数。描述可以跨多行，并可以使用HTML标记。一个方法的所有`@param`标记必须放在一起。
* `@return description` 描述返回值。描述可以跨多行，并可以使用HTML标记。
* `@throws class description` 描述方法抛出的异常及条件。

下面是一个方法注释的例子：

```java
/**
* Raises the salary of an employee.
* @param byPercent the percentage by which to raise the salary (e.g., 10 means 10%)
* @return the amount of the raise
*/
public double raiseSalary(double byPercent) {
    double raise = salary * byPercent / 100;
    salary += raise;
    return raise;
}
```

![方法注释](/assets/images/java-note-v1ch04-objects-and-classes/方法注释.png)

### 4.10.4 字段注释
只需要对公有字段（通是静态常量）增加文档注释。例如：

```java
/**
* The "Hearts" card suit
*/
public static final int HEARTS = 1;
```

![字段注释](/assets/images/java-note-v1ch04-objects-and-classes/字段注释.png)

### 4.10.5 通用注释
标记`@since text`表示引入这个特性的版本。例如`@since 1.7.1`。

类文档注释中可以使用下面的标记：
* `@author name` 表示作者。可以有多个`@author`标记，每个对应一个作者。
* `@version text` 表示当前版本。

通过`@see`和`@link`标记可以使用`javadoc`文档其他部分或外部文档的超链接。

标记`@see reference`添加一个超链接，可以用于类和方法，`reference`可以是以下之一：

```
package.class#feature label
<a href="...">label</a>
"text"
```

第一种情况是最有用的。只要提供类、方法或字段的名字，`javadoc`就在文档中插入一个超链接。例如：

```
@see com.horstmann.corejava.Employee#raiseSalary(double)
```

如果省略包名则表示当前包，省略类名则表示当前类。注意，必须使用`#`而不是`.`分隔类名和方法（或字段）名。

如果`@see`标记后面跟着`<`字符，就需要指定一个超链接，可以链接到任何URL。例如：

```
@see <a href="www.horstmann.com/corejava.html">The Core Java home page</a>
```

在这两种情况下，都可以指定一个可选的标签，将显示为链接文本。如果省略标签，用户看到的就是目标代码名或URL。

如果`@see`标记后面跟着`"`字符，就显示指定的文本。例如：

```
@see "Core Java 2 volume 2"
```

可以为一个特性添加多个`@see`标记，但必须将它们放在一起。

如果愿意，可以在文档注释中的任何位置放置指向其他类或方法的超链接，只需插入特殊标记`{@link package.class#feature label}`。这里的特性描述规则与`@see`标记相同。

从Java 9起，可以使用`{@index entry}`标记为搜索框添加一个条目。

### 4.10.6 包注释
类、方法和字段的注释可以直接放置在Java源文件中。但是，要生成包注释，就需要在每个包目录中添加一个单独的文件。有两个选择：
* 提供一个名为package-info.java的Java文件。这个文件必须包含一个文档注释，后面是一个`package`语句。不能包含更多的代码或注释。
* 提供一个名为package.html的HTML文件。`<body>...</body>`之前的所有文本会被提取出来。

### 4.10.7 注释提取
假设希望将HTML文件放在目录docDirectory下。执行以下步骤：
1. 切换到包含要生成文档的源文件的基目录。
2. 对于单个包，运行命令：

```shell
javadoc -d docDirectory nameOfPackage
```

对于多个包，运行：

```shell
javadoc -d docDirectory nameOfPackage1 nameOfPackage2 ...
```

对于无名包，则应该运行：

```shell
javadoc -d docDirectory *.java
```

关于其他选项，参见`javadoc`工具的在线文档：
* <https://docs.oracle.com/en/java/javase/17/docs/specs/man/javadoc.html>
* <https://docs.oracle.com/en/java/javase/17/javadoc/javadoc.html>

## 4.11 类设计技巧
**1.一定要保证数据私有。** 这是最重要的，绝对不要破坏封装性。

**2.一定要初始化数据。** Java不会初始化局部变量，但是会初始化对象的实例字段。不要依赖于默认值，而是应该显式地初始化所有变量。

**3.不要在类中使用过多的基本类型。** 应该将多个相关的基本类型字段替换为其他的类。例如，可以使用一个名为`Address`的新类替换`Customer`类中的以下实例字段：

```java
private String street;
private String city;
private String state;
private int zip;
```

**4.不是所有字段都需要单独的访问器和修改器。** 你可能需要获得和设置员工的工资。而一旦构造了员工对象，肯定不需要修改雇佣日期。

**5.分解职责过多的类。** 如果明显可以将一个复杂的类分解成两个概念上更为简单的类，就应该将其分解。这是一个糟糕的设计示例：

```java
// bad design
public class CardDeck {
    private int[] value;
    private int[] suit;
    public CardDeck() { ... }
    public void shuffle() { ... }
    public int getTopValue() { ... }
    public int getTopSuit() { ... }
    public void draw() { ... }
}
```

实际上，这个类实现了两个独立的概念：一副牌（包含洗牌和抽牌方法）和一张牌（包含查看面值和花色的方法）。应该引入一个表示一张牌的`Card`类。现在有两个类，每个类完成自己的职责：

```java
public class CardDeck {
    private Card[] cards;
    public CardDeck() { ... }
    public void shuffle() { ... }
    public Card getTop() { ... }
    public void draw() { ... }
}

public class Card {
    private int value;
    private int suit;
    public Card(int aValue, int aSuit) { ... }
    public int getValue() { ... }
    public int getSuit() { ... }
}
```

**6.类名和方法名要能够体现它们的职责。** 变量名应该能够反映其含义，类也应该如此。一个好的惯例是：类名应当是一个名词(`Order`)、形容词+名词(`RushOrder`)或动名词+名词(`BillingAddress`)。对于方法来说，要遵循标准惯例：访问器方法以小写`get`开头，修改器方法以小写`set`开头。

**7.优先使用不可变的类。** 可变对象的问题在于，如果多个线程同时修改一个对象，结果是不可预测的。如果类是不可变的，就可以安全地在多个线程间共享对象。因此，要尽可能让类是不可变的，计算会生成新值而不是更新原来的值（如`LocalDate.plusDays()`）。当然，并不是所有类都应该是不可变的。如果员工加薪时让`raiseSalary()`方法返回一个新的`Employee`对象会很奇怪。
