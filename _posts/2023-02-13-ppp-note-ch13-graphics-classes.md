---
title: 《C++程序设计原理与实践》笔记 第13章 图形类
date: 2023-02-13 01:10:11 +0800
categories: [C/C++, PPP]
tags: [cpp, gui, fltk]
---
第12章介绍了如何使用一组简单的接口类创建图形。本章将介绍每个接口类的设计、使用和实现。

## 13.1 图形类概览
作者的GUI库提供的主要接口类：

| 类 | 描述 |
| --- | --- |
| `Color` | 用于设置线、文本及形状填充的颜色 |
| `Line_style` | 用于设置线型 |
| `Point` | 表示屏幕上和窗口内的位置 |
| `Line` | 线段，用两个`Point`（端点）定义 |
| `Open_polyline` | 相连的线段序列，用一系列`Point`定义 |
| `Closed_polyline` | 类似于`Open_polyline`，但有一条线段连接最后一个点和第一个点 |
| `Polygon` | 多边形，即所有线段均不相交的`Closed_polyline` |
| `Text` | 字符串 |
| `Lines` | 线段集合，用多个`Point`对定义 |
| `Rectangle` | 矩形，针对快速、方便地显示进行了优化 |
| `Circle` | 圆，用圆心和半径定义 |
| `Ellipse` | 椭圆，用圆心和两个半轴定义 |
| `Function` | 一元函数，绘制一个区间内的图像 |
| `Axis` | 带标签的坐标轴 |
| `Mark` | 用字符标记的一个点 |
| `Marks` | 一系列带标记的点 |
| `Marked_polyline` | 点带有标记的`Open_polyline` |
| `Image` | 图像文件的内容 |

第15章将介绍`Function`和`Axis`。第16章介绍主要的GUI接口类：

| 类 | 描述 |
| --- | --- |
| `Window` | 窗口，屏幕的一个区域，用来显示图形对象 |
| `Simple_indow` | 带有 "Next" 按钮的窗口 |
| `Button` | 按钮，窗口中的矩形构件，通常带标签，可以点击来执行对应的函数 |
| `In_box` | 输入框，窗口中的一个框，通常带标签，用户可以在其中输入文本 |
| `Out_box` | 输出框，窗口中的一个框，通常带标签，程序可以向其中输出字符串 |
| `Menu` | 菜单，`Button`的向量 |

源文件组织如12.4节所示。

除了图形类，GUI库还提供了一个用于保存`Shape`或`Widget`的容器类`Vector_ref`。

## 13.2 Point和Line
在任何图形系统中，**点**(point)都是最基本的部分。这里使用整数坐标(x, y)来定义点，如12.5节所述。`Point`就是一对`int`，定义在[Point.h](https://github.com/ZZy979/PPP-code/blob/main/GUI/Point.h)中：

```cpp
struct Point {
    int x, y;
    Point(int xx, int yy) :x(xx), y(yy) {}
    Point() :x(0), y(0) {}
};
```

[Graph.h](https://github.com/ZZy979/PPP-code/blob/main/GUI/Graph.h)定义了`Shape`和`Line`：

```cpp
class Shape {
    // ...
}

struct Line : Shape {
    Line(Point p1, Point p2);
};
```

其中，`: Shape`意味着“`Line`是一种`Shape`”。`Shape`称为`Line`的**基类**(base class)，将在第14章进行解释。

`Line`由两个`Point`定义。下面的程序创建并绘制了两条线：

[绘制线段](https://github.com/ZZy979/PPP-code/blob/main/ch13/draw_two_lines.cpp)

![绘制线段](/assets/images/ppp-note-ch13-graphics-classes/draw_two_lines.png)

`Line`的构造函数的实现非常简单：

```cpp
Line::Line(Point p1, Point p2) {    // construct a line from two points
    add(p1);    // add p1 to this shape
    add(p2);    // add p2 to this shape
}
```

即简单地“添加”了两个点。添加到哪里？`Line`是如何在窗口中绘制的？答案在`Shape`类中，我们将在第14章介绍，`Shape`能够保存一些点、绘制由点对构成的线，并提供了`add()`函数来添加一个点。

## 13.3 Lines
我们很少仅仅画一条线。对象通常由很多条线组成，例如多边形、路径、迷宫、网格、柱状图、数学函数数据图等。最简单的“复合图形”是`Lines`：

```cpp
struct Lines : Shape {                 // related lines
    void draw_lines() const;
    void add(Point p1, Point p2);      // add a line defined by two points
};
```

`Lines`对象就是一个线的集合，每条线由一对`Point`定义。例如，13.2节的例子中的两条线可以作为单个对象：

[绘制线段2](https://github.com/ZZy979/PPP-code/blob/main/ch13/draw_two_lines2.cpp)

![绘制线段2](/assets/images/ppp-note-ch13-graphics-classes/draw_two_lines2.png)

一组`Line`对象和一个`Lines`对象中的一组线的区别完全是我们看问题的视角不同。使用`Lines`，我们是想表达两条线是联系在一起的，必须一起处理。例如，我们使用单个命令就可以改变`Lines`对象中所有线的颜色。另一方面，我们可以为每个`Line`对象设置不同的颜色。一个更实际的例子是定义网格。网格由一些等间隔的水平线和垂直线组成，我们将网格视为一个整体，因此将这些线定义为一个`Lines`对象`grid`：

[绘制网格](https://github.com/ZZy979/PPP-code/blob/main/ch13/draw_grid.cpp)

注意这里使用`x_max()`和`y_max()`获得窗口的尺寸。

![绘制网格](/assets/images/ppp-note-ch13-graphics-classes/draw_grid.png)

`Lines`的成员函数`add()`用于添加一条线（由一对点定义）：

```cpp
void Lines::add(Point p1, Point p2) {
    Shape::add(p1);
    Shape::add(p2);
}
```

其中限定符`Shape::`是必需的，否则编译器会调用`Lines`的`add()`（非法）而不是`Shape`的`add()`。

`draw_lines()`函数绘制`add()`定义的线：

```cpp
void Lines::draw_lines() const {
    if (color().visibility())
        for (int i=1; i<number_of_points(); i+=2)
            fl_line(point(i-1).x,point(i-1).y,point(i).x,point(i).y);
}
```

即每次取两个点，并使用底层库(FLTK)的画线函数`fl_line()`来绘制两点之间的线。

`draw_lines()`是（在调用`win.wait_for_button()`之后）被GUI系统调用的。我们不需要检查点的数目是否为偶数，因为`Lines::add()`每次只能添加两个点。函数`number_of_points()`和`point()`定义在`Shape`类中（见14.2节）。成员函数`draw_lines()`不修改形状，因此将其定义为`const`。

`Lines`的默认构造函数创建一个空对象，即开始没有线，按需要逐步添加。另外，也可以定义一个接受初始化列表的构造函数：

```cpp
void Lines::Lines(initializer_list<pair<Point, Point>> lst) {
    for (auto p : lst)
        add(p.first, p.second);
}
```

其中，`auto`表示让编译器自动推断类型（这里是`pair<Point, Point>`），`first`和`second`是标准库类型`pair`的两个成员，标准库类型`initializer_list`表示初始化列表。从而可以以字面值的形式创建`Lines`对象：

```cpp
Lines x = {
    {Point(100, 100), Point(200, 100)},  // first line: horizontal
    {Point(150, 50), Point(150, 150)}    // second line: vertical
};
```

或者

```cpp
Lines x = {
    { {100, 100}, {200, 100} },  // first line: horizontal
    { {150, 50}, {150, 150} }    // second line: vertical
};
```

其中，`{100, 100}`表示一个`Point`，`{ {100, 100}, {200, 100} }`表示一个`pair<Point, Point>`，整个初始化列表表示一个`initializer_list<pair<Point, Point>>`。

## 13.4 Color
`Color`是用于表示颜色的类型。可以像这样使用：

```cpp
grid.set_color(Color::red);
```

![绘制红色网格](/assets/images/ppp-note-ch13-graphics-classes/draw_red_grid.png)

`Color`定义了颜色的表示方法(`Fl_Color`)，并给出了一个常用颜色的符号名字（`Color_type`枚举），见Graph.h。

注：FLTK使用`Fl_Color`类型（`unsigned int`的别名）表示颜色，即一个32位无符号整数0xrrggbbii，该整数有两种含义：
* 低8位ii表示[FLTK默认颜色表](https://www.fltk.org/doc-1.3/drawing.html#drawing_colors)中的索引，范围为0~255，例如99为<font color="#006D3F">暗绿色</font>
* 高24位rrggbb表示RGB颜色值，其中rr、gg和bb分别是红、绿、蓝分量，范围都是0~255，例如0x2B91AF00 = RGB(43, 145, 175) = <font color="#2B91AF">蓝绿色</font>

`Color`的目标是：
* 隐藏实现的颜色表示方式，即FLTK的`Fl_Color`类型
* 将`Color_type`映射到`Fl_Color`
* 给颜色常量一个作用域
* 提供一个简单的透明度机制（可见和不可见）

有几种方式选择颜色：
* 使用命名常量，例如`Color::dark_blue`
* 使用0~255之间的索引，例如`Color(99)`
* 使用RGB值，例如`Color(0x2B91AF00)`，参考：
    * [RGB color model - Wikipedia](https://en.wikipedia.org/wiki/RGB_color_model)
    * [RGB Color Codes Chart](https://www.rapidtables.com/web/color/RGB_Color.html)

`Color`的构造函数允许从`Color_type`或者普通的`int`创建`Color`对象，例如：

```cpp
Color red = Color::red;
Color green = 0x00FF0000;
Color blue = 4;
```

`Color`提供了`as_int()`函数，返回颜色对应的`int`值。

颜色的透明度/可见性用`Color::visible`和`Color::invisible`。例如，如果不想显示形状的轮廓，只显示填充颜色，可以将轮廓颜色设置为不可见：

```cpp
r.set_color(Color::invisible);
r.set_fill_color(Color::red);
```

## 13.5 Line_style
线性是描述线的外形的一种模式。可以像这样使用`Line_style`：

```cpp
grid.set_style(Line_style::dot);
```

![绘制红色点线网格](/assets/images/ppp-note-ch13-graphics-classes/draw_red_dotted_grid.png)


也可以调整线宽（粗细）。`Line_style`类型也定义在Graph.h。

定义`Line_style`所使用的编程技术与`Color`完全一样——隐藏了FLTK使用普通`int`表示线型的细节，因为这些细节可能会随着库的升级而发生变化。

大多数情况下，我们无需关心线型，使用默认值即可（默认宽度和实线）。`Line_style`包括两部分：样式（例如实线或虚线）和宽度（粗细）。宽度用整数表示，默认为0。例如，可以像这样设置加粗的虚线：

```cpp
grid.set_style(Line_style(Line_style::dash, 2));
```

![绘制红色加粗虚线网格](/assets/images/ppp-note-ch13-graphics-classes/draw_fat_dashed_red_grid.png)

注意，颜色和线型会对形状中的所有线起作用，这是将许多线组合为单个图形对象（例如`Lines`、`Open_polyline`或`Polygon`）的好处之一。如果想分别控制线的颜色或线型，必须将它们定义为独立的`Line`对象。例如：

```cpp
horizontal.set_color(Color::red);
vertical.set_color(Color::green);
```

![绘制有颜色的线段](/assets/images/ppp-note-ch13-graphics-classes/draw_two_lines_colored.png)

## 13.6 Open_polyline
`Open_polyline`是由一系列依次相连的线段组成的形状，由一系列点定义。 "Poly" 是希腊语中“许多”的意思， "polyline" 表示由许多线组成的形状。例如：

```cpp
Open_polyline opl = { {100, 100}, {150, 200}, {250, 250}, {300, 200} };
```

[绘制Open_polyline](https://github.com/ZZy979/PPP-code/blob/main/ch13/draw_open_polyline.cpp)

![绘制Open_polyline](/assets/images/ppp-note-ch13-graphics-classes/draw_open_polyline.png)

`Open_polyline`类的定义如下：

```cpp
struct Open_polyline : Shape {         // open sequence of lines
    Open_polyline() :Shape() {}
    Open_polyline(initializer_list<Point> points) :Shape(points) {}
    void add(Point p) { Shape::add(p); }
    void draw_lines() const override;
};
```

`Open_polyline`继承自`Shape`两个构造函数分别调用了`Shape`对应的构造函数，上面的程序使用了初始化列表构造函数。
* 注：书中给出的定义使用`using`声明(`using Shape::Shape;`)继承了`Shape`的构造函数，但`Shape`的构造函数是`protected`，通过`using`继承的构造函数也是`protected`，无法在程序中使用。

`Open_polyline`的`add()`函数是为了允许用户访问`Shape::add()`（本身是`protected`）。不必定义`draw_lines()`，因为`Shape`类的默认定义就是用线依次连接通过`add()`添加的点。

## 13.7 Closed_polyline
`Closed_polyline`与`Open_polyline`类似，唯一区别是还需要画一条从最后一个点到第一个点的线。例如：

[绘制Closed_polyline](https://github.com/ZZy979/PPP-code/blob/main/ch13/draw_closed_polyline.cpp)

![绘制Closed_polyline](/assets/images/ppp-note-ch13-graphics-classes/draw_closed_polyline.png)

`Closed_polyline`的定义为：

```cpp
struct Closed_polyline : Open_polyline { // closed sequence of lines
    using Open_polyline::Open_polyline;
    void draw_lines() const override;
};

void Closed_polyline::draw_lines() const {
    Open_polyline::draw_lines();    // first draw the "open poly line part"
    // then draw closing line:
    if (number_of_points()>2 && color().visibility())
        fl_line(point(number_of_points()-1).x, 
            point(number_of_points()-1).y,
            point(0).x,
            point(0).y);
}
```

`Closed_polyline`需要定义自己的`draw_lines()`来绘制连接最后一个点到第一个点的线。我们只需编写`Closed_polyline`与`Open_polyline`不同的部分即可：调用FLTK的画线函数`fl_line()`来绘制最后一条线，直接调用`Open_polyline::draw_lines()`来绘制其他的线。

## 13.8 Polygon
`Polygon`与`Closed_polyline`非常相似，唯一的区别是`Polygon`不允许交叉的线。例如，上一节中的`Closed_polyline`是一个多边形，但如果再添加一个点：`cpl.add(Point(100, 250));`，则不再是一个多边形：

![绘制Closed_polyline 2](/assets/images/ppp-note-ch13-graphics-classes/draw_closed_polyline2.png)


`Polygon`是不存在交叉线的`Closed_polyline`，因此可以让`Polygon`继承`Closed_polyline`，并在`add()`函数中检查是否有线段相交：

```cpp
struct Polygon : Closed_polyline {    // closed sequence of non-intersecting lines
    using Closed_polyline::Closed_polyline;
    void add(Point p);
    void draw_lines() const override;
};

void Polygon::add(Point p) {
    // check that the new line doesn't intersect existing lines (code not shown)
    Closed_polyline::add(p);
}

void Polygon::draw_lines() const {
    if (number_of_points() < 3) error("less than 3 points in a Polygon");
    Closed_polyline::draw_lines();
}
```

通过继承节省了大量工作，还避免了重复代码。不幸的是，每次调用`add()`都需要检查是否有线段相交，这导致一个低效的(O(N<sup>2</sup>))算法——定义一个具有N个点的`Polygon`需要做N*(N-1)/2次检查。因此，我们假设`Polygon`只用于顶点数较少的多边形。

例如：

[绘制多边形](https://github.com/ZZy979/PPP-code/blob/main/ch13/draw_polygon.cpp)

![绘制多边形](/assets/images/ppp-note-ch13-graphics-classes/draw_polygon.png)

`Polygon::add()`中省略的相交检查是整个GUI库中最复杂的部分。麻烦在于`Polygon`的不变式“这些点表示一个多边形”只有在定义了所有点之后才能被验证，因此无法在构造函数中建立不变式（虽然这是最好的方式）。

## 13.9 Rectangle
屏幕上最常见的形状是矩形。因此GUI系统直接支持矩形，而不是当作四个角恰好都是直角的多边形。

```cpp
struct Rectangle : Shape {
    Rectangle(Point xy, int ww, int hh);
    Rectangle(Point x, Point y);
    void draw_lines() const override;

    int height() const { return h; }
    int width() const { return w; }
private:
    int h;    // height
    int w;    // width
};
```

可以使用两个点（左上角和右下角）或者一个点（左上角）和宽度、高度来定义矩形。

[绘制矩形](https://github.com/ZZy979/PPP-code/blob/main/ch13/draw_rectangles.cpp)

![绘制矩形](/assets/images/ppp-note-ch13-graphics-classes/draw_rectangles.png)

当不设置填充颜色时，矩形是透明的，因此可以看到黄色矩形`rect00`的一角。

可以在窗口内移动形状：

```cpp
rect11.move(400, 0);    // to the right of rect21
rect11.set_fill_color(Color::white);
```

![绘制矩形2](/assets/images/ppp-note-ch13-graphics-classes/draw_rectangles2.png)

注意，白色矩形`rect11`位于窗口之外的部分被“剪裁”掉了。

另外请注意形状的层次：后绘制的形状会覆盖先绘制的形状。GUI库的`Window`类提供了一种重新排列形状次序的方法：`put_on_top()`将一个形状放在顶层（必须在`attach()`之后调用）。例如：

```cpp
win.put_on_top(rect00);
```

![绘制矩形3](/assets/images/ppp-note-ch13-graphics-classes/draw_rectangles3.png)

可以看到，即使矩形有填充颜色仍然有边框，可以将其移除：

```cpp
rect00.set_color(Color::invisible);
```

![绘制矩形4](/assets/images/ppp-note-ch13-graphics-classes/draw_rectangles4.png)

注意，在填充颜色和线的颜色都被设置为`invisible`后，`rect22`就看不到了。

`Rectangle`的`draw_lines()`必须处理线的颜色和填充颜色，因此有些复杂：

```cpp
void Rectangle::draw_lines() const {
    if (fill_color().visibility()) {    // fill
        fl_color(fill_color().as_int());
        fl_rectf(point(0).x,point(0).y,w,h);
        fl_color(color().as_int());    // reset color
    }

    if (color().visibility()) {    // lines on top of fill
        fl_color(color().as_int());
        fl_rect(point(0).x,point(0).y,w,h);
    }
}
```

FLTK提供了绘制矩形填充(`fl_rectf()`)和矩形轮廓(`fl_rect()`)的函数。默认情况下，我们两者都绘制（轮廓在上）。

## 13.10 管理未命名对象
到目前为止，所有图形对象都是命名的。当处理大量对象时，这种方法就不可行了。例如，绘制FLTK调色板中256中颜色构成的颜色表，即绘制256个不同颜色填充的格子，构成一个16×16的矩阵，如下图所示。

![绘制16×16颜色表](/assets/images/ppp-note-ch13-graphics-classes/draw_color_matrix.png)

命名256个格子不但繁琐，而且不明智。任何一个格子都可以用坐标(i, j)来标识，左上角的格子是(0, 0)。因此我们需要一种表示对象矩阵的方法。无法使用`vector<Rectangle>`，因为`Shape`类不可拷贝；使用`vector<Rectangle*>`则需要手动`delete`。本例的解决方案：采用一种能够保存命名和未命名对象的向量类型：

```cpp
template<class T> class Vector_ref {
public:
    // ...
    void push_back(T& s);  // add a named object
    void push_back(T* p);  // add an unnamed object

    T& operator[](int i);  // subscripting: read and write access
    const T& operator[](int i) const;

    int size() const;
};
```

与标准库`vector`的使用方法非常类似：

```cpp
Vector_ref<Rectangle> rect;

Rectangle x(Point(100, 200), Point(200, 300));
rect.push_back(x);  // add named

rect.push_back(new Rectangle(Point(50, 60), Point(80, 90)));  // add unnamed

for (int i = 0; i < rect.size(); ++i)
    rect[i].move(10, 10);  // use rect
```

第17章将解释`new`运算符。`Vector_ref`的实现在Graph.h，现在只知道可以用它保存未命名对象就够了。

可以这样绘制颜色表：

[绘制16×16颜色表](https://github.com/ZZy979/PPP-code/blob/main/ch13/draw_color_matrix.cpp)

## 13.11 Text
`Text`用于显示文本。例如，为13.8节中“奇怪”的`Closed_polyline`添加标签：

```cpp
Text t(Point(200, 200), "A closed polyline that isn't a polygon");
t.set_color(Color::blue);
```

![显示文本](/assets/images/ppp-note-ch13-graphics-classes/draw_closed_polyline_with_text.png)

`Text`对象定义了以给定的`Point`为左下角的一行文本。限制文本为单行的原因是保证跨系统的可移植性。不要尝试放入换行符，在窗口中不一定有效果（经过测试，在Windows系统上确实无效）。字符串流对于构造`Text`中显示的字符串是很有用的。

`Text`的定义如下：

```cpp
struct Text : Shape {
    // the point is the bottom left of the first letter
    Text(Point x, const string& s) : lab(s), fnt(fl_font()), fnt_sz(fl_size()) { add(x); }

    void draw_lines() const override;

    void set_label(const string& s) { lab = s; }
    string label() const { return lab; }

    void set_font(Font f) { fnt = f; }
    Font font() const { return Font(fnt); }

    void set_font_size(int s) { fnt_sz = s; }
    int font_size() const { return fnt_sz; }
private:
    string lab;    // label
    Font fnt;
    int fnt_sz;
};

void Text::draw_lines() const {
    fl_font(fnt.as_int(),fnt_sz);
    fl_draw(lab.c_str(),point(0).x,point(0).y);
}
```

字符的颜色和形状颜色一样，可以通过`set_color()`设置。Graph.h中的`Font`类提供了一些预定义的字体。

## 13.12 Circle
`Circle`是由圆心和半径定义的：

```cpp
struct Circle : Shape {
    Circle(Point p, int rr);    // center and radius

    void draw_lines() const override;
    Point center() const;
    void set_radius(int rr) {
        set_point(0,Point(center().x-rr,center().y-rr));
        r=rr;
    }
    int radius() const { return r; }
private:
    int r;
};

Circle::Circle(Point p, int rr)    // center and radius
    :r(rr) {
    add(Point(p.x-r,p.y-r));       // store top-left corner
}

Point Circle::center() const {
    return Point(point(0).x+r, point(0).y+r);
}

void Circle::draw_lines() const {
  	if (fill_color().visibility()) {	// fill
		fl_color(fill_color().as_int());
		fl_pie(point(0).x,point(0).y,r+r-1,r+r-1,0,360);
		fl_color(color().as_int());	// reset color
	}
	if (color().visibility()) {
		fl_color(color().as_int());
		fl_arc(point(0).x,point(0).y,r+r,r+r,0,360);
	}
}
```

可以像这样使用`Circle`：

[绘制圆](https://github.com/ZZy979/PPP-code/blob/main/ch13/draw_circles.cpp)

![绘制圆](/assets/images/ppp-note-ch13-graphics-classes/draw_circles.png)

`Circle`类实现的奇怪之处是它存储的点不是圆心，而是外接正方形的左上角，因为FLTK的画圆函数`fl_arc()`使用这个点。`Circle`提供了一个例子：对于一个概念，一个类如何呈现与其实现不同的（可能更好的）视角。

`fl_arc()`函数用于绘制椭圆的弧，其中前两个参数表示椭圆外接矩形的左上角，之后两个参数是矩形的宽和高（即椭圆的长轴和短轴），最后两个参数是绘制的起止角度(0~360)。

## 13.13 Ellipse
`Ellipse`和`Circle`类似，但通过圆心、半长轴和半短轴定义：

```cpp
struct Ellipse : Shape {
    Ellipse(Point p, int ww, int hh);   // center, min, and max distance from center

    void draw_lines() const override;

    Point center() const;
    Point focus1() const;
    Point focus2() const;
    
    void set_major(int ww) { set_point(0,Point(center().x-ww,center().y-h)); w=ww; }
    int major() const { return w; }
    void set_minor(int hh) { set_point(0,Point(center().x-w,center().y-hh)); h=hh; }
    int minor() const { return h; }
private:
    int w;
    int h;
};

void Ellipse::draw_lines() const {
   if (fill_color().visibility()) {	// fill
		fl_color(fill_color().as_int());
		fl_pie(point(0).x,point(0).y,w+w-1,h+h-1,0,360);
		fl_color(color().as_int());	// reset color
	}
	if (color().visibility()) {
		fl_color(color().as_int());
		fl_arc(point(0).x,point(0).y,w+w,h+h,0,360);
	}
}
```

可以像这样使用`Ellipse`：

[绘制椭圆](https://github.com/ZZy979/PPP-code/blob/main/ch13/draw_ellipses.cpp)

![绘制椭圆](/assets/images/ppp-note-ch13-graphics-classes/draw_ellipses.png)

在几何上，椭圆的长轴与短轴相等时看起来就是一个圆。GUI库没有把`Circle`定义为`Ellipse`的子类，因为这样会增加一个成员，带来不必要的空间开销。但主要原因是必须`set_major()`和`set_minor()`，使类的定义变得更加复杂（这和`Rectangle`不是`Polygon`的子类的原因是类似的）。

在设计类时，我们应该小心不要自作聪明，也不要被“直觉”欺骗。相反，我们应该注意**如何用类表达某些概念**，而不仅仅是数据和函数成员的集合。不思考要表达的思想/概念，只是将代码简单地堆积在一起会导致难以解释、难以调试、难以维护的代码。

## 13.14 Marked_polyline
我们通常需要对图中的点做“标记”。`Marked_polyline`就是点带有“标记”的`Open_polyline`。例如：

[绘制Marked_polyline](https://github.com/ZZy979/PPP-code/blob/main/ch13/draw_marked_polyline.cpp)

![绘制Marked_polyline](/assets/images/ppp-note-ch13-graphics-classes/draw_marked_polyline.png)

`Marked_polyline`的定义为：

```cpp
struct Marked_polyline : Open_polyline {
    Marked_polyline(const string& m) :mark(m) { if (m=="") mark = "*"; }
    Marked_polyline(const string& m, initializer_list<Point> points)
        :Open_polyline(points), mark(m) {
        if (m=="") mark = "*";
    }
    void draw_lines() const override;
private:
    string mark;
};

void Marked_polyline::draw_lines() const {
    Open_polyline::draw_lines();
    for (int i=0; i<number_of_points(); ++i) 
        draw_mark(point(i),mark[i%mark.size()]);
}
```

通过继承`Open_polyline`，我们“免费”获得了对点的处理，因此只需处理标记。`Marked_polyline::draw_lines()`首先调用`Open_polyline::draw_lines()`画线，之后依次选择字符串中的字符绘制标记：`mark[i%mark.size()]`通过取模运算循环遍历字符串`mark`，选择下一个标记字符。绘制标记字符使用了辅助函数`draw_mark()`：

```cpp
void draw_mark(Point xy, char c) {
    static const int dx = 4;
    static const int dy = 4;

    string m(1,c);
    fl_draw(m.c_str(),xy.x-dx,xy.y+dy);
}
```

其中常量`dx`和`dy`用于使字符位居中，字符串`m`被初始化为单个字符`c`。

## 13.15 Marks
有时，我们需要显示没有线连接的标记，因此提供了`Marks`类。例如：

[绘制Marks](https://github.com/ZZy979/PPP-code/blob/main/ch13/draw_marks.cpp)

![绘制Marks](/assets/images/ppp-note-ch13-graphics-classes/draw_marks.png)

`Marks`就是线的颜色是`invisible`的`Marked_polyline`：

```cpp
struct Marks : Marked_polyline {
    Marks(const string& m) :Marked_polyline(m) {
        set_color(Color(Color::invisible));
    }
    Marks(const string& m, initializer_list<Point> points) :Marked_polyline(m, points) {
        set_color(Color(Color::invisible));
    }
};
```

`:Marked_polyline(m)`表示调用基类的构造函数。这种语法是成员初始化语法的一个变体。

## 13.16 Mark
`Mark`用于标记单个点，由一个点和一个字符初始化。例如：

[绘制标记圆心的圆](https://github.com/ZZy979/PPP-code/blob/main/ch13/draw_circles.cpp)

![绘制标记圆心的圆](/assets/images/ppp-note-ch13-graphics-classes/draw_circles_with_centers_marked.png)

`Mark`就是直接给定一个点和字符的`Marks`：

```cpp
struct Mark : Marks {
    Mark(Point xy, char c) : Marks(string(1,c)) {
        add(xy);
    }
};
```

`string(1, c)`是`string`的一个构造函数，将字符串初始化为仅包含单个字符`c`。

## 13.17 Image
我们希望在程序中显示图像。例如，下面的程序显示了飓风Rita到达得克萨斯州墨西哥湾的路线图的一部分，并加入从太空中拍摄的Rita的照片：

[绘制图像](https://github.com/ZZy979/PPP-code/blob/main/ch13/draw_images.cpp)

![绘制图像](/assets/images/ppp-note-ch13-graphics-classes/draw_images.png)

`set_mask()`选择要显示图像的一个子图像。这里我们从图像rita_path.gif（加载到`path`）选择了一个600×400像素大小、左上角位于`path`中的(50, 250)的子图像。

形状按照附加到窗口的顺序确定层次。由于`path`先于`rita`附加到窗口，因此位于`rita`下层。

图像的编码格式非常多，GUI库只处理最常用的两种，JPEG和GIF：

```cpp
Suffix get_encoding(const string& s);
```

在GUI库中，使用`Image`类的对象表示内存中的图像：

```cpp
struct Image : Shape {
    Image(Point xy, string file_name, Suffix e = Suffix::none);
    ~Image() { delete p; }
    void draw_lines() const override;
    void set_mask(Point xy, int ww, int hh) { w=ww; h=hh; cx=xy.x; cy=xy.y; }
private:
    int w,h;  // define "masking box" within image relative to position (cx,cy)
    int cx,cy; 
    Fl_Image* p;
    Text fn;
};
```

`Image`的构造函数使用给定的文件名打开文件，然后按参数或文件后缀指定的编码格式创建图像。如果图像无法显示（例如未找到文件）则显示`Bad_image` (☒)。

## 简单练习
[魔塔](https://github.com/ZZy979/PPP-code/blob/main/ch13/magic_tower.cpp)

## 习题
* [13-1](https://github.com/ZZy979/PPP-code/blob/main/ch13/exec13-1.cpp)
* [13-2](https://github.com/ZZy979/PPP-code/blob/main/ch13/exec13-2.cpp)
* [13-3](https://github.com/ZZy979/PPP-code/blob/main/ch13/exec13-3.cpp)
* [13-4](https://github.com/ZZy979/PPP-code/blob/main/ch13/exec13-4.cpp)
* [13-5](https://github.com/ZZy979/PPP-code/blob/main/ch13/exec13-5.cpp)
* [13-6](https://github.com/ZZy979/PPP-code/blob/main/ch13/exec13-6.cpp)
* [13-8~13-10](https://github.com/ZZy979/PPP-code/blob/main/ch13/exec13-9.cpp)
* [13-11](https://github.com/ZZy979/PPP-code/blob/main/ch13/exec13-11.cpp)
* [13-12](https://github.com/ZZy979/PPP-code/blob/main/ch13/exec13-12.cpp)
* [13-13](https://github.com/ZZy979/PPP-code/blob/main/ch13/draw_color_matrix.cpp)
