---
title: 【Python】for语句的本质——迭代器
date: 2019-02-09 09:19 +0800
categories: [Python]
tags: [python, for statement, iterator]
---
`for`语句将创建一个迭代器并使用`next()`函数得到下一个元素直到迭代结束（参考Python文档：[The for statement](https://docs.python.org/3/reference/compound_stmts.html#the-for-statement)），即：

```python
for x in s:
    f(x)
```

等价于

```python
it = iter(s)
while True:
    try:
        x = next(it)
        f(x)
    except StopIteration:
        break
```

其中`s`是可迭代对象（有`__iter__()`方法，返回其迭代器），具体类型有
* 内置类型：列表、元组、集合、字典、字符串
* 迭代器（`__iter__()`方法返回其自身，`__next__()`方法返回下一个元素）
* 生成器（有`yield`语句的函数或生成器表达式，`__iter__()`方法返回其自身，`__next__()`方法产生下一个元素）

巧妙运用：将自定义类的`__iter__()`方法定义为一个生成器，则对其进行迭代时可按自己指定的方式产生元素。例如（例子来源：[Word2vec Tutorial](https://rare-technologies.com/word2vec-tutorial/)）：

```python
import os

class MySentences:
    def __init__(self, dirname):
        self.dirname = dirname

    def __iter__(self):
        for fname in os.listdir(self.dirname):
            for line in open(os.path.join(self.dirname, fname)):
                yield line.split()

def iterate_sentences(sentences):
    for sentence in sentences:
        for word in sentence:
            print(word, end=' ')
```

```python
>>> sentences = MySentences('E:\\a')
>>> iterate_sentences(sentences)
```

将依次遍历每个文件的每一行，并拆分为单词列表。如果E:\\a目录下有两个文件1.txt和2.txt，其内容分别为

```
1 a
11 aa
111 aaa
```

和

```
2 b
22 bb
222 bbb
```

则输出`1 a 11 aa 111 aaa 2 b 22 bb 222 bbb`，而`list(sentences)`将返回`[['1', 'a'], ['11', 'aa'], ['111', 'aaa'], ['2', 'b'], ['22', 'bb'], ['222', 'bbb']]`。 

在`for sentence in sentences`的等价代码中，`it = iter(sentences)`将得到一个**生成器**（`MySentences`类的`__iter__()`方法），而`Generator`类是`Iterator`类的子类（参考Python文档：<https://docs.python.org/3/library/collections.abc.html#module-collections.abc>），因此生成器可以当作迭代器使用，不断使用`next()`函数得到下一个元素。
