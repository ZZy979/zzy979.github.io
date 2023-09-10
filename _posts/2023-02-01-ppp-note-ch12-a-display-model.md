---
title: 《C++程序设计原理与实践》笔记 第12章 一个显示模型
date: 2023-02-01 00:33:33 +0800
categories: [C/C++, PPP]
tags: [cpp, gui, fltk]
---
本章介绍了一个显示模型(display model)（GUI的输出部分），并给出了使用示例和基本概念，例如屏幕坐标、线和颜色等。

## 12.1 为什么需要图形？
我们为什么用四章的篇幅介绍图形以及一章介绍GUI？

* 图形很有用。例如，科学计算、数据分析需要数据图形化。
* 图形很有趣。一段代码的效果能够立刻呈现出来。
* 图形提供了许多有趣的代码来阅读。学习编程的一部分是阅读大量代码，寻找好代码的感觉。
* 图形的设计实例非常丰富。图形领域有非常丰富的设计决策和设计技术的具体、实际的例子。
* 图形有利于解释**面向对象程序设计**(object-oriented programming)的概念和语言特性。图形提供了一些非常易于理解的面向对象设计实例。
* 一些关键的图形学概念并不是简单直白的。

## 12.2 一个显示模型
iostream库是面向字符的输入/输出流。本章和之后的四章介绍另外一种技术：**图形**(graphics)和**图形用户界面**(graphical user interface, GUI)，可以直接显示在计算机屏幕上。图形包括点、线、矩形、圆等**形状**(shape)，GUI包括窗口、按钮、输入框等**构件**(widget)。

基本模型如下：我们利用图形库提供的基本图形（例如线）组合出更复杂的图形（例如多边形）；之后将这些图形对象**附加**(attach)到一个表示物理屏幕的窗口对象；最后，显示引擎（图形库/GUI库）在屏幕上绘制这些图形。

![显示模型](/assets/images/ppp-note-ch12-a-display-model/显示模型.png)

## 12.3 第一个例子
下面的程序在屏幕上绘制一个三角形：

[绘制三角形](https://github.com/ZZy979/PPP-code/blob/main/ch12/draw_triangle.cpp)

运行结果如下：

![draw_triangle](/assets/images/ppp-note-ch12-a-display-model/draw_triangle.png)

下面逐行分析这个程序。首先包含（作者提供的）图形库头文件：

```cpp
#include "Graph.h"          // get access to our graphics library facilities
#include "Simple_window.h"  // get access to our window library
```

接着，告诉编译器图形相关的类在命名空间`Graph_lib`中：

```cpp
using namespace Graph_lib;  // our graphics facilities are in Graph_lib
```

然后，在`main()`中创建一个窗口`win`：

```cpp
Point tl(100, 100);             // to become top left corner of window
Simple_window win(tl, 600, 400, "Canvas");  // make a simple window
```

其中，`tl`表示窗口左上角在屏幕上的坐标，宽度和高度分别为600和400像素，字符串 "Canvas" 将显示在窗口左上角的标题栏。

接下来，创建一个多边形对象`poly`，并添加三个点，从而得到一个三角形：

```cpp
Graph_lib::Polygon poly;        // make a shape (a polygon)

poly.add(Point(300, 200));      // add a point
poly.add(Point(350, 100));      // add another point
poly.add(Point(400, 200));      // add a third point
```

这里的点表示的是窗口内的坐标。

之后将三角形的边设置为红色（纯粹是为了展示）：

```cpp
poly.set_color(Color::red);     // adjust properties of poly
```

最后，将`poly`附加到窗口`win`：

```cpp
win.attach(poly);               // connect poly to the window
```

只有这样图形才会被绘制出来。程序的最后一行

```cpp
win.wait_for_button();          // give control to the display engine
```

`wait_for_button()`将控制权交给GUI系统，并告诉GUI系统将窗口显示出来（同时绘制附加到该窗口的图形）。之后，将会等待用户点击（`Simple_window`自带的） "Next" 按钮。当点击该按钮后，程序将终止并关闭窗口。

操作系统会为每个窗口添加标题栏/窗口框(frame)，因此你“免费”得到了右上角的三个按钮。

## 12.4 使用GUI库
要编写GUI程序，必须使用GUI库（用于调用操作系统的底层接口来绘制窗口和图形）。然而，C++并没有提供标准GUI库，因此作者从很多可用的[C++ GUI库](https://www.stroustrup.com/C++.html#GUI)中选择了一个。

本书使用的GUI库叫做FLTK（Fast Light Toolkit，读作 "fulltick" ），可以从[fltk.org](https://www.fltk.org/)下载。我们的代码可以移植到任何使用FLTK的平台（Windows、Linux、macOS等）。

注：书中的代码不是直接使用FLTK，而是使用了作者提供的GUI库。这个GUI库是在FLTK的基础上又封装了一层，提供了形状和GUI构件的接口类（如无特殊说明，后面提到“GUI库”就是指作者提供的这个GUI库）。

GUI库的源代码可以从[作者网站](https://www.stroustrup.com/programming_support.html)上下载：<https://www.stroustrup.com/Programming/code.tar>

注：代码中存在几处编译错误，需要如下修改
* Window.h：开头增加两个`#include`和两个`using`声明：

```cpp
#include <string>
#include <vector>
// ...

using std::string;
using std::vector;
// ...
```

* Simple_window.h：`struct Simple_window : Window`改为`struct Simple_window : Graph_lib::Window`
* Graph.h：`Vector_ref(const Vector<T>&);`和`Vector_ref& operator=(const Vector<T>&);`中的`Vector`改为`Vector_ref`

该GUI库由5个头文件和4个源文件组成：
* 头文件
    * Point.h
    * Window.h
    * Simple_window.h
    * Graph.h
    * GUI.h
* 源文件
    * Window.cpp
    * Simple_window.cpp
    * Graph.cpp
    * GUI.cpp

头文件之间的包含关系如下：

![GUI库头文件](/assets/images/ppp-note-ch12-a-display-model/GUI库头文件.png)

其中，Window.h、Simple_window.h和GUI.h提供了`Window`、`Simple_window`和`Button`等GUI构件，类图如下：

![GUI库窗口和构件类](/assets/images/ppp-note-ch12-a-display-model/GUI库窗口和构件类.png)

Graph.h提供了形状类`Shape`以及`Line`、`Rectangle`、`Circle`等子类，类图如下：

![GUI库图形类](/assets/images/ppp-note-ch12-a-display-model/GUI库图形类.png)

## 12.8 让程序运行起来
注：和第6章一样，本章中作者又是到最后一节才讲如何让程序运行起来（正文中也没有详细讲，只提到了附录D，而附录D也只介绍了使用Visual Studio一种方法）。实际上对于初学者来说这是最困难的部分。在不同操作系统上、使用不同构建工具的构建方式也各不相同，但基本上包括以下步骤：
* 编译FLTK源代码，生成FLTK库文件；
* 编译GUI库源代码，生成GUI库文件；
* 编译我们自己的源代码，并链接到GUI库和FLTK库，生成可执行文件。

下面详细介绍使用Make和CMake两种构建方法。

### 12.8.1 使用Make
#### 12.8.1.1 安装FLTK
本节在Ubuntu系统上使用Make命令编译并安装FLTK库。

首先从[FLTK官方网站](https://www.fltk.org/software.php)下载源代码（例如fltk-1.3.8-source.tar.gz），解压后执行以下命令：

```bash
cd fltk-1.3.8/
./configure --prefix=/usr/local
make
sudo make install
sudo ldconfig
```

其中，`configure`命令用于生成构建配置文件，`--prefix`选项指定安装路径，头文件将被拷贝到\$prefix/include目录，库文件将被拷贝到\$prefix/lib目录。如果报错 "Configure could not find required X11 libraries, aborting" 则需要先安装X11库：

```bash
sudo apt-get install libx11-dev
```

`make`命令将源代码编译为库文件，生成在fltk-1.3.8/lib目录；`make install`命令将头文件以及编译生成的库文件和可执行文件等拷贝到`--prefix`选项指定的目录下（这里是/usr/local）：

```bash
$ cd /usr/local
$ ls bin/
fltk-config  fluid  ...

$ ls include/
FL  ...

$ ls lib/
lib/libfltk.a        lib/libfltk_images.a  lib/libfltk_png.a
lib/libfltk_forms.a  lib/libfltk_jpeg.a    lib/libfltk_z.a
...
```

其中，fltk-config是一个辅助程序，可以自动生成编译和链接选项：

```bash
$ fltk-config --cxx
g++

$ fltk-config --cxxflags
-I/usr/local/include -I/usr/local/include/FL/images -D_LARGEFILE_SOURCE -D_LARGEFILE64_SOURCE -D_THREAD_SAFE -D_REENTRANT

$ fltk-config --ldflags
-L/usr/local/lib -lfltk -lpthread -ldl -lm -lX11
```

例如，下面是FLTK官方文档[Writing Your First FLTK Program](https://www.fltk.org/doc-1.3/basics.html)中的Hello World程序：

```cpp
#include <FL/Fl.H>
#include <FL/Fl_Box.H>
#include <FL/Fl_Window.H>

int main(int argc, char* argv[]) {
    Fl_Window* window = new Fl_Window(340, 180);
    Fl_Box* box = new Fl_Box(20, 40, 300, 100, "Hello, World!");
    box->box(FL_UP_BOX);
    box->labelfont(FL_BOLD_ITALIC);
    box->labelsize(36);
    box->labeltype(FL_SHADOW_LABEL);
    window->end();
    window->show(argc, argv);
    return Fl::run();
}
```

可以使用以下命令编译该程序：

```bash
g++ `fltk-config --cxxflags` -o hello_fltk hello_fltk.cpp `fltk-config --ldflags`
```
该命令等价于

```bash
g++ -I/usr/local/include -I/usr/local/include/FL/images -D_LARGEFILE_SOURCE -D_LARGEFILE64_SOURCE -D_THREAD_SAFE -D_REENTRANT -o hello_fltk hello_fltk.cpp -L/usr/local/lib -lfltk -lpthread -ldl -lm -lX11
```

运行结果如下：

![hello_fltk](/assets/images/ppp-note-ch12-a-display-model/hello_fltk_ubuntu.png)

另外，fltk-config还提供了一个更简便的命令：

```bash
fltk-config --compile hello_fltk.cpp
```

该命令自动将编译选项和链接选项组合成一个编译命令并执行，和上面的编译命令也是等价的。

注：由于/usr/local/include和/usr/local/lib在默认的头文件和库文件搜索路径列表中，因此可以省略编译命令中的`-I`和`-L`选项，从而简化为

```bash
g++ -o hello_fltk hello_fltk.cpp -lfltk -lpthread -ldl -lm -lX11
```

要卸载FLTK，在fltk-1.3.8目录下执行

```bash
sudo make uninstall
```

参考：
* FLTK官方文档 [Building and Installing FLTK Under UNIX and Apple OS X](https://www.fltk.org/doc-1.3/intro.html#intro_unix)

#### 12.8.1.2 编译GUI库
安装好FLTK后，下面将作者提供的GUI库源代码编译为库文件。

首先下载源代码[code.tar](https://www.stroustrup.com/Programming/code.tar)，解压出GUI目录，并按照12.4节的方式修正编译错误。由于作者提供的源代码中已经包含了Makefile，因此直接在GUI目录下执行`make`命令，即可构建出库文件libbookgui.a，与源代码在同一个目录下。

#### 12.8.1.3 编译自己的程序代码
假设12.3节中的程序保存在ch12/draw_triangle.cpp，目录ch12与GUI在同级目录下：

```
PPP-code/
    GUI/
        Window.h
        Graph.h
        ...
        libbookgui.a
    ch12/
        draw_triangle.cpp
```

则在ch12目录中执行以下命令：

```bash
g++ -I../GUI -o draw_triangle draw_triangle.cpp -L../GUI -lbookgui `fltk-config --ldflags --use-images`
```

即可得到可执行文件draw_triangle。其中，`-I../GUI`选项告诉编译器GUI头文件的搜索路径，从而`#include "Graph.h"`能够找到GUI目录下的Graph.h；`-L../GUI`选项告诉链接器GUI库文件的搜索路径，从而`-lbookgui`能够找到GUI目录下的libbookgui.a；`fltk-config --ldflags --use-images`将自动生成需要链接的FLTK库。

运行结果如下：

![draw_triangle](/assets/images/ppp-note-ch12-a-display-model/draw_triangle_ubuntu.png)

### 12.8.2 使用CMake（推荐）
[CMake](https://cmake.org/)是一个开源的、跨平台的C++构建工具，新版本的FLTK已经支持CMake构建。与Make相比，使用CMake构建具有以下优点：
* 简单，不需要手动编写复杂的编译命令
* 不需要手动安装FLTK，使用CMake的`FetchContent`模块可以自动下载并构建FLTK库
* CMake是跨平台的，因此在Windows、Linux和macOS系统上都适用
* 不仅可以在命令行使用，任何支持CMake的IDE（例如CLion、Visual Studio等）都适用

CMake的安装可参考[CMake构建工具使用教程]({% post_url 2023-02-21-cmake-tutorial %})。CMake使用名为CMakeLists.txt的文件声明构建目标。在项目根目录、GUI目录和ch12目录下分别创建一个CMakeLists.txt文件：

```
PPP-code/
    CMakeLists.txt
    GUI/
        CMakeLists.txt
        Window.h
        Graph.h
        ...
    ch12/
        CMakeLists.txt
        draw_triangle.cpp
```

根目录下的CMakeLists.txt内容如下：

```cmake
cmake_minimum_required(VERSION 3.20)
project(PPP_code)

set(CMAKE_CXX_STANDARD 14)

include(FetchContent)
set(FLTK_BUILD_TEST OFF CACHE BOOL " " FORCE)
FetchContent_Declare(
  fltk
  GIT_REPOSITORY https://github.com/fltk/fltk.git
  GIT_TAG release-1.3.8
)
FetchContent_MakeAvailable(fltk)

add_subdirectory(GUI)
add_subdirectory(ch12)
```

其中，`FetchContent_Declare()`命令声明了fltk库及其Git仓库地址，`FetchContent_MakeAvailable()`将自动下载fltk库的源代码。

GUI/CMakeLists.txt内容如下：

```cmake
file(GLOB SOURCES RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} *.cpp)
add_library(GUI ${SOURCES})
target_link_libraries(GUI fltk fltk_images)
target_include_directories(GUI PUBLIC ${fltk_SOURCE_DIR} ${fltk_BINARY_DIR})
target_include_directories(GUI INTERFACE ${CMAKE_CURRENT_SOURCE_DIR})
```

这里声明了一个库文件GUI，并链接到fltk和fltk_images库。

ch12/CMakeLists.txt内容如下：

```cmake
add_executable(draw_triangle draw_triangle.cpp)
target_link_libraries(draw_triangle GUI)
```

这里声明了一个可执行文件draw_triangle，并链接到GUI库（CMake会自动处理传递依赖，并将其链接到fltk库）。

在项目根目录下执行以下命令：

```bash
cmake -G "Unix Makefiles" -B cmake-build
cmake --build cmake-build --target draw_triangle
```

初次构建时CMake将自动下载并构建FLTK库，构建完成后即可得到可执行文件cmake-build/ch12/draw_triangle，运行可得到与上面同样的效果。

注：这里给出的方法就是我自己的代码仓库[PPP-code](https://github.com/ZZy979/PPP-code)使用的方法。

参考：
* [README.CMake.txt](https://github.com/fltk/fltk/blob/master/README.CMake.txt)
* [Using CMake to build an FLTK application](https://www.fltk.org/articles.php?L834+I80+T+P1+Q)
* [Provide easier instructions for modern CMake](https://www.fltk.org/newsgroups.php?gfltk.issues+v:1627)

### 12.8.3 使用Visual Studio
参考：
* FLTK官方文档 [Building FLTK Under Microsoft Windows](https://www.fltk.org/doc-1.3/intro.html#intro_windows)
* [Install FLTK for use with Visual C++](https://alf-p-steinbach.github.io/Install-FLTK-for-use-with-Visual-C-/)

## 12.5 坐标系
计算机屏幕是由**像素**(pixel)组成的矩形区域。像素是一个可以设置为某种颜色的点。在程序中最常见的建模屏幕的方式是由像素组成的矩形，每个像素由x（水平）坐标和y（垂直）坐标确定，屏幕左上角是坐标原点，如下图所示。

![屏幕坐标系](/assets/images/ppp-note-ch12-a-display-model/屏幕坐标系.png)

与数学上的坐标系不同的是，y坐标是向下增长的。

在GUI程序中，**窗口**(window)是从屏幕中划分出的由程序控制的矩形区域。可以将窗口看作一个小屏幕（窗口内的坐标系建模方式屏幕相同）。例如：

```cpp
Simple_window win(tl, 600, 400, "Canvas");
```

创建了一个宽600像素、高400像素的窗口（不包括标题栏），从而窗口内的x坐标从左到右为0\~599，y坐标从上到下为0\~399。能够进行绘制的窗口区域通常称为**画布**(canvas)。

## 12.6 形状
GUI库的头文件Graph.h定义了一组形状类，见12.4节。

## 12.7 使用形状类
本节介绍GUI库的一些基本特性：`Simple_window`, `Window`, `Shape`, `Text`, `Polygon`, `Line`, `Lines`, `Rectangle`, `Color`, `Line_style`, `Point`, `Axis`，目的是让你对这些特性有一个全面的了解，而不是详细理解某个类。下一章将会介绍每个类的设计。

### 12.7.1 图形头文件和main函数
首先需要包含GUI库的头文件：

```cpp
#include "Graph.h"
#include "Window.h"
```

或者

```cpp
#include "Graph.h"
#include "Simple_window.h"
```

Window.h包含于窗口有关的特性，Graph.h包含在窗口上绘制形状（包括文本）有关的特性。这些特性都定义在`Graph_lib`命名空间中。为了简单起见，使用`using`指令使得程序可以直接使用`Graph_lib`中的名字：

```cpp
using namespace Graph_lib;
```

### 12.7.2 一个几乎空白的窗口

```cpp
Point tl(100, 100);     // top left corner of our window
Simple_window win(tl, 600, 400, "Canvas");
// screen coordinate tl for top left corner
// window size(600*400)
// title: Canvas
win.wait_for_button();  // display!
```

这段代码创建了一个`Simple_window`，即一个带有 "Next" 按钮的窗口，左上角位于(100, 100)，宽度和高度分别为600和400，标题为 "Canvas" 。通过调用`win.wait_for_button()`将控制权交给GUI系统，从而将窗口绘制在屏幕上，如下图所示。

![几乎空白的窗口](/assets/images/ppp-note-ch12-a-display-model/almost_blank_window.png)

### 12.7.3 坐标轴

```cpp
Axis xa(Axis::x, Point(20, 300), 280, 10, "x axis");    // make an Axis
win.attach(xa);     // attach xa to the window, win
Axis ya(Axis::y, Point(20, 300), 280, 10, "y axis");
ya.set_color(Color::cyan);              // choose a color
ya.label.set_color(Color::dark_red);    // choose a color for the text
win.attach(ya);
```

这段代码创建了一个x轴(`Axis::x`)和一个y轴(`Axis::y`)，原点都在(20, 300)（窗口坐标系），长度为280，有10个刻度。通过调用`win.attach()`将坐标轴对象附加到窗口，从而能够被绘制出来。

![坐标轴](/assets/images/ppp-note-ch12-a-display-model/axis.png)

### 12.7.4 绘制函数图像
下面绘制一个正弦函数图像：

```cpp
Function sine(sin, 0, 100, Point(20, 150), 1000, 50, 50);   // sine curve
win.attach(sine);
```

其中，名为`sine`的`Function`对象使用标准库函数`sin()`生成的函数值绘制正弦曲线，自变量范围为[0, 100)，以(20, 150)为“原点”，共绘制1000个点，x和y坐标均放缩50倍。

![函数](/assets/images/ppp-note-ch12-a-display-model/function.png)

注意，超出窗口矩形区域的点将被GUI系统忽略。

### 12.7.5 多边形
多边形`Polygon`描述为一系列点，通过线连接起来：

```cpp
Polygon poly;
poly.add(Point(300, 200));      // three points make a triangle
poly.add(Point(350, 100));
poly.add(Point(400, 200));
poly.set_color(Color::red);
poly.set_style(Line_style::dash);
win.attach(poly);
```

与12.3节的例子一样，我们添加了一个三角形，作为多边形的例子，并设置了颜色和线型。所有形状都有线型，默认为实线(`Line_style::solid`)，这里设置为虚线(`Line_style::dash`)。

![多边形](/assets/images/ppp-note-ch12-a-display-model/polygon.png)

### 12.7.6 矩形
矩形是最容易处理的形状。例如，矩形易于描述（左上角、宽和高，或者左上角和右下角，等等），易于判断一个点是否在矩形内，以及易于用硬件快速绘制由像素构成的矩形。

与其他封闭形状相比，大多数图形库能够更好地处理矩形。因此，我们提供了一个独立于`Polygon`类的`Rectangle`类。`Rectangle`由左上角、宽和高来描述：

```cpp
Graph_lib::Rectangle r(Point(200, 200), 100, 50);   // top left corner, width, height
win.attach(r);
```

![矩形](/assets/images/ppp-note-ch12-a-display-model/rectangle.png)

可以创建一个看起来像矩形的`Closed_polyline`：

```cpp
Closed_polyline poly_rect;
poly_rect.add(Point(100, 50));
poly_rect.add(Point(200, 50));
poly_rect.add(Point(200, 100));
poly_rect.add(Point(100, 100));
win.attach(poly_rect);
```

![封闭多边形](/assets/images/ppp-note-ch12-a-display-model/closed_polyline.png)

尽管`poly_rect`的绘制结果是一个矩形，但在内存中它并不是一个`Rectangle`对象。最简单的证明方法是再添加一个点：

```cpp
poly_rect.add(Point(50, 75));
```

此时它的绘制结果就不再是一个矩形：

![封闭多边形2](/assets/images/ppp-note-ch12-a-display-model/closed_polyline2.png)

而`Rectangle`不是碰巧看起来像一个矩形，它从根本上保证是一个矩形。

### 12.7.7 填充
前面绘制的形状都是轮廓，也可以填充颜色：

```cpp
r.set_fill_color(Color::yellow);    // color the inside of the rectangle
poly.set_style(Line_style(Line_style::dash, 4));
poly_rect.set_style(Line_style(Line_style::dash, 2));
poly_rect.set_fill_color(Color::green);
```

![填充](/assets/images/ppp-note-ch12-a-display-model/fill.png)

任何封闭形状都可以填充。

### 12.7.8 文本
可以使用`Text`对象将文本放置在任何位置：

```cpp
Text t(Point(150, 150), "Hello, graphical world!");
win.attach(t);
```

其中(150, 150)是文本框左下角坐标。

![文本](/assets/images/ppp-note-ch12-a-display-model/text.png)

利用上图中的基本图形元素，你可以构建任意复杂、微妙的显示效果。注意，本章中的代码没有循环和选择语句，所有数据都是“硬编码”的，输出只是基本图形元素的简单组合。一旦我们开始使用数据和算法来组合这些基本图形，事情就开始变得有趣了。

12.7.3节中坐标轴的标签就是一个`Text`对象，可以通过`set_color()`改变文本颜色。另外，也可以设置字体和字号：

```cpp
t.set_font(Font::times_bold);
t.set_font_size(20);
```

![文本字体](/assets/images/ppp-note-ch12-a-display-model/text_font.png)

### 12.7.9 图像
可以从文件中加载图像：

```cpp
Image ii(Point(100, 50), "img/image.jpg");  // 400*212-pixel jpg
win.attach(ii);
```

![图像](/assets/images/ppp-note-ch12-a-display-model/image.png)

如果图像太大，则超出窗口区域的部分会被“剪裁”掉。

注：文件名 "img/image.jpg" 是相对路径，相对于**工作目录**，因此必须在img所在目录下运行程序，否则会找不到文件。如果是在命令行运行程序，则工作目录就是命令行的当前工作目录：

```bash
## working directory is "/home/zzy/PPP-code/ch12"
[zzy@ubuntu ~/PPP-code/ch12]$ ../cmake-build/ch12/shape_primitives
```

如果是在IDE运行程序，可以手动设置工作目录：

![设置工作目录](/assets/images/ppp-note-ch12-a-display-model/设置工作目录.png)

### 12.7.10 还有很多
下面的代码展示了图形库的更多特性：

```cpp
Circle c(Point(100, 200), 50);
win.attach(c);

Ellipse e(Point(100, 200), 75, 25);
e.set_color(Color::dark_red);
win.attach(e);

Mark m(Point(100, 200), 'x');
win.attach(m);

std::ostringstream oss;
oss << "screen size: " << x_max() << '*' << y_max()
        << "; window size: " << win.x_max() << '*' << win.y_max();
Text sizes(Point(100, 20), oss.str());
win.attach(sizes);

Image cal(Point(225, 225), "img/snow_cpp.gif");     // 320*240-pixel gif
cal.set_mask(Point(40, 40), 200, 150);  // display center part of image
win.attach(cal);
```

完整代码：

[基本形状](https://github.com/ZZy979/PPP-code/blob/main/ch12/shape_primitives.cpp)

![最终](/assets/images/ppp-note-ch12-a-display-model/final.png)

## 简单练习
[基本形状](https://github.com/ZZy979/PPP-code/blob/main/ch12/shape_primitives.cpp)

## 习题
* [12-1](https://github.com/ZZy979/PPP-code/blob/main/ch12/exec12-1.cpp)
* [12-2](https://github.com/ZZy979/PPP-code/blob/main/ch12/exec12-2.cpp)
* [12-3](https://github.com/ZZy979/PPP-code/blob/main/ch12/exec12-3.cpp)
* [12-4](https://github.com/ZZy979/PPP-code/blob/main/ch12/exec12-4.cpp)
* [12-7](https://github.com/ZZy979/PPP-code/blob/main/ch12/exec12-7.cpp)
* [12-8](https://github.com/ZZy979/PPP-code/blob/main/ch12/exec12-8.cpp)
* [12-11](https://github.com/ZZy979/PPP-code/blob/main/ch12/exec12-11.cpp)
* [12-12](https://github.com/ZZy979/PPP-code/blob/main/ch12/exec12-12.cpp)
