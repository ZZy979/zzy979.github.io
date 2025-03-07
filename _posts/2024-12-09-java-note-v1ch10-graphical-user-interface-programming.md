---
title: 《Java核心技术》笔记 卷I 第10章 图形用户界面
date: 2024-12-09 21:39:59 +0800
categories: [Java, Core Java]
tags: [java, gui, swing, property, event handling, configuration]
---
在本章和下一章中，将讨论使用Swing工具包实现用户界面编程的基础知识。

## 10.1 Java用户界面工具包简史
* AWT (Abstract Window Toolkit)
* IFC (Internet Foundation Classes)
* Swing：Java官方GUI工具包，支持跨平台
* JavaFX

注释：Swing不是完全替代AWT，而是构建在AWT架构之上。

## 10.2 显示窗体
在Java中，顶层窗口（即没有包含在其他窗口中的窗口）称为**窗体**(frame)。AWT库中用于描述这个顶层窗口的类名为`Frame`。这个类的Swing版本名为`JFrame`，扩展了`Frame`类。`JFrame`是极少数几个不在画布上绘制的Swing组件之一，其修饰部件（按钮、标题栏、图标等）由操作系统绘制，而不是Swing。

警告：绝大多数Swing组件类都以 "J" 开头：`JButton`、`JFrame`等。也有`Button`和`Frame`这样的类，但它们属于AWT组件。如果不小心遗漏了 "J" ，程序仍然可以编译和运行，但是混合使用Swing和AWT组件将会导致视觉和行为的不一致。

### 10.2.1 创建窗体
本节将介绍使用`JFrame`的最常见的方法。程序清单10-1给出了在屏幕上显示一个空窗体的简单程序，如下图所示。

[程序清单10-1 simpleFrame/SimpleFrameTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch10/simpleFrame/SimpleFrameTest.java)

![最简单的可见窗体](/assets/images/java-note-v1ch10-graphical-user-interface-programming/最简单的可见窗体.png)

下面来逐行分析这个程序。

Swing类位于`javax.swing`包中。包名中的 "x" 表示这是一个Java扩展(eXtension)包，而不是核心包。

默认情况下，窗体的大小为0×0像素，这没有什么用。我们定义了一个子类`SimpleFrame`，它的构造器将窗体大小设置为300×200像素。

在`SimpleFrameTest`类的`main()`方法中，我们构造了一个`SimpleFrame`类并使其可见。

在每个Swing程序中，需要解决两个技术问题。

首先，所有Swing组件必须由**事件分派线程**(event dispatch thread)配置，该线程将鼠标点击、按键等事件传递给用户界面组件。下面的代码段用来在事件分派线程中执行语句：

```java
EventQueue.invokeLater(() -> {
    // statements
});
```

接下来，定义用户关闭窗体时的响应动作。对于这个程序而言，让程序直接退出。为此使用以下语句：

```java
frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
```

在其他包含多个窗体的程序中，你肯定不希望用户关闭其中一个窗体时程序就退出。默认情况下，用户关闭窗体时只是将窗体隐藏，而程序并没有终止。（如果一旦最后一个窗体不可见程序就终止，这样或许会很好，但Swing并不是这样工作的。）

仅仅构造窗体并不会自动显示它。窗体起初是不可见的，这就给了程序员一个机会，可以在窗体第一次显示之前向其中添加组件。为了显示窗体，需要调用`setVisible()`方法。

完成了初始化语句后，`main()`方法退出。注意，退出`main()`并没有终止程序，终止的只是主线程。事件分派线程会保持程序处于激活状态，直到通过关闭窗体或调用`System.exit()`方法终止程序（注：上面的语句就是让窗体在关闭时调用`System.exit(0)`）。

### 10.2.2 窗体属性
`JFrame`类包含一些用于改变窗体外观的方法，其中大多数方法都来自超类`Component`和`Window`。以下是最重要的几个方法：
* `setSize(width, height)`设置窗体的宽和高。
* `setLocation(x, y)`使窗体左上角位于坐标(x, y)。
* `setBounds(x, y, width, height)`方法同时设置窗体的位置和大小。
* `setIconImage()`方法用于设置在标题栏上显示的图标。
* `setTitle()`方法用于改变标题栏的文本。
* `setResizable()`方法设置是否允许用户改变窗体大小。

组件类的很多方法是以getter/setter形式成对出现的，例如`Frame`类的`getTitle()`和`setTitle()`方法。这样的一对getter/setter称为**属性**(property)。属性具有名称和类型。将`get`或`set`之后的第一个字母改为小写就可以得到属性名称。例如，`Frame`类有一个名为`title`、类型为`String`的属性。但有一个例外：对于`boolean`类型的属性，getter方法以`is`开头。例如，`isResizable()`和`setResizable()`两个方法定义了`resizable`属性。

要确定适当的窗体大小，首先要得到屏幕的大小。调用`Toolkit`类的`getDefaultToolkit()`方法得到`Toolkit`对象。然后调用`getScreenSize()`方法，这个方法以`Dimension`对象的形式返回屏幕的大小。`Dimension`对象用公有（！）实例字段`width`和`height`保存宽度和高度。然后可以使用屏幕大小的适当百分比指定窗体的大小。下面是相关的代码：

```java
Toolkit kit = Toolkit.getDefaultToolkit();
Dimension screenSize = kit.getScreenSize();
int screenWidth = screenSize.width;
int screenHeight = screenSize.height;
setSize(screenWidth / 2, screenHeight / 2);
```

还可以提供窗体图标：

```java
Image img = new ImageIcon("icon.gif").getImage();
setIconImage(img);
```

[sizedFrame/SizedFrame.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch10/sizedFrame/SizedFrame.java)

注：`ImageIcon`类可以通过字符串或URL指定图像文件名。如果使用字符串则与普通文件一样，相对路径是相对于命令行当前工作目录，但无法定位到JAR包内的文件。如果使用URL则与`Class`类加载资源的方式相同，本地文件和JAR包内的文件都适用，详见5.9.3节。

## 10.3 在组件中显示信息
本节将介绍如何在窗体中显示信息（如下图所示）。

![显示信息的窗体](/assets/images/java-note-v1ch10-graphical-user-interface-programming/显示信息的窗体.png)

可以将字符串直接绘制在窗体上，但这并不是一种好的编程习惯。在Java中，窗体被设计为组件的容器。通常，应该在另一个组件上绘制，并将这个组件添加到窗体上。

使用`add()`方法将组件添加到窗体中：

```java
Component c = ...;
frame.add(c);
```

要在一个组件上进行绘制（即自定义组件），需要定义一个扩展`JComponent`的类，并覆盖`paintComponent()`方法。如下所示：

```java
class MyComponent extends JComponent {
    public void paintComponent(Graphics g) {
        // code for drawing
    }
}
```

注：Swing提供了很多内置组件，如文本框、按钮、单选框、复选框、菜单等，将在第11章介绍。

每当窗口需要重新绘制时（如最大化/最小化窗口、被其他窗口覆盖等），事件处理器就会通知组件，自动调用所有组件的`paintComponent()`方法。绝对不要自己调用这个方法。

提示：如果需要强制重新绘制屏幕，调用`repaint()`方法。

所有的绘制都必须通过`Graphics`对象完成。它包含绘制图像和文本的方法，并保存着一些设置（如字体和颜色）。

`Graphics`对象的度量单位是**像素**(pixel)。屏幕/组件左上角的坐标是(0, 0)，x轴向右、y轴向下，如下图所示。

![屏幕坐标系](/assets/images/ppp-note-ch12-a-display-model/屏幕坐标系.png)

`Graphics`类的`drawString()`方法用于显示文本：`g.drawString(text, x, y)`绘制字符串`text`，第一个字符的基线位于坐标(x, y)（注意：不是左上角的坐标，字符基线将在10.3.3节介绍）。

最后，组件要告诉用户它会有多大。覆盖`getPreferredSize()`方法，返回一个包含首选宽度和高度的`Dimension`对象。在窗体中添加一个或多个组件时，如果只想使用它们的首选大小，可以调用`pack()`方法而不是`setSize()`。

程序清单10-2给出了完整的代码。

[程序清单10-2 notHelloWorld/NotHelloWorldComponent.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch10/notHelloWorld/NotHelloWorldComponent.java)

### 10.3.1 处理2D图形
从Java 1.0开始，`Graphics`类就包含绘制直线、矩形和椭圆等方法。但是这些绘制操作非常有限（例如，不能改变线的粗细，不能旋转图形）。我们将使用Java 1.2引入的Java 2D库的图形类。

要使用这个库，需要获得一个`Graphics2D`类的对象。这个类是`Graphics`类的子类。自从Java 1.2以来，`paintComponent()`等方法会自动地接收一个`Graphics2D`对象。只需进行一次强制类型转换，如下所示：

```java
public void paintComponent(Graphics g) {
    var g2 = (Graphics2D) g;
    ...
}
```

Java 2D库采用面向对象的方式组织几何图形。例如，表示直线、矩形和椭圆的类：`Line2D`、`Rectangle2D`和`Ellipse2D`。这些类都实现了`Shape`接口。

要绘制一个图形，首先创建一个实现了`Shape`接口的类的对象，然后调用`Graphics2D`类的`draw()`方法。例如：

```java
Rectangle2D rect = ...;
g2.draw(rect);
```

Java 2D库针对像素采用的是浮点坐标，而不是整数坐标。内部计算采用单精度`float`完成。

然而，有时处理`float`值不太方便，必须使用后缀`F`和强制类型转换。因此2D库的设计者决定为每个图形类提供**两个版本**：一个采用`float`坐标（给节俭的程序员），另一个采用`double`坐标（给懒惰的程序员）。（本书采用第二个版本）

库的设计者采用了一种古怪的机制来包装这些选择。例如，`Rectangle2D`是一个抽象类，有两个具体子类（也是静态内部类）：`Rectangle2D.Float`和`Rectangle2D.Double`。最好忽略这两个子类是静态内部类的事实——这个把戏只是为了避免像`FloatRectangle2D`和`DoubleRectangle2D`这样的名字。

构造这两个子类的对象时，分别提供`float`和`double`值。参数表示矩形的左上角坐标、宽和高。

```java
var floatRect = new Rectangle2D.Float(10.0F, 25.0F, 22.5F, 20.0F);
var doubleRect = new Rectangle2D.Double(10.0, 25.0, 22.5, 20.0);
```

`Rectangle2D`方法的参数和返回值均使用`double`类型。例如，尽管`Rectangle2D.Float`对象将宽度存储为`float`值，但`getWidth()`方法会返回一个`double`值。

提示：直接使用`Double`图形类可以避免处理`float`值。不过，如果需要创建上千个图形对象，还是考虑使用`Float`类以节省内存。

上述讨论也适用于其他图形类。下图显示了图形类之间的关系。这里省略了`Double`和`Float`子类。遗留类用灰色填充。

![图形类之间的关系](/assets/images/java-note-v1ch10-graphical-user-interface-programming/图形类之间的关系.png)

程序清单10-3中的程序绘制了一个矩形、矩形的内接椭圆、矩形的一条对角线以及和矩形同心的圆。

[程序清单10-3 draw/DrawComponent.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch10/draw/DrawComponent.java)

![绘制几何图形](/assets/images/java-note-v1ch10-graphical-user-interface-programming/绘制几何图形.png)

### 10.3.2 使用颜色
使用`Graphics2D`类的`setPaint()`方法可以为后续的绘制操作选择颜色。例如：

```java
g2.setPaint(Color.RED);
g2.drawString("Warning!", 100, 100);
```

可以调用`fill()`填充一个封闭图形的内部：

```java
Rectangle2D rect = ...;
g2.setPaint(Color.RED);
g2.fill(rect); // fills rect with red
```

要想用多种颜色绘制，就需要选择一个颜色，绘制，再选择另一种颜色，再绘制。

`Color`类用于定义颜色。这个类为以下13种标准颜色提供了预定义的常量：

```java
BLACK, BLUE, CYAN, DARK_GRAY, GRAY, GREEN, LIGHT_GRAY, MAGENTA, ORANGE, PINK, RED, WHITE, YELLOW
```

可以通过按红、绿、蓝(RGB)分量创建`Color`对象来指定自定义颜色，每个分量取值为0~255：

```java
g2.setPaint(new Color(0, 128, 128)); // a dull blue-green
g2.drawString("Welcome!", 75, 125);
```

[fill/FillComponent.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch10/fill/FillComponent.java)

注释：除了纯色，还可以用其他实现了`Paint`接口的类实例调用`setPaint()`，例如渐变色和纹理。

要设置背景颜色，使用`Component`类（`JComponent`的祖先）的`setBackground()`方法。

```java
var component = new MyComponent();
component.setBackground(Color.PINK);
```

另外，还有一个`setForeground()`方法，用于指定在组件上进行绘制时使用的默认颜色。

### 10.3.3 使用字体
程序清单10-2用默认字体显示了一个字符串。有时，你可能希望用不同的字体显示文本。可以通过**字体名**(font face name)指定一种字体。字体名由**字体族名**（font family name，如 "Helvetica" ）和可选的后缀（如 "Bold" ）组成。

要想知道某台特定计算机上有哪些可用的字体，可以调用`GraphicsEnvironment`类的`getAvailableFontFamilyNames()`方法。

[listFonts/ListFonts.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch10/listFonts/ListFonts.java)

AWT定义了5个逻辑(logical)字体名：

```
SansSerif
Serif
Monospaced
Dialog
DialogInput
```

这些字体名将被映射到客户机上的实际字体。例如，在Windows系统中，SansSerif被映射到Arial。

另外，Oracle JDK总是包含3个字体族，名为 "Lucida Sans" 、 "Lucida Bright" 和 "Lucida Sans Typewriter" 。

要使用某种字体绘制字符，首先创建一个`Font`类的对象，指定字体名、样式和字号，之后调用`setFont()`方法。下面是一个构造`Font`对象的例子：

```java
var sansbold14 = new Font("SansSerif", Font.BOLD, 14);
```

在`Font`构造器中，可以使用逻辑字体名代替字体名。将第二个参数设置为以下值之一来指定样式（常规、 **加粗** 、 _斜体_ 或 **_加粗斜体_**）。

```java
Font.PLAIN
Font.BOLD
Font.ITALIC
Font.BOLD + Font.ITALIC
```

第三个参数是以点数度量的字体大小(point size)。在排版中，通常使用点数指示字体大小，每英寸包含72个点。

下面这段代码使用系统中的标准sans serif字体（14号加粗）显示字符串 "Hello, World!" ：

```java
var sansbold14 = new Font("SansSerif", Font.BOLD, 14);
g2.setFont(sansbold14);
var message = "Hello, World!";
g2.drawString(message, 75, 100);
```

接下来，将这个字符串在组件中居中，而不是绘制在任意位置。为此，需要知道字符串的宽度和高度（像素）。这两个值取决于三个因素：字体、字符串和绘制字体的设备。

要获得表示屏幕设备字体属性的对象，调用`Graphics2D`类的`getFontRenderContext()`方法，直接将返回的对象传递给`Font`类的`getStringBounds()`方法：

```java
FontRenderContext context = g2.getFontRenderContext();
Rectangle2D bounds = sansbold14.getStringBounds(message, context);
```

`getStringBounds()`方法返回包围字符串的矩形。为了解释这个矩形的大小，需要了解几个基本的排版术语（如下图所示）。**基线**(baseline)是一条虚构的线，例如字母 'n' 所在的底线。**上坡度**(ascent)是从基线到**坡顶**(ascender)（如字母 'h' 的上面部分）的距离。**下坡度**(descent)是从基线到**坡底**(descender)（如字母 'p' 的下面部分）的距离。（注：详见[Font metrics](https://en.wikipedia.org/wiki/Typeface#Font_metrics)）

![排版线条术语](/assets/images/java-note-v1ch10-graphical-user-interface-programming/Typography_Line_Terms.svg)

**行间距**(leading)是一行的坡底与下一行的坡顶之间的空隙。字体的**高度**(height)是连续两个基线之间的距离，等于下坡度+行间距+上坡度。

`getStringBounds()`方法返回的矩形宽度是字符串水平方向的宽度，高度是字体的高度。这个矩形的原点位于第一个字符的基线，因此左上角x坐标为0，y坐标为负值（上坡度的相反数），如下图所示。

![字符串边框矩形](/assets/images/java-note-v1ch10-graphical-user-interface-programming/字符串边框矩形.png)

因此，可以如下获得字符串的宽度、高度和上坡度：

```java
double stringWidth = bounds.getWidth();
double stringHeight = bounds.getHeight();
double ascent = -bounds.getY();
```

如果需要知道下坡度或行间距，可以使用`Font`类的`getLineMetrics()`方法：

```java
LineMetrics metrics = f.getLineMetrics(message, context);
float descent = metrics.getDescent();
float leading = metrics.getLeading();
```

注释：如果需要在`paintComponent()`方法外部计算布局大小，无法从`Graphics2D`对象得到字体渲染上下文。应改为调用`JComponent`类的`getFontMetrics()`方法，之后调用`getFontRenderContext()`：

```java
FontRenderContext context = getFontMetrics(f).getFontRenderContext();
```

为了说明位置是正确的，程序清单10-4中的示例程序将字符串在窗体中居中，并绘制了基线和边框矩形。屏幕显示结果如下图所示。

[程序清单10-4 font/FontComponent.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch10/font/FontComponent.java)

![绘制基线和字符串边框](/assets/images/java-note-v1ch10-graphical-user-interface-programming/绘制基线和字符串边框.png)

注：`drawString()`方法的第2、3个参数是第一个字符的基线坐标，因此要从窗体中心点推算出字符串边框左上角的坐标，进而得到基线坐标，如下图所示。

![计算坐标](/assets/images/java-note-v1ch10-graphical-user-interface-programming/计算坐标.png)

假设窗体的宽度为w、高度为h，字符串边框的宽度为w<sub>b</sub>、高度为h<sub>b</sub>，上坡度为a
* 中心点：x<sub>c</sub> = w / 2, y<sub>c</sub> = h / 2
* 边框左上角：x = x<sub>c</sub> - w<sub>b</sub> / 2 = (w - w<sub>b</sub>) / 2, y = y<sub>c</sub> - h<sub>b</sub> / 2 = (h - h<sub>b</sub>) / 2
* 基线位置：x<sub>b</sub> = x, y<sub>b</sub> = y + a

### 10.3.4 显示图像
可以使用`ImageIcon`类从文件读取图像：

```java
Image image = new ImageIcon(filename).getImage();
```

可以使用`Graphics`类的`drawImage()`方法显示这个图像。

```java
g.drawImage(image, x, y, null);
```

可以再进一步，用图像平铺窗口，结果如下图所示。首先在左上角绘制图像的一个副本，然后使用`copyArea()`将其复制到整个窗口：

```java
for (int i = 0; i * imageWidth <= getWidth(); i++)
    for (int j = 0; j * imageHeight <= getHeight(); j++)
        if (i + j > 0)
            g.copyArea(0, 0, imageWidth, imageHeight, i * imageWidth, j * imageHeight);
```

![平铺图像的窗口](/assets/images/java-note-v1ch10-graphical-user-interface-programming/平铺图像的窗口.png)

下面是图像显示程序的完整代码。

[image/ImageComponent.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch10/image/ImageComponent.java)

## 10.4 事件处理
任何支持GUI的操作环境都会持续监控按键或鼠标点击之类的事件，并将这些事件报告给正在运行的程序。然后程序决定如何对这些事件做出响应（如果需要）。

### 10.4.1 基本事件处理概念
在Java AWT中，**事件源**(event source)（如按钮或滚动条）有一些方法，允许你注册**事件监听器**(event listener)——对事件做出所需响应的对象。

当事件监听器收到关于某个事件的通知时，该事件的相关信息会封装在一个**事件对象**(event object)中。在Java中，所有的事件对象最终都派生于`java.util.EventObject`类。对于每种事件类型有相应的子类，例如`ActionEvent`和`WindowEvent`。

不同的事件源可以产生不同类型的事件。例如，按钮可以发送`ActionEvent`对象，而窗口会发送`WindowEvent`对象。

总之，AWT事件处理机制的工作原理概述如下：
* 事件监听器是实现了监听器接口的类的实例。
* 事件源能够注册监听器并向其发送事件对象。
* 当事件发生时，事件源将事件对象发送给所有注册的监听器。
* 监听器对象使用事件对象中的信息决定如何对事件做出响应。

下面是一个指定监听器的示例：

```java
ActionListener listener = ...;
var button = new JButton("OK");
button.addActionListener(listener);
```

只要按钮发生了“动作事件”，`listener`对象就会得到通知。对于按钮来说，“动作事件”就是点击按钮。

要实现`ActionListener`接口，监听器类必须提供`actionPerformed()`方法，该方法接收一个`ActionEvent`对象作为参数。

注：`ActionListener`是函数式接口，因此也可以使用Lambda表达式作为监听器对象，详见10.4.3节。6.1.7节已经使用过这个接口。

每当用户点击按钮时，`JButton`对象就会创建一个`ActionEvent`对象，并调用`listener.actionPerformed(event)`传入这个事件对象。一个事件源可以有多个监听器。在这种情况下，事件源会调用所有监听器的`actionPerformed()`方法。

### 10.4.2 示例：处理按钮点击事件
在这个示例中，我们在一个面板中放置三个按钮，并添加动作监听器。用户点击按钮时，将改变面板的背景色。

要创建一个按钮，需要在构造器中指定标签字符串或图标，或二者都指定。例如：

```java
var yellowButton = new JButton("Yellow");
var blueButton = new JButton(new ImageIcon("blue-ball.gif"));
```

调用`add()`方法将按钮添加到面板中：

```java
buttonPanel.add(yellowButton);
```

接下来需要添加监听器，这需要一个实现了`ActionListener`接口的类。把所需的颜色存储在监听器类中：

```java
class ColorAction implements ActionListener {
    private Color backgroundColor;

    public ColorAction(Color c) {
        backgroundColor = c;
    }

    public void actionPerformed(ActionEvent event) {
        // set panel background color
    }
}
```

然后，为每种颜色构造一个对象，并将其设置为按钮监听器。

```java
var yellowAction = new ColorAction(Color.YELLOW);
yellowButton.addActionListener(yellowAction);
```

还有一个问题：`ColorAction`对象不能访问`buttonPanel`变量。可以采用两种方法解决这个问题。一种方法是将面板存储在`ColorAction`对象中，并在构造器中设置它；或者，更方便的方法是将`ColorAction`设计为`ButtonFrame`类的内部类，这样它的方法就自动地能够访问外部类的面板了。

程序清单10-5包含了完整的窗体类。只要点击任何一个按钮，对应的动作监听器就会改变面板的背景色。

[程序清单10-5 button/ButtonFrame.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch10/button/ButtonFrame.java)

![填充了按钮的面板](/assets/images/java-note-v1ch10-graphical-user-interface-programming/填充了按钮的面板.png)

![改变了背景色的面板](/assets/images/java-note-v1ch10-graphical-user-interface-programming/改变了背景色的面板.png)

注：`JPanel`是可以包含其他组件的容器。这里使用`JPanel`而不是直接将按钮添加到`JFrame`，是因为`JFrame`默认是边框布局，添加第二个按钮会覆盖第一个；而`JPanel`默认是流布局，可以将添加的组件依次排列（详见11.2.2节）。

### 10.4.3 简洁地指定监听器
一个监听器类有多个实例的情况并不多见。更常见的情况是，每个监听器执行一个单独的动作。在这种情况下，就没有必要定义单独的类，可以直接使用Lambda表达式：

```java
exitButton.addActionListener(event -> System.exit(0));
```

考虑有多个相关联的动作的情况，如上一节中的颜色按钮。在这种情况下，可以实现一个辅助方法：

```java
public void makeButton(String name, Color backgroundColor) {
    var button = new JButton(name);
    buttonPanel.add(button);
    button.addActionListener(event -> buttonPanel.setBackground(backgroundColor));
}
```

然后只需要调用

```java
makeButton("Yellow", Color.YELLOW);
makeButton("Blue", Color.BLUE);
makeButton("Red", Color.RED);
```

### 10.4.4 适配器类
并不是所有的事件处理都像按钮点击那样简单。当用户试图关闭一个窗口时，`JFrame`对象就是`WindowEvent`的事件源。如果希望捕获这个事件，就必须有一个合适的监听器对象，并将它添加到窗体的监听器列表中。

```java
WindowListener listener = ...;
frame.addWindowListener(listener);
```

窗口监听器是实现`WindowListener`接口的类的对象。这个接口包含7个方法：

| | 方法 | 描述 |
| --- | --- | --- |
| ① | `windowOpened()` | 窗口首次可见时调用 |
| ② | `windowClosing()` | 用户尝试关闭窗口时调用 |
| ③ | `windowClosed()` | 窗口已关闭时调用 |
| ④ | `windowIconified()` |  窗口最小化时调用 |
| ⑤ | `windowDeiconified()` | 窗口从最小化恢复时调用 |
| ⑥ | `windowActivated()` | 窗口获得焦点时调用 |
| ⑦ | `windowDeactivated()` | 窗口失去焦点时调用 |

注：通过打印语句查看各方法的调用顺序：
* 程序刚启动：⑥①
* 最小化窗口：④⑦
* 从最小化恢复：⑤⑥
* 最大化窗口：无（见下注释）
* 关闭窗口：②
  * 如果没有调用`setDefaultCloseOperation()`则还会调用⑦，否则不调用
  * 不会调用③（只有在调用`dispose()`时才会被调用）

注释：为了能够查看窗口是否被最大化，需要实现`WindowStateListener`并覆盖`windowStateChanged()`方法。

前面说过，在Java中，实现一个接口的类必须实现其中的所有方法。在这里意味着需要实现7个方法。假设只对`windowClosing()`方法感兴趣。当然，可以让其他6个方法不做任何事，但这样会很繁琐。为了简化这个任务，每个包含多个方法的AWT监听器接口有配有一个**适配器**(adapter)类，这个类实现了接口中的所有方法，但每个方法并不做任何事情。可以扩展适配器类，只覆盖感兴趣的方法。

可以如下定义一个覆盖了`windowClosing()`方法的窗口监听器：

```java
class Terminator extends WindowAdapter {
    @Override
    public void windowClosing(WindowEvent e) {
        if (/* user agrees */)
            System.exit(0);
    }
}
```

现在可以注册`Terminator`对象作为窗口监听器：

```java
var listener = new Terminator();
frame.addWindowListener(listener);
```

注释：如今，可能会把`WindowListener`接口中什么也不做的方法实现为默认方法。不过，Swing早在有默认方法很多年之前就已经问世了。

### 10.4.5 动作
通常，有多种方式激活同一个命令。用户可以通过菜单、按键或工具栏上的按钮选择特定的功能。在AWT事件模型中这非常容易实现：将所有事件关联到同一个监听器。例如，假设`blueAction`是一个动作监听器，用于将背景色变成蓝色。可以将其作为多个事件源的监听器：
* 工具栏按钮 "Blue"
* 菜单项 "Blue"
* 按键Ctrl+B

现在改变颜色命令将以统一的方式处理，无论它是由点击按钮、选择菜单还是按键引起的。

Swing包提供了一种非常有用的机制来封装命令并将其关联到多个事件源，这就是`Action`接口。**动作**(action)是封装了以下内容的对象：
* 命令的描述（文本字符串和可选的图标）
* 执行命令所需要的参数（例如上面示例中的颜色）

`Action`接口包含以下方法：

```java
void actionPerformed(ActionEvent event)
void setEnabled(boolean b)
boolean isEnabled()
void putValue(String key, Object value)
Object getValue(String key)
void addPropertyChangeListener(PropertyChangeListener listener)
void removePropertyChangeListener(PropertyChangeListener listener)
```

第一个方法很熟悉：实际上，`Action`接口扩展了`ActionListener`接口。因此，任何需要`ActionListener`对象的地方都可以使用`Action`对象。

`setEnabled()`和`isEnabled()`方法设置和检查这个动作是否启用。当一个动作关联到菜单或工具栏而且被禁用时，相应选项就会变成灰色。

`putValue()`和`getValue()`方法允许存储和获取动作对象中的任意名字/值对（注：即动作对象可以当作映射，但值是以`Object`类型存储的，因此获取时必须使用强制类型转换）。有两个重要的预定义字符串：`Action.NAME`和`Action.SMALL_ICOM`，用于作为键将名字和图标存储到动作对象中：

```java
action.putValue(Action.NAME, "Blue");
action.putValue(Action.SMALL_ICON, new ImageIcon("blue-ball.gif"));
```

所有预定义键名称参见`Action`接口的API文档。

如果将动作对象添加到菜单或工具栏上，其名称和图标就会被自动获取并显示在菜单项或工具栏按钮中，`SHORT_DESCRIPTION`值会作为工具提示。

`Action`接口的最后两个方法能够让其他对象（尤其是触发动作的菜单或工具栏）在动作对象的属性发生变化时得到通知。

注意，`Action`是接口，而不是类。任何实现这个接口的类都必须实现上述7个方法。幸运的是，有好心人已经提供了一个类`AbstractAction`，实现了除`actionPerformed()`之外的所有方法。这个类负责存储名字/值对，并管理属性变更监听器。我们只需扩展`AbstractAction`类，并提供`actionPerformed()`方法。

注：总之，动作≈动作监听器+映射。

下面构造一个可以执行改变颜色命令的动作对象。

```java
public class ColorAction extends AbstractAction {
    public ColorAction(String name, Icon icon, Color c) {
        putValue(Action.NAME, name);
        putValue(Action.SMALL_ICON, icon);
        putValue(Action.SHORT_DESCRIPTION, "Set panel color to " + name.toLowerCase());
        putValue("color", c);
    }

    public void actionPerformed(ActionEvent event) {
        Color c = (Color) getValue("color");
        buttonPanel.setBackground(c);
    }
}
```

测试程序创建了这个类的三个对象，例如：

```java
var blueAction = new ColorAction("Blue", new ImageIcon("blue-ball.gif"), Color.BLUE);
```

接下来，将这个动作与一个按钮关联起来。可以使用接受一个`Action`对象的`JButton`构造器：

```java
var blueButton = new JButton(blueAction);
```

这个构造器会读取动作的名字和图标，将描述设置为工具提示，将动作设置为监听器。图标和工具提示如下图所示。

![按钮显示动作对象中的图标和工具提示](/assets/images/java-note-v1ch10-graphical-user-interface-programming/按钮显示动作对象中的图标和工具提示.png)

在下一章中将会看到，将这个动作添加到菜单也非常容易。

最后，为按键添加动作对象，以便在用户键入键盘命令时执行相应的动作。为此，首先需要生成`KeyStroke`类的对象，它封装了对按键的描述。

```java
KeyStroke ctrlBKey = KeyStroke.getKeyStroke("ctrl B");
```

为了理解下一个步骤，需要知道**键盘焦点**(keyboard focus)的概念。用户界面中可能有许多按钮、菜单、滚动条以及其他组件。当用户按键时，这个动作会被发送给拥有焦点的组件。例如，有焦点的按钮文本周围有一个很细的矩形边框。可以使用Tab键在组件之间移动焦点。当按下空格键时，就会点击有焦点的按钮。其他键会执行不同的动作，例如方向键可以移动滚动条。

然而，在我们的示例中，我们并不希望将按键发送给拥有焦点的组件。否则，每个按钮都需要知道如何处理组合键Ctrl+Y、Ctrl+B和Ctrl+R。

这是一个常见的问题。Swing设计者给出了一种便捷的解决方案。每个`JComponent`有三个**输入映射**(input map)，分别将`KeyStroke`映射到关联的动作。这三个输入映射对应三种不同的条件，见下表（标志常量定义在`JComponent`类中）。

| 标志 | 调用动作 |
| --- | --- |
| `WHEN_FOCUSED` | 当这个组件拥有焦点时 |
| `WHEN_ANCESTOR_OF_FOCUSED_COMPONENT` | 当这个组件包含拥有焦点的组件时 |
| `WHEN_IN_FOCUSED_WINDOW` | 当这个组件与拥有焦点的组件在同一个窗口中时 |

按键处理将按照以下顺序检查这些映射：
1. 检查拥有焦点的组件的`WHEN_FOCUSED`映射。如果这个按键存在且启用了相应的动作，则执行这个动作并停止处理。
2. 从拥有焦点的组件开始，检查其父组件的`WHEN_ANCESTOR_OF_FOCUSED_COMPONENT`映射。一旦找到有这个按键且相应的动作已启用的映射，就执行这个动作并停止处理。
3. 在拥有焦点的窗口中查看所有可见、已启用，且`WHEN_IN_FOCUSED_WINDOW`映射中注册了这个按键的组件。（按照按键注册的顺序）依次让这些组件尝试执行相应的动作。一旦执行了第一个已启用的动作，就停止处理。

可以使用`getInputMap()`方法从组件获得一个输入映射（注：需要指定上表中的标志之一，默认为`WHEN_FOCUSED`）。例如：

```java
InputMap imap = panel.getInputMap(JComponent.WHEN_FOCUSED);
```

在我们的示例中，不应该使用`WHEN_FOCUSED`映射，另外两个映射都可以。示例程序中使用的是（`buttonPanel`的）`WHEN_ANCESTOR_OF_FOCUSED_COMPONENT`映射。

`InputMap`不是直接将`KeyStroke`对象映射到`Action`对象，而是先映射到任意对象。然后由`ActionMap`类实现的第二个映射将对象映射到动作。这样可以更容易地让不同输入映射中的按键共享一个动作。

因此，每个组件有三个输入映射和一个动作映射。为了将它们关联起来，需要为动作命名（使用字符串作为中间对象）。例如，可以如下将按键Ctrl+Y（通过字符串名称`"panel.yellow"`）关联到动作`yellowAction`：

```java
imap.put(KeyStroke.getKeyStroke("ctrl Y"), "panel.yellow");
ActionMap amap = panel.getActionMap();
amap.put("panel.yellow", yellowAction);
```

习惯上使用字符串`"none"`表示空动作。这样可以轻松地取消一个按键：

```java
imap.put(KeyStroke.getKeyStroke("ctrl C"), "none");
```

下面总结让按钮、菜单项或按键执行相同动作的方式：
1. 实现一个扩展`AbstractAction`类的类。可以使用同一个类表示多个相关的动作。
2. 构造一个动作类的对象。
3. 使用动作对象构造一个按钮或菜单项。
4. 为了能够通过按键触发动作，必须执行额外几步：
    1. 首先，找到窗口的顶层组件（例如包含所有其他组件的面板）。
    2. 然后，获取顶层组件的`WHEN_ANCESTOR_OF_FOCUSED_COMPONENT`输入映射。为需要的按键创建一个`KeyStroke`对象。创建一个动作键对象（如描述动作的字符串）。将（按键，动作键）对添加到输入映射中。
    3. 最后，获取顶层组件的动作映射。将（动作键，动作对象）对添加到映射中。

下面给出了将按钮和按键关联到动作的完整程序代码。尝试点击按钮或按Ctrl+Y、Ctrl+B和Ctrl+R键来改变面板颜色。

[action/ActionFrame.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch10/action/ActionFrame.java)

### 10.4.6 鼠标事件
如果只希望用户能够点击按钮或菜单，就不需要显式地处理鼠标事件。这些鼠标操作将由各种组件内部处理。不过，如果希望用户能使用鼠标画图，就需要捕获鼠标移动、点击和拖动事件。

本节将展示一个简单的图形编辑器应用，允许用户在画布上放置、移动和擦除方块（如下图所示）。

![鼠标测试程序](/assets/images/java-note-v1ch10-graphical-user-interface-programming/鼠标测试程序.png)

当用户点击鼠标按钮时，会调用三个监听器方法：按下时调用`mousePressed()`，松开时调用`mouseReleased()`，最后调用`mouseClicked()`。如果只对整个点击感兴趣，就可以忽略前两个方法。使用`MouseEvent`参数的`getX()`和`getY()`方法可以获得点击鼠标时鼠标指针的x和y坐标。要区分单击、双击和三击，需要使用`getClickCount()`方法。

在我们的示例程序中，提供了`mousePressed()`和`mouseClicked()`方法。当点击在所有已绘制的方块之外的像素时，就会添加一个新的方块。这个操作是在`mousePressed()`方法中实现的。如果在某个方块中双击，就会将其擦除。这个操作是在`mouseClicked()`方法中实现的，因为需要知道点击次数。

```java
public void mousePressed(MouseEvent event) {
    current = find(event.getPoint());
    if (current == null) // not inside a square
        add(event.getPoint());
}

public void mouseClicked(MouseEvent event) {
    current = find(event.getPoint());
    if (current != null && event.getClickCount() >= 2)
        remove(current);
}
```

当鼠标在窗口上移动时，窗口将会收到一连串的鼠标移动事件（每移动一个像素就发生一次）。因此有两个独立的接口`MouseListener`（按下、释放、点击）和`MouseMotionListener`（移动、拖动）。这样做是为了效率——当用户移动鼠标时，会有大量鼠标事件，只关心鼠标点击的监听器就不会被鼠标移动所干扰。

测试程序捕获了鼠标移动事件，以便在光标位于方块之上时将其变成不同的形状（十字）。这是使用`Cursor`类的`getPredefinedCursor()`方法完成的。

```java
public void mouseMoved(MouseEvent event) {
    if (find(event.getPoint()) == null)
        setCursor(Cursor.getDefaultCursor());
    else
        setCursor(Cursor.getPredefinedCursor(Cursor.CROSSHAIR_CURSOR));
}
```

注释：还可以利用`Toolkit`类的`createCustomCursor()`方法自定义光标。

如果用户在移动鼠标的同时按下鼠标按钮，就会生成`mouseDragged()`而不是`mouseMoved()`调用。在测试应用中，用户可以拖动光标下的方块：更新当前拖动的方块，使其以鼠标位置为中心，然后重新绘制画布。

```java
public void mouseDragged(MouseEvent event) {
    if (current != null) {
        int x = event.getX();
        int y = event.getY();
        current.setFrame(x - SIDELENGTH / 2, y - SIDELENGTH / 2, SIDELENGTH, SIDELENGTH);
        repaint();
    }
}
```

注释：只有鼠标停留在组件内部才会调用`mouseMoved()`方法。不过，即使鼠标拖动到组件外部也会调用`mouseDragged()`方法。

还有另外两个鼠标事件方法：`mouseEntered()`和`mouseExited()`。这两个方法会在鼠标移入或移出组件时调用。

最后来看如何监听鼠标事件。在示例程序中，我们对鼠标点击和移动都感兴趣。这里定义了两个内部类：`MouseHandler`和`MouseMotionHandler`。前者扩展了`MouseAdapter`类，因为它只定义了5个`MouseListener`方法中的两个。后者实现了`MouseMotionListener`接口并定义了其中的两个方法。程序清单10-6给出了这个程序的代码。

注：`MouseMotionAdapter`只实现了`MouseMotionHandler`，而`MouseAdapter`同时实现了`MouseHandler`和`MouseMotionHandler`。

[程序清单10-6 mouse/MouseComponent.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch10/mouse/MouseComponent.java)

### 10.4.7 AWT事件继承层次
`EventObject`类有一个子类`AWTEvent`，它是所有AWT事件的父类。下图显示了AWT事件的继承图。

![AWT事件类继承图](/assets/images/java-note-v1ch10-graphical-user-interface-programming/AWT事件类继承图.png)

事件对象封装了事件源与监听器通信的事件信息（例如`MouseEvent`包含鼠标指针位置、鼠标按钮、点击次数等）。在必要时可以分析传递给监听器的事件对象。

AWT将事件分为**底层**(low-level)事件和**语义**(semantic)事件。语义事件是表示用户动作的事件，底层事件是使语义事件得以发生的事件。下面是`java.awt.event`包中最常用的语义事件类：
* `ActionEvent`（点击按钮、选择菜单、选择列表项或在文本框中按回车）
* `AdjustmentEvent`（调节滚动条）
* `ItemEvent`（从复选框或列表框中选择一项）

常用的5个底层事件类：
* `KeyEvent`（按下或松开键）
* `MouseEvent`（按下、松开、移动或拖动鼠标）
* `MouseWheelEvent`（滚动鼠标滚轮）
* `FocusEvent`（组件获得或失去焦点）
* `WindowEvent`（窗口状态改变）

下表列出了最重要的AWT监听器接口、事件和事件源。

| 监听器接口 | 事件 | 事件源 |
| --- | --- | --- |
| `ActionListener` | `ActionEvent` | `AbstractButton` <br> `JMenuItem` <br> `JComboBox` <br> `JTextField` <br> `Timer` |
| `AdjustmentListener` | `AdjustmentEvent` | `JScrollBar` |
| `ChangeListener` | `ChangeEvent` | `JSlider` |
| `ItemListener` | `ItemEvent` | `AbstractButton` <br> `JComboBox` |
| `FocusListener` | `FocusEvent` | `Component` |
| `KeyListener` | `KeyEvent` | `Component` |
| `MouseListener` | `MouseEvent` | `Component` |
| `MouseMotionListener` | `MouseEvent` | `Component` |
| `MouseWheelListener` | `MouseWheelEvent` | `Component` |
| `WindowListener` | `WindowEvent` | `Window` |
| `WindowFocusListener` | `WindowEvent` | `Window` |
| `WindowStateListener` | `WindowEvent` | `Window` |

## 10.5 首选项API
本章的最后来讨论`Preferences` API。桌面程序通常都会存储用户首选项，例如用户上次处理的文件、窗口位置等等。

在第9章已经看到，利用`Properties`类可以很容易地加载和保存程序的配置信息。不过，使用属性文件有以下缺点：
* 有些操作系统没有主目录的概念，很难为配置文件找到一个统一的位置。
* 配置文件的命名没有标准约定，用户安装多个Java应用时更容易发生命名冲突。

有些操作系统有一个存储配置信息的中心存储库。最著名的例子就是Windows注册表。`Preferences`类以一种平台无关的方式提供了这样一个中心存储库。在Windows中，`Preferences`类使用注册表来存储信息；在Linux中，信息则存储在本地文件系统中。当然，存储库的实现对使用`Preferences`类的程序员是透明的。

注：`Preferences`实际上是一个抽象类
* 在Windows中，实现类为`WindowsPreferences`，系统配置的注册表路径为HKEY_LOCAL_MACHINE\Software\JavaSoft\Prefs，用户配置的注册表路径为HKEY_CURRENT_USER\Software\JavaSoft\Prefs，参见Open JDK实现[WindowsPreferences.java](https://github.com/openjdk/jdk/blob/master/src/java.prefs/windows/classes/java/util/prefs/WindowsPreferences.java)。
* 在Linux中，实现类为`FileSystemPreferences`，系统配置的存储路径默认为/etc/.java/.systemPrefs，用户配置的存储路径默认为$HOME/.java/.userPrefs，参见Open JDK实现[FileSystemPreferences.java](https://github.com/openjdk/jdk/blob/master/src/java.prefs/unix/classes/java/util/prefs/FileSystemPreferences.java) `setupSystemRoot()`和`setupUserRoot()`。

`Preferences`存储库有一个树状结构，节点路径名类似于`/com/mycompany/myapp`。与包名一样，只要用逆序的域名作为路径开头，就可以避免命名冲突。API的设计者建议配置节点路径与程序中的包名一致。

存储库中的每个节点分别有一个单独的键/值对表，可用于存储数值、字符串或字节数组，但不能存储可序列化的对象。

为了增加灵活性，有多个并行的树。每个程序用户分别有一棵树；另外还有一颗系统树，用于存放所有用户的公共设置。`Preferences`类使用操作系统的“当前用户”概念来访问相应的用户树。

首先获取用户树或系统树的根：

```java
Preferences root = Preferences.userRoot();
```

或

```java
Preferences root = Preferences.systemRoot();
```

要访问树中的节点，只需提供节点路径名：

```java
Preferences node = root.node("/com/mycompany/myapp");
```

如果节点的路径名等于类的包名，还有一种便捷方式来获得这个节点：

```java
Preferences node = Preferences.userNodeForPackage(obj.getClass());
```

或


```java
Preferences node = Preferences.systemNodeForPackage(obj.getClass());
```

一旦有了节点，就可以用以下方法访问键/值表：

```java
String get(String key, String defval)
int getInt(String key, int defval)
long getLong(String key, long defval)
float getFloat(String key, float defval)
double getDouble(String key, double defval)
boolean getBoolean(String key, boolean defval)
byte[] getByteArray(String key, byte[] defval)
```

注意，读取信息时必须指定默认值。

相对应地，可以用以下方法向存储库写数据：

```java
put(String key, String value)
putInt(String key, int value)
...
```

可以用`keys()`方法枚举一个节点中的所有键，使用`remove(key)`删除指定的键。

目前没有办法找出特定的键对应值的类型。

注释：节点名称和键限制为最长80个字符，字符串值最长8192个字符。

像Windows注册表这样的中心存储库存在两个问题：
* 它们会变成充斥着过期信息的“垃圾场”。
* 很难把配置数据迁移到新平台。

`Preferences`类为第二个问题提供了解决方案。可以调用以下方法导出一个子树或者一个节点的首选项：

```java
void exportSubtree(OutputStream out)
void exportNode(OutputStream out)
```

数据用XML格式保存。可以调用以下方法将其导入另一个存储库：

```java
void importPreferences(InputStream in)
```

下面是一个示例文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE preferences SYSTEM "http://java.sun.com/dtd/preferences.dtd">
<preferences EXTERNAL_XML_VERSION="1.0">
  <root type="user">
    <map/>
    <node name="com">
      <map/>
      <node name="horstmann">
        <map/>
        <node name="corejava">
          <map>
            <entry key="height" value="200.0" />
            <entry key="left" value="1027.0" />
            <entry key="filename" value="/home/cay/books/cj11/code/v1ch11/raven.html" />
            <entry key="top" value="380.0" />
            <entry key="width" value="300.0" />
          </map>
        </node>
      </node>
    </node>
  </root>
</preferences>
```

如果你的程序使用首选项，应该让用户有机会导出和导入首选项，从而可以很容易地将设置从一台计算机迁移到另一台计算机。程序清单10-7中的程序展示了这种技术。程序会保存窗口位置和上次加载的文件。试着调整窗口大小，然后导出你的首选项，移动窗口，退出并重启应用。窗口的状态应该与之前退出时是一样的。导入你的首选项，窗口会恢复到之前的位置。

[程序清单10-7 preferences/ImageViewerFrame.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch10/preferences/ImageViewerFrame.java)
