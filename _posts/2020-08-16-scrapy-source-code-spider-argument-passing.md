---
title: 【Scrapy源码阅读】Spider参数传递
date: 2020-08-16 18:46:44 +0800
categories: [Scrapy]
tags: [python, scrapy, crawler]
---
官方文档[Spider参数](https://docs.scrapy.org/en/latest/topics/spiders.html#spider-arguments)中提到，可以使用`scrapy crawl`命令的`-a`选项向Spider传递参数：

`scrapy crawl myspider -a arg1=value1 -a arg2=value2`

这些参数会被传递到自定义的`MySpider`类的构造函数，并且超类`Spider`的构造函数会将其拷贝到属性中：

```python
import scrapy

class MySpider(scrapy.Spider):
    name = 'myspider'

    def __init__(self, *args, **kwargs):
        # kwargs['arg1'] == 'value1'
        # kwargs['arg2'] == 'value2'
        super().__init__(*args, **kwargs)
        # self.arg1 == 'value1'
        # self.arg2 == 'value2'
```

那么这些命令行参数是如何被解析，并最终设置为Spider的属性的？下面通过源码分析这一过程。

## 1.解析命令行参数
从Scrapy的命令行模块`scrapy.cmdline`入手，通过命令行输入的命令由该模块中的`execute()`函数执行：

![execute函数](/assets/images/scrapy-source-code-spider-argument-passing/execute函数.png)

这里有两个关键的对象：`parser`是Python内置模块`optparse`中的`OptionParser`类的对象，用于解析命令行参数；`cmd`是命令类的对象（`crawl`命令对应`scrapy.commands.crawl.Command`类）

该函数的几个关键步骤：
* 调用`cmd.add_options(parser)`方法为解析器添加可识别的选项
* 解析命令行参数并将结果保存在`opts`和`args`两个变量中（`opts`包含选项参数，`args`包含位置参数）
* 调用`cmd.process_options(args, opts)`方法处理解析结果
* 将`cmd.crawler_process`设置为一个新的`CrawlerProcess`对象
* 以这两个变量为参数调用命令对象的`run()`方法

(1)添加选项

`-a`选项在`scrapy.commands.BaseRunSpiderCommand`类的`add_option()`方法中被添加：

![添加选项](/assets/images/scrapy-source-code-spider-argument-passing/添加选项.png)

其中`action="append"`表示该选项可重复，参数值将被存储在一个列表中，`dest="spargs"`表示该选项在解析结果中的属性名称为`spargs`

例如，待解析的命令行参数为`-a arg1=value1 -a arg2=value2`，解析结果为`opts`，则`opts.spargs`是长度为2的列表`['arg1=value1', 'arg2=value2']`

(2)处理解析结果

`BaseRunSpiderCommand`的`process_options()`方法解析了`opts.spargs`并将其转换为字典

![处理解析结果](/assets/images/scrapy-source-code-spider-argument-passing/处理解析结果.png)

![转换参数](/assets/images/scrapy-source-code-spider-argument-passing/转换参数.png)

因此`['arg1=value1', 'arg2=value2']`将变为`{'arg1': 'value1', 'arg2': 'value2'}`

至此，解析命令行参数已完成，下面分析`cmd.run()`如何使用这些解析结果。

## 2.运行命令
查看`scrapy.commands.crawl.Command`的`run()`方法的代码：

![run方法](/assets/images/scrapy-source-code-spider-argument-passing/run方法.png)

参数`opts`为之前的解析结果，`run()`方法以关键字参数的形式将`opts.spargs`传入`scrapy.crawler.CrawlerProcess`类的`crawl()`方法，该方法继承自`CrawlerRunner.crawl()`，继续跟踪该方法的调用过程

![CrawlerRunner.crawl()](/assets/images/scrapy-source-code-spider-argument-passing/CrawlerRunner.crawl.png)

![Crawler.crawl()](/assets/images/scrapy-source-code-spider-argument-passing/Crawler.crawl.png)

经过`CrawlerRunner.crawl()->CrawlerRunner._crawl()->Crawler.crawl()->Crawler()._create_spider()`几次调用后，`kwargs`（即之前的`opts.spargs`）最终被传递到`Spider.from_crawler()`方法，上图中最后一行的`spidercls`就是自定义的`MySpider`类

## 3.创建Spider对象
查看`Spider.from_crawler()`方法的代码：

![Spider.from_crawler()](/assets/images/scrapy-source-code-spider-argument-passing/Spider.from_crawler.png)

可以看到，`kwargs`被传入`cls`（即自定义的`MySpider类`）的构造函数，如果`MySpider`类没有定义构造函数则继承`Spider`类的构造函数

查看`Spider`类的代码，发现其构造函数中的下面这行代码将`kwargs`中的键值对转换为自身的属性：

![Spider构造函数](/assets/images/scrapy-source-code-spider-argument-passing/Spider构造函数.png)

至此Spider参数的传递过程已经分析清楚。
