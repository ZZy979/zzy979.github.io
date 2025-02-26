---
title: 【Django】静态文件
date: 2020-04-02 17:45 +0800
categories: [Django]
tags: [django]
render_with_liquid: false
---
静态文件包括图片、JavaScript和CSS。

官方文档：
* <https://docs.djangoproject.com/en/stable/howto/static-files/>
* <https://docs.djangoproject.com/en/stable/ref/contrib/staticfiles/>

静态文件查找目录：每个应用目录下的static目录（需手动创建），其中再创建一个和应用名相同的目录，例如mysite/polls/static/polls/style.css

另外可使用`STATICFILES_DIRS`设置指定额外的查找目录。

## 1.配置静态文件
如果`INSTALLED_APPS`中包含了`django.contrib.staticfiles`，则在`Debug==True`时Django会自动查找静态文件，而`Debug==False`时访问静态文件会报404，此时要手动配置。

（1）确认`INSTALLED_APPS`包含`django.contrib.staticfiles`。

（2）在settings.py中添加

```python
STATIC_URL = 'static/'
STATIC_ROOT = '/path/to/static'
```

分别指定静态文件URL前缀和`collectstatic`命令存放静态文件的位置。

（3）在整个项目的URLconf中添加提供静态文件的视图

```python
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    # ... the rest of your URLconf goes here ...
] + static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)
```

（4）部署代码后运行`python manage.py collectstatic`命令，Django将把所有用到的静态文件拷贝到`STATIC_ROOT`指定的目录下。

完成以上步骤后即可正常访问静态文件。

## 2.引用静态文件
在HTML模板中的引用方式：

```html
{% load static %}
<link rel="stylesheet" type="text/css" href="{% static 'polls/style.css' %}">
```

admin应用使用的静态文件：[django/contrib/admin/static](https://github.com/django/django/tree/main/django/contrib/admin/static)。

一个很好的HTML背景图片网站：<https://www.toptal.com/designers/subtlepatterns/>
