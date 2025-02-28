---
title: 【Django】自定义django-admin命令
date: 2025-02-28 21:20:38 +0800
categories: [Django]
tags: [django]
---
应用可以注册自定义的manage.py命令。

官方文档：<https://docs.djangoproject.com/en/stable/howto/custom-management-commands/>

注：`django-admin`、`python manage.py`和`python -m django`这三个命令是等价的，都是执行`django.core.management.execute_from_command_line()`。

## 1.创建自定义命令
假设要为[教程](https://docs.djangoproject.com/en/stable/intro/tutorial01/)中的polls应用创建一个自定义命令`closepoll`。为此，在应用目录下创建一个management/commands目录。Django将为该目录中名字不以下划线开头的每个模块注册一个manage.py命令。例如：

```
polls/
    __init__.py
    management/
        __init__.py
        commands/
            __init__.py
            closepoll.py
    models.py
    tests.py
    views.py
    ...
```

只要项目的`INSTALLED_APPS`包含polls应用，就可以使用`closepoll`命令。

closepoll.py必须定义一个`Command`类并继承`BaseCommand`或其子类。

```python
from django.core.management.base import BaseCommand, CommandError
from polls.models import Question as Poll


class Command(BaseCommand):
    help = "Closes the specified poll for voting"

    def add_arguments(self, parser):
        parser.add_argument("poll_ids", nargs="+", type=int)

    def handle(self, *args, **options):
        for poll_id in options["poll_ids"]:
            try:
                poll = Poll.objects.get(pk=poll_id)
            except Poll.DoesNotExist:
                raise CommandError('Poll "%s" does not exist' % poll_id)

            poll.opened = False
            poll.save()

            self.stdout.write(
                self.style.SUCCESS('Successfully closed poll "%s"' % poll_id)
            )
```

注意：命令的输出应该写入`self.stdout`和`self.stderr`，以便于测试。

这个命令可以通过以下方式调用：

```shell
python manage.py closepoll poll_ids...
```

`handle()`方法接受一个或多个`poll_ids`，并将每个`poll.opened`设置为`False`。

## 2.接受可选参数
可以在`add_arguments()`方法中添加自定义选项（命令行参数）。例如，添加一个`--delete`选项来删除（而不是关闭）给定的投票：

```python
class Command(BaseCommand):
    def add_arguments(self, parser):
        # Positional arguments
        parser.add_argument("poll_ids", nargs="+", type=int)

        # Named (optional) arguments
        parser.add_argument(
            "--delete",
            action="store_true",
            help="Delete poll instead of closing it",
        )

    def handle(self, *args, **options):
        # ...
        if options["delete"]:
            poll.delete()
        # ...
```

关于`add_argument()`的用法详见Python的[argparse](https://docs.python.org/3/library/argparse.html)模块。

## 3.测试
manage.py命令可以用`call_command()`函数来测试，输出可以重定向到`StringIO`中。

```python
from io import StringIO
from django.core.management import call_command
from django.test import TestCase

class ClosepollTest(TestCase):
    def test_command_output(self):
        out = StringIO()
        call_command("closepoll", poll_ids=[1], stdout=out)
        self.assertIn('Successfully closed poll "1"', out.getvalue())
```
