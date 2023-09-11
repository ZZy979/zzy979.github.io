---
title: 《C++程序设计原理与实践》笔记 第16章 图形用户界面
date: 2023-04-09 15:53:48 +0800
categories: [C/C++, PPP]
tags: [cpp, gui, fltk]
---
**图形用户界面**(graphical user interface, GUI)允许用户通过点击按钮、选择菜单、以不同的方式输入数据以及在屏幕上显示文本和图形等方式与程序进行交互。在本章中，我们将介绍编写代码来定义和控制GUI应用的基本方法。

## 16.1 用户界面的选择
每个程序都有用户接口/界面(interface)。程序员有三种主要的选择：控制台（命令行）、图形用户界面和网络浏览器。

GUI是一种I/O形式，应用程序的主要逻辑和I/O相互分离是我们对软件的理想之一。这种分离允许我们改变程序呈现给用户的方式、将程序移植到不同的I/O系统，最重要的是允许我们将程序逻辑和用户交互分开来考虑。

## 16.2 "Next" 按钮
`Simple_window`的 "Next" 按钮是如何实现的？

第12~15章代码的典型结构如下：

```cpp
// create objects and/or manipulate objects
// attach them to win
win.wait_for_button();

// create objects and/or manipulate objects
// attach them to win
win.wait_for_button();

// ...
```

每当运行到`wait_for_button()`，就可以在屏幕上看到要显示的对象，直到我们点击 "Next" 按钮来得到程序下一部分的输出。

从程序逻辑的观点来看，这与输出到控制台窗口，然后停下来再从键盘接收输入的程序没有区别。例如：

```cpp
// define variables and/or compute values, produce output
cin >> var;  // wait for input

// define variables and/or compute values, produce output
cin >> var;  // wait for input

// ...
```

但是从实现的观点来看，这两种程序截然不同。当程序执行到`cin >> var`时将停下来等待输入字符，而GUI程序运行在一种截然不同的模式下：
* “系统”（GUI库+操作系统）负责跟踪鼠标、键盘等设备，并监听其产生的动作/事件（例如“点击按钮”）。
* 用户程序对于关心的事件注册监听器/回调函数（例如“当按钮被点击时执行什么操作”）。
* 在GUI检测到程序感兴趣的动作之前，程序一直处于等待状态。

注：GUI程序与控制台程序的主要区别在于：GUI程序的其主体是一个无限等待循环，何时结束是由用户决定的（点击窗口的关闭按钮）；控制台程序何时结束是由程序员决定的（执行完`main()`函数）。

## 16.3 一个简单的窗口
`Simple_window`类定义在[Simple_window.h](https://github.com/ZZy979/PPP-code/blob/main/GUI/Simple_window.h)中，包含一个成员`next_button`（表示 "Next" 按钮）：

```cpp
struct Simple_window : Graph_lib::Window {
    Simple_window(Point xy, int w, int h, const string& title);
    void wait_for_button(); // simple event loop

private:
    Button next_button;     // the "next" button
    bool button_pushed;     // implementation detail

    static void cb_next(Address, Address); // callback for next_button
    void next();            // action to be done when next_button is pressed
};
```

`Simple_window`派生自`Window`类， "Next" 按钮在构造函数中被初始化：

```cpp
Simple_window::Simple_window(Point xy, int w, int h, const string& title)
    :Window(xy,w,h,title),
    next_button(Point(x_max()-70,0), 70, 20, "Next", cb_next),
    button_pushed(false)
{
    attach(next_button);
}
```

`Simple_window`和`Button`的参数表示（左上角）位置、尺寸和标签（与`Rectangle`类似，因为窗口和按钮都是矩形形状）。`Button`的第四个参数`cb_next`是**按钮被点击时的回调函数**，将在下一节解释。

最后，我们将按钮`attach()`到窗口（类似于`Shape`对象），告知窗口显示这个按钮。

`button_pushed`是一个相当隐晦的细节，用于跟踪自从上次执行`next()`之后按钮是否被点击了。

### 16.3.1 回调函数
函数`cb_next()`就是我们希望**当GUI系统检测到按钮被点击时调用的那个函数**，称为**回调函数**(callback function)。

我们的程序运行在多“层”代码之上：

![代码“层”](/assets/images/ppp-note-ch16-graphical-user-interfaces/代码层.png)

当 "Next" 按钮被点击时，“下层的”某些代码就会调用`cb_next()`函数。

注：GUI系统只知道按钮何时被点击，用户程序只知道按钮被点击时执行什么操作，因此必须通过这种注册监听器/回调函数的方式来实现程序的功能。

在FLTK中，回调函数是接受两个地址参数、没有返回值的函数。因此`cb_next()`函数声明如下：

```cpp
static void cb_next(Address, Address);
```

其中，关键字`static`表示这是一个**静态成员函数**。类的静态成员是指不绑定到任何对象、可以直接通过类名访问的成员。

类型`Address`实际上是`void*`的别名，这是一个**指针**类型（类似于引用），表示某个对象在内存中的地址：

```cpp
typedef void* Address;
```

回调函数的第一个`Address`参数是触发回调的GUI实体(`Widget`)的地址（在这里是 "Next" 按钮）。第二个`Address`参数是包含这个`Widget`的窗口的地址，对于`cb_next()`来说就是当前的`Simple_window`对象。

注：在FLTK中，回调函数的第二个指针参数用于传递任意的用户数据。作者的GUI库将“用户数据”的含义规定为“窗口的地址”。

`cb_next()`的定义如下：

```cpp
void Simple_window::cb_next(Address, Address pw)
// call Simple_window::next() for the window located at pw
{  
    reference_to<Simple_window>(pw).next();    
}
```

`reference_to<Simple_window>(pw)`将指针`pw`强制转换为`Simple_window`的引用。最后，调用成员函数`next()`来执行实际的操作。

注：
* `reference_to`是作者在自定义的一个“强制转换运算符”，等价于`*static_cast<Simple_window*>(pw)`。
* `pw`之所以能够成功转换为`Simple_window*`，是因为`Button`在注册回调函数时提供了包含它的窗口的地址，当按钮被点击时FLTK又将这个地址作为回调函数的第二个参数，见[GUI.cpp](https://github.com/ZZy979/PPP-code/blob/main/GUI/GUI.cpp)中`Button::attach()`的定义。

我们本来可以将所有要执行的代码都放在`cb_next()`中，但是像大多数好的GUI程序员一样，我们更愿意把麻烦的底层内容与精巧的用户代码分离，因此使用两个函数来处理回调：
* `cb_next()`将系统约定的回调函数映射到一个普通的成员函数(`next()`)。
* `next()`实际执行我们期望的动作（不需要了解回调函数的复杂约定）。

使用两个函数的本质原因是一个通用原则：“一个函数应该执行单一的逻辑动作”。

总结：
* 程序初始化时：`Simple_window` → `Button` → FLTK
* 点击按钮时：FLTK → `cb_next()` → `next()`

一旦理解了`next()`是如何被调用的，我们就基本上理解了如何在带有GUI界面的程序中处理每一个动作。

### 16.3.2 等待循环
每次 "Next" 按钮被点击时，`next()`函数要做什么呢？
* 当程序执行到`win.wait_for_button()`时将会暂停，同时绘制已经被`attach()`到窗口上的形状和构件，并等待 "Next" 按钮被按下；
* 当用户点击 "Next" 按钮时，将使得程序继续执行。

实际上，`wait_for_button()`就是一个简单的**等待循环**(wait loop)，（简化的）实现如下：

```cpp
void Simple_window::wait_for_button()
// modified event loop:
// handle all events (as per default), quit when button_pushed becomes true
// this allows graphics without control inversion
{
    button_pushed = false;
    while (!button_pushed) Fl::wait();
    Fl::redraw();
}
```

FLTK提供了一个函数`Fl::wait()`，用于暂停程序运行，直到某个事件发生。

因此，当用户点击 "Next" 按钮时，`next()`函数只需将`button_pushed`置为`true`即可，从而打破上面的等待循环：

```cpp
void Simple_window::next()
{
    button_pushed = true;
}
```

下面以15.5节“指数函数近似”程序为例解释等待循环的工作过程：
* 首先创建`Simple_window`，并添加坐标轴和真正的指数函数`real_exp`。
* 第一次循环(n=0)，添加函数exp0并调用`wait_for_button()`，绘制并显示窗口，进入等待循环。
* 用户点击 "Next" 按钮，程序继续执行，移除函数exp0。
* 第二次循环(n=1)，添加函数exp1并调用`wait_for_button()`，绘制并显示窗口，进入等待循环。
* 用户点击 "Next" 按钮，程序继续执行，移除函数exp1。
* ……

### 16.3.3 Lambda表达式作为回调函数
在`Simple_window`的构造函数中，可以用Lambda表达式来代替`cb_next()`函数：

```cpp
[](Address, Address pw) { reference_to<Simple_window>(pw).next(); }
```

## 16.4 Button和其他Widget
### 16.4.1 Widget
**构件**(widget)也叫**控件**(control)或**组件**(component)，用于定义通过GUI与程序进行交互的形式。

常见的构件包括文本标签(text label)、按钮(button)、输入框(input box)、单选按钮(radio button)、复选框(check box)、下拉列表(combo box)、菜单(menu)等。窗口本身也是一种构件。

不同的构件支持不同的事件，例如点击、右键单击、双击、鼠标移动、按下键盘、内容/值改变、获得/失去焦点、窗口最大化/最小化等，不同的GUI库支持的事件也不同。可以通过监听器/回调函数来设置当这些事件发生时要执行的动作。

注：
* FLTK提供的构件见[Common Widgets and Attributes](https://www.fltk.org/doc-1.3/common.html)。
* 作者的GUI库提供了四种构件：按钮、输入框、输出框和“菜单”，实际上是对FLTK构件的简单封装，类图见12.4节。其中只有`Button`支持设置回调函数，并且只支持“点击”这一种事件。

`Widget`接口类如下：

```cpp
class Widget {
// Widget is a handle to an Fl_widget - it is *not* an Fl_widget
// We try to keep our interface classes at arm's length from FLTK
public:
    Widget(Point xy, int w, int h, const string& s, Callback cb);

    virtual void move(int dx,int dy);
    virtual void hide();
    virtual void show();
    virtual void attach(Window&) = 0;

    Point loc;
    int width;
    int height;
    string label;
    Callback do_it;

    virtual ~Widget() { }

protected:
    Window* own;    // every Widget belongs to a Window
    Fl_Widget* pw;  // connection to the FLTK Widget
private:
    Widget& operator=(const Widget&); // don't copy Widgets
    Widget(const Widget&);
};
```

`hide()`和`show()`分别隐藏和显示`Widget`对象，默认是可见的。

像`Shape`一样，在使用`Widget`之前必须将其`attach()`到窗口，但是`Window::attach(Widget&)`调用了`Widget::attach()`。注意，`Widget::attach()`声明为一个纯虚函数：`Widget`的每个派生类必须定义自己的`attach()`函数，在该函数中创建底层的FLTK构件（例如，`Button::attach()`创建了一个`Fl_Button`并设置回调函数）。结果是一个窗口知道它所包含的构件，每个构件也知道它所属的窗口。

注意，一个`Window`并不知道它所处理的`Widget`的具体类型。如14.4节所述，我们使用面向对象程序设计来保证`Window`可以处理每种类型的`Widget`。同样，一个`Widget`也不知道它所属的`Window`的具体类型。

`Widget`及其派生类的定义在[GUI.h](https://github.com/ZZy979/PPP-code/blob/main/GUI/GUI.h)中。

### 16.4.2 Button
`Button`是最简单的`Widget`。它所做的只是在我们点击它时调用回调函数：

```cpp
struct Button : Widget {
    Button(Point xy, int w, int h, const string& label, Callback cb)
        : Widget(xy,w,h,label,cb) {}

    void attach(Window& win);
};
```

### 16.4.3 In_box和Out_box
这两种构件用于文本输入/输出：

```cpp
struct In_box : Widget {
    In_box(Point xy, int w, int h, const string& s)
        :Widget(xy,w,h,s,0) { }
    int get_int();
    string get_string();

    void attach(Window& win);
};

struct Out_box : Widget {
    Out_box(Point xy, int w, int h, const string& s)
        :Widget(xy,w,h,s,0) { }
    void put(int);
    void put(const string&);

    void attach(Window& win);
};
```

`In_box`可以接受用户输入的文本，我们可以使用`get_string()`将文本读取为字符串，或者使用`get_int()`读取为整数。如果要读取为其他类型，可以使用`stringstream`。

`Out_box`用于向用户呈现信息。我们可以使用`put()`放入字符串或整数。

16.5节给出了使用`In_box`和`Out_box`的例子。

### 16.4.4 Menu
我们提供了一个非常简单的“菜单”：

```cpp
struct Menu : Widget {
    enum Kind { horizontal, vertical };
    Menu(Point xy, int w, int h, Kind kk, const string& label);

    Vector_ref<Button> selection;
    Kind k;
    int offset;
    int attach(Button& b);      // Menu does not delete &b
    int attach(Button* p);      // Menu deletes p

    void show();                // show all buttons
    void hide();                // hide all buttons
    void move(int dx, int dy);  // move all buttons
    void attach(Window& win);   // attach all buttons

};
```

`Menu`本质上就是一个按钮的向量（水平或竖直排列），使用`Vector_ref`跟踪所有的`Button`。宽度和高度的作用是，当给菜单添加按钮时重设按钮的大小。每个按钮（“菜单项”）都是一个独立的`Widget`，通过`Menu::attach(Button&)`添加到`Menu`。`Menu::attach(Window&)`（来自`Widget`的虚函数）将所有`Button`添加到窗口。如果想要“弹出式”菜单，你只能自己实现，参见16.7节。

## 16.5 一个实例
这是一个简单的画线程序，允许用户绘制一系列由坐标对指定的线段：

![画线程序](/assets/images/ppp-note-ch16-graphical-user-interfaces/draw_lines.png)

用户输入的第一个坐标对作为起点，之后每次输入的新坐标对都会用来绘制一条线：一条从当前点（其坐标显示在 "current (x,y)" 框中）到新输入的点(x, y)之间的线，之后(x, y)称为新的当前点。

这实际上绘制了一个`Open_polyline`。该程序展示了几个有用的GUI功能：文本输入/输出、线的绘制和按钮。上面的窗口展示了输入两个坐标对之后的结果，输入7个坐标对得到：

![画线程序2](/assets/images/ppp-note-ch16-graphical-user-interfaces/draw_lines2.png)

下面定义一个表示这种窗口的类：

[Lines_window](https://github.com/ZZy979/PPP-code/blob/fac21209c56d0174a13b906fa79e24c04dfbf01a/ch16/lines_window.h)

线表示为`Open_polyline`。还声明了按钮、输入框和输出框，并且为每个按钮定义了一个实现其功能的成员函数。这里使用Lambda表达式来消除回调函数“样板代码”（类似于`Simple_window::cb_next()`）。

"Quit" 按钮关闭窗口（相当于右上角的关闭按钮），`quit()`函数使用了一个奇怪的FLTK特性——简单地隐藏窗口。

"Next point" 按钮的实际工作在`next()`函数中完成：从输入框读取一对坐标，向`Open_polyline`添加点，更新输出框，并重新绘制窗口。直到调用`Window`的`redraw()`之前，屏幕上显示的一直是旧图像。

这个程序的奇怪之处在于`main()`函数：

[主程序](https://github.com/ZZy979/PPP-code/blob/main/ch16/draw_lines.cpp)

`main()`函数只是定义了窗口并调用函数`gui_main()`，而`gui_main()`只是调用了FLTK的`run()`函数，`run()`只是一个简单的无限等待循环（窗口被关闭后结束循环）：

```cpp
int Fl::run() {
  while (Fl_X::first) wait(FOREVER);
  return 0;
}
```

所以到底发生了什么呢？

## 16.6 控制反转
这里发生的事情是：**我们将执行顺序的控制权从程序交给了构件（用户）**。例如，程序一直等待用户操作，用户点击一个按钮就会调用其回调函数，回调函数返回后，程序就会挂起，继续等待用户执行下一个操作。

“常规程序”的组织结构为

![常规程序](/assets/images/ppp-note-ch16-graphical-user-interfaces/常规程序.png)

“GUI程序”的组织结构为

![GUI程序](/assets/images/ppp-note-ch16-graphical-user-interfaces/GUI程序.png)

“**控制反转**”(control inversion)意味着执行顺序完全由用户的行为决定。这使得程序的组织和调试都更加复杂。因为很难想像用户会做什么，也很难想像一个随机的回调序列所有可能的影响。为了尽量减少麻烦，关键是保证程序的GUI部分简单，以及增量式构建一个GUI程序，在每个阶段都要测试。在编写GUI程序时，画一个对象以及它们之间的交互图是很关键的。

## 16.7 添加菜单
下面通过为画线程序添加菜单来探究“控制反转”带来的控制和通信问题。

首先，我们提供一个菜单，允许用户改变线的颜色。

按钮是动态添加到菜单上的，`Menu::attach()`会调整按钮的尺寸和位置（因此创建时都指定为0），并将其添加到窗口中。

![带有菜单的画线程序](/assets/images/ppp-note-ch16-graphical-user-interfaces/draw_lines_with_menu.png)

为了实现“弹出式”菜单，我们添加了一个 "color menu" 按钮，点击它时“弹出”颜色菜单，选择颜色之后，再次隐藏菜单并显示该按钮。

注：“菜单”由 "red" 、 "blue" 和 "black" 这三个按钮组成， "color menu" 按钮不是菜单的一部分，只是为了实现“弹出式”菜单的效果而用来显示和隐藏菜单的。

这是添加了几条线之后的窗口：

![带有弹出式菜单的画线程序](/assets/images/ppp-note-ch16-graphical-user-interfaces/draw_lines_with_popup_menu.png)

点击 "color menu" 按钮，该按钮会隐藏并出现菜单：

![画线程序-点击菜单按钮](/assets/images/ppp-note-ch16-graphical-user-interfaces/draw_lines_with_menu_black.png)

点击 "blue" 按钮，线会变成蓝色，菜单再次隐藏， "color menu" 按钮重新出现：

![画线程序-蓝色的线](/assets/images/ppp-note-ch16-graphical-user-interfaces/draw_lines_with_menu_blue.png)

下面是`Lines_window`的完整实现：

[Lines_window](https://github.com/ZZy979/PPP-code/blob/cd294cb86ccb4ba9601d73caed18bbf97aeb1981/ch16/lines_window.h)

注意，初始化的顺序和数据成员的声明顺序是一致的。事实上，**成员初始化总是按照成员声明的顺序执行的**。

## 16.8 调试GUI代码
在GUI程序正常工作之前通常会有一个充满挫折的阶段。例如：

```cpp
int main() {
    Lines_window (Point(100, 100), 600, 400, "lines");
    return gui_main();
}
```

程序可以编译通过并运行，但窗口在屏幕上一闪而过，之后程序就结束了。我们“忘记”了`Lines_window`的名字`win`，因此创建了一个未命名的临时变量，之后就立刻销毁了。

另一个常见的问题是将一个窗口/形状恰好放在了另一个窗口/形状上面，看起来好像只有一个窗口/形状，于是浪费宝贵的时间去寻找代码中并不存在的bug。

可能会让事情更糟的是，当使用GUI库时，异常并不总是按期望的那样工作。

在调试中发现的常见问题包括：`Shape`和`Widget`没有被添加到窗口而没有显式；由于超出对象的作用域而导致出错。例如：

```cpp
// helper function for loading buttons into a menu
void load_disaster_menu(Menu& m) {
    Point orig(0, 0);
    Button b1(orig, 0, 0, "flood", cb_flood);
    Button b2(orig, 0, 0, "fire", cb_fire);
    // ...
    m.attach(b1);
    m.attach(b2);
    // ...
}

int main() {
    // ...
    Menu disasters(Point(100, 100), 60, 20, Menu::horizontal, "disasters");
    load_disaster_menu(disasters);
    win.attach(disasters);
    // ...
}
```

这段代码不能运行。这些按钮都是`load_disaster_menu()`函数的局部变量，在该函数返回后，这些局部对象已经被销毁，`disaster`菜单引用的是不存在的（已经被销毁的）对象。详见18.6.4节（不要返回指向局部变量的指针）。解决方法是使用`new`创建未命名对象：

```cpp
// helper function for loading buttons into a menu
void load_disaster_menu(Menu& m) {
    Point orig(0, 0);
    m.attach(new Button(orig, 0, 0, "flood", cb_flood));
    m.attach(new Button(orig, 0, 0, "fire", cb_fire));
    // ...
}
```

正确方法甚至比错误方法更简单。

## 简单练习
[Lines_window](https://github.com/ZZy979/PPP-code/blob/main/ch16/lines_window.h)

## 习题
* [16-3](https://github.com/ZZy979/PPP-code/blob/main/ch16/exec16-3.cpp)
* [16-4](https://github.com/ZZy979/PPP-code/blob/main/ch16/exec16-4.cpp)
* [16-6](https://github.com/ZZy979/PPP-code/blob/main/ch16/exec16-6.cpp)
* [16-9](https://github.com/ZZy979/PPP-code/blob/main/ch16/exec16-9.cpp)
* [16-10](https://github.com/ZZy979/PPP-code/blob/main/ch16/exec16-10.cpp)
