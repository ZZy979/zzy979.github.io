---
title: 【Scrapy源码阅读】response处理过程
date: 2020-03-15 22:59:17 +0800
categories: [Scrapy]
tags: [python, crawler]
---
以官方教程[QuotesSpider](https://docs.scrapy.org/en/latest/intro/tutorial.html)为例，结合源码分析一下Scrapy中response的处理过程。

下面是待爬取的网页，红框中的是目标HTML标签：

1.quote文字内容

![quote文字内容](/assets/images/scrapy-source-code-response-processing/quote文字内容.png)

2.下一页链接

![下一页链接](/assets/images/scrapy-source-code-response-processing/下一页链接.png)

QuotesSpider代码如下：

```python
import scrapy


class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = [
        'http://quotes.toscrape.com/page/1/',
    ]

    def parse(self, response):
        for quote in response.css('div.quote'):
            yield {
                'text': quote.css('span.text::text').get(),
                'author': quote.css('small.author::text').get(),
                'tags': quote.css('div.tags a.tag::text').getall()
            }

        next_page = response.css('li.next a::attr(href)').get()
        if next_page is not None:
            yield scrapy.Request(response.urljoin(next_page), callback=self.parse)
```

可以看出`parse`方法是一个[生成器](https://docs.python.org/3/glossary.html#term-generator)，其中有两个`yield`语句，一个产生包含quote内容的字典对象，这是我们要获取的数据；另一个产生包含下一页链接的`Request`对象，用于继续爬取。先运行一下这个爬虫，得到下面的输出结果：

![输出结果](/assets/images/scrapy-source-code-response-processing/输出结果.png)

可以看到`parse`方法产生的字典对象都通过日志输出了，而`Request`对象包含的后续链接也被正确爬取了。那么这两种类型的对象在哪里被处理？分别被如何处理？

首先在`parse`方法中设置断点，进入调试状态（具体方法见[在PyCharm中调试Scrapy爬虫]({% post_url 2020-03-15-debugging-scrapy-in-pycharm %})），如下图所示：

![开始调试](/assets/images/scrapy-source-code-response-processing/开始调试.png)

在左下角的调用栈中找到上一层调用，如下图所示：

![中间件调用1](/assets/images/scrapy-source-code-response-processing/中间件调用1.png)

可以看到`parse`方法（生成器）在一个叫作`_evaluate_iterable`的函数中作为参数iterable被迭代，而这个函数本身也是一个生成器。

继续寻找上一层调用栈，发现`_evaluate_iterable`函数（生成器）在`DepthMiddleware.process_spider_output`方法中被以[生成器表达式](https://docs.python.org/3/reference/expressions.html#generator-expressions)的形式迭代：

![中间件调用2](/assets/images/scrapy-source-code-response-processing/中间件调用2.png)

这两层调用就是一个内置中间件depth，可以看出所谓中间件其实就是对spider生成的对象做一个过滤，其中的`_filter`函数就是中间件自己定义的过滤规则，其他的中间件也都包含这一行相同的代码。

spider产生的对象通过中间件的过程对应官方文档[整体架构图](http://docs.scrapy.org/en/latest/topics/architecture.html)中的第7步。

在调用栈中继续向上寻找，越过其他的中间件，找到最顶层调用在`scrapy.utils.defer.parallel`函数的一个生成器表达式work中，如下图所示（这张图实际上是在`parse`方法的下一次循环时截的，否则调试窗口中看不到elem），将本次产生的elem对象应用于一个callable对象，从调试窗口中可以看到这个callable是`Scraper._process_spidermw_output`方法（从方法名中可以猜测这是处理经过所有中间件后得到的最终输出结果的方法）

![顶层调用](/assets/images/scrapy-source-code-response-processing/顶层调用.png)

打开菜单Navigate->Class...，输入Scraper，打开这个类的源代码

![查找Scraper类](/assets/images/scrapy-source-code-response-processing/查找Scraper类.png)

找到`_process_spidermw_output`方法，如下图所示，可以看到这个方法中果然有对output类型的判断，从调试窗口中可以看到参数output就是刚才的elem，即`parse`方法本次通过`yield`语句产生的对象

![_process_spidermw_output方法](/assets/images/scrapy-source-code-response-processing/process_spidermw_output方法.png)

很明显，这个方法对不同类型的对象做不同的处理，对应上面整体架构图中的第8步：

![整体架构图](https://docs.scrapy.org/en/latest/_images/scrapy_architecture_02.png)

* 如果output是`Request`对象，则将其交给引擎准备继续爬取，打开`ExecutionEngine`类的`crawl`方法可以看到引擎直接将其交给了调度器
* 如果output是`BaseItem`或字典类型，则将其交给[Item Pipeline](http://docs.scrapy.org/en/latest/topics/item-pipeline.html#topics-item-pipeline)处理，下面具体分析一下这一步的处理过程

回到`_process_spidermw_output`方法，由于本次的output是字典类型，因此将其应用于`self.itemproc.process_item`方法（其中`self.itemproc`是一个`ItemPipelineManager`类型的对象，也是一种中间件）。如果项目自定义了Item Pipeline（例如将结果保存到数据库等），其`process_item`方法将会在此处被调用。

`self.itemproc.process_item`方法返回了一个包含数据信息（在result属性）的`Deferred`类型的对象dfd（完整类名是`twisted.internet.defer.Deferred`，属于Scrapy的底层依赖库Twisted）

![_process_spidermw_output方法2](/assets/images/scrapy-source-code-response-processing/process_spidermw_output方法2.png)

接下来以`self._itemproc_finished`为"callback"参数调用了`dfd.addBoth`方法，继续跟踪源代码，经过`addBoth->addCallbacks->_runCallbacks`几次调用，在下图的位置找到了对callback这个参数的调用：

![_runCallbacks方法](/assets/images/scrapy-source-code-response-processing/runCallbacks方法.png)

这个参数就是刚才传入的`Scraper._itemproc_finished`，所以调用又回到`Scraper`类，终于在下图的位置找到了日志输出的来源：

![_itemproc_finished方法](/assets/images/scrapy-source-code-response-processing/itemproc_finished方法.png)

logkws是日志输出的格式，其中的item参数就是`parse`方法产生的字典对象，通过调试单步执行这一行，可以看到打印这个字典对象的日志输出就是这一行代码：

![_itemproc_finished方法2](/assets/images/scrapy-source-code-response-processing/itemproc_finished方法2.png)

![日志输出](/assets/images/scrapy-source-code-response-processing/日志输出.png)

至此已经搞清楚了Scrapy中spider的`parse`方法生成结果的处理过程，即整体架构图中的第7~8步。然而这只是Scrapy源码的一小部分，其他部分有机会再继续研究。
