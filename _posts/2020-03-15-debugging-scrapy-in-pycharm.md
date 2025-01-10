---
title: 在PyCharm中调试Scrapy爬虫
date: 2020-03-15 11:46:24 +0800
categories: [Scrapy]
tags: [ide, scrapy, crawler]
---
通常运行Scrapy爬虫的方法是在工程目录下执行`scrapy crawl <spider>`命令，而不是直接运行Python脚本，因此无法直接命中断点。执行scrapy命令时实际上是执行了scrapy.cmdline模块，因此在PyCharm中添加一个运行该模块的配置即可。

1.点击左上角的"Add Configuration..."

![Add Configuration](/assets/images/debugging-scrapy-in-pycharm/Add Configuration.png)

2.添加一个Python运行配置

![添加Python运行配置](/assets/images/debugging-scrapy-in-pycharm/添加Python运行配置.png)

3.点击右边 "Script path" 后边的三角，选择 "Module name" ，表示要执行一个Python模块，然后输入scrapy.cmdline

![选择Module name](/assets/images/debugging-scrapy-in-pycharm/选择Module name.png)

4.然后输入参数crawl \<spider\>，这里爬虫名是quotes，下面的工作目录选择工程根目录

![输入运行参数](/assets/images/debugging-scrapy-in-pycharm/输入运行参数.png)

5.点击确定，然后按正常方式开始调试即可成功命中断点

![开始调试](/assets/images/debugging-scrapy-in-pycharm/开始调试.png)

![命中断点](/assets/images/debugging-scrapy-in-pycharm/命中断点.png)
