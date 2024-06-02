---
title: 《Python基础教程》笔记 第21章 项目2：画幅好画
date: 2024-05-27 22:59:06 +0800
categories: [Python, Beginning Python]
tags: [python, plot]
---
本章介绍如何使用Python创建图形。具体地说，你将创建一个PDF文件，其中包含的图表对从文本文件读取的数据进行可视化。

## 21.1 问题描述
Python很善于分析数据。俗话说，一图胜千言(A picture is worth a thousand words)。在本章中，你将学习ReportLab包的基本知识，它让你能够像创建文本文件一样轻松地创建PDF格式（和其他格式）的图形和文档。

本章将根据太阳黑子数据创建一个折线图。程序应该具备如下功能：
* 从网上下载数据文件
* 解析数据文件并提取感兴趣的部分
* 基于这些数据创建PDF图形

## 21.2 有用的工具
在这个项目中，最重要的工具是图形生成包。这样的包有很多，这里选择的是ReportLab (<https://www.reportlab.com/>)，因为它易于使用，并且提供了丰富的PDF图形和文档生成功能。如果想更进一步，可以考虑使用PYX图形包(<https://pyx-project.org/>)，其功能非常强大，并且支持基于TEX的排版。

要获取ReportLab包，可以使用`pip`来安装。其官网包含文档和示例。

```shell
$ pip install reportlab
```

## 21.3 准备工作
开始编程之前，需要一些用来测试程序的数据。本章选择了有关太阳黑子的数据，这些数据可以从空间天气预报中心的网站(<https://www.swpc.noaa.gov/>)下载。示例中使用的数据可以在 <ftp://ftp.swpc.noaa.gov/pub/weekly/Predict.txt> 找到。

注：
* 书中给出的原链接已不可用，提示的新地址为<https://www.swpc.noaa.gov/products/predicted-sunspot-number-and-radio-flux>，但新数据是JSON格式，而不是书中使用的数据格式。在[Wayback Machine](https://web.archive.org/web/20141026073827/http://www.swpc.noaa.gov/ftpdir/weekly/Predict.txt)上可以找到最后一次更新（2014年10月）的与书中格式一致的数据。
* NOAA网站上也提供了基于这些数据的可视化图表：<https://www.swpc.noaa.gov/products/solar-cycle-progression>

这个数据文件包含有关太阳黑子和辐射流量的数据。得到这个文件后，就可以着手解决问题了。

下面是这个文件的一部分，从中能看到数据是什么样子。

```
#         Predicted Sunspot Number And Radio Flux Values
#                     With Expected Ranges
#
#         -----Sunspot Number------  ----10.7 cm Radio Flux----
# YR MO   PREDICTED    HIGH    LOW   PREDICTED    HIGH    LOW
#--------------------------------------------------------------
2016 03        30.9    31.9    29.9       96.9    97.9    95.9
2016 04        30.5    32.5    28.5       96.1    97.1    95.1
2016 05        30.4    33.4    27.4       94.9    96.9    92.9
2016 06        30.3    35.3    25.3       93.2    96.2    90.2
2016 07        30.2    35.2    25.2       91.6    95.6    87.6
2016 08        30.0    36.0    24.0       90.3    94.3    86.3
2016 09        29.8    36.8    22.8       89.5    94.5    84.5
2016 10        30.0    37.0    23.0       88.9    94.9    82.9
2016 11        30.1    38.1    22.1       88.1    95.1    81.1
2016 12        30.5    39.5    21.5       87.8    95.8    79.8
```

完整数据文件：[Predict.txt](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch21/Predict.txt)

## 21.4 初次实现
在初次实现中，直接以元组列表的形式将数据放在源代码中，以便于访问。例如：

```python
data = [
    # Year Month Predicted High Low
    (2016, 03, 30.9, 31.9, 29.9),
    (2016, 04, 30.5, 32.5, 28.5),
    # Add more data here
]
```

### 21.4.1 使用ReportLab绘图
我们将使用ReportLab的高级图形框架（`reportlab.graphics`包），它能够创建各种形状并添加到`Drawing`对象，之后可以将其输出到PDF文件中。

代码清单21-1是一个示例程序，它在一个100×100的PDF图形中央绘制字符串 "Hello, world!" 。

[代码清单21-1 一个简单的ReportLab程序](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch21/hello_report.py)

![一个简单的ReportLab图形](/assets/images/python-note-ch21-project-2-painting-a-pretty-picture/一个简单的ReportLab图形.png)

### 21.4.2 绘制折线
为了绘制太阳黑子折线图，需要绘制一些相连的直线。ReportLab为此提供了一个专门的类：`PolyLine`。

创建`PolyLine`的第一个参数是坐标列表，形如`[(x0, y0), (x1, y1), ...]`，其中每对x和y坐标指定折线上的一个点。下图是一条简单的折线。

注：原点在左下角（与数学坐标系一样）而不是左上角（GUI坐标系）。

![折线](/assets/images/python-note-ch21-project-2-painting-a-pretty-picture/折线.png)

要绘制折线图，必须为数据集中的每列数据绘制一条折线。折线上每个点的横坐标是时间（年月），纵坐标是值（太阳黑子数）。要获得一列的值，可以使用列表推导式。

```python
pred = [row[2] for row in data]
```

每行的时间需要根据年和月来计算，例如year+month/12。

有了值和时间之后，可以像这样添加折线：

```python
drawing.add(PolyLine(list(zip(times, pred)), strokeColor=colors.blue))
```

### 21.4.3 编写原型
现在可以编写程序的第一个版本了，其源代码如代码清单21-2所示。

[代码清单21-2 太阳黑子图形程序的原型](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch21/sunspots_proto.py)

![一个简单的太阳黑子图](/assets/images/python-note-ch21-project-2-painting-a-pretty-picture/一个简单的太阳黑子图.png)

## 21.5 再次实现
我们从原型中学习到了什么呢？我们明白了使用ReportLab进行绘图的基本知识，还知道了如何提取数据以便于绘图。然而，这个程序存在一些缺陷。为了让内容处于正确的位置，我对值和时间做了一些专门的(ad hoc)修改（魔数）。另外，这个程序并没有从任何地方读取数据。

不同于项目1，这个项目的再次实现并不会比初次实现复杂很多，只是做了增量改进：使用更合适的ReportLab特性，并从网上获取数据。

### 21.5.1 获取数据
第14章介绍过，可以使用标准模块`urllib`从网上获取文件。其中的函数`urlopen()`与`open()`很类似，但将URL（而不是文件名）作为参数。打开文件并读取内容时，要过滤掉不需要的内容。该文件包含空行和以特殊字符（#和:）开头的行，程序应忽略这些行（见21.3节的示例）。

注意：如果你使用的是自己的数据源（或者等你阅读本书时，太阳黑子文件的数据格式已经发生了变化），就需要相应地修改代码。

### 21.5.2 使用LinePlot类
在这种情况下，最好浏览一下文档，看看是否已经有了能够完成任务的特性，这样就无需全部自己实现了。幸运的是，确实有这样的特性：模块`reportlab.graphics.charts.lineplots`中的`LinePlot`类。

`LinePlot`实例化不需要任何参数，然后设置其属性并添加到`Drawing`对象。需要设置的主要属性包括`x`、`y`、`height`、`width`和`data`。前4个属性不言自明，`data`是点列表的列表（表示要绘制的多条折线）。

为了加以区分，我们设置了每条线的颜色。最终代码如代码清单21-3所示。

[代码清单21-3 最终的太阳黑子程序](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch21/sunspots.py)

![最终的太阳黑子图](/assets/images/python-note-ch21-project-2-painting-a-pretty-picture/最终的太阳黑子图.png)

## 21.6 进一步探索
Python图形和绘图包有很多。除ReportLab外，另一个不错的选择是本章前面提到的PYX。无论使用哪个绘图包，都可以尝试将自动生成的图形嵌入文档（甚至也生成文档的各个部分）。可以使用第20章的技术给文本添加标记。如果要创建PDF文档，可以使用ReportLab中的Platypus（也可以使用LATEX等排版系统来集成PDF图形）。如果要创建网页，也有很多使用Python创建像素映射图形（例如GIF或PNG）的方法——可以在网上搜索这个主题。

如果你的主要目标是绘制数据图表（就像这个项目一样），那么除ReportLab和PYX外还有很多其他的选择，其中一个很好的选择是Matplotlib (<https://matplotlib.org/>)。
