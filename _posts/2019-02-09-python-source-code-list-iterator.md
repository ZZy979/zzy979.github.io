---
title: 【Python源码阅读】list迭代器
date: 2019-02-09 10:30 +0800
categories: [Python]
tags: [python, list, iterator]
---
Python源代码GitHub地址：<https://github.com/python/cpython>

在Python Shell中输入以下命令：

```python
>>> a = [1, 2, 3]
>>> it = iter(a)
>>> it.__class__
<class 'list_iterator'>
```

可以看到`list`的迭代器的类型为`list_iterator`。

## “`list_iterator`类”的定义
`list_iterator`实际上是`collections.abc`模块定义的一个别名：

```python
list_iterator = type(iter([]))
```

## `list.__iter__()`方法源代码
内置函数`iter(obj)`等价于`obj.__iter__()`，因此尝试寻找`list`类的`__iter__()`方法源代码。`list`类的源代码在CPython的[Objects/listobject.c](https://github.com/python/cpython/blob/v3.8.0/Objects/listobject.c)文件中：

```c
PyTypeObject PyList_Type = {
    PyVarObject_HEAD_INIT(&PyType_Type, 0)
    "list",
    sizeof(PyListObject),
    ...
    list_iter,                                  /* tp_iter */
    0,                                          /* tp_iternext */
    ...
};
```

其中`list_iter`为构造其迭代器的函数（对应`__iter__()`方法），其定义为：

```c
static PyObject *
list_iter(PyObject *seq)
{
    listiterobject *it;

    if (!PyList_Check(seq)) {
        PyErr_BadInternalCall();
        return NULL;
    }
    it = PyObject_GC_New(listiterobject, &PyListIter_Type);
    if (it == NULL)
        return NULL;
    it->it_index = 0;
    Py_INCREF(seq);
    it->it_seq = (PyListObject *)seq;
    _PyObject_GC_TRACK(it);
    return (PyObject *)it;
}
```

该函数返回了一个`listiterobject`类型的指针`it`。`listiterobject`类型的定义：

```c
typedef struct {
    PyObject_HEAD
    Py_ssize_t it_index;
    PyListObject *it_seq; /* Set to NULL when iterator is exhausted */
} listiterobject;
```

可以看到`listiterobject`保存了一个指向数据域的指针`it_seq`和一个索引值`it_index`（基地址+偏移量），使用`iter(lst)`或`lst.__iter__()`构造`list`的迭代器时直接将`list`对象的数据域指针`seq`赋给了迭代器的`it_seq`：`it->it_seq = (PyListObject *)seq;`

## `list_iterator`的底层类型及其`__next__()`方法
`list_iterator`对应的底层实际类型为`PyListIter_Type`（相当于`listiterobject`的包装类型）：

```c
PyTypeObject PyListIter_Type = {
    PyVarObject_HEAD_INIT(&PyType_Type, 0)
    "list_iterator",                            /* tp_name */
    sizeof(listiterobject),                     /* tp_basicsize */
    ...
    PyObject_SelfIter,                          /* tp_iter */
    (iternextfunc)listiter_next,                /* tp_iternext */
    listiter_methods,                           /* tp_methods */
    0,                                          /* tp_members */
};
```

`PyObject_SelfIter`和`listiter_next`两个函数即分别为迭代器的`__iter__()`和`__next__()`方法的底层函数。迭代器的`__iter__()`方法总是返回其自身，`__next__()`方法的底层函数`listiter_next`的定义：

```c
static PyObject *
listiter_next(listiterobject *it)
{
    PyListObject *seq;
    PyObject *item;

    assert(it != NULL);
    seq = it->it_seq;
    if (seq == NULL)
        return NULL;
    assert(PyList_Check(seq));

    if (it->it_index < PyList_GET_SIZE(seq)) {
        item = PyList_GET_ITEM(seq, it->it_index);
        ++it->it_index;
        Py_INCREF(item);
        return item;
    }

    it->it_seq = NULL;
    Py_DECREF(seq);
    return NULL;
}
```

可以很清楚地看到迭代时执行的操作：当迭代未结束时，根据索引值`it_index`（相当于偏移量）从数据域`it_seq`中取得下一个元素`item`，将索引值+1并返回取得的元素；当迭代结束时返回`NULL`，上层函数将抛出`StopIteration`异常。
