---
title: 《Java核心技术》笔记 卷I 第6章 接口、Lambda表达式和内部类
date: 2024-09-19 22:56:02 +0800
categories: [Java, Core Java]
tags: [java, interface, sort, lambda expression, method reference, inner class, service loader, proxy]
---
本章将介绍几种常用的高级技术：接口、lambda表达式、内部类和代理。

## 6.1 接口
### 6.1.1 接口的概念
**接口**(interface)用来描述类应该做**什么**（提供哪些方法），而不指定具体应该**如何**做（如何实现这些方法）。

通常，某个服务的提供者会说：“如果你的类符合某个特定接口，我就会履行这项服务。”下面给出一个具体的示例。`Arrays`类的`sort()`方法承诺对对象数组进行排序，但有一个条件：对象所属的类必须实现`Comparable`接口。

`Comparable`接口如下所示：

```java
public interface Comparable<T> {
    int compareTo(T other);
}
```

当调用`x.compareTo(y)`时，这个方法必须能够比较两个对象，并返回比较结果：当`x`小于`y`时，返回一个负数；当`x`等于`y`时，返回0；否则返回一个正数。

在这个接口中，`compareTo()`方法是**抽象的**，它没有实现。任何实现`Comparable`接口的类都需要包含`compareTo()`方法。否则，这个类也是抽象的。

接口中的所有方法都自动是公有的。因此在接口中声明方法时，不必提供关键字`public`。

接口可以包含多个方法。稍后将看到，接口还可以定义常量。然而，接口不能有实例字段。在Java 8之前，接口中的所有方法都是抽象的。

现在，假设希望用`Arrays.sort()`方法对`Employee`对象数组进行排序，那么`Employee`类就必须实现`Comparable`接口。

一个类可以**实现**(implement)一个或多个接口。要让类实现一个接口，需要：
1. 使用关键字`implements`声明类实现给定的接口。
2. 对接口中的所有方法提供定义。

假设希望根据员工的薪水进行比较，以下是`compareTo()`方法的实现：

```java
class Employee implements Comparable<Employee> {
    @Override
    public int compareTo(Object other) {
        return Double.compare(salary, other.salary);
    }
    ...
}
```

警告：接口中的所有方法都自动是公有的。不过，在实现接口时，必须把类中的方法声明为`public`，否则编译器将认为这个方法是包访问的，并报错指出你试图提供更严格的访问权限（见5.1.2节“警告”）。

提示：`Comparable`接口的`compareTo()`方法返回的整数只有符号重要，具体值并不重要。在比较整数字段时这种灵活性非常有用。例如，假设每个员工都有一个唯一的整数`id`，并希望根据员工ID进行排序，那么可以直接返回`id - other.id`。但有一点需要注意：整数的范围要足够小，以免减法运算溢出（例如，x = 2147483647, y = -1，则x - y将发生溢出，结果为-2147483648）。如果无法保证不会溢出，就应该调用静态方法`Integer.compare()`（注：`Integer`类也实现了`Comparable`接口，`x.compareTo(y)`等价于`Integer.compare(x, y)`）。这个相减技巧不适用于浮点数。如果两个浮点数很接近但不相等（例如100和100+1e-15），它们的差可能被舍入为0。应该调用`Double.compare()`。

注释：`Comparable`接口的文档建议`compareTo()`方法应当与`equals()`方法兼容，即`x.compareTo(y) == 0`当且仅当`x.equals(y)`。Java API中大多数实现`Comparable`接口的类都遵从了这个建议。一个值得注意的例外是`BigDecimal`。考虑`x = new BigDecimal("1.0")`和`y = new BigDecimal("1.00")`，`x.equals(y)`为`false`，因为两个数的精度不同，但`x.compareTo(y)`为0。

为什么不能在`Employee`类中直接提供`compareTo()`方法而不实现`Comparable`接口呢？主要原因在于Java是一种**强类型**(strongly typed)语言。调用方法时，编译器要能检查这个方法确实存在。在`sort()`方法中可能会有类似下面的语句：

```java
if (a[i].compareTo(a[j]) > 0) {
    // rearrange a[i] and a[j]
    ...
}
```

编译器必须确认`a[i]`确实有`compareTo()`方法。如果`a`是一个`Comparable`对象的数组，就可以确保存在`compareTo()`方法，因为每个实现`Comparable`接口的类都必须提供这个方法。

注：与Java相反，Python没有接口，而是依赖于“鸭子类型”，详见[《Python基础教程》笔记 第7章]({% post_url 2024-01-24-python-note-ch07-more-abstraction %})和[《Python基础教程》笔记 第9章]({% post_url 2024-02-21-python-note-ch09-magic-methods-properties-and-iterators %}) 9.3节。

注释：你可能认为`Arrays.sort()`方法定义为接受一个`Comparable[]`，如果调用`sort()`方法时所提供数组的元素类型没有实现`Comparable`接口，编译器就能报错。遗憾的是，事实并非如此。实际上，`sort()`方法接受一个`Object[]`，并使用笨拙的强制类型转换：

```java
// approach used in the standard library--not recommended
if (((Comparable) a[i]).compareTo(a[j]) > 0) {
    // rearrange a[i] and a[j]
    ...
}
```

如果`a[i]`不属于实现了`Comparable`接口的类，虚拟机就会抛出一个异常(`ClassCastException`)。

程序清单6-1给出了对`Employee`类（程序清单6-2）数组进行排序的完整代码。

[程序清单6-1 interfaces/EmployeeSortTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch06/interfaces/EmployeeSortTest.java)

[程序清单6-2 interfaces/Employee.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch06/interfaces/Employee.java)

注释：语言标准规定：对于任意的`x`和`y`，实现者必须确保`sgn(x.compareTo(y)) = -sgn(y.compareTo(x))`，其中`sgn`是符号函数。这意味着如果`x.compareTo(y)`抛出异常，`y.compareTo(x)`也必须抛出异常。与`equals()`方法一样，使用继承时可能会出现问题。由于`Manager`继承了`Employee`，因此它实现的是`Comparable<Employee>`而不是`Comparable<Manager>`。如果`Manager`要覆盖`compareTo()`，就必须能比较经理和员工，而不能简单地将员工强制转换成经理：

```java
class Manager extends Employee {
    public int compareTo(Employee other) {
        Manager otherManager = (Manager) other; // NO
        ...
    }
    ...
}
```

这违反了“反对称”规则。如果`x`是一个`Employee`对象、`y`是一个`Manager`对象，则调用`x.compareTo(y)`不会抛出异常，它只是将`x`和`y`都作为员工进行比较。但是反过来，`y.compareTo(x)`会抛出`ClassCastException`。

这种情况与第5章中讨论的`equals()`方法一样，解决方法也一样。有两种不同的情况。如果子类的比较有不同的含义，就应该将属于不同类的对象之间的比较视为非法，`compareTo()`方法应该首先进行检测：`if (getClass() != other.getClass()) throw new ClassCastException();`。如果存在一个比较子类对象的通用算法，那么就在超类中提供一个`compareTo()`方法，并将其声明为`final`。

### 6.1.2 接口的特性
接口不是类。特别是，不能使用`new`运算符实例化一个接口：

```java
x = new Comparable(...); // ERROR
```

注：除非是创建匿名内部类的对象，见6.3.6节。

不过，仍然可以声明接口变量。接口变量必须引用实现了这个接口的类对象。

```java
Comparable x = new Employee(...); // OK provided Employee implements Comparable
```

也可以使用`instanceof`检查对象是否实现了某个接口：

```java
if (anObject instanceof Comparable) { ... }
```

与类一样，也可以扩展接口。例如：

```java
public interface Moveable  {
    void move(double x, double y);
}

public interface Powered extends Moveable {
    double milesPerGallon();
}
```

虽然接口中不能包含实例字段，但是可以包含常量。例如：

```java
public interface Powered extends Moveable {
    double milesPerGallon();
    double SPEED_LIMIT = 95; // a public static final constant
}
```

就像接口中的方法自动是`public`一样，接口中的常量总是`public static final`。

尽管每个类只能有一个超类，但可以实现多个接口。这就为定义类的行为提供了极大的灵活性。使用逗号将各个接口分隔开：

```java
class Employee implements Cloneable, Comparable
```

注释：记录和枚举不能扩展其他类（因为它们隐式扩展了`Record`和`Enum`类）。不过，它们可以实现接口。

注释：接口可以是密封的。与密封类一样，直接子类型（可以是类或接口）必须在`permits`子句中声明，或者放在同一个源文件中。

### 6.1.3 接口和抽象类
为什么Java的设计者要引入接口概念，而不直接使用抽象类呢？因为每个类只能扩展一个类。假设`Employee`已经扩展了`Person`类，就不能再扩展`Comparator`类了：

```java
class Employee extends Person, Comparable // ERROR
```

但是每个类可以实现任意多个接口：

```java
class Employee extends Person implements Comparable // OK
```

其他编程语言（尤其是C++）允许一个类有多个超类，这个特性称为**多重继承**(multiple inheritance)。Java的设计者选择不支持多重继承，因为它会让语言变得非常复杂（如C++）或者降低效率（如Eiffel）。接口可以提供多重继承的大多数好处，同时避免复杂性和低效率。

C++注释：C++支持多重继承，但随之带来了虚基类、优先性规则(rules of dominance)和横向转换(side cast)等复杂特性。很少有C++程序员使用多重继承，甚至有人说永远不应该使用多重继承。也有人建议只对“混入”(mix-in)风格的继承使用多重继承。在混入风格中，一个主要基类描述父对象，其他的基类（混入类）提供辅助特性（例如Python的`socketserver.ForkingMixIn`）。这种风格类似于一个Java类扩展一个超类并实现多个接口。

提示：`String`和`StringBuilder`都实现了`CharSequence`接口。这个接口包含所有管理字符序列的类的公共方法。有一个公共的接口会鼓励程序员编写使用`CharSequence`接口的方法。如果要处理字符串，而这个接口的操作已经能满足你的任务要求，就可以接受`CharSequence`实例而不是字符串。

### 6.1.4 静态和私有方法
从Java 8起，允许在接口中添加静态方法。只是这似乎有违将接口作为抽象规范的初衷。

到目前为止，通常的做法都是将静态方法放在伴随类中。在标准库中，会看到成对出现的接口和工具类，例如`Collection/Collections`或`Path/Paths`。

可以使用静态方法`Paths.get()`构造一个文件或目录的路径，如`Paths.get("jdk-17", "conf", "security")`。在Java 11中，`Path`接口提供了等价的方法：

```java
public interface Path {
    public static Path of(URI uri) { ... }
    public static Path of(String first, String... more) { ... }
    ...
}
```

这样，`Paths`类就不再是必要的了。在实现自己的接口时，没有理由再为工具方法提供单独的伴随类。

从Java 9起，接口中的方法可以是私有的。私有方法可以是静态方法或实例方法。由于私有方法只能在接口本身的方法中使用，所以它们的用途仅限于作为接口中其他方法的辅助方法。

### 6.1.5 默认方法
从Java 8起，可以为任何接口方法提供一个**默认**实现，使用`default`修饰符标记。例如，在第9章会看到`Iterator`接口用于访问一个数据结构中的元素：

```java
public interface Iterator<E> {
    boolean hasNext();
    E next();
    default void remove() { throw new UnsupportedOperationException("remove"); }
    ...
}
```

如果要实现一个迭代器，就需要提供`hasNext()`和`next()`方法。这两个方法没有默认实现，因为它们取决于要遍历的数据结构。但如果你的迭代器是只读的，就不必操心`remove()`方法（其默认实现是抛出一个异常）。

默认方法可以调用其他方法。例如，`Collection`接口可以用`size()`来定义`isEmpty()`方法：

```java
public interface Collection {
    int size(); // an abstract method
    default boolean isEmpty() { return size() == 0; }
    ...
}
```

这样，实现`Collection`接口的类就无需实现`isEmpty()`方法了。

注释：Java API中的`Collection`接口实际上并没有这样做，而是在一个实现了`Collection`接口的`AbstractCollection`类中这样定义了`isEmpty()`（因为这个接口在JDK 1.2中就存在了，那时还没有默认方法），并建议集合的实现者扩展`AbstractCollection`。这种技术已经过时，现在可以直接在接口中实现方法。

默认方法的一个重要用途是**接口演化**(interface evolution)。以`Collection`接口为例，这个接口作为Java的一部分已经有很多年了。假设很久以前你定义了一个实现这个接口的类`Bag`。后来，在Java 8中，这个接口又添加了一个方法`stream()`。

假设`stream()`方法不是默认方法，那么`Bag`类将不能编译，因为它没有实现这个新方法。因此，为接口添加非默认方法不是**源代码兼容**(source compatible)的。

假设不重新编译这个类，而是使用包含这个类的旧的JAR文件。这个类仍能正常加载，程序仍然可以构造`Bag`实例。因此，为接口添加方法是**二进制兼容**(binary compatible)的。不过，如果程序对`Bag`实例调用`stream()`方法，就会出现`AbstractMethodError`。

将`stream()`方法实现为默认方法就可以解决这两个问题。`Bag`类又能正常编译了。另外如果没有重新编译而直接加载这个类，并对`Bag`实例调用`stream()`方法，则会调用`Collection.stream()`方法。

### 6.1.6 解决默认方法冲突
如果在一个接口中将一个方法定义为默认方法，然后又在超类或另一个接口中定义了同样的方法，就会发生冲突。诸如Scala和C++等语言对于解决这种歧义性有一些复杂的规则。幸运的是，在Java中规则要简单得多：
1. 超类优先(Superclasses win)。如果超类提供了一个具体方法，具有相同签名的默认方法会被忽略。
2. 接口冲突(Interfaces clash)。如果一个接口提供了一个默认方法，另一个接口包含一个具有相同签名的方法（无论是否是默认方法），必须覆盖这个方法来解决冲突。

下面来看第二个规则。考虑两个包含`getName()`方法的接口：

```java
interface Person {
    default String getName() { return ""; }
}
interface Named {
    default String getName() { return getClass().getName() + "_" + hashCode(); }
}
```

如果一个类同时实现了这两个接口，编译器将报错：

```java
class Student implements Person, Named { ... }
```

只需在`Student`类中提供一个`getName()`方法即可。在这个方法中，可以选择两个冲突方法中的一个，如下所示：

```java
class Student implements Person, Named {
    public String getName() { return Person.super.getName(); }
    ...
}
```

如果至少有一个接口提供了实现，编译器就会报错，程序员必须解决这个歧义性。

注释：如果两个接口都没有为共享方法提供默认实现，那么就不存在冲突。实现类有两个选择：实现这个方法，或者不实现。在后一种情况中，这个类本身是抽象的。

现在考虑另一种情况：一个类扩展了一个超类并实现了一个接口，从二者继承了相同的方法。例如，假设`Person`是一个类，`Student`定义为：

```java
class Student extends Person implements Named { ... }
```

在这种情况下（如果有签名相同的方法），只会考虑超类方法，接口中的默认方法会被忽略。在这个例子中，`Student`从`Person`继承了`getName()`方法，`Named`接口是否为`getName()`提供默认实现没有任何区别。这就是“超类优先”规则。这一规则确保与Java 7的兼容性。如果为一个接口添加默认方法，这对于（Java 8引入默认方法）之前能正常工作的代码不会有任何影响。

警告：由于“超类优先”规则，永远无法创建一个重新定义`Object`类中方法的默认方法。

### 6.1.7 接口与回调
**回调**(callback)是一种常见的程序设计模式。在这个模式中，可以指定当某个特定事件发生时应该执行的动作。例如，在GUI程序中，点击按钮或选择菜单项时执行特定的操作。

`javax.swing`包包含一个`Timer`类，可用于在经过一定的时间间隔时得到通知。构造定时器时，需要设置时间间隔和要执行的动作。计时器要求你指定一个实现了`ActionListener`接口的类的对象：

```java
public interface ActionListener {
    void actionPerformed(ActionEvent event);
}
```

当到达指定的时间间隔时，定时器就调用`actionPerformed()`方法。

假设你希望每秒打印一条消息 "At the tone, the time is ..." ，然后响一声(beep)，那么可以定义如下的类：

```java
class TimePrinter implements ActionListener {
    @Override
    public void actionPerformed(ActionEvent event) {
        System.out.println("At the tone, the time is " + Instant.ofEpochMilli(event.getWhen()));
        Toolkit.getDefaultToolkit().beep();
    }
}
```

注意这个方法的`ActionEvent`参数。这个参数提供了事件的相关信息，例如事件发生的时间(`event.getWhen()`)。

接下来，构造一个这个类的对象，并将它传递给`Timer`构造器，最后启动定时器：

```java
var listener = new TimePrinter();
Timer t = new Timer(1000, listener);
t.start();
```

`Timer`构造器的第一个参数是时间间隔，单位是毫秒。第二个参数是监听器对象。

程序清单6-3展示了定时器和动作监听器的具体使用。

[程序清单6-3 timer/TimerTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch06/timer/TimerTest.java)

### 6.1.8 Comparator接口
在6.1.1节中，已经了解如何排序一个对象数组，前提是这些对象是实现了`Comparable`接口的类的实例。例如，可以对一个字符串数组排序，因为`String`类实现了`Comparable<String>`，其`compareTo()`方法按字典序比较字符串。

假设我们希望按长度递增的顺序而不是字典序对字符串进行排序。我们无法让`String`类用两种不同的方式实现`compareTo()`方法。为了处理这种情况，`Arrays.sort()`方法还有第二个版本，其参数是一个数组和一个**比较器**(comparator)——实现了`Comparator`接口的类的实例。

```java
public interface Comparator<T> {
    int compare(T first, T second);
}
```

要按长度比较字符串，可以如下定义一个`LengthComparator`类：

```java
class LengthComparator implements Comparator<String> {
    @Override
    public int compare(String first, String second) {
        return first.length() - second.length();
    }
}
```

实际进行比较时，需要创建一个实例：

```java
var comp = new LengthComparator();
if (comp.compare(words[i], words[j]) > 0) ...
```

这个调用与`words[i].compareTo(words[j])`相比，`compare()`方法是在比较器对象上调用，而不是字符串本身。

注释：尽管`LengthComparator`对象没有状态，仍然需要创建一个实例，因为`compare()`不是静态方法。

要对一个数组排序，需要将比较器传递给`Arrays.sort()`方法：

```java
String[] friends = { "Peter", "Paul", "Mary" };
Arrays.sort(friends, new LengthComparator());
```

在6.2节中将会看到，利用lambda表达式可以更容易地使用`Comparator`。

### 6.1.9 对象克隆
本节将讨论`Cloneable`接口，这个接口表示一个类提供了安全的`clone()`方法。

要理解克隆的含义，先来回忆拷贝。拷贝一个对象变量时，原变量和副本将引用同一个对象（见下图）。这意味着修改任何一个变量都会影响另一个变量。

```java
var original = new Employee("John Public", 50000);
Employee copy = original;
copy.raiseSalary(10); // oops--also changed original
```

如果希望`copy`是一个新对象，就要使用`clone()`方法。

```java
Employee copy = original.clone();
copy.raiseSalary(10); // OK--original unchanged
```

![拷贝和克隆](/assets/images/java-note-v1ch06-interfaces-lambda-expressions-and-inner-classes/拷贝和克隆.png)

不过并没有这么简单。`clone()`是`Object`的一个`protected`方法，这意味着你的代码不能直接调用这个方法。这个限制是有原因的。考虑`Object`类如何实现`clone()`：它对于这个对象一无所知，所以只能逐个字段地进行拷贝。如果对象中的所有实例字段都是基本类型，拷贝这些字段没有任何问题。但如果对象包含子对象的引用，拷贝字段就会引用同一个子对象，这样原对象和克隆的对象仍然会共享一些信息。

下图显示了使用`Object.clone()`方法克隆`Employee`对象的结果。可以看到，默认的克隆操作是“浅拷贝”，并没有克隆对象中引用的其他对象。

![浅拷贝](/assets/images/java-note-v1ch06-interfaces-lambda-expressions-and-inner-classes/浅拷贝.png)

注：Java中默认的克隆操作相当于C++中编译器自动生成的拷贝构造函数，在这一点上Java和Python一样（见[《Python基础教程》笔记 第4章]({% post_url 2023-12-20-python-note-ch04-dictionaries-when-indices-wont-do %}) 4.2.4节）。

浅拷贝会有什么影响吗？这要看具体情况。如果原对象和克隆对象共享的子对象是不可变的（如`String`），那么这种共享就是安全的。然而，如果子对象是可变的，就必须重新定义`clone()`方法来创建一个**深拷贝**(deep copy)（也克隆子对象）。在这个例子中，`hireDay`字段是`Date`类型，是可变的，所以它也必须被克隆（如果`hireDay`是不可变的`LocalDate`类型，就无需做任何处理了）。

对于每一个类，需要决定：
1. 默认的`clone()`方法是否能满足要求；
2. 是否需要克隆可变的子对象；
3. 是否不该使用`clone()`。

实际上第三个是默认选项。如果选择第一个或第二个选项，类必须：
1. 实现`Cloneable`接口；
2. 重新定义`clone()`方法，并指定`public`访问修饰符。

注释：虽然所有类都是`Object`的子类，但是子类只能调用受保护的`clone()`方法来克隆**它自己的**对象。必须将`clone()`重新定义为公有才能被其他类的方法调用。

`Cloneable`接口并没有指定`clone()`方法，这个方法是从`Object`类继承的。这个接口仅仅作为一个标记，表示类设计者了解克隆过程。如果在一个对象上调用`clone()`，但这个对象的类并没有实现`Cloneable`接口，`Object.clone()`方法就会抛出一个检查型异常`CloneNotSupportedException`。

注释：`Cloneable`接口是Java提供的少数**标记接口**(tagging interface)之一。接口的通常用途是确保类实现了特定的方法。标记接口不包含任何方法，它的唯一作用就是允许使用`instanceof`：`if (obj instanceof Cloneable) ...`。建议不要在自己的程序中使用标记接口。

即使`clone()`的默认实现（浅拷贝）能够满足要求，仍然需要实现`Cloneable`接口，将`clone()`重新定义为`public`，并调用`super.clone()`。下面是一个例子：

```java
class Employee implements Cloneable {
    ...
    // public access, change return type
    @Override
    public Employee clone() throws CloneNotSupportedException {
        return (Employee) super.clone();
    }
}
```

注释：在Java 1.4之前，`clone()`方法的返回类型总是`Object`，而现在可以为你的`clone()`方法指定正确的返回类型。这是协变返回类型的一个例子（见5.1.6节）。

下面是创建深拷贝的`clone()`方法的一个例子：

```java
class Employee implements Cloneable {
    ...
    @Override
    public Employee clone() throws CloneNotSupportedException {
        // call Object.clone()
        Employee cloned = (Employee) super.clone();
        // clone mutable fields
        cloned.hireDay = (Date) hireDay.clone();
        return cloned;
    }
}
```

必须当心子类的克隆。例如，一旦为`Employee`类定义了`clone()`方法，任何人都可以用它来克隆`Manager`对象。是否能正确克隆，这取决于`Manager`类的字段。在这里没有问题，因为`bonus`字段是基本类型。但是`Manager`可能有需要深拷贝或者不可克隆的字段。不能保证子类的实现者一定会修正`clone()`方法让它正确工作。

克隆没有你想象中那么常用。标准库中只有不到5%的类实现了`clone()`。

注释：所有数组类型都有一个公有的（而不是受保护的）`clone()`方法。可以用这个方法创建一个新数组，包含原数组所有元素的副本。例如：

```java
int[] luckyNumbers = { 2, 3, 5, 7, 11, 13 };
int[] cloned = luckyNumbers.clone();
cloned[5] = 12; // doesn't change luckyNumbers[5]
```

程序清单6-4中的程序克隆了`Employee`类（程序清单6-5）的一个实例，然后调用两个修改器方法。这两个修改器方法都不会影响原来的对象，因为`clone()`定义为创建一个深拷贝。

[程序清单6-4 clone/CloneTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch06/clone/CloneTest.java)

[程序清单6-5 clone/Employee.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch06/clone/Employee.java)

## 6.2 Lambda表达式
**Lambda表达式**是一个可传递的代码块，可以在以后执行一次或多次。

### 6.2.1 为什么引入lambda表达式
在6.1.7节定时器的例子中，通过`ActionListener.actionPerformed()`方法指定要执行的动作。在6.1.8节按长度排序字符串数组的例子中，通过`Comparator.compare()`方法指定比较标准。这两个例子有一些共同点：都是将一个代码块传递到某处（`Timer`构造器或`sort()`方法），这个代码块会在将来某个时间调用（到达定时器指定的时间间隔或`sort()`方法比较两个元素）。

到目前为止，在Java中不能直接传递代码块。Java是一种面向对象的语言，所以必须构造一个对象，这个对象的类要有一个方法包含所需的代码。

### 6.2.2 lambda表达式的语法
Lambda表达式的简单形式为：`(params) -> expr`。例如，`LengthComparator`可以用lambda表达式简写为

```java
(String first, String second) -> first.length() - second.length()
```

这个lambda表达式接受两个`String`参数，返回一个`int`，刚好满足`Comparator<String>`的`compare()`方法的要求。

如果代码要完成的计算无法放在一个表达式中，可以用`{}`括起来：`(params) -> { statement; }`。例如：

```java
(String first, String second) -> {
    if (first.length() < second.length()) return -1;
    else if (first.length() > second.length()) return 1;
    else return 0;
}
```

即使lambda表达式没有参数，仍然要提供空括号，就像无参数方法一样：

```java
() -> {
    for (int i = 100; i >= 0; i--) System.out.println(i);
}
```

如果可以推导出lambda表达式的参数类型，就可以将其省略。例如：

```java
Comparator<String> comp = (first, second) -> first.length() - second.length();
```

在这里，编译器可以推导出`first`和`second`必然是字符串，因为这个lambda表达式被赋给一个`Comparator<String>`（下一节会更详细地分析这个赋值）。

如果lambda表达式只有一个参数，而且类型可以推导得出，甚至还可以省略括号：

```java
ActionListener listener = event ->
    System.out.println("The time is " + Instant.ofEpochMilli(event.getWhen()));
```

不能指定lambda表达式的返回类型，返回类型总是会由上下文推导得出。如果`{}`中不包含`return`语句，则返回类型为`void`。

可以用`var`表示推导的类型。这不常见。发明这个语法是为了添加注解（见卷II第8章）：

```java
(@NonNull var first, @NonNull var second) -> first.length() - second.length()
```

注释：lambda表达式只在某些分支返回值、其他分支不返回值是不合法的。例如，`(int x) -> { if (x >= 0) return 1; }`就不合法。

程序清单6-6中的程序展示了如何将lambda表达式用于比较器和动作监听器。

[程序清单6-6 lambda/LambdaTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch06/lambda/LambdaTest.java)

### 6.2.3 函数式接口
只有一个抽象方法的接口称为**函数式接口**(functional interface)。**需要这种接口的对象时，就可以提供一个lambda表达式**（前提是lambda表达式的参数和返回类型与接口的抽象方法匹配）。

考虑`Arrays.sort()`方法，它的第二个参数需要一个`Comparator`实例。`Comparator`是函数式接口，所以可以提供一个lambda表达式：

```java
Arrays.sort(words, (first, second) -> first.length() - second.length());
```

在底层，`Arrays.sort()`方法接收到实现了`Comparator<String>`的某个类的对象，在这个对象上调用`compare()`方法会执行这个lambda表达式的体。

实际上，在Java中对lambda表达式所能做的也只是转换为函数式接口。在其他支持函数字面值的编程语言（如Scala）中，可以声明函数类型的变量并保存函数表达式，而Java没有函数类型。

注释：甚至不能把lambda表达式赋给`Object`类型的变量，因为`Object`不是函数式接口。

Java API在`java.util.function`包中定义了很多非常通用的函数式接口（详见6.2.7节）。一个尤其有用的接口是`Predicate`：

```java
public interface Predicate<T> {
    boolean test(T t);
    // additional default and static methods
}
```

`ArrayList`类有一个`removeIf()`方法，用于删除所有满足条件的元素，其参数就是一个`Predicate`。例如，下面的语句从数组列表中删除所有`null`值：

```java
list.removeIf(e -> e == null);
```

另一个有用的函数式接口是`Supplier`：

```java
public interface Supplier<T> {
    T get();
}
```

`Supplier`没有参数，调用时生成一个`T`类型的值。`Supplier`用于实现**惰性求值**(lazy evaluation)。例如，考虑以下调用：

```java
LocalDate hireDay = Objects.requireNonNullElse(day, new LocalDate.of(1970, 1, 1));
```

这不是最优的。我们预计`day`很少为`null`，所以希望只在必要时才构造默认的`LocalDate`。通过使用`Supplier`可以延迟这个计算：

```java
LocalDate hireDay = Objects.requireNonNullElseGet(day, () -> new LocalDate.of(1970, 1, 1));
```

`requireNonNullElseGet()`方法只在需要值（第一个参数为`null`）时才调用`Supplier`。

### 6.2.4 方法引用
有时，lambda表达式直接调用一个已有的方法。例如，假设你希望在出现定时器事件时打印事件对象。当然，可以调用

```java
var timer = new Timer(1000, event -> System.out.println(event));
```

如果可以直接把`println()`方法传递给`Timer`构造器就更好了。具体做法如下：

```java
var timer = new Timer(1000, System.out::println);
```

表达式`System.out::println`是一个**方法引用**(method reference)。它等价于lambda表达式`e -> System.out.println(e)`。

注释：`System.out`中有10个重载的`println()`方法，编译器需要根据上下文确定使用哪一个方法。在这个例子中，方法引用`System.out::println`必须转换为一个包含方法`void actionPerformed(ActionEvent e)`的`ActionListener`实例。这样会从10个重载的`println()`方法中选出`println(Object)`方法，因为`Object`与`ActionEvent`最匹配。`actionPerformed()`方法被调用时，就会打印这个事件对象。

现在假设把同样的方法引用赋给一个不同的函数式接口：`Runnable task = System.out::println;`。函数式接口`Runnable`有一个无参数的抽象方法`void run()`。在这种情况下，就会选择无参数的`println()`方法。调用`task.run()`会打印一个空行。

再来看一个例子，假设你想对字符串进行排序，而不区分大小写。可以传递以下方法引用：

```java
Arrays.sort(strings, String::compareToIgnoreCase);
```

这等价于lambda表达式`(x, y) -> x.compareToIgnoreCase(y)`，在这里充当`Comparator<String>`。

从这些例子可以看出，`::`运算符分隔方法名与对象或类名。有3种变体：
* `object::instanceMethod`，等价于`(arg1, arg2, ...) -> object.instanceMethod(arg1, arg2, ...)`
* `Class::staticMethod`，等价于`(arg1, arg2, ...) -> Class.staticMethod(arg1, arg2, ...)`
* `Class::instanceMethod`，等价于`(arg1, arg2, ...) -> arg1.instanceMethod(arg2, ...)`

对于第1种变体，方法引用等价于lambda表达式，其参数传递给方法。例如，`System.out::println`等价于`x -> System.out.println(x)`。

对于第2种变体，所有参数都传递给静态方法。例如，`Math::pow`等价于`(x, y) -> Math.pow(x, y)`。

对于第3种变体，第1个参数会成为方法的隐式参数。例如，`String::compareToIgnoreCase`等价于`(x, y) -> x.compareToIgnoreCase(y)`。

注：另见[Java 8的方法引用]({% post_url 2019-08-24-java-8-method-reference %})。

注意，只有当lambda表达式的体只调用一个方法而不做其他操作时才能重写为方法引用。例如，lambda表达式`s -> s.length() == 0`中还有一个比较操作，所以不能使用方法引用。

注释：类似于lambda表达式，方法引用不会独立存在，总是会转换为函数式接口的实例。

注释：有时，API包含一些专门用作方法引用的方法。例如，`Objects`类有一个方法`isNull()`，用于测试一个对象引用是否为`null`。这看上去好像没有什么用，因为`obj == null`比`Objects.isNull(obj)`更易读。不过，可以把这个方法引用传递给任何有`Predicate`参数的方法。例如，要从一个列表删除所有`null`，可以调用`list.removeIf(Objects::isNull);`

注释：包含对象的方法引用与等价的lambda表达式还有一个细微的差别。例如，如果`obj`为`null`，构造`obj::equals`会立即抛出`NullPointerException`，而lambda表达式`x -> obj.equals(x)`只有在被调用时才会抛出异常。

可以在方法引用中使用`this`参数。例如，`this::equals`等同于`x -> this.equals(x)`。使用`super`也是合法的。方法引用`super::instanceMethod`会使用`this`调用给定方法的超类版本。

例如：

```java
class Greeter {
    public void greet(ActionEvent event) {
        System.out.println("Hello, the time is " + Instant.ofEpochMilli(event.getWhen()));
    }
}

class RepeatedGreeter extends Greeter {
    public void greet(ActionEvent event) {
        var timer = new Timer(1000, super::greet);
        timer.start();
    }
}
```

`RepeatedGreeter.greet()`方法会构造一个定时器，定期执行`super::greet`（即`Greeter.greet()`方法）。

### 6.2.5 构造器引用
构造器引用与方法引用很类似，只不过方法名为`new`。例如，`Person::new`是`Person`类的构造器引用。具体哪一个构造器取决于上下文。

假设有一个字符串列表，可以在每个字符串上调用构造器，将其转换为`Person`对象数组：

```java
ArrayList<String> names = ...;
Stream<Person> stream = names.stream().map(Person::new);
List<Person> people = stream.toList();
```

我们将在卷II的第1章讨论流的详细内容。就现在来说，重点是`map()`方法会为每个列表元素调用`Person(String)`构造器，因为编译器从上下文推导出这是在调用带字符串参数的构造器。

注：在这里构造器引用`Person::new`等价于`(String s) -> new Person(s)`，充当`Function<String, Person>`接口。

可以用数组类型建立构造器引用。例如，`int[]::new`是一个数组构造器引用，它有一个参数：数组的长度。这等价于lambda表达式`n -> new int[n]`。

注：数组构造器引用`T[]::new`可以充当`IntFunction<T[]>`或`Function<Integer, T[]>`接口。

在第8章中将会看到，Java有一个限制：无法构造泛型类型`T`的数组（表达式`new T[n]`会产生错误，因为这会被“擦除”为`new Object[n]`）。数组构造器引用对于克服这个限制很有用。例如，假设我们需要一个`Person`对象数组。`Stream`接口有一个`toArray()`方法可以返回`Object`数组：

```java
Object[] people = stream.toArray();
```

不过这并不令人满意。用户希望得到一个`Person[]`而不是`Object[]`。流库利用构造器引用解决了这个问题，可以把`Person[]::new`传递给`toArray()`方法：

```java
Person[] people = stream.toArray(Person[]::new);
```

详见卷II第1章1.8节。

### 6.2.6 变量作用域
通常，你可能希望能够在lambda表达式中访问外围方法或类中的变量。考虑下面这个例子：

```java
public static void repeatMessage(String text, int delay) {
    ActionListener listener = event -> {
        System.out.println(text);
        Toolkit.getDefaultToolkit().beep();
    };
    new Timer(delay, listener).start();
}
```

考虑以下调用：

```java
repeatMessage("Hello", 1000); // prints Hello every 1,000 milliseconds
```

注意变量`text`并不是在lambda表达式中定义的，而是`repeatMessage()`方法的参数变量。lambda表达式的代码可能在调用`repeatMessage()`返回很久以后才运行，而那时这个参数变量已经不存在了。`text`变量是如何保留下来的呢？

lambda表达式有3个部分：
1. 参数
2. 代码块
3. **自由变量**（即不是参数也不是在代码块中定义的变量）的值

在上面的例子中，这个lambda表达式有一个自由变量`text`。表示lambda表达式的数据结构必须存储自由变量的值，在这里是字符串`"Hello"`。我们说这些值被lambda表达式**捕获**(captured)。如何做到这一点是实现细节。例如，可以把lambda表达式转换为包含单个方法的对象，并将自由变量的值复制到这个对象的实例字段中（注：C++编译器就是这样翻译lambda表达式的，详见[C++函数式编程]({% post_url 2023-10-04-cpp-functional-programming %}) 5.2节）。

注释：代码块连同自由变量值一起有一个术语：**闭包**(closure)。在Java中，lambda表达式就是闭包。

在lambda表达式中，不能修改捕获的变量。例如，下面是不合法的：

```java
public static void countDown(int start, int delay) {
    ActionListener listener = event -> {
        start--; // ERROR: Can't mutate captured variable
        System.out.println(start);
    };
    new Timer(delay, listener).start();
}
```

这个限制是有原因的。当并发执行多个动作时，在lambda表达式中修改变量是不安全的。关于这个问题参见第12章。

在lambda表达式中引用一个在外部修改的变量也是不合法的。例如，下面是不合法的：

```java
public static void repeat(String text, int count) {
    for (int i = 1; i <= count; i++) {
        ActionListener listener = event -> {
            System.out.println(i + ": " + text); // ERROR: Cannot refer to changing i
        };
        new Timer(1000, listener).start();
    }
}
```

这里的规则是：lambda表达式中捕获的变量必须是**事实最终变量**(effectively final)。事实最终变量是指这个变量在初始化之后就不会再为它赋新值（即使没有声明为`final`）。在这个例子中，`text`始终引用同一个字符串对象，因此可以被捕获。然而，`i`的值会改变，因此不能被捕获。

lambda表达式的体与嵌套块有**相同的作用域**。命名冲突和遮蔽的规则同样适用。在lambda表达式中声明与（外围方法中）局部变量同名的参数或局部变量是不合法的。

```java
Path first = Path.of("/usr/bin");
Comparator<String> comp = (first, second) -> first.length() - second.length();
  // ERROR: Variable first already defined
```

在lambda表达式中使用`this`关键字时，是指创建这个lambda表达式的方法的`this`参数。例如：

```java
public class Application {
    public void init() {
        ActionListener listener = event -> {
            System.out.println(this.toString());
            ...
        }
    ...
    }
}
```

表达式`this.toString()`调用`Application`对象的`toString()`方法，而不是`ActionListener`实例的方法。

### 6.2.7 处理lambda表达式
你已经了解如何把lambda表达式传递给需要函数式接口的方法。下面来看如何编写接受lambda表达式的方法。

使用lambda表达式的重点是**延迟执行**(deferred execution)。例如：
* 在一个单独的线程中运行代码
* 多次运行代码
* 在算法的适当位置运行代码（例如排序中的比较操作）
* 发生某种情况时运行代码（例如点击了按钮）
* 只在必要时运行代码

下面来看一个简单的例子。假设你想要重复一个动作n次，将动作和次数传递给一个`repeat()`方法：

```java
repeat(10, () -> System.out.println("Hello, World!"));
```

要接受这个lambda表达式，需要选择（偶尔需要提供）一个函数式接口。下表列出了Java API中提供的最重要的函数式接口。在这里，可以使用`Runnable`接口（因为这个lambda表达式是无参数无返回值的）：

```java
public static void repeat(int n, Runnable action) {
    for (int i = 0; i < n; i++) action.run();
}
```

| 函数式接口 | 抽象方法 | 描述 | 其他方法 |
| --- | --- | --- | --- |
| `Runnable` | `void run()` | 运行一个无参数无返回值的动作 | |
| `Supplier<T>` | `T get()` | 提供一个`T`类型的值 | |
| `Consumer<T>` | `void accept(T)` | 消费一个`T`类型的值 | `andThen()` |
| `BiConsumer<T, U>` | `void accept(T, U)` | 消费`T`和`U`类型的值 | `andThen()` |
| `Function<T, R>` | `R apply(T)` | 接受`T`参数、返回`R`的函数 | `andThen()`, `compose()`, `identity()` |
| `BiFunction<T, U, R>` | `R apply(T, U)` | 接受`T`和`U`参数、返回`R`的函数 | `andThen()` |
| `UnaryOperator<T>` | `T apply(T)` | `T`类型的一元运算符 | `andThen()`, `compose()`, `identity()` |
| `BinaryOperator<T>` | `T apply(T, T)` | `T`类型的二元运算符 | `andThen()`, `maxBy()`, `minBy()` |
| `Predicate<T>` | `boolean test(T)` | `T`类型的谓词 | `and()`, `or()`, `not()`, `negate()`, `isEqual()` |
| `BiPredicate<T, U>` | `boolean test(T, U)` | `T`和`U`类型的谓词 | `and()`, `or()`, `negate()` |

现在让这个例子更复杂一些。我们希望告诉这个动作它出现在哪一次循环中（即将循环变量`i`传给`action`）。为此，需要选择一个有`int`参数、无返回值的方法的函数式接口。处理`int`值的标准接口为`IntConsumer`。下面是`repeat()`方法的改进版本：

```java
public static void repeat(int n, IntConsumer action) {
    for (int i = 0; i < n; i++) action.accept(i);
}
```

可以这样调用它：

```java
repeat(10, i -> System.out.println("Countdown: " + (9 - i)));
```

下表列出了基本类型`int`、`long`和`double`的特殊化接口。由于减少了自动装箱，使用这些特殊化接口比泛型接口更高效。出于这个原因，在上面的例子中使用了`IntConsumer`而不是`Consumer<Integer>`。

| 函数式接口 | 抽象方法 |
| --- | --- |
| `BooleanSupplier` | `boolean getAsBoolean()` |
| `PSupplier` | `p getAsP()` |
| `PConsumer` | `void accept(p)` |
| `ObjPConsumer<T>` | `void accept(T, p)` |
| `PFunction<R>` | `R apply(p)` |
| `PToQFunction` | `q applyAsQ(p)` |
| `ToPFunction<T>` | `p applyAsP(T)` |
| `ToPBiFunction<T, U>` | `p applyAsP(T, U)` |
| `PUnaryOperator` | `p applyAsP(p)` |
| `PBinaryOperator` | `p applyAsP(p, p)` |
| `PPredicate` | `boolean test(p)` |

注：`p`, `q`是`int`, `long`, `double`；`P`, `Q`是`Int`, `Long`, `Double`。

提示：最好使用上面两个表中的接口。例如，假设要编写一个方法来处理满足特定条件的文件。对此有一个遗留接口`java.io.FileFilter`，不过最好使用标准的`Predicate<File>`。不这么做的唯一原因是，你已经有很多有用的方法生成`FileFilter`实例。

注释：大多数标准函数式接口都提供了非抽象方法来生成或组合函数。例如，`Predicate.isEqual(a)`等同于`a::equals`，但如果`a`为`null`也能正常工作。`Predicate`还提供了默认方法`and()`、`or()`和`negate()`来组合谓词。例如，`Predicate.isEqual(a).or(Predicate.isEqual(b))`等同于`x -> a.equals(x) || b.equals(x)`。

注释：如果设计自己的接口只有一个抽象方法，可以用`@FunctionalInterface`注解来标记。这样做有两个优点。如果你无意中添加了另一个抽象方法，编译器会给出错误消息。另外，javadoc页面中会指出你的接口是一个函数式接口。并不是必须使用注解。根据定义，任何只有一个抽象方法的接口都是函数式接口。不过使用`@FunctionalInterface`注解确实是一个好主意。

注释：有些程序员喜欢链式方法调用，例如

```java
String input = " 618970019642690137449562111 ";
boolean isPrime = input.strip().transform(BigInteger::new).isProbablePrime(20);
```

`String`类的`transform()`方法（Java 12中新增）对字符串应用一个`Function`并返回结果。同样地，这些调用也可以写为

```java
boolean prime = new BigInteger(input.strip()).isProbablePrime(20);
```

但是这样你的视线必须左右跳来跳去，找出哪个先执行哪个后执行。不过如果你更喜欢按顺序从左到右的链式方法调用，`transform()`会是你的得力助手。遗憾的是，它只适用于字符串。`Object`类没有`transform()`方法。

### 6.2.8 再谈Comparator
`Comparator`接口包含很多方便的静态方法来创建比较器。这些方法旨在与lambda表达式或方法引用一起使用。

静态方法`comparing()`接受一个“键提取器”函数（将类型`T`映射为可比较的类型）并返回一个比较器。该比较器对要比较的对象应用这个函数，然后对返回的键进行比较。例如，假设有一个`Person`对象数组，可以如下对其按名字进行排序：

```java
Arrays.sort(people, Comparator.comparing(Person::getName));
```

与手动实现一个`Comparator`相比，这当然要容易得多，代码也更为清晰。

可以用`thenComparing()`方法把比较器串起来，用于打破平局（如果第一个比较器判断两个对象相等，则使用第二个比较器）。例如：

```java
Arrays.sort(people, Comparator.comparing(Person::getLastName)
    .thenComparing(Person::getFirstName));
```

首先比较姓，如果两个人的姓相同，则比较名。

这些方法有几个变体形式。可以为`comparing()`和`thenComparing()`方法提取的键指定一个比较器。例如，可以如下按姓名长度排序：

```java
Arrays.sort(people, Comparator.comparing(Person::getName,
    (s, t) -> Integer.compare(s.length(), t.length())));
```

另外，`comparing()`和`thenComparing()`方法都有基本类型的变体，可以避免装箱。例如，上面的排序还有一种更容易的做法：

```java
Arrays.sort(people, Comparator.comparingInt(p -> p.getName().length()));
```

或者

```java
Arrays.sort(people, Comparator.comparing(Person::getName, Comparator.comparingInt(String::length)));
```

如果键函数可能返回`null`，可以使用`nullsFirst()`和`nullsLast()`适配器。这些静态方法会修改现有的比较器，从而在遇到`null`值时不会抛出异常，而是认为其小于或大于正常值。例如，假设一个人没有中名时`getMiddleName()`返回`null`，就可以使用`comparing(Person::getMiddleName, nullsFirst(...))`。

`naturalOrder()`方法可以为任何实现了`Comparable`的类创建一个比较器（直接利用其`compareTo()`方法进行比较）。下面是按可能为`null`的中名排序的完整调用。这里使用了`import static java.util.Comparator.*;`使这个表达式更清晰。

```java
Arrays.sort(people, comparing(Person::getMiddleName, nullsFirst(naturalOrder())));
```

使用`reversed()`实例方法得到逆序比较器。静态方法`reverseOrder()`提供自然顺序的逆序，等同于`naturalOrder().reversed()`。

[comparator/ComparatorTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch06/comparator/ComparatorTest.java)

小结：

| 比较器 | 等价形式 | 说明 |
| --- | --- | --- |
| `comparing(f)` | `(x, y) -> f(x).compareTo(f(y))` | `f`的返回类型必须实现`Comparable`接口 |
| `comparing(f, comp)` | `(x, y) -> comp.compare(f(x), f(y))` | |
| `comp1.thenComparing(comp2)` | `(x, y) -> comp1.compare(x, y) != 0 ? comp1.compare(x, y) : comp2.compare(x, y)` | |
| `naturalOrder()` | `(x, y) -> x.compareTo(y)` | 比较的类型必须实现`Comparable`接口 |
| `reverseOrder()` | `(x, y) -> y.compareTo(x)` | 比较的类型必须实现`Comparable`接口 |
| `comp.reversed()` | `(x, y) -> comp.compare(y, x)` | |

## 6.3 内部类
**内部类**(inner class)是定义在另一个类中的类。使用内部类有两个原因：
* 内部类可以对同一个包中的其他类隐藏。
* 内部类方法可以访问所在作用域中的数据，包括私有的数据。

C++注释：在Java中，内部类的对象有一个隐式引用，指向实例化这个对象的外部类对象。而静态内部类没有这个引用。Java的静态内部类相当于C++的嵌套类。

### 6.3.1 使用内部类访问对象状态
下面重构`TimerTest`示例，提取出一个`TalkingClock`类。构造语音时钟需要两个参数：发出通知的间隔和开关铃声的标志。

```java
public class TalkingClock {
    private int interval;
    private boolean beep;
    public TalkingClock(int interval, boolean beep) { ... }
    public void start() { ... }

    // an inner class
    public class TimePrinter implements ActionListener {
        ...
    }
}
```

`TimePrinter`类是`TalkingClock`的内部类。下面是`actionPerformed()`方法的实现：

```java
public void actionPerformed(ActionEvent event) {
    System.out.println("At the tone, the time is " + Instant.ofEpochMilli(event.getWhen()));
    if (beep) Toolkit.getDefaultToolkit().beep();
}
```

令人惊讶的是，`TimePrinter`并没有名为`beep`的实例字段或变量。实际上，`beep`引用创建这个`TimePrinter`的`TalkingClock`对象中的字段。

可以看到，内部类方法既可以访问自身的实例字段，也可以访问创建它的外部类对象的实例字段。为此，**内部类对象有一个隐式引用，指向创建它的外部类对象**（如下图所示）。

![内部类对象有一个外部类对象的引用](/assets/images/java-note-v1ch06-interfaces-lambda-expressions-and-inner-classes/内部类对象有一个外部类对象的引用.png)

这个引用在内部类的定义中是不可见的。不过，为了说明这个概念，这里将其称为 "outer" 。于是，`actionPerformed()`方法等价于：

```java
public void actionPerformed(ActionEvent event) {
    ...
    if (outer.beep) Toolkit.getDefaultToolkit().beep();
}
```

外部类的引用在构造器中设置。编译器会修改所有的内部类构造器，添加一个外部类引用的参数（注：这意味着不能单独实例化内部类）。`TimePrinter`类没有定义构造器，因此编译器会生成一个无参数构造器，生成的代码如下所示：

```java
// automatically generated code
public TimePrinter(TalkingClock clock) {
    outer = clock;
}
```

注意， "outer" 不是Java关键字，这里只是用它说明内部类的机制。

在`TalkingClock.start()`方法中构造一个`TimePrinter`对象时，编译器就会将当前`TalkingClock`的`this`引用传递给这个构造器：

```java
var listener = new TimePrinter(this); // parameter automatically added
```

程序清单6-7给出了测试这个内部类的完整程序。

[程序清单6-7 innerClass/InnerClassTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch06/innerClass/InnerClassTest.java)

注释：也可以将`TimePrinter`类声明为`private`。这样，只有`TalkingClock`类的方法才能构造`TimePrinter`对象。只有内部类可以是私有的，常规类只能是公有或包访问。

### 6.3.2 内部类的特殊语法规则
在上一节中，将外部类的引用叫做 "outer" 。实际上，这个引用的正规语法还要复杂一些。表达式`OuterClass.this`表示外部类引用。例如，可以将`TimePrinter.actionPerformed()`方法写为：

```java
public void actionPerformed(ActionEvent event) {
    ...
    if (TalkingClock.this.beep) Toolkit.getDefaultToolkit().beep();
}
```

反过来，可以使用以下语法构造内部类对象：

```java
outerObject.new InnerClass(parameters)
```

在这里，`outerObject`就是内部类对象的外部类引用。例如：

```java
var jabberer = new TalkingClock(1000, true);
TalkingClock.TimePrinter listener = jabberer.new TimePrinter();
```

注意，在外部类的作用域之外，需要这样引用内部类：`OuterClass.InnerClass`。

注释：内部类中声明的静态字段必须是`final`，并初始化为一个编译时常量。内部类不能有静态方法。

### 6.3.3 内部类是否有用、必要和安全
不可否认，内部类的语法很复杂。内部类与其他语言特性（如访问控制和安全性）之间如何交互不是很明确。

编译器会将内部类编译为常规类文件，用`$`分隔外部与内部类名。例如，`TalkingClock.TimePrinter`类将被编译成类文件TalkingClock$TimePrinter.class。可以尝试下面的实验：运行第5章的`ReflectionTest`程序，并提供`innerClass.TalkingClock$TimePrinter`类来进行反射：

```shell
java -cp .:../v1ch05 reflection.ReflectionTest innerClass.TalkingClock\$TimePrinter
```

或者，也可以直接使用`javap`工具：

```shell
javap -private innerClass.TalkingClock\$TimePrinter
```

注释：如果使用UNIX，需要对`$`进行转义。

将会得到以下输出：

```java
public class innerClass.TalkingClock$TimePrinter implements java.awt.event.ActionListener {
  final innerClass.TalkingClock this$0;
  public innerClass.TalkingClock$TimePrinter(innerClass.TalkingClock);
  public void actionPerformed(java.awt.event.ActionEvent);
}
```

可以清楚地看到，编译器生成了一个额外的实例字段`this$0`，表示外部类的引用（不能在自己的代码中引用它）。另外，还可以看到构造器的`TalkingClock`参数。

如果将`TimePrinter`定义成一个常规类并手动实现这种机制，会发现无法访问`outer.beep`：

```java
class TimePrinter implements ActionListener {
    private TalkingClock outer;
    public TimePrinter(TalkingClock clock) {
        outer = clock;
    }
    @Override
    public void actionPerformed(ActionEvent event) {
        ...
        if (outer.beep) ... // Error
    }
}
```

内部类可以访问外部类的私有数据，但常规类不行。可见由于内部类拥有更大的访问权限，所以比常规类功能更加强大。

内部类如何得到那些额外的访问权限呢？在Java 11之前，内部类纯粹是一种编译器现象，虚拟机对此并不了解。那时，用`ReflectionTest`程序或者`javap`工具查看`TalkingClock`类，会显示：

```java
class innerClass.TalkingClock {
  private int interval;
  private boolean beep;
  public innerClass.TalkingClock(int, boolean);
  public void start();
  static boolean access$000(innerClass.TalkingClock);
}
```

注意编译器在外部类添加的静态方法`access$000`，它返回参数对象的`beep`字段。

这是一个潜在的安全风险，而且会使分析类文件的工具变得复杂。从Java 11开始，虚拟机了解类之间的嵌套关系，不再生成这种访问方法。

### 6.3.4 局部内部类
在`TalkingClock`示例中，`TimePrinter`类的名字只出现了一次：在`start()`方法中创建这个类型的对象时。

在这种情况下，可以在方法中定义**局部类**(local class)：

```java
public void start() {
    class TimePrinter implements ActionListener {...}
    var listener = new TimePrinter();
    var timer = new Timer(interval, listener);
    timer.start();
}
```

声明局部类时不能有访问修饰符。其作用域限定在声明这个局部类的块中。

局部类有一个很大的优点：对外部完全隐藏。除了`start()`，没有任何方法知道`TimePrinter`类的存在。

[localInnerClass/LocalInnerClassTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch06/localInnerClass/LocalInnerClassTest.java)

小结：
* 类中的方法：实例方法（最常见的情况）
* 类中的类：内部类
* 方法中的类：局部类
* 方法中的方法：不合法

### 6.3.5 访问外部方法的变量
局部类不仅能访问外部类的字段，还能访问外部方法的局部变量。不过，那些局部变量必须是事实最终变量（注：这与lambda表达式访问自由变量的道理是一样的，见6.2.6节）。

下面是一个典型的示例，将`TalkingClock`构造器的`interval`和`beep`参数移至`start()`方法：

```java
public void start(int interval, boolean beep) {
    class TimePrinter implements ActionListener {
        @Override
        public void actionPerformed(ActionEvent event) {
            ...
            if (beep) Toolkit.getDefaultToolkit().beep();
        }
    }
    var listener = new TimePrinter();
    var timer = new Timer(interval, listener);
    timer.start();
}
```

要让`actionPerformed()`方法正常工作，`TimePrinter`类必须在`beep`参数消失（`start()`方法结束）之前将其保留下来。实际上也是这样做的。如果再次用`ReflectionTest`程序或者`javap`工具查看`TalkingClock$1TimePrinter`类，将会得到以下输出：

```java
class innerClass.TalkingClock$1TimePrinter implements java.awt.event.ActionListener {
  final boolean val$beep;
  final innerClass.TalkingClock this$0;
  innerClass.TalkingClock$1TimePrinter();
  public void actionPerformed(java.awt.event.ActionEvent);
}
```

创建`TimePrinter`对象时，`beep`变量的当前值会存储在`val$beep`字段中。即使局部变量超出作用域，内部类字段也会持久保存。

### 6.3.6 匿名内部类
使用局部类时，如果只想创建单个对象，甚至不需要给这个类命名。这称为**匿名内部类**(anonymous inner class)。

```java
public void start(int interval, boolean beep) {
    var listener = new ActionListener() {
        @Override
        public void actionPerformed(ActionEvent event) {
            System.out.println("At the tone, the time is " + Instant.ofEpochMilli(event.getWhen()));
            if (beep) Toolkit.getDefaultToolkit().beep();
        }
    };
    var timer = new Timer(interval, listener);
    timer.start();
}
```

其含义是：创建一个实现`ActionListener`接口的（匿名）类的新对象。

一般地，语法如下：

```java
new SuperType(parameters) {
    // inner class methods and data
}
```

其中，`SuperType`可以是接口，这样匿名内部类将实现这个接口；也可以是类，这样匿名内部类将扩展这个类。

由于匿名内部类没有名字，因此不能有构造器。构造器参数被传给**超类构造器**。如果匿名内部类实现一个接口，就不能有任何构造参数，但仍然要提供小括号（如上面示例所示）。

注意构造普通对象与构造匿名内部类对象之间的区别：

```java
// a Person object
var queen = new Person("Mary");

// an object of an inner class extending Person
var count = new Person("Dracula") { ... };
```

如果构造参数列表的小括号后面跟着一个大括号，就是在定义匿名内部类。

注释：尽管匿名类不能有构造器，但可以提供对象初始化块：

```java
var count = new Person("Dracula") {
    { initialization }
    ...
};
```

程序清单6-8包含了使用匿名内部类实现语音时钟程序的完整代码。

[程序清单6-8 anonymousInnerClass/AnonymousInnerClassTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch06/anonymousInnerClass/AnonymousInnerClassTest.java)

多年来，Java程序员习惯使用匿名内部类实现事件监听器和其他回调。现在最好使用lambda表达式。例如，使用lambda表达式来编写`start()`方法会简洁得多：

```java
public void start(int interval, boolean beep) {
    var timer = new Timer(interval, event -> {
        System.out.println("At the tone, the time is " + Instant.ofEpochMilli(event.getWhen()));
        if (beep) Toolkit.getDefaultToolkit().beep();
    });
    timer.start();
}
```

注释：如果将一个匿名类实例存储在一个用`var`定义的变量中，这个变量会了解增加的方法或字段：

```java
var bob = new Object() { String name = "Bob"; }
System.out.println(bob.name);
```

如果将`bob`的类型声明为`Object`，`bob.name`将无法编译。`bob`的实际类型为“有一个`String name`字段的`Object`”。这是一个**不可表示的**(non-denotable)类型，即无法用Java语法表达。不过，编译器理解这个类型。（注：所有的匿名类实际上都有名字，可以通过`getClass().getName()`查看。例如，假设上面的代码在`Foo`类的方法中，则`bob`的类型名为`Foo$1`。）

注释：下面的技巧称为**双大括号初始化**(double brace initialization)，利用了内部类语法。例如：

```java
invite(new ArrayList<String>() { { add("Harry"); add("Tony"); } });
```

外层大括号创建了一个`ArrayList`的匿名子类，内层大括号是对象初始化块（见4.6.7节）。在实际中，很少使用这个技巧。可以直接用`Arrays.asList()`或`List.of()`（Java 9新增）。

警告：使用匿名内部类时要当心`equals()`方法。在第5章中，曾建议`equals()`方法使用`getClass()`检测。匿名子类会时这个测试失败。

提示：生成日志或调试消息时，通常希望包含当前类的类名，例如

```java
System.err.println("Something awful happened in " + getClass());
```

但是不能用于静态方法。因为`getClass()`调用`this.getClass()`，而静态方法没有`this`。应该使用以下表达式：

```java
new Object(){}.getClass().getEnclosingClass() // gets class of static method
```

在这里，`new Object(){}`创建一个`Object`的匿名子类对象，`getEnclosingClass()`获取其外围类，也就是包含这个静态方法的类。

### 6.3.7 静态内部类
有时，使用内部类只是为了把一个类隐藏在另一个类的内部，并不需要内部类有外部类对象的引用。为此，可以将内部类声明为`static`。

下面是一个使用静态内部类的典型例子。考虑计算数组中的最小值和最大值的任务。当然，可以编写两个方法，一个计算最小值，另一个计算最大值。在调用这两个方法时，数组被遍历了两次。如果只遍历数组一次，同时计算最小值和最大值，这样会更高效。

不过，这个方法必须返回两个数。为此，可以定义一个包含两个值的类`Pair`：

```java
class Pair {
    private double first;
    private double second;

    public Pair(double f, double s) {
        first = f;
        second = s;
    }
    public double getFirst() { return first; }
    public double getSecond() { return second; }
}
```

`minmax()`方法可以返回一个`Pair`类型的对象：

```java
class ArrayAlg {
    public static Pair minmax(double[] values) {
        ...
        return new Pair(min, max);
    }
}
```

当然，`Pair`是一个十分大众化的名字。在大型项目中，其他人很有可能也使用这个名字。可以将`Pair`定义为`ArrayAlg`的公有内部类，从而解决潜在的命名冲突。这样就可以通过`ArrayAlg.Pair`访问这个类。

不过，与前面的例子中所使用的内部类不同，我们不希望`Pair`对象中有其他对象的引用。为此，可以将其声明为`static`：

```java
class ArrayAlg {
    public static class Pair {
        ...
    }
    ...
}
```

只有内部类可以声明为`static`。静态内部类与其他内部类完全一样，除了静态内部类对象没有外部类对象的引用。在这个示例中，必须使用静态内部类，因为`Pair`对象是在静态方法`minmax()`中构造的。

注释：只要内部类不需要访问外部类对象，就应该使用静态内部类。

注释：与常规内部类不同，静态内部类可以有静态字段和方法。

注释：在接口中声明的类自动是`static`和`public`。在类中声明的接口、记录和枚举自动是`static`。

程序清单6-9包含`ArrayAlg`类和内部类`Pair`的完整代码。

[程序清单6-9 staticInnerClass/StaticInnerClassTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch06/staticInnerClass/StaticInnerClassTest.java)

## 6.4 服务加载器
JDK提供了一个加载服务的简单机制。提供一个服务时，通常希望给服务设计者一些关于如何实现服务特性的自由。另外还希望有多个实现可供选择。利用`ServiceLoader`类可以很容易地加载符合一个公共接口的服务。

定义一个接口（或超类），包含服务的各个实例应该提供的方法。例如，假设要提供加解密服务：

```java
package serviceLoader;

public interface Cipher {
    byte[] encrypt(byte[] source, byte[] key);
    byte[] decrypt(byte[] source, byte[] key);
    int strength();
}
```

服务提供者可以提供一个或多个实现这个服务的类，例如：

```java
package serviceLoader.impl;

public class CaesarCipher implements Cipher {
    public byte[] encrypt(byte[] source, byte[] key) { ... }
    public byte[] decrypt(byte[] source, byte[] key) { ... }
    public int strength() { return 1; }
}
```

实现类可以在任意包中。每个实现类必须有一个无参数构造器。

现在把这些类的名字添加到META-INF/services目录下的一个文本文件中，文件名必须与接口的完全限定名一致。在这个例子中，文件META-INF/services/serviceLoader.Cipher包含

```
serviceLoader.impl.CaesarCipher
```

完成这个准备工作之后，程序初始化一个服务加载器，如下：

```java
public static ServiceLoader<Cipher> cipherLoader = ServiceLoader.load(Cipher.class);
```

这应该在程序中只做一次。

服务加载器的`iterator()`方法返回一个迭代器，遍历所有提供的服务实现（有关迭代器详见第9章）。最简单的做法是使用for each循环进行遍历，选择一个适当的对象来完成服务。

```java
public static Cipher getCipher(int minStrength) {
    for (Cipher cipher : cipherLoader) { // implicitly calls cipherLoader.iterator()
        if (cipher.strength() >= minStrength) return cipher;
    }
    return null;
}
```

或者，也可以使用流（见卷II第1章）：

```java
public static Optional<Cipher> getCipher2(int minStrength) {
    return cipherLoader.stream()
        .filter(descr -> descr.type() == serviceLoader.impl.CaesarCipher.class)
        .findFirst()
        .map(ServiceLoader.Provider::get);
}
```

[serviceLoader/ServiceLoaderTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch06/serviceLoader/ServiceLoaderTest.java)

使用以下命令编译、打包并运行这个程序：

```shell
$ javac serviceLoader/ServiceLoaderTest.java serviceLoader/impl/CaesarCipher.java
$ jar cfe ServiceLoaderTest.jar serviceLoader.ServiceLoaderTest serviceLoader/*.class serviceLoader/impl/*.class META-INF
$ java -jar ServiceLoaderTest.jar
Phhw#ph#dw#wkh#wrjd#sduw|1
```

## 6.5 代理
**代理**(proxy)用于在运行时创建实现一组给定接口的新类。只有在编译时无法确定需要实现哪个接口时才有必要使用代理。对于编写应用程序的程序员来说，这种情况很少见。不过，对于某些系统应用（例如Spring），代理提供的灵活性可能十分重要。

### 6.5.1 何时使用代理
假设你想构造一个类的对象，这个类实现了一个或多个接口，但是在编译时不知道这些接口的确切类型。这个问题确实有些难度。要构造一个具体类的对象，可以使用反射找出构造器并使用`newInstance()`方法，但是不能实例化接口。需要在运行的程序中定义一个新类。

为了解决这个问题，有些程序会生成代码，将代码放在一个文件中，调用编译器，然后再加载得到的类文件。自然，这会比较慢，并且需要将编译器和程序一起部署。代理机制是一种更好的解决方案。代理可以在运行时创建全新的类，并实现指定的接口。具体地，代理类包含以下方法：
* 指定接口所需的全部方法
* `Object`类定义的全部方法

然而，不能在运行时为这些方法定义新代码，而是要提供一个**调用处理器**(invocation handler)。调用处理器是实现了`InvocationHandler`接口的类的对象。这个接口只有一个方法：

```java
Object invoke(Object proxy, Method method, Object[] args)
```

每当调用代理对象的方法时，调用处理器的`invoke()`方法就会被调用，并提供`Method`对象和原调用参数。之后调用处理器必须确定如何处理这个调用。

### 6.5.2 创建代理对象
要创建代理对象，需要使用`Proxy`类的`newProxyInstance()`方法。这个方法有三个参数：
* 一个**类加载器**(class loader)
* 一个`Class`对象数组（对应要实现的各个接口）
* 一个调用处理器

如何定义处理器？对于得到的代理对象能够做什么？这两个问题的答案取决于我们想用代理机制解决什么问题。代理可用于多种目的，例如：
* 将方法调用路由到远程服务器
* 在程序运行期间将用户界面事件与动作关联起来
* 为了调试而跟踪方法调用

在示例程序中，我们使用代理和调用处理器跟踪方法调用。我们定义了一个`TraceHandler`类存储一个包装的对象，其`invoke()`方法会打印所调用方法的名字和参数，然后对包装的对象调用这个方法。

```java
class TraceHandler implements InvocationHandler {
    private Object target;

    public TraceHandler(Object t) { target = t; }

    public Object invoke(Object proxy, Method m, Object[] args)  throws Throwable {
        // print method name and parameters
        ...
        // invoke actual method
        return m.invoke(target, args);
    }
}
```

可以如下构造一个代理对象：

```java
Object value = ...;
// construct wrapper
var handler = new TraceHandler(value);
// construct proxy for one or more interfaces
var interfaces = new Class[] {Comparable.class};
Object proxy = Proxy.newProxyInstance(ClassLoader.getSystemClassLoader(), interfaces, handler);
```

现在，只要在`proxy`上调用了某个接口的方法（例如`compareTo()`），就会打印这个方法的名字和参数，之后再对`value`调用这个方法。

在程序清单6-10所示的程序中，使用代理对象跟踪二分查找。

[程序清单6-10 proxy/ProxyTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch06/proxy/ProxyTest.java)

其中，代理对象属于一个在运行时定义的类（其名字类似于`$Proxy0`）。这个类也实现了`Comparable`接口，不过它的`compareTo()`方法调用处理器的`invoke()`方法。这个方法会打印方法名和参数，之后在包装的`Integer`对象上调用`compareTo()`。

注：`Arrays.binarySearch()`方法的第一个参数就是`Object[]`，在方法中会将元素强制转换为`Comparable`。

下面是程序运行时完整的跟踪结果：

```
500.compareTo(288)
250.compareTo(288)
375.compareTo(288)
312.compareTo(288)
281.compareTo(288)
296.compareTo(288)
288.compareTo(288)
288.toString()
```

### 6.5.3 代理类的特性
代理类是在程序运行过程中动态创建的。不过，一旦被创建，它们就是常规的类，与虚拟机中的其他类没有什么区别。

所有的代理类都扩展`Proxy`类。代理类只有一个实例字段——调用处理器，定义在`Proxy`超类中。完成代理对象的任务所需的任何额外数据都必须存储在调用处理器中。例如，在程序清单6-10中，`TraceHandler`包装了实际的对象。

所有的代理类都覆盖了`Object`类的`toString()`、`equals()`和`hashCode()`方法。与所有代理方法一样，这些方法只是在处理器上调用`invoke()`。`Object`类中的其他方法没有重新定义。

代理类的名字没有定义。Oracle虚拟机中的`Proxy`类会生成以`$Proxy`开头的类名。

对于特定的类加载器和一组接口，只有一个代理类。也就是说，如果使用同一个类加载器和接口数组调用两次`newProxyInstance()`方法，将得到同一个代理类的两个对象。可以利用`getProxyClass()`方法获得这个类：`Class proxyClass = Proxy.getProxyClass(classLoader, interfaces);`

代理类总是`public`和`final`。如果代理类实现的所有接口都是`public`，这个代理类就不属于任何特定的包；否则，所有非公有接口必须属于同一个包，并且代理类也属于这个包。

可以通过调用`Proxy.isProxyClass()`方法检测一个特定的`Class`对象是否表示一个代理类。

注释：调用代理的默认方法会触发调用处理器。要真正调用这个方法，使用`InvocationHandler`接口的静态方法`invokeDefault()`。例如：

```java
InvocationHandler handler = (proxy, method, args) -> {
    if (method.isDefault())
        return InvocationHandler.invokeDefault(proxy, method, args)
    else
        return method.invoke(target, args);
};
```
