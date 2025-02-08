---
title: 【Django】设置文件
date: 2021-10-18 18:10 +0800
categories: [Django]
tags: [django]
---
官方文档：
* <https://docs.djangoproject.com/en/stable/topics/settings/>
* <https://docs.djangoproject.com/en/stable/ref/settings/>

## 1.指定设置文件
Django创建项目时自动创建了一个设置文件\<project_name\>/settings.py。

执行`django-admin`或manage.py的任何命令时都要使用`DJANGO_SETTINGS_MODULE`环境变量或`--settings`选项指定使用的设置文件。例如：

Linux Shell：

```shell
export DJANGO_SETTINGS_MODULE=mysite.settings
python manage.py runserver
```

Windows CMD：

```shell
SET DJANGO_SETTINGS_MODULE=mysite.settings
python manage.py runserver
```

使用`--settings`选项：

```shell
python manage.py runserver --settings=mysite.settings
```

其中的名称应当是一个可导入的Python模块名称（`mysite.settings`即默认创建的设置文件mysite/settings.py）。

自动生成的manage.py已经设置了`DJANGO_SETTINGS_MODULE`环境变量的默认值：

```python
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'mysite.settings')
```

因此如果使用自动生成的设置文件则可以省略这一步。

## 2.在代码中使用设置
导入`django.conf.settings`对象，例如：

```python
from django.conf import settings

if settings.DEBUG:
    # Do something
```

注意：不是导入自定义的设置模块`mysite.settings`。

## 3.多个设置文件
如果需要在开发环境、测试环境、生产环境等场景下使用不同的设置文件，则可以创建多个设置文件并通过上述方式指定要使用的设置文件。设置文件的位置和文件名没有要求，只要是可导入的Python模块即可。

例如，在mysite/settings目录下有三个设置文件，common.py包含公共设置，dev.py包含开发环境设置，prod.py包含生产环境设置（注意修改`BASE_DIR`）：

```
mysite/
    manage.py
    mysite/
        __init__.py
        settings/
            common.py
            dev.py
            prod.py
        urls.py
        asgi.py
        wsgi.py
```

在dev.py和prod.py中可以导入common.py中的设置，也可以覆盖其中的设置。例如：

common.py

```python
ALLOWED_HOSTS = ['localhost']
DEBUG = False
```

dev.py

```python
from .common import *  # noqa

DEBUG = True
```

prod.py

```python
from .common import *  # noqa

ALLOWED_HOSTS = ['www.example.com']
```

要使用dev.py和prod.py两个设置文件，将`DJANGO_SETTINGS_MODULE`环境变量或`--settings`选项的值分别设置为`mysite.settings.dev`和`mysite.settings.prod`即可。
