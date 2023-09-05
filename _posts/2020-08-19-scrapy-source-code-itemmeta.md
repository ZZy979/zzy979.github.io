---
title: 【Scrapy源码阅读】ItemMeta类
date: 2020-08-19 18:20:43 +0800
categories: [Scrapy]
tags: [python, crawler, metaclass]
---
下面的代码定义了一个Item类：

```python
class MyItem(scrapy.Item):
    foo = scrapy.Field()
    bar = scrapy.Field()
```

按照Scrapy官方文档的说法，使用关键字参数创建Item对象，和字典一样使用下标访问和修改字段的值，此外还有一个`fields`属性用于访问字段本身（由于`Field`类就是`dict`，因此`foo = Field()`等价于`foo = {}`）：

```python
>>> item = MyItem(foo='abc', bar=123)
>>> item
{'bar': 123, 'foo': 'abc'}
>>> item['foo']
'abc'
>>> item.fields
{'bar': {}, 'foo': {}}
>>> item.fields['foo']
{}
```

但是，直接通过属性访问字段则会报错：

```python
>>> item.foo
Traceback (most recent call last):
  File "<input>", line 1, in <module>
  ...
AttributeError: Use item['foo'] to get field value
>>> 'foo' in dir(item)
False
>>> 'foo' in dir(MyItem)
False
```

那么问题来了：`MyItem`类明明定义了`foo`和`bar`两个属性，为什么通过实例访问`foo`这个属性时会报错，并且`dir(item)`甚至`dir(MyItem)`中根本没有这个属性？`foo`属性是何时被删除的？`fields`属性又是何时被添加的？

官方文档的"[Declaring fields](https://docs.scrapy.org/en/latest/topics/items.html#declaring-fields)"一节中提到了这一问题，但没有提供更多细节：

> It's important to note that the Field objects used to declare the item do not stay assigned as class attributes. Instead, they can be accessed through the Item.fields attribute.

只能从源代码入手寻找原因。能够对对象进行“改造”的地方，一种可能是构造函数，但`Item`类的构造函数（继承自`DictItem`类）所做的工作只有将关键字参数保存到`_values`属性：

```python
class DictItem(MutableMapping, BaseItem):

    fields = {}

    def __init__(self, *args, **kwargs):
        self._values = {}
        if args or kwargs:  # avoid creating dict for most common case
            for k, v in dict(*args, **kwargs).items():
                self[k] = v

    def __getitem__(self, key):
        return self._values[key]

    def __setitem__(self, key, value):
        if key in self.fields:
            self._values[key] = value
        else:
            raise KeyError("%s does not support field: %s" % (self.__class__.__name__, key))  
```

另一种可能是`__new__()`方法，但是`MyItem`类的`__new__()`方法只能操作该类的实例，不能修改类本身的属性。因此只有一种可能——元类。

[元类](https://docs.python.org/3/reference/datamodel.html#metaclasses)是类的类，一般类的元类都是`type`（例如`5`的类型是`int`，而`int`的类型是`type`），而Scrapy自定义了`Item`类的元类`ItemMeta`，即**ItemMeta类的实例是Item类（或其子类），Item类的实例是item对象**。

```python
class Item(DictItem, metaclass=ItemMeta):
    """
    Base class for scraped items.
    ...
    """
```

Python语言定义一个类的[\_\_new__()](https://docs.python.org/3/reference/datamodel.html#object.__new__)方法返回该类的实例，因此元类的`__new__()`方法返回类对象，即：
```

ItemMeta.__new__(ItemMeta, 'MyItem', ...)->MyItem
MyItem.__new__(MyItem, foo='abc', bar=123)->item
```

因此关键就在于`ItemMeta`类的`__new__()`方法，下面对该方法的源代码进行探究。该方法的代码如下：

![ItemMeta.__new__()](/assets/images/scrapy-source-code-itemmeta/ItemMeta.__new__.png)

该方法的四个参数分别为要创建的类的类型`mcs`（这里就是`ItemMeta`本身，因为要创建的实例是`MyItem`，其类型是`ItemMeta`）、名称`class_name`（例如`"Item"`, `"MyItem"`或其他`Item`的子类）、基类`bases`以及属性集合`attrs`（重点）。

该方法的大致逻辑是：首先使用旧属性集合`attrs`创建一个名为`x_MyItem`的临时类`_class`，该临时类应当是包含属性`foo`和`bar`的；之后遍历`_class`的所有属性，将`Field`类型的属性（即自定义的字段）保存在`fields`属性中；最后将`fields`放入新属性集合`new_attrs`，使用`new_attrs`创建真正的`MyItem`类并返回。

从以上过程中可以看出，创建真正的`MyItem`类时使用的属性集合`new_attrs`并不包含`foo`和`bar`，这就是自定义的字段只能通过`item.fields['foo']`访问而不能通过`item.foo`访问的原因。

下面通过调试验证以上猜想。在`ItemMeta`类的`__new__()`方法中设置断点，当程序执行到`MyItem`类的定义时就会调用该方法。

![调试ItemMeta.__new__()](/assets/images/scrapy-source-code-itemmeta/调试ItemMeta.__new__.png)

从调试窗口中可以看到，传入的参数`attrs`确实包含`'foo'`和`'bar'`这两项，由此创建出的临时类`_class`也确实包含`foo`和`bar`这两个属性。

综上，`Item`类及其子类在`ItemMeta`类的`__new__()`方法中被“改造”了，所有的字段属性被添加到`fields`属性中，而没有被包含在最终创建的类中，`fields`属性在`Item`类的超类`DictItem`类中定义：

```python
class DictItem(MutableMapping, BaseItem):
    fields = {}
    # ...
```

至此，已经搞清了Item类自定义字段的问题。
