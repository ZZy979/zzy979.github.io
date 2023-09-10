---
title: 【Scrapy】Item Pipeline
date: 2020-08-17 01:00:09 +0800
categories: [Scrapy]
tags: [python, scrapy, crawler]
---
项目管道(Item Pipeline)用于处理Spider返回的Item对象，如果定义了多个项目管道，则按优先级顺序执行

官方文档：<https://docs.scrapy.org/en/latest/topics/item-pipeline.html>

项目管道就是实现了`process_item()`方法的Python类，用于处理Spider返回的Item对象

注意：Scrapy并没有提供所谓的`ItemPipeline`基类，自定义的项目管道类只要有`process_item()`方法即可，这是利用了Python的协议(protocol)或鸭子类型([duck-typing](https://docs.python.org/3/glossary.html#term-duck-typing))的特性，即只关心一个类是否有所需的属性或方法，而不是（像Java那样）通过基类或接口来限制对象的类型

## 自定义项目管道
项目管道类必须实现以下方法：

```python
process_item(self, item, spider)
```

对接收到的Item对象执行某些操作（例如保存到数据库），并决定该Item对象继续被后续的项目管道处理，还是不再继续处理

`item`是要处理的Item对象（可能是字典、`Item`类或其他），`spider`是对应的爬虫对象

该方法要么返回一个Item对象，给其他项目管道继续处理；要么产生一个`DropItem`异常，表示不再继续处理

以下方法是可选的：

```python
open_spider(self, spider)
```

当`spider`开始时该方法被调用

```python
close_spider(self, spider)
```

当`spider`关闭时该方法被调用

```python
from_crawler(cls, crawler)
```

这是一个类方法，用于从一个`Crawler`对象创建项目管道实例

## 项目管道的典型用途
* 清理HTML数据
* 验证爬取到的数据（是否包含特定的字段）
* 去重
* 将爬取到的数据保存到数据库

定义项目管道后要在设置文件中激活：

```python
ITEM_PIPELINES = {
    'myproject.pipelines.MyPipeline1': 300,
    'myproject.pipelines.MyPipeline2': 800,
}
```

数字为优先级，范围0~1000，数字越小优先级越高

## 示例
### 验证字段

```python
from itemadapter import ItemAdapter
from scrapy.exceptions import DropItem

class PricePipeline:
    vat_factor = 1.15

    def process_item(self, item, spider):
        adapter = ItemAdapter(item)
        if adapter.get('price'):
            if adapter.get('price_excludes_vat'):
                adapter['price'] *= self.vat_factor
            return item
        else:
            raise DropItem('Missing price in {}'.format(item))
```

### 写入JSON文件

```python
import json

from itemadapter import ItemAdapter

class JsonWriterPipeline:

    def open_spider(self, spider):
        self.file = open('items.jsonl', 'w')

    def close_spider(self, spider):
        self.file.close()

    def process_item(self, item, spider):
        line = json.dumps(ItemAdapter(item).asdict()) + '\n'
        self.file.write(line)
        return item
```

### 保存到MongoDB

```python
import pymongo
from itemadapter import ItemAdapter

class MongoPipeline:
    collection_name = 'scrapy_items'

    def __init__(self, mongo_uri, mongo_db):
        self.mongo_uri = mongo_uri
        self.mongo_db = mongo_db

    @classmethod
    def from_crawler(cls, crawler):
        return cls(
            mongo_uri=crawler.settings.get('MONGO_URI'),
            mongo_db=crawler.settings.get('MONGO_DATABASE', 'items')
        )

    def open_spider(self, spider):
        self.client = pymongo.MongoClient(self.mongo_uri)
        self.db = self.client[self.mongo_db]

    def close_spider(self, spider):
        self.client.close()

    def process_item(self, item, spider):
        self.db[self.collection_name].insert_one(ItemAdapter(item).asdict())
        return item
```

### 去重

```python
from itemadapter import ItemAdapter
from scrapy.exceptions import DropItem

class DuplicatesPipeline:

    def __init__(self):
        self.ids_seen = set()

    def process_item(self, item, spider):
        adapter = ItemAdapter(item)
        if adapter['id'] in self.ids_seen:
            raise DropItem('Duplicate item found: {}'.format(item))
        else:
            self.ids_seen.add(adapter['id'])
            return item
```
