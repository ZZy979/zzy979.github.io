---
title: 《Python基础教程》笔记 第29章 项目10：自制街机游戏
date: 2024-07-06 11:24:22 +0800
categories: [Python, Beginning Python]
tags: [python, pygame]
---
欢迎来到最后一个项目。在本章中，你将学习如何使用Pygame (<https://www.pygame.org/>)，这个库让你能够使用Python编写功能齐全的街机游戏。Pygame虽然易于使用，功能却非常强大。它由多个组件组成，Pygame文档(<https://www.pygame.org/docs/>)做了详尽的介绍。

## 29.1 问题描述
游戏的基本设计过程与其他程序类似，但开发对象模型前，必须先设计游戏本身：游戏的角色、设定和目标等。

这里将创建的游戏是基于巨蟒剧团的著名短剧 "Self-Defense Against Fresh Fruit" 改编的。在这个短剧中，军士长John Cleese指挥士兵使用防守战术抵御入侵者使用新鲜水果（例如石榴、芒果、青梅和香蕉）发起的进攻。防守战术包括使用枪支、放老虎以及在敌人头顶扔下16吨重的秤砣。在这个游戏中，我们将反过来——玩家控制一个香蕉，躲开从天而降的秤砣，尽力在防御战中存活下来。这个游戏叫做Squish（压扁）比较合适。

这个项目的目标是围绕着游戏设计展开的。这款游戏应该像设计的那样：香蕉应该可以移动，秤砣应该从天而降。另外，与往常一样，代码应该是模块化、易于扩展的。一个重要的需求是，设计应该包含**游戏状态**（例如游戏简介、不同关卡和游戏结束），并且可以很容易地添加新状态。

## 29.2 有用的工具
这个项目需要的唯一新工具就是Pygame。最简单的方式是使用`pip`安装Pygame。

```shell
$ pip install pygame
```

Pygame包含多个模块，接下来的几小节将描述需要用到的模块（只讨论需要用到的具体函数或类）。

### pygame
`pygame`模块自动导入其他所有的Pygame模块，因此如果在程序开头放置`import pygame`，就能访问其他模块（例如`pygame.display`和`pygame.font`）。

`pygame`模块包含`Surface`类。`Surface`对象就是一个指定尺寸的空图像，用于绘制和传输。传输（调用`Surface`对象的`blit()`方法）意味着将一个`Surface`的内容转移到另一个。（单词 "blit" 是从技术术语**块传输**(block transfer)的缩写BLT衍生而来的。）

`init()`函数是Pygame游戏的核心，必须在游戏进入主事件循环前调用。这个函数自动初始化其他所有模块（例如`font`和`image`）。

如果要捕获Pygame特有的错误，就需要使用`error`类。

### pygame.locals
`pygame.locals`模块包含事件类型、键、视频模式等的名称。可以导入这个模块的所有内容(`from pygame.locals import *`)，但如果知道需要的名称，应该导入更具体的内容（例如`from pygame.locals import FULLSCREEN`）。

### pygame.display
`pygame.display`模块包含处理Pygame显示的函数。在这个项目中，需要用到以下函数：
* `flip()`：更新显示。一般来说，修改当前屏幕要经过两步。首先，对`get_surface()`返回的`Surface`对象做必要的修改，然后调用`flip()`来更新显示以反映所做的修改。
* `update()`：只想更新屏幕的一部分时，使用这个函数而不是`flip()`。
* `set_mode()`：设置显示的尺寸和类型（普通窗口或全屏）。
* `set_caption()`：设置Pygame程序的窗口标题。
* `get_surface()`：返回一个`Surface`对象（屏幕），可以在其中绘制图形，再调用`pygame.display.flip()`或`pygame.display.blit()`。

### pygame.font
`pygame.font`模块包含`Font`类。`Font`对象表示不同的字体，可用于将文本渲染为可在Pygame中作为普通图形使用的图像。

### pygame.sprite
`pygame.sprite`模块包含两个非常重要的类：`Sprite`和`Group`。

`Sprite`类是所有可见游戏对象（在这个项目中是香蕉和秤砣）的基类。要实现自定义的游戏对象，需要继承`Sprite`，覆盖构造函数以设置`image`和`rect`属性（决定了外观和位置），再覆盖`update()`方法（在`Sprite`需要更新时调用）。

`Group`类（及其子类）的实例用作`Sprite`对象的容器。在简单的游戏（例如这个项目）中，只需创建一个组，并将所有的`Sprite`对象添加到其中。这样，当你调用`Group`对象的`update()`方法时，将自动调用所有`Sprite`对象的`update()`方法。另外，`Group`对象的`clear()`方法用于清除它包含的所有`Sprite`对象，而`draw()`方法可用于绘制所有的`Sprite`对象。

在这个项目中，将使用`Group`的子类`RenderUpdates`，其`draw()`方法返回受影响的矩形列表。可以将这个列表传递给`pygame.display.update()`，以只更新需要更新的部分。这可能会极大地改善游戏的性能。

### pygame.mouse
在这个项目中，只使用`pygame.mouse`模块来做两件事：使用`pygame.mouse.set_visible(False)`隐藏鼠标，以及使用`pygame.mouse.get_pos()`获取鼠标位置。

### pygame.event
`pygame.event`模块跟踪各种事件，例如鼠标单击、鼠标移动、按下或松开键盘等。要获取最近的事件列表，可以使用`pygame.event.get()`函数。

### pygame.image
`pygame.image`模块用于处理图像，例如GIF、PNG、JPEG等其他文件格式。在这个项目中，只需要使用`load()`函数，它读取图像文件并创建一个包含该图像的`Surface`。

注：官方教程提供了一个弹跳球的动画示例： <https://www.pygame.org/docs/tut/PygameIntro.html> ，代码如下。

[弹跳球动画](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch29/bouncing_ball.py)

## 29.3 准备工作
在编写游戏的第一个原型之前需要做些准备工作。首先，确保安装了Pygame。

还需要准备几幅图像（可以从 <https://openclipart.org/> 或Google上搜索）。本章的游戏需要两幅图像，分别表示16吨秤砣和香蕉，如下图所示。图像的尺寸最好在100×100到200×200之间，应该使用常见的图像文件格式，例如GIF、PNG或JPEG。

<figure>
  <img src="/assets/images/python-note-ch29-project-10-do-it-yourself-arcade-game/weight.png" alt="16吨秤砣" />
  <img src="/assets/images/python-note-ch29-project-10-do-it-yourself-arcade-game/banana.png" alt="香蕉" />
</figure>

## 29.4 初次实现
使用Pygame这样的新工具时，应该让第一个原型尽可能简单，并专注于学习新工具的基本知识，而不是程序本身的细节，这样做通常大有裨益。因此，游戏Squish的第一个版本只是秤砣从天而降的动画。需要的步骤如下：
1. 使用`pygame.init()`、`pygame.display.set_mode()`和`pygame.mouse.set_visible()`初始化Pygame。
2. 加载秤砣图像。
3. 使用这个图像创建自定义的`Weight`类（`Sprite`的子类）的实例，将这个对象添加到名为`sprites`的`RenderUpdates`组中。
4. 使用`pygame.display.get_surface()`获取屏幕表面，使用`fill()`方法将屏幕填充为白色，并调用`pygame.display.flip()`显示所做的修改。
5. 使用`pygame.event.get()`获取所有的最近事件，并依次检查这些事件。如果发现`QUIT`类型的事件，或者按下Escape键(`K_ESCAPE`)触发的`KEYDOWN`类型的事件，就退出程序。（事件类型和键分别存储在事件对象的`type`和`key`属性中。`QUIT`等常量可以从`pygame.locals`模块导入。）
6. 调用`sprites`组的`clear()`和`update()`方法。`clear()`方法使用回调函数来清除所有的`Sprite`对象（这里是秤砣），而`update()`方法调用`Weight`实例的`update()`方法（后者必须自己实现，在其中更新秤砣的位置）。
7. 调用`sprites.draw()`，以屏幕表面作为参数，在当前位置绘制秤砣。（每次调用`update()`时位置都会变化。）
8. 调用`pygame.display.update()`，以`sprites.draw()`返回的矩形列表作为参数，只在需要的位置更新显示。（如果不在乎性能，可以使用`pygame.display.flip()`更新整个显示。）
9. 重复第5~8步。

实现这些步骤的代码见代码清单29-1。

[代码清单29-1 简单的“掉落秤砣”动画](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch29/weights.py)

注：秤砣的实际掉落速度不仅取决于`speed`，还与帧率有关。直接运行书中代码可能会看到秤砣飞快地掉落，因为帧率过高。可以使用`pygame.time.Clock`限制帧率。

可以使用下面的命令运行这个程序：

```shell
$ python weights.py
```

执行这个命令时，需要确保weights.py和weight.png都在当前目录中。下图展示了程序的屏幕截图。

![简单的掉落秤砣动画](/assets/images/python-note-ch29-project-10-do-it-yourself-arcade-game/简单的掉落秤砣动画.png)

这些代码大都是不言自明的，但有几点需要解释一下：
* 所有的`Sprite`对象都有`image`和`rect`属性，前者是一个`Surface`对象（图像），后者是一个矩形对象（使用`self.image.get_rect()`初始化）。绘制`Sprite`对象时将用到这两个属性。通过修改`self.rect`可以移动`Sprite`对象。
* `Surface`对象有一个`convert()`方法，可用于创建使用不同颜色模型的副本。
* 颜色是使用RGB三元组（`(red, green, blue)`，每个值的范围都是0~255）指定的，因此元组`(255, 255, 255)`表示白色。
* 要修改矩形，可以给矩形的属性（`top`、`bottom`、`left`、`right`等）赋值，或者调用`inflate()`、`move()`等方法（详见文档 <https://www.pygame.org/docs/ref/rect.html> ）。

## 29.5 再次实现
在本节中，不再演示如何逐步设计和实现游戏，而在源代码中添加了大量的注释和文档字符串，如代码清单29-2~29-4所示。这里简要地解释其中的要点（以及一些不那么直观的细节）：
* 游戏由5个文件组成：config.py包含各种配置变量；objects.py包含游戏对象的实现；squish.py包含主类`Game`和各种游戏状态类；weight.png和banana.png是游戏使用的两个图像。
* 矩形方法`clamp()`确保一个矩形位于另一个矩形内，必要时移动矩形。这用于避免香蕉移到屏幕外。
* 矩形方法`inflate()`调整矩形的尺寸（水平和垂直方向的像素数量）。这用于收缩香蕉的边界，从而在判定碰撞（“压扁”）前允许香蕉和秤砣有一定的重叠。
* 游戏本身由一个游戏对象和各种游戏状态组成。游戏对象在一个时刻只有一种状态，而状态负责处理事件并将自己显示在屏幕上。状态还能让游戏切换到另一种状态（例如，`Level`状态可以让游戏切换到`GameOver`状态）。

[代码清单29-2 Squash游戏配置文件](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch29/config.py)

[代码清单29-3 Squash游戏对象](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch29/objects.py)

[代码清单29-4 游戏主模块](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch29/squish.py)

执行squish.py文件运行游戏：

```shell
$ python squish.py
```

下面是一些游戏截图。

![Squish游戏开始画面](/assets/images/python-note-ch29-project-10-do-it-yourself-arcade-game/Squish游戏开始画面.png)

![即将被压扁的香蕉](/assets/images/python-note-ch29-project-10-do-it-yourself-arcade-game/即将被压扁的香蕉.png)

![过关画面](/assets/images/python-note-ch29-project-10-do-it-yourself-arcade-game/过关画面.png)

![游戏结束画面](/assets/images/python-note-ch29-project-10-do-it-yourself-arcade-game/游戏结束画面.png)

## 29.6 进一步探索
下面是一些改进这个游戏的点子：
* 添加声音。
* 记录得分。例如，每躲开一个秤砣得16分。可以使用文件或在线服务器保存最高分（分别使用第24章和第27章讨论的`asyncore`和XML-RPC）。
* 让更多的物体同时掉落。
* 将逻辑反过来：要求玩家尽可能接住而不是躲避掉落的物体。
* 让玩家拥有多条命。
* 创建游戏的独立可执行版（参见第18章）。

有关更精致（且娱乐性极高）的Pygame编程示例，参阅Pygame维护者Pete Shinners开发的游戏SolarWolf (<https://www.pygame.org/shredwheat/solarwolf/index.shtml>)。在Pygame网站(<https://www.pygame.org/tags/all>)上还能找到很多其他游戏。如果Pygame让你迷上了游戏开发，可以参阅网站 <https://www.gamedev.net/> 和 <https://gamedev.stackexchange.com/> 。
