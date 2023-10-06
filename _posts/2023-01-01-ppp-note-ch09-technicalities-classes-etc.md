---
title: 《C++程序设计原理与实践》笔记 第9章 类相关的技术细节
date: 2023-01-01 14:49:44 +0800
categories: [C/C++, PPP]
tags: [cpp, class, struct, constructor, enumeration, operator overloading]
---
在本章中，我们继续关注主要的程序设计工具——C++语言。本章主要介绍与用户自定义类型（即类和枚举）相关的语言技术细节。这些语言特性大部分是以逐步改进一个Date类型的方式来介绍的。采用这种方式，我们还可以顺便介绍一些有用的类设计技术。

## 9.1 用户自定义类型
C++语言提供了一些**内置类型**(built-in type)，例如`char`、`int`和`double`。对于一个类型，如果编译器无须借助程序员在源代码中提供的声明就知道如何表示这种类型的对象以及可以对它进行什么样的运算（例如`+`和`*`），则称这种类型是内置的。

非内置的类型称为**用户自定义类型**(user-defined type, UDT)。用户自定义类型可以是标准库类型，如`string`、`vector`、`ostream`；也可以是我们为自己创建的类型，如`Token`、`Token_stream`。与内置类型一样，大多数用户自定义类型提供运算。例如，`vector`有`[]`和`size()`，`ostream`有`<<`，`Token_stream`有`get()`，`Shape`有`add()`和`set_color()`（见14.2节）。

编译器不可能知道我们想在程序中使用的所有类型，因此我们需要自己创建类型。这些类型带来的帮助体现在两个方面：
* **表示**(representation)：类型“知道”如何表示对象需要的数据
* **操作/运算**(operation)：类型“知道”可以对对象进行什么操作/运算

很多想法都遵循这种模式：“某个东西”有一些数据表示当前状态（值），和一组可以进行的操作（LY：抽象数据类型(ADT) = 数学模型 + 操作）。例如：计算机文件、网页、烤面包机、音乐播放器、咖啡杯、汽车引擎、手机、电话号码簿——这些都可以用一些数据描述，并且或多或少支持一组固定的标准操作，操作的结果依赖于对象的数据（“当前状态”）。

我们希望在代码中将这样一个“想法”或“概念”表示为一个数据结构加上一组函数。在C++中可以通过用户自定义类型来实现。

C++提供了两种用户自定义类型：类和枚举。

## 9.2 类和成员
**类**(class)是一个用户自定义类型，由**数据成员**(data member)（可以是内置类型或其他用户自定义类型）、**成员函数**(member function)和**成员类型**(member type)组成，这些用来定义类的组成部分统称为**成员**(member)。例如：

```cpp
class X {
public:
    int m;  // data member

    // function member
    int mf(int v) {
        int old = m;
        m = v;
        return old;
    }
};
```

注意：**不要漏掉结尾的分号！**

数据成员定义了类对象的表示方法，成员函数提供了对象的运算（操作）。可以使用符号 `对象.成员` 来访问成员。例如：

```cpp
X var;              // var is a variable of type X
var.m = 7;          // assign to var's data member m
int x = var.mf(9);  // call var's member function mf()
```

数据成员可以像普通变量一样读写，成员函数可以像普通函数一样调用。

在成员函数中，成员名称指的是成员函数被调用的那个对象中的成员。因此，调用`var.mf(9)`时，`mf()`定义中的`m`指的是`var.m`。

## 9.3 接口和实现
我们通常把类看作一个**接口**(interface)加一个**实现**(implementation)。接口是类的用户直接访问的部分，实现是用户通过接口间接访问的部分。公共接口使用标签`public:`标识，实现使用标签`private:`标识。可以像这样理解类声明：

```cpp
class X {  // this class's name is X
public:
    // public members:
    //     – the interface to users (accessible by all)
    // functions
    // types
    // data (often best kept private)
private:
    // private members:
    //     – the implementation details (used by members of this class only)
    // functions
    // types
    // data
};
```

**类成员默认是私有的。** 用户（类外部的代码）不能直接访问私有成员，必须通过使用它的公有函数来访问。

注：**成员访问限制是针对类，而不是针对类的不同对象。** 因此在一个类的成员函数中既可以访问当前对象的私有成员，也可以访问同类型其他对象的私有成员。

例如：

```cpp
class X {
    int m;
    int mf(int);
public:
    int f(int i) { m = i; return mf(i); }
    int g(X x) { return m + x.m; }  // OK
};

X x;
int y = x.mf();  // error: mf is private (i.e., inaccessible)
int z = x.f(2);  // OK
```

我们使用`public`和`private`来表示接口（用户视角的类）和实现细节（实现者视角的类）之间的重要区别。对于单纯的数据，这种区别没有意义。因此，对于没有私有实现细节的类，C++提供了一种简化的语法：**结构体** `struct`。**结构体就是成员默认为公有的类**：

```cpp
struct X {
    int m;
    int mf(int);
};
```

意味着

```cpp
class X {
public:
    int m;
    int mf(int);
};
```

结构体主要用于成员可以取任意值的数据结构，即我们不能定义任何有意义的不变式（见9.4.3节）。

## 9.4 演化一个类
下面通过展示如何以及为什么将一个简单的数据结构逐步演化为一个具有私有实现细节和运算的类，来解释支持类的语言功能和使用类的基本技术。这里使用一个看似微不足道的问题：如何在程序中表示日期（例如1954年8月14日）。

### 9.4.1 结构体和函数
如何表示一个日期？最简单的方式是使用年、月、日。第一次尝试是使用一个简单的`struct`：

```cpp
// simple Date (too simple?)
struct Date {
    int y;  // year
    int m;  // month in year
    int d;  // day of month
};

Date today{2005, 12, 24};
```

一个`Date`对象就是三个`int`（没有隐藏的“魔法”）：

![Date对象](/assets/images/ppp-note-ch09-technicalities-classes-etc/Date对象.png)

对于这个版本的`Date`，我们可以访问其对象的成员并任意读写，因此可以对它做任何操作。也正因为这样，任何操作都不方便，也容易出错。例如：

```cpp
// print today: tedious
cout << today.y << '-' << today.m << '-' << today.d << endl;

// invalid date
today.y = -3;
today.m = 13;
today.d = 32;
```

较好的方式是提供一些辅助函数来完成最常见的操作，从而不必再一次次重复相同的代码，也不必再一次次犯相同的错误以及查找、修正这些错误。例如，对于`Date`类可以编写初始化和增加日期值的辅助函数：

```cpp
// helper functions:

// check that (y,m,d) is a valid date
// if it is, use it to initialize dd
void init_day(Date& dd, int y, int m, int d) {
    // ...
}

// increase dd by n days
void add_day(Date& dd, int n) {
    // ...
}
```

注意这些“操作”（这里实现为辅助函数而不是成员函数）是很有用的。

每当我们决定提供一个类型都要问自己：“我们希望对这种类型执行哪些操作？”

### 9.4.2 成员函数和构造函数
我们为`Date`提供了初始化函数，它提供了重要的合法性检查功能。然而，如果使用不当的话，检查函数将毫无用处。例如：

```cpp
void f() {
    Date today;
    // ... use today
    init_day(today, 2008, 3, 30);
    // ...
    Date tomorrow;
    tomorrow.y = today.y;
    tomorrow.m = today.m;
    tomorrow.d = today.d + 1;  // add 1 to today
    // ... use tomorrow
}
```

在这段代码中，我们“忘记了”立即对`today`进行初始化，而在调用`init_day()`之前就使用了它。另外，我们通过将成员`d`加1的方式手动构造了`tomorrow`，而不是调用`add_day()`，这会成为定时炸弹：当`today`表示月底那一天时，加1会产生一个非法日期。这段“问题严重的代码”最大的问题在于它看起来没什么问题。

因此，我们需要一种不会被忘记的初始化函数和不太可能被忽略的操作。实现这些目标的基本工具是**成员函数**(member function)，即在类内部声明为类成员的函数，例如：

```cpp
// simple Date
// guarantee initialization with constructor
// provide some notational convenience
struct Date {
    int y, m, d;                // year, month, day
    Date(int y, int m, int d);  // check for valid date and initialize
    void add_day(int n);        // increase the Date by n days
};
```

与类同名、没有返回值的特殊成员函数称为**构造函数**(constructor)，用于类对象的初始化（构造）。如果一个类没有定义构造函数，则编译器会自动生成一个无参数的构造函数（默认构造函数）；**如果定义了有参数的构造函数，则初始化对象时未提供需要的参数将导致编译错误，除非显式定义了默认构造函数**。

#### C++初始化语法
C++提供了专门的、方便的语法来进行这种初始化：

（1）[直接初始化](https://en.cppreference.com/w/cpp/language/direct_initialization)

```cpp
T object(arg1, arg2, ...);  // (1)
T(arg1, arg2, ...)          // (2)
```

其中，形式(1)通过调用构造函数`T::T(arg1, arg2, ...)`初始化`object`，形式(2)调用同样的构造函数初始化一个未命名的临时对象。

（2）[列表初始化](https://en.cppreference.com/w/cpp/language/list_initialization)

```cpp
T object{arg1, arg2, ...};     // (1) 
T{arg1, arg2, ...}             // (2)
T object = {arg1, arg2, ...};  // (3)
```

其中，形式(2)等价于直接初始化的形式(2)，形式(1)和(3)这种初始化语法有多种解释：
* 如果类`T`没有定义构造函数，则参数为各个公有成员的初始值，称为[聚合初始化](https://en.cppreference.com/w/cpp/language/aggregate_initialization)。
    * 例如 `Token t{'8', 3.14};`（见6.3.3节）
* 如果类`T`定义了只接受一个`std::initializer_list`类型参数的构造函数，则调用该构造函数。
    * 例如：`vector<int> v{1, 2, 3, 4, 5};`
* 否则调用类`T`对应的构造函数`T::T(arg1, arg2, ...)`，此时等价于直接初始化，如果没有匹配的构造函数则编译器报错。
    * 例如：`Date today{2005, 12, 24};`

注：这里仅列出了（相当）简化的规则，C++标准定义的准确规则相当复杂，见[列表初始化](https://en.cppreference.com/w/cpp/language/list_initialization) - Explanation一节

对于本节定义的`Date`：

```cpp
Date my_birthday;                   // error: my_birthday not initialized
Date today{12,24,2007};             // oops! run-time error
Date last{2000,12,31};              // OK (colloquial style)
Date last2(2000,12,31);             // OK (old colloquial style)
Date next = {2014,2,14};            // also OK (slightly verbose)
Date christmas = Date{1976,12,24};  // also OK (verbose style)
```

`my_birthday`的定义是错误的，因为`Date`定义了有参数的构造函数，并且没有定义默认构造函数，而这里没有提供参数。`today`的定义会通过编译，但构造函数中的检查代码（这里没有给出）会在运行时捕获非法的日期（12年24月2007日）。`last`的定义提供了初值——`Date`的**构造函数所需的参数，位置是紧跟在变量名后的{}列表（这里等价于()），这是具有带参数构造函数的类的变量最常见的初始化方式。** 另外，也可以使用更加冗长的风格，例如`next`和`christmas`的定义（`christmas`定义中的`{}`也可以换成`()`），除非你确实喜欢打字，否则你很快就会厌烦这种方式。

现在可以使用新定义的变量。调用成员函数的语法是`对象.成员函数名(参数表)`，例如：

```cpp
last.add_day(1);
```

本节定义的`Date`没有给出成员函数的定义，9.4.4节将介绍如何定义成员函数。

### 9.4.3 保持细节私有性
现在还有一个问题：如果有人忘记使用成员函数`add_day()`怎么办？如果有人决定直接修改月份怎么办？

```cpp
Date birthday{1960,12,31};
++birthday.d;  // ouch! Invalid date (birthday.d==32 makes today invalid)

Date today{1970,2,3};
today.m = 14;  // ouch! Invalid date (today.m==14 makes today invalid)
```

只要`Date`的表示还是对所有人可访问的（即数据成员是公有的），那么就会有人（有意或无意地）把事情搞乱，即制造出非法的日期值。

因此，**类的表示对用户应该是不可访问的，除非通过类提供的公有成员函数。** 下面是改进后的版本：

```cpp
// simple Date (control access)
class Date {
public:
    Date(int y, int m, int d);  // constructor: check for valid date and initialize
    void add_day(int n);        // increase the Date by n days
    int year() { return y; }
    int month() { return m; }
    int day() { return d; }
private:
    int y, m, d;  // year, month, day
};
```

“有效/合法日期”的概念是**有效值/合法值**(valid value)思想的一个特例。我们设计类型时应保证所有的值都是有效的，即隐藏表示，构造函数只创建有效对象，成员函数被调用时期望有效值、返回时留下的仍然是（修改后的）有效值。对象的值通常称为**状态**(state)，因此有效值通常称为对象的**有效状态/合法状态**(valid state)。

构成有效值的规则称为**不变式**(invariant)。`Date`的不变式：一个`Date`对象必须对应日历上的某一天，同时需要考虑闰年等问题。如果不能想出一个好的不变式，那么我们可能处理的是普通数据，如果是这样就使用`struct`。

### 9.4.4 定义成员函数
到目前为止，我们已经从接口设计者和用户的视角看了`Date`类，但迟早要实现那些成员函数。

注意9.3节给出的类定义框架，通常把公共接口放在开头，私有实现细节放在后面，因为接口是大多数人所感兴趣的。编译器并不关心成员的声明顺序。

在类外定义一个成员时，需要使用`类名::成员名`指明它是哪个类的成员：

```cpp
Date::Date(int yy, int mm, int dd)  // constructor
        :y(yy), m(mm), d(dd) {      // member initializers
}

void Date::add_day(int n) {
    // ...
}

int month() {   // oops: we forgot Date::
    return m;   // not the member function, can't access m
}
```

在构造函数的定义中，`:y(yy), m(mm), d(dd)`是初始化成员的语法，叫做**成员初始化列表**([member initializer list](https://en.cppreference.com/w/cpp/language/constructor))（见6.8.1节），每个括号前面是成员名，括号内是对应的初值。用在成员初始化列表中的构造函数参数可以和成员同名。

注：
* 成员初始化列表中的每一项实际上就是变量初始化的语法。对于内置类型的成员，`y(yy)`等价于`y{yy}`，相当于`int y(yy);`、`int y{yy};`以及`int y = yy;`；对于用户自定义类型的成员，可以使用9.4.2节中的直接初始化和列表初始化语法。
* 基类和派生类的构造函数相关问题见14.3.2节。

构造函数也可以写成：

```cpp
Date::Date(int yy, int mm, int dd) {
    y = yy;
    m = mm;
    d = dd;
}
```

但这样是先对成员进行默认初始化，然后再赋值（对于内置类型，默认初始化就是未初始化，见3.3、3.9和8.2.3节）。这种写法无法排除在初始化之前使用成员的可能性，而成员初始化列表更直接地表达了我们的意图，二者之间的区别和下面两段代码的区别是一样的：

```cpp
int x;  // first define the variable x
// ...
x = 2;  // later assign to x
```

```cpp
int x = 2;  // define and immediately initialize with 2
```

也可以直接在类内定义成员函数：

```cpp
class Date {
public:
    Date(int y, int m, int d)
            :y(yy), m(mm), d(dd) {
    }

    void add_day(int n) {
        // ...
    }

    int year() { return y; }
    int month() { return m; }
    int day() { return d; }
private:
    int y, m, d;  // year, month, day
};
```

这会使得类定义变得大而凌乱。因此，不要在类内定义大函数。而对于`month()`这种小而简单的函数可以考虑直接在类内定义。

注意，`month()`可以引用定义在其下面的`m`。**类成员对其他成员的引用并不依赖于成员在类中的声明位置。** 8.2节中“名字必须先声明后使用”的规则在类作用域中可以放宽。

将成员函数的定义放在类内有三方面的影响：
* 函数将称为**内联**(inline)的，即编译器直接将函数体嵌入到调用点，而不是生成函数调用指令，从而避免了函数调用的开销（传递参数和返回值）。对于`month()`这种做的工作很少、又被频繁使用的函数，这会带来很大的性能提升。
* 每当对内联函数体做出修改时，所有使用这个类的代码都不得不重新编译。如果函数体位于类外，则只有在类定义本身改变时使用代码才需要重新编译，对于大程序来说这是一个巨大的优势。
* 类定义会变得更大，因此在成员函数定义之间寻找接口和成员会更加困难。

经验法则是：**不要将成员函数定义放在类内，除非需要通过内联非常小的函数获得性能提升。**

### 9.4.5 引用当前对象
考虑`Date`类的一个简单使用：

```cpp
void f(Date d1, Date d2) {
    cout << d1.month() << ' ' << d2.month() << '\n';
}
```

`Date::month()`没有任何参数，它是如何知道第一次调用返回`d1.m`、第二次调用返回`d2.m`的呢？类成员函数有一个**隐式参数**（`this`指针）用来识别调用它的对象，详见17.10节。

### 9.4.6 报告错误
当我们发现非法日期时应该抛出异常，检查代码应该放在构造函数中。如9.4.3节所述，如果没有创建非法的`Date`对象（即初始状态是有效状态），而且成员函数也编写正确（即有效状态→有效状态），那么就永远不会得到具有无效值的`Date`对象。

```cpp
// simple Date (prevent invalid dates)
class Date {
public:
    class Invalid {};           // to be used as exception
    Date(int y, int m, int d);  // constructor: check for valid date and initialize
    // ...
private:
    int y, m, d;      // year, month, day
    bool is_valid();  // return true if date is valid
};
```

这里将检查有效性的代码放到一个单独的函数`is_valid()`中，并在构造函数中调用该函数。`Date::Invalid`是成员类型，用于发现非法日期时抛出的异常。

```cpp
Date::Date(int yy, int mm, int dd)
        :y(yy), m(mm), d(dd) {
    if (!is_valid()) throw Invalid();  // check for validity
}

// return true if date is valid
bool Date::is_valid() {
    if (m < 1 || m > 12) return false;
    // ...
}
```

从而可以写出如下代码：

```cpp
void f(int x, int y) {
    try {
        Date dxy{2004, x, y};
        cout << dxy << '\n';   // see §9.8 for a declaration of <<
        dxy.add_day(2);
    }
    catch(Date::Invalid) {
        error("invalid date");  // error() defined in §5.6.3
    }
}
```

我们将在9.7节完成`Date`类的演化。在此之前，先介绍几个常用的语言功能：枚举和运算符重载。

## 9.5 枚举
**枚举**(enumeration) `enum` 是一种非常简单的用户自定义类型，将一组值（**枚举项/枚举成员**(enumerator)）指定为符号常量。例如：

```cpp
enum class Month {
    jan = 1, feb, mar, apr, may, jun, jul, aug, sep, oct, nov, dec
};
```

`enum class`中的`class`意味着**枚举项在枚举的作用域内**，因此必须通过`Month::jan`来引用`jan`。

可以为枚举项指定特定的值，例如上面的`jan = 1`。如果不指定，编译器将为每个枚举项赋予前一个枚举项的值加1，第一个枚举项的值默认为0。**同一个枚举中的不同枚举项可以具有相同的值。** 例如：

```cpp
enum class Foo { a, b, c = 10, d, e = 1, f, g = f + c };
// a = 0, b = 1, c = 10, d = 11, e = 1, f = 2, g = 12
```

可以像这样使用`Month`：

```cpp
Month m = Month::feb;
Month m2 = feb;       // error: feb is not in scope
```

枚举的底层类型是`int`（或其他整数类型），但枚举是一个独立的类型。每个枚举项都有一个等价的整型值，但反之不成立。

注意：
* `enum class`枚举类型不能使用`int`初始化，不能与`int`相互隐式转换，也不能参与整型运算（除非重载了运算符）。例如：

```cpp
Month m = 7;          // error: can't assign an int to a Month
int n = m;            // error: can't assign a Month to an int
++m;                  // error: no match for operator++ (operand type is Month)
int x = m + 1;        // error: no match for operator+ (operand types are Month and int)
cout << m << endl;    // error: no match for operator<< (operand types are std::ostream and Month)
```

* 枚举类型可以通过显式类型转换或`static_cast`与`int`相互转换，此时编译器不检查整型值是否为该枚举的有效值（毕竟程序员可能更清楚自己在做什么）。例如：

```cpp
Month bad = Month(9999);              // convert int to Month (unchecked)
Month bad2 = static_cast<Month>(-1);  // convert int to Month (unchecked)
int x = int(bad);                     // convert Month to int, x == 9999
int y = static_cast<int>(bad2);       // convert Month to int, y == -1
```

我们不能为枚举定义构造函数来检查初始值，但可以编写一个简单的检查函数：

```cpp
Month int_to_month(int x) {
    if (x < int(Month::jan) || x > int(Month::dec)) error("bad month");
    return Month(x);
}
```

**枚举适用于需要一组命名整型常量的情况。** 例如：

```cpp
enum class Position { up, down };
enum class Answer { yes, no, maybe };
enum class Switch { on, off };
enum class Direction { n, ne, e, se, s, sw, w, nw };
enum class Color { red, blue, green, yellow, maroon, crimson, black };
```

### 9.5.1 “普通”枚举
`enum class`也叫作**有作用域枚举**(scoped enumeration)。旧式的“普通”枚举也叫作无作用域枚举(unscoped enumeration)隐式地将其枚举项“导出”到枚举所在的作用域，并允许隐式转换为`int`。例如：

```cpp
enum Month {  // note: no “class”
    jan = 1, feb, mar, apr, may, jun, jul, aug, sep, oct, nov, dec
};

Month m = feb;          // OK: feb in scope
Month m2 = Month::feb;  // also OK
m = 7;                  // error: can't assign an int to a Month
int n = m;              // OK: we can assign a Month to an int
Month mm = Month(7);    // convert int to Month (unchecked)
```

显然，无作用域枚举不如有作用域枚举严格。无作用域枚举项会“污染”枚举所在的作用域。例如，如果将`Month`与\<iostream\>一起使用（并且使用了`using namespace std;`），则表示十二月的`dec`会与表示十进制(decimal)的`dec`冲突，表示十月的`oct`会与表示八进制(octal)的`oct`冲突。

另外，枚举值能够转换为`int`可能会很方便，但有时会导致意外的结果。例如：

```cpp
void my_code(Month m) {
    if (m == 17) do_something();           // huh: 17th month?
    if (m == monday) do_something_else();  // huh: compare month to Monday?
}
```

如果`Month`是有作用域枚举，则两个条件都会编译失败。如果`monday`是无作用域枚举项，则`m == monday`能够编译通过，但很可能不是想要的结果。

因此，优先使用更简单、更安全的有作用域枚举，但在旧的代码中可能会看到无作用域枚举，因为`enum class`是C++11引入的新特性。

注：C语言的枚举本质上就是整数，支持使用`int`初始化、与`int`相互隐式转换以及参与整型运算，见[《C程序设计语言》笔记 第2章 类型、运算符与表达式]({% post_url 2022-02-24-tcpl-note-ch2-types-operators-and-expressions %}) “枚举常量”一节。

## 9.6 运算符重载
你可以对类或枚举对象定义几乎所有的C++运算符，这称为**运算符重载**(operator overloading)。重载运算符就是具有特殊名字的函数，例如`operator+`、`operator bool`。这种机制用于为用户自定义类型提供习惯的符号表示。例如，为`Month`定义前缀自增运算符：

```cpp
// prefix increment operator
Month operator++(Month& m) {
    m = (m == dec) ? jan : Month(int(m) + 1);  // "wrap around"
    return m;
}

Month m = Month::sep;
++m;  // m becomes oct
++m;  // m becomes nov
++m;  // m becomes dec
++m;  // m becomes jan ("wrap around")
```

其中`?:`是“算术if”运算符：`expr1 ? expr2 : expr3`，当`expr1`为真时表达式的值为`expr2`，否则为`expr3`。

也可以定义输出运算符：

```cpp
vector<string> month_tbl{
    "Unknown", "January", "February", "March", "April", "May", "June",
    "July", "August", "September", "October", "November", "December"
};

ostream& operator<<(ostream& os, Month m) {
    return os << month_tbl[int(m)];
}
```

你可以为自己的类型定义几乎所有的C++运算符，如`+`, `–`, `*`, `/`, `%`, `[]`, `()`, `^`, `!`, `&`, `<`, `<=`, `>`, `>=`等。

注意：
* 不能定义新的运算符，例如`**`、`$=`。
* **重载运算符时，操作数的个数必须与原来一样**（操作数个数即函数参数个数）。例如，不能定义一元的`<=`或二元的`!`。
* **重载运算符必须至少有一个用户自定义类型的操作数。** 例如，不能定义`int operator+(int, int)`。
* 有些重载运算符可以定义为成员函数或非成员函数，而有些必须定义为成员函数。具体规则见[operator overloading](https://en.cppreference.com/w/cpp/language/operators)。
* 重载运算符本质上仍然是函数调用。例如，`a + b`等价于`a.operator+(b)`或`operator+(a, b)`，`++a`等价于`a.operator++()`或`operator++(a)`。

一般性的原则是：**除非你真正确定重载运算符能大大改善代码，否则不要为你的类型定义重载运算符。** 而且，应该按照常规含义定义运算符：`+`就应该表示加法，二元`*`表示乘法，`[]`表示元素访问，`()`表示调用，等等。

注意，重载最多的运算符不是`+`、`-`、`*`、`/`，而是`=`、`==`、`!=`、`<`、`<<`、`[]`和`()`。

## 9.7 类接口
9.3节已经提到过类的公共接口和实现细节应该分离。以下是一些设计好的接口的一般原则：
* 保持接口完整
* 保持接口最小化
* 提供构造函数
* 支持（或禁止）拷贝（见14.2.4节）
* 使用类型来提供完善的参数检查
* 标识出`const`成员函数（见9.7.4节）
* 在析构函数中释放所有资源（见17.5节）

### 9.7.1 参数类型
9.4.3节中为`Date`定义的构造函数使用了三个`int`作为参数。这会带来一些问题：

```cpp
Date d1{4, 5, 2005};  // oops: year 4, day 2005
Date d2{2005, 4, 5};  // April 5 or May 4?
```

第一个问题（非法日期）比较容易处理，在构造函数中检查即可。第二个问题（混淆月和日）是由于书写日期的习惯不同而造成的：例如，4/5在美国表示4月5日，而在英国表示5月4日。一种显然的解决方案是使用`Month`类型：

```cpp
// simple Date (use Month type)
class Date {
public:
    Date(int y, Month m, int d);  // check for valid date and initialize
    // ...
private:
    int y;  // year
    Month m;
    int d;  // day
};
```

这样编译器将会捕获颠倒月和日的错误。另外，使用符号名字比直接使用数字更不容易出错：

```cpp
Date dx1{1998, 4, 3};            // error: 2nd argument not a Month
Date dx2{1998, 4, Month::mar};   // error: 2nd argument not a Month
Date dx2{4, Month::mar, 1998};   // oops: run-time error: day 1998
Date dx2{Month::mar, 4, 1998};   // error: 2nd argument not a Month
Date dx3{1998, Month::mar, 30};  // OK
```

注意代码中使用枚举项`mar`的限定名称`Month::mar`，而不是`Month.mar`，因为`Month`是类型而不是对象，`mar`是枚举项（符号常量）而不是数据成员。**在类、枚举和命名空间的名字后使用`::`，在对象的名字后使用`.`。**

如果可以选择，最好在编译时而不是运行时捕获错误，这样就不需要编写和执行检查代码。

### 9.7.2 拷贝
为了编写构造函数，必须确定如何初始化对象，以及什么样的值是有效值（什么是不变式）。下一个要考虑的问题是：可以拷贝对象吗？如果可以，如何拷贝？

对象的拷贝由**拷贝构造函数**(copy constructor)完成。只要不特别声明，编译器就会提供一个默认的拷贝构造函数——拷贝所有成员。对于`Date`和`Month`，拷贝操作就是默认方式。

**当一个对象由另一个相同类型的对象初始化时，拷贝构造函数将被调用。** 包括变量初始化、函数参数传递和函数返回值。例如：

```cpp
Date holiday{1978, Month::jul, 4};    // initialization
Date d2 = holiday;                    // copy
Date d3 = Date{1978, Month::jul, 4};  // copy
```

其中，`holiday`的初始化没有拷贝，`d2`和`d3`的初始化各执行一次拷贝。

`Date{1978, Month::jul, 4}`（或者`Date(1978, Month::jul, 4)`）创建了一个未命名的`Date`对象，这里对构造函数的使用可以充当类的字面值（见9.4.2节直接初始化和列表初始化的最后一种格式）。

如果不想要拷贝的默认行为，可以定义自己的拷贝构造函数（见18.3节），或者删除拷贝构造函数和拷贝赋值运算符（见14.2.4节）。

拷贝构造函数规则的完整定义见[Copy constructors](https://en.cppreference.com/w/cpp/language/copy_constructor)。

### 9.7.3 默认构造函数
未初始化的变量是错误之源。为了解决这个问题，可以用构造函数来保证类的每个对象都被初始化。例如：

```cpp
Date d0;                  // error: no initializer
Date d1{};                // error: empty initializer
Date d2{1998};            // error: too few arguments
Date d3{1,2,3,4};         // error: too many arguments
Date d4{1,"jan",2};       // error: wrong argument type
Date d5{1,Month::jan,2};  // OK: use the three-argument constructor
Date d6{d5};              // OK: use the copy constructor
Date d7 = d6;             // OK: use the copy constructor
```

注意，除了三个参数的构造函数，还可以通过拷贝构造函数来初始化`Date`。

很多类都具有默认值的概念，即“没有提供初始值时应该具有什么值？”例如：

```cpp
string s;          // default value: empty string " "
vector<string> v;  // default value: empty vector; no elements
```

这是通过`vector`和`string`的默认构造函数实现的，可以隐式地进行所需的初始化。

**对于类型`T`，符号`T{}`或`T()`表示默认值，由默认构造函数定义。** 因此以上代码等价于

```cpp
string s = string();
vector<string> v = vector<string>();
```

**对于内置类型，“默认构造函数”符号表示0。** 因此`int{}`表示`0`，`double{}`表示`0.0`。

基本上，没有构造函数就无法建立不变式，也就不能保证变量中的值是有效的。**我们必须坚持对变量进行初始化。**

为类型提供默认值的方法是定义一个无参数的构造函数，称为**默认构造函数**(default constructor)。但是，对于很多类型来说，并不容易找到一个合理的默认值概念，例如`Date`。

下面为`Date`定义一个默认构造函数（只是为了说明可以这样做），选择21世纪的第一天作为默认值：

```cpp
class Date {
public:
    Date();  // default constructor
    // ...
};

Date::Date() :y(2001), m(Month::jan), d(1) {
}
```

除了将成员默认值放在构造函数中，也可以放在成员声明中：

```cpp
class Date {
    Date(int y);  // January 1 of year y
    // ...
private:
    int y = 2001;
    Month m = Month::jan;
    int d = 1;
};

Date::Date(int yy) :y(yy) {
    if (!is_valid()) throw Invalid();  // check for validity
}
```

这种在类成员声明中指定的初始值称为**类内初始值**(in-class initializer)。这些初始值在所有构造函数中都是可用的。例如，上面的构造函数`Date(int yy)`没有显式初始化`m`和`d`，因此使用类内初始值`Month::jan`和`1`，而`y`被初始化为`yy`。

### 9.7.4 const成员函数
类的成员函数分为两类：可修改(modifying)和不可修改(nonmodifying)。nonmodifying成员函数也叫**const成员函数**，是指不会修改数据成员的值的成员函数，可以在`const`对象上调用。例如：

```cpp
class Date {
public:
    // ...
    int year() const;       // const member: can't modify the object
    Month month() const;    // const member: can't modify the object
    int day() const;        // const member: can't modify the object

    void add_year(int n);   // non-const member: can modify the object
    void add_month(int n);  // non-const member: can modify the object
    void add_day(int n);    // non-const member: can modify the object
    // ...
};

Date d{2000, Month::jan, 20};
const Date cd{2001, Month::feb, 21};

cout << d.day() << " — " << cd.day() << '\n';  // OK
d.add_day(1);   // OK
cd.add_day(1);  // error: cd is a const
```

通过在参数表后紧跟着`const`来表示该成员函数是`const`成员函数，声明和定义要保持一致。一旦将一个成员函数声明为`const`，编译器将保证该函数不会修改数据成员。例如：

```cpp
int Date::day() const {
    ++d;  // error: attempt to change object from const member function
    return d;
}
```

### 9.7.5 成员和“辅助函数”
当我们设计接口使其最小化时，不得不忽略大量有用的操作。**如果一个函数可以简单、优美、高效地实现为一个独立函数（即非成员函数），就应该在类外实现。** 这样，函数中的bug就不会直接破坏类对象中的数据——debug时首先检查的是直接访问类的表示的函数。另一个重要原因是，如果类的表示改变了，只有直接访问表示的函数（成员函数）才需要重写。

下面是一些**辅助函数**(helper function)的例子：

```cpp
Date next_Sunday(const Date& d) { /* ... */ }

Date next_weekday(const Date& d) { /* ... */ }

bool leapyear(int y) { /* ... */ }

bool operator==(const Date& a, const Date& b) {
    return a.year() == b.year()
            && a.month() == b.month()
            && a.day() == b.day();
}

bool operator!=(const Date& a, const Date& b) {
    return !(a == b);
}
```

辅助函数也称为convenience function、auxiliary function等。“辅助函数”只是一种设计概念，而不是编程语言概念。辅助函数通常接受它所辅助的类对象作为参数，但也有例外，例如`leapyear()`。

注意`==`和`!=`是典型的辅助函数。由于它们不是对所有类都有意义，因此编译器无法像拷贝构造函数和拷贝赋值运算符那样为你提供默认定义。

## 9.8 Date类
现在，将本章所有的思想组合在一起得到最终的`Date`类。

将声明放在头文件[Chrono.h](https://github.com/ZZy979/PPP-code/blob/main/ch09/Chrono.h)中，定义放在[Chrono.cpp](https://github.com/ZZy979/PPP-code/blob/main/ch09/Chrono.cpp)，并使用命名空间`Chrono`。

## 简单练习
[测试Date类](https://github.com/ZZy979/PPP-code/blob/main/ch09/Chrono_test.cpp)

## 习题
* [9-2~9-3](https://github.com/ZZy979/PPP-code/blob/main/ch09/name_pairs.h)
* [9-4](https://github.com/ZZy979/PPP-code/blob/main/ch09/nesting_example.cpp)
* [9-5~9-7](https://github.com/ZZy979/PPP-code/blob/main/ch09/book.h)
* [9-8](https://github.com/ZZy979/PPP-code/blob/main/ch09/patron.h)
* [9-9](https://github.com/ZZy979/PPP-code/blob/main/ch09/library.h)
* [9-10~9-11](https://github.com/ZZy979/PPP-code/blob/main/ch09/Chrono.cpp)
* [9-13](https://github.com/ZZy979/PPP-code/blob/main/ch09/relational.h)
