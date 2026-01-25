---
title: 《Java核心技术》笔记 卷II 第11章 高级Swing和图形
date: 2025-12-09 23:40:44 +0800
categories: [Java, Core Java]
tags: [java, gui, swing, image processing]
math: true
---
在本章中，继续对卷I的Swing工具包和AWT图形进行讨论。

## 11.1 表格
`JTable`组件用于显示二维表格。在接下来几节中，将介绍如何制作简单的表格，用户如何与其交互，以及如何进行一些最常见的调整。

### 11.1.1 一个简单表格
`JTable`并不存储数据，而是从**表格模型**(table model)中获取（参见卷I第11章 11.1节）。`JTable`类有一个构造器能够将二维对象数组包装进一个默认模型。

下图展示了一个典型的表格，用于描述太阳系各个行星的属性。

![一个简单表格](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/一个简单表格.png)

在程序清单11-1中可以看到，表格的数据是以`Object`二维数组的形式存储的：

```java
Object[][] cells = {
    {"Mercury", 2440.0, 0, false, Color.YELLOW},
    {"Venus", 6052.0, 0, false, Color.YELLOW},
    ...
}
```

注释：这里利用了自动装箱，第二、三、四列中的项会自动转换成`Double`、`Integer`和`Boolean`类型的对象。

该表格直接调用了每个对象的`toString()`方法来显示它们，这也是颜色显示为`java.awt.Color[r=...,g=...,b=...]`的原因。

用一个单独的字符串数组提供列名：

```java
String[] columnNames = {"Planet", "Radius", "Moons", "Gaseous", "Color"};
```

然后使用单元格和列名数组构造一个表格：

```java
var table = new JTable(cells, columnNames);
```

可以通过常用方式——将表格包装到一个`JScrollPane`中来添加滚动条：

```java
var pane = new JScrollPane(table);
```

在滚动表格时，表头不会滑到视图之外。

点击某一列的表头并拖动可以将这一列移动到其他位置（见下图）。这只会在视图上重新排列，对数据模型没有影响。

![移动一列](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/移动一列.png)

要调整列的大小，只需将鼠标移动到两列之间并拖动边界（见下图）。

![调整列大小](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/调整列大小.png)

用户可以通过点击行中的任何地方来选中一行，选中的行会高亮显示。还可以通过单击单元格并键入数据来编辑单元格。但是，在本节的示例中，编辑并不会改变底层数据。在程序中，应该要么使单元格不可编辑，要么处理单元格编辑事件并更新模型。将在本节的后面对这些问题进行讨论。

点击一列的表头，行会自动按这一列排序。再次点击，排序顺序就会反过来。这个行为通过调用`table.setAutoCreateRowSorter(true)`激活。

注：`JTable`默认按**字典序**进行排序。例如，按Moon一列降序排序的结果是8, 2, 18, 17, ...

最后，可以使用`table.print()`来打印表格。

警告：如果没有将表格包装在滚动面板中，就需要显式地添加表头：

```java
add(table.getTableHeader(), BorderLayout.NORTH);
```

[程序清单11-1 table/PlanetTableFrame.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch11/table/PlanetTableFrame.java)

### 11.1.2 表格模型
在上一个示例中，表格数据存储在一个二维数组中。不过，通常不应该在自己的代码中使用这种策略。应该考虑实现自己的表格模型。

表格模型实现起来特别简单，因为可以利用`AbstractTableModel`类，它实现了大部分必需的方法。你只需提供以下三个方法：

```java
public int getRowCount();
public int getColumnCount();
public Object getValueAt(int row, int column);
```

实现`getValueAt()`有很多方式。例如，如果想显示包含数据库查询结果的`RowSet`的内容，可以这样实现：

```java
public Object getValueAt(int r, int c) {
    try {
        rowSet.absolute(r + 1);
        return rowSet.getObject(c + 1);
    }
    catch (SQLException e) {
        e.printStackTrace();
        return null;
    }
}
```

示例程序更简单。我们构建了一个用来展示计算结果的表格，即不同利率下的投资增长额（如下图所示）。

![投资增长额](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/投资增长额.png)

`getValueAt()`方法计算给定年份和利率对应的金额并格式化（行号对应年份，列号对应利率）：

```java
public Object getValueAt(int r, int c) {
    double rate = (c + minRate) / 100.0;
    int nperiods = r;
    double futureBalance = INITIAL_BALANCE * Math.pow(1 + rate, nperiods);
    return "%.2f".formatted(futureBalance);
}
```

`getRowCount()`和`getColumnCount()`返回行数和列数：

```java
public int getRowCount() { return years; }
public int getColumnCount() { return maxRate - minRate + 1; }
```

如果不提供列名，`AbstractTableModel`会将列命名为A、B、C等。要改变默认的列名，需要覆盖`getColumnName()`方法。在这个示例中，列名为利率。

```java
public String getColumnName(int c) { return (c + minRate) + "%"; }
```

程序清单11-2给出了完整的源代码。

[程序清单11-2 tableModel/InvestmentTableModel.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch11/tableModel/InvestmentTableModel.java)

### 11.1.3 处理行和列
这个小节将介绍如何操作表格中的行和列。Swing表格是“不对称的”——可以对行和列执行的操作是不同的。这是因为表格组件是为了显示具有相同结构的行优化的，而不是任意的二维表格。

#### 11.1.3.1 列类型
下一个示例将再次展示行星数据，但这次会给出更多有关表格列类型的信息。这是通过覆盖表格模型的`getColumnClass()`方法实现的，该方法返回描述列类型的`Class`对象。

`JTable`会为该类选取合适的绘制器，下表显示了默认绘制动作（注：参见`JTable.getDefaultRenderer()`方法）。

| 类型 | 绘制结果 |
| --- | --- |
| `Boolean` | 复选框 |
| `Icon` | 图像 |
| `Object` | 字符串 |

可以在下图中看到复选框（Gaseous列）和图像（Image列）（行星图像来自[Jim
Evins](https://web.archive.org/web/20150418144913/http://snaught.com/JimsCoolIcons/Planets/)）。

![行星数据表格](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/行星数据表格.png)

要绘制其他类型，可以安装自定义绘制器，参见11.1.4节。

#### 11.1.3.2 访问表格列
`JTable`类将有关列的信息存放在`TableColumn`类型的对象中，由`TableColumnModel`对象负责管理这些列。下图展示了最重要的表格类之间的关系。

![表格类之间的关系图](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/表格类之间的关系图.png)

如果不想动态地插入或删除列，就不会经常用到列模型。列模型最常见的用法是获取一个`TableColumn`对象：

```java
int columnIndex = ...;
TableColumn column = table.getColumnModel().getColumn(columnIndex);
```

#### 11.1.3.3 改变列大小
`TableColumn`可以控制改变列大小的行为。可以分别使用`setPreferredWidth()`、`setMinWidth()`和`setMaxWidth()`设置首选、最小和最大宽度。使用`setResizable()`方法控制是否允许用户改变列大小。可以使用`setWidth()`方法在程序中改变列的大小。

改变一列的大小时，默认情况下表格的总宽度保持不变，而改变右侧所有列的大小。这使得用户可以从左到右将所有列调整为期望的宽度。可以使用`JTable`类的`setAutoResizeMode()`方法来设置其他行为，如下表所示。

| 模式 | 行为 |
| --- | --- |
| `AUTO_RESIZE_OFF` | 不改变其他列的大小，而是改变表格宽度 |
| `AUTO_RESIZE_NEXT_COLUMN` | 只改变下一列的大小 |
| `AUTO_RESIZE_SUBSEQUENT_COLUMNS` | 均匀地改变后续所有列的大小，默认行为 |
| `AUTO_RESIZE_LAST_COLUMN` | 只改变最后一列的大小 |
| `AUTO_RESIZE_ALL_COLUMNS` | 改变所有列的大小 |

#### 11.1.3.4 改变行大小
行的高度是直接由`JTable`类管理的。如果单元格比默认值高，可以像这样设置行高：`table.setRowHeight(height)`。

默认情况下，表格的所有行具有相同的高度。可以使用`table.setRowHeight(row, height)`设置单个行的高度。

行的实际高度等于设置的高度减去行边距。默认行边距是1像素，可以使用`table.setRowMargin(margin)`来修改。

#### 11.1.3.5 选择行、列和单元格
根据选择模式，用户可以选择表格中的行、列或单元格。默认情况下是行选择，点击一个单元格内部就会选择整行（见11.1.3.1节中的图）。调用`table.setRowSelectionAllowed(false)`可以禁用行选择。

当启用行选择时，可以控制是否允许用户选择单一行、连续几行或者任意几行。为此，需要获取**选择模型**(selection model)并使用其`setSelectionMode()`方法：

```java
table.getSelectionModel().setSelectionMode(mode);
```

其中，`mode`是`ListSelectionModel`类的常量`SINGLE_SELECTION`、`SINGLE_INTERVAL_SELECTION`或`MULTIPLE_INTERVAL_SELECTION`之一。

默认情况下，列选择是禁用的。可以通过调用`table.setColumnSelectionAllowed(true)`开启。

同时启用行选择和列选择等价于启用单元格选择，这样用户就可以选择一个范围内的单元格（见下图）。也可以通过调用`table.setCellSelectionEnabled(true)`开启单元格选择。

![选择一个范围内的单元格](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/选择一个范围内的单元格.png)

可以运行程序清单11-3中的程序来观察单元格选择的效果。在Selection菜单中分别启用行、列和单元格选择，然后观察选择行为是如何改变的。

可以通过调用`getSelectedRows()`和`getSelectedColumns()`方法来查看哪些行和列被选中了。这两个方法都返回被选择项的索引构成的数组。注意，这些索引值是表格视图的，而不是底层表格模型的。尝试选中一些行和列，然后将列拖拽到不同位置，并点击列头来对行排序。使用Edit - Print Selection菜单项来查看它会报告哪些行和列被选中。

如果需要将视图索引值转换为模型索引值，可以使用`JTable`的`convertRowIndexToModel()`和`convertColumnIndexToModel()`方法。

#### 11.1.3.6 排序行
正如在第一个表格示例中看到的，向`JTable`添加行排序是很容易的，只需调用`setAutoCreateRowSorter(true)`。但是，要对排序行为进行细粒度的控制，需要安装一个`TableRowSorter<M>`对象并对其进行定制。类型参数`M`表示表格模型，必须是`TableModel`接口的子类型。

```java
var sorter = new TableRowSorter<TableModel>(model);
table.setRowSorter(sorter);
```

有些列不应该是可排序的，例如行星数据中的图像列。可以通过调用`sorter.setSortable(IMAGE_COLUMN, false)`来关闭排序。

可以对每一列安装一个自定义比较器。在我们的示例中，通过依次比较蓝、绿、红来对Color列中的颜色进行排序（例如：RGB(255,255,128) < RGB(0,0,255), RGB(255,128,255) < RGB(0,255,255)）。当点击Color列时，将会看到蓝色行星出现在表格底部。这是通过以下调用完成的：

```java
sorter.setComparator(COLOR_COLUMN, new Comparator<Color>() {
    public int compare(Color c1, Color c2) {
        int d = c1.getBlue() - c2.getBlue();
        if (d != 0) return d;
        d = c1.getGreen() - c2.getGreen();
        if (d != 0) return d;
        return c1.getRed() - c2.getRed();
    }
});
```

注：这个比较器等价于`Comparator.comparing(Color::getBlue).thenComparing(Color::getGreen).thenComparing(Color::getRed)`。

如果不指定列的比较器，排序顺序按如下方式确定：

1.如果列类型是`String`，则使用`Collator.getInstance()`返回的默认比较器。它按照适用于当前locale的方式对字符串排序（详见第7章 7.4节）。

2.如果列类型实现了`Comparable`，则使用其`compareTo()`方法。

3.如果为排序器设置了`TableStringConverter`，则使用默认比较器对转换器的`toString()`方法返回的字符串进行排序。可以像这样设置转换器：

```java
sorter.setStringConverter(new TableStringConverter() {
    public String toString(TableModel model, int row, int column) {
        Object value = model.getValueAt(row, column);
        // convert value to a string and return it
    }
});
```

4.否则，对单元格的值调用`toString()`方法，并使用默认比较器对其进行排序。

#### 11.1.3.7 过滤行
除了排序，`TableRowSorter`还可以对行进行过滤。为此需要设置`RowFilter`。例如，要筛选所有至少有一个卫星的行星，调用

```java
sorter.setRowFilter(RowFilter.numberFilter(ComparisonType.NOT_EQUAL, 0, MOONS_COLUMN));
```

这里使用了预定义的数字过滤器。要构造数字过滤器，需要提供：
* 比较类型（`EQUAL`、`NOT_EQUAL`、`AFTER`或`BEFORE`之一）。
* 一个`Number`的子类（例如`Integer`或`Double`）的对象。只有相同类型的对象才在考虑范围内。
* 零个或多个列索引。如果指定多个，则任何一列满足条件时该行就会被保留。如果未指定则所有列都被搜索。

静态方法`RowFilter.dateFilter()`以同样的方式构造日期过滤器，只是需要提供`Date`对象而不是`Number`对象。

静态方法`RowFilter.regexFilter()`构造的过滤器查找匹配某个正则表达式的字符串。例如，

```java
sorter.setRowFilter(RowFilter.regexFilter(".*[^s]$", PLANET_COLUMN));
```

只显示名字不以 "s" 结尾的行星。

还可以用`andFilter()`、`orFilter()`和`notFilter()`方法组合过滤器。例如，要过滤名字不以 "s" 结尾并且至少有一颗卫星的行星，可以使用以下过滤器组合：

```java
sorter.setRowFilter(RowFilter.andFilter(List.of(
    RowFilter.regexFilter(".*[^s]$", PLANET_COLUMN),
    RowFilter.numberFilter(ComparisonType.NOT_EQUAL, 0, MOONS_COLUMN))));
```

要实现自己的过滤器，需要提供一个`RowFilter`的子类，并实现`include()`方法来指示应该显示的行。`RowFilter<M, I>`类有两个类型参数——模型类型和行标识符类型。在处理表格时，模型是`TableModel`的子类型，标识符类型是`Integer`。

行过滤器必须实现以下方法：

```java
public boolean include(RowFilter.Entry<? extends M, ? extends I> entry)
```

`RowFilter.Entry`类提供了获取模型、行标识符和给定索引处的值的方法。因此，可以同时按照行的标识符和内容进行过滤。

例如，下面的过滤器将隔行显示：

```java
var filter = new RowFilter<TableModel, Integer>() {
    public boolean include(Entry<? extends TableModel, ? extends Integer> entry) {
        return entry.getIdentifier() % 2 == 0;
    }
};
```

如果想只显示具有偶数个卫星的行星，测试条件应为`((Integer) entry.getValue(MOONS_COLUMN)) % 2 == 0`。

在示例程序中，我们允许用户隐藏任意的行。我们将隐藏行的索引存储在一个集合中，行过滤器将展示所有索引不在这个集合中的行。

过滤机制并不是为条件随时间变化的过滤器而设计的。因此在示例程序中，每当隐藏行的集合发生变化，就需要调用`sorter.setRowFilter(filter)`。

设置过滤器后会立即应用。

#### 11.1.3.8 隐藏和显示列
正如在前一节中看到的，可以根据内容或标识符来过滤表格行。而隐藏表格列使用的是完全不同的机制。

`JTable`类的`removeColumn()`方法从表格视图中删除一列。该列的数据实际上并没有从模型中删除，只是从视图中隐藏了。该方法接受一个`TableColumn`参数。如果你有的是列号（例如来自`getSelectedColumns()`调用），就需要从表格模型获取列对象：

```java
TableColumnModel columnModel = table.getColumnModel();
TableColumn column = columnModel.getColumn(i);
table.removeColumn(column);
```

如果保存了列对象，稍后可以再把它添加回去：`table.addColumn(column)`。该方法将列添加到表格末尾。如果想让它出现在其他地方，需要调用`moveColumn()`方法。

还可以通过添加一个新的`TableColumn`对象来添加一个对应表格模型中列索引的新列：

```java
table.addColumn(new TableColumn(modelColumnIndex));
```

可以让多个表格列展示模型中的同一列。

程序清单11-3中的程序演示了行和列的选择与过滤。

[程序清单11-3 tableRowColumn/PlanetTableFrame.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch11/tableRowColumn/PlanetTableFrame.java)

### 11.1.4 单元格的绘制和编辑
在11.1.3.1节中已经看到，列类型决定了单元格如何绘制。`Boolean`和`Icon`类型有默认的绘制器，分别绘制复选框和图标。而对于其他类型，需要安装自定义绘制器。

#### 11.1.4.1 绘制单元格
表格的单元格绘制器实现了`TableCellRenderer`接口，只有一个方法：

```java
Component getTableCellRendererComponent(JTable table, Object value,
    boolean isSelected, boolean hasFocus, int row, int column)
```

该方法在表格需要绘制单元格时被调用，返回一个组件，然后其`paint()`方法会被调用以填充单元格区域。

下图中的表格包含`Color`类型的单元格。绘制器直接返回一个面板，其背景色为存储在该单元格中的颜色对象（作为`value`参数传递）。

![具有单元格绘制器的表格](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/具有单元格绘制器的表格.png)

```java
class ColorTableCellRenderer extends JPanel implements TableCellRenderer {
    public Component getTableCellRendererComponent(JTable table, Object value,
            boolean isSelected, boolean hasFocus, int row, int column) {
        setBackground((Color) value);
        if (hasFocus)
            setBorder(UIManager.getBorder("Table.focusCellHighlightBorder"));
        else
            setBorder(null);
        return this;
    }
}
```

可以看到，当单元格获得焦点时，绘制器将绘制一个边框。（从`UIManager`获取正确的边框。为了找到用于获取边框的键，需要查看`DefaultTableCellRenderer`类的源代码。）

提示：如果你的绘制器只是绘制文本字符串或者图标，可以扩展`DefaultTableCellRenderer`类。该类会负责绘制焦点和选择状态。

你需要告诉表格使用这个绘制器去绘制所有`Color`类型的对象。使用`JTable`类的`setDefaultRenderer()`方法：

```java
table.setDefaultRenderer(Color.class, new ColorTableCellRenderer());
```

如果想要基于其他标准选择绘制器，则需要继承`JTable`类并覆盖`getCellRenderer()`方法。

#### 11.1.4.2 绘制表头
为了在表头中显示图标，需要使用`TableColumn`类的`setHeaderValue()`方法设置表头值：

```java
moonColumn.setHeaderValue(new ImageIcon("Moons.gif"));
```

但是，表头不会自动为表头值选择合适的绘制器，必须手动安装。例如，要在列头显示图标，调用

```java
moonColumn.setHeaderRenderer(table.getDefaultRenderer(ImageIcon.class));
```

#### 11.1.4.3 编辑单元格
为了使单元格可编辑，表格模型必须通过定义`isCellEditable()`方法来指明哪些单元格是可编辑的。最常见的情况是你想使某几列可编辑。在示例程序中，允许对以下四列进行编辑：

```java
public boolean isCellEditable(int r, int c) {
    return c == PLANET_COLUMN || c == MOONS_COLUMN || c == GASEOUS_COLUMN || c == COLOR_COLUMN;
}
```

注释：`AbstractTableModel`定义的`isCellEditable()`总是返回`false`。`DefaultTableModel`覆盖了该方法，总是返回`true`。

运行示例程序（程序清单11-4到11-7），你会注意到可以点击Gaseous列中的复选框来改变选中状态，点击Moons列中的单元格就会出现一个组合框（见下图）。稍后将介绍如何安装组合框作为单元格编辑器。点击第一列中的单元格并键入数据，单元格的内容就会改变。

![单元格编辑器](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/单元格编辑器.png)

上面看到的是`DefaultCellEditor`类的三种变体。`DefaultCellEditor`可以用`JTextField`、`JCheckBox`或者`JComboBox`来构造。`JTable`类会自动为`Boolean`类型的单元格安装复选框编辑器，为其他可编辑、但未提供自己的绘制器的单元格安装文本框编辑器（注：参见`JTable.createDefaultEditors()`方法）。

编辑完成后，可以通过调用编辑器的`getCellEditorValue()`方法获取编辑后的值。该方法应该返回一个正确类型的值（即模型的`getColumnType()`方法返回的类型）。

为了获得组合框编辑器，需要手动设置，因为`JTable`并不知道什么样的值适合特定的类型。对于Moons列来说，我们希望让用户选择0~20之间的值。下面是初始化组合框的代码：

```java
var moonCombo = new JComboBox();
for (int i = 0; i <= 20; i++)
    moonCombo.addItem(i);
```

然后构造一个`DefaultCellEditor`并在构造器中提供这个组合框：

```java
var moonEditor = new DefaultCellEditor(moonCombo);
```

接下来，需要安装这个编辑器。与单元格绘制器不同，编辑器不依赖于列类型（我们不希望将它用于所有`Integer`类型的列）。而是需要把它安装到特定的列：

```java
moonColumn.setCellEditor(moonEditor);
```

#### 11.1.4.4 自定义编辑器
点击示例程序中的颜色单元格，会弹出一个颜色选择对话框。选择一种颜色并点击OK，单元格颜色就会更新（见下图）。

![编辑单元格颜色](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/编辑单元格颜色.png)

颜色单元格编辑器不是一种标准编辑器，而是自定义实现。要创建自定义单元格编辑器，需要实现`TableCellEditor`接口。这个接口有点繁琐，Java提供了`AbstractCellEditor`类用于负责事件处理细节。

注：`AbstractCellEditor`并没有实现`TableCellEditor`接口，而是实现了其超接口`CellEditor`。因此自定义编辑器类应该扩展`AbstractCellEditor`类**同时**实现`TableCellEditor`接口。

`TableCellEditor`接口的`getTableCellEditorComponent()`方法返回用于绘制单元格的组件。除了没有`focus`参数，它与`TableCellRenderer`接口的`getTableCellRendererComponent()`方法完全相同。当单元格被编辑时，就假定它已经获得了焦点。在编辑过程中，编辑器组件会暂时**取代**绘制器。在我们的示例中，返回的是一个没有颜色的空面板，用于告诉用户该单元格正在被编辑。

接下来，当用户点击颜色单元格时，应该弹出颜色选择对话框。`JTable`类会用一个事件（例如鼠标点击）调用编辑器的`isCellEditable()`方法，以确定该事件是否可以触发编辑。`AbstractCellEditor`类将该方法定义为始终返回`true`，即接受所有事件。如果该方法返回`false`，表格就不会插入编辑器组件。

插入编辑器组件后，就会调用编辑器的`shouldSelectCell()`方法。应该在这个方法中启动编辑过程，例如弹出外部编辑对话框。

```java
public boolean shouldSelectCell(EventObject anEvent) {
    colorDialog.setVisible(true);
    return true;
}
```

如果用户取消编辑，表格会调用`cancelCellEditing()`方法。如果用户点击了另一个单元格，表格会调用`stopCellEditing()`方法。在这两种情况下，都应该将对话框隐藏。当`stopCellEditing()`方法被调用时，表格可能会使用被部分编辑的值。如果当前值有效，就应该返回`true`。另外，应该调用超类方法以便进行事件触发，否则编辑无法正确地取消。

```java
public void cancelCellEditing() {
    colorDialog.setVisible(false);
    super.cancelCellEditing();
}
```

最后，需要实现`getCellEditorValue()`方法以返回用户编辑的值：

```java
public Object getCellEditorValue() {
    return colorChooser.getColor();
}
```

总之，自定义编辑器应该遵循以下几点：
1. 扩展`AbstractCellEditor`类，并实现`TableCellEditor`接口。
2. 定义`getTableCellEditorComponent()`方法以提供编辑器组件，如文本框、组合框或空面板（如果要弹出对话框）。
3. 定义`shouldSelectCell()`、`stopCellEditing()`和`cancelCellEditing()`方法来处理编辑过程的启动、完成和取消。后两个方法应该调用超类方法以确保监听器能够接收到通知。
4. 定义`getCellEditorValue()`方法返回编辑结果值。

最后通过调用`stopCellEditing()`和`cancelCellEditing()`方法来指明用户何时完成编辑。在创建颜色对话框时，我们安装了用于触发这些事件的确认和取消按钮回调。

```java
colorDialog = JColorChooser.createDialog(null, "Planet Color", false, colorChooser,
    EventHandler.create(ActionListener.class, this, "stopCellEditing"),
    EventHandler.create(ActionListener.class, this, "cancelCellEditing"));
```

还剩下一个问题：如何使用用户编辑的值更新模型。当编辑完成时，`JTable`类会调用表格模型的`setValueAt(value, row, column)`方法来存储新值，`value`参数是单元格编辑器返回的对象。如果你实现了自定义编辑器，那么你知道`getCellEditorValue()`方法返回的对象类型。对于`DefaultCellEditor`，这个值有三种可能：如果单元格编辑器是复选框，那么就是`Boolean`值；如果是文本框，那么就是字符串；如果是组合框，那么就是用户选择的对象。

如果`value`对象不具有合适的类型，就需要对它进行转换。最常见的情况是在文本框中编辑数字。示例程序中使用的是`Integer`类型的组合框，所以无需转换。

[程序清单11-4 tableCellRender/TableCellRenderFrame.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch11/tableCellRender/TableCellRenderFrame.java)

[程序清单11-5 tableCellRender/PlanetTableModel.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch11/tableCellRender/PlanetTableModel.java)

[程序清单11-6 tableCellRender/ColorTableCellRenderer.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch11/tableCellRender/ColorTableCellRenderer.java)

[程序清单11-7 tableCellRender/ColorTableCellEditor.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch11/tableCellRender/ColorTableCellEditor.java)

## 11.2 树
日常生活中有很多树结构的例子，例如计算机文件系统，国家、州和城市的层次结构（如下图所示）。

![国家、州和城市的层次结构](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/国家、州和城市的层次结构.png)

我们经常需要显示树结构。幸运的是，Swing库有一个用于此目的的`JTree`类。`JTree`（及其辅助类）负责布局树状结构，以及展开或折叠节点。在本节中将介绍如何使用`JTree`类。

在继续深入之前，先介绍一些术语（见下图）。**树**(tree)由**节点**(node)组成。每个节点要么是**叶节点**(leaf)，要么有**子节点**(child node)。除了**根节点**(root)，每个节点都有唯一的**父节点**(parent)。一棵树有且只有一个根节点。树的集合称为**森林**(forest)。

![树的术语](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/树的术语.png)

### 11.2.1 简单的树
在第一个示例程序中，我们将展示一棵具有几个节点的树。为了构造`JTree`，需要在构造器中提供树模型：

```java
TreeModel model = ...;
var tree = new JTree(model);
```

注释：还有一些构造器可以从元素集合来构建树（例如`JTree(Object[] value)`），其中每个元素作为（隐藏）根节点的子节点，看起来像是一些单节点的树构成的森林。

本章后面会介绍如何通过实现`TreeModel`接口来构建自己的树模型，目前直接使用Swing库提供的`DefaultTreeModel`。

为了构造默认树模型，必须提供一个根节点。

```java
TreeNode root = ...;
var model = new DefaultTreeModel(root);
```

`TreeNode`也是一个接口。这里使用Swing提供的具体节点类`DefaultMutableTreeNode`。这个类实现了`MutableTreeNode`接口，它是`TreeNode`的子接口（见下图）。

![树类的关系图](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/树类的关系图.png)

默认可变树节点包含一个**用户对象**(user object)（其实就是节点表示的值）。树会为所有节点绘制用户对象。除非指定一个绘制器，否则树会显示用户对象`toString()`方法的结果字符串。

在第一个示例程序中使用了字符串作为用户对象。在实际中，通常会使用更具表现力的用户对象。例如，在显示目录树时，节点使用`File`对象是有意义的。

可以在构造器中指定用户对象，也可以稍后用`setUserObject()`方法设置：

```java
var node = new DefaultMutableTreeNode("Texas");
...
node.setUserObject("California");
```

接下来，需要建立节点之间的父/子关系。从根节点开始，使用`add()`方法添加子节点：

```java
var root = new DefaultMutableTreeNode("World");
var country = new DefaultMutableTreeNode("USA");
root.add(country);
var state = new DefaultMutableTreeNode("California");
country.add(state);
```

下图展示了这棵树的外观。

![一棵简单的树](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/一棵简单的树.png)

按照这种方式将所有节点连接起来。然后用根节点构造一个`DefaultTreeModel`。最后，用树模型构造一个`JTree`。

```java
var treeModel = new DefaultTreeModel(root);
var tree = new JTree(treeModel);
```

或者使用快捷方式，直接将根节点传递给`JTree`构造器，树会自动构造一个默认树模型：

```java
var tree = new JTree(root);
```

程序清单11-8给出了完整的代码。

[程序清单11-8 tree/SimpleTreeFrame.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch11/tree/SimpleTreeFrame.java)

运行这个程序时，最初的树如下图所示，只有根节点及其子节点是可见的。

![最初的树显示](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/最初的树显示.png)

单击圆圈图标（把手(handle)）展开子树。当子树折叠时，从把手图标伸出的线指向右方，当子树展开时指向下方（见下图）。

![折叠和展开的子树](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/折叠和展开的子树.png)

注释：树的显示还取决于所选择的观感(look-and-feel)。前面看到的是Metal观感。在Windows观感中，把手具有更熟悉的样子——带方框的 "+" 或 "-" （见下图）。（注：卷I第10章原本有改变观感的示例[plaf/PlafFrame.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch10/plaf/PlafFrame.java)，但是在新版书中被删除了）

![具有Windows观感的树](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/具有Windows观感的树.png)

可以使用下面的代码取消父子节点之间的连接线（见下图）。

```java
tree.putClientProperty("JTree.lineStyle", "None");
```

![不带连接线的树](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/不带连接线的树.png)

相反，如果要确保显示连接线，则使用

```java
tree.putClientProperty("JTree.lineStyle", "Angled");
```

默认情况下，没有用于折叠根节点的把手。如果愿意，可以通过调用`tree.setShowsRootHandles(true)`添加，结果如下图所示。

![具有根把手的树](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/具有根把手的树.png)

相反，也可以将根节点完全隐藏。这样会显示一个森林，即树的集合，每棵树都有自己的根节点。

```java
tree.setRootVisible(false);
```

如下图所示，看起来有两个根节点 "USA" 和 "Germany" ，而实际的根节点 "World" 是不可见的（此时的“根节点”没有把手，只能通过双击来展开或折叠）。

![一个森林](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/一个森林.png)

注意，叶节点的图标和其他节点是不同的。叶节点用文件图标表示，非叶节点用文件夹图标表示（注：这恰好对应了叶/非叶节点在文件系统树中的角色）。

节点绘制器需要知道每个节点使用哪种图标。默认情况下，如果一个节点的`isLeaf()`方法返回`true`则使用文件图标（也叫叶图标），否则使用文件夹图标（有打开和关闭两种）。

如果某个节点没有子节点，那么`DefaultMutableTreeNode`类的`isLeaf()`方法返回`true`。因此，有子节点的使用文件夹图标，没有子节点的使用文件图标。

有时，这种做法并不合适。假设我们向国家城市树中添加一个 "Montana" （州）节点，但是还不知道要添加哪些城市。此时，我们并不希望这个州节点使用叶图标，因为从概念上讲，只有城市才是叶节点。

`JTree`类不知道哪些节点是叶节点，它要询问树模型。树模型默认通过节点的`isLeaf()`方法来判断。要使用不同的判断标准，可以使用节点的“允许子节点”(`allowsChildren`)属性。

首先，对于那些不应该有子节点的节点调用

```java
node.setAllowsChildren(false);
```

然后，告诉树模型通过节点的`getAllowsChildren()`方法（而不是`isLeaf()`）来确定是否是叶节点。使用`DefaultTreeModel`类的`setAsksAllowsChildren()`方法（注：这不是`TreeModel`接口的方法）：

```java
model.setAsksAllowsChildren(true);
```

根据这个判断标准，允许子节点的节点将显示文件夹图标，不允许子节点的显示文件图标。

另外，如果是通过根节点构造`JTree`，则在构造器中设置`asksAllowsChildren`属性：

```java
var tree = new JTree(root, true); // nodes that don't allow children get leaf icons
```

### 编辑树和树路径
下一个示例程序将展示如何编辑一棵树。下图显示了其界面。点击Add Sibling或Add Child按钮会向树中添加一个新节点（标签为 "New" ）。点击Delete按钮会删除当前选中的节点。

![编辑一棵树](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/编辑一棵树.png)

为了实现这种行为，需要找出当前选中的是哪个节点。`JTree`用一种令人惊讶的方式来标识树中的节点：它并不处理节点，而是处理**树路径**(tree path)。树路径是从根节点开始的节点序列，例如上图中的 "World - USA - Michigan - Ann Arbor" 。

`JTree`为什么需要整个路径，而不是获得一个`TreeNode`然后不断调用`getParent()`方法？实际上，`JTree`完全不知道`TreeNode`接口，该接口也没有被`TreeModel`接口使用过，它只被`DefaultTreeModel`实现类用到了。你完全可以使用其他树模型，其中节点根本没有实现`TreeNode`接口，也就没有`getParent()`方法。将节点连接起来是树模型的职责。`JTree`本身对节点的连接方式一无所知，因此需要操作完整的路径。

`TreePath`管理着一个`Object`（不是`TreeNode`）序列。许多`JTree`方法返回`TreePath`对象。对于树路径，通常只需要知道终端节点，这可以通过`getLastPathComponent()`方法得到。例如，可以像这样找出树中当前选中的节点：

```java
TreePath selectionPath = tree.getSelectionPath();
var selectedNode = (DefaultMutableTreeNode) selectionPath.getLastPathComponent();
```

由于这种查询很常见，因此有一个便捷方法直接返回选中的节点：

```java
var selectedNode = (DefaultMutableTreeNode) tree.getLastSelectedPathComponent();
```

该方法之所以不叫做`getSelectedNode`，是因为树并不了解节点，其树模型只处理路径。

注释：除了树路径，`JTree`类描述节点的另一种方式是**行位置**(row position)——节点在树显示中的行号（从0开始）。只有可见的节点才有行号，并且如果一个节点之前的其他节点被展开、折叠或修改，这个节点的行号也会随之改变。因此，应该避免使用行位置。所有使用行位置的`JTree`方法都有使用树路径的等价方法。

一旦选中了一个节点，就可以对其进行编辑了。但是，不能直接向节点添加子节点：

```java
selectedNode.add(newNode); // No!
```

这样只改变了模型，而关联的视图没有被通知到。应该使用`DefaultTreeModel`类的`insertNodeInto()`方法，该方法会负责发送通知（添加节点后调用`nodesWereInserted()`）。例如，下面的调用将一个新节点添加为选中节点的最后一个子节点，并通知树的视图：

```java
model.insertNodeInto(newNode, selectedNode, selectedNode.getChildCount());
```

调用`removeNodeFromParent()`将一个节点从其父节点中移除并通知视图：

```java
model.removeNodeFromParent(selectedNode);
```

如果不改变节点结构，但改变了用户对象（节点值），应该调用以下方法：

```java
model.nodeChanged(changedNode);
```

自动通知是使用`DefaultTreeModel`的主要优势。如果你提供自己的树模型，就必须手动实现这种自动通知。

警告：`DefaultTreeModel`类有一个`reload()`方法能够重新加载整个模型。但是，不要在进行了少数几个修改之后只是为了更新树而调用`reload()`。在重新生成树时，会重置折叠状态（根的子节点之后的所有节点将再次折叠）。如果用户在每次修改之后都必须重新展开树，那将非常令人烦心。

如果向正处于折叠状态的节点添加子节点，视图不会将其自动展开以显示新添加的子节点。这就没有给用户提供任何该命令已经执行的反馈。可以使用`JTree`类的`makeVisible()`方法实现这个目的。该方法接受一个树路径参数，指向要变为可见的节点（展开路径上的所有中间节点）。

因此，需要构建一条从根节点到新添加节点的树路径。为了获得树路径，首先调用`DefaultTreeModel`类的`getPathToRoot()`方法，它返回根到指定节点之间所有节点的`TreeNode[]`数组。然后将这个数组传递给`TreePath`构造器。

例如，可以像这样使新节点可见：

```java
TreeNode[] nodes = model.getPathToRoot(newNode);
var path = new TreePath(nodes);
tree.makeVisible(path);
```

假设树包含在一个滚动面板中。展开树节点后，新节点可能仍然不可见，因为落在了视图之外。为了解决这个问题，调用`scrollPathToVisible()`而不是`makeVisible()`。该调用将展开路径中的所有节点，并告诉外围的滚动面板将路径末端的节点滚动进视图中。

默认情况下，树节点是不可编辑的。如果调用`tree.setEditable(true)`，用户就可以编辑节点：双击（或者按快捷键F2），编辑字符串，然后按回车键。双击会调用默认编辑器，由`DefaultCellEditor`类实现（为什么叫“单元格”编辑器？），如下图所示。也可以安装其他编辑器，其过程与表格单元格编辑器中讨论的一样。

![默认节点编辑器](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/默认节点编辑器.png)

程序清单11-9给出了树编辑程序的完整源代码。运行该程序，添加几个节点，然后通过双击进行编辑。观察折叠的节点是怎样展开以展示添加的子节点的，以及滚动面板是怎样让添加的节点保持在视图中的。

[程序清单11-9 treeEdit/TreeEditFrame.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch11/treeEdit/TreeEditFrame.java)

### 11.2.2 节点枚举
有时为了查找树中的一个节点，需要从根开始遍历所有节点，直到找到匹配的节点。`DefaultMutableTreeNode`类有几个用于遍历节点的便捷方法。

`breadthFirstEnumeration()`和`depthFirstEnumeration()`方法返回一个枚举对象，其`nextElement()`方法分别使用**广度优先**(breadth-first)和**深度优先**(depth-first)遍历访问当前节点的所有子节点。下图展示了这两种遍历方式，节点标签表示访问顺序。

![树的遍历顺序](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/树的遍历顺序.png)

广度优先枚举是按层遍历的：首先访问根节点，然后是它的所有子节点，接着是孙子节点，以此类推。深度优先枚举也称为**后序遍历**(postorder traversal)：先访问子节点，后访问父节点。`postOrderEnumeration()`方法是`depthFirstEnumeration()`的同义词。为了完整性，还有一个`preOrderEnumeration()`方法，它也是一种深度优先遍历，但是先访问父节点，后访问子节点（即**先序遍历**(preorder traversal)）。

下面是典型用法：

```java
Enumeration breadthFirst = node.breadthFirstEnumeration();
while (breadthFirst.hasMoreElements())
    // do something with breadthFirst.nextElement();
```

最后，还有一个相关方法`pathFromAncestorEnumeration()`，用于查找从给定祖先到当前节点的路径，并枚举该路径上的节点。这并不复杂，只需不断地调用`getParent()`直到找到祖先节点（用`==`判断），然后逆序遍历路径即可。

在下一个示例程序中将运用到节点枚举。该程序显示了类的继承树。在窗体底部的文本框中输入一个类名并点击Add按钮，这个类及其所有超类就会被添加到树中（见下图）。

![一棵继承树](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/一棵继承树.png)

在这个示例中，我们利用了树节点的用户对象可以是任何类型这一事实。由于这里的节点描述的是类，因此在节点中存储`Class`对象。

我们不希望两次添加同一个类，因此需要检查一个类是否已经在树中存在。下面的方法利用广度优先遍历查找具有给定用户对象的节点：

```java
public DefaultMutableTreeNode findUserObject(Object obj) {
    Enumeration e = root.breadthFirstEnumeration();
    while (e.hasMoreElements()) {
        DefaultMutableTreeNode node = (DefaultMutableTreeNode) e.nextElement();
        if (node.getUserObject().equals(obj))
            return node;
    }
    return null;
}
```

### 11.2.3 绘制节点
在应用中可能经常需要改变树组件绘制节点的方式。最常见的改变是为节点选择不同的图标，其他改变可能涉及节点标签的字体或在节点上绘制图像。这些改变都可以通过安装新的**树节点绘制器**(tree cell renderer)来实现。默认情况下，`JTree`类使用`DefaultTreeCellRenderer`对象来绘制节点。该类扩展了`JLabel`类，包含节点图标和节点标签。

注释：绘制器并不绘制用于展开和折叠子树的“把手”。把手是观感的一部分，建议不要改变它们。

可以通过以下三种方式自定义节点绘制：
* 可以改变`DefaultTreeCellRenderer`使用的图标、字体和背景颜色。这些设置适用于树中所有节点。
* 可以安装一个扩展了`DefaultTreeCellRenderer`类的绘制器，并为每个节点改变图标、字体和背景颜色。
* 可以安装一个实现了`TreeCellRenderer`接口的绘制器，并为每个节点绘制自定义图像。

下面逐个进行介绍。最简单的自定义方式是构造一个`DefaultTreeCellRenderer`对象，改变图标，并将它安装到树中：

```java
var renderer = new DefaultTreeCellRenderer();
renderer.setLeafIcon(new ImageIcon("blue-ball.gif")); // used for leaf nodes
renderer.setClosedIcon(new ImageIcon("red-ball.gif")); // used for collapsed nodes
renderer.setOpenIcon(new ImageIcon("yellow-ball.gif")); // used for expanded nodes
tree.setCellRenderer(renderer);
```

可以在上一节的图中看到运行效果。

不建议改变整棵树的字体或背景颜色，因为这实际上是观感的职责。不过，改变树中个别节点的字体以突出显示它们还是很有用的。仔细观察上一节的图，会发现抽象类设置为斜体。

为了改变单个节点的外观，需要安装一个树节点绘制器（类似于本章前面讨论的表格单元格绘制器）。`TreeCellRenderer`接口只有一个方法：

```java
Component getTreeCellRendererComponent(JTree tree, Object value, boolean selected,
    boolean expanded, boolean leaf, int row, boolean hasFocus)
```

`DefaultTreeCellRenderer`类的该方法返回`this`，也就是一个标签（`DefaultTreeCellRenderer`扩展了`JLabel`类）。要自定义绘制器，需要扩展`DefaultTreeCellRenderer`类，并按照以下方式覆盖`getTreeCellRendererComponent()`方法：调用超类方法以准备标签数据，自定义标签属性，最后返回`this`。

```java
class MyTreeCellRenderer extends DefaultTreeCellRenderer {
    public Component getTreeCellRendererComponent(JTree tree, Object value, boolean selected,
        boolean expanded, boolean leaf, int row, boolean hasFocus) {
        Component comp = super.getTreeCellRendererComponent(tree, value, selected,
            expanded, leaf, row, hasFocus);
        DefaultMutableTreeNode node = (DefaultMutableTreeNode) value;
        // look at node.getUserObject();
        Font font = ...;
        comp.setFont(font);
        return comp;
    }
}
```

警告：`getTreeCellRendererComponent()`方法的`value`参数是节点对象，而不是用户对象！回忆一下，用户对象是`DefaultMutableTreeNode`的特性，而`JTree`可以包含任意类型的节点。如果树使用的是`DefaultMutableTreeNode`，就必须通过类型转换和`getUserObject()`获取用户对象，如上面的代码示例所示。

警告：`DefaultTreeCellRenderer`对所有节点使用**同一个**标签对象，只为每个节点改变标签文本（仅调用`paint()`方法来绘制节点，因此可以复用）。如果为某个特定节点改变字体，那么必须在再次调用`getTreeCellRendererComponent()`方法时将其设置回默认值（见程序清单11-11）。否则，所有后续节点都会用改变后的字体进行绘制。

程序清单11-11中的`ClassNameTreeCellRenderer`类根据`Class`对象有无`ABSTRACT`修饰符将类名设置为常规或斜体。由于所有`getTreeCellRendererComponent()`调用都返回一个共享的`JLabel`对象，因此需要保存原始字体，并在下一次调用该方法时恢复。

### 11.2.4 监听树事件
通常情况下，树组件会与其他组件配对使用。当用户选择树节点时，会在另一个窗口中显示某些信息（见下图）。当用户选择一个类时，这个类的静态和实例字段会显示在右侧的文本区中。

![一个类浏览器](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/一个类浏览器.png)

为了实现这个功能，需要安装一个**树选择监听器**(tree selection listener)。监听器必须实现`TreeSelectionListener`接口，该接口只有一个方法：

```java
void valueChanged(TreeSelectionEvent event)
```

每当用户选中或取消选中树节点时，该方法就会被调用。

像这样向树中添加监听器：

```java
tree.addTreeSelectionListener(listener);
```

可以指定是否允许用户选择单个节点、连续范围内的节点或者任意的（可能不连续的）一组节点。`JTree`类使用`TreeSelectionModel`来管理节点选择。需要先获取选择模型，然后将选择模式设置为`SINGLE_TREE_SELECTION`、`CONTIGUOUS_TREE_SELECTION`或`DISCONTIGUOUS_TREE_SELECTION`（默认）之一。例如，在类浏览器示例中，我们希望只允许选择单个类：

```java
int mode = TreeSelectionModel.SINGLE_TREE_SELECTION;
tree.getSelectionModel().setSelectionMode(mode);
```

注释：用户如何选择多个项取决于观感。在Metal观感中，按住Ctrl键并点击一项可以选中或取消选中，按住Shift键并点击一项可以选中一个范围（注：这与Windows文件浏览器的操作一致）。

要获得当前选择，可以使用树的`getSelectionPaths()`方法：

```java
TreePath[] selectedPaths = tree.getSelectionPaths();
```

如果限制了用户只能单选，可以使用便捷方法`getSelectionPath()`，该方法返回第一条被选择的路径，如果没有选择则返回`null`。

警告：`TreeSelectionEvent`类有一个`getPaths()`方法也返回`TreePath[]`数组，但该数组描述的是选择的**变化**，而不是当前选择。

程序清单11-10给出了类浏览器程序的窗体类。该程序显示类的继承层次结构，并通过自定义绘制器将抽象类显示为斜体（绘制器参见代码清单11-11）。可以在窗体底部的文本框中输入任何类名（例如`java.util.ArrayList`），然后点击Add按钮将该类及其超类添加到树中。

`addClass()`方法使用反射机制来构造类树。该方法通过调用`findUserObject()`方法使用广度优先搜索来检查当前类是否已经在树中存在。当选择一个树节点时，右侧的文本区将填充为所选择类的字段。

[程序清单11-10 treeRender/ClassTreeFrame.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch11/treeRender/ClassTreeFrame.java)

[程序清单11-11 treeRender/ClassNameTreeCellRenderer.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch11/treeRender/ClassNameTreeCellRenderer.java)

### 11.2.5 自定义树模型
在最后一个示例中，我们实现了一个能够查看对象内容的程序，就像IDE的调试器一样（见下图）。

![对象检查树](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/对象检查树.png)

每个节点对应一个实例字段。如果该字段是一个对象，可以展开查看该对象的实例字段。顶级对象是当前窗体。

该程序的不同之处在于树并没有使用`DefaultTreeModel`。如果你已经有了按照层次结构组织的数据，你可能不想再用节点构建一棵重复的树，并担心保持两棵树的同步。这正是我们面临的情形——对象已经通过引用相互链接了，因此不需要再复制链接结构。

`TreeModel`接口只有少数几个方法。第一组方法使得`JTree`能够获得根节点和子节点。

```java
Object getRoot()
int getChildCount(Object parent)
Object getChild(Object parent, int index)
```

这个示例显示了为什么`TreeModel`接口像`JTree`类那样不需要明确的节点概念。根及其子节点可以是任何对象。`TreeModel`负责告诉`JTree`它们是怎样连接的。

下一个方法与`getChild()`相反，返回给定子节点的索引：

```java
int getIndexOfChild(Object parent, Object child)
```

树模型通过`isLeaf()`方法告诉`JTree`哪些节点应该显示为叶节点：

```java
boolean isLeaf(Object node)
```

如果你的代码改变了树模型，必须通知树以便重新绘制。树会向模型添加`TreeModelListener`，因此模型必须支持通常的监听器管理方法：

```java
void addTreeModelListener(TreeModelListener l)
void removeTreeModelListener(TreeModelListener l)
```

可以在程序清单11-13中看到这些方法的实现。

当模型修改了树的内容时，它会调用`TreeModelListener`接口的4个方法之一：`treeNodesChanged()`、`treeNodesInserted()`、`treeNodesRemoved()`或`treeStructureChanged()`。这些方法的参数`TreeModelEvent`对象描述了改变的位置。程序清单11-13展示了如何通过替换根节点来触发一个事件。

提示：为了简化事件触发的代码，我们使用了辅助类`javax.swing.EventListenerList`来收集监听器。

最后，如果用户编辑了树节点，模型的以下方法就会被调用：

```java
void valueForPathChanged(TreePath path, Object newValue)
```

现在转向示例程序的具体实现。我们的树将包含`Variable`类型的对象。

注释：在这个示例的自定义树模型中，节点本身就是`Variable`对象。假如使用`DefaultTreeModel`，节点将会是具有`Variable`类型用户对象的`DefaultMutableTreeNode`对象。

例如，假设查看变量

```java
Employee joe;
```

该变量的类型为`Employee`，名字为`"joe"`，值为对象引用`joe`的值。在程序清单11-14中定义了`Variable`类，用来描述程序中的变量：

```java
var v = new Variable(Employee.class, "joe", joe);
```

如果变量的类型是基本类型，必须使用包装器对象：

```java
new Variable(double.class, "salary", Double.valueOf(salary));
```

如果变量的类型是一个类，那么该变量就有字段。可以使用反射枚举所有字段，并将其存放到一个`ArrayList`中。由于`Class`类的`getFields()`方法不返回超类的字段，因此还需要对所有超类调用`getFields()`。可以在`Variable`构造器中找到这些代码。

`Variable`类的`getFields()`方法返回字段数组。`toString()`方法格式化节点标签，标签包含变量的类型和名称。如果是基本类型，标签还包含变量的值。

注释：如果变量是数组类型，我们不会显示数组的元素，但这并不难实现。

下面继续介绍树模型的实现。前两个方法很简单，`root`是查看的变量。

```java
public Object getRoot() {
    return root;
}

public int getChildCount(Object parent) {
    return ((Variable) parent).getFields().size();
}
```

`getChild()`方法返回一个新的`Variable`对象，用于描述给定索引位置上的字段。使用反射获得字段的类型、名字和值。

```java
public Object getChild(Object parent, int index) {
    ArrayList<Field> fields = ((Variable) parent).getFields();
    var f = (Field) fields.get(index);
    Object parentValue = ((Variable) parent).getValue();
    try {
        return new Variable(f.getType(), f.getName(), f.get(parentValue));
    }
    catch (IllegalAccessException e) {
        return null;
    }
}
```

这三个方法向`JTree`揭示了对象树的结构。其余的是一些常规方法，源代码见程序清单11-13。

关于这个树模型有一个值得关注的地方：它实际上描述了一棵**无限**树。可以通过查看`WeakReference`对象来验证这一点：点击名为`referent`的字段，又会回到初始的对象。你会看到一棵相同的子树，并且可以再次展开它的`WeakReference`对象，无穷无尽。当然，你无法存储无限的节点集合，树模型只是在用户展开父节点时按需生成节点。

程序清单11-12给出了这个示例程序的窗体类。

[程序清单11-12 treeModel/ObjectInspectorFrame.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch11/treeModel/ObjectInspectorFrame.java)

[程序清单11-13 treeModel/ObjectTreeModel.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch11/treeModel/ObjectTreeModel.java)

[程序清单11-14 treeModel/Variable.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch11/treeModel/Variable.java)

注：在Java 17中运行这个程序时会遇到与卷I第5章的`ObjectAnalyzer`程序同样的问题： "java.lang.reflect.InaccessibleObjectException: Unable to make field xxx accessible: module java.desktop does not "opens javax.swing" to unnamed module" 。这个问题可以通过添加`--add-opens`选项解决，但这个程序查看的是Swing窗体对象，其字段总共涉及几十个包，难以逐个添加对应的选项，也无法用一个选项打开所有包。因此这里改用了第2章程序清单2-3中的员工示例，不过仍然需要打开`java.util`包：

```shell
java --add-opens=java.base/java.util=ALL-UNNAMED treeModel.ObjectInspectorTest
```

![员工对象树](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/员工对象树.png)

## 11.3 高级AWT
你可以使用`Graphics`类的方法创建简单的绘图（卷I第10章已经介绍过）。这些方法对于简单的应用程序来说已经足够了，但是当你需要创建复杂的形状或者需要完全控制图形的外观时，它们就显得不足了。Java 2D API是一个更先进的类库，可以用来生成高质量的绘图。下面几节将概要地介绍该API。

### 11.3.1 绘制流程
最初的JDK 1.0具有非常简单的绘制形状的机制：选择了颜色和绘制模式，并调用`Graphics`类的方法，如`drawRect()`或`fillOval()`。Java 2D API支持更多选项。

要绘制一个形状，执行以下步骤：

1.获得一个`Graphics2D`类的对象。该类是`Graphics`类的子类。从Java 1.2开始，诸如`paint()`和`paintComponent()`等方法就会自动接收一个`Graphics2D`类的对象，只需使用如下的强制类型转换：

```java
public void paintComponent(Graphics g) {
    var g2 = (Graphics2D) g;
    ...
}
```

2.使用`setRenderingHints()`方法设置**渲染提示**(rendering hints)，它是速度和绘图质量之间的权衡。

3.使用`setStroke()`方法设置**笔划**(stroke)，用于绘制形状的轮廓。可以选择粗细和实线/虚线。

4.使用`setPaint()`方法设置**颜料**(paint)，用于填充笔划或者形状内部等区域。可以创建纯色、渐变色或平铺填充图案。

5.使用`clip()`方法**剪裁**(clip)形状，将其限制在任意区域内。

6.使用`transform()`方法设置从用户空间到设备空间的**变换**(transformation)，用于移动、缩放、旋转或拉伸形状。如果在自定义坐标系中定义形状比使用像素坐标更容易，就可以使用变换。

7.使用`setComposite()`方法设置**组合规则**(composition rule)，用于描述如何将新像素与现有像素组合起来。

8.创建一个形状。Java 2D API提供了很多形状对象和用于组合形状的方法。

9.使用`draw()`或`fill()`方法绘制或填充该形状。绘制仅画出轮廓，填充会将内部着色。

当然，在许多实际情况下并不需要所有这些步骤。2D图形上下文的设置有合理的默认值，只有在需要时才修改这些设置。

上述各种`set`方法只是设置了2D图形上下文的状态，并不进行任何绘制操作。同样，在构造`Shape`对象时也不会进行绘制。只有当调用`draw()`或者`fill()`方法时才会绘制形状。此时，这个新图形由**绘制流程**(rendering pipeline)进行计算（见下图）。

![绘制流程](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/绘制流程.png)

### 11.3.2 形状
`Graphics`类中绘制形状的方法包括`drawLine()`、`drawRectangle()`、`drawRoundRect()`、`drawOval()`、`drawArc()`等，它们还有对应的`fillXxx`方法。Java 2D API使用了一种完全不同的、面向对象的方式。即不再使用方法，而是使用类：`Line2D`、`Rectangle2D`、`RoundRectangle2D`、`Ellipse2D`、`Arc2D`等。这些类都实现了`Shape`接口，将在下面几节中介绍。

#### 11.3.2.1 Shape类层次结构
要绘制一个形状，首先创建一个实现了`Shape`接口的类的对象，然后调用`Graphics2D`类的`draw()`方法。

`Rectangle2D`、`RoundRectangle2D`、`Ellipse2D`和`Arc2D`类都继承自一个公共超类`RectangularShape`。诚然，椭圆和弧形都不是矩形，但它们都有一个矩形边界。

每个名字以 "2D" 结尾的类都有两个子类，用于指定坐标是`float`还是`double`类型。例如，在卷I第10章中已经见过`Rectangle2D.Float`和`Rectangle2D.Double`。所有图形类在内部都使用`float`坐标，因为`float`数占用较少的存储空间，但具有足够高的几何计算精度。但是，Java语言操作`float`数有点繁琐，必须使用后缀`F`和强制类型转换。因此，图形类的大多数方法都使用`double`类型的参数和返回值。只有在构造2D形状对象时才需要选择`float`还是`double`坐标的构造器。例如，

```java
var floatRect = new Rectangle2D.Float(5F, 10F, 7.5F, 15F);
var doubleRect = new Rectangle2D.Double(5, 10, 7.5, 15);
```

`Xxx2D.Float`和`Xxx2D.Double`类都是`Xxx2D`的子类（同时也是静态内部类）。

最后，`Point2D`类用于描述具有x和y坐标的点。点用于定义形状，但它本身并不是形状。

下图展示了形状类之间的关系，图中省略了`Float`和`Double`子类，遗留类用灰色填充。

![形状类之间的关系](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/形状类之间的关系.png)

#### 11.3.2.2 使用形状类
在卷I第10章 10.3.1节中已经介绍了如何使用`Rectangle2D`、`Ellipse2D`和`Line2D`类。本节将介绍如何使用其他的2D形状。

使用`RoundRectangle2D`类创建圆角矩形，指定左上角坐标、宽、高和圆角尺寸（见下图）。

![圆角矩形](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/圆角矩形.png)

例如，调用

```java
var r = new RoundRectangle2D.Double(150, 200, 100, 50, 20, 20);
```

构造了一个左上角坐标为(150, 200)、宽100、高50、圆角半径为20的圆角矩形。

使用`Arc2D`类创建弧形，指定边界框、起始角、弧角（单位：度，见下图）和闭合类型。

```java
var a = new Arc2D(x, y, width, height, startAngle, arcAngle, closureType);
```

![椭圆弧](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/椭圆弧.png)

闭合类型是`OPEN`、`PIE`或`CHORD`之一（见下图）。

![弧类型](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/弧类型.png)

警告：角度是相对于矩形边界指定的（而非真实角度），使得45°总是落在从椭圆中心到矩形边界右上角的连线上。

Java 2D API还支持**二次曲线**(quadratic curve)和**三次曲线**(cubic curve)。如下图所示，二次和三次曲线由两个**端点**以及一个或两个**控制点**指定的。移动控制点会改变曲线的形状。

![二次曲线](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/二次曲线.png)

![三次曲线](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/三次曲线.png)

要创建二次和三次曲线，需要给出端点和控制点的坐标。例如，

```java
var q = new QuadCurve2D.Double(startX, startY, controlX, controlY, endX, endY);
var c = new CubicCurve2D.Double(startX, startY, control1X, control1Y,
    control2X, control2Y, endX, endY);
```

可以构建线段、二次曲线和三次曲线的任意序列，并将其存放在一个`GeneralPath`对象中。使用`moveTo()`方法指定路径的第一个坐标，例如：

```java
var path = new GeneralPath();
path.moveTo(10, 20);
```

然后可以通过调用`lineTo()`、`quadTo()`或`curveTo()`方法分别用直线、二次曲线或三次曲线来扩展路径。调用`lineTo()`时，需要指定下一个端点；对于两个曲线方法，需要指定控制点，然后是端点。例如，

```java
path.lineTo(20, 30);
path.curveTo(control1X, control1Y, control2X, control2Y, endX, endY);
```

通过调用`closePath()`方法来闭合路径，它会绘制一条回到路径起点的直线。

要绘制一个多边形，只需调用`moveTo()`到达第一个顶点，然后反复地调用`lineTo()`到达其他顶点，最后调用`closePath()`。

`GeneralPath`不必是连通的，可以随时调用`moveTo()`开始一段新的路径。

最后，可以使用`append()`方法向路径添加任意的`Shape`对象，形状的轮廓会被添加到路径的末尾。如果新形状应该连接到路径的最后一个端点，那么`append()`方法的第二个参数为`true`，否则为`false`。例如，

```java
Rectangle2D r = ...;
path.append(r, true);
```

会添加一条从路径终点到矩形起点的直线，然后将矩形的轮廓添加到路径。

注：矩形的“起点”由路径迭代器定义。`Shape`接口的`getPathIterator()`方法返回一个`PathIterator`对象，该迭代器生成描述形状轮廓的片段（直线或曲线）序列。

程序清单11-15中的程序让你能够创建各种形状。可以从组合框选择一种形状绘制器，包括：
* 直线
* 矩形、圆角矩形和椭圆
* 弧线（除了弧线本身外，还显示边界矩形和起始及结束角度）
* 多边形（使用`GeneralPath`）
* 二次和三次曲线

可以用鼠标来调整控制点。在移动控制点时，形状会持续地重绘。

抽象超类`ShapeMaker`封装了形状绘制器类的共性。每个形状都有固定数量的控制点，用户可以将其到处移动。抽象方法`Shape makeShape(Point2D[] points)`根据给定的控制点当前位置计算实际的形状。

注：`ShapeMaker`及其子类相当于一种自定义的形状类层次结构，统一使用点数组来创建图形，并通过`makeShape()`方法映射到Swing形状类。

为了启用控制点拖动功能，`ShapePanel`类需要同时处理鼠标事件和鼠标移动事件。如果鼠标在一个矩形上面被按下，那么后续的鼠标拖动就会移动该矩形。

[程序清单11-15 shape/ShapeComponent.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch11/shape/ShapeComponent.java)

![ShapeTest程序](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/ShapeTest程序.png)

### 11.3.3 区域
上一节介绍了如何构造由直线和曲线构成的复杂形状。有时，通过组合**区域**(area)（如矩形、多边形或椭圆）来描述形状会更容易。Java 2D API支持四种**构造性区域几何**(constructive area geometry)操作，用于将两个区域组合成一个新区域。
* `add`：组合区域包含所有在第一个或第二个区域中的点（“或”/并集）。
* `subtract`：组合区域包含所有在第一个但不在第二个区域中的点（“减”/差集）。
* `intersect`：组合区域包含所有在第一个和第二个区域中的点（“与”/交集）。
* `exclusiveOr`：组合区域包含所有要么在第一个要么在第二个区域中、但不同时在两个区域中的点（“异或”/对称差）。

下图显示了这些操作的结果。

![构造性区域几何操作](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/构造性区域几何操作.png)

要构造一个复杂区域，首先创建一个默认的`Area`对象：

```java
var a = new Area();
```

然后将该区域与其他形状组合：

```java
a.add(new Area(new Ellipse2D.Double(...)));
a.subtract(new Area(new Rectangle2D.Double(...)));
...
```

`Area`类实现了`Shape`接口。可以用`Graphics2D`类的`draw()`方法绘制区域边界，或者使用`fill()`方法给区域内部着色。

[area/AreaComponent.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch11/area/AreaComponent.java)

### 11.3.4 笔划
`Graphics2D`类的`draw()`方法使用当前选择的**笔划**(stroke)来绘制形状的轮廓。默认情况下，笔划是1像素宽的实线。可以通过调用`setStroke()`方法并提供一个实现了`Stroke`接口的类的对象来选择不同的笔划。Java 2D API只定义了一个这样的类，即`BasicStroke`。在本节中将介绍该类的功能。

可以构造任意粗细的笔划。例如，可以像这样绘制一条10像素宽的线条：

```java
g2.setStroke(new BasicStroke(10.0F));
g2.draw(new Line2D.Double(...));
```

当笔划的粗细大于1个像素时，笔划的末端可以采用不同的**端头样式**(end cap style)（如下图所示）：
* **平头**(butt cap)：在笔划末端处就结束。
* **圆头**(round cap)：在笔划末端处添加半圆。
* **方头**(square cap)：在笔划末端处添加半个方块（默认）。

![笔划的端头样式](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/笔划的端头样式.png)

当两个较粗的笔划相遇时，有三种**连接样式**(join style)（如下图所示）：
* **斜面连接**(bevel join)：用一条与夹角平分线相垂直的直线连接。
* **圆形连接**(round join)：延长两条笔划形成一个圆头。
* **斜尖连接**(miter join)：延长两条笔划形成一个尖峰（默认）。

![笔划的连接样式](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/笔划的连接样式.png)

注：连接样式只应用于矩形、路径等**整体**形状的笔划连接处。独立的两条线段即使相交也没有任何连接样式（如上图中“无连接”所示）。

如果两条线以非常小的角度斜尖连接在一起，则会改用斜面连接，以防止出现过长的尖峰。**斜尖限制**(miter limit)可以控制这种转换。从技术上讲，斜尖限制是尖峰的内角和外角的距离除以笔划的宽度（如下图所示）。默认的斜尖限制是10，对应大约11.48°的角(2arccsc 10)。

![斜尖限制](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/斜尖限制.png)

可以在`BasicStroke`构造器中指定这些选项，例如：

```java
g2.setStroke(new BasicStroke(10.0F, BasicStroke.CAP_ROUND, BasicStroke.JOIN_ROUND));
g2.setStroke(new BasicStroke(10.0F, BasicStroke.CAP_BUTT, BasicStroke.JOIN_MITER,
    15.0F /* miter limit */));
```

最后，可以通过设置**虚线模式**(dash pattern)来创建虚线。在程序清单11-16的程序中，可以选择拼出摩斯电码中SOS（三短三长三短）的虚线模式。虚线模式是一个`float[]`数组，包含连接和断开的间隔长度（见下图）。

![一种虚线模式](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/一种虚线模式.png)

在构造`BasicStroke`时可以指定虚线模式和**虚线相位**(dash phase)。虚线相位指示每条线应该从虚线模式的何处开始，通常将其设置为0。

```java
float[] dashPattern = {10, 10, 10, 10, 10, 10, 30, 10, 30, ...};
g2.setStroke(new BasicStroke(10.0F, BasicStroke.CAP_BUTT, BasicStroke.JOIN_MITER,
    10.0F /* miter limit */, dashPattern, 0 /* dash phase */));
```

注释：端头样式应用于虚线模式中**每条短线**的末端。

程序清单11-16中的程序可以指定端头样式、连接样式和虚线。可以用鼠标拖动端点。

[程序清单11-16 stroke/StrokeComponent.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch11/stroke/StrokeComponent.java)

![StrokeTest程序](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/StrokeTest程序.png)

### 11.3.5 颜料
当填充一个形状时，其内部会被涂上**颜料**(paint)。使用`setPaint()`方法将颜料样式设置为一个实现了`Paint`接口的类的对象。Java 2D API提供了三个这样的类（见下图）：
* `Color`：纯色，例如`g2.setPaint(Color.RED)`
* `GradientPaint`：渐变色
* `TexturePaint`：用平铺图案填充

![颜料样式](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/颜料样式.png)

可以通过指定两个点以及这两个点处的颜色来构造`GradientPaint`对象：

```java
g2.setPaint(new GradientPaint(p1, Color.RED, p2, Color.YELLOW));
```

颜色将沿着连接两点的线渐变，而与连线垂直方向上的颜色是不变的。超出连线端点的点赋予端点处的颜色。

另外，如果构造器的第五个参数`cyclic`为`true`，那么颜色将**循环**变化，并在端点之外仍然保持变化。

要构造`TexturePaint`对象，需要指定一个`BufferedImage`和一个**锚位**(anchor)矩形：

```java
BufferedImage bufferedImage = ImageIO.read(new File("blue-ball.gif"));
var anchorRectangle = new Rectangle2D.Double(...);
g2.setPaint(new TexturePaint(bufferedImage, anchorRectangle));
```

本章后面详细讨论图像时再介绍`BufferedImage`类。锚位矩形在x和y方向上无限延伸，以平铺整个坐标平面。图像会被缩放以适应锚位矩形，然后复制到每个图块中（注：如果不希望缩放，则将锚位矩形设置为原图像的大小）。

[paint/PaintComponent.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch11/paint/PaintComponent.java)

### 11.3.6 坐标变换
`Graphics2D`类的`scale()`将图形上下文的**坐标变换**(coordinate transformation)设置为缩放变换。这种变换能够将**用户坐标**(user coordinates)（用户指定的单位，如米）转换成**设备坐标**(device
coordinates)（像素）。

```java
double pixelsPerMeter = 72;
g2.scale(pixelsPerMeter, pixelsPerMeter);
g2.draw(new Line2D.Double(/* coordinates in meters */)); // converts to pixels and draws scaled line
```

坐标变换在实际中非常有用。它使你可以使用方便的坐标值（例如用以米为单位的实际尺寸绘制汽车），图形上下文负责将其转换为像素的繁重工作。

有四种基本的变换：
* **缩放**(scaling)：放大或缩小与一个固定点的所有距离。
* **旋转**(rotation)：绕一个固定中心旋转所有点。
* **平移**(translation)：将所有点移动一个固定距离。
* **切变**(shear)：将一条线保持固定，并将平行于它的线“滑动”与固定线的距离成比例的量。

下图展示了对单位正方形进行这四种基本变换的效果。

![基本变换](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/基本变换.png)

`Graphics2D`类的`scale()`、`rotate()`、`translate()`和`shear()`分别将图形上下文的坐标变换设置为这几种基本变换。

也可以**组合**坐标变换。假设你想对图形进行旋转并放大两倍，只需同时提供旋转和缩放变换：

```java
g2.rotate(angle);
g2.scale(2, 2);
g2.draw(...);
```

在这种情况下，变换的顺序无关紧要。但是，对于大多数变换，顺序却是很重要的。例如，先旋转后切变和先切变后旋转将产生不同的图形。图形上下文将按照你所提供的**相反**顺序来应用变换操作——也就是说，最后提供的变换会被最先应用。

你可以提供任意数量的变换。例如，下面的变换序列

```java
g2.translate(x, y);   // (3)
g2.rotate(a);         // (2)
g2.translate(-x, -y); // (1)
```

先将形状从点(x, y)平移到原点，然后绕着原点旋转角度a（单位：弧度），最后从原点移回(x, y)。总体效果是绕着点(x, y)旋转角度a（如下图所示）。由于绕着任意点旋转是一个很常见的操作，因此提供了捷径：`g2.rotate(a, x, y)`。

![组合变换](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/组合变换.png)

如果对矩阵理论有所了解，就会知道这些基本变换及其组合都可以用以下形式的变换矩阵来表示：

$$
\begin{bmatrix}
x_{new} \newline
y_{new} \newline
1
\end{bmatrix}
=
\begin{bmatrix}
a & c & e \newline
b & d & f \newline
0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
x \newline
y \newline
1
\end{bmatrix}
=
\begin{bmatrix}
ax + cy + e \newline
bx + dy + f \newline
1
\end{bmatrix}
$$

这种变换称为**仿射变换**(affine transformation)。组合变换等价于变换矩阵相乘。

Java 2D API中的`AffineTransform`类用于描述这种变换。如果知道变换矩阵的元素，就可以像这样直接构造：

```java
var t = new AffineTransform(a, b, c, d, e, f);
```

另外，以下静态工厂方法可以构造表示相应变换类型的矩阵：

| 变换类型 | 工厂方法 | 变换矩阵 |
| --- | --- | --- |
| 缩放 | `getScaleInstance(sx, sy)` | $ \begin{bmatrix} sx & 0 & 0 \newline 0 & sy & 0 \newline 0 & 0 & 1 \end{bmatrix} $ |
| 旋转 | `getRotateInstance(a)` | $ \begin{bmatrix} \cos a & -\sin a & 0 \newline \sin a & \cos a & 0 \newline 0 & 0 & 1 \end{bmatrix} $ |
| 平移 | `getTranslateInstance(tx, ty)` | $ \begin{bmatrix} 1 & 0 & tx \newline 0 & 1 & ty \newline 0 & 0 & 1 \end{bmatrix} $ |
| 切变 | `getShearInstance(shx, shy)` | $ \begin{bmatrix} 1 & shx & 0 \newline shy & 1 & 0 \newline 0 & 0 & 1 \end{bmatrix} $ |

注意，`AffineTransform`对象是可变的！实例方法`setToRotation()`、`setToScale()`、`setToTranslation()`和`setToShear()`将变换对象设置为一种新类型，例如：

```java
t.setToRotation(angle); // sets t to a rotation
```

`concatenate()`方法将变换对象与另一个变换组合。便捷方法`t.rotate(a)`等价于`t.concatenate(AffineTransform.getRotateInstance(a))`。

可以把图形上下文的坐标变换设置为一个`AffineTransform`对象：

```java
g2.setTransform(t); // replaces current transformation
```

不过，在实际中不应该调用`setTransform()`方法，因为它会取代任何现有的变换（注：图形上下文会自动根据父组件的位置设置平移变换，覆盖它可能会导致图形绘制位置不正确）。应该调用`transform()`方法将现有变换与新变换组合：

```java
g2.transform(t); // composes current transformation with t
```

如果只想临时应用某个变换，应该先获得旧的变换，然后将其与新变换组合，最后恢复旧的变换：

```java
AffineTransform oldTransform = g2.getTransform(); // save old transform
g2.transform(t); // apply temporary transform
// draw on g2
g2.setTransform(oldTransform); // restore old transform
```

[transform/TransformationComponent.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch11/transform/TransformationComponent.java)

### 11.3.7 剪裁
通过在图形上下文中设置**剪裁形状**(clipping shape)可以将所有绘制操作限制在该形状内部。

```java
g2.setClip(clipShape); // but see below
g2.draw(shape); // draws only the part that falls inside the clipping shape
```

与坐标变换类似，`setClip()`会取代任何现有的剪裁形状。应该调用`clip()`，将现有剪裁形状与新的剪裁形状相交（注：另见11.5.1节第二个“警告”）。

```java
g2.clip(clipShape); // better
```

如果只想临时应用某个剪裁形状，应该先获得旧的剪裁，然后添加新的剪裁，最后恢复旧的剪裁：

```java
Shape oldClip = g2.getClip(); // save old clip
g2.clip(clipShape); // apply temporary clip
// draw on g2
g2.setClip(oldClip); // restore old clip
```

下图展示了剪裁功能，按照复杂形状（即一组字母的轮廓）剪裁线条图案。

![使用字母形状剪裁线条图案](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/使用字母形状剪裁线条图案.png)

为了获得字符轮廓，需要**字体渲染上下文**(font render context)。使用`Graphics2D`类的`getFontRenderContext()`方法：

```java
FontRenderContext context = g2.getFontRenderContext();
```

接着，使用字符串、字体和该上下文创建一个`TextLayout`对象：

```java
var layout = new TextLayout("Hello", font, context);
```

文本布局对象用于描述由特定字体渲染上下文所渲染的字符序列的布局。`getOutline()`方法返回一个`Shape`对象，用于描述在文本布局中字符轮廓的形状。轮廓形状从原点(0, 0)开始，可以通过仿射变换指定想要显示的位置。

```java
AffineTransform transform = AffineTransform.getTranslateInstance(100, 100);
Shape outline = layout.getOutline(transform);
```

然后，把轮廓附加到剪裁形状：

```java
var clipShape = new GeneralPath();
clipShape.append(outline, false);
```

最后，设置剪裁形状并绘制一组线条。线条只会出现在字符边界内部。

```java
g2.clip(clipShape);
var p = new Point2D.Double(0, 0);
for (int i = 0; i < NLINES; i++) {
    double x = ...;
    double y = ...;
    var q = new Point2D.Double(x, y);
    g2.draw(new Line2D.Double(p, q)); // lines are clipped
}
```

[clip/ClippingComponent.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch11/clip/ClippingComponent.java)

### 11.3.8 透明和组合
在标准RGB颜色模型中，每种颜色都是由红、绿、蓝三种分量描述的。但是，用它来描述**透明**或者半透明的图像区域也是非常方便的。当你将一个图像置于现有图像上面时，透明像素完全不会遮挡它下面的像素，而半透明像素会与它下面的像素混合。下图显示了在图像上叠加一个半透明矩形的效果。

![在图像上叠加一个半透明矩形](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/在图像上叠加一个半透明矩形.png)

在Java 2D API中，透明度是由**alpha通道**(alpha channel)描述的。除了红、绿、蓝分量外，每个像素还有一个介于0（完全透明）和1（完全不透明）之间的alpha值。例如，上图中的矩形填充了透明度50%的淡黄色：

```java
new Color(0.7F, 0.7F, 0.0F, 0.5F);
```

注：如果alpha值用整数表示则是0~255之间。

[transparency/TransparencyComponent.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch11/transparency/TransparencyComponent.java)

如果将两个形状重叠在一起，就需要把源像素和目标像素的颜色和透明度**组合**(compose)起来。计算机图形学领域的研究人员Porter和Duff已经阐明了12种可能的**组合规则**(composition rule)，但其中只有两种具有实际意义。Java 2D API实现了所有这些规则，通常只需使用`SRC_OVER`规则，这是`Graphics2D`的默认规则，会产生最直观的结果。

下面是这些规则的原理。假设有一个透明度为a<sub>S</sub>的**源像素**(source pixel)和一个已存在的透明度为a<sub>D</sub>的**目标像素**(destination pixel)，你希望把二者组合起来。下图展示了如何设计组合规则。

![设计组合规则](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/设计组合规则.png)

Porter和Duff将透明度作为像素颜色被使用的概率。从源像素的角度来看，它有a<sub>S</sub>的概率使用源色，1 - a<sub>S</sub>的概率不在乎。对于目标像素也是这样。当组合颜色时，假设这两个概率是独立的，那么就有四种情况，如上图所示。如果源像素要使用源色，而目标像素不在乎，那么使用源色就是合理的。因此图中右上角标注了 "S" ，这种情况的概率为a<sub>S</sub>(1 - a<sub>D</sub>)。类似地，左下角标注为 "D" 。

如果源像素和目标像素都想选择自己的颜色，就要用到Porter-Duff规则。如果我们认为源像素比较重要，就将右下角也标注为 "S" ，这个规则称为`SRC_OVER`。在这个规则中，将源色以权重a<sub>S</sub>和目标色以权重(1 - a<sub>S</sub>)a<sub>D</sub>组合起来。其视觉效果是源色与目标色的混合，并优先选择源色。特别地，如果a<sub>S</sub> = 1，那么根本不考虑目标色，源像素直接覆盖目标像素；如果a<sub>S</sub> = 0，那么源像素是完全透明的，目标色不变。

下表展示了Java 2D API支持的所有规则（上图中右上角为S或空，左下角为D或空，右下角为S、D或空，共2×2×3=12种组合）。

| 规则 | 解释 |
| --- | --- |
| `CLEAR` | 源清除目标 |
| `SRC` | 源覆盖目标和空像素 |
| `DST` | 源不影响目标 |
| `SRC_OVER` | 源与目标混合，并覆盖空像素 |
| `DST_OVER` | 源不影响目标，但覆盖空像素 |
| `SRC_IN` | 源覆盖目标 |
| `SRC_OUT` | 源清除目标，并覆盖空像素 |
| `DST_IN` | 源透明度修改目标 |
| `DST_OUT` | 源透明度取反修改目标 |
| `SRC_ATOP` | 源与目标混合 |
| `DST_ATOP` | 源透明度修改目标，并覆盖空像素 |
| `XOR` | 源透明度取反修改目标，并覆盖空像素 |

下图显示了当透明度为0.75的矩形源区域和透明度为1.0的椭圆目标区域组合时各种规则的效果。

![Porter-Duff组合规则](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/Porter-Duff组合规则.png)

可以看到，大多数规则并不是非常有用。`DST_IN`规则就是一个极端的例子。它根本不考虑源色，却使用了源像素透明度来修改目标像素。

使用`Graphics2D`类的`setComposite()`方法设置组合规则，提供一个实现了`Composite`接口的类的对象。Java 2D API提供了一个这样的类`AlphaComposite`，它实现了上图中所有的Porter-Duff规则。该类的工厂方法`getInstance()`产生`AlphaComposite`对象，需要提供规则和用于源像素的透明度。例如：

```java
int rule = AlphaComposite.SRC_OVER;
float alpha = 0.5f;
g2.setComposite(AlphaComposite.getInstance(rule, alpha));
g2.setPaint(Color.blue);
g2.fill(rectangle);
```

程序清单11-17中的程序让你能够探究这些组合规则。可以从组合框中选择一个规则，并使用滑块来设置透明度。

这个程序有一个重要的细节。不能保证与屏幕对应的图形上下文一定具有alpha通道（实际上通常没有）。当像素被放到没有alpha通道的目标上时，其颜色会与alpha值相乘，而alpha值会被丢弃。但是许多Porter-Duff规则都使用了目标的alpha值，这意味着目标的alpha通道很重要。因此，我们使用带有ARGB颜色模型的缓冲图像来组合形状。

```java
var image = new BufferedImage(getWidth(), getHeight(), BufferedImage.TYPE_INT_ARGB);
Graphics2D gImage = image.createGraphics();
// now draw to gImage
g2.drawImage(image, null, 0, 0);
```

程序清单11-17和11-18给出了窗体和组件类。程序清单11-19中的`Rule`类为每条规则提供了简要解释（见下图）。

[程序清单11-17 composite/CompositeTestFrame.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch11/composite/CompositeTestFrame.java)

[程序清单11-18 composite/CompositeComponent.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch11/composite/CompositeComponent.java)

[程序清单11-19 composite/Rule.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch11/composite/Rule.java)

![CompositeTest程序](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/CompositeTest程序.png)

## 11.4 栅格图像
Java 2D API可以绘制直线、曲线和区域，这是“向量”API，因为需要指定形状的数学属性。但是，对于处理由像素构成的图像，需要使用颜色数据的“栅格”(raster)。下面几节将介绍如何在Java中处理**栅格图像**(raster image)。

### 11.4.1 图像reader和writer
`javax.imageio`包支持读写几种常见的图像文件格式，包括GIF、JPEG、PNG、BMP （Windows位图）和WBMP（无线位图）等。同时能够为其他格式添加reader和writer。

该库的基本用法非常简单。要加载一个图像，使用`ImageIO`类的静态方法`read()`：

```java
File f = ...;
BufferedImage image = ImageIO.read(f);
```

`ImageIO`类会根据文件类型选择合适的reader。为此它可能会查看文件扩展名和文件开头的“魔数”。如果没有找到合适的reader或者不能解码文件内容，`read()`方法将返回`null`。

将图像写入文件也一样简单：

```java
File f = ...;
String format = ...;
ImageIO.write(image, format, f);
```

其中`format`是标识图像格式的字符串，如`"JPEG"`或`"PNG"`。`ImageIO`类会选择合适的writer。

#### 11.4.1.1 获得适合图像文件类型的reader和writer
对于更高级的图像读写操作，首先需要获得合适的`ImageReader`或`ImageWriter`对象。`ImageIO`类枚举了匹配下列条件之一的reader和writer：
* 图像格式（如`"JPEG"`）
* 文件后缀（如`"jpg"`）
* MIME类型（如`"image/jpeg"`）

注释：MIME是多用途互联网邮件扩展(Multipurpose Internet Mail Extensions)的缩写。MIME标准定义了常用的数据格式，例如`image/jpeg`和`application/pdf`。（完整列表见 <https://www.iana.org/assignments/media-types/media-types.xhtml> ）

例如，可以像这样获得一个读取JPEG文件的reader：

```java
ImageReader reader = null;
Iterator<ImageReader> iter = ImageIO.getImageReadersByFormatName("JPEG");
if (iter.hasNext()) reader = iter.next();
```

`getImageReadersBySuffix()`和`getImageReadersByMIMEType()`方法分别枚举匹配文件扩展名和MIME类型的reader。

`ImageIO`类可能会找到多个reader都能读取某种特定的图像类型（上述方法返回一个迭代器）。在这种情况下，必须从中选择一个。要了解关于reader的更多信息，可以获得它的**服务提供者接口**(service provider interface)，然后可以得到供应商的名字和版本号：

```java
ImageReaderSpi spi = reader.getOriginatingProvider();
String vendor = spi.getVendorName();
String version = spi.getVersion();
```

或许这些信息能够帮助你决定如何选择，或者直接向用户展示一个reader列表让用户选择。目前来说，我们直接选择第一个reader。

在程序清单11-20的示例程序中，我们想找出所有可用reader支持的所有文件后缀，以用作文件对话框过滤器。为此可以使用静态方法`ImageIO.getReaderFileSuffixes()`：

```java
String[] extensions = ImageIO.getReaderFileSuffixes();
chooser.setFileFilter(new FileNameExtensionFilter("Image files", extensions));
```

保存文件更麻烦一些。我们希望向用户展示一个所有支持的图像类型的菜单。遗憾的是，`ImageIO.getWriterFormatNames()`方法返回了一个相当奇怪的列表，其中包含冗余的名字，例如

```
jpg, BMP, bmp, JPG, jpeg, wbmp, png, JPEG, PNG, WBMP, GIF, gif
```

而我们需要的是一个“首选”格式名称的列表。为此提供了一个辅助方法`getWriterFormats()`（见程序清单11-20）：查找与每种格式名称关联的第一个writer，然后获取它支持的第一个格式名称并转换为大写。

#### 11.4.1.2 读取和写入带有多个图像的文件
有些文件（特别是GIF动画）包含多个图像，而`ImageIO`类的`read()`方法只能读取单个图像。为了读取多个图像，需要将输入源（例如输入流或文件）转换成`ImageInputStream`。

```java
InputStream in = ...;
ImageInputStream imageIn = ImageIO.createImageInputStream(in);
```

然后把图像输入流附加到reader：

```java
reader.setInput(imageIn, true);
```

第二个参数为`true`表示输入模式是“只向前搜索”，否则采用随机访问（通过缓冲输入流或随机文件访问）。某些操作必须使用随机访问。例如，为了找出一个GIF文件中的图像个数，需要读取整个文件。这时如果想获取某个图像，就必须再次读取输入。

如果从文件读取图像，可以直接使用

```java
File f = ...;
ImageInputStream imageIn = ImageIO.createImageInputStream(f);
reader.setInput(imageIn);
```

一旦有了reader，就可以通过调用`read()`方法来读取图像。

```java
BufferedImage image = reader.read(index);
```

其中`index`是图像索引，从0开始。

如果输入采用“只向前搜索”模式，就应该不断地读取图像，直到`read()`方法抛出`IndexOutOfBoundsException`。否则，可以调用`getNumImages()`方法获得图像数量：

```java
int n = reader.getNumImages(true);
```

该方法只能用于随机访问模式。可能需要搜索整个文件才能确定图像数量（例如GIF文件）。如果该方法的参数`allowSearch`为`true`，则会进行必要的搜索并返回正确结果；否则，如果有之前的调用缓存的结果或者无需搜索（例如PNG文件只包含一个图像）则直接返回，如果无法在不搜索的情况下确定图像数量则返回-1（此时只能采用判断下标越界的方式）。

有些文件包含**缩略图**(thumbnail)，即用于预览的小版本图像。可以通过以下调用得到某个图像的缩略图数量：

```java
int count = reader.getNumThumbnails(index);
```

然后获得一个特定的缩略图：

```java
BufferedImage thumbnail = reader.getThumbnail(index, thumbnailIndex);
```

有时你希望在实际读取图像之前得到图像的大小，特别是当图像很大或者来自较慢的网络连接时。使用以下方法获得给定索引的图像的大小：

```java
int width = reader.getWidth(index);
int height = reader.getHeight(index);
```

要将多个图像写入一个文件中，首先需要一个`ImageWriter`。

```java
String format = ...;
ImageWriter writer = null;
Iterator<ImageWriter> iter = ImageIO.getImageWritersByFormatName(format);
if (iter.hasNext()) writer = iter.next();
```

接着，将输出流或文件转换成`ImageOutputStream`，并将其附加到writer。例如，

```java
File f = ...;
ImageOutputStream imageOut = ImageIO.createImageOutputStream(f);
writer.setOutput(imageOut);
```

必须将每个图像包装到`IIOImage`对象中（注：`IIO`表示 "Image I/O" ），可以提供可选的缩略图列表和图像元数据（例如压缩算法和颜色信息）。

```java
var iioImage = new IIOImage(images[i], null, null);
```

使用`write()`方法写入**第一个**图像：

```java
writer.write(iioImage);
```

对于后续的图像，使用

```java
if (writer.canInsertImage(i))
    writer.writeInsert(i, iioImage, null);
```

第三个参数可以是一个`ImageWriteParam`对象，用于设置图像写入细节（如平铺和压缩），如果为`null`则使用默认值。

并非所有文件格式都能够处理多个图像。在这种情况下，`canInsertImage()`方法当`i > 0`时返回`false`，只会保存一个图像。

程序清单11-20中的程序能够以不同的格式加载和保存图像文件。程序显示了多个图像（见下图），但没有缩略图。

[程序清单11-20 imageIO/ImageIOFrame.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch11/imageIO/ImageIOFrame.java)

![一个GIF动画图像](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/一个GIF动画图像.png)

### 11.4.2 图像操作
假设你希望改善一个图像的外观，就需要访问并替换其中的像素。或者你想要从头生成一个图像的像素，来展示物理测量或数学计算的结果。`BufferedImage`类使你可以控制图像中的像素，实现了`BufferedImageOp`接口的类可以对图像进行变换。

#### 11.4.2.1 构建栅格图像
你处理的大多数图像都是直接从图像文件中读取的。在本节中，将介绍逐个像素地构建图像的技术。

要创建一个图像，首先构造一个`BufferedImage`对象。

```java
var image = new BufferedImage(width, height, BufferedImage.TYPE_INT_ARGB);
```

然后，调用`getRaster()`方法获得一个`WritableRaster`类型的对象，使用该对象来访问和修改图像的像素。

```java
WritableRaster raster = image.getRaster();
```

使用`setPixel()`方法设置单个像素。这里的复杂性在于不能只是将像素设置为一个`Color`，必须知道缓冲图像如何指定颜色值，这取决于图像的类型（`BufferedImage`构造器的第三个参数）。如果图像类型为`TYPE_INT_ARGB`，那么每个像素由四个值描述——红、绿、蓝和透明度(alpha)，每个都在0~255之间。必须以四个整数的数组提供：

```java
int[] black = {0, 0, 0, 255};
raster.setPixel(i, j, black);
```

用Java 2D API的术语来说，这些值叫做像素的**样本值**(sample value)。

警告：还有数组参数类型为`float[]`和`double[]`的`setPixel()`方法。但是，这些数组中的值仍然是0~255，而不是0.0~1.0之间归一化的值。

可以使用`setPixels()`方法批量设置一个矩形区域内的像素，需要指定起始像素的位置、矩形的宽和高，以及一个包含所有像素样本值的数组（前4个元素表示第一个像素，第5~8个元素表示第二个像素，以此类推）：

```java
var pixels = new int[4 * width * height];
pixels[0] = ...; // red value for first pixel
pixels[1] = ...; // green value for first pixel
pixels[2] = ...; // blue value for first pixel
pixels[3] = ...; // alpha value for first pixel
...
raster.setPixels(x, y, width, height, pixels);
```

反过来，使用`getPixel()`方法读取一个像素，提供四个整数的数组来存放样本值：

```java
var sample = new int[4];
raster.getPixel(x, y, sample);
var color = new Color(sample[0], sample[1], sample[2], sample[3]);
```

可以使用`getPixels()`方法读取多个像素：

```java
var samples = new int[4 * width * height];
raster.getPixels(x, y, width, height, samples);
```

对于其他的图像类型，仍然可以使用`getPixel()`/`setPixel()`方法，不过必须知道这种图像类型是如何表示像素值的。

如果需要处理任意未知类型的图像则会更麻烦一些。每种图像类型都有一个**颜色模型**(color model)，能够在样本值数组与标准RGB颜色模型之间进行转换。

`BufferedImage`类的`getColorModel()`方法返回颜色模型：

```java
ColorModel model = image.getColorModel();
```

为了获得一个像素的颜色值，可以调用`Raster`类的`getDataElements()`方法。该方法返回一个`Object`，包含与特定颜色模型相关的颜色值描述（实际上是一个样本值的数组）。

```java
Object data = raster.getDataElements(x, y, null);
```

颜色模型能够将该对象转换成标准的ARGB值。`getRGB()`方法返回一个`int`（四个字节分别表示alpha、红、绿、蓝），可以使用这个整数来构造`Color`：

```java
int argb = model.getRGB(data);
var color = new Color(argb, true);
```

要把一个像素设置为特定的颜色，反向执行这些步骤。`Color`类的`getRGB()`方法生成一个`int`值。把这个值提供给`ColorModel`类的`getDataElements()`方法，返回一个描述颜色值的`Object`。再将这个对象传递给`WritableRaster`类的`setDataElements()`方法。

```java
int argb = color.getRGB();
Object data = model.getDataElements(argb, null);
raster.setDataElements(x, y, data);
```

为了演示如何使用这些方法从像素构建图像，我们绘制了一个曼德勃罗集(Mandelbrot set)，如下图所示。

![曼德勃罗集](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/曼德勃罗集.png)

曼德勃罗集的思想是平面上的每个点都与一个数字序列关联。如果序列是收敛的，则这个点属于该集合（涂成红色）；如果序列是发散的，则这个点不属于该集合（不涂色）。

下面是曼德勃罗集的数学定义。对于复平面上的每个点 $C = a + b\mathrm{i}$ ，定义序列：

$$
\begin{align}
z_0 &= 0 \newline
z_{n+1} &= z_n^2 + C
\end{align}
$$

使该序列收敛的所有点C构成的集合就是曼德勃罗集。

注：
* 不使用复数的等价定义如下。对于每个点(a, b)，定义从(0, 0)开始的序列，按以下公式迭代：

$$
\begin{align}
x_{new} &= x^2 - y^2 + a \newline
y_{new} &= 2xy + b
\end{align}
$$

如果这个序列是收敛的，则点(a, b)属于该集合，否则不属于该集合。

* 这个网站提供了曼德勃罗集的可视化：<https://mathigon.org/course/fractals/mandelbrot>
* 曼德勃罗集的几何形状是分形的，具有自相似性，即局部放大后与整体相似（如下图所示）。

<img src="/assets/images/java-note-v2ch11-advanced-swing-and-graphics/mandel-1.jpg" width="75%" alt="Mandelbrot set">

<img src="/assets/images/java-note-v2ch11-advanced-swing-and-graphics/mandel-10.jpg" width="75%" alt="Scaled Mandelbrot set">

程序清单11-21给出了代码。这个程序演示了如何使用`ColorModel`类将`Color`值转换成像素数据。这个过程与图像类型无关。你可以把缓冲图像的颜色类型改为`TYPE_BYTE_GRAY`，不需要改变程序中的任何代码，颜色模型会自动处理从颜色到样本值的转换。

[程序清单11-21 rasterImage/RasterImageFrame.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch11/rasterImage/RasterImageFrame.java)

#### 11.4.2.2 过滤图像
上一节介绍了如何从头开始构建图像。然而，通常是因为另一个原因访问图像数据：从某些方面对已有图像进行改进。当然，可以使用上一节中的`getPixel()`/`getDataElements()`方法来读取和处理图像数据并写回。幸运的是，Java 2D API已经提供了许多**过滤器**(filter)，能够执行常见的图像处理操作。

所有图像操作（即过滤器）都实现了`BufferedImageOp`接口。构造操作对象后只需调用`filter()`方法即可转换图像。

```java
BufferedImageOp op = ...;
BufferedImage filteredImage = new BufferedImage(image.getWidth(), image.getHeight(), image.getType());
op.filter(image, filteredImage);
```

有些操作可以原地转换图像（即`op.filter(image, image)`），但大多数不能。

这五个类实现了该接口：`AffineTransformOp`, `ConvolveOp`, `ColorConvertOp`, `RescaleOp`和`LookupOp`。

`AffineTransformOp`对像素执行仿射变换。例如，可以像这样将一个图像绕着其中心旋转：

```java
AffineTransform transform = AffineTransform.getRotateInstance(
    Math.toRadians(angle), image.getWidth() / 2, image.getHeight() / 2);
var op = new AffineTransformOp(transform, interpolation);
op.filter(image, filteredImage);
```

`AffineTransformOp`构造器需要一个仿射变换（11.3.6节已介绍过）和一个**插值**(interpolation)策略。如果源像素被变换到两个像素之间的位置（例如旋转），就必须使用插值来确定目标像素。有三种插值策略：`TYPE_BICUBIC`、`TYPE_BILINEAR`和`TYPE_NEAREST_NEIGHBOR`。双三次插值需要的时间较长，但效果比另外两个更好。

程序清单11-22的程序可以把图像旋转5°（见下图）。

![旋转的图像](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/旋转的图像.png)

`RescaleOp`对图像中的每个颜色分量执行重缩放(rescaling)操作：$x_{new} = ax + b$ （alpha分量不受影响）。用a > 1缩放的效果是使图像变亮（注：不是调整图像大小）。通过指定缩放参数来构造`RescaleOp`：

```java
float a = 1.1f;
float b = 20.0f;
var op = new RescaleOp(a, b, null);
```

也可以为每个颜色分量提供单独的缩放参数，参见API文档。

`LookupOp`操作允许指定任意的样本值映射。其构造器需要一个`LookupTable`类型的对象。`LookupTable`是抽象类，有两个具体子类：`ByteLookupTable`和`ShortLookupTable`。因为RGB颜色可以用`byte`表示（0~255），`ByteLookupTable`应该就足够了。但是，由于bug [JDK-6183251](https://bugs.java.com/bugdatabase/JDK-6183251)，我们使用`ShortLookupTable`。在示例程序中，我们计算了所有颜色的反色，即将c改为255-c：

```java
var negative = new short[256];
for (int i = 0; i < 256; i++)
    negative[i] = (short) (255 - i);
var table = new ShortLookupTable(0, negative);
var op = new LookupOp(table, null);
```

查表操作分别应用于每个颜色分量，但不应用于alpha分量。也可以为每个颜色分量提供不同的映射表，参见API文档。

注释：不能将`LookupOp`用于带有索引颜色模型的图像。在这种图像中，每个样本值都是调色板中的偏移量。

`ColorConvertOp`用于颜色空间转换，这里不讨论。

最强大的操作是`ConvolveOp`，用于执行**卷积**(convolution)变换。其基本思想很简单。例如，考虑**模糊过滤器**(blur filter)（见下图）。

![模糊的图像](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/模糊的图像.png)

模糊效果是通过用每个像素及其8个相邻像素的平均值来替换该像素实现的。从数学上讲，这种平均可以表示为以这个矩阵为**核**(kernel)的卷积操作：

$$
\begin{bmatrix}
\frac{1}{9} & \frac{1}{9} & \frac{1}{9} \newline
\frac{1}{9} & \frac{1}{9} & \frac{1}{9} \newline
\frac{1}{9} & \frac{1}{9} & \frac{1}{9}
\end{bmatrix}
$$

卷积核是一个矩阵，表示相邻像素的权重。下面的卷积核用于进行**边缘检测**(edge detection)，定位颜色变化的区域（见下图）：

$$
\begin{bmatrix}
0 & -1 & 0 \newline
-1 & 4 & -1 \newline
0 & -1 & 0
\end{bmatrix}
$$

![边缘检测并反色](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/边缘检测并反色.png)

要构造一个卷积操作，首先设置包含卷积核各个值的数组并构造一个`Kernel`对象。然后使用卷积核构造一个`ConvolveOp`对象并进行过滤。

```java
float[] elements = {
    0.0f, -1.0f, 0.0f,
    -1.0f, 4.0f, -1.0f,
    0.0f, -1.0f, 0.0f
};
var kernel = new Kernel(3, 3, elements);
var op = new ConvolveOp(kernel);
op.filter(image, filteredImage);
```

程序清单11-22中的程序允许用户加载一个GIF或JPEG图像并执行各种图像操作。

[程序清单11-22 imageProcessing/ImageProcessingFrame.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch11/imageProcessing/ImageProcessingFrame.java)

## 11.5 打印
在以下几节中，将介绍如何在单页纸上打印图形，如何管理多页打印输出，以及如何将打印内容保存为PostScript文件。

### 11.5.1 图形打印
在本节中，将处理最常见的打印情景：打印2D图形（可能包含不同字体的文本）。

要生成打印输出，需要执行以下两个步骤：
* 提供一个实现了`Printable`接口的对象。
* 启动打印作业。

`Printable`接口只有一个方法：

```java
int print(Graphics g, PageFormat format, int page)
```

每当打印引擎需要打印一页时都会调用该方法。你的代码将要打印的图形和文本绘制到图形上下文`g`。`format`指定了纸张大小和页边距，`page`是要打印的页码（从0开始）。返回值稍后介绍。

要启动打印作业，需要使用`PrinterJob`类。首先，调用静态方法`getPrinterJob()`得到一个打印作业对象。然后设置要打印的`Printable`对象。

```java
Printable canvas = ...;
PrinterJob job = PrinterJob.getPrinterJob();
job.setPrintable(canvas);
```

警告：`PrintJob`类处理的是JDK 1.1风格的打印，现在已经过时了。不要把它与`PrinterJob`类混淆。

在开始打印作业之前，应该调用`printDialog()`方法显示打印对话框（见下图）。这个对话框让用户可以选择要使用的打印机，要打印的页码范围，以及各种打印机设置。

![打印对话框](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/打印对话框.png)

用一个实现了`PrintRequestAttributeSet`接口的类的对象来收集打印机设置，例如`HashPrintRequestAttributeSet`类。

```java
var attributes = new HashPrintRequestAttributeSet();
```

使用`add()`方法添加属性设置，并将`attributes`对象传递给`printDialog()`方法。

如果用户点击了OK，`printDialog()`方法返回`true`；如果用户取消了对话框则返回`false`。如果用户确认了，就调用`PrinterJob`类的`print()`方法启动打印过程。下面是打印代码的基本框架：

```java
if (job.printDialog(attributes)) {
    try {
        job.print(attributes);
    }
    catch (PrinterException e) {
        ...
    }
}
```

注释：在JDK 1.4之前，打印系统使用的是平台本地的打印对话框。要展示本地打印对话框，调用无参数的`printDialog()`方法（不能收集属性设置）。

在打印过程中，`PrinterJob`类的`print()`方法会不断地调用关联的`Printable`对象的`print()`方法。只要后者返回`Printable.PAGE_EXISTS`，打印作业就继续；当返回`Printable.NO_SUCH_PAGE`时，打印作业停止。因此，打印作业预先并不知道准确的页数，打印对话框无法显示正确的页码范围，而只能显示 "Pages 1 to 1" 。在下一节中将介绍如何通过为打印作业提供一个`Book`对象来避免这个缺陷。

在打印过程中，打印作业可能会对**同一页**多次调用`print()`方法（例如，对于使用**条带打印**(banding)的打印机，每次将图形上下文的剪裁区域设置为所需要的条带）。因此不应该在`print()`方法内对页进行计数，而是始终依赖页码参数。

警告：`print()`方法的`Graphics`参数会按照页边距进行剪裁。如果想进一步限制剪裁区域，应该调用`clip()`而不是`setClip()`。如果必须要移除剪裁区域，务必在`print()`方法开始处调用`getClip()`并在之后还原。

`print()`方法的`PageFormat`参数包含有关当前打印页的信息。其`getWidth()`和`getHeight()`方法返回纸张大小，单位是**磅**(point)（1磅=1/72英寸，1英寸=2.54厘米）。

但并不是所有纸张区域都是可打印的，纸张的周围会有页边距(margin)。`getImageableWidth()`和`getImageableHeight()`方法返回实际可打印区域的大小。但是，页边距不一定是对称的，所以还必须知道可打印区域的左上角（见下图），这可以通过`getImageableX()`和`getImageableY()`方法获得。

![页面格式计量](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/页面格式计量.png)

提示：`print()`方法接收的图形上下文的剪裁区域已经排除了页边距，但坐标系的原点仍然是纸张的左上角。应该将坐标系平移到可打印区域的左上角。只需在`print()`方法开头添加

```java
g.translate(pageFormat.getImageableX(), pageFormat.getImageableY());
```

如果希望用户选择页边距设置，或者在纵向和横向之间切换，调用`PrinterJob`类的`pageDialog()`方法：

```java
PageFormat format = job.pageDialog(attributes);
```

注释：打印对话框的一个选项卡包含了页面设置对话框（见下图）。你可能仍然希望在打印之前为用户提供设置页面格式的选项，特别是当你的程序显示待打印页面的预览时。`pageDialog()`方法返回包含用户设置的`PageFormat`对象。

![页面设置对话框](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/页面设置对话框.png)

程序清单11-23和11-24中的程序展示了如何在屏幕和打印页面上绘制同一组形状。这个示例显示并打印了11.3.7中的图形，即使用字符串 "Hello World" 的轮廓作为一组线条的剪裁区域。点击Print按钮开始打印，或者点击Page setup按钮打开页面设置对话框。

注释：要显示本地页面设置对话框，将一个默认的`PageFormat`对象传递给`pageDialog()`方法。该方法会克隆这个对象，根据用户在对话框中的选择来修改它，然后返回克隆的对象。

```java
PageFormat defaultFormat = printJob.defaultPage();
PageFormat selectedFormat = printJob.pageDialog(defaultFormat);
```

[程序清单11-23 print/PrintTestFrame.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch11/print/PrintTestFrame.java)

[程序清单11-24 print/PrintComponent.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch11/print/PrintComponent.java)

### 11.5.2 多页打印
在实际中，通常不会将原始的`Printable`对象传递给打印作业，而应该获得一个实现了`Pageable`接口的类的对象。Java平台提供了一个这样的类，叫做`Book`。一个book由多个**节**(section)组成，每一节都是一个`Printable`对象。要构建一个book，通过`append()`方法添加`Printable`对象及其页数：

```java
var book = new Book();
Printable coverPage = ...;
Printable bodyPages = ...;
book.append(coverPage, pageFormat); // append 1 page
book.append(bodyPages, pageFormat, pageCount);
```

然后，使用`setPageable()`方法把`Book`对象传递给打印作业：

```java
printJob.setPageable(book);
```

现在打印作业确切地知道要打印的页数，因此打印对话框可以显示准确的页码范围，用户可以选择整个范围或者子范围。

警告：当打印作业调用`Printable`对象的`print()`方法时，它传递的是book的当前页码，而不是每一节的。这意味着每一节必须知道之前所有节的页数才能计算出正确的页码。

程序清单11-26展示了如何产生多页打印输出。该程序用非常大的字符在多个页面上打印了一条消息。可以剪掉页边距并将页面粘在一起形成一个横幅(banner)（见下图）。

![一个横幅](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/一个横幅.png)

`Banner`类的`getPageCount`方法首先用页面可打印高度除以字符串高度得到缩放因子，然后将字符串宽度乘以该因子并除以页面可打印宽度，向上取整后得到页数。由于字符被分散到多个页面上，在打印每一页时，需要将字符串的左上角向左平移，然后设置与当前页面相等的剪裁矩形。

这个程序还展示了变换操作的另一种用法：显示打印预览。

[程序清单11-25 book/BookTestFrame.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch11/book/BookTestFrame.java)

[程序清单11-26 book/Banner.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch11/book/Banner.java)

[程序清单11-27 book/PrintPreviewDialog.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch11/book/PrintPreviewDialog.java)

[程序清单11-28 book/PrintPreviewCanvas.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch11/book/PrintPreviewCanvas.java)

### 11.5.3 打印服务
到目前为止，已经介绍了如何打印2D图形。但是，打印API提供了更大的灵活性。该API定义了大量的数据类型，并且可以查找能够打印这些数据类型的**打印服务**(print service)。数据类型包括：
* GIF、JPEG或PNG格式的图像
* 文本、HTML、PostScript或PDF格式的文档
* 原始打印机代码数据
* 实现了`Printable`、`Pageable`或`RenderableImage`的类的对象

数据本身可以存放在字节或字符源中，例如输入流、URL或数组。**文档风格**(document flavor)描述了数据源和数据类型的组合。`DocFlavor`类为各种数据源定义了许多内部类（同时也是子类），每个内部类都定义了指定风格的常量。例如，常量`DocFlavor.INPUT_STREAM.GIF`描述了从输入流读取的GIF图像。完整列表参见`DocFlavor`类的API文档。

假设你想打印一个文件中的GIF图像。首先，确认是否有能够处理该任务的打印服务。`PrintServiceLookup`类的静态方法`lookupPrintServices()`返回一个能够处理给定文档风格的`PrintService`对象的数组。

```java
DocFlavor flavor = DocFlavor.INPUT_STREAM.GIF;
PrintService[] services = PrintServiceLookup.lookupPrintServices(flavor, null);
```

第二个参数是打印服务必须支持的属性，将在下一节介绍。

如果查找返回了多个打印服务，需要从中选择一个。可以调用`PrintService`类的`getName()`方法获得打印机名称并让用户选择。

接着，从打印服务获取一个文档打印作业：

```java
DocPrintJob job = services[i].createPrintJob();
```

为了打印，需要一个实现了`Doc`接口的类的对象。为此Java提供了一个`SimpleDoc`类。其构造器需要数据源对象、文档风格和可选的属性集。例如，

```java
var in = new FileInputStream(fileName);
var doc = new SimpleDoc(in, flavor, null);
```

最后，就可以打印了：

```java
job.print(doc, null);
```

注意，这个打印过程与上一节的完全不同。这里没有通过打印对话框的用户交互。例如，可以实现一个服务器端打印机制，用户可以通过Web表单提交打印作业。

[printService/PrintServiceTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch11/printService/PrintServiceTest.java)

### 11.5.4 流打印服务
打印服务将打印数据发送给打印机。流打印服务生成同样的打印数据，但将其发送到流（例如为了延迟打印或者由其他程序处理）。尤其是，如果打印数据格式是PostScript，那么将其保存到文件可能很有用，因为许多程序都能够处理PostScript文件(.ps)。Java平台包括一个流打印服务，能够从图像和2D图形产生PostScript输出。可以在任何系统上使用这个服务，即使没有本地打印机。

查找流打印服务比普通打印服务复杂一些，需要指定文档风格和流输出的MIME类型，返回一个`StreamPrintServiceFactory`数组。

```java
DocFlavor flavor = DocFlavor.SERVICE_FORMATTED.PRINTABLE;
String mimeType = "application/postscript";
StreamPrintServiceFactory[] factories =
    StreamPrintServiceFactory.lookupStreamPrintServiceFactories(flavor, mimeType);
```

使用输出流调用工厂对象的`getPrintService()`方法来得到一个`StreamPrintService`对象。

```java
var out = new FileOutputStream(fileName);
StreamPrintService service = factories[0].getPrintService(out);
```

`StreamPrintService`类是`PrintService`的子类。要产生打印输出，只需按照上一节的步骤即可。

程序清单11-29中的程序演示了如何使用流打印服务将Java 2D形状打印到PostScript文件。然后使用外部工具可以很容易地将结果转换成PDF或EPS。（遗憾的是，Java不支持直接打印到PDF。）

[程序清单11-29 streamPrintService/StreamPrintServiceTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch11/streamPrintService/StreamPrintServiceTest.java)

注释：在这个示例中，我们在`Graphics2D`对象上调用了绘制Java 2D形状的`draw()`方法。如果想要绘制一个组件（例如表格或树），那么使用下面的代码：

```java
private static int IMAGE_WIDTH = component.getWidth();
private static int IMAGE_HEIGHT = component.getHeight();
public static void draw(Graphics2D g2) { component.paint(g2); }
```

### 11.5.5 打印属性
打印服务API包含一组复杂的接口和类，用于指定不同种类的属性。有四组重要的属性，前两组指定对打印机的请求：
* **打印请求属性**(print request attribute)为一个打印作业中的所有文档对象指定特定的属性，例如双面打印或纸张大小。
* **文档属性**(doc attribute)是仅应用于单个文档对象的请求属性。

另外两组属性包含关于打印机和作业状态的信息：
* **打印服务属性**(print service attribute)提供关于打印服务的信息，例如打印机品牌型号、打印机当前是否接受作业。
* **打印作业属性**(print job attribute)提供关于特定打印作业状态的信息，例如作业是否已完成。

`Attribute`接口及其子接口用于描述各种属性：

```
PrintRequestAttribute
DocAttribute
PrintServiceAttribute
PrintJobAttribute
SupportedValuesAttribute
```

每个属性类都实现了上面的一个或几个接口。例如，`Copies`类描述了打印份数，该类实现了`PrintRequestAttribute`和`PrintJobAttribute`两个接口：打印请求可以指定要打印的份数，打印作业状态可以包含实际打印的份数。

`SupportedValuesAttribute`接口表示打印服务的能力。例如，`CopiesSupported`类实现了该接口，可以描述某个打印机支持打印1~99份。

下图展示了属性类层次结构的类图。

![属性类层次结构](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/属性类层次结构.png)

除了单个属性，打印API还定义了表示属性集的接口和类。`AttributeSet`接口有4个子接口：

```
PrintRequestAttributeSet
DocAttributeSet
PrintServiceAttributeSet
PrintJobAttributeSet
```

每个接口都有一个`Hash`开头的实现类。下图展示了属性集类层次结构的类图。

![属性集类层次结构](/assets/images/java-note-v2ch11-advanced-swing-and-graphics/属性集类层次结构.png)

例如，可以像这样构造一个打印请求属性集：

```java
var attributes = new HashPrintRequestAttributeSet();
```

子接口会检查属性类型是否正确。例如，`DocAttributeSet`的`add()`方法只接受实现了`DocAttribute`接口的对象，否则会抛出异常。

属性集是一种特殊的映射，其键是`Class`类型，值属于实现了`Attribute`接口的类（注：`HashAttributeSet`的底层就是一个`HashMap<Class<?>, Attribute>`）。例如，如果向属性集中插入一个`Copies`对象，那么键就是`Copies.class`。这个键称为属性的**类别**(category)。`Attribute`接口的`getCategory()`方法返回属性的类别。

向属性集中添加属性时会自动获取类别。例如，

```java
attributes.add(new Copies(10));
```

大致等价于

```java
var attr = new Copies(10);
attrMap.put(attr.getCategory(), attr);
```

如果后续添加另一个具有相同类别的属性，就会覆盖前一个。

要获取一个属性，需要使用其类别作为键。例如，

```java
AttributeSet attributes = job.getAttributes();
var copies = (Copies) attribute.get(Copies.class);
```

最后，属性是按照其值的类型组织的。`Copies`属性有整数值，因此扩展了`IntegerSyntax`类，`getValue()`方法返回属性的值。例如：

```java
int n = copies.getValue();
```

`TextSyntax`、`DateTimeSyntax`和`URISyntax`类分别封装了字符串、日期时间和URI值。

许多属性都接受有限数量的值。例如，`PrintQuality`属性有三个值：`DRAFT`、`NORMAL`和`HIGH`。这种属性扩展了`EnumSyntax`类。添加属性时，只需使用常量名字：

```java
attributes.add(PrintQuality.HIGH);
```

可以使用`==`和`!=`检查属性的值：

```java
if (attributes.get(PrintQuality.class) == PrintQuality.HIGH)
    ...
```

打印属性的完整列表参见`Attribute`类的API文档。

注释：属性的数量很多，其中许多都是专用的。大多数属性都来源于互联网打印协议1.1 ([RFC 2911](https://www.ietf.org/rfc/rfc2911.txt))。
