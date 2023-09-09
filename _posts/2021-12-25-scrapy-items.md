---
title: 【Scrapy】Item
date: 2021-12-25 16:41:33 +0800
categories: [Scrapy]
tags: [python, scrapy, crawler]
---
Item用于存储从页面中提取出的结构化数据，相当于实体类

官方文档：<https://docs.scrapy.org/en/latest/topics/items.html>

Scrapy支持多种Item对象，包括Python字典、`scrapy.Item`类以及其他几种键值对对象

自定义Item：

```python
class MyItem(scrapy.Item):
    foo = scrapy.Field()
    bar = scrapy.Field()
```

使用`Field`类定义`Item`支持的字段（`Field`实际上仅仅是`dict`的别名），可以给`Field`指定任何类型的“元数据”，例如：

```python
foo = scrapy.Field(type=str, length=32)
```

通过`Item.fields`属性获取所有声明的字段

```python
>>> MyItem.fields
{'bar': {}, 'foo': {'type': <class 'str'>, 'length': 32}}
```

`Item`的使用方式与`dict`类似，可通过关键字参数创建，通过下标访问和给字段赋值

```python
>>> item = MyItem(foo='abc', bar=123)
>>> item['foo']
'abc'
>>> item.fields['foo']
{'type': <class 'str'>, 'length': 32}
>>> item['baz'] = 1
Traceback (most recent call last):
  File "<input>", line 1, in <module>
  ...
KeyError: 'MyItem does not support field: baz'
```

Scrapy 2.2提供了`itemadapter.ItemAdapter`类和`itemadapter.is_item()`函数用于为处理不同类型的Item对象提供统一的接口
