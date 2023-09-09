---
title: 【Scrapy】命令行工具
date: 2021-12-25 16:11:36 +0800
categories: [Scrapy]
tags: [python, scrapy, crawler]
---
Scrapy提供了一个命令行工具scrapy，位于{Python安装目录}\Scripts\scrapy.exe，对应的模块：`scrapy.cmdline`

官方文档：<https://docs.scrapy.org/en/latest/topics/commands.html>

无参数运行该命令将打印帮助信息：

```
D:\PyCharm\projects>scrapy
Scrapy 2.3.0 - no active project

Usage:
  scrapy <command> [options] [args]

Available commands:
...
```

如果当前目录在一个Scrapy项目中则会打印项目名称，并且可以使用项目相关的命令：

```
D:\PyCharm\projects\myproject>scrapy
Scrapy 2.3.0 - project: myproject

Usage:
  scrapy <command> [options] [args]

Available commands:
...
```

使用`scrapy <command> -h`可查看某个命令的具体帮助

## 常用命令
### 全局命令
#### startproject：创建项目
```shell
scrapy startproject <project_name> [project_dir]
```

如果未指定项目路径则与项目名称相同

#### genspider：创建爬虫
```shell
scrapy genspider [-t <template>] <name> <domain>
```

在当前目录中或当前项目的spiders目录中创建一个新的爬虫，`<name>`参数用于设置爬虫的`name`属性，`<domain>`参数用于生成爬虫的`allowed_domains`和`start_urls`属性

可用的模板在{Python安装目录}\scrapy\templates\spiders目录下，默认为basic

#### shell：打开Scrapy shell
```shell
scrapy shell [url]
```

使用给定的URL发送一个请求，并进入交互式控制台

#### runspider：运行单个爬虫
```shell
scrapy runspider <spider_file>
```

### 仅在项目中使用的命令
#### crawl：运行爬虫
```shell
scrapy crawl [options] <spider>
```

运行指定名称的爬虫，可通过`-a NAME=VALUE`选项指定[Spider参数](https://docs.scrapy.org/en/latest/topics/spiders.html#spider-arguments)

#### list：列出可用爬虫
```shell
scrapy list
```
