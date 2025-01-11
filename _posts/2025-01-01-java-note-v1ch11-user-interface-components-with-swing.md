---
title: 《Java核心技术》笔记 第11章 Swing用户界面组件
date: 2025-01-01 15:15:52 +0800
categories: [Java, Core Java]
tags: [java, gui, swing]
---
本章首先介绍Swing的底层架构。然后会介绍Swing中最常用的用户界面组件，如文本框、单选按钮和菜单等。接下来，你会了解如何使用布局管理器排列这些组件。最后将介绍如何在Swing中实现对话框。

本章涵盖基本的Swing组件。高级Swing组件将在卷II第11章介绍。

## 11.1 Swing和MVC设计模式
思考以下构成用户界面组件的各个组成部分：
* **内容**（如按钮是否按下、文本框中的文本）
* **外观**（颜色、大小等）
* **行为**（对事件的反应）

Swing设计者采用了一种很有名的设计模式：**模型-视图-控制器**(model-view-controller, MVC)模式。这种设计模式要求提供三个独立的对象：
* **模型**(model)：存储内容
* **视图**(view)：显示内容
* **控制器**(controller)：处理用户输入

例如，对于`JButton`：

| | 接口 | 实现类 |
| --- | --- | --- |
| 模型 | `ButtonModel` | `DefaultButtonModel` |
| 视图 | `ButtonUI` | `BasicButtonUI` |
| 控制器 | | `BasicButtonListener` |

## 11.2 布局管理概述
在讨论各个Swing组件之前，首先介绍如何在窗体中排列这些组件。

### 11.2.1 布局管理器
先来回顾[程序清单10-5](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch10/button/ButtonFrame.java)中的程序，这个程序使用按钮来改变窗体的背景色（如下图所示）。这几个按钮包含在一个`JPanel`对象中，用**流布局**(flow layout)管理，这是`JPanel`的默认布局管理器。

![填充了按钮的面板](/assets/images/java-note-v1ch10-graphical-user-interface-programming/填充了按钮的面板.png)

下图显示了向面板中添加更多按钮后的效果。可以看到，当一行的空间不够时，会显示在新的一行上。

![用流布局管理六个按钮的面板](/assets/images/java-note-v1ch11-user-interface-components-with-swing/用流布局管理六个按钮的面板.png)

另外，按钮总是在面板中居中，即使用户调整了窗体大小也是如此（如下图所示）。

![改变面板大小会自动重新排列按钮](/assets/images/java-note-v1ch11-user-interface-components-with-swing/改变面板大小会自动重新排列按钮.png)

[layoutManager/FlowLayoutFrame.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch11/layoutManager/FlowLayoutFrame.java)

一般而言，**组件**(component)放置在**容器**(container)中，**布局管理器**(layout manager)决定容器中组件的位置和大小。

按钮等用户界面组件都扩展了`Component`类。组件可以放置在容器（如面板）中。容器本身也可以放置在其他容器中，因此`Container`类扩展了`Component`类。下图显示了`Component`的继承层次结构。

![Component类的继承层次结构](/assets/images/java-note-v1ch11-user-interface-components-with-swing/Component类的继承层次结构.png)

注释：遗憾的是，这个继承层次结构在两方面有些不太清楚。首先，顶层窗口（如`JFrame`）是`Container`的子类，所以也是`Component`的子类，却不能放在其他容器中。另外，`JComponent`是`Container`的子类，而不是直接继承`Component`。因此，可以将其他组件添加到`JButton`中（但这些组件不会显示）。

每个容器都有一个默认的布局管理器，但可以使用`setLayout()`方法重新设置。向容器中添加组件时，容器的`add()`方法将把组件和位置传递给布局管理器。

### 11.2.2 边框布局
**边框布局**(border layout)是`JFrame`内容窗格的默认布局管理器。流布局完全控制每个组件的位置，而边框布局允许你选择放置每个组件的位置：东、西、南、北、中，如下图所示。

![边框布局](/assets/images/java-note-v1ch11-user-interface-components-with-swing/边框布局.png)

[layoutManager/BorderLayoutFrame.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch11/layoutManager/BorderLayoutFrame.java)

先放置边缘组件，剩余空间由中间组件占据。添加组件时可以指定位置，如`BorderLayout.SOUTH`。如果没有提供任何值，则默认为`CENTER`。并不是所有的位置都需要占据（例如，只在西、北、中部添加了组件的边框布局如下图所示）。

![只占据了部分位置的边框布局](/assets/images/java-note-v1ch11-user-interface-components-with-swing/只占据了部分位置的边框布局.png)

与流布局不同，边框布局会扩展所有组件以填满可用空间（流布局会维持每个组件的首选尺寸，即`getPreferredSize()`）。添加按钮时这会有问题：

```java
frame.add(yellowButton, BorderLayout.SOUTH); // don't
```

下图显示了执行上述代码的结果。按钮会扩展至填满窗体的整个南部区域。而且，如果再将另一个按钮添加到南部区域，就会取代第一个按钮。

![边框布局管理的一个按钮](/assets/images/java-note-v1ch11-user-interface-components-with-swing/边框布局管理的一个按钮.png)

解决这个问题的方法是使用面板。如下图所示，三个按钮都包含在一个面板中，面板放置在窗体的南部区域。

![面板放置在窗体的南部区域](/assets/images/java-note-v1ch11-user-interface-components-with-swing/面板放置在窗体的南部区域.png)

```java
var panel = new JPanel();
panel.add(yellowButton);
panel.add(blueButton);
panel.add(redButton);
frame.add(panel, BorderLayout.SOUTH);
```

### 11.2.3 网格布局
**网格布局**(grid layout)按行和列排列所有的组件，就像电子表格一样。所有组件的大小都相同。下图显示的计算器程序就使用了网格布局来排列计算器按钮。当调整窗口大小时，按钮将随之变大或变小，但所有按钮的大小始终保持一致。

![计算器](/assets/images/java-note-v1ch11-user-interface-components-with-swing/计算器.png)

在网格布局对象的构造器中，需要指定行数和列数：

```java
panel.setLayout(new GridLayout(4, 4));
```

添加组件时，从第一行第一列开始，然后是第一行第二列，以此类推。

```java
panel.add(new JButton("7"));
panel.add(new JButton("8"));
...
```

下面是计算器程序的源代码。在这个程序中，将组件添加到窗体之后调用了`pack()`方法，从而使用所有组件的首选尺寸来计算窗体的宽度和高度。

[calculator/CalculatorPanel.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch11/calculator/CalculatorPanel.java)

当然，极少应用会有像计算器这样整齐的布局。在实际中，小网格（通常只有一行或一列）对于组织窗口的部分区域会很有用。例如，如果想要一行大小相同的按钮，可以将这些按钮放在一个面板中，并使用只有一行的网格布局进行管理。

## 11.3 文本输入
终于可以开始介绍Swing用户界面组件了。首先来介绍允许用户输入和编辑文本的组件。可以使用文本框(`JTextField`)和文本区(`JTextArea`)组件输入文本。文本框只能接受单行文本，而文本区能够接受多行文本。密码框(`JPasswordField`)也接受单行文本，但不会将内容显示出来。

这三个类都继承自`JTextComponent`类。获取或设置文本等方法实际上都在`JTextComponent`类中。

### 11.3.1 文本框
把文本框添加到窗口的常用方法是将其添加到面板或其他容器中：

```java
var panel = new JPanel();
var textField = new JTextField("Default input", 20);
panel.add(textField);
```

这段代码添加了一个文本框，并在初始化时设置了字符串`"Default input"`。构造器的第二个参数设置宽度为20列。一列是当前字体一个字符的宽度。另外要记住，列数只是首选大小，布局管理器可能会调整文本框的大小。当用户输入文本的长度超过文本框的长度时，输入就会左右滚动。如果需要在运行时重新设置列数，可以使用`setColumns()`方法。

提示：使用`setColumns()`改变文本框的大小之后，需要调用外围容器的`revalidate()`方法，该方法会重新计算容器内所有组件的大小和布局。如果想重新计算`JFrame`中的所有组件，则需要调用`validate()`方法——`JFrame`没有扩展`JComponent`。

要构造一个空白文本框，只需省略构造器的字符串参数：

```java
var textField = new JTextField(20);
```

可以使用`setText()`方法改变文本框的内容，调用`getText()`方法获取文本。要改变显示文本的字体，使用`setFont()`方法。

### 11.3.2 标签
标签(`JLabel`)是容纳文本的组件。它没有任何修饰（如边框），也不能响应用户输入。可以使用标签来标识组件。例如，与按钮不同，文本框不带标签。要对这种本身不带标识的组件加标签：
1. 用正确的文本构造一个`JLabel`组件。
2. 将它放置在距离所要标识的组件足够近的地方。

`JLabel`的构造器允许指定文本或图标，以及可选的对齐方式。例如，可以如下创建一个右对齐的标签：

```java
var label = new JLabel("Username: ", JLabel.RIGHT);
```

`setText()`和`setIcon()`方法可以在运行时设置标签的文本和图标。

提示：可以在标签中使用纯文本或HTML。只需将标签字符串放在`<html>...</html>`之间，如下：

```java
var label = new JLabel("<html><b>Required</b> entry:</html>");
```

注意，第一个带有HTML标签的组件可能需要一些时间才能显示，因为需要加载相当复杂的HTML渲染代码。

### 11.3.3 密码框
密码框(`JPasswordField`)是一种特殊的文本框。为了避免别人看到密码，用户输入的字符并不真正显示出来，而是用**回显字符**(echo character)标识，如 • 或 * 。

密码框也是一个体现MVC模式强大功能的例子。密码框使用与常规文本框相同的模型来存储数据，但是它的视图将所有字符显示为回显字符。

### 11.3.4 文本区
有时，用户的输入可能超过一行，为此可以使用`JTextArea`组件。在文本区中，用户可以输入多行文本，用回车键换行。每行都以一个`'\n'`结尾。下图显示了文本区的使用。

![文本组件](/assets/images/java-note-v1ch11-user-interface-components-with-swing/文本组件.png)

在`JTextArea`的构造器中，指定行数和列数。例如：

```java
var textArea = new JTextArea(8, 40); // 8 lines of 40 columns each
```

还可以使用`setRows()`和`setColumns()`方法改变行数和列数。和文本框一样，这些值只是首选大小，布局管理器可能会调整文本区的大小。用户也不受限于行数和列数，当输入过长时，文本会滚动。

如果文本超出了可显示的范围，剩余文本就不可见了。可以通过开启自动换行来避免这一问题：

```java
textArea.setLineWrap(true); // long lines are wrapped
```

换行只是视觉效果，文本没有改变——并没有在文本中自动插入`'\n'`字符。

注：自动换行只能解决文本超过列数的问题，超过行数的文本仍然不可见。

### 11.3.5 滚动窗格
在Swing中，文本区没有滚动条。如果需要滚动条，必须将文本区放在**滚动窗格**(scroll pane)中。

```java
var textArea = new JTextArea(8, 40);
var scrollPane = new JScrollPane(textArea);
```

现在滚动窗格管理文本区的视图。如果文本超出了文本区可以显示的范围，滚动条就会自动出现；如果删除部分文本后剩余文本能够在文本区内显示，滚动条就会消失。滚动是由滚动窗格内部处理的，你的程序无需处理滚动事件。

这是一种适用于所有组件的通用机制。要想为组件添加滚动条，只需将其放在一个滚动窗格中。

程序清单11-1展示了各种文本组件。这个程序显示了一个文本框、一个密码框和一个带滚动条的文本区。文本框和密码框都有标签。点击 "Insert" 会将输入框中的内容插入到文本区中。

[程序清单11-1 text/TextComponentFrame.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch11/text/TextComponentFrame.java)

注释：`JTextArea`只显示纯文本，没有特殊字体或格式。要显示带格式的文本（如HTML），可以使用`JEditorPane`。将在卷II中讨论。

## 11.4 选择组件
在很多情况下，更加愿意给用户一组有限的选项，而不是在文本组件中输入数据。在本节中，将介绍如何使用复选框、单选按钮、组合框以及滑块。

### 11.4.1 复选框
如果只想收集“是”或“否”的输入，就可以使用**复选框**(checkbox)组件。复选框自带标签作为标识。用户通过点击复选框将其选中，再次点击则取消选中。当复选框获得焦点时，按空格键也可以切换选中状态。

下图所示的简单程序有两个复选框，一个用于控制字体的斜体，另一个用于控制加粗。每次用户点击其中一个复选框时，就会使用新的字体属性刷新屏幕。

![复选框](/assets/images/java-note-v1ch11-user-interface-components-with-swing/复选框.png)

复选框需要一个紧邻的标签来标识其用途。在`JCheckBox`构造器中指定标签文本：

```java
var bold = new JCheckBox("Bold");
```

可以使用`setSelected()`和`isSelected()`方法设置或获取复选框的当前状态。

用户点击复选框时将触发一个动作事件(`ActionEvent`)。与按钮一样，可以为复选框关联动作监听器。在这个程序中，两个复选框使用了同一个动作监听器。

```java
ActionListener listener = ...;
bold.addActionListener(listener);
italic.addActionListener(listener);
```

监听器会查询`bold`和`italic`复选框的状态，并设置面板中的字体样式。

```java
ActionListener listener = event -> {
    int mode = 0;
    if (bold.isSelected()) mode += Font.BOLD;
    if (italic.isSelected()) mode += Font.ITALIC;
    label.setFont(new Font(Font.SERIF, mode, FONTSIZE));
};
```

注：也可以通过事件对象获得复选框是否被选中：`((JCheckBox) event.getSource()).isSelected()`。

程序清单11-2给出了复选框例子的代码。

[程序清单11-2 checkBox/CheckBoxFrame.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch11/checkBox/CheckBoxFrame.java)

### 11.4.2 单选按钮
复选框之间是相互独立的。在很多情况下，我们希望用户只选择几个选项当中的一个。选择另一项时，前一项就自动取消选择。这种选项通常称为**单选按钮**(radio button)，因为这很像收音机(radio)上的电台选择按钮——当按下一个按钮时，之前按下的按钮就会自动弹起。下图展示了一个典型的例子。这里允许用户在几个选项中选择字体大小：小、中、大或超大，当然每次只允许选择一种大小。

![单选按钮](/assets/images/java-note-v1ch11-user-interface-components-with-swing/单选按钮.png)

在Swing中实现单选按钮非常简单。为每组单选按钮构造一个`ButtonGroup`对象。然后，将`JRadioButton`对象添加到按钮组。按钮组对象负责在点击一个按钮时取消之前选中的按钮（注：实际上这个逻辑是在`JToggleButton.ToggleButtonModel.setSelected()`中实现的）。

```java
var group = new ButtonGroup();

var smallButton = new JRadioButton("Small", false);
group.add(smallButton);

var mediumButton = new JRadioButton("Medium", true);
group.add(mediumButton);
...
```

对于初始要选中的按钮，构造器的第二个参数为`true`。注意，按钮组只控制按钮的**行为**。如果出于**布局**目的将按钮分组，仍然需要将其添加到`JPanel`等容器中。

单选按钮的事件通知机制与其他按钮一样。当用户选中单选按钮时，将产生一个动作事件。在示例程序中，我们定义了一个动作监听器，将字体大小设置为特定值：

```java
ActionListener listener = event -> label.setFont(new Font("Serif", Font.PLAIN, size));
```

注：点击单选按钮一定是选中，因此不需要判断是否选中。

将这个监听器与复选框示例的监听器做一个对比。每个单选按钮都有一不同的监听器对象，而两个复选框共用同一个监听器。

对于单选按钮也可以使用同样的方法。可以用单个监听器来计算字体大小，如下：

```java
if (smallButton.isSelected()) size = 8;
else if (mediumButton.isSelected()) size = 12;
...
```

不过，我们更愿意使用单独的监听器对象，因为这样可以将字体大小的值与按钮更紧密地绑定在一起。

注释：要是能够快速知道组中的哪个按钮被选中就好了。`ButtonGroup`类有一个`getSelection()`方法，但是这个方法并不是返回被选中的单选按钮，而是那个按钮的模型(`ButtonModel`)。

程序清单11-3是用于选择字体大小的程序的完整代码。

[程序清单11-3 radioButton/RadioButtonFrame.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch11/radioButton/RadioButtonFrame.java)

### 11.4.3 边框
如果在一个窗口中有多组单选按钮，你可能希望以可见的方式指明哪些按钮属于一组。Swing为此提供了一组有用的**边框**(border)。最常见的用法是在面板周围放置一个边框，然后用其他组件（如单选按钮）填充面板。

有很多不同的边框可供选择，不过使用步骤完全一样。
1. 调用`BorderFactory`的静态方法创建边框。可以选择以下风格（如下图所示）：凹斜面(lowered bevel)、凸斜面(raised bevel)、蚀刻(etched)、直线(line)、磨砂(matte)、空(empty)。
2. 如果愿意，可以通过将边框传递给`BorderFactory.createTitledBorder()`来为边框添加标题。
3. 如果需要，可以通过调用`BorderFactory.createCompoundBorder()`组合多种边框。
4. 调用`JComponent`类的`setBorder()`方法将得到的边框添加到组件。

![测试边框类型](/assets/images/java-note-v1ch11-user-interface-components-with-swing/测试边框类型.png)

例如，可以如下为面板添加一个带标题的蚀刻边框：

```java
Border etched = BorderFactory.createEtchedBorder();
Border titled = BorderFactory.createTitledBorder(etched, "A Title");
panel.setBorder(titled);
```

[border/BorderFrame.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch11/border/BorderFrame.java)

不同的边框有不同的选项用于设置边框的宽度和颜色。另外，还有一个`SoftBevelBorder`类用于有柔和圆角的斜面边框，`LineBorder`类也有圆角。只能使用构造器来构造这些边框，没有相应的工厂方法。

### 11.4.4 组合框
如果有很多选项，使用单选按钮就不太适合了，因为它们会占据太多屏幕空间。此时可以使用组合框（即下拉列表）。当用户点击这个组件时，会下拉一个选项列表，用户可以从中选择一项（见下图）。

![组合框](/assets/images/java-note-v1ch11-user-interface-components-with-swing/组合框.png)

如果下拉列表框被设置成可编辑的，就可以编辑当前选项，就好像这是一个文本框一样。鉴于这个原因，这种组件被称为**组合框**(combo box)——它组合了文本框的灵活性与一组预定义的选项。`JComboBox`类提供了组合框组件。

从Java 7起，`JComboBox`是一个泛型类，`JComboBox<E>`包含`E`类型的选项。

调用`setEditable()`方法使组合框可编辑。注意，编辑只影响所选择的项，而不会改变选项列表。

可以调用`getSelectedItem()`方法获得当前选项（如果组合框是可编辑的，当前选项可能已经编辑过）。不过，对于可编辑组合框，选项可以是任何类型（返回`Object`），这取决于编辑器（接受用户输入并转换为一个对象）（注：默认编辑器调用`T.valueOf(String)`方法转换为对象，参见`BasicComboBoxEditor.getItem()`）。如果组合框是不可编辑的，最好调用

```java
combo.getItemAt(combo.getSelectedIndex())
```

这会返回具有正确类型的选择项。

在示例程序中，用户可以从字体列表中选择一种字体，也可以输入其他的字体。

可以使用`addItem()`方法添加选项：

```java
var faceCombo = new JComboBox<String>();
faceCombo.addItem("Serif");
faceCombo.addItem("SansSerif");
...
```

这个方法将选项添加到列表末尾。可以使用`insertItemAt()`方法在列表的任何位置插入新选项。

```java
faceCombo.insertItemAt("Monospaced", 0); // add at the beginning
```

选项可以是任何类型（`JComboBox`的类型变量），组合框会调用每个选项的`toString()`方法来显示。

如果需要在运行时删除选项，可以使用`removeItem()`或`removeItemAt()`方法。

```java
faceCombo.removeItem("Monospaced");
faceCombo.removeItemAt(0); // remove first item
```

`removeAllItems()`方法一次性删除所有选项。

提示：如果需要在组合框中添加大量选项，`addItem()`方法的性能会很差。应该构造一个`DefaultComboBoxModel`，调用`addElement()`填充选项，然后调用`JComboBox`类的`setModel()`方法。

注：也可以在`JComboBox`构造器中直接用数组指定选项。

当用户从组合框中选择一项时，组合框将产生一个动作事件。要找出选择了哪个选项，可以调用`((JComboBox) event.getSource()).getSelectedItem()`，需要将返回值强制转换为适当的类型（或者使用前面提到的`getItemAt()`和`getSelectedIndex()`）。

注：示例程序并不是通过这种方式获得组合框的引用，而是直接在监听器中访问外部变量`faceCombo`。

程序清单11-4给出了完整的程序。

[程序清单11-4 comboBox/ComboBoxFrame.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch11/comboBox/ComboBoxFrame.java)

### 11.4.5 滑块
组合框允许用户从一组离散值中进行选择。**滑块**(slide)则允许从连续值中选择，例如1~100的任意数值。

构造滑块最常用的方法如下：

```java
var slider = new JSlider(min, max, initialValue);
```

如果省略最小值、最大值和初始值，则分别默认为0、100和50。

如果需要垂直滑块，可以调用以下构造器：

```java
var slider = new JSlider(SwingConstants.VERTICAL, min, max, initialValue);
```

这些构造器会创建无格式的滑块（没有刻度和标签），如下图中最上面的滑块。下面来看如何为滑块添加装饰。

![滑块](/assets/images/java-note-v1ch11-user-interface-components-with-swing/滑块.png)

当用户滑动滑块时，滑块的值会在最小值和最大值之间变化。当值发生变化时，会产生一个`ChangeEvent`。为了得到改变的通知，需要调用`addChangeListener()`方法并提供一个实现了`ChangeListener`接口的对象。在回调方法中，可以使用`getValue()`获取滑块的值：

```java
ChangeListener listener = event -> {
    JSlider slider = (JSlider) event.getSource();
    int value = slider.getValue();
    ...
};
```

可以通过显示**刻度**(tick)对滑块进行修饰。例如，在示例程序中，第二个滑块使用了以下设置：

```java
slider.setMajorTickSpacing(20);
slider.setMinorTickSpacing(5);
```

这个滑块每20个单位显示一个大刻度标记，每5个单位显示一个小刻度标记。这里的单位是指滑块值，而不是像素。

要将刻度真正显示出来，还需要调用

```java
slider.setPaintTicks(true);
```

大小刻度标记是相互独立的。大刻度间距不必是小刻度间距的整数倍，但是这样看起来会显得非常凌乱。

可以强制滑块**对齐刻度**(snap to tick)。这样，当用户完成拖放时，滑块会立即移动到最近的刻度。激活这种模式需要调用

```java
slider.setSnapToTicks(true);
```

警告：“对齐刻度”的滑块在真正对齐之前，监听器仍然会报告与刻度不对应的值。

可以调用以下方法为大刻度显示标签：

```java
slider.setPaintLabels(true);
```

例如，对于一个范围为0~100、大刻度间距为20的滑块，刻度标签为0、20、40、60、80和100。

还可以提供其他刻度标签，如字符串或图标（见上图中最下面两个滑块）。这个过程有些繁琐。首先需要填充一个键为`Integer`类型、值为`Component`类型的散列表(`Hashtable`)。然后调用`setLabelTable()`方法，这些组件会放置在刻度下面。通常使用`JLabel`对象。如下将刻度标签设置为A、B、C、D、E和F。

```java
var labelTable = new Hashtable<Integer, Component>();
labelTable.put(0, new JLabel("A"));
labelTable.put(20, new JLabel("B"));
...
labelTable.put(100, new JLabel("F"));
slider.setLabelTable(labelTable);
```

提示：如果刻度标记或标签没有显示，再次确认是否调用了`setPaintTicks(true)`和`setPaintLabels(true)`。

上图中的第4个滑块没有轨道。要隐藏滑块的轨道，可以调用

```java
slider.setPaintTrack(false);
```

第5个滑块是逆向的，为此需要调用

```java
slider.setInverted(true);
```

程序清单11-5中的示例程序演示了不同视觉效果的滑块。每个滑块的监听器将当前值显示在窗体底部的文本框中。

[程序清单11-5 slider/SliderFrame.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch11/slider/SliderFrame.java)

## 11.5 菜单
Swing还支持另一种常见的用户界面元素——**菜单**(menu)。

位于窗口顶部的**菜单栏**(menu bar)包含各菜单的名字。点击一个名字会打开菜单，包含**菜单项**(menu item)和**子菜单**(submenu)。当用户点击一个菜单项时，所有菜单都会关闭，并产生一个动作事件。下图显示了一个典型的菜单。

![带子菜单的菜单](/assets/images/java-note-v1ch11-user-interface-components-with-swing/带子菜单的菜单.png)

### 11.5.1 创建菜单
创建菜单很容易。首先，创建一个菜单栏：

```java
var menuBar = new JMenuBar();
```

菜单栏是一个可以添加到任何位置的组件。通常放置在窗体顶部，可以调用`setJMenuBar()`方法（注意字母 "J" ）将其添加到那里：

```java
frame.setJMenuBar(menuBar);
```

为每个菜单创建一个菜单对象：

```java
var editMenu = new JMenu("Edit");
```

将顶层菜单添加到菜单栏：

```java
menuBar.add(editMenu);
```

向菜单中添加菜单项、分隔符和子菜单：

```java
var pasteItem = new JMenuItem("Paste");
editMenu.add(pasteItem);
editMenu.addSeparator();
JMenu optionsMenu = ...; // a submenu
editMenu.add(optionsMenu);
```

当用户选择一个菜单项时，将触发一个动作事件。需要为每个菜单项安装一个动作监听器：

```java
ActionListener listener = ...;
pasteItem.addActionListener(listener);
```

`JMenu.add(String s)`方法可以很方便地将一个菜单项添加到菜单末尾，并返回创建的菜单项，因此可以获取它并添加监听器，如下所示：

```java
JMenuItem pasteItem = editMenu.add("Paste"); // same as editMenu.add(new JMenuItem("Paste"))
pasteItem.addActionListener(listener);
```

通常情况下，菜单项触发的命令也可以通过其他组件（如工具栏按钮）激活。在10.4.5节中已经看到了如何通过`Action`对象来指定命令。`Action`对象也可用于创建菜单项：

```java
Action exitAction = ...;
JMenuItem exitItem = fileMenu.add(exitAction);
```

这会使用动作名称为菜单添加一个菜单项，这个动作对象将作为其监听器。这是以下语句的简写形式：

```java
var exitItem = new JMenuItem(exitAction);
fileMenu.add(exitItem);
```

### 11.5.2 菜单项中的图标
菜单项与按钮很相似。实际上，`JMenuItem`类扩展了`AbstractButton`类。与按钮一样，菜单项可以只有文本标签、只有图标，或二者都有。可以通过构造器或`setIcon()`方法指定图标，例如：

```java
var cutItem = new JMenuItem("Cut", new ImageIcon("cut.gif"));
```

在上图中，可以看到Cut、Copy和Paste菜单项旁边的图标。默认情况下，菜单项文本放在图标的右侧。如果喜欢将文本放在左侧，可以调用`setHorizontalTextPosition()`方法：

```java
cutItem.setHorizontalTextPosition(SwingConstants.LEFT);
```

也可以将图标添加到动作对象。当使用动作构造菜单项时，`Action.NAME`值将作为菜单项的文本，而`Action.SMALL_ICON`值将作为图标。

```java
cutAction.putValue(Action.NAME, "Cut");
cutAction.putValue(Action.SMALL_ICON, new ImageIcon("cut.gif"));
var cutItem = new JMenuItem(cutAction);
```

### 11.5.3 复选框和单选按钮菜单项
复选框和单选按钮菜单项会在文本旁边显示一个复选框或单选按钮（见上图中的Read-only、Insert和Overtype）。当用户选择这种菜单项时，就会自动切换选择状态。

例如，可以如下创建复选框菜单项：

```java
var readonlyItem = new JCheckBoxMenuItem("Read-only");
optionsMenu.add(readonlyItem);
```

单选按钮菜单项与普通单选按钮的工作方式一样，必须将它们添加到按钮组中。当组中的一个按钮被选中时，其他按钮就会自动取消选中。

```java
var group = new ButtonGroup();
var insertItem = new JRadioButtonMenuItem("Insert");
insertItem.setSelected(true);
var overtypeItem = new JRadioButtonMenuItem("Overtype");
group.add(insertItem);
group.add(overtypeItem);
optionsMenu.add(insertItem);
optionsMenu.add(overtypeItem);
```

使用这些菜单项时，不一定需要在用户选择时得到通知，而是稍后使用`isSelected()`方法来测试菜单项的当前状态（注：例如，打开文件时才需要检测是否是只读模式、插入还是覆写）（当然，这意味着应该保留这个菜单项的一个引用，保存在一个实例字段中）。可以使用`setSelected()`方法设置状态。

### 11.5.4 弹出菜单
**弹出菜单**(pop-up menu)是不固定在菜单栏中、随处浮动的菜单（如下图所示）。

![弹出菜单](/assets/images/java-note-v1ch11-user-interface-components-with-swing/弹出菜单.png)

创建弹出菜单与创建常规菜单类似，只是弹出菜单没有标题。

```java
var popup = new JPopupMenu();
```

然后用常规的方法添加菜单项：

```java
var item = new JMenuItem("Cut");
item.addActionListener(listener);
popup.add(item);
```

弹出菜单不像常规菜单栏那样始终显示在窗体顶部，必须调用`show()`方法显示，需要指定父组件和弹出位置坐标。例如：

```java
popup.show(panel, x, y);
```

通常，你希望用户点击鼠标右键时弹出菜单，这就是所谓的**弹出式触发器**(pop-up trigger)。为此，调用以下方法：

```java
component.setComponentPopupMenu(popup);
```

注：`JFrame`没有`setComponentPopupMenu()`方法，因此必须借助面板等组件实现“窗体右键菜单”的效果。

偶尔可能会把一个组件放在另一个有弹出菜单的组件中。通过调用以下方法，子组件可以继承父组件的弹出菜单：

```java
child.setInheritsPopupMenu(true);
```

### 11.5.5 键盘助记符和快捷键
对于有经验的用户来说，通过**键盘助记符**(keyboard mnemonic)选择菜单项确实非常方便。可以在菜单项构造器中指定一个助记字母来为菜单项创建键盘助记符：

```java
var aboutItem = new JMenuItem("About", 'A');
```

注：也可以使用`setMnemonic()`方法指定助记字母。助记字母可以用字符常量（如`'A'`）或`KeyEvent`中的常量（如`KeyEvent.VK_A`）表示。

键盘助记符会自动显示在菜单中，助记字母带有下划线（如下图所示）。在上面的示例中，菜单项文本将显示为 "<u>A</u>bout" 。当菜单打开时，用户只需要按下A键就可以选择这个菜单项。（如果助记字母不在菜单项文本中，按这个键仍然会选择菜单项，但助记符不会在菜单中显示。自然，这种不可见的助记符没有多大作用。）

![键盘助记符](/assets/images/java-note-v1ch11-user-interface-components-with-swing/键盘助记符.png)

有时候不希望将菜单项中第一个与助记符匹配的字母加下划线。例如， "Save <u>A</u>s" 而不是 "S<u>a</u>ve As" 。可以调用`setDisplayedMnemonicIndex()`方法指定希望对哪个字符加下划线。

如果有一个`Action`对象，可以添加助记符作为`Action.MNEMONIC_KEY`键的值。例如：

```java
aboutAction.putValue(Action.MNEMONIC_KEY, KeyEvent.VK_A);
```

菜单项构造器可以指定助记字母，但菜单构造器不能。要为菜单关联助记符，需要调用`setMnemonic()`方法：

```java
var helpMenu = new JMenu("Help");
helpMenu.setMnemonic('H');
```

要从菜单栏选择一个顶级菜单，可以同时按下Alt键和助记字母。例如，按下Alt+H选择Help菜单。

键盘助记符用于从当前打开的菜单中选择子菜单或菜单项。而**快捷键**(accelerator)可以在不打开菜单的情况下选择菜单项。例如，很多程序把快捷键Ctrl+O和Ctrl+S关联到File菜单中的Open和Save菜单项。可以使用`setAccelerator()`方法将快捷键关联到菜单项，这个方法接受一个`Keystroke`类型的对象（见10.4.5节）。例如，下面的调用将快捷键Ctrl+O关联到`openItem`菜单项：

```java
openItem.setAccelerator(KeyStroke.getKeyStroke("ctrl O"));
```

将快捷键添加到菜单项时，会自动在菜单中显示按键组合（见下图）。

![快捷键](/assets/images/java-note-v1ch11-user-interface-components-with-swing/快捷键.png)

### 11.5.6 启用和禁用菜单项
有时，某个菜单项应该只在特定的环境下才能选择。例如，当文件以只读方式打开时，Save菜单项就没有意义。此时可以禁用这个菜单项。被禁用的菜单项显示为灰色，不允许选择（见下图）。

![禁用的菜单项](/assets/images/java-note-v1ch11-user-interface-components-with-swing/禁用的菜单项.png)

启用或禁用菜单项需要调用`setEnabled()`方法：

```java
saveItem.setEnabled(false);
```

注：如果菜单项是使用动作对象创建的，也可以禁用动作对象。

启用和禁用菜单项有两种策略。每次环境发生变化时，可以对相关菜单项或动作调用`setEnabled()`（示例程序采用了这种方法）。例如，每当用户点击Read-only复选框菜单项时，就启用/禁用Save和Save As菜单项：

```java
readonlyItem.addActionListener(event -> {
    boolean saveOk = !readonlyItem.isSelected();
    saveAction.setEnabled(saveOk);
    saveAsAction.setEnabled(saveOk);
});
```

或者，可以在即将显示菜单之前（即用户点击了菜单栏上的菜单时）禁用菜单项。为此，必须为`Menu`对象注册“菜单选择”事件监听器。`MenuListener`接口的`menuSelected()`方法在菜单显示之前调用，因此可用于启用或禁用菜单项。

```java
fileMenu.addMenuListener(new MenuListener() {
    public void menuSelected(MenuEvent event) {
        boolean saveOk = !readonlyItem.isSelected();
        saveAction.setEnabled(saveOk);
        saveAsAction.setEnabled(saveOk);
    }
    ...
});
```

注：与第一种方法不同，这种方法在用户点击Read-only复选框菜单项时什么都不做，而是在打开File菜单时才判断其选中状态。

警告：在即将显示菜单之前禁用菜单项这种方式不适用于带有快捷键的菜单项。因为按下快捷键时并没有打开菜单，动作没有被禁用，因此快捷键仍然会触发这个动作。

程序清单11-6的示例程序创建了一组菜单。这个程序演示了本节介绍的所有特性：子菜单、禁用菜单项、复选框和单选按钮菜单项、弹出菜单以及键盘助记符和快捷键。

[程序清单11-6 menu/MenuFrame.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch11/menu/MenuFrame.java)

### 11.5.7 工具栏
**工具栏**(toolbar)是一个按钮栏，可以快速访问程序中最常用的命令（如下图所示）。

![工具栏](/assets/images/java-note-v1ch11-user-interface-components-with-swing/工具栏.png)

工具栏的特殊之处在于可以将它随处移动。可以将其拖动到窗体的四个边框上，如下图所示。

![拖动工具栏](/assets/images/java-note-v1ch11-user-interface-components-with-swing/拖动工具栏.png)

![将工具栏拖动到另一个边框](/assets/images/java-note-v1ch11-user-interface-components-with-swing/将工具栏拖动到另一个边框.png)

注释：只有当工具栏位于采用边框布局（或任何支持North、South、East和West约束的布局管理器）的容器内才能够拖动。

工具栏甚至可以完全脱离窗体。脱离的工具栏包含在独立的窗体中，如下图所示。关闭包含工具栏的独立窗体时，工具栏会回到原窗体中。

![脱离的工具栏](/assets/images/java-note-v1ch11-user-interface-components-with-swing/脱离的工具栏.png)

创建工具栏并添加组件很容易：

```java
var toolbar = new JToolBar();
toolbar.add(blueButton);
```

`JToolBar`类还有一个添加`Action`对象的`add()`方法，动作的小图标将会显示在工具栏中。

可以使用`addSeparator()`方法添加分隔线将按钮分组。

还可以在构造器中指定工具栏的标题，当工具栏脱离时就会显示这个标题：

```java
var toolbar = new JToolBar(titleString);
```

默认情况下，工具栏初始是水平的。如果希望工具栏初始是垂直的，使用

```java
var toolbar = new JToolBar(SwingConstants.VERTICAL);
```

按钮是工具栏中最常用的组件。不过对于可以添加到工具栏中的组件并没有限制。例如，可以在工具栏中添加组合框。

### 11.5.8 工具提示
工具栏有一个缺点是用户常常对工具栏中小图标的含义感到困惑。为了解决这个问题，设计者发明了**工具提示**(tooltip)。当鼠标在一个按钮上停留片刻时，就会显示工具提示，如下图所示（在10.4.5节已经看到过）。

![工具提示](/assets/images/java-note-v1ch11-user-interface-components-with-swing/工具提示.png)

在Swing中，可以调用`setToolTipText()`方法为任何组件添加工具提示：

```java
exitButton.setToolTipText("Exit");
```

或者，如果使用`Action`对象，可以将工具提示与`SHORT_DESCRIPTION`键关联：

```java
exitAction.putValue(Action.SHORT_DESCRIPTION, "Exit");
```

下面的示例程序演示了如何将一个`Action`对象添加到菜单和工具栏中。

[toolBar/ToolBarFrame.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch11/toolBar/ToolBarFrame.java)

## 11.6 复杂布局管理
到目前为止，在示例程序中我们只使用了边框布局、流布局和网格布局。对于更复杂的任务，只有这些还不够。

自从Java 1.0以来，AWT就包含**网格包布局**(grid bag layout)，这种布局按行和列排列组件。行和列的大小可以灵活改变，而且组件可以跨多行多列。这种布局管理器非常灵活，但也非常复杂。

在接下来的小节中，将介绍网格包布局。另外会介绍一种策略，在通常情况下可以让网格包布局使用相对简单些。最后，你会了解如何编写自己的布局管理器。

### 11.6.1 网格包布局
网格包布局是所有布局管理器之母。可以将网格包布局看成没有任何限制的网格布局。在网格包布局中，行和列的大小可以改变，可以合并相邻的单元格以容纳较大的组件。组件不需要填充整个单元格区域，而且可以指定它们在单元格内的对齐方式。

考虑下图所示的字体选择器，它由以下组件构成：
* 两个用于指定字体和字号的组合框
* 这两个组合框的标签
* 两个用于选择加粗和斜体的复选框
* 一个用于显示示例字符串的文本区

![字体选择器](/assets/images/java-note-v1ch11-user-interface-components-with-swing/字体选择器.png)

现在将容器分解成网格，如下图所示。（行和列的大小不必相同。）每个复选框横跨两列，文本区跨四行。

![设计中使用的网格](/assets/images/java-note-v1ch11-user-interface-components-with-swing/设计中使用的网格.png)

为了向网格包管理器描述这个布局，需要以下过程：
1. 创建一个`GridBagLayout`类型的对象。不需要指定底层网格的行数和列数，布局管理器会根据后面所给的信息猜测。
2. 将这个`GridBagLayout`对象设置为容器的布局管理器。
3. 对于每个组件，创建一个`GridBagConstraints`类型的对象。设置该对象的字段值来指定组件在网格包中如何摆放。
4. 最后，通过调用`add(component, constraints)`使用指定的约束添加各个组件。

下面是一个所需代码的示例（稍后将更加详细地介绍各种约束）。

```java
var layout = new GridBagLayout();
panel.setLayout(layout);
var constraints = new GridBagConstraints();
constraints.weightx = 100;
constraints.weighty = 100;
constraints.gridx = 0;
constraints.gridy = 2;
constraints.gridwidth = 2;
constraints.gridheight = 1;
panel.add(component, constraints);
```

关键是要知道如何设置`GridBagConstraints`对象的字段。

（1）`gridx`、`gridy`、`gridwidth`和`gridheight`

这些约束定义了组件在网格中的位置。`gridx`和`gridy`分别指定了组件左上角单元格的列和行，`gridwidth`和`gridheight`分别指定组件占据的列数和行数。

网格坐标从0开始，`gridx = 0, gridy = 0`表示左上角单元格。例如，示例程序中的文本区`gridx = 2, gridy = 0`，因为它起始于第0行第2列；其`gridwidth = 1, gridheight = 4`，因为它占据4行1列。

（2）`weightx`和`weighty`

当容器超过首选大小时，权重会指出按什么比例将空闲空间分配给各个单元格。问题是，权重是行和列的属性，而不是单个单元格，但网格包布局并不暴露行和列。行和列的权重计算为每行或每列中单元格权重的最大值。

如果将`weightx`/`weighty`设置为0，则在调整窗口大小时，组件宽度/高度始终保持不变。如果想让一行或一列的大小保持不变，就需要将这行或这列中所有组件的（对应方向的）权重都设置为0。如果不需要固定大小，建议将所有权重都设置为100。

（3）`fill`和`anchor`

如果不希望组件拉伸至填充整个单元格，就需要设置`fill`约束。有4个可能的取值：`NONE`、`HORIZONTAL`、`VERTICAL`和`BOTH`，定义在`GridBagConstraints`类中。

如果组件没有填充整个单元格，可以通过设置`anchor`字段指定它在单元格中的位置。可能的取值有`CENTER`（默认）、`NORTH`、`NORTHEAST`、`EAST`等。

（4）`insets`、`ipadx`和`ipady`

可以通过设置`insets`字段在组件周围增加额外的空白区域。设置`Insets`对象的`left`、`top`、`right`和`bottom`字段为组件周围的空间大小（单位为像素）。这称为**外边距**(external padding)。

`ipadx`和`ipady`指定**内边距**(internal padding)。这些值会增加到组件的最小宽度和高度上，以保证组件不会收缩至其最小尺寸以下。

（5）指定`gridx`、`gridy`、`gridwidth`和`gridheight`参数的替代方法

AWT文档建议不要将`gridx`和`gridy`设置为绝对位置，而应该设置为常量`GridBagConstraints.RELATIVE`。然后，按照标准的顺序添加组件，即从上到下、从左到右。

对于`gridwidth`和`gridheight`字段，如果组件扩展到最后一行或最后一列，就可以使用常量`GridBagConstraints.REMAINDER`。

（6）网格包布局技巧

在实际中，利用下面的技巧可以让网格包布局的使用相对容易一些：
1. 在纸上画出组件布局草图。
2. 找出一种网格，使得小组件分别包含在一个单元格中，较大的组件跨越多个单元格。
3. 用0, 1, 2...标记网格的行和列。现在可以得出`gridx`、`gridy`、`gridwidth`和`gridheight`的值。
4. 对于每个组件，考虑：是否需要水平或垂直填充它所在的单元格？如果不需要，希望如何对齐？这样就能得到`fill`和`anchor`参数的值。
5. 将所有的权重设置为100。不过，如果希望某行或某列始终保持默认大小，则将这行或这列中所有组件的`weighty`或`weightx`设置为0。
6. 编写代码。仔细地检查`GridBagConstraints`的设置。一个错误的约束可能会破坏整个布局。
7. 编译，运行。

（7）网格包约束辅助类

网格包布局最麻烦的方面就是编写设置约束的代码。大多数程序员会为此编写辅助函数或辅助类。下面给出一个辅助类。这个类有以下特性：
* 名字简短：`GBC`而不是`GridBagConstraints`。
* 扩展了`GridBagConstraints`，因此常量可以使用更短的名字，如`GBC.EAST`。
* 当添加组件时使用`GBC`对象，例如

  `add(component, new GBC(1, 2));`
* 有两个构造器用来设置最常用的参数：

  `new GBC(gridx, gridy)`和`new GBC(gridx, gridy, gridwidth, gridheight)`
* 对于x/y对形式的字段提供了方便的setter方法：

  `new GBC(1, 2).setWeight(100, 100)`
* setter方法返回`this`，因此可以链式调用：

  `new GBC(1, 2).setAnchor(GBC.EAST).setWeight(100, 100)`
* `setInsets()`方法将为你构造`Insets`对象。要得到1个像素的边距，只需调用

  `new GBC(1, 2).setAnchor(GBC.EAST).setInsets(1)`

注：实际上`GridBagConstraints`类有一个设置所有字段的构造器，但使用辅助类`GBC`会使代码可读性更好。

程序清单11-7展示了字体选择器示例的窗体类，程序清单11-8是`GBC`辅助类。

[程序清单11-7 gridbag/FontFrame.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch11/gridbag/FontFrame.java)

[程序清单11-8 gridbag/GBC.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch11/gridbag/GBC.java)

### 11.6.2 自定义布局管理器
你可以设计自己的布局管理器。作为一个有趣的例子，将容器中的所有组件摆成一个圆形，如下图所示。

![圆形布局](/assets/images/java-note-v1ch11-user-interface-components-with-swing/圆形布局.png)

自定义布局管理器必须实现`LayoutManager`接口，并实现下面5个方法：

```java
void addLayoutComponent(String name, Component c)
void removeLayoutComponent(Component c)
Dimension preferredLayoutSize(Container parent)
Dimension minimumLayoutSize(Container parent)
void layoutContainer(Container parent)
```

添加或删除组件时会调用前两个方法（例如，`JPanel.add()`会调用其布局管理器的`addLayoutComponent()`方法）。如果不需要保存组件的任何附加信息，可以让这两个方法什么都不做。接下来的两个方法计算组件的首选和最小布局所需要的空间。这二者通常是相等的。最后一个方法完成具体的工作，会调用所有组件的`setBounds()`方法。

注释：AWT还有一个接口`LayoutManager2`，其主要特点是允许使用带有约束的`add()`方法。例如，`BorderLayout`和`GridBagLayout`都实现了`LayoutManager2`接口。

程序清单11-9给出了`CircleLayout`管理器的代码，在父容器中沿着圆形摆放组件（没什么实用价值）。示例程序的窗体类见程序清单11-10。

[程序清单11-9 circleLayout/CircleLayout.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch11/circleLayout/CircleLayout.java)

[程序清单11-10 circleLayout/CircleLayoutFrame.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch11/circleLayout/CircleLayoutFrame.java)

## 11.7 对话框
在GUI应用中，通常希望弹出单独的对话框来向用户显示信息或者获取信息。

与大多数窗口系统一样，AWT也区分**模态**(modal)和**非模态**(modeless)对话框。模态对话框在结束处理之前不允许用户与应用的其余窗口交互。模态对话框用于需要先获取用户信息才能继续执行的情况。例如，用户想要打开一个文件时，可以弹出一个模态对话框，用户必须选择一个文件后程序才能继续执行。

非模态对话框允许用户同时在这个对话框和应用的其他部分输入信息。工具栏就是非模态对话框的一个例子。

### 11.7.1 选项对话框
Swing有一组现成的简单对话框，足以让用户提供一些信息。`JOptionPane`有4个静态方法来显示这些简单对话框：

| 方法 | 描述 |
| --- | --- |
| `showMessageDialog()` | 显示一条消息并等待用户点击OK |
| `showConfirmDialog()` | 显示一条消息并获取用户确认（如OK/Cancel） |
| `showOptionDialog()` | 显示一条消息并获取用户在一组选项中的选择 |
| `showInputDialog()` | 显示一条消息并获取用户输入的一行文本 |

下图显示了一个典型的对话框。可以看到，对话框有以下组件：
* 一个图标
* 一条消息
* 一个或多个选项按钮

![选项对话框](/assets/images/java-note-v1ch11-user-interface-components-with-swing/选项对话框.png)

输入对话框有一个额外的组件用于接收用户输入，可能是文本框或组合框。

图标取决于**消息类型**(message type)：

| 消息类型 | 图标 |
| --- | --- |
| `ERROR_MESSAGE` | ![error](/assets/images/java-note-v1ch11-user-interface-components-with-swing/error.png) |
| `INFORMATION_MESSAGE` | ![info](/assets/images/java-note-v1ch11-user-interface-components-with-swing/info.png) |
| `WARNING_MESSAGE` | ![warning](/assets/images/java-note-v1ch11-user-interface-components-with-swing/warning.png) |
| `QUESTION_MESSAGE` | ![question](/assets/images/java-note-v1ch11-user-interface-components-with-swing/question.png) |
| `PLAIN_MESSAGE` | 无 |

注：Swing使用的图标文件在java.desktop模块的javax/swing/plaf/metal/icons/ocean目录中，模块文件位于$jdk/jmods/java.desktop.jmod。参见 <https://github.com/openjdk/jdk/tree/jdk-17-ga/src/java.desktop/share/classes/javax/swing/plaf/metal/icons/ocean> 。

每种类型的对话框都可以提供自己的图标（通过`JOptionPane`静态方法的`icon`参数）。

消息可以是字符串、图标、用户界面组件，或任何其他对象（`message`参数是`Object`类型）。消息对象显示方式如下：

| 消息对象 | 显示方式 |
| --- | --- |
| `String` | 显示字符串 |
| `Icon` | 显示图标 |
| `Component` | 显示组件 |
| `Object[]` | 显示数组中的所有对象，堆叠起来 |
| 任何其他对象 | 调用`toString()`显示结果字符串 |

位于底部的按钮取决于对话框类型和**选项类型**(option type)。当调用`showMessageDialog()`和`showInputDialog()`时，只能得到一组标准按钮（分别是OK和OK/Cancel）。当调用`showConfirmDialog()`时，可以在下面四种选项类型中选择：

| 选项类型 | 按钮 |
| --- | --- |
| `DEFAULT_OPTION` | OK |
| `YES_NO_OPTION` | Yes/No |
| `YES_NO_CANCEL_OPTION` | Yes/No/Cancel |
| `OK_CANCEL_OPTION` | OK/Cancel |

使用`showOptionDialog()`时，可以指定一组任意的选项。需要提供一个对象数组作为选项，每个数组元素显示方式如下：

| 元素类型 | 显示方式 |
| --- | --- |
| `String` | 创建一个按钮，将字符串作为标签 |
| `Icon` | 创建一个按钮，将图标作为标签 |
| `Component` | 显示组件 |
| 任何其他对象 | 调用`toString()`，创建一个按钮，将结果字符串作为标签 |

这些静态方法的返回值如下：

| 方法 | 返回值 |
| --- | --- |
| `showMessageDialog()` | 无 |
| `showConfirmDialog()` | 表示所选选项的整数 |
| `showOptionDialog()` | 表示所选选项的整数 |
| `showInputDialog()` | 用户输入或选择的字符串 |

`showConfirmDialog()`和`showOptionDialog()`返回一个整数，表示用户选择了哪个按钮。对于选项对话框来说，就是所选选项的索引。如果用户没有选择而是关闭了对话框，则返回`CLOSED_OPTION`。对于确认对话框，返回值可以是下列值之一：

```java
OK_OPTION
CANCEL_OPTION
YES_OPTION
NO_OPTION
CLOSED_OPTION
```

这些选择看起来让人眼花缭乱，但实际上很简单。遵循以下步骤：
1. 选择对话框类型（消息、确认、选项或者输入）。
2. 选择图标（错误、信息、警告、问题、无或者自定义）。
3. 选择消息（字符串、图标、自定义组件或者它们的组合）。
4. 对于确认对话框，选择选项类型（OK、Yes/No、Yes/No/Cancel或者OK/Cancel）。
5. 对于选项对话框，选择选项（字符串、图标或自定义组件）和默认选项。
6. 对于输入对话框，选择文本框或者组合框。
7. 调用`JOptionPane`中的相应方法。

例如，假设要显示上面图中的对话框，则调用如下：

```java
int selection = JOptionPane.showConfirmDialog(
    parent, "Message", "Title",
    JOptionPane.OK_CANCEL_OPTION,
    JOptionPane.QUESTION_MESSAGE);
if (selection == JOptionPane.OK_OPTION) ...
```

提示：消息字符串可以包含换行符，这样字符串会多行显示。

示例程序显示了6个按钮面板（如下图所示）。点击Show按钮时，会显示所选的对话框。

![对话框示例程序](/assets/images/java-note-v1ch11-user-interface-components-with-swing/对话框示例程序.png)

[optionDialog/OptionDialogFrame.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch11/optionDialog/OptionDialogFrame.java)

### 11.7.2 自定义对话框
在上一节中，已经了解了如何使用`JOptionPane`类来显示简单对话框。本节将介绍如何手动创建这样一个对话框。

下图显示了一个典型的“关于”对话框。当用户点击About按钮时就会显示这样一个程序信息对话框。

![About对话框](/assets/images/java-note-v1ch11-user-interface-components-with-swing/About对话框.png)

要实现一个对话框，需要扩展`JDialog`类，这与扩展`JFrame`的过程基本是一样的。具体过程如下：
1. 在构造器中，调用超类`JDialog`的构造器。
2. 添加对话框的用户界面组件。
3. 添加事件处理器。
4. 设置对话框的大小。

在调用超类构造器时，需要提供拥有者窗体(owner frame)、对话框标题以及模态(modality)。

下面是“关于”对话框的代码：

```java
public AboutDialog extends JDialog {
    public AboutDialog(JFrame owner) {
        super(owner, "About DialogTest", true);
        add(
            new JLabel("<html><h1><i>Core Java</i></h1><hr>By Cay Horstmann</html>"),
            BorderLayout.CENTER);
        var panel = new JPanel();
        var ok = new JButton("OK");

        ok.addActionListener(event -> setVisible(false));
        panel.add(ok);
        add(panel, BorderLayout.SOUTH);
        setSize(250, 150);
    }
}
```

要显示对话框，需要创建一个新的对话框对象，并使其可见。实际上，在示例程序中只创建了一次对话框，每当用户点击About按钮时可以复用它。

```java
if (dialog == null) // first time
    dialog = new AboutDialog(this);
dialog.setVisible(true);
```

当用户点击OK按钮时，对话框会隐藏。当用户点击右上角的关闭按钮时，对话框也会隐藏。与`JFrame`一样，可以使用`setDefaultCloseOperation()`方法覆盖这个行为。

程序清单11-11是测试程序窗体类的代码，程序清单11-12是对话框类。

[程序清单11-11 dialog/DialogFrame.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch11/dialog/DialogFrame.java)

[程序清单11-12 dialog/AboutDialog.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch11/dialog/AboutDialog.java)

### 11.7.3 数据交换
使用对话框最常见的原因是获取用户输入的信息。下面来看如何将数据传入/传出对话框。

考虑下图中的登录对话框，这个对话框用于获得用户名和密码来连接某个在线服务。

![登录对话框](/assets/images/java-note-v1ch11-user-interface-components-with-swing/登录对话框.png)

你的对话框应该提供设置默认数据的方法。例如，示例程序中的`PasswordChooser`类提供了一个`setUser()`方法来设置默认用户名。（注：示例程序中所谓的默认用户名是 "yourname" ，实际上用于充当占位符。而输入框旁边的标签已经起到了这个作用，因此这里设置默认用户名并没有什么意义。）

之后调用`setVisible(true)`显示对话框。有一点很重要：在用户关闭这个对话框（点击OK、Cancel或关闭按钮）之前，`setVisible(true)`调用会一直阻塞。这样就能很容易地实现模态对话框。

你希望知道用户是接受还是取消了这个对话框（注：即输入数据并点击了OK按钮，还是点击了Cancel按钮）。示例程序在显示对话框之前将`ok`标志设置为`false`，只有OK按钮的事件处理器将其设置为`true`。这就是从对话框获取用户输入的方式。

注释：从非模态对话框传输数据就没有这么简单了。显示非模态对话框时，`setVisible(true)`调用并不阻塞，程序会继续运行。如果用户输入了数据然后点击OK，对话框需要向程序中的某个监听器发送一个事件（注：即通过回调的方式异步获取数据）。

示例程序还包含另一个有用的改进。构造`JDialog`对象时，需要指定拥有者窗体。但是，在很多情况下，你希望在不同窗体中显示同一个对话框。所以最好在准备显示对话框时再选择拥有者窗体，而不是在构造对话框对象时。

这里的技巧是让`PasswordChooser`扩展`JPanel`而不是`JDialog`。在`showDialog()`方法中动态(on the fly)构建一个`JDialog`对象：

```java
public boolean showDialog(Frame owner, String title) {
    ok = false;
    if (dialog == null || dialog.getOwner() != owner) {
        dialog = new JDialog(owner, true);
        dialog.add(this);
        dialog.pack();
    }
    dialog.setTitle(title);
    dialog.setVisible(true);
    return ok;
}
```

注意，让`owner`等于`null`是安全的。

注：也就是说，`PasswordChooser`是一个组件而不是对话框，其`showDialog()`方法构造一个`JDialog`，并将自身添加到对话框中。

还可以做得更好。有时，拥有者窗体并不容易获得。不过可以很容易地从任意`parent`组件得出，如下所示：

```java
Frame owner;
if (parent instanceof Frame)
    owner = (Frame) parent;
else
    owner = (Frame) SwingUtilities.getAncestorOfClass(Frame.class, parent);
```

`JOptionPane`类也使用了这种机制（参见`JOptionPane.getRootFrame()`）。

很多对话框都有一个**默认按钮**，如果用户按下Enter键就会自动选择它。默认按钮有特殊的标记，通常有加粗的轮廓。

可以在对话框的**根窗格**(root pane)中设置默认按钮：

```java
dialog.getRootPane().setDefaultButton(okButton);
```

如果遵循前面的建议，就必须注意，只有将面板包装进对话框后才能设置默认按钮。面板本身没有根窗格。

示例程序展示了如何将数据传入/传出对话框（程序清单11-1的对话框版本）。程序清单11-13是窗体类，程序清单11-14是“对话框”类。

[程序清单11-13 dataExchange/DataExchangeFrame.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch11/dataExchange/DataExchangeFrame.java)

[程序清单11-14 dataExchange/PasswordChooser.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch11/dataExchange/PasswordChooser.java)

### 11.7.4 文件对话框
在一个应用中，用户通常希望可以打开和保存文件。幸运的是，Swing提供了`JFileChooser`类来显示文件对话框，如下图所示。文件对话框总是模态的。注意，`JFileChooser`类并不是`JDialog`的子类。需要调用`showOpenDialog()`显示打开文件的对话框，或者调用`showSaveDialog()`显示保存文件的对话框，而不是调用`setVisible(true)`。确认按钮会自动地使用标签Open或Save，也可以使用`showDialog()`方法提供自己的按钮标签。

![文件选择对话框](/assets/images/java-note-v1ch11-user-interface-components-with-swing/文件选择对话框.png)

下面是创建文件对话框并获取用户选择的步骤：

1.创建一个`JFileChooser`对象。与`JDialog`类的构造器不同，不需要提供父组件。这允许你在多个窗体中复用一个文件选择对话框。例如：

```java
var chooser = new JFileChooser();
```

2.调用`setCurrentDirectory()`方法设置目录。例如，要使用当前工作目录：

```java
chooser.setCurrentDirectory(new File("."));
```

3.如果有一个希望用户选择的默认文件名，则使用`setSelectedFile()`方法指定：

```java
chooser.setSelectedFile(new File(filename));
```

4.如果允许用户选择多个文件，则调用`setMultiSelectionEnabled()`方法。这是可选的，而且并不常见。

```java
chooser.setMultiSelectionEnabled(true);
```

5.如果要限制对话框显示的文件类型（例如所有扩展名为.gif的文件），需要设置**文件过滤器**(file filter)，本节稍后将会讨论。

6.默认情况下，用户只能选择文件。如果希望用户选择目录，则使用`setFileSelectionMode()`方法，参数可以是`FILES_ONLY`（默认）、`DIRECTORIES_ONLY`或`FILES_AND_DIRECTORIES`。

7.调用`showOpenDialog()`或`showSaveDialog()`方法显示对话框。在这些调用中必须提供父组件，例如：

```java
int result = chooser.showOpenDialog(parent);
```

这两个方法唯一的区别是“确认按钮”的标签不同。也可以调用`showDialog()`方法提供确认按钮的文本：

```java
int result = chooser.showDialog(parent, "Select");
```

只有当用户确认、取消或者关闭对话框时这些调用才会返回。返回值可以是`APPROVE_OPTION`、`CANCEL_OPTION`或`ERROR_OPTION`。

8.使用`getSelectedFile()`或`getSelectedFiles()`方法获得用户选择的一个或多个文件。这些方法返回`File`或`File[]`。如果只需要知道文件名，可以调用`getPath()`方法。例如：

```java
String filename = chooser.getSelectedFile().getPath();
```

大多数情况下，这些步骤都很简单。使用文件对话框的主要困难在于指定用户应该选择的文件子集。例如，假设用户应该选择GIF图像文件，文件选择器就应该只显示扩展名为.gif的文件；如果用户应该选择JPEG图像文件，扩展名就可以是.jpg或.jpeg。要限制所显示的文件，可以提供一个扩展了抽象类`javax.swing.filechooser.FileFilter`的对象。

目前提供了两个子类：一个是接受所有文件的默认过滤器(`AcceptAllFileFilter`)，另一个过滤器接受具有给定扩展名的文件(`FileNameExtensionFilter`)。不过，很容易编写专用的文件过滤器，只需实现`FileFilter`超类中的两个抽象方法：

```java
public boolean accept(File f);
public String getDescription();
```

第一个方法检测是否应该接受一个文件，第二个方法返回文件类型的描述。

一旦有了文件过滤器对象，就可以调用`JFileChooser`类的`setFileFilter()`方法将其安装到文件选择器对象中：

```java
chooser.setFileFilter(new FileNameExtensionFilter("Image files", "gif", "jpg"));
```

可以多次调用`addChoosableFileFilter()`为文件选择器安装多个过滤器。

用户可以从文件对话框底部的组合框中选择过滤器。默认情况下，组合框中总是有 "All files" 过滤器。如果想禁用它，需要调用`chooser.setAcceptAllFileFilterUsed(false)`。

警告：如果复用一个文件选择器来打开和保存不同类型的文件，需要在添加新的过滤器之前调用`resetChoosableFilters()`。

可以为文件选择器显示的每个文件提供特定的图标和描述来定制文件选择器。为此，需要提供一个扩展了`FileView`类的对象，然后使用`setFileView()`方法将其安装到文件选择器中。示例程序包含了一个简单的文件视图类`FileIconView`，这个类为所有图像文件显示一个调色板图标。

最后，可以添加一个**配件**(accessory)组件来定制文件选择器。例如，下图所示的预览配件显示了当前选中文件的缩略图。

![带预览配件的文件对话框](/assets/images/java-note-v1ch11-user-interface-components-with-swing/带预览配件的文件对话框.png)

配件可以是任何Swing组件。在这个示例中，`ImagePreviewer`扩展了`JLabel`类，并将其图标设置为图像文件的缩小副本。

为了在用户选择不同的文件时更新预览图像，可以通过`addPropertyChangeListener()`方法安装属性变化监听器。

示例程序对第2章中的ImageViewer程序做了一定的修改，通过自定义的文件视图和预览配件增强了文件选择器的功能。

[fileChooser/ImageViewerFrame.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch11/fileChooser/ImageViewerFrame.java)

[fileChooser/ImagePreviewer.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch11/fileChooser/ImagePreviewer.java)

[fileChooser/FileIconView.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch11/fileChooser/FileIconView.java)
