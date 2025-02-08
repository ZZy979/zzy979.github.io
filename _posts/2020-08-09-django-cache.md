---
title: 【Django】缓存
date: 2020-08-09 09:50 +0800
categories: [Django]
tags: [django]
---
缓存可以理解为一个URL到页面的映射。如果用户请求的URL已在缓存中则直接返回结果页面；否则生成页面，加入缓存并返回。

官方文档：<https://docs.djangoproject.com/en/stable/topics/cache/>

## 1.配置
设置文件中的`CACHES`。

## 2.缓存类型
### 2.1 Memcached
基于内存的缓存（第三方库[Memcached](https://memcached.org/)）。

### 2.2 数据库缓存

```python
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.db.DatabaseCache',
        'LOCATION': 'my_cache_table',
    }
}
```

创建缓存表：

```shell
python manage.py createcachetable
```

### 2.3 文件缓存

```python
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.filebased.FileBasedCache',
        'LOCATION': '/path/to/django_cache',
    }
}
```

### 2.4 本地内存缓存

```python
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.locmem.LocMemCache',
        'LOCATION': 'unique-snowflake',
    }
}
```

默认选项，其中`LOCATION`用于区分多个本地内存缓存，如果只有一个则可省略。

## 3.缓存参数
* `TIMEOUT`：默认过期时间，单位为秒，默认为300，设置为`None`表示永不过期，设置为0表示立即过期（不缓存）
* `OPTIONS`：传递给缓存后端的选项
  * `MAX_ENTRIES`：最大条目数量，默认为300
  * `CULL_FREQUENCY`：达到最大条目数量时清理几分之一的最近最少使用条目，默认为3

例如：使用文件缓存后端，默认超时时间为60秒，最大条目为1000条

```python
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.filebased.FileBasedCache',
        'LOCATION': '/var/tmp/django_cache',
        'TIMEOUT': 60,
        'OPTIONS': {
            'MAX_ENTRIES': 1000
        }
    }
}
```

## 4.使用缓存
### 4.1 视图缓存
使用装饰器`@cache_page`装饰视图函数，这将使用URL作为键，视图返回的响应作为值。

```python
from django.views.decorators.cache import cache_page

@cache_page(60 * 15)
def my_view(request):
    ...
```

### 4.2 直接使用缓存API
访问缓存：

```python
from django.core.cache import caches
cache = caches['default']
```

或直接

```python
from django.core.cache import cache
```

基本用法：
* `cache.set(key, value, timeout=DEFAULT_TIMEOUT)`
* `cache.get(key, default=None)`
* `cache.has_key(key)`
* `key in cache`
* `cache.delete(key)`

其中`key`为字符串，`value`为Python的picklable对象，`timeout`单位为秒（将覆盖设置文件中的默认超时时间）。

完整方法可查看`django.core.cache.backends.base.BaseCache`及其子类。
