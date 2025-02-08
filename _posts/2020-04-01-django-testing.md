---
title: 【Django】测试
date: 2020-04-01 23:59 +0800
categories: [Django]
tags: [django, unit test]
---
官方文档：<https://docs.djangoproject.com/en/stable/topics/testing/>

## 1.编写测试
在应用目录下的tests.py中编写测试。

Django的测试框架基于Python的`unittest`模块。要创建测试用例，继承`django.test.TestCase`类并在添加一个或多个以`test`开头的测试方法（以`test`开头是`unittest`模块的要求，而不是Django的）。

例如：

```python
from django.test import TestCase
from django.utils import timezone

from .models import Question

class QuestionModelTests(TestCase):

    def test_was_published_recently_with_future_question(self):
        """was_published_recently() returns False for questions whose pub_date
        is in the future.
        """
        time = timezone.now() + datetime.timedelta(days=30)
        future_question = Question(pub_date=time)
        self.assertFalse(future_question.was_published_recently())
```

## 2.运行测试

```shell
python manage.py test app_name
```

运行测试时将创建临时数据库，测试结束后删除，对每个测试方法重置临时数据库（通过事务回滚）。

测试数据库的默认名字为`"test_"`+默认数据库名。如果想自定义测试数据库的名字，则在设置文件的`DATABASES`中增加`"TEST"`设置：

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'db_name',
        'USER': 'username',
        'PASSWORD': 'password',
        'HOST': '192.168.0.1',
        'PORT': '3306',
        'TEST': {
            'NAME': 'test_db_name',
        },
    }
}
```

如果不想或（因数据库用户权限等原因）不能每次运行测试时创建和删除测试数据库，则在运行测试时增加`--keepdb`选项：

```shell
python manage.py test --keepdb app_name
```

使用`self.assert*`方法做断言，包括继承自`unittest.TestCase`类的（完整列表见[Python文档](https://docs.python.org/3/library/unittest.html#assert-methods)）和`django.test.TestCase`增加的：<https://docs.djangoproject.com/en/stable/topics/testing/tools/#assertions> 。

## 3.测试视图
为了模拟用户通过浏览器点击链接（发送请求）并判断是否返回预期的响应，Django提供了`django.test.Client`类。

`django.test.TestCase`类已经提供了一个`Client`实例`self.client`，可直接使用：

```python
class QuestionIndexViewTests(TestCase):
    def test_no_questions(self):
        """If no questions exist, an appropriate message is displayed."""
        response = self.client.get(reverse('polls:index'))
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, "No polls are available.")
        self.assertQuerysetEqual(response.context['latest_question_list'], [])
```

其中`assertContains()`和`assertQuerysetEqual()`是`django.test.TestCase`类增加的方法。

## 4.建议
* 给软件增加的每一个功能都应伴随一个测试，可以先写测试后编码，也可以先编码后测试。
* 不必担心测试代码会越来越多。
* 当软件功能变化时现有的测试可能会失败，此时需要更新测试。
* 随着开发的进行，有些测试可能变得冗余，但这也没有问题——在测试中冗余是好事。
* 良好的组织测试的规则：
  * 为每个模型和视图编写一个测试类
  * 为每一组想要测试的条件编写一个测试方法（测试用例）
  * 测试方法名描述其功能

## 5.初始测试数据
可以使用`TestCase.setUpTestData()`：<https://docs.djangoproject.com/en/stable/topics/testing/tools/#django.test.TestCase.setUpTestData>

也可以使用fixture：<https://docs.djangoproject.com/en/stable/topics/testing/tools/#fixture-loading>
