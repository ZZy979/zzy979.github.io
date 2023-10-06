---
title: 《C++程序设计原理与实践》笔记 第14章 设计图形类
date: 2023-03-06 01:49:03 +0800
categories: [C/C++, PPP]
tags: [cpp, gui, fltk, inheritance, virtual function, object-oriented programming]
---
本章借助图形接口类介绍接口设计的思想和继承的概念。为此，本章将介绍与面向对象程序设计直接相关的语言特性：类派生、虚函数和访问控制。

## 14.1 设计原则
我们的图形接口类的设计原则是什么？

### 14.1.1 类型
我们的程序设计理念是**在代码中直接表示应用领域的概念**。例如，`Window`表示窗口，`Line`表示一条线，`Point`表示一个坐标点，`Color`表示颜色，`Shape`表示所有形状的统称。最后一个例子`Shape`与其他例子的区别在于它是一个一般化的、抽象的概念。我们永远无法在屏幕上看到一个“一般形状”，只能看到线、六边形等具体形状。这一点已经反应在我们的类型定义中：创建一个`Shape`变量将导致编译错误。

我们的图形接口类构成了一个库。这些类旨在作为你定义其他图形类时的参考示例，以及作为复杂图形类的基本组件。考虑到我们的库的规模以及图形应用领域的庞大，我们不能指望完整性。相反，我们的目标是**简洁性和可扩展性**。

一个关键设计决策是**提供很多具有较少操作的“小”类**，而不是一个带有很多参数和操作的类。我们认为这样能够更加直接、有效地建模我们的图形领域。

### 14.1.2 操作
我们的理念是**用最小的接口来实现我们想做的事情**。更加便捷的操作可以通过非成员函数或新的类来实现。

我们希望类的接口具有一致的风格。例如，在不同的类中所有执行相似操作的函数有相同的名字，接受相同类型的参数，还可能要求参数的顺序也相同。
* 构造函数：如果形状需要一个位置，则接受一个`Point`作为第一个参数。
* 所有处理点的函数都使用`Point`，而不是一对`int`。
* 如果函数需要宽度和高度，参数总是按这个顺序出现。在这种微小的细节上保持一致会极大地方便使用，并减少运行时错误。
* 逻辑上等价的操作有相同的名字。例如，向任何形状添加点的函数都叫`add()`，任何画线的函数都叫`draw_lines()`。这种一致性能帮助我们记忆（需要记住的细节更少）以及设计新类（“就跟往常一样”）。有时，这种一致性甚至允许我们编写能用于很多不同类型的代码，称为泛型，见19~21章。

### 14.1.3 命名
逻辑上不同的操作应该有不同的名字。但是，为什么是将`Shape` "attach" 到`Window`，而将`Point` "add" 到`Shape`？这两种情况都是“将一个东西放到另一个东西中”，但这种相似性背后隐藏了一个根本的不同点：`Shape::add()`的参数是传值，而`Window::attach()`的参数是传引用。例如，对于

```cpp
opl.add(Point(100, 100));
```

`opl`会保存这个点的副本，而实际参数`Point(100, 100)`这个临时对象在`add()`调用之后就消失了。另一方面，对于

```cpp
win.attach(opl);
```

`win`并不会创建`opl`的副本，它只是保存`opl`的一个引用。因此，我们必须保证在`win`使用`opl`时不能离开`opl`的作用域。这意味着我们不能创建一个对象，将它附加到窗口之后就立即销毁，例如：

```cpp
win.attach(Rectangle(Point(100, 200), 50, 30));
```

在这种情况下，当`add()`调用完成后，`win`所引用的对象已经不存在了（变成了“野指针”）。这就是13.10节的例子中必须使用`Vector_ref`管理未命名对象的原因。

### 14.1.4 可变性
当我们设计一个类时，“谁可以修改其数据（表示）？”以及“如何修改？”是必须回答的关键问题。我们试图保证只有类自身能够修改其对象的状态（即`private`成员），从而有机会检查“愚蠢的”值，例如半径为负数的`Circle`。

## 14.2 Shape类
`Shape`类是一个一般概念，表示可以显示在屏幕上的对象：
* 将图形对象与`Window`关联起来，从而提供了与物理屏幕的联系。
* 处理画线所使用的颜色和线型，因此它包含一个`Line_style`、一个画线的`Color`和一个填充的`Color`。
* 包含了一系列`Point`以及画线的默认方法（依次连接）。

下面首先给出完整的类，之后讨论其实现细节。

```cpp
class Shape  {        // deals with color and style, and holds sequence of lines 
public:
    void draw() const;                 // deal with color and draw lines
    virtual void move(int dx, int dy); // move the shape +=dx and +=dy

    void set_color(Color col);
    Color color() const;
    void set_style(Line_style sty);
    Line_style style() const;
    void set_fill_color(Color col);
    Color fill_color() const;

    Point point(int i) const;       // read only access to points
    int number_of_points() const;

    Shape(const Shape&) = delete;   // prevent copying
    Shape& operator=(const Shape&) = delete;

    virtual ~Shape() {}
protected:
    Shape();
    Shape(initializer_list<Point> points);  // add() the Points to this Shape

    virtual void draw_lines() const;   // draw the appropriate lines
    void add(Point p);                 // add p to points
    void set_point(int i,Point p);     // points[i]=p;
private:
    vector<Point> points;              // not used by all shapes
    Color lcolor;                      // color for lines and characters
    Line_style ls; 
    Color fcolor;                      // fill color
};
```

### 14.2.1 一个抽象类
`Shape`的构造函数是`protected`，这意味着：
* 只有`Shape`的派生类可以直接使用它（使用`:Shape`语法）。
* `Shape`只能用作其他类（例如`Line`和`Open_polyline`）的基类。
* 不能直接创建`Shape`对象，这反映了“我们无法看见一个一般形状”的思想（如14.1.1节所述）。

如果一个类只能被用作基类，则它是**抽象类**(abstract class)。另一种更常用的定义抽象类的方法是纯虚函数，见14.3.5节。与抽象类相对的是**具体类**(concrete class)，即可以创建对象的类。

声明`virtual ~Shape() {}`定义了一个虚析构函数，将在17.5.2节中解释。

### 14.2.2 访问控制
`Shape`类将所有数据成员均声明为`private`，因此需要提供访问函数。这里选择了一种较为简单、方便、易读的风格：**如果有一个表示属性`X`的成员，则提供一对函数`X()`和`set_X()`分别用于该成员的读写**（这种成员函数分别叫做取值函数(**getter**)和设值函数(**setter**)）。例如：

```cpp
void Shape::set_color(Color col) {
    lcolor = col;
}

Color Shape::color() const {
    return lcolor;
}
```

这种风格最主要的不便之处在于成员变量和取值函数不能用相同的名字。我们给取值函数选择最方便的名字，因为它们是公共接口的一部分，而私有变量的命名不那么重要（注：另一种常用的方法是让所有成员变量以`_`结尾）。注意，我们用`const`指出取值函数不修改对象。

`Shape`保存了一个`Point`的向量，叫做`points`。由于`points`是私有的，因此提供了一组访问函数：

```cpp
// protected
void Shape::add(Point p) {
    points.push_back(p);
}

// protected
void Shape::set_point(int i, Point p) {
    points[i] = p;
}

Point Shape::point(int i) const {
    return points[i];
}

int Shape::number_of_points() const {
    return points.size();
}
```

只有`Shape`的派生类（例如`Circle`和`Polygon`）才知道这些点的含义，而`Shape`只是存储它们。因此，派生类需要控制如何添加点：
* `Circle`和`Rectangle`不允许用户添加点，因为没有意义。
* `Lines`只允许添加成对的点。
* `Open_polyline`和`Marks`允许添加任意多个点。
* `Polygon`需要对添加的点进行相交性检查。

函数`add()`是`protected`（即只能被类自身和派生类访问），从而保证由派生类控制如何添加点，而如果`add()`是`public`或`private`都无法实现这一保证。

类似地，`set_point()`也是`protected`，即只有派生类知道是否可以在不违反不变式的前提下修改点。

在派生类的成员函数中可以使用这些函数，例如`Lines::draw_lines()`（见13.3节）。

这些访问函数不会使程序变慢，因为它们会被编译器优化掉（内联）。调用`number_of_points()`和直接调用`points.size()`使用一样多的内存，执行一样多的指令。

### 14.2.3 绘制形状
`Shape`类最基本的功能是绘制形状。它借助FLTK和操作系统的机制来完成这一工作，但是从用户的视角，它只是提供了两个函数：
* `draw()`：设置线型和颜色，然后调用`draw_lines()`。
* `draw_lines()`：在屏幕上绘制像素。

`draw()`函数只是简单地调用FLTK函数来设置颜色和线型，接着调用`draw_lines()`在屏幕上进行实际的绘制，最后将颜色和线型恢复到调用之前：

```cpp
void Shape::draw() const {
    Fl_Color oldc = fl_color();
    // there is no good portable way of retrieving the current style
    fl_color(lcolor.as_int());            // set color
    fl_line_style(ls.style(),ls.width()); // set style
    draw_lines();
    fl_color(oldc);      // reset color (to previous)
    fl_line_style(0);    // reset line style to default
}
```

（既然每次绘制之前都要设置颜色和线型，那么恢复的意义是什么？）

注意，`draw()`不处理填充颜色或者线的可见性，这些都由`draw_lines()`处理。

现在考虑如何实现`draw_lines()`。让一个`Shape`类的函数来完成每一种形状的绘制是非常困难的，而`Text`、`Rectangle`和`Circle`等大多数派生类都有更好的绘制方式。因此`Shape`类为每个派生类提供了自己定义绘制方式的机会，这就是将`Shape::draw_lines()`声明为`virtual`的意义所在：

```cpp
class Shape {
    // ...
    // let each derived class define its own draw_lines() if it so chooses
    virtual void draw_lines() const;
    // ...
};

struct Circle : Shape {
    // ...
    void draw_lines() const override;  // override Shape::draw_lines()
    // ...
};
```

因此，如果`Shape`是一个`Circle`，那么`Shape`的`draw_lines()`必须以某种方式调用`Circle`的`draw_lines()`；如果`Shape`是一个`Rectangle`，则调用`Rectangle`的`draw_lines()`。这正是关键字`virtual`所保证的：**如果派生类定义了与基类的虚函数名字和类型相同的函数，那么（通过基类指针或引用）调用该函数时，调用的是派生类的函数而不是基类的函数**，这种技术称为**覆盖**(overriding)。

注意，尽管在`Shape`中处于核心地位，`draw_lines()`还是被定义成`protected`。它不是给“一般用户”调用的（这是`draw()`的目的），而只是作为一个“实现细节”被`draw()`和`Shape`的派生类使用。

这样就完成了12.2节中的显示模型：驱动屏幕的系统知道`Window`；`Window`知道`Shape`，并可以调用其`draw()`函数；最后，`draw()`调用特定形状类的`draw_lines()`函数。

![显示模型](/assets/images/ppp-note-ch14-graphics-class-design/显示模型.png)

`gui_main()`简单地调用`Fl::run()`，我们使用`Simple_window::wait_for_button()`来代替了。

`Shape`的`move()`函数简单地将保存的每个点相对于当前位置移动一个偏移量：

```cpp
// move the shape +=dx and +=dy
void Shape::move(int dx, int dy) {
    for (int i = 0; i<points.size(); ++i) {
        points[i].x+=dx;
        points[i].y+=dy;
    }
}
```

`move()`也是虚函数，因为派生类可能有需要移动的数据，而`Shape`并不知道，例如`Axis`。包含不在`Shape`中存储的点的形状类必须定义自己的`move()`。

### 14.2.4 拷贝和可变性
`Shape`类将拷贝构造函数和拷贝赋值运算符声明为`delete`：

```cpp
Shape(const Shape&) = delete;   // prevent copying
Shape& operator=(const Shape&) = delete;
```

这样做的效果是**禁用默认的拷贝操作**。

但是拷贝在很多地方都有用。如果没有拷贝，甚至难以使用`vector`（`push_back()`将参数的**拷贝**放在向量中）。所以为什么要禁止拷贝？**如果一个类型的默认拷贝操作可能引起麻烦，就应该禁止拷贝。**

标准库中禁止拷贝的例子：
* `istream`和`ostream`：输入/输出流有复杂的内部状态，允许拷贝意味着两个流共享输入/输出设备，这可能引起麻烦。
* `unique_ptr`：本身的含义就是“拥有对象**唯一**所有权的指针”，允许拷贝违反了这一不变式。

注：
* `= delete`是C++11引入的新语法，在此之前通过将拷贝构造函数和拷贝赋值运算符声明为`private`来禁止拷贝。
* 书中给出的禁止拷贝的第一个原因是“截断”，例如将一个`Circle`赋值给一个`Shape`，或者添加到`vector<Shape>`。然而这是C++语言本身的特性决定的，不应该成为禁止拷贝的原因。
* 第二个原因是传引用参数，例如`Window::attach()`的参数必须是传引用，`Window`不能保存`Shape`的拷贝，因为对原`Shape`对象的修改不会影响到副本。但这是由`Window`本身决定的，与`Shape`是否可拷贝无关。

当把一个派生类对象（通过值拷贝）赋给基类对象时，如果基类没有禁止拷贝，并且派生类有额外的成员，则会发生**截断**(slicing)。例如，`Circle`比`Shape`多一个成员`r`，如果将一个`Circle`赋值给`Shape`，则这个`Circle`对象会被截断，成员`r`并不会被拷贝。

![截断](/assets/images/ppp-note-ch14-graphics-class-design/截断.png)

注：书中说“使用拷贝后的`Shape`可能会引起崩溃，因为没有拷贝成员`r`”是不正确的。因为只有在值拷贝时才会发生截断，而此时变量类型是`Shape`（而不是`Shape*`或`Shape&`），通过该变量调用`draw_lines()`函数时调用的是`Shape::draw_lines()`而不是`Circle::draw_lines()`，根本不可能访问到成员`r`。

下面是通过禁止拷贝来避免截断的例子：

```cpp
void my_fct(Open_polyline& op, const Circle& c) {
    Open_polyline op2 = op;  // error: Shape's copy constructor is deleted
    vector<Shape> v;
    v.push_back(c);          // error: Shape's copy constructor is deleted
    // ...
    op = op2;                // error: Shape's assignment is deleted
}

Marked_polyline mp("x");
Circle c(p,10);
my_fct(mp,c);  // the Open_polyline argument refers to a Marked_polyline
```

* `v.push_back(c)`将一个`const Circle&`传递给`const Shape&`参数，这一步没有问题。但`push_back()`内部将参数拷贝到向量中时会发生截断，因为向量元素类型是`Shape`，而参数实际引用了一个`Circle`对象。
* 函数`my_fct()`的参数`op`实际引用了一个`Marked_polyline`对象，如果将其拷贝到`op2`则会发生截断。

如果想要拷贝一个默认拷贝操作已经被禁用的类型的对象，可以显式地写一个函数来完成这一工作。这种拷贝函数通常叫做`clone()`。显然，只有当读取成员的函数足够表达构造副本所需的内容时才能编写出`clone()`，所有的`Shape`类都已满足这一条件。

## 14.3 基类和派生类
下面从一个更加技术性的视角来讨论基类和派生类。当设计图形接口库时，我们依赖三个关键的语言机制：
* **派生**(derivation)：从一个类构造另一个类，使得新类可以代替原来的类。其中新类叫做**派生类**(derived class)或**子类**(subclass)，原来的类叫做**基类**(base class)或**父类/超类**(superclass)。这通常称为**继承**(inheritance)，因为派生类除了自己的成员外，还获得（“继承”）了基类的所有成员。例如，`Circle`派生（继承）自`Shape`，换句话说，“`Circle`是一种`Shape`”或者“`Shape`是`Circle`的基类”。
* **虚函数**(virtual function)：在基类中定义一个函数、在派生类中有一个名称和类型相同的函数，**当用户（通过基类指针或引用）调用基类函数时，调用的实际上是派生类的函数**。这通常称为**运行时多态**(run-time polymorphism)、**动态分派**(dynamic dispatch)或**覆盖**(overriding)，因为调用哪个函数是在运行时根据实际使用的对象类型来确定的。例如，当`Window`（通过`Shape*`或`Shape&`）对一个实际是`Circle`的`Shape`调用`draw_lines()`函数时，实际调用的是`Circle`的`draw_lines()`，而不是`Shape`本身的`draw_lines()`。
* **私有和保护成员**(private and protected members)：保持类的实现细节（数据成员）为私有的，保护它们不被直接使用而使得维护复杂化。这通常称为**封装**(encapsulation)。

继承、多态和封装的使用是**面向对象程序设计**(object-oriented programming)最常见的定义。因此，除了其他的程序设计风格之外，C++直接支持面向对象程序设计。

12.4节给出了图形接口类的继承关系图，箭头从派生类指向基类。

注：**派生类的指针或引用可以直接赋给基类的指针或引用**；反之则必须使用`dynamic_cast`，如果转换失败则返回空指针或抛出`std::bad_cast`。例如：

```cpp
Circle c(Point(100, 100), 50);
Shape* ps = &c;  // OK
Circle* pc = dynamic_cast<Circle*>(ps);  // OK
Rectangle* pr = dynamic_cast<Rectangle*>(ps);  // nullptr

Shape& rs = c;  // OK
Circle& rc = dynamic_cast<Circle&>(rs);  // OK
Rectangle& rr = dynamic_cast<Rectangle&>(rs);  // std::bad_cast
```

### 14.3.1 对象布局
如9.4.1节所述，一个类的成员定义了对象在内存中的布局：**数据成员在内存中一个接一个地存储。** 当使用继承时，**派生类的成员被添加在基类成员之后**。例如，`Shape`和`Circle`的对象布局如14.2.4节所示。

为了处理虚函数调用，我们必须在对象中存储更多信息，用于区分调用虚函数时实际调用的是哪个函数。常用方法是增加一个函数表的地址，这个表通常称为`vtbl`（ "virtual table" 或 "virtual function table" ，**虚函数表**），它的地址通常称为`vptr`（ "virtual pointer" ，**虚指针**）。将`vptr`和`vtbl`加入布局图中得到下图：

![vtbl](/assets/images/ppp-note-ch14-graphics-class-design/vtbl.png)

基本上，**虚函数调用生成的代码简单地寻找`vptr`，通过它找到`vtbl`，然后调用其中正确的函数。** 其代价大约是两次内存访问加上一次普通函数调用，既简单又快速。

每个具有虚函数的类只有一个`vtbl`，而不是每个对象都有一个，因此`vtbl`并不会显著增加程序目标代码的大小。

注意，上图中没有画出任何非虚函数，因为这种函数的调用方式没有任何特殊之处，它们不会增加对象的大小。

定义一个和基类中虚函数的名字和类型都相同的函数，使得派生类的函数代替基类的版本被放入`vtbl`的技术称为**覆盖**(overriding)。

### 14.3.2 派生类和定义虚函数
我们通过在类名后给出一个基类来指定一个类是派生类。例如：

```cpp
class D : public B {
    // ...
};
```

或

```cpp
struct D : B {
    // ...
};
```

其中基类前的`public`关键字表示公有继承，详见14.3.4节。

虚函数必须在类内被声明为`virtual`（ "virtual" 意味着“可以被覆盖”(can be overriden)）。但是如果把函数定义放在类外，则函数定义中不必也不能使用关键字`virtual`。例如：

```cpp
class B {
public:
    virtual void f();
};

void B::f() {
    // ...
}  
```

注：
* **不要在构造函数中调用虚函数。** 因为在基类和派生类的构造函数中，虚函数表是不一样的。这意味着在构造函数中调用虚函数时调用的是当前类的函数，无法多态调用。如果允许多态调用，则在基类的构造函数中调用派生类的虚函数时，由于派生类还未初始化，因此有可能访问到未初始化的数据成员。
* 由于以上原因，`Shape`类的`add()`和`set_color()`函数不是虚函数。如果派生类需要自定义这两个函数只能使用隐藏。

关于基类和派生类的构造函数：
* 派生类可以在构造函数的成员初始化列表中调用基类的构造函数（例如[Graph.h](https://github.com/ZZy979/PPP-code/blob/main/GUI/Graph.h) `Open_polyline`）。
* 如果派生类构造函数没有显式调用基类构造函数，则调用基类的默认构造函数。如果基类没有默认构造函数，则编译器将报错。
* 派生类可以使用`using`声明继承构造函数（例如`using Base::Base;`），继承的构造函数与基类构造函数有相同的可见性（例如[Graph.h](https://github.com/ZZy979/PPP-code/blob/main/GUI/Graph.h) `Closed_polyline`）。

### 14.3.3 覆盖
覆盖虚函数时，必须使用与基类中完全相同的名字和类型（参数表、是否`const`）。例如：

```cpp
class B {
public:
    virtual int f(int x);
};

class D : public B {
public:
    int f(int x) override;
};
```

注：
* 如果派生类的函数覆盖了基类的虚函数，则派生类的函数也是虚函数。
* 在派生类中可以通过`B::f(x)`调用基类被覆盖的函数。
* 从C++11开始，可以使用`override`来声明虚函数覆盖。如果一个函数声明了`override`，但并未覆盖基类的虚函数，或者基类对应的函数不是`virtual`，将产生编译错误，从而可以让编译器来保证覆盖了正确的虚函数。这对于大而复杂的类层次结构是非常有帮助的。
* 覆盖虚函数时，派生类中的函数可以声明或不声明为`virtual`，也可以声明或不声明为`override`。最好的做法是：只将基类中的虚函数声明为`virtual`，派生类中覆盖的函数声明为`override`（这已经意味着该函数也是虚函数），如上面的例子所示。
* **C++允许派生类改变基类虚函数的可见性**：覆盖虚函数只要求具有相同的函数名和参数表，而不关心返回类型和访问修饰符。这意味着派生类的`public`函数可以覆盖基类的`private`虚函数，或者相反（在Java中这是不可行的，子类方法的可见性不能低于覆盖的父类方法，子类也不能覆盖父类的`private`方法）。详见[virtual function specifier - In detail](https://en.cppreference.com/w/cpp/language/virtual#In_detail)。例如：

```cpp
class B {
public:
    void f() { do_f(); }
    virtual void g() { cout << "B::g()\n"; }
private:
    virtual void do_f() { cout << "B::f()\n"; }
};

class D : public B {
public:
    void do_f() override { cout << "D::f()\n"; }
private:
    void g() override { cout << "D::g()\n"; }
};

int main() {
    D d;
    B* p = &d;
    p->f();
    p->g();
    return 0;
}
```

程序将输出

```
D::f()
D::g()
```

* 如果派生类定义了一个与基类中名字和类型完全相同的函数，但基类中的函数不是虚函数，则称为**隐藏**(hide)。隐藏与覆盖的区别是：当通过基类指针或引用调用函数时，覆盖调用的是派生类的函数，而隐藏调用的是基类的函数。见下面的例子。

★下面通过一个纯技术性的例子来解释覆盖：

[覆盖的例子](https://github.com/ZZy979/PPP-code/blob/main/ch14/overriding_example.cpp)

程序的输出如下：

```
B::f()
B::g()
D::f()
B::g()
D::f()
B::g()
B::f()
B::g()
D::f()
D::g()
DD::f()
DD:g()
```

这里的几个关键点：
* `call()`的参数类型是`const B&`，不知道参数的实际类型，只知道是`B`或`B`的派生类，因此通过运行时多态来确定实际调用的函数。另外，只能调用`const`成员函数。
* 第3~4行由`call(d)`输出，参数的实际类型是`D`。由于`D::f()`覆盖了`B::f()`，因此`b.f()`调用的是`D::f()`；由于`D::g()`不是`const`，因此`b.g()`调用的是的`B::g()`。
* 第5~6行由`call(dd)`输出，参数的实际类型是`DD`。由于`DD::f()`不是`const`，且`DD`未覆盖`D::f()`，因此`b.f()`调用的是`D::f()`；由于`DD::g()`隐藏（而不是覆盖）了`B::g()`，因此`b.g()`调用的是`B::g()`。
* 对于最后6行，当通过变量调用函数时，由于已知变量的实际类型，且变量不是`const`，因此会优先调用变量的类型自己定义的函数。

当你理解了为什么是这样的结果，你就会明白继承和虚函数机制了。

### 14.3.4 访问
C++为类成员访问提供了一个简单的模型。类成员可以是：
* **公有的**(public)：如果一个成员是`public`，则可以被所有函数访问。
* **受保护的**(protected)：如果一个成员是`protected`，则只能被类自身和派生类访问。
* **私有的**(private)：如果一个成员是`private`，则只能被类自身访问。

继承也分为公有继承、受护继承和私有继承：
* **公有继承**(public inheritance)：基类的`public`成员对派生类是`public`，`protected`成员对派生类是`protected`，`private`成员对派生类不可见。
* **受保护继承**(protected inheritance)：基类的`public`和`protected`成员对派生类是`protected`，`private`成员对派生类不可见。
* **私有继承**(private inheritance)：基类的`public`和`protected`成员对派生类是`private`，`private`成员对派生类不可见。

| 继承方式\基类成员 | `public` | `protected` | `private` |
| --- | --- | --- | --- |
| `public` | `public` | `protected` | 不可见 |
| `protected` | `protected` | `protected` | 不可见 |
| `private` | `private` | `private` | 不可见 |

注：
* 这些定义忽略了友元(friend)的概念和一些次要的细节，这不在本书的范围之内。
* **类默认私有继承，结构体默认公有继承。**

### 14.3.5 纯虚函数
**抽象类**(abstract class)是只能作为基类的类。我们使用抽象类来表示抽象的概念。抽象概念的思想是极其有用的，例如形状/矩形，动物/狗，水果/苹果。在程序中，抽象类通常定义了一组相关的类（**类层次结构**(class hierarchy)）的接口。

14.2.1节展示了如何通过将构造函数声明为`protected`来定义抽象类。另一种更常用的方法是声明一个或多个**纯虚函数**(pure virtual function)，即必须被派生类覆盖的虚函数。纯虚函数通过语法`= 0`来声明，例如：

```cpp
class B {
public:
    virtual void f() = 0;  // pure virtual function
    virtual void g() = 0;
};

B b;  // error: B is abstract
```

因为`B`有纯虚函数，我们不能创建`B`类的对象。覆盖所有的纯虚函数可以解决这一“问题”：

```cpp
class D1 : public B {
public:
    void f() override;
    void g() override;
};

D1 d1; // OK
```

注意，**除非所有的纯虚函数都被覆盖了，否则派生类仍然是抽象的**：

```cpp
class D2 : public B {
public:
    void f() override;
    // no g()
};

D2 d2;  // error: D2 is (still) abstract

class D3 : public D2 {
public:
    void g() override;
};

D3 d3;  // OK
```

带有纯虚函数的类通常作为纯粹的接口，即它们通常没有数据成员（数据成员将在派生类中定义），因此也没有构造函数。

## 14.4 面向对象程序设计的好处
通过继承，我们可以获得（其中之一或两者都有）：
* **接口继承**(interface inheritance)：需要基类（指针或引用）参数的函数可以接受一个派生类对象（并且可以通过基类提供的接口使用派生类对象）。例如，`Window::attach(Shape&)`可以接受`Shape`的任何派生类对象。
* **实现继承**(implementation inheritance)：当定义派生类及其成员函数时，我们可以使用基类提供的功能（例如数据成员和成员函数）。例如，`Line`直接复用了`Shape::draw_lines()`，`Closed_polyline::draw_lines()`调用`Open_polyline::draw_lines()`来完成大部分的绘制。

一个不能提供接口继承的设计（即派生类对象不能被当作其公有基类的对象使用）是一个拙劣且容易出错的设计。

接口继承之所以得名，是因为其优点：使用基类提供的接口的代码无需知道具体的派生类。实现继承之所以得名，是因为其优点：基类提供的功能简化了派生类的实现。

注意，我们的图形库设计严重依赖于接口继承：“图形引擎”调用`Shape::draw()`，进而调用虚函数`Shape::draw_lines()`完成实际的绘制工作。无论是“图形引擎”还是`Shape`类都不知道有哪些具体形状。特别是，“图形引擎”（FLTK加上操作系统的图形功能）是在我们的图形类之前若干年就编写、编译好的！我们只是定义了特定的形状，并将其作为`Shape`附加到`Window`中。而且，由于`Shape`类不知道你的图形类，当你每次定义新的图形类时，不需要重新编译`Shape`类。

换句话说，我们可以向程序中添加新形状，而不用修改已有的代码。这是一个软件设计/开发/维护的圣杯：扩展一个系统而不用修改它。哪些改进不必修改已有的类还是有一定限制的（例如`Shape`提供了非常有限的服务），同时这种技术也不是对所有的程序设计问题都能很好地应用（例如第17~19章定义的`vector`，继承机制对其没什么用处）。然而，接口继承仍然是设计和实现对于改进需求的鲁棒性（健壮性）很强的系统的最有力的技术之一。

同时，实现继承也能带来很多好处，但它不是灵丹妙药。通过将有用的服务放在`Shape`中，我们避免了在派生类中一遍又一遍地重复工作。这对于现实世界中的程序设计尤为重要。然而，它的代价是任何对于`Shape`接口或数据成员布局的修改都必须重新编译所有的派生类及其用户代码。对于一个广泛使用的库来说，这种重新编译是绝对行不通的。

## 简单练习
[drill14](https://github.com/ZZy979/PPP-code/blob/main/ch14/drill14.cpp)

## 习题
* [14-1](https://github.com/ZZy979/PPP-code/blob/main/ch14/exec14-1.cpp)
* [14-11~14-13](https://github.com/ZZy979/PPP-code/blob/main/ch14/exec14-11.cpp)
* [14-15](https://github.com/ZZy979/PPP-code/blob/main/ch14/iterator.h)
* [14-16](https://github.com/ZZy979/PPP-code/blob/main/ch14/controller.h)
* [14-17](https://github.com/ZZy979/PPP-code/blob/main/ch14/standard_exception_class_hierarchy.png)
