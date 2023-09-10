---
title: 【Scrapy】Spider
date: 2020-08-16 23:48:42 +0800
categories: [Scrapy]
tags: [python, scrapy, crawler]
---
Spider是用于定义如何从指定的网站爬取信息的类，功能包括定义初始请求、解析页面并提取数据以及跟踪后续链接

官方文档：<https://docs.scrapy.org/en/latest/topics/spiders.html>

## 典型的爬取循环
1. 通过start_urls属性或start_requests()方法定义初始请求URL及其回调函数
2. 在回调函数中使用选择器从响应页面中提取数据和新的URL，并通过yield语句分别返回Item对象和Request对象
3. 回调函数返回的Item对象将被送往Item Pipeline进行处理（例如保存到数据库），Request对象将被送往调度器从而实现跟踪链接

## Spider类
所有爬虫必须继承该类，常用的方法和属性如下：
* `name`：爬虫的名字，在一个项目中必须是唯一的
* `start_urls`：字符串列表，初始请求URL
* `logger`：使用爬虫的名字创建的Python logger对象，用于日志输出
* `settings`：用于访问设置（是一个`scrapy.settings.Settings`对象，可当作字典使用）
* `start_requests()`：返回初始请求的可迭代对象（可以是`Request`对象的列表或生成器）；如果未覆盖该方法，默认使用`start_urls`中的每个url和`self.parse`作为回调函数构造`Request`对象
* `parse(response)`：用于处理响应的默认回调函数，参数`response`是一个`scrapy.http.Response`类型的对象，该方法解析响应包含的页面内容，从中提取数据和新的URL，返回Item对象和/或`Request`对象的可迭代对象（使用`yield`语句）
* `log(message, level)`：日志输出，等价于`self.logger.log(message, level)`

## Spider参数
在`scrapy crawl`命令中可通过`-a NAME=VALUE`选项指定Spider参数

官方文档：<https://docs.scrapy.org/en/latest/topics/spiders.html#spider-arguments>
