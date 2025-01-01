---
title: 《Java核心技术》笔记 第5章 继承
date: 2024-09-05 22:00:23 +0800
categories: [Java, Core Java]
tags: [java, class, inheritance, polymorphism, type conversion, array list, variadic argument, enumeration, sealed class, reflection, exception, jar file]
---
本章将学习面向对象程序设计的另一个基本概念：**继承**(inheritance)。继承的基本思想是可以基于已有的类创建新的类。继承已有的类就是复用（继承）这些类的方法和字段，而且可以添加新的方法和字段，以满足新的需求。这是Java编程中的一项核心技术。

## 5.1 类、超类和子类
回到上一章讨论的`Employee`类。假设经理的待遇与普通员工存在差异，经理除了领取薪水还能得到奖金。这种情形就需要使用继承，因为需要定义一个新类`Manager`并增加一些功能，但可以重用`Employee`类已有的部分功能，并保留所有字段。更抽象地说，每个经理都**是一名**员工： "is-a" 关系是继承的标志。

### 5.1.1 定义子类
下面定义从`Employee`类继承的`Manager`类，使用关键字`extends`表示继承。

```java
public class Manager extends Employee {
    added methods and fields
}
```

C++注释：在Java中所有的继承都是公有继承，而没有C++中的私有继承和受保护继承。

关键字`extends`表示正在构造的新类派生于一个已存在的类。这个已存在的类称为**超类**(superclass)、**基类**(base class)或**父类**(parent class)，新类称为**子类**(subclass/child class)或**派生类**(derived class)。

超类并不是比子类拥有更多的功能。实际上恰恰相反，子类比超类拥有更多的功能。

注释：前缀“超”(super)和“子”(sub)来自数学中的集合语言。员工集合是经理集合的超集，也可以说经理集合是员工集合的子集。

`Manager`类增加了一个用于存储奖金的新字段，以及一个用于设置这个字段的新方法：

```java
public class Manager extends Employee {
    private double bonus;
    ...
    public void setBonus(double bonus) {
        this.bonus = bonus;
    }
}
```

如果有一个`Manager`对象，就可以使用`setBonus()`方法。当然，如果有一个`Employee`对象，不能使用`setBonus()`方法，因为这不是`Employee`类中定义的方法。不过，可以对`Manager`对象使用`getName()`和`getHireDay()`方法。尽管没有在`Manager`类中显式定义这些方法，它们也会自动地从超类`Employee`继承。

每个`Manager`对象都有4个字段：`name`、`salary`、`hireDay`和`bonus`。其中，前3个字段是从超类得来的。

通过扩展超类定义子类时，只需要指出子类与超类的**不同之处**。在设计类的时候，应该将最一般的方法放在超类中，而将更特殊的方法放在子类中。这种做法在面向对象程序设计中十分普遍。

注释：不能扩展记录，而且记录也不能扩展其他类。

### 5.1.2 覆盖方法
超类中的有些方法对子类并不一定适用。具体来说，`Manager`类的`getSalary()`方法应该返回基本工资和奖金的总和。为此，需要提供一个新的方法来**覆盖**(override)超类方法：

```java
public class Manager extends Employee {
    ...
    public double getSalary() {
        ...
    }
    ...
}
```

应该如何实现这个方法呢？乍一看似乎很简单，只要返回`salary`和`bonus`字段的总和就可以了：

```java
public double getSalary() {
    return salary + bonus; // won't work
}
```

然而，这样行不通。因为**子类方法不能直接访问超类的私有字段**。如果要访问私有字段，就必须像其他方法一样使用公共接口，在这里就是`Employee`类的`getSalary()`方法。

再试一下，调用`getSalary()`而不是直接访问`salary`字段：

```java
public double getSalary() {
    double baseSalary = getSalary(); // still won't work
    return baseSalary + bonus;
}
```

上面这段代码仍然有问题。问题在于调用`getSalary()`只是在调用自身（就是正在实现的这个方法）。结果是无限地调用自己，最终导致程序崩溃。

我们希望调用的是超类`Employee`中的`getSalary()`方法，而不是当前类的这个方法。为此，使用特殊的关键字`super`：`super.getSalary()`调用`Employee`类的`getSalary()`方法。下面是正确版本：

```java
public double getSalary() {
    double baseSalary = super.getSalary();
    return baseSalary + bonus;
}
```

注释：有些人认为`super`与`this`是类似的概念。然而，这种类比并不十分准确：`super`不是超类对象的引用。例如，不能将`super`赋给另一个对象变量。`super`只是一个指示编译器调用超类方法的特殊关键字。

子类可以增加字段、增加方法或覆盖超类的方法。然而，继承绝对不会删除任何字段或方法。

警告：覆盖方法时，子类方法的可见性**不能低于**超类方法。具体地，如果超类方法是`public`，子类方法也必须声明为`public`。

### 5.1.3 子类构造器
最后，提供一个构造器：

```java
public Manager(String name, double salary, int year, int month, int day) {
    super(name, salary, year, month, day);
    bonus = 0;
}
```

由于子类不能访问超类的私有字段，所以必须通过构造器来初始化这些字段。可以使用`super(...)`调用超类构造器，这个语句必须是子类构造器的第一条语句。

如果子类构造器没有显式地调用超类构造器，那么超类必须有一个无参数构造器。这个构造器将在子类构造之前调用。

注：C++和Python支持构造函数继承，而Java不支持。详见[关于超类构造函数的问题]({% post_url 2020-08-16-python-superclass-constructor %})。

注释：关键字`this`有两个含义：一是表示隐式参数，二是调用该类的其他构造器。类似地，关键字`super`也有两个含义：一是调用超类的方法，二是调用超类的构造器。用来调用构造器时，都是只能作为另一个构造器的第一条语句出现。

代码清单5-1的程序展示了`Employee`（程序清单5-2）和`Manager`（程序清单5-3）对象在薪水计算上的区别。

[程序清单5-1 inheritance/ManagerTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch05/inheritance/ManagerTest.java)

[程序清单5-2 inheritance/Employee.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch05/inheritance/Employee.java)

[程序清单5-3 inheritance/Manager.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch05/inheritance/Manager.java)

值得注意的是，`e.getSalary()`能选出**正确的** `getSalary()`方法。注意，尽管`e`的**声明**类型是`Employee`，但`e`引用的对象的**实际**类型可以是`Employee`或`Manager`。虚拟机知道`e`引用的对象的实际类型，因此能够调用正确的方法。

一个对象变量可以引用多种实际类型，这称为**多态**(polymorphism)。在运行时自动地选择适当的方法，这称为**动态绑定**(dynamic binding)。在本章中将详细讨论这两个概念。

C++注释：在C++中，如果希望实现动态绑定，需要将成员函数声明为`virtual`。在Java中，动态绑定是默认的行为。如果不希望一个方法是 "virtual" ，可以将它标记为`final`（将在5.1.7节介绍）。

### 5.1.4 继承层次结构
继承并不仅限于一个层次。例如，可以有一个继承`Manager`的`Executive`类。继承一个公共超类的所有类的集合称为**继承层次结构**(inheritance hierarchy)，如下图所示。在继承层次结构中，从某个类到其祖先的路径称为该类的**继承链**(inheritance chain)。

![Employee继承层次结构](/assets/images/java-note-v1ch05-inheritance/Employee继承层次结构.png)

C++注释：在C++中，一个类可以有多个超类（多重继承）。Java不支持多重继承，但提供了类似的功能，参见6.1节。

### 5.1.5 多态
有一个简单的规则可以用来判断是否应该将类设计为继承关系—— "is-a" 规则：每个子类对象都**是一个**超类对象。例如，每个经理都是一名员工，因此将`Manager`类设计为`Employee`类的子类是有道理的，反之则不然。

"is-a" 规则的另一种表述是**替换原则**(substitution principle)：程序中需要超类对象的任何地方都可以使用子类对象替换。例如，**可以将一个子类对象赋给超类变量**：

```java
Employee e;
e = new Employee(...); // Employee object expected
e = new Manager(...); // OK, Manager can be used as well
```

在程序清单5-1中就利用了替换原则：`staff[0] = boss;`

在Java中，对象变量是**多态的**(polymorphic)。一个`Employee`类型的变量既可以引用`Employee`类型的对象，也可以引用`Employee`类的任何子类的对象。

**不能将超类对象赋给子类变量。** 例如，下面的赋值是非法的：

```java
Manager m = staff[i]; // ERROR
```

原因很清楚：不是所有的员工都是经理。如果赋值成功，那么`m`有可能引用一个不是经理的`Employee`对象，而后面有可能会调用`m.setBonus()`，这就会发生运行时错误。

警告：在Java中，子类数组可以转换成超类数组，而不需要使用强制类型转换。例如：

```java
Manager[] managers = new Manager[10];
Employee[] staff = managers; // OK
```

但实际上会发生一些令人惊讶的事情。要切记`managers`和`staff`引用的是同一个数组。现在考虑这条语句：

```java
staff[0] = new Employee("Harry Hacker", ...);
```

假如编译器允许这个赋值，但是`staff[0]`和`managers[0]`是相同的引用，似乎我们把一个普通员工擅自归入经理行列中了。这非常糟糕，当调用`managers[0].setBonus(1000)`时，将会试图访问一个不存在的实例字段，进而破坏相邻内存的内容。

为了确保不发生这类错误，数组会记住创建时的元素类型，并监督仅将类型兼容的引用存储到数组中。例如，上面的赋值会引发`ArrayStoreException`。

### 5.1.6 理解方法调用
准确地理解方法调用如何应用于对象非常重要。假设要调用`x.f(args)`，隐式参数`x`声明为类`C`的对象。下面是详细过程：

1.编译器列举类`C`中所有名为`f`的方法和其超类中所有名为`f`的可访问的方法（超类的私有方法不可访问）。注意，可能存在多个重载。例如，可能有方法`f(int)`和`f(String)`。

至此，编译器已经获得所有可能调用的候选方法。

2.编译器将方法调用中提供的参数（实参）类型与方法声明中的参数（形参）类型进行匹配，选择要调用的方法。这一过程称为**重载解析**(overloading resolution)。例如，对于调用`x.f("Hello")`，编译器将选择`f(String)`而不是`f(int)`。由于允许类型转换（`int`到`double`，`Manager`到`Employee`，等等），这个过程可能会很复杂。如果没有找到匹配的方法，或者匹配到多个方法，编译器就会报错。

至此，编译器已经知道要调用的方法的名字和参数类型（即签名）。

注释：如果在子类中定义了一个与超类方法签名相同的方法，这个方法就会覆盖超类方法。返回类型不是签名的一部分。不过，在覆盖方法时，需要保证返回类型的兼容性。允许子类将覆盖方法的返回类型改为原类型的子类型。例如，假设`Employee`类有方法

```java
public Employee getBuddy() { ... }
```

子类`Manager`可以如下覆盖这个方法：

```java
public Manager getBuddy() { ... } // OK to change return type
```

我们说这两个`getBuddy()`方法有**协变**(covariant)的返回类型。

3.如果这个方法是`private`、`static`、`final`或构造器，那么编译器就确切地知道要调用哪个方法（因为不可能被子类覆盖）。这称为**静态绑定**(static binding)。否则，要调用的方法取决于隐式参数的实际类型，必须在运行时使用动态绑定。在这个示例中，编译器会生成一条采用动态绑定调用`f(String)`的指令。

4.程序运行时，虚拟机必须调用适合于`x`所引用对象的实际类型的那个方法。假设`x`的实际类型是`D`，它是`C`的子类。如果类`D`定义了方法`f(String)`，就调用这个方法；否则，在`D`的超类中寻找`f(String)`，以此类推。

每次调用方法时都进行这个搜索会很耗时。因此，虚拟机为每个类预先计算了一个**方法表**(method table)，其中列出了所有方法签名和要调用的实际方法（注：类似于C++的虚函数表）。这样一来，真正调用方法时，虚拟机只查找这个表就行了。

下面来看程序清单5-1中调用`e.getSalary()`的详细过程。`e`声明为`Employee`类型。`Employee`类只有一个名为`getSalary()`的方法，没有参数。因此，在这里不必担心重载解析的问题。

由于`getSalary()`方法不是`private`、`static`或`final`，所以采用动态绑定。虚拟机为`Employee`和`Manager`类生成方法表：

```
Employee:
    getName() -> Employee.getName()
    getSalary() -> Employee.getSalary()
    getHireDay() -> Employee.getHireDay()
    raiseSalary(double) -> Employee.raiseSalary(double)

Manager:
    getName() -> Employee.getName()
    getSalary() -> Manager.getSalary()
    getHireDay() -> Employee.getHireDay()
    raiseSalary(double) -> Employee.raiseSalary(double)
    setBonus(double) -> Manager.setBonus(double)
```

这里略去了`Object`方法。

在运行时，虚拟机获取`e`的实际类型的方法表，查找`getSalary()`签名，并调用对应的方法。
* 在第1次循环中，`e`引用`staff[0]`，实际类型为`Manager`，因此调用`Manager.getSalary()`。
* 在第2次和第3次循环中，`e`引用`staff[1]`和`staff[2]`，实际类型为`Employee`，因此调用`Employee.getSalary()`。

动态绑定有一个非常重要的特性：无需修改现有的代码就可以对程序进行**扩展**。假设新增一个子类`Executive`，不需要对包含调用`e.getSalary()`的代码重新编译，如果`e`引用一个`Executive`类的对象，就会自动地调用`Executive.getSalary()`方法。

### 5.1.7 阻止继承：final类和方法
有时，可能希望阻止其他人继承某个类。在类定义中使用`final`修饰符表示这个类不能被扩展。例如：

```java
public final class Executive extends Manager {
    ...
}
```

也可以将类中的某个特定方法声明为`final`，这样子类就不能覆盖这个方法。例如：

```java
public class Employee {
    ...
    public final String getName() {
        return name;
    }
    ...
}
```

注释：`final`类中的所有方法自动地成为`final`方法，但字段不会。

将方法或类声明为`final`只有一个原因：确保语义不会在子类中改变。例如，`String`类是`final`类，这意味着不允许定义`String`的子类。换言之，如果有一个`String`引用，它引用的一定是一个`String`对象，而不可能是其他类的对象。

注释：枚举和记录总是`final`，它们不允许扩展。

### 5.1.8 强制类型转换
有时候可能需要将一个类的对象引用转换成另一个类。对象引用的强制类型转换语法与数值表达式的强制类型转换类似。例如：

```java
Manager boss = (Manager) staff[0];
```

进行强制类型转换的唯一原因是：在暂时忘记对象的实际类型之后使用对象的全部功能。例如，在`ManagerTest`类中，将`boss`赋给`staff[0]`之后再调用`setBonus()`。

将子类引用赋给超类变量，编译器是允许的。但将超类引用赋给子类变量时，必须进行强制类型转换。如果引用对象的实际类型与转换的目标类型不兼容，将产生`ClassCastException`。例如：

```java
Manager boss = (Manager) staff[1]; // ERROR
```

如果没有捕获这个异常，程序就会终止。因此，在强制类型转换之前先查看是否能够成功地转换，这是一个良好的编程习惯。只需使用`instanceof`运算符。例如：

```java
if (staff[i] instanceof Manager) {
    Manager boss = (Manager) staff[i];
    boss.setBonus(5000);
}
```

注：
* 目标类型可以是实际类型本身或超类。例如，假设有4个类`D -> C -> B -> A`，变量`A x = new C()`，那么`(C) x`和`(B) x`都是合法的，但`(D) x`是非法的。
* Java中超类和子类引用之间的转换相当于C++中基类和派生类**指针或引用**之间的转换，因此不存在截断问题（见[《C++程序设计原理与实践》笔记第14章]({% post_url 2023-03-06-ppp-note-ch14-graphics-class-design %}) 14.2.4节）。

最后，如果目标类型不是实际类型的子类，编译器就不会允许这个转换。例如：

```java
String c = (String) staff[i];
```

将产生编译错误，因为`String`不是`Employee`的子类。

综上所述：
* 只能在继承层次结构内进行强制类型转换。
* 在将超类强制转换成子类之前，应该使用`instanceof`进行检查。

注释：如果`x`为`null`，则`x instanceof C`不会产生异常，只是返回`false`。

实际上，通过强制类型转换来转换对象的类型通常并不是一个好主意。如果出于某种原因发现需要对`Employee`对象调用`setBonus()`方法，那么就应该自问超类的设计是否有问题，可能重新设计超类并添加`setBonus()`方法才是合理的。一般情况下，最好尽量少用强制类型转换和`instanceof`运算符。

C++注释：Java使用的强制类型转换语法来自C语言“古老的过去”，但处理过程像C++安全的`dynamic_cast`操作。例如，

```java
Manager boss = (Manager) staff[i]; // Java
```

等价于

```cpp
Manager* boss = dynamic_cast<Manager*>(staff[i]); // C++
```

只有一点重要的区别：当转换失败时，Java不会返回`null`引用，而是抛出异常。从这个意义上讲，它像C++的引用转换。

### 5.1.9 instanceof模式匹配
先使用`instanceof`检查再进行强制类型转换的代码实在有些冗长。从Java 16起，有一种更简单的方式。可以直接在`instanceof`测试中声明子类变量：

```java
if (staff[i] instanceof Manager boss) {
    boss.setBonus(5000);
}
```

如果`staff[i]`是`Manager`类的一个实例，则变量`boss`设置为`staff[i]`，其类型为`Manager`，从而跳过强制类型转换。如果`staff[i]`并非引用一个`Manager`，那么不会设置`boss`，将跳过`if`语句的主体。

当`instanceof`模式引入一个变量时，可以立即在同一个表达式中使用：

```java
Employee e = ...;
if (e instanceof Manager m && m.getBonus() > 10000) ...
```

由于`&&`运算符的“短路”逻辑，如果会计算右边，说明`m`必然是一个`Manager`实例。

然而，下面的代码会导致编译错误：

```java
if (e instanceof Manager m || m.getBonus() > 10000) ... // ERROR
```

当`||`的左边为`false`时，会计算右边，而`m`并没有绑定到`Manager`实例。

注：`instanceof`模式引入变量的作用域**不限于**`if`语句内部，在`if`语句之后仍然可以使用。

下面是另一个使用条件运算符的例子：

```java
double bonus = e instanceof Manager m ? m.getBonus() : 0;
```

注释：声明变量的`instanceof`形式称为“模式匹配”，是因为它类似于`switch`中的类型模式，这是Java 17中的一个预览特性。下面是这个语法的一个例子：

```java
String description = switch (e) {
    case Executive exec -> "An executive with a fancy title of " + exec.getTitle();
    case Manager m -> "A manager with a bonus of " + m.getBonus();
    default -> "A lowly employee with a salary of " + e.getSalary();
}
```

警告：`instanceof`模式定义的局部变量也会遮蔽实例字段。

### 5.1.10 受保护访问
任何声明为`private`的特性都不能在其他类中访问，这对于子类也同样适用。不过，有时可能希望限制一个方法或字段只能被子类访问。在这种情况下，可以将其声明为受保护(`protected`)。例如，如果`Employee`将`hireDay`字段声明为`protected`而不是`private`，那么`Manager`方法就可以直接访问这个字段。

在Java中，受保护字段只能被子类和**同一个包**中的类访问。考虑与`Employee`类在不同包中的子类`Administrator`。`Administrator`类的方法只能访问`Administrator`对象的`hireDay`字段，而不能访问`Employee`对象的这个字段。例如：

```java
public boolean isSeniorTo(Administrator other) {
    return this.hireDay.isBefore(other.hireDay); // OK
}

public boolean isSeniorTo(Employee other) {
    return this.hireDay.isBefore(other.hireDay); // ERROR
}
```

如果`Administrator`和`Employee`在同一个包中，那么第二种写法也是合法的。

这个限制能避免滥用`protected`机制派生子类来访问受保护字段。

在实际应用中，要慎用受保护字段。因为其他程序员可能会继承你的类并访问受保护字段。在这种情况下，就不能修改你的类的实现而不影响那些程序员。这违背了OOP提倡数据封装的精神。

受保护的方法更有意义。一个很好的例子是`Object`类的`clone()`方法，详细内容参见第6章。

C++注释：Java中的受保护特性允许所有子类以及同一个包中的所有其他类访问，这与C++中受保护的含义稍有不同。

下面对Java中的4个访问控制修饰符做个小结：
* `public`：任何类可访问
* `private`：仅本类可访问
* `protected`：本包和所有子类可访问
* 无：本包可访问

注：假设`B`类方法访问`A`类的字段`x`，下表总结了各种情况下的可访问性。其中“是否通过当前类访问”是指通过`B`类对象访问还是`A`类对象访问。

| 修饰符 | 是否在相同包 | 是否是子类 | 是否通过当前类访问 | 可访问 |
| --- | --- | --- | --- | --- |
| `public` | - | - | - | √ |
| `private` | - | - | - | × |
| `protected` | √ | √ | √ | √ |
| `protected` | √ | √ | × | √ |
| `protected` | √ | × | - | √ |
| `protected` | × | √ | √ | √ |
| `protected` | × | √ | × | × |
| `protected` | × | × | - | × |
| 无 | √ | - | - | √ |
| 无 | × | - | - | × |

另见[Controlling Access to Members of a Class](https://docs.oracle.com/javase/tutorial/java/javaOO/accesscontrol.html)。

## 5.2 Object：所有类的超类
`Object`类是所有类的始祖，Java中的每个类都扩展了`Object`。如果没有明确地指出超类，那么`Object`就是这个类的超类。

### 5.2.1 Object类型的变量
可以使用`Object`类型的变量引用任何类型的对象：

```java
Object obj = new Employee("Harry Hacker", 35000);
```

当然，`Object`类型的变量只能用作任意值的通用容器。要对其中的值进行具体的操作，需要清楚原始类型并进行强制类型转换：

```java
Employee e = (Employee) obj;
```

在Java中，只有基本类型的值（数值、字符和布尔值）不是对象。

所有的数组类型（不管是对象数组还是基本类型的数组）都是扩展了`Object`的类类型。

```java
Employee[] staff = new Employee[10];
obj = staff; // OK
obj = new int[10]; // OK
```

### 5.2.2 equals方法
`Object`类的`equals()`方法检测一个对象是否等于另一个对象。在`Object`类中，这个方法判断两个对象引用是否相同（即`obj1.equals(obj2)`等价于`obj1 == obj2`）。这是一个合理的默认行为：如果两个对象相同就一定相等。然而，经常需要基于状态检测对象的相等性。例如，如果两个员工的姓名、薪水和雇佣日期都一样，就认为是相等的。

```java
public class Employee {
    ...
    public boolean equals(Object otherObject) {
        // a quick test to see if the objects are identical
        if (this == otherObject) return true;

        // must return false if the explicit parameter is null
        if (otherObject == null) return false;

        // if the classes don't match, they can't be equal
        if (getClass() != otherObject.getClass()) return false;

        // now we know otherObject is a non-null Employee
        Employee other = (Employee) otherObject;

        // test whether the fields have identical values
        return name.equals(other.name)
            && salary == other.salary
            && hireDay.equals(other.hireDay);
    }
}
```

`getClass()`方法返回对象的类（将在5.9.1节介绍）。

提示：为了防备`name`或`hireDay`可能为`null`的情况，可以使用`Objects.equals()`方法：如果两个参数都为`null`，`Objects.equals(a, b)`返回`true`；如果其中一个参数为`null`，则返回`false`；否则，调用`a.equals(b)`。利用这个方法，`Employee.equals()`方法的最后一条语句改写为：

```java
return Objects.equals(name, other.name)
    && salary == other.salary
    && Objects.equals(hireDay, other.hireDay);
```

在子类中定义`equals()`方法时，首先调用超类的`equals()`。如果检测失败，那么对象就不可能相等。如果超类字段都相等，则比较子类的字段。

```java
public class Manager extends Employee {
    ...
    public boolean equals(Object otherObject) {
        if (!super.equals(otherObject)) return false;
        // super.equals checked that this and otherObject belong to the same class
        Manager other = (Manager) otherObject;
        return bonus == other.bonus;
    }
}
```

注释：记录的状态完全由标准构造器中设置的字段定义。记录会自动定义一个比较字段的`equals()`方法。

### 5.2.3 相等测试与继承
如果隐式和显式参数不属于同一个类，`equals()`方法应该如何处理呢？这是一个很有争议的问题。在前面的例子中，如果类不匹配，`equals()`方法就返回`false`。但是许多程序员却喜欢使用`instanceof`检测：

```java
if (!(otherObject instanceof Employee)) return false;
```

这样允许`otherObject`属于一个子类。但是这种方法可能会带来麻烦。

Java语言规范要求`equals()`方法具有以下性质（详见API文档[Object.equals()](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Object.html#equals(java.lang.Object))）：
1. **自反性**(reflexive)：对于任何非空引用`x`，`x.equals(x)`应该返回`true`。
2. **对称性**(symmetric)：对于任何非空引用`x`和`y`，`x.equals(y)`返回`true`当且仅当`y.equals(x)`返回`true`。
3. **传递性**(transitive)：对于任何非空引用`x`、`y`和`z`，如果`x.equals(y)`返回`true`且`y.equals(z)`返回`true`，则`x.equals(z)`也应该返回`true`。
4. **一致性**(consistent)：如果`x`和`y`引用的对象没有发生变化，则反复调用`x.equals(y)`应该返回同样的结果。
5. 对于任何非空引用`x`，`x.equals(null)`应该返回`false`。

这些规则当然很合理。然而，当参数属于不同的类时，对称性规则会有微妙的结果。考虑调用`e.equals(m)`，其中`e`是一个`Employee`对象，`m`是一个`Manager`对象，并且二者恰好有相同的姓名、薪水和雇佣日期。如果`Employee.equals()`使用`instanceof`检测，则这个调用返回`true`。但是这意味着反过来调用`m.equals(e)`也需要返回`true`。这就使得`Manager`类陷入困境：其`equals()`方法必须能够将自己与任何`Employee`对象进行比较，而不考虑经理特有的那部分信息！

有些作者认为`getClass()`检测是错误的，因为它违反了替换原则。一个经常提到的例子是`AbstractSet`类的`equals()`方法，它检测两个集合是否有相同的元素。`AbstractSet`类有两个具体子类：`TreeSet`和`HashSet`，它们使用不同的算法查找集合元素。你肯定希望能够比较任意两个集合，不论它们如何实现（即能够比较两个不同子类的对象）。

总之，有两种不同的情形：
* 如果子类有自己的相等性概念，则使用`getClass()`检测以满足对称性（例如`Employee`和`Manager`）。
* 如果由超类决定相等性概念，则可以使用`instanceof`检测，这样不同子类的对象也可能相等（例如`AbstractSet`）。

注释：Java标准库包含150多个`equals()`方法的实现，包括使用`instanceof`、调用`getClass()`、捕获`ClassCastException`或者什么也不做等各种做法。查看`java.sql.Timestamp`类的API文档，在这里实现者不无尴尬地指出，他们让自己陷入了困境。`Timestamp`继承自`java.util.Date`，而后者的`equals()`方法使用了`instanceof`检测。这样一来就无法覆盖`equals()`，使之同时做到对称且正确。

下面是一个编写完美`equals()`方法的技巧：

1.将显式参数命名为`otherObject`，稍后需要将它强制转换成另一个名为`other`的变量。

2.检测`this`和`otherObject`是否相同（引用同一个对象）：

```java
if (this == otherObject) return true;
```

3.检测`otherObject`是否为`null`，如果是则返回`false`。这个检测是必要的。

```java
if (otherObject == null) return false;
```

4.比较`this`和`otherObject`的类。如果相等性语义可以在子类中改变，就使用`getClass()`检测：

```java
if (getClass() != otherObject.getClass()) return false;
ClassName other = (ClassName) otherObject;
```

如果相等性语义对所有的子类都相同，就使用`instanceof`检测：

```java
if (!(otherObject instanceof ClassName other)) return false;
```

5.根据相等性概念的要求来比较字段。使用`==`比较基本类型字段，使用`Object.equals()`比较对象字段。如果所有字段都匹配则返回`true`，否则返回`false`。

```java
return field1 == other.field1
    && Objects.equals(field2, other.field2)
    && ...;
```

如果在子类中覆盖`equals()`，则包含一个`super.equals(other)`调用。

提示：对于数组类型的字段，可以使用静态方法`Arrays.equals()`检查相应的数组元素是否相等。对于多维数组，使用`Arrays.deepEquals()`方法。

警告：下面是实现`equals()`方法时一个常见的错误：

```java
public class Employee {
    public boolean equals(Employee other) {
        return other != null
            && getClass() == other.getClass()
            && Objects.equals(name, other.name)
            && salary == other.salary
            && Objects.equals(hireDay, other.hireDay);
    }
    ...
}
```

这个方法声明的参数类型是`Employee`。结果并没有覆盖`Object`类的`equals()`方法，而是定义了一个完全无关的方法。为了避免这种错误，可以使用`@Override`标记要覆盖超类的方法：

```java
@Override public boolean equals(Object other)
```

如果没有覆盖超类方法，编译器将报错。

### 5.2.4 hashCode方法
**散列码**(hash code)是由对象导出的一个整数。散列码应该是没有规律的——两个不同的对象的散列码基本上不会相同。

`String`类使用以下算法计算散列码：

```java
int hash = 0;
for (int i = 0; i < length(); i++)
    hash = 31 * hash + charAt(i);
```

下表列出了几个字符串的散列码示例：

| 字符串 | 散列码 |
| --- | --- |
| Hello | 69609650 |
| Harry | 69496448 |
| Hacker | -2141031506 |

`hashCode()`方法定义在`Object`类中，因此每个对象都有一个默认的散列码，其值由对象的内存地址得出。

`hashCode()`方法应该返回一个整数（可以是负数）。要合理地组合实例字段的散列码，使得不同对象的散列码尽量分散开。例如，下面是`Employee`类的`hashCode()`方法：

```java
public int hashCode() {
    return 7 * name.hashCode()
        + 11 * Double.valueOf(salary).hashCode()
        + 13 * hireDay.hashCode();
}
```

不过，还可以做得更好。首先，使用null安全的方法`Objects.hashCode()`。如果参数为`null`则返回0，否则返回对参数调用`hashCode()`的结果。另外，使用静态方法`Double.hashCode()`来避免创建`Double`对象。

```java
public int hashCode() {
    return 7 * Objects.hashCode(name)
        + 11 * Double.hashCode(salary)
        + 13 * Objects.hashCode(hireDay);
}
```

更好的做法是，使用`Objects.hash()`组合多个对象的散列值。这样，`Employee.hashCode()`方法就是

```java
public int hashCode() {
    return Objects.hash(name, salary, hireDay);
}
```

如果重新定义了`equals()`方法，也需要重新定义`hashCode()`方法，以便用户可以将对象插入散列表中（散列表将在9.4节中讨论）。`equals()`和`hashCode()`的定义必须相容：如果`x.equals(y)`返回`true`，那么`x.hashCode()`和`y.hashCode()`必须返回相同的值。

提示：对于数组类型的字段，可以使用静态方法`Arrays.hashCode()`计算一个散列码，这个散列码由数组元素的散列码组成。

注释：记录会自动提供`hashCode()`方法，它会由字段值的散列码得出一个散列码。

### 5.2.5 toString方法
`Object`中的另一个重要的方法是`toString()`，它返回对象的字符串表示。下面是一个典型的例子，`java.awt.Point`类的`toString()`方法返回类似于`"java.awt.Point[x=10,y=20]"`的字符串。

大多数（但不是全部）`toString()`方法都遵循这样的格式：首先是类名，随后是方括号括起来的字段值。下面是`Employee`类的`toString()`方法的实现：

```java
public String toString() {
    return "Employee[name=" + name
        + ",salary=" + salary
        + ",hireDay=" + hireDay
        + "]";
}
```

实际上，最好通过调用`getClass().getName()`获得类名的字符串，而不要将类名硬编码到`toString()`方法中。

子类应该定义自己的`toString()`方法，只要调用`super.toString()`并添加子类的字段。例如，下面是`Manager`类的`toString()`方法：

```java
public String toString() {
    return super.toString()
        + "[bonus=" + bonus
        + "]";
}
```

现在，`Manager`对象将打印为`"Manager[name=...,salary=...,hireDay=...][bonus=...]"`。

`toString()`方法无处不在的重要原因是：只要将对象和字符串通过`+`运算符拼接起来，Java编译器就会自动地调用`toString()`方法获得对象的字符串表示。例如：

```java
var p = new Point(10, 20);
String message = "The current position is " + p; // automatically invokes p.toString()
```

提示：`x.toString()`也可以写作`"" + x`。与`toString()`不同的是，即使`x`是基本类型，这条语句也能正常工作。

如果`x`是任意对象，并调用`System.out.println(x);`，`println()`方法就会调用`x.toString()`，并打印得到的字符串。

`Object`类定义了`toString()`方法，会打印对象的类名和散列码。例如，调用`System.out.println(System.out)`将输出 "java.io.PrintStream@2f6684" （每次运行散列码都不同）。这是因为`PrintStream`没有覆盖`toString()`方法。

警告：令人烦恼的是，数组继承了`Object`类的`toString()`方法，数组类型采用一种古老的格式打印。例如：

```java
int[] luckyNumbers = { 2, 3, 5, 7, 11, 13 };
String s = "" + luckyNumbers;
```

会生成字符串 "[I@1a46e30" （前缀`[I`表示整型数组）。正确方式是调用静态方法`Arrays.toString()`。例如，`Arrays.toString(luckyNumbers)`生成字符串 "[2, 3, 5, 7, 11, 13]" 。要正确地打印多维数组，使用`Arrays.deepToString()`方法。

`toString()`方法是一种非常有用的调试工具。标准库中的许多类都定义了`toString()`方法，以便获得关于对象状态的有用信息。这在日志消息中尤其有用：

```java
System.out.println("Current position = " + position);
```

提示：强烈建议为自定义的每个类添加`toString()`方法。这样自己和使用这个类的其他程序员都会从日志支持中受益匪浅。

注释：记录自动提供了`toString()`方法，它会列出类名和所有字段值的字符串表示。

程序清单5-4的程序测试了`Employee`类（程序清单5-5）和`Manager`类（程序清单5-6）的`equals()`、`hashCode()`和`toString()`方法。

[程序清单5-4 equals/EqualsTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch05/equals/EqualsTest.java)

[程序清单5-5 equals/Employee.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch05/equals/Employee.java)

[程序清单5-6 equals/Manager.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch05/equals/Manager.java)

## 5.3 泛型数组列表
在一些编程语言（特别是C和C++）中，必须在编译时就确定所有数组的大小。在Java中，情况有所改善。可以在运行时设置数组的大小，但这并没有完全解决运行时动态修改数组的问题。一旦设置了数组的大小，就无法轻易地改变了。在Java中，可以使用`ArrayList`类来处理这种常见情况。`ArrayList`类与数组类似，但能够在添加或删除元素时自动地调整容量。

`ArrayList`是一个有**类型参数**(type parameter)的**泛型类**(generic class)。要指定数组列表的元素类型，需要用一对尖括号将类名括起来加在后面，例如`ArrayList<Employee>`。第8章将介绍如何自定义泛型类，不过使用`ArrayList`不需要了解任何技术细节。

下面几节将介绍如何使用数组列表。

### 5.3.1 声明数组列表
如下声明和构造一个保存`Employee`对象的数组列表：

```java
ArrayList<Employee> staff = new ArrayList<Employee>();
```

从Java 10起，可以使用`var`关键字避免重复写类名：

```java
var staff = new ArrayList<Employee>();
```

如果不使用`var`关键字，可以省略右边的类型参数：

```java
ArrayList<Employee> staff = new ArrayList<>();
```

这称为“菱形”语法，因为空尖括号`<>`就像是一个菱形。

警告：如果使用`var`声明`ArrayList`，就不要使用菱形语法。声明`var elements = new ArrayList<>();`会生成一个`ArrayList<Object>`。

注释：Java 5以前没有泛型类，而是有一个保存`Object`类型元素的`ArrayList`类。现在仍然可以使用没有`<...>`后缀的`ArrayList`，它被认为是一个擦除了类型参数的“原始”类型（详见8.5节）。

使用`add()`方法将新元素添加到数组列表中。例如：

```java
staff.add(new Employee("Harry Hacker", ...));
staff.add(new Employee("Tony Tester", ...));
```

数组列表管理着一个内部的对象引用数组，如下图所示。

![数组列表](/assets/images/java-note-v1ch05-inheritance/数组列表.png)

如果调用`add()`而内部数组已经满了，数组列表就会自动地创建一个更大的数组，并将所有对象引用从原数组拷贝到新数组中，如下图所示（注：标准库实际实现并不是扩容为2倍，而是1.5倍）。

![数组列表的扩容](/assets/images/java-note-v1ch05-inheritance/数组列表的扩容.png)

如果已经知道或者能够估计出要存储的元素数量，就可以在填充数组列表之前调用`ensureCapacity()`方法：

```java
staff.ensureCapacity(100);
```

这个调用将分配一个大小为100的内部数组。这样，前100次调用`add()`将不会发生开销很大的重新分配（扩容）。

也可以把初始容量传递给构造器：

```java
ArrayList<Employee> staff = new ArrayList<>(100);
```

警告：如下创建一个数组列表：

```java
new ArrayList<>(100) // size is 0, capacity is 100
```

**不同于**如下分配一个新数组：

```java
new Employee[100] // size is 100
```

数组列表的容量(capacity)与数组的大小(size)有一个非常重要的区别。大小为100的数组的所有元素都是可访问的；而容量为100的数组列表只是可能保存100个元素，但是只有前size个元素是可访问的。

`size()`方法返回数组列表中的元素个数，它等价于数组的`a.length`。

一旦能够确认数组列表的大小不再发生变化，可以调用`trimToSize()`方法将容量调整为当前元素数量。垃圾收集器将回收多余的内存空间。

C++注释：`ArrayList`类似于C++的`vector`模板，二者都是泛型类型。但是C++的`vector`重载了`[]`运算符以便于元素访问；而Java没有运算符重载，因此必须使用显式的方法调用。此外，C++向量是值拷贝（深拷贝），而Java的赋值会让两个变量引用同一个数组列表（浅拷贝）。

### 5.3.2 访问数组列表元素
不能使用`[]`语法访问数组列表的元素，而要使用`get()`和`set()`方法。

使用`staff.set(i, harry)`设置第`i`个元素，这等价于数组的`a[i] = harry`。使用`staff.get(i)`获得第`i`个元素，这等价于数组的`a[i]`。与数组一样，索引值从0开始。

警告：访问数组列表的元素时，索引必须满足`0 <= i < size`。例如，下面的代码是错误的：

```java
var list = new ArrayList<Employee>(100); // capacity 100, size 0
list.set(0, x); // no element 0 yet
```

可以使用`toArray()`方法将数组列表转换为数组。

有时需要在数组列表的中间插入元素，为此使用带索引参数的`add()`方法：`staff.add(i, e)`。位置`i`及之后的元素都要向后移动一个位置。

类似地，可以从数组列表的中间删除元素：`staff.remove(i)`。这个位置之后的元素都向前移动一个位置。

插入和删除元素的效率很低。对于较小的数组列表来说，不必担心这个问题。但如果存储的元素很多，又经常需要在中间插入和删除元素，就应该考虑使用链表。链表将在9.3.1节介绍。

可以使用for each循环遍历数组列表的内容：

```java
for (Employee e : staff)
    // do something with e
```

这个循环等价于

```java
for (int i = 0; i < staff.size(); i++) {
    Employee e = staff.get(i);
    // do something with e
}
```

程序清单5-7对程序清单4-2中的`EmployeeTest`做了修改，将`Employee[]`替换为`ArrayList<Employee>`。

[程序清单5-7 arrayList/ArrayListTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch05/arrayList/ArrayListTest.java)

### 5.3.3 类型化与原始数组列表的兼容性
见8.5.4节。

## 5.4 对象包装器与自动装箱
有时，需要将`int`这样的基本类型转换为对象。所有的基本类型都有一个与之对应的类。例如，`Integer`类对应基本类型`int`。这些类通常称为**包装器**(wrapper)，其名字显而易见：`Integer`、`Long`、`Float`、`Double`、`Short`、`Byte`、`Character`和`Boolean`（前6个继承自公共超类`Number`）。包装器类是**不可变的**。同时，包装器类还是`final`。

遗憾的是，尖括号中的类型参数不能是基本类型，因此不能使用`ArrayList<int>`。这里就需要用到包装器类。可以声明`Integer`对象的数组列表：

```java
var list = new ArrayList<Integer>();
```

警告：由于每个值分别包装在一个对象中，`ArrayList<Integer>`的效率远低于`int[]`。只有当程序员的方便性比效率更重要时，才会考虑对较小的集合使用这种构造。

幸运的是，有一个很有用的特性可以很容易地将`int`类型的元素添加到`ArrayList<Integer>`。调用`list.add(3);`将自动地翻译成`list.add(Integer.valueOf(3));`。这种转换称为**自动装箱**(autoboxing)。

相反，将一个`Integer`对象赋给一个`int`值时，将会自动拆箱(unbox)。也就是说，编译器将`int n = list.get(i);`翻译成`int n = list.get(i).intValue();`。

自动装箱和拆箱也适用于算术表达式。例如，可以将自增运算符用于包装器引用：

```java
Integer n = 3;
n++;
```

编译器将自动地插入指令对对象拆箱，将结果值加1，最后再装箱。即：

```java
int x = n.intValue();
x++;
n = Integer.valueOf(x);
```

注：从上面的代码可以看出，自增前后`n`引用的是两个不同的`Integer`对象（如下图所示）。

![自增Integer对象](/assets/images/java-note-v1ch05-inheritance/自增Integer对象.png)

大多数情况下，会有一种错觉，认为基本类型和包装器是一样的。但它们有一点显著不同之处：同一性。`==`运算符检测两个对象变量是否引用同一个对象。因此下面的比较可能会失败：

```java
Integer a = 1000;
Integer b = 1000;
if (a == b) ...
```

然而，Java实现**可能**将经常出现的值包装到相同的对象中，这样上面的比较就会成功。这种不确定性并不是我们想要的。解决方法是使用`equals()`方法比较包装器对象。

注释：自动装箱规范要求布尔值、`\u0000`~`\u007F`之间的字符和-128~127之间的整数包装到固定的对象中（小整数缓存）。例如，在前面的例子中，如果将`a`和`b`初始化为`100`，那么`a == b`一定成立。

关于自动装箱还有几点需要说明。首先，由于包装器类引用可以为`null`，所以自动拆箱有可能会抛出`NullPointerException`：

```java
Integer n = null;
System.out.println(2 * n); // throws NullPointerException
```

另外，如果在一个条件表达式中混合使用`Integer`和`Double`类型，则`Integer`值会拆箱，提升为`double`，再装箱为`Double`：

```java
Integer n = 1;
Double x = 2.0;
System.out.println(true ? n : x); // prints 1.0
```

最后强调一下，自动装箱和拆箱是编译器的语法糖，而不是虚拟机的。编译器生成类的字节码时会插入必要的方法调用。虚拟机只是执行这些字节码。

注释：Java未来的版本可能允许类似基本类型的用户自定义类型。例如，基本类型`Point`的值（具有`double`字段`x`和`y`）只是一个16字节的内存块，有两个相邻的`double`值。可以拷贝，但不能有它的引用。如果需要引用，使用自动生成的伴随类。装箱和拆箱是自动的。参见[JEP 401](https://openjdk.org/jeps/401)、[JEP 402](https://openjdk.org/jeps/402)、[Project Valhalla](https://openjdk.org/projects/valhalla/)。

使用数值包装器还有一个原因。Java设计者发现将某些基本方法放在包装器中会很方便。例如，将字符串转换成整数：

```java
int x = Integer.parseInt(s);
```

`parseInt()`是一个静态方法，但`Integer`类是放置这个方法的一个好地方。

警告：包装器对象是不可变的，因此不能使用包装器类来创建修改数值参数的方法。

## 5.5 参数个数可变的方法
可以定义参数个数可变的方法（有时称为“变参”(varargs)方法）。

前面已经见过这样的方法：`printf()`。例如，`System.out.printf("%d", n);`和`System.out.printf("%d %s", n, "widgets");`这两条语句都调用同一个方法。

`printf()`方法是这样定义的：

```java
public class PrintStream {
    public PrintStream printf(String fmt, Object... args) {
        return format(fmt, args);
    }
}
```

这里的省略号`...`是Java代码的一部分，表示这个方法可以接收任意数量的对象（除`fmt`参数外）。

实际上，`printf()`方法接收两个参数：一个是格式字符串，另一个是保存所有其他参数的`Object`数组（如果调用者提供的是基本类型的值，则会将其自动装箱为对象）。对于`printf()`的实现者来说，参数类型`Object...`与`Object[]`完全一样。

编译器需要转换每个`printf()`调用，将参数打包到一个数组中，并根据需要自动装箱：

```java
System.out.printf("%d %s", new Object[] { Integer.valueOf(n), "widgets" } );
```

自己也可以定义有可变参数的方法，可以为参数指定任意类型，甚至是基本类型。下面是一个简单的示例：这个函数计算若干个数值中的最大值。

```java
public static double max(double... values) {
    double largest = Double.NEGATIVE_INFINITY;
    for (double v : values) if (v > largest) largest = v;
    return largest;
}
```

可以像这样调用这个函数：

```java
double m = max(3.1, 40.4, -5);
```

编译器将`new double[] { 3.1, 40.4, -5 }`传递给`max()`函数。

注释：允许将数组作为最后一个参数传递给有可变参数的方法。因此，如果一个已有方法的最后一个参数是数组，则可以把它重新定义为有可变参数的方法，而不会破坏任何已有的代码。例如，可以将`main()`方法声明为`public static void main(String... args)`。

## 5.6 抽象类
如果从下往上看继承层次结构，位于上层的类更加通用，也可能更加抽象。有时，祖先类足够通用，以至于只将它视为其他类的基类，而不是用来构造特定实例。例如，员工是一个人，学生也是一个人。下面将`Person`和`Student`类加入类层次结构。下图显示了这些类之间的继承关系。

![Person及其子类的继承图](/assets/images/java-note-v1ch05-inheritance/Person及其子类的继承图.png)

何苦提供这样高层次的抽象呢？每个人都有一些属性，如姓名。学生和员工都有姓名，通过引入一个公共超类，就可以把`getName()`方法放在继承层次结构中更高的一层。

现在，再增加一个`getDescription()`方法，返回对一个人的简短描述。例如： "an employee with a salary of $50,000.00" 、 "a student majoring in computer science" 。

`Employee`和`Student`类实现这个方法很容易，但是`Person`类除了姓名之外对这个人一无所知。可以使用`abstract`关键字将方法声明为**抽象的**(abstract)，这样就不需要实现这个方法了。

```java
public abstract String getDescription(); // no implementation required
```

包含一个或多个抽象方法的类本身必须被声明为抽象的。

```java
public abstract class Person {
    ...
}
```

除了抽象方法之外，抽象类还可以有字段和具体(concrete)方法。例如，`Person`类保存着姓名和返回姓名的具体方法：

```java
public abstract class Person {
    private String name;
    public Person(String name) { this.name = name; }
    public abstract String getDescription();
    public String getName() { return name; }
}
```

提示：应该将公共的字段和方法（不管是否是抽象的）放在超类（不管是否是抽象的）中。

抽象方法充当在子类中实现的方法的占位符。扩展抽象类时，可以不实现或实现部分抽象方法，这样必须将子类也标记为抽象的；也可以实现全部抽象方法，这样子类就不再是抽象的了。

即使不含抽象方法，也可以将类声明为抽象的。

**抽象类不能实例化。** 例如，表达式`new Person("Vince Vu")`是错误的。不过，可以创建具体子类的对象。

注意，仍然可以创建抽象类的对象变量，但这种变量只能引用具体子类的对象。例如：

```java
Person p = new Student("Vince Vu", "Economics");
```

C++注释：在C++中，抽象方法称为纯虚函数，在末尾用`= 0`标记。如果至少有一个纯虚函数，这个类就是抽象类。在C++中，没有用于表示抽象类的特殊关键字。

程序清单5-8中的程序定义了抽象超类`Person`（程序清单5-9）和两个具体子类`Employee`（程序清单5-10）及`Student`（程序清单5-11）。

[程序清单5-8 abstractClasses/PersonTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch05/abstractClasses/PersonTest.java)

[程序清单5-9 abstractClasses/Person.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch05/abstractClasses/Person.java)

[程序清单5-10 abstractClasses/Employee.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch05/abstractClasses/Employee.java)

[程序清单5-11 abstractClasses/Student.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch05/abstractClasses/Student.java)

`p.getDescription()`不是调用了一个没有定义的方法吗？要记住，由于不可能构造抽象类`Person`的对象，所以变量`p`永远不会引用`Person`对象，而总是引用一个具体子类（如`Employee`或`Student`）的对象。这些对象都定义了`getDescription()`方法。

如果省略超类中的抽象方法，而仅在子类中定义`getDescription()`方法，这样就不能通过变量`p`调用`getDescription()`方法了。

在Java语言中，抽象方法是一个重要的概念。在接口中将会看到更多的抽象方法。有关接口的更多信息参见第6章。

## 5.7 枚举类
在第3章已经看到如何定义枚举类型。下面是一个典型的例子：

```java
public enum Size { SMALL, MEDIUM, LARGE, EXTRA_LARGE }
```

实际上，这个声明定义的类型是一个类。这个类刚好有4个实例，不能构造新的对象。因此，在比较枚举类型的值时，不需要使用`equals()`，可以直接使用`==`比较。

如果需要，可以为枚举类型添加构造器、方法和字段。当然，构造器只有在构造枚举常量的时候被调用。下面是一个示例：

```java
public enum Size {
    SMALL("S"), MEDIUM("M"), LARGE("L"), EXTRA_LARGE("XL");

    private String abbreviation;

    Size(String abbreviation) { this.abbreviation = abbreviation; } // automatically private
    public String getAbbreviation() { return abbreviation; }
}
```

注：枚举常量是枚举类型的静态常量。例如，`SMALL`的定义等价于

```java
public static final Size SMALL = new Size("S");
```

枚举的构造器总是私有的，因此可以省略`private`修饰符。如果将枚举构造器声明为`public`或`protected`会产生语法错误。

所有枚举类型都是抽象类`Enum`的子类，它们继承了这个类的许多方法。其中最有用的一个是`toString()`，这个方法返回枚举常量名。例如，`Size.SMALL.toString()`返回字符串`"SMALL"`。

`toString()`的逆方法是静态方法`valueOf()`。例如，语句

```java
Size s = Enum.valueOf(Size.class, "SMALL");
```

将`s`设置为`Size.SMALL`（注：也可以使用`Size.valueOf("SMALL")`）。如果没有对应的常量则抛出异常。

每个枚举类型都有一个静态的`values()`方法，返回一个包含全部枚举值的数组。例如，

```java
Size[] values = Size.values();
```

返回包含元素`{ Size.SMALL, Size.MEDIUM, Size.LARGE, Size.EXTRA_LARGE }`的数组。

`ordinal()`方法返回枚举常量在枚举声明中的位置，从0开始计数。例如，`Size.MEDIUM.ordinal()`返回1。

注释：`Enum`类有一个类型参数。例如，枚举类型`Size`实际上扩展了`Enum<Size>`。类型参数在`compareTo()`方法中使用（`compareTo()`方法将在第6章中介绍，类型参数将在第8章中介绍）。

程序清单5-12演示了如何使用枚举类型。

[程序清单5-12 enums/EnumTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch05/enums/EnumTest.java)

## 5.8 密封类
假设你需要编写自己的JSON库。[JSON标准](https://www.json.org/json-en.html)规定：JSON值是一个数组、数值、字符串、布尔值、对象或null。一种显而易见的方法是使用`JSONArray`、`JSONNumber`等类来表示，这些类都扩展了抽象类`JSONValue`：

```java
public abstract class JSONValue {
    // Methods that apply to all JSON values
}
public final class JSONArray extends JSONValue {
    ...
}
public final class JSONNumber extends JSONValue {
    ...
}
```

通过将`JSONArray`、`JSONNumber`等类声明为`final`，可以确保没有人能定义它们的子类。但无法阻止别人定义`JSONValue`的另一个子类。

为什么要控制这一点呢？考虑以下代码：

```java
JSONValue v = ...;
if (v instanceof JSONArray a) ...
else if (v instanceof JSONNumber n) ...
else if (v instanceof JSONString s) ...
else if (v instanceof JSOBoolean b) ...
else if (v instanceof JSONObject o) ...
else ... // Must be JSONNull
```

这里的控制流表明我们知道`JSONValue`的所有直接子类。这不是一个开放性的类层次结构，因为JSON标准不会改变。我们不希望别人扰乱这个类层次结构。

在Java中，**密封类**(sealed class)控制哪些类可以继承它。密封类在Java 15中作为预览特性加入，并在Java 17中最终确定。

可以使用关键字`sealed`将一个类声明为密封类，并使用`permits`子句指定允许的子类。

```java
public abstract sealed class JSONValue
    permits JSONArray, JSONNumber, JSONString, JSONBoolean, JSONObject, JSONNull {
    ...
}
```

定义一个未经允许的子类是错误的：

```java
public class JSONComment extends JSONValue { ... } // Error
```

这是有道理的，因为JSON不支持注释。所以，密封类可以准确地描述领域约束。

一个密封类允许的子类必须是可访问的，不能是嵌套在另一个类中的私有类，也不能是位于另一个包中的包可见类。对于允许的公有子类，规则要更为严格。它们必须与密封类在同一个包中。不过，如果使用模块（参见卷II第9章），则必须在同一个模块中。

注释：声明密封类可以不加`permits`子句。这样，它的所有直接子类必须声明在同一个文件中，不能访问这个文件的程序员就不能定义它的子类。一个文件最多只能有一个公有类，所以这种组织似乎只有当子类不是供公共使用时才有用。不过，下一章会看到，可以使用内联类作为公有子类。

使用密封类的一个重要原因是编译时检查。考虑`JSONValue`类的`type()`方法，其中使用了带模式匹配的`switch`表达式（Java 17中的一个预览特性）：

```java
public String type() {
    return switch (this) {
        case JSONArray j -> "array";
        case JSONNumber j -> "number";
        case JSONString j -> "string";
        case JSONBoolean j -> "boolean";
        case JSONObject j -> "object";
        case JSONNull j -> "null";
        // No default needed here
    };
}
```

编译器可以检查出这里不需要`default`子句，因为`JSONValue`的所有直接子类都已经出现在`case`分支中。

注释：前面的`type()`方法看起来不太面向对象。按照OOP的精神，这6个子类应当提供自己的`type()`方法，并依赖多态而不是`switch`。对于开放性的类层次结构，这是一种好方法。然而，对于一组固定的类，在一个方法中处理所有候选通常更方便。

乍一看，密封类的子类似乎必须是`final`。但对于穷尽测试（如上面的`type()`方法），我们只需要知道所有直接子类。那些类有自己的子类并没有问题。例如，可以重新组织JSON类层次结构，如下图所示。

![表示JSON值的完整类层次结构](/assets/images/java-note-v1ch05-inheritance/表示JSON值的完整类层次结构.png)

在这个层次结构中，`JSONValue`允许有3个子类，`JSONPrimitive`类也是密封的：

```java
public abstract sealed class JSONValue
    permits JSONObject, JSONArray, JSONPrimitive {
    ...
}

public abstract sealed class JSONPrimitive extends JSONValue
    permits JSONString, JSONNumber, JSONBoolean, JSONNull {
    ...
}
```

密封类的子类必须指定它是`sealed`、`final`还是允许继承。对于最后一种情况，必须声明为`non-sealed`。

注释：关键字`non-sealed`是第一个带连字符的Java关键字。在语言中增加关键字总是会带来风险，现有代码可能无法编译。由于这个原因，`sealed`是一个“上下文”关键字。仍然可以声明名为`sealed`的变量或方法：

```java
int sealed = 1; // OK to use contextual keyword as identifier
```

利用带连字符的关键字，可以不用担心这个问题。唯一的歧义性是减法：

```java
int non = 0;
non = non-sealed; // Subtraction, not keyword
```

为什么需要`non-sealed`子类呢？考虑一个XML节点类，有6个直接子类：

```java
public abstract sealed class Node
    permits Element, Text, Comment, CDATASection, EntityReference, ProcessingInstruction {
    ...
}
```

允许定义`Element`的任意子类：

```java
public non-sealed class Element extends Node {
    ...
}
public class HTMLDivElement extends Element {
    ...
}
```

本节介绍了密封类。下一章将介绍接口，接口是抽象类的泛化。接口也可以有子类型。密封接口的工作方式与密封类完全相同，即控制直接子类型。

程序清单5-13实现了JSON类层次结构。在这个例子中使用了接口而不是抽象类，从而`JSONNumber`和`JSONString`可以是记录，`JSONBoolean`和`JSONNull`可以是枚举。记录和枚举可以实现接口，但不能扩展类。

[程序清单5-13 sealed/SealedTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch05/sealed/SealedTest.java)

## 5.9 反射
能够分析类的能力的程序称为**反射**(reflection)。反射机制极其强大，可以用来：
* 在运行时分析类的能力
* 在运行时检查对象
* 实现泛型数组操作代码
* 利用`Method`对象（类似于C++中的函数指针）

### 5.9.1 Class类
在程序运行期间，Java运行时系统始终为所有对象维护一个**运行时类型标识**(runtime type identification)。这个信息跟踪每个对象所属的类，虚拟机利用这个信息选择动态绑定要调用的正确方法。

不过，还可以使用一个特殊的Java类访问这些信息——`Class`类（类似于Python的`type`类）。`Object`类的`getClass()`方法返回一个`Class`类型的实例。

就像`Employee`对象描述一个特定员工的属性，`Class`对象描述一个特定类的属性。最常用的`Class`类方法是`getName()`，返回类的名字。例如，如果`e`是一个`Employee`对象，则`e.getClass().getName()`返回`"Employee"`。

如果类在一个包中，包名也作为类名的一部分：

```java
var generator = new Random();
Class cl = generator.getClass();
String name = cl.getName(); // name is set to "java.util.Random"
```

可以使用静态方法`forName()`获得类名对应的`Class`对象。

```java
String className = "java.util.Random";
Class cl = Class.forName(className);
```

如果类名保存在一个会在运行时变化的字符串中，就可以使用这个方法。`className`必须是一个类名或接口名，否则将抛出`ClassNotFoundException`。

获得`Class`对象的第三种方法是一种方便的简写：如果`T`是任意的Java类型（或`void`），则`T.class`是对应的类对象。例如：

```java
Class cl1 = Random.class; // if you import java.util.*;
Class cl2 = int.class;
Class cl3 = Double[].class;
```

注意，`Class`对象实际上描述的是一个**类型**，可能是类，也可能是基本类型。

警告：由于历史原因，`getName()`方法对于数组类型会返回有些奇怪的名字：`int[].class.getName()`返回`"[I"`，`Double[].class.getName()`返回`"[Ljava.lang.Double;"`。

虚拟机为每个类型管理一个唯一的`Class`对象。因此，可以使用`==`运算符比较两个类对象。例如：

```java
if (e.getClass() == Employee.class) ...
```

如果`e`是一个`Employee`实例，则这个测试通过。与`e instanceof Employee`不同，如果`e`是一个子类（如`Manager`）的实例，则这个测试失败。

如果有一个`Class`对象，可以用它构造类的实例。调用`getConstructor()`方法得到一个`Constructor`类型的对象（表示无参构造器），然后使用`newInstance()`方法构造实例。例如：

```java
var className = "java.util.Random"; // or any other name of a class with a no-arg constructor
Class cl = Class.forName(className);
Object obj = cl.getConstructor().newInstance();
```

如果这个类没有无参数的构造器，`getConstructor()`方法会抛出一个异常。5.9.7节将介绍如何调用其他构造器。

C++注释：`Class`类类似于C++中的`type_info`类，`getClass()`方法则等价于`typeid`运算符。

### 5.9.2 声明异常入门
第7章将全面地介绍异常处理，但现在时常遇到一些可能抛出异常的方法。

当程序在运行时发生错误时，就会“抛出异常”。抛出异常比终止程序要灵活得多，因为你可以提供一个**处理器**(handler)“捕获”异常并进行处理。如果没有提供处理器，程序就会终止，并在控制台上打印错误消息。

异常有两种类型：**检查型**(checked)异常和**非检查型**(unchecked)异常。对于检查型异常，编译器会检查你是否处理了异常（使用`try/catch`捕获或使用`throws`声明抛出）。然而，很多常见的异常（例如下标越界或者空指针异常）都属于非检查型。编译器并不强制你处理这些异常——毕竟这些错误是可避免的。但不是所有的错误都是可以避免的。`Class.forName()`方法就是一个例子——没有办法确保有指定名字的类一定存在。

第7章将介绍几种异常处理策略。现在只介绍最简单的策略：如果一个方法包含可能抛出检查型异常的语句，就在方法名后添加`throws`子句。

```java
public static void doSomethingWithClass(String name)
    throws ReflectiveOperationException {
    Class cl = Class.forName(name); // might throw exception
    // do something with cl
}
```

调用这个方法的任何方法也都需要`throws`声明，包括`main()`方法。如果确实发生了异常，`main()`方法将终止并打印栈轨迹(stack trace)。

只需要为检查型异常提供`throws`子句。很容易找出哪些方法会抛出检查型异常——只要调用了可能抛出检查型异常的方法而没有提供处理器，编译器就会报错。

### 5.9.3 资源
类通常有一些关联的数据文件，例如图像、声音和文本文件。在Java中，这些关联的文件称为**资源**(resource)。

例如，考虑一个显示消息的对话框，如下图所示。

![显示图像和文本资源](/assets/images/java-note-v1ch05-inheritance/显示图像和文本资源.png)

为了便于修改，将标题和版权文本放在一个文件中，而不是硬编码为字符串。将这些文本文件与其他程序文件一起放在JAR文件中会很方便。

`Class`类提供了一个很有用的服务可以查找资源文件。下面是必要的步骤：
1. 获得拥有资源的类的`Class`对象，例如`ResourceTest.class`。
2. 有些方法（如`ImageIcon`类的构造器）接受描述资源位置的URL，那么可以调用`getResource()`。
3. 否则，使用`getResourceAsStream()`方法获得一个输入流来读取文件中的数据。

重点在于Java虚拟机知道如何查找一个类，所以能搜索**相同位置**上的关联资源。例如，假设`ResourceTest`类在`resources`包中，则ResourceTest.class文件就位于resources目录中，可以把图标文件放在同一个目录下。另外，还可以提供相对或绝对路径，例如data/about.txt或/corejava/title.txt。

另一个经常使用资源的地方是程序的国际化。与语言相关的字符串（如消息和用户界面标签）存放在资源文件中，每种语言对应一个文件。国际化API支持一种标准方法来组织和访问这些本地化文件，将在卷II第7章介绍。

程序清单5-14的程序展示了资源加载。

[程序清单5-14 resources/ResourceTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch05/resources/ResourceTest.java)

编译、构建一个JAR文件并执行：

```shell
javac resources/ResourceTest.java
jar cvfe ResourceTest.jar resources.ResourceTest resources/*.class resources/*.gif resources/data/*.txt corejava/*.txt
java -jar ResourceTest.jar
```

将JAR文件移动到一个不同的目录再次运行，以确认程序是从JAR文件而不是当前目录读取资源。

注：
* 在这个示例中，源代码、类文件和资源文件的目录结构如下：

```
project/
    resources/
        ResourceTest.java
        ResourceTest.class
        about.gif
        data/
            about.txt
    corejava/
        title.txt
```

构建的JAR文件目录结构如下：

```
ResourceTest.jar/
    META-INF/
        MANIFEST.MF
    resources/
        ResourceTest.class
        about.gif
        data/
            about.txt
    corejava/
        title.txt
```

* `Class`类会在**类路径**中查找资源文件（就像查找类文件一样）：
  * 如果文件名以 "/" 开头，则在类路径下查找。
  * 否则，在当前类文件所在目录（即 "类路径/包路径" ）下查找。

在JAR文件中，类路径为JAR文件根目录。

在上面的示例中，假设将项目根目录/path/to/project或JAR文件根目录/path/to/ResourceTest.jar添加到类路径，则`ResourceTest`类加载各个资源文件使用的文件名和最终定位的URL如下：

| 文件名 | 文件系统URL | JAR文件内URL |
| --- | --- | --- |
| `"about.gif"` | file:/path/to/project/resources/about.gif | jar:file:/path/to/ResourceTest.jar!/resources/about.gif |
| `"data/about.txt"` | file:/path/to/project/resources/data/about.txt | jar:file:/path/to/ResourceTest.jar!/resources/data/about.txt |
| `"/corejava/title.txt"` | file:/path/to/project/corejava/title.txt | jar:file:/path/to/ResourceTest.jar!/corejava/title.txt |

### 5.9.4 使用反射分析类的能力
下面简要介绍反射机制最重要的内容——检查类的结构。

`java.lang.reflect`包中有三个类`Field`、`Method`和`Constructor`，分别用于描述类的字段、方法和构造器。这三个类都有返回名字的`getName()`方法。`Field`类的`getType()`方法返回字段类型。`Method`和`Constructor`类有报告参数类型的方法，`Method`类还有报告返回类型的方法。这三个类都有一个`getModifiers()`方法，返回一个整数，用位开关描述所使用的修饰符。可以使用`Modifier`类的静态方法分析这个整数。还可以使用`Modifier.toString()`方法打印修饰符。

注：`Field`类似于C++的数据成员指针，`Method`类似于C++的成员函数指针，但C++的成员指针并不是真正的反射，因为无法通过字符串名字获得对应的成员。C++目前并不支持反射，但有一个[实验性的反射扩展](https://en.cppreference.com/w/cpp/experimental/reflect)。

`Class`类的`getFields()`、`getMethods()`和`getConstructors()`方法分别返回这个类的**公有**字段、方法和构造器的数组，其中包括超类的公有成员。`Class`类的`getDeclaredFields()`、`getDeclaredMethods()`和`getDeclaredConstructors()`方法分别返回这个类中声明的全部字段、方法和构造器的数组，其中包括私有、受保护和包访问成员，但不包括超类的成员。

程序清单5-15展示了如何打印一个类的全部信息。这个程序提示用户输入一个类名，然后输出类中所有方法和构造器的签名以及所有实例字段名。

[程序清单5-15 reflection/ReflectionTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch05/reflection/ReflectionTest.java)

例如，如果输入java.lang.Double，程序将会输出：[ReflectionTest_output.txt](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch05/testdata/ReflectionTest_output.txt)

令人赞叹的是，这个程序可以分析Java解释器能加载的任何类（例如`inheritance.Manager`），而不仅仅是编译程序时可用的类。在下一章中，还将使用这个程序查看Java编译器自动生成的内部类。

### 5.9.5 使用反射在运行时分析对象
在前一节中，已经知道如何查看任意对象实例字段的名字和类型。本节将进一步查看字段的内容。当然，如果知道字段名和类型，查看对象指定字段的内容很容易。而利用反射可以查看在编译时还不知道的对象字段（例如字段名是用户输入的字符串）。

要做到这一点，关键方法是`Field`类中的`get()`方法。如果`f`是一个`Field`类型的对象（对应类`C`中名为`x`的字段），`obj`是一个类`C`的对象，则`f.get(obj)`返回字段`obj.x`的当前值。下面来看一个例子：

```java
var harry = new Employee("Harry Hacker", 50000, 10, 1, 1989);
Class cl = harry.getClass(); // the class object representing Employee
Field f = cl.getDeclaredField("name"); // the name field of the Employee class
Object v = f.get(harry); // the value of the name field of the harry object, i.e., the String object "Harry Hacker"
```

其中`f.get(harry)`等价于`harry.name`。对于基本类型的字段，可以使用`getInt()`、`getDouble()`等方法，也可以使用`get()`（此时会自动装箱）。

也可以设置字段的值。调用`f.set(obj, value)`将`obj`中`f`表示的字段设置为新值（等价于`obj.x = value`）。

实际上，这段代码存在一个问题。由于`name`是一个私有字段，所以`get()`和`set()`方法会抛出`IllegalAccessException`。Java安全机制允许查看一个对象有哪些字段，但是除非有访问权限，否则不允许读写那些字段的值。

反射机制的默认行为是遵守Java的访问控制。不过，可以调用`Field`、`Method`或`Constructor`对象的`setAccessible()`方法覆盖访问控制。例如：

```java
f.setAccessible(true); // now OK to call f.get(harry)
```

`setAccessible()`是`AccessibleObject`类的方法，它是`Field`、`Method`和`Constructor`类的公共超类。这个特性是为调试、持久存储和类似的机制提供的。本节稍后将用它编写一个通用的`toString()`方法。如果不允许访问，`setAccessible()`调用会抛出一个异常。访问可能被模块系统（卷II第9章）或安全管理器（卷II第10章）拒绝。

下面来看一个可用于任意类的通用`toString()`方法（见代码清单5-16）。这个通用的`toString()`方法需要解决几个复杂的问题。循环引用可能导致无限递归，因此`ObjectAnalyzer`（代码清单5-17）会跟踪已访问过的对象。另外，要查看数组内部，需要采用一种不同的方法，细节将在下一节介绍。

[程序清单5-16 objectAnalyzer/ObjectAnalyzerTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch05/objectAnalyzer/ObjectAnalyzerTest.java)

[程序清单5-17 objectAnalyzer/ObjectAnalyzer.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch05/objectAnalyzer/ObjectAnalyzer.java)

在Java 9到Java 16中运行这个程序时，会出现警告消息 "An illegal reflective access operation has occurred" 。在Java 17中运行这个程序时，会出现`InaccessibleObjectException`异常。要让程序能够运行，需要把`java.base`模块中的`java.util`和`java.lang`包“打开”到“无名模块”（详见卷II第9章）：

```shell
java --add-opens java.base/java.util=ALL-UNNAMED \
    --add-opens java.base/java.lang=ALL-UNNAMED \
    objectAnalyzer.ObjectAnalyzerTest
```

例如，对包含[1, 4, 9, 16, 25]的`ArrayList<Integer>`对象调用这个方法，将生成以下结果：

```
java.util.ArrayList[elementData=class java.lang.Object[]{java.lang.Integer[value=1][][],
java.lang.Integer[value=4][][],java.lang.Integer[value=9][][],java.lang.Integer[value=16][][],
java.lang.Integer[value=25][][],null,null,null,null,null},size=5][modCount=5][][]
```

### 5.9.6 使用反射编写泛型数组代码
`java.lang.reflect`包中的`Array`类允许动态地创建数组。例如，`Arrays.copyOf()`方法（3.10.4节）的实现中就使用了这个类。这个方法可以用于扩展一个已经满的数组：

```java
var a = new Employee[100];
...
// array is full
a = Arrays.copyOf(a, 2 * a.length);
```

如何编写这样一个通用的方法呢？`Employee[]`能够转换为`Object[]`，下面是第一次尝试：

```java
public static Object[] badCopyOf(Object[] a, int newLength) { // not useful
    var newArray = new Object[newLength];
    System.arraycopy(a, 0, newArray, 0, Math.min(a.length, newLength));
    return newArray;
}
```

然而，在使用得到的数组时会遇到一个问题：这段代码返回的数组类型是`Object[]`，不能强制转换成`Employee[]`。关键是，Java数组会记住元素的类型（即`new`表达式中使用的类型）。将一个`Employee[]`临时转换成`Object[]`然后再转换回来是合法的，但一开始就是`Object[]`的数组永远不能转换成`Employee[]`（否则会引发`ClassCastException`）。

为了编写这类泛型数组代码，需要能够创建与原数组**类型相同**的新数组。为此，需要使用`Array.newInstance()`方法，提供元素类型和数组长度。

可以通过调用`Array.getLength(a)`获得数组的长度。要获得数组`a`的元素类型：
1. 获得`a`的类对象。
2. 使用`isArray()`确认它确实是一个数组。
3. 使用`getComponentType()`得到数组的元素类型（只为表示数组的类对象定义了这个方法）。
4. 反过来，对于表示类`C`的类对象，`arrayType()`方法生成表示`C[]`的类对象。

下面是正确的代码：

```java
public static Object goodCopyOf(Object a, int newLength) {
    Class cl = a.getClass();
    if (!cl.isArray()) return null;
    Class componentType = cl.getComponentType();
    int length = Array.getLength(a);
    Object newArray = Array.newInstance(componentType, newLength);
    System.arraycopy(a, 0, newArray, 0, Math.min(length, newLength));
    return newArray;
}
```

注：关于泛型数组的问题另见8.6.6节。

注意，这个方法可以用来扩展任意类型的数组，而不仅是对象数组。

```java
int[] a = { 1, 2, 3, 4, 5 };
a = (int[]) goodCopyOf(a, 10);
```

`goodCopyOf()`的第一个参数类型声明为`Object`而不是`Object[]`——整型数组`int[]`可以转换为`Object`，但不能转换为`Object[]`。

程序清单5-18使用了这两个方法。注意，对`badCopyOf()`的返回值进行强制类型转换会抛出一个异常。

[程序清单5-18 arrays/CopyOfTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch05/arrays/CopyOfTest.java)

### 5.9.7 调用任意方法和构造器
反射机制允许你调用任意方法。`Method`类有一个`invoke()`方法，可以调用包装在当前`Method`对象中的方法。`invoke()`方法的签名为

```java
Object invoke(Object obj, Object... args)
```

第一个参数是隐式参数，其余的参数是显式参数。如果`m`是一个`Method`类型的对象（对应类`C`中名为`f`的方法），`obj`是一个类`C`的对象，则`m.invoke(obj, args...)`等价于调用`obj.f(args...)`。

对于静态方法，第一个参数将被忽略，可以设置为`null`。

例如，如果`m1`表示`Employee`类的`getName()`方法，可以如下调用这个方法：

```java
String n = (String) m1.invoke(harry);
```

等价于`String n = harry.getName();`。

如果返回类型是基本类型，`invoke()`方法会返回其包装器类型。例如，假设`m2`表示`Employee`类的`getSalary()`方法，那么返回的对象实际上是一个`Double`，可以使用自动拆箱将其转换为`double`：

```java
double s = (Double) m2.invoke(harry);
```

如何获得`Method`对象呢？可以调用`getDeclaredMethods()`或`getMethods()`方法，然后搜索返回的`Method`数组。也可以调用`getMethod()`方法，并提供方法名和参数类型。`getMethod()`方法的签名为

```java
Method getMethod(String name, Class... parameterTypes)
```

例如，下面展示了如何获得`Employee`类的`getName()`和`raiseSalary()`方法对应的`Method`对象：

```java
Method m1 = Employee.class.getMethod("getName");
Method m2 = Employee.class.getMethod("raiseSalary", double.class);
```

可以使用类似的方法调用任意构造器：

```java
Class cl = Random.class; // or any other class with a constructor that accepts a long parameter
Constructor cons = cl.getConstructor(long.class);
Object obj = cons.newInstance(42L);
```

下面实际使用`Method`对象。程序清单5-19中的程序会打印一个数学函数（如sqrt或sin）的函数值表。打印结果如下所示：

```
public static double java.lang.Math.sqrt(double)
    1.0000 |     1.0000
    2.0000 |     1.4142
    3.0000 |     1.7321
    4.0000 |     2.0000
    5.0000 |     2.2361
    6.0000 |     2.4495
    7.0000 |     2.6458
    8.0000 |     2.8284
    9.0000 |     3.0000
   10.0000 |     3.1623
```

[程序清单5-19 methods/MethodTableTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch05/methods/MethodTableTest.java)

这种编程风格不是很方便，而且总是容易出错。如果在调用方法时提供了错误的参数，`invoke()`方法将抛出一个异常。另外，`invoke()`的参数和返回类型必须是`Object`类型，这意味着必须来回进行多次强制类型转换。这样，编译器将丧失检查代码的机会，等到测试阶段才会发现错误，而此时查找和修正错误会麻烦得多。不仅如此，使用反射的代码要比直接调用方法的代码慢得多。

鉴于此，建议仅在绝对必要时才使用`Method`对象。更好的做法是使用接口以及Java 8引入的lambda表达式（下一章介绍）。

## 5.10 继承的设计技巧
在本章的最后给出一些使用继承时有用的提示。

**1.将公共方法和字段放在超类中。** 因此，我们将姓名字段放在`Person`类中，而没有重复放在`Employee`和`Student`类中。

**2.不要使用受保护字段。** `protected`机制并不能提供太多保护，原因有两个。第一，子类集合是无限的，任何人都能继承你的类并编写直接访问`protected`字段的代码，从而破坏封装性。第二，在Java中，同一个包中的所有类都可以访问`protected`字段，无论是否是子类。

**3.使用继承来建模 "is-a" 关系。** 使用继承可以节省代码量，但有时也会被滥用。例如，假设需要一个钟点工类`Contractor`。钟点工有姓名和雇佣日期，但是没有工资。他们按小时计薪，并且不会因为待的时间足够长而加薪。这似乎在诱导人们由`Employee`派生出子类`Contractor`，并添加一个`hourlyWage`字段。

```java
public class Contractor extends Employee {
    private double hourlyWage;
    ...
}
```

然而，这并不是一个好主意，因为每个钟点工对象会同时有工资和时薪两个字段。在实现打印薪水或税单的方法时，这会带来无尽的麻烦。与不使用继承相比，使用继承最后反而会多写很多代码。钟点工与员工之间不是 "is-a" 关系，钟点工不是员工的特例。

**4.除非所有继承的方法都有意义，否则不要使用继承。** 假设我们想编写一个`Holiday`类。毫无疑问，每个假日也是一天，并且一天可以用`GregorianCalendar`类的实例表示，因此可以使用继承。

```java
class Holiday extends GregorianCalendar { ... }
```

遗憾的是，在继承的操作中，假日集合不是闭合的。`GregorianCalendar`的一个公有方法`add()`可以将假日转换成非假日：

```java
Holiday christmas;
christmas.add(Calendar.DAY_OF_MONTH, 12);
```

因此，继承对于这个例子来说不合适。如果扩展一个不可变类（如`LocalDate`）就不会出现这个问题。

**5.覆盖方法时，不要改变预期的行为。** 替换原则不仅适用于语法，更重要的是适用于行为。覆盖一个方法时，不应该毫无缘由地改变它的行为。例如，可以重新定义`add()`方法来“修正”`Holiday`类的问题，可能让它什么也不做，或者抛出异常，或者前进到下一个假日。然而，这种“修正”违反了替换原则。不管`x`的类型是`GregorianCalendar`还是`Holiday`，语句序列

```java
int d1 = x.get(Calendar.DAY_OF_MONTH);
x.add(Calendar.DAY_OF_MONTH, 1);
int d2 = x.get(Calendar.DAY_OF_MONTH);
System.out.println(d2 - d1);
```

都应该有**预期的行为**。问题就在这里：什么是预期的行为？重要的是，在子类中覆盖方法时，不要偏离最初的设计初衷。

**6.使用多态，而不是类型信息。** 只要看到以下形式的代码

```java
if (x instanceof type1)
    action1(x);
else if (x instanceof type2)
    action2(x);
```

就应该考虑使用多态。如果`action1`和`action2`表示相同的概念，就应该将其定义为这两个类型的公共超类或接口的方法。然后，就可以调用`x.action()`，并利用多态固有的动态绑定机制执行正确的动作。与使用多个类型检测的代码相比，使用多态方法或接口实现的代码更易于维护和扩展。

**7.不要滥用反射。** 反射机制对于系统编程极其有用，但是通常并不适合编写应用程序。反射很脆弱——如果使用反射，编译器将无法帮助你查找编程错误，在运行时发现的任何错误都会导致异常。
