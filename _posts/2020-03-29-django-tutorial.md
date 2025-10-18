---
title: Django入门教程
date: 2020-03-29 20:22:09 +0800
categories: [Django]
tags: [django, hello world, model, view, template]
render_with_liquid: false
---
## 1.简介
Django是一个免费、开源的Python web框架，解决了web开发的许多麻烦，使开发者可以专注于编写应用而无需重复造轮子。

* 网站：<https://www.djangoproject.com/>
* 官方文档：<https://docs.djangoproject.com/>
* 源代码：<https://github.com/django/django>

## 2.安装
使用pip安装最新版本的Django：

```bash
pip install django
```

也可以手动指定版本：

```bash
pip install django==3.2
```

Django支持的Python版本见[What Python version can I use with Django?](https://docs.djangoproject.com/en/stable/faq/install/#faq-python-version-support)

安装完成后，可以在命令行使用`django-admin`辅助工具：

```bash
$ django-admin --version
3.2.11
```

## 3.入门教程
官方教程介绍了如何创建一个简单的投票应用：[Writing your first Django app](https://docs.djangoproject.com/en/stable/intro/tutorial01/)

该应用由两部分组成：
* 一个公共站点，用户可以查看问题并投票。
* 一个管理站点，管理员可以添加、修改和删除问题。

完整代码：<https://github.com/ZZy979/DjangoTutorial>

### 3.1 第1部分：创建项目和应用
#### 3.1.1 创建项目
在命令行中执行以下命令：

```bash
django-admin startproject mysite
```

该命令创建了一个名为mysite的**项目**(project)，目录结构如下：

```
mysite/
    manage.py
    mysite/
        __init__.py
        settings.py
        urls.py
        asgi.py
        wsgi.py
```

* 外层mysite目录：项目根目录，可以放在任何位置，也可以任意重命名。
* manage.py：Django命令行工具，详见[django-admin and manage.py](https://docs.djangoproject.com/en/stable/ref/django-admin/)。
* 内层mysite目录：项目的Python包，用于导入模块（例如`mysite.urls`）。
* mysite/\_\_init\_\_.py：空文件，告诉Python该目录是一个Python包。
* mysite/settings.py：设置/配置，详见[Django settings](https://docs.djangoproject.com/en/stable/topics/settings/)和[The Settings Reference](https://docs.djangoproject.com/en/stable/ref/settings/)。
* mysite/urls.py：URL声明，相当于网站的“目录”，详见[URL dispatcher](https://docs.djangoproject.com/en/stable/topics/http/urls/)。
* mysite/asgi.py：ASGI web服务器的入口，详见[How to deploy with ASGI](https://docs.djangoproject.com/en/stable/howto/deployment/asgi/)。
* mysite/wsgi.py：WSGI web服务器的入口，详见[How to deploy with WSGI](https://docs.djangoproject.com/en/stable/howto/deployment/wsgi/)。

在项目根目录下执行以下命令：

```bash
python manage.py runserver
```

该命令启动了Django内置的web服务器（开发服务器）。在浏览器中访问 <http://127.0.0.1:8000/>，将会看到Congratulations页面：

![Congratulations](/assets/images/django-tutorial/Congratulations.png)

默认情况下，`runserver`命令在内部IP地址127.0.0.1、8000端口号启动web服务器。要改变端口号，可以通过命令行参数指定：

```bash
python manage.py runserver 8080
```

要监听所有公共IP，则执行

```bash
python manage.py runserver 0.0.0.0:8000
```

Django的开发服务器会自动重新加载Python代码，因此修改代码后无需重启服务器。

#### 3.1.2 创建投票应用
有了项目之后就可以开始编写应用了。**应用**(application/app)就是一个遵循特定约定的Python包。

注：Django项目和应用的区别：应用是一个具体的web应用，例如博客应用、投票应用等。项目是一个特定网站的配置和应用的集合。一个项目可以包含多个应用，一个应用也可以属于多个项目。

应用可以位于[Python path](https://docs.python.org/3/tutorial/modules.html#tut-searchpath)的任何路径。在本教程中，将投票应用创建在项目根目录下。在项目根目录下执行以下命令：

```bash
python manage.py startapp polls
```

该命令创建了一个名为polls的应用，目录结构如下：

```
polls/
    __init__.py
    admin.py
    apps.py
    migrations/
        __init__.py
    models.py
    tests.py
    urls.py             # 手动创建
    views.py
    templates/          # 手动创建
        polls/
            index.html
    static/             # 手动创建
        polls/
            images/
                background.png
            style.css
```

<!-- 由于添加了render_with_liquid: false，这里不能使用post_url标签，只能使用文章的绝对链接 -->

* admin.py：注册模型
* apps.py：包含应用配置类
* migrations：存放数据库变更记录（见[模型](/posts/django-models/) 1.2节）
* models.py：定义[模型](/posts/django-models/)
* tests.py：编写[测试](/posts/django-testing/)
* urls.py：定义URL映射(URLconf)
* views.py：定义[视图](/posts/django-views/)
* templates：存放[模板](/posts/django-templates/)
* static：存放[静态文件](/posts/django-static-files/)，包括图片、CSS和JavaScript

#### 3.1.3 编写第一个视图
打开文件polls/views.py，编写以下代码：

```python
from django.http import HttpResponse


def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
```

这里创建了一个名为`index`的**视图**(view)。为了调用该视图，需要一个映射到它的URL，即**URL配置**(URLconf)。

注：URL配置相当于Java Web中的Servlet mapping或Spring MVC中的Controller，用于将URL模式映射到视图。

为了创建URLconf，在polls目录下创建一个文件urls.py，内容如下：

```python
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```

之后在项目根目录下的urls.py中的`urlpatterns`增加一个`include()`：

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('polls/', include('polls.urls')),
]
```

其中，[include()](https://docs.djangoproject.com/en/stable/ref/urls/#django.urls.include)函数引用另一个URLConf，表示所有以 "polls/" 开头的URL都由`polls.urls`（即polls/urls.py）处理。

现在已经将`index()`视图关联到了URLconf。下面使用`runserver`命令启动服务器，并在浏览器中访问 <http://localhost:8000/polls/>，将会看到`index`视图返回的文本 "Hello, world. You're at the polls index." ，如下图所示：

![index视图](/assets/images/django-tutorial/index视图.png)

#### 3.1.4 path()函数
[path()](https://docs.djangoproject.com/en/stable/ref/urls/#django.urls.path)函数用于指定URL模式到视图的映射，接受4个参数：
* `route`：指定URL模式，不包括域名、GET和POST参数，可使用`<type:name>`定义路径参数，没有必要在结尾加 ".html"
    * 注意：若URL模式中最后有 "/" 而请求的URL最后没有 "/" 则会返回301（浏览器会自动在最后添加 "/" ）；若URL模式中最后没有 "/" 而请求的URL最后有 "/" 则会返回404
* `view`：匹配到URL模式时调用的视图，调用时以一个`HttpRequest`对象作为第一个参数，捕获的路径参数作为关键字参数
* `kwargs`：任意关键字参数，用于传递给视图函数
* `name`：URL模式的名称

### 3.2 第2部分：模型和admin应用
#### 3.2.1 数据库配置
settings.py中的`DATABASES`是数据库配置，默认的数据库引擎是SQLite。这里使用自动生成的默认配置即可：

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}
```

另外，将`TIME_ZONE`修改为合适的时区（例如 "Asia/Shanghai" ）。

#### 3.2.2 创建模型
接下来要定义模型。**模型**(model)是应用中的数据源，对应数据库表。每个模型由若干**字段**(field)组成，对应数据库字段。

在投票应用中需要创建两个模型：`Question`和`Choice`。`Question`有两个字段：问题文本和发布日期；`Choice`也有两个字段：选项文本和票数。每个`Choice`关联到一个`Question`。

在polls/models.py中创建模型类：

```python
import datetime

from django.db import models
from django.utils import timezone


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')

    def __str__(self):
        return self.question_text

    def was_published_recently(self):
        now = timezone.now()
        return self.pub_date >= now - datetime.timedelta(days=1)


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)

    def __str__(self):
        return self.choice_text
```

注：`__str__()`方法用于以人类可读的方式展示对象，模型也可以有自定义方法（例如`was_published_recently()`）。

每个模型由`django.db.models.Model`的一个子类表示。每个模型有若干类变量，表示数据库字段。

每个字段由`Field`类的子类实例表示，例如`CharField`表示文本字段，`DateTimeField`表示日期时间字段。数据库将使用字段名称（例如`question_text`和`pub_date`）作为列名。

可以使用`Field`的第一个位置参数指定一个人类可读的名字。例如，`Question.pub_date`字段指定了人类可读的名字`date published`。

`CharField`必须用`max_length`参数指定最大长度。另外，`Field`还有很多可选参数，例如`default`指定默认值。

注：如果没有指定主键(`PrimaryKey`)，则Django会自动生成一个名为`id`的自增主键字段。

`ForeignKey`用于定义外键。`Choice.question`字段告诉Django：`Choice`与`Question`是多对一的关系。Django支持所有常见的数据库关系：多对一、多对多和一对一。

#### 3.2.3 激活模型
根据模型的定义，Django能够使用`CREATE TABLE`语句创建数据库表，以及创建数据库查询API。

首先，需要先将polls应用安装到mysite项目。在settings.py的`INSTALLED_APPS`中添加`'polls.apps.PollsConfig'`或者`'polls'`：

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'polls',
]
```

之后执行以下命令：

```bash
python manage.py makemigrations polls
```

`makemigrations`命令根据模型定义的变化（新增、删除、修改字段等）创建**migration**，生成在polls/migrations目录下。

可以使用`sqlmigrate`命令查看migration对应的SQL语句：

```bash
$ python manage.py sqlmigrate polls 0001
BEGIN;
--
-- Create model Question
--
CREATE TABLE "polls_question" ("id" integer NOT NULL PRIMARY KEY AUTOINCREMENT, "question_text" varchar(200) NOT NULL, "pub_date" datetime NOT NULL);
...
COMMIT;
```

接下来执行`migrate`命令，将更改应用到数据库：

```bash
python manage.py migrate
```

Django将自动记录已经执行过的migration编号。

**修改模型的三个步骤**：
* 修改models.py中的模型定义
* 运行`python manage.py makemigrations`，为更改创建migration
* 运行`python manage.py migrate`，将更改应用到数据库

#### 3.2.4 查询API
现在可以在Python交互式窗口中尝试Django提供的数据库查询API。使用`shell`命令打开Python shell：

```bash
python manage.py shell
```

该命令与直接执行`python`命令的区别是manage.py会读取settings.py，从而能够访问配置的数据库。

现在可以使用数据库查询API进行增删改查：

```python
>>> from polls.models import Choice, Question

## No questions are in the system yet.
>>> Question.objects.all()
<QuerySet []>

## Create a new Question.
>>> from django.utils import timezone
>>> q = Question(question_text="What's new?", pub_date=timezone.now())

## Save the object into the database. You have to call save() explicitly.
>>> q.save()

## Now it has an ID.
>>> q.id
1

## Access model field values via Python attributes.
>>> q.question_text
"What's new?"
>>> q.pub_date
datetime.datetime(2012, 2, 26, 13, 0, 0, 775217, tzinfo=datetime.timezone.utc)

## Change values by changing the attributes, then calling save().
>>> q.question_text = "What's up?"
>>> q.save()

## objects.all() displays all the questions in the database.
>>> Question.objects.all()
<QuerySet [<Question: What's up?>]>

## Django provides a rich database lookup API that's entirely driven by
## keyword arguments.
>>> Question.objects.filter(id=1)
<QuerySet [<Question: What's up?>]>
>>> Question.objects.filter(question_text__startswith='What')
<QuerySet [<Question: What's up?>]>

## Get the question that was published this year.
>>> from django.utils import timezone
>>> current_year = timezone.now().year
>>> Question.objects.get(pub_date__year=current_year)
<Question: What's up?>

## Request an ID that doesn't exist, this will raise an exception.
>>> Question.objects.get(id=2)
Traceback (most recent call last):
    ...
DoesNotExist: Question matching query does not exist.

## Lookup by a primary key is the most common case, so Django provides a
## shortcut for primary-key exact lookups.
## The following is identical to Question.objects.get(id=1).
>>> Question.objects.get(pk=1)
<Question: What's up?>

## Make sure our custom method worked.
>>> q = Question.objects.get(pk=1)
>>> q.was_published_recently()
True

## Display any choices from the related object set -- none so far.
>>> q.choice_set.all()
<QuerySet []>

## Create three choices.
>>> q.choice_set.create(choice_text='Not much', votes=0)
<Choice: Not much>
>>> q.choice_set.create(choice_text='The sky', votes=0)
<Choice: The sky>
>>> c = q.choice_set.create(choice_text='Just hacking again', votes=0)

## Choice objects have API access to their related Question objects.
>>> c.question
<Question: What's up?>

## And vice versa: Question objects get access to Choice objects.
>>> q.choice_set.all()
<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>
>>> q.choice_set.count()
3

## Find all Choices for any question whose pub_date is in this year
## (reusing the 'current_year' variable we created above).
>>> Choice.objects.filter(question__pub_date__year=current_year)
<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>

## Let's delete one of the choices. Use delete() for that.
>>> c = q.choice_set.filter(choice_text__startswith='Just hacking')
>>> c.delete()
```

数据库查询API的完整细节参考文档[Making queries](https://docs.djangoproject.com/en/stable/topics/db/queries/)和[Related objects reference](https://docs.djangoproject.com/en/stable/ref/models/relations/)。

#### 3.2.5 Django admin应用简介
Django自带的admin应用为模型的增删改查提供了一个方便的界面。该应用是为网站管理者而不是用户提供的。

首先创建一个可以登录到admin应用的用户。执行以下命令：

```bash
python manage.py createsuperuser
```

输入用户名、邮箱和密码。

之后使用`runserver`命令启动服务器，在浏览器中访问 <http://127.0.0.1:8000/admin/>，将会看到admin应用的登录界面：

![admin应用登录界面](/assets/images/django-tutorial/admin应用登录界面.png)

登录后即可看到admin首页：

![admin首页](/assets/images/django-tutorial/admin首页.png)

可以看到目前只有两个可编辑的模型：用户`User`和用户组`Group`，它们是由用户认证应用`django.contrib.auth`提供的。

为了使polls应用定义的模型展示在admin应用中，需要修改polls/admin.py，内容如下：

```python
from django.contrib import admin

from .models import Question, Choice

admin.site.register(Question)
admin.site.register(Choice)
```

刷新页面，即可看到`Question`和`Choice`两个模型：

![admin应用首页2](/assets/images/django-tutorial/admin应用首页2.png)

点击 "Questions" ，即可展示该模型在数据库中的所有数据：

![列表界面](/assets/images/django-tutorial/列表界面.png)

其中 "What's up?" 就是`__str__()`方法返回的字符串。

点击 "What's up?" ，即可打开编辑界面：

![编辑界面](/assets/images/django-tutorial/编辑界面.png)

其中的表单是根据模型定义自动生成的，不同的模型字段类型对应适当的HTML输入组件。

### 3.3 第3部分：编写视图和模板
#### 3.3.1 概述
**视图**(view)是Django应用中处理请求并返回特定web页面的函数或类。

投票应用有以下视图：
* 问题列表视图`index`：展示最新的几个问题。
* 问题详情视图`detail`：展示问题文本、选项和投票表单。
* 投票结果视图`results`：展示特定问题的投票结果。
* 投票视图`vote`：处理一个特定问题、特定选项的投票动作。

Django会根据请求的URL路径（域名之后的部分）查找URLconf，并选择匹配的视图。

#### 3.3.2 编写更多视图
在polls/view.py中增加几个视图。与`index()`不同的是，这几个视图接受一个参数。

```python
def detail(request, question_id):
    return HttpResponse("You're looking at question {}.".format(question_id))


def results(request, question_id):
    return HttpResponse("You're looking at the results of question {}.".format(question_id))


def vote(request, question_id):
    return HttpResponse("You're voting on question {}.".format(question_id))
```

之后在polls/urls.py中添加这些视图的URLconf：

```python
from django.urls import path

from . import views

urlpatterns = [
    # ex: /polls/
    path('', views.index, name='index'),
    # ex: /polls/5/
    path('<int:question_id>/', views.detail, name='detail'),
    # ex: /polls/5/results/
    path('<int:question_id>/results/', views.results, name='results'),
    # ex: /polls/5/vote/
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```

例如，在浏览器中访问 <http://localhost:8000/polls/34/> 时，请求的URL路径是 "/polls/34/" 。Django首先会读取项目的根URLconf `mysite.urls`，匹配到 "polls/" ，并将剩余部分 "34/" 交给`polls.urls`处理。之后匹配到URL模式`'<int:question_id>/'`，因此调用视图`detail(request, question_id=34)`，关键字参数`question_id=34`来自URL模式捕获的路径参数`<int:question_id>`，其中`int`表示参数类型，`question_id`对应参数名。

#### 3.3.3 编写实际执行操作的视图
每个视图负责做两件事：返回一个`HttpResponse`对象，其中包含请求的页面内容（例如HTML页面）；或者产生一个异常（例如`Http404`）。其余是业务逻辑，例如访问数据库。

下面修改`index()`视图，使其展示发布日期最新的5个问题。可以像这样查询最新的5个问题：

```python
Question.objects.order_by('-pub_date')[:5]
```

为了在HTML页面上展示查询结果，需要使用Django的**模板系统**(template system)。**模板**(template)是一种包含特殊占位符的HTML页面，用于根据视图返回的数据动态生成HTML页面。

首先，在polls目录下创建一个templates目录，并在其中再创建一个polls目录。之后，在内层polls目录下创建一个名为index.html的文件，即polls/templates/polls/index.html。在代码中可以用`'polls/index.html'`引用该模板。

注：在templates目录下再创建一个与应用同名的目录是为了避免不同应用中有相同名字的模板，相当于一种“命名空间”。

模板文件的内容如下：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>polls index</title>
</head>
<body>
{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
</body>
</html>
```

现在更新`index()`视图，使其使用该模板：

```python
from django.http import HttpResponse
from django.template import loader

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    template = loader.get_template('polls/index.html')
    context = {'latest_question_list': latest_question_list}
    return HttpResponse(template.render(context, request))
```

注：`template.render()`返回一个字符串，内容是使用context渲染模板生成的HTML页面。

该视图加载了名为polls/index.html的模板，并传递了一个context，即将模板变量映射到Python对象的字典。

启动服务器，并在浏览器中访问 <http://127.0.0.1:8000/polls/>，将会看到一个包含之前创建的 "What's up?" 问题的列表，点击标题将会跳转到详情页。

![问题列表](/assets/images/django-tutorial/问题列表.png)

![问题详情页](/assets/images/django-tutorial/问题详情页.png)

##### 3.3.3.1 捷径：render()
加载一个模板、填充context，并返回一个包含渲染结果的`HttpResponse`是一种非常常见的用法。因此Django提供了一个捷径：`render()`。下面是重写后的`index()`视图：

```python
from django.shortcuts import render

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    context = {'latest_question_list': latest_question_list}
    return render(request, 'polls/index.html', context)
```

其中，`render(request, template_name, context)`等价于`HttpResponse(loader.get_template(template_name).render(context, request))`。

#### 3.3.4 抛出404错误
下面处理`detail()`视图，该页面展示问题文本和选项：

```python
from django.http import Http404
from django.shortcuts import render

from .models import Question


def detail(request, question_id):
    try:
        question = Question.objects.get(pk=question_id)
    except Question.DoesNotExist:
        raise Http404("Question does not exist")
    return render(request, 'polls/detail.html', {'question': question})
```

当请求的问题id不存在时，该视图将抛出`Http404`，Django将返回HTTP状态码404。

模板polls/detail.html的内容将在下一节介绍。

##### 3.3.4.1 捷径：get_object_or_404()
使用`get()`查询对象，当不存在时抛出`Http404`是一种非常常见的用法。Django提供了一个捷径：`get_object_or_404()`。下面是重写后的`detail()`视图：

```python
from django.shortcuts import get_object_or_404, render

from .models import Question


def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/detail.html', {'question': question})
```

函数`get_object_or_404()`的第一个参数是要查询的模型类，之后的任意关键字参数作为查询条件将被传递给模型的`get()`函数，如果不存在则抛出`Http404`。

另外，还有一个类似的函数`get_list_or_404()`，区别是使用`filter()`而不是`get()`，如果查询结果是空列表则抛出`Http404`。

#### 3.3.5 使用模板系统
模板polls/detail.html的内容如下：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>question detail</title>
</head>
<body>
<h1>{{ question.question_text }}</h1>
<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }}</li>
{% endfor %}
</ul>
</body>
</html>
```

模板系统使用`.`语法来访问变量属性，例如`{{ question.question_text }}`（其中`question`是`detail()`视图通过context传递到模板的变量，是一个`Question`对象）。

`{% for %}`循环中的`question.choice_set.all`被解释为Python代码`question.choice_set.all()`，返回一个`Choice`的可迭代对象。

在浏览器中访问 <http://127.0.0.1:8000/polls/1/> 会进入问题 "What's up?" 的详情页：

![问题详情页](/assets/images/django-tutorial/问题详情页2.png)

如果请求的问题id不存在，例如 <http://127.0.0.1:8000/polls/2/>，会返回404页面：

![404页面](/assets/images/django-tutorial/404页面.png)

#### 3.3.6 删除模板中硬编码的URL
在polls/index.html模板中的链接是硬编码的：

```html
<li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
```

这种方法的问题在于，当修改URL时需要修改所有使用该URL的模板。为了解决这一问题，可以使用`{% url %}`标签，通过urls.py中`path()`函数指定的名字来引用URL。例如：

```html
<li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
```

#### 3.3.7 URL命名空间
为了避免不同应用中有相同名字的URL，可以在urls.py中添加一个变量`app_name`，作为URL的命名空间。在polls/urls.py中的`urlpatterns`之前增加

```python
app_name = 'polls'
```

则polls/index.html模板中的链接可以改为

```html
<li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
```

### 3.4 第4部分：表单和通用视图
#### 3.4.1 编写表单
目前的模板polls/detail.html只展示了问题和选项，无法投票。修改该模板：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>question detail</title>
</head>
<body>
<form action="{% url 'polls:vote' question.id %}" method="post">
    {% csrf_token %}
    <fieldset>
        <legend><h1>{{ question.question_text }}</h1></legend>
        {% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}
        {% for choice in question.choice_set.all %}
            <input id="choice{{ forloop.counter }}" type="radio" name="choice" value="{{ choice.id }}">
            <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br>
        {% endfor %}
    </fieldset>
    <input type="submit" value="Vote">
</form>
</body>
</html>
```

效果如下图所示：

![问题详情页](/assets/images/django-tutorial/问题详情页3.png)

注意：
* 每个选项前有一个单选按钮，其`value`属性为选项id，`name`属性为`"choice"`。这意味着提交表单后，将会发送POST数据`choice=#`，其中`#`是选择的选项id。这是HTML表单的基本概念。
* 表单的`method`属性设置为`"post"`，因为提交该表单会修改服务端的数据。**任何修改服务端数据的表单都应该使用POST方法。** 这是web开发的最佳做法，并不限于Django。
* `forloop.counter`是`{% for %}`循环的计数器，从1开始。
* POST表单需要考虑**跨站点请求伪造**(Cross Site Request Forgeries, CSRF)攻击。为此，Django提供了`{% csrf_token %}`标签。

在3.3.2节中增加了`vote()`视图，该视图用于处理表单提交的数据——将指定选项的投票计数加1。下面是该视图的实现：

```python
from django.http import HttpResponse, HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse

from .models import Choice, Question


def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = question.choice_set.get(pk=request.POST['choice'])
    except (KeyError, Choice.DoesNotExist):
        # Redisplay the question voting form.
        return render(request, 'polls/detail.html', {
            'question': question,
            'error_message': "You didn't select a choice.",
        })
    else:
        selected_choice.votes += 1
        selected_choice.save()
        # Always return an HttpResponseRedirect after successfully dealing with POST data.
        # This prevents data from being posted twice if a user hits the Back button.
        return HttpResponseRedirect(reverse('polls:results', args=(question.id,)))
```

* `request.POST`是一个类似于字典的对象，用于访问表单提交的POST数据，值都是字符串。在这里`request.POST['choice']`返回用户选择的选项id（即单选按钮的值）。Django还提供了`request.GET`来访问GET数据。
* 如果POST数据中没有`choice`，`request.POST['choice']`将抛出`KeyError`，此时将重新展示问题表单并显示一个错误信息。
* 增加投票计数后，该视图返回`HttpResponseRedirect`而不是`HttpResponse`，`HttpResponseRedirect`用于重定向到给定的URL。**成功处理POST请求后都应该重定向**，以避免用户点击后退按钮时表单被重复提交。这也是web开发的最佳做法，不限于Django。
* `reverse()`函数用于根据URL的名字和路径参数（如果有）构造URL，从而避免视图函数中硬编码URL（类似于模板中的`{% url %}`标签）。例如，`reverse('polls:results', args=(3,))`将返回`'/polls/3/results/'`。

用户投票后，`vote()`视图将重定向到问题结果页面。`results()`视图与`detail()`几乎完全一样，唯一的区别的模板名称（下一节将解决这一冗余问题）：

```python
def results(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/results.html', {'question': question})
```

模板polls/results.html如下：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>vote results</title>
</head>
<body>
<h1>{{ question.question_text }}</h1>

<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}</li>
{% endfor %}
</ul>

<a href="{% url 'polls:detail' question.id %}">Vote again?</a>
</body>
</html>
```

![结果页面](/assets/images/django-tutorial/结果页面.png)

注：`vote()`视图没有处理竞争条件，即两个用户同时投票可能导致错误的数据。

#### 3.4.2 通用视图
投票应用的`detail()`、`results()`和`index()`视图都很短，也很冗余。这些视图代表了web开发的常见模式：根据URL参数从数据库中获取数据，渲染并返回模板。为此，Django提供了一种捷径——**通用视图**(generic view)。通用视图抽象了公共模式，可以节省很多重复代码。

注：在实际编写Django应用时应首先考虑使用通用视图。本教程为了介绍基本概念而有意地先使用普通方式编写视图。

##### 3.4.2.1 修改URLconf
首先修改polls/urls.py：

```python
from django.urls import path

from . import views

app_name = 'polls'
urlpatterns = [
    path('', views.IndexView.as_view(), name='index'),
    path('<int:pk>/', views.DetailView.as_view(), name='detail'),
    path('<int:pk>/results/', views.ResultsView.as_view(), name='results'),
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```

注意`detail`和`results`视图的路径参数名由`question_id`改为了`pk`。

##### 3.4.2.2 修改视图
删除polls/views.py中旧的`index`、`detail`和`results`视图，改为：

```python
from django.http import HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse
from django.views import generic

from .models import Choice, Question


class IndexView(generic.ListView):
    template_name = 'polls/index.html'
    context_object_name = 'latest_question_list'

    def get_queryset(self):
        """Return the last five published questions."""
        return Question.objects.order_by('-pub_date')[:5]


class DetailView(generic.DetailView):
    model = Question
    template_name = 'polls/detail.html'


class ResultsView(generic.DetailView):
    model = Question
    template_name = 'polls/results.html'


def vote(request, question_id):
    ... # same as above, no changes needed.
```

这里使用了两个通用视图：`ListView`和`DetailView`。这两个视图分别抽象了“展示一个对象列表”和“展示一个特定对象的详情页”的概念。
* 每个通用视图需要通过`model`属性指定要操作的模型。
* `DetailView`通过名为`pk`的URL路径参数指定主键的值。

默认情况下，`DetailView`使用名为`<app_name>/<model_name>_detail.html`的模板，对于投票应用则是polls/question_detail.html。可以通过`template_name`属性指定模板名称。

类似地，`ListView`使用的默认模板名为`<app_name>/<model_name>_list.html`，也可以通过`template_name`属性指定。

`DetailView`使用的默认context变量名为`<model_name>`（例如`question`），`ListView`默认使用`<model_name>_list`（例如`question_list`），也可以通过`context_object_name`属性指定。例如，`IndexView`仍然使用之前的变量名`latest_question_list`。

`ListView`的`get_queryset()`方法返回查询结果。

### 3.5 第5部分：测试
#### 3.5.1 自动化测试
**测试**(test)是检查代码行为的例程。

测试运行在不同的层次，可以验证小的细节（例如一个方法的返回值是否符合预期），也可以验证软件的整体行为（例如用户在网站上的一系列输入是否产生预期的结果）。

自动化测试可以在你修改代码时检查代码是否仍然按预期的方式工作，而无需手动执行测试。

#### 3.5.2 编写第一个测试
##### 3.5.2.1 发现bug
投票应用有一个bug：`Question.was_published_recently()`对于发布日期在将来的`Question`也会返回`True`，这显然是错误的。

可以使用`shell`命令来验证：

```python
>>> import datetime
>>> from django.utils import timezone
>>> from polls.models import Question
>>> future_question = Question(pub_date=timezone.now() + datetime.timedelta(days=30))
>>> future_question.was_published_recently()
True
```

##### 3.5.2.2 编写测试来暴露bug
下面将刚刚在`shell`中测试问题的过程转换为自动化测试。

测试代码通常放在应用目录下的tests.py中。在polls/tests.py中编写以下代码：

```python
import datetime

from django.test import TestCase
from django.utils import timezone

from .models import Question


class QuestionModelTests(TestCase):

    def test_was_published_recently_with_future_question(self):
        """
        was_published_recently() returns False for questions whose pub_date
        is in the future.
        """
        time = timezone.now() + datetime.timedelta(days=30)
        future_question = Question(pub_date=time)
        self.assertFalse(future_question.was_published_recently())
```

这里创建了一个`django.test.TestCase`的子类和一个测试方法。

##### 3.5.2.3 运行测试
执行以下命令：

```bash
python manage.py test polls
```

该命令查找并运行polls应用的测试。将会看到测试失败的错误信息：

```
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
F
======================================================================
FAIL: test_was_published_recently_with_future_question (polls.tests.QuestionModelTests)
was_published_recently() returns False for questions whose pub_date
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/path/to/mysite/polls/tests.py", line 18, in test_was_published_recently_with_future_question
    self.assertFalse(future_question.was_published_recently())
AssertionError: True is not false

----------------------------------------------------------------------
Ran 1 test in 0.002s

FAILED (failures=1)
Destroying test database for alias 'default'...
```

从错误信息中可以知道失败的测试和所在行数。

##### 3.5.2.4 修复bug
下面修复`Question.was_published_recently()`方法的bug，只有当`pub_date`介于昨天和今天之间时才返回`True`：

```python
    def was_published_recently(self):
        now = timezone.now()
        return now - datetime.timedelta(days=1) <= self.pub_date <= now
```

再次运行测试：

```
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.
----------------------------------------------------------------------
Ran 1 test in 0.001s

OK
Destroying test database for alias 'default'...
```

发现bug之后，我们编写了一个测试来暴露它，并修复了bug使测试通过。

在将来，我们的应用可能还会有其他的bug，但通过测试我们可以确保不会再无意地重新引入这个bug。

##### 3.5.2.5 更全面的测试
为了更全面地测试`was_published_recently()`方法，在`QuestionModelTests`中增加两个测试方法，分别使用过去和最近的问题来测试：

```python
def test_was_published_recently_with_old_question(self):
    """
    was_published_recently() returns False for questions whose pub_date
    is older than 1 day.
    """
    time = timezone.now() - datetime.timedelta(days=1, seconds=1)
    old_question = Question(pub_date=time)
    self.assertFalse(old_question.was_published_recently())

def test_was_published_recently_with_recent_question(self):
    """
    was_published_recently() returns True for questions whose pub_date
    is within the last day.
    """
    time = timezone.now() - datetime.timedelta(hours=23, minutes=59, seconds=59)
    recent_question = Question(pub_date=time)
    self.assertTrue(recent_question.was_published_recently())
```

#### 3.5.3 测试视图
投票应用的`index`视图也存在一个问题：它会展示发布日期在将来的问题，这是不合理的。

##### 3.5.3.1 视图的测试
**测试驱动开发**(test-driven development)主张先写测试后写代码（“让测试通过之前先让测试失败”）。但实际上写代码和测试的顺序不重要。

上一节中的测试主要关注代码的内部行为，而这一个测试将检查用户通过浏览器体验到的行为。

##### 3.5.3.2 Django测试客户端
Django提供了一个测试客户端`django.test.Client`来模拟用户在视图层面的交互行为，可以在tests.py或者`shell`中使用。例如：

```python
>>> from django.test.utils import setup_test_environment
>>> setup_test_environment()
>>> from django.test import Client
>>> client = Client()
>>> response = client.get('/')
Not Found: /
>>> response.status_code
404
>>> from django.urls import reverse
>>> response = client.get(reverse('polls:index'))
>>> response.status_code
200
>>> response.content
b'<!DOCTYPE html>\n<html lang="en">\n<head>\n    <meta charset="UTF-8">\n    <title>polls index</title>\n</head>\n<body>\n\n    <ul>\n    \n        <li><a href="/polls/1/">What&#x27;s up?</a></li>\n    \n    </ul>\n\n</body>\n</html>\n'
>>> response.context['latest_question_list']
<QuerySet [<Question: What's up?>]>
```

##### 3.5.3.3 改进视图
为了不展示发布日期在将来的问题，修改`IndexView.get_queryset()`方法：

```python
def get_queryset(self):
    """Return the last five published questions
    (not including those set to be published in the future).
    """
    return Question.objects.filter(pub_date__lte=timezone.now()).order_by('-pub_date')[:5]
```

这里增加了一个条件：`pub_date`小于等于当前时间。

##### 3.5.3.4 测试新视图
为了验证以上修改是否符合预期，可以使用`runserver`启动服务器，创建几个发布日期分别在过去和将来的`Question`，并在浏览器中访问`index`视图，检查是否只展示了已发布的问题。然而，每次修改代码都重复这一过程无疑是很麻烦的，因此创建一个自动化测试。

在polls/tests.py中增加一个创建问题的辅助函数和几个新的测试类：

```python
def create_question(question_text, days):
    """
    Create a question with the given `question_text` and published the
    given number of `days` offset to now (negative for questions published
    in the past, positive for questions that have yet to be published).
    """
    time = timezone.now() + datetime.timedelta(days=days)
    return Question.objects.create(question_text=question_text, pub_date=time)


class QuestionIndexViewTests(TestCase):

    def test_no_questions(self):
        """If no questions exist, an appropriate message is displayed."""
        response = self.client.get(reverse('polls:index'))
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, "No polls are available.")
        self.assertQuerysetEqual(response.context['latest_question_list'], [])

    def test_past_question(self):
        """Questions with a pub_date in the past are displayed on the index page."""
        question = create_question(question_text="Past question.", days=-30)
        response = self.client.get(reverse('polls:index'))
        self.assertQuerysetEqual(
            response.context['latest_question_list'],
            [question],
        )

    def test_future_question(self):
        """Questions with a pub_date in the future aren't displayed on the index page."""
        create_question(question_text="Future question.", days=30)
        response = self.client.get(reverse('polls:index'))
        self.assertContains(response, "No polls are available.")
        self.assertQuerysetEqual(response.context['latest_question_list'], [])

    def test_future_question_and_past_question(self):
        """Even if both past and future questions exist, only past questions
        are displayed.
        """
        question = create_question(question_text="Past question.", days=-30)
        create_question(question_text="Future question.", days=30)
        response = self.client.get(reverse('polls:index'))
        self.assertQuerysetEqual(
            response.context['latest_question_list'],
            [question],
        )

    def test_two_past_questions(self):
        """The questions index page may display multiple questions."""
        question1 = create_question(question_text="Past question 1.", days=-30)
        question2 = create_question(question_text="Past question 2.", days=-5)
        response = self.client.get(reverse('polls:index'))
        self.assertQuerysetEqual(
            response.context['latest_question_list'],
            [question2, question1],
        )
```

其中：
* `assertContains()`和`assertQuerysetEqual`是`django.test.TestCase`提供的额外的断言方法，`assertEqual()`是继承自Python标准库`unittest.TestCase`类的方法。
* `django.test.TestCase`自带了一个测试客户端`self.client`，不需要手动创建。

实际上，我们是在使用测试从用户体验的角度来保证发布的是符合预期的结果。

##### 3.5.3.5 测试DetailView
修复了上面的问题后，虽然未发布的问题不会出现在`index`页面，但如果用户猜到了URL，仍然可以访问这些问题的详情页。因此需要给`DetailView`增加一个类似的限制：

```python
class DetailView(generic.DetailView):
    # ...
    def get_queryset(self):
        """Excludes any questions that aren't published yet."""
        return Question.objects.filter(pub_date__lte=timezone.now())
```

`DetailView.get_queryset()`用于限定对象的查询范围，之后在这个范围内查询指定的id。如果不在这个范围内，即使请求的id在数据库中存在，`DetailView`也会返回404。

下面增加一些测试，分别验证发布日期在过去和将来的问题详情页是否会被展示：

```python
class QuestionDetailViewTests(TestCase):

    def test_future_question(self):
        """The detail view of a question with a pub_date in the future
        returns a 404 not found.
        """
        future_question = create_question(question_text='Future question.', days=5)
        url = reverse('polls:detail', args=(future_question.id,))
        response = self.client.get(url)
        self.assertEqual(response.status_code, 404)

    def test_past_question(self):
        """The detail view of a question with a pub_date in the past
        displays the question's text.
        """
        past_question = create_question(question_text='Past Question.', days=-5)
        url = reverse('polls:detail', args=(past_question.id,))
        response = self.client.get(url)
        self.assertContains(response, past_question.question_text)
```

##### 3.5.3.6 更多测试
我们的应用还可以继续改进，同时增加测试：
* `ResultsView`也可以增加一个类似的`get_queryset()`。
* 没有选项的问题不应该被展示。
* 管理员可以查看未发布的问题，而普通用户不能。

总之，**给软件增加的每一个功能都应伴随一个测试，无论先写测试后编码，还是先编码后测试。**

#### 3.5.4 越多越好
* 不必担心测试代码会越来越多。
* 当软件功能变化时可能需要更新测试。
* 随着开发的进行，有些测试可能变得冗余，但这也没有问题——在测试中冗余是好事。
* 良好的组织测试的规则：
    * 为每个模型或视图编写一个测试类
    * 为每一组想要测试的条件编写一个测试方法（测试用例）
    * 测试方法名描述其功能

关于测试的完整细节参考[Testing in Django](https://docs.djangoproject.com/en/stable/topics/testing/)。

### 3.6 第6部分：样式
除了web服务器生成的HTML，web应用通常需要提供额外的文件，例如图片、JavaScript或CSS，用于渲染完整的web页面。Django将这些文件统称为**静态文件**(static files)。

`django.contrib.staticfiles`应用用于将所有应用的静态文件收集到一个线上环境容易访问的地方。

#### 3.6.1 自定义样式
首先，在polls目录下创建一个static目录，并在其中再创建一个polls目录。之后，在内层polls目录下创建一个名为style.css的文件，即polls/static/polls/style.css。

注：和模板类似，内层polls目录是为了作为“命名空间”，避免不同应用中有相同名字的静态文件。

样式表style.css的内容如下：

```css
li a {
    color: green;
}
```

接下来，在模板polls/index.html的`<head>`部分添加以下内容：

```html
{% load static %}
<link rel="stylesheet" type="text/css" href="{% static 'polls/style.css' %}">
```

`{% static %}`模板标签生成静态文件的绝对URL。

重新加载 <http://localhost:8000/polls/>，将会看到问题链接变为绿色（Django风格）：

![样式表生效](/assets/images/django-tutorial/样式表生效.png)

#### 3.6.2 添加背景图片
在polls/static/polls/目录下创建一个子目录images，在该目录下任意添加一张背景图片，名为background.png。

之后在style.css中添加对图片的引用：

```css
body {
    background: white url("images/background.png") no-repeat;
}
```

重新加载页面，将会看到背景图片出现在屏幕左上角：

![添加背景图片](/assets/images/django-tutorial/添加背景图片.png)

关于静态文件的更多细节参考[How to manage static files](https://docs.djangoproject.com/en/stable/howto/static-files/)。

### 3.7 第7部分：自定义admin应用
#### 3.7.1 自定义表单
在3.2.5节中，通过`admin.site.register(Question)`将`Question`模型注册到admin应用，Django能够为模型创建默认表单。另外也可以对表单进行自定义。

将polls.admin.py中的`admin.site.register(Question)`替换为以下内容：

```python
from django.contrib import admin

from .models import Question


class QuestionAdmin(admin.ModelAdmin):
    fields = ['pub_date', 'question_text']

admin.site.register(Question, QuestionAdmin)
```

创建一个`ModelAdmin`的子类，并将其传递给`admin.site.register()`的第二个参数，即可改变该模型的admin选项。这里通过`fields`属性改变了`Question`表单的字段顺序，如下图所示：

![自定义表单字段顺序](/assets/images/django-tutorial/自定义表单字段顺序.png)

对于有很多字段的表单，可以通过`fieldsets`属性将表单划分为字段组：

```python
class QuestionAdmin(admin.ModelAdmin):
    fieldsets = [
        (None, {'fields': ['question_text']}),
        ('Date information', {'fields': ['pub_date']}),
    ]
```

![自定义表单字段组](/assets/images/django-tutorial/自定义表单字段组.png)

#### 3.7.2 添加关联对象
目前的`Question`表单还未展示选项。有两种方式来解决这一问题。

第一种方式是像`Question`一样注册`Choice`模型：

```python
admin.site.register(Choice)
```

增加`Choice`的表单如下：

![Choice表单](/assets/images/django-tutorial/Choice表单.png)


其中，"Question" 字段表示为下拉列表`<select>`，因为`Choice.question`是一个引用`Question`的外键。

注意 "Question" 字段后面的 "+" 按钮，点击后会弹出一个包含`Question`表单的窗口，从而无需退出当前表单即可直接新增`Question`。点击保存后，Django会将其保存到数据库，并添加到`Choice`表单的下拉列表中。

另一种更好的方式是在创建`Question`对象时直接添加选项。首先删除`admin.site.register(Choice)`，之后修改polls/admin.py：

```python
from django.contrib import admin

from .models import Question, Choice


class ChoiceInline(admin.StackedInline):
    model = Choice
    extra = 3


class QuestionAdmin(admin.ModelAdmin):
    fieldsets = [
        (None, {'fields': ['question_text']}),
        ('Date information', {'fields': ['pub_date'], 'classes': ['collapse']}),
    ]
    inlines = [ChoiceInline]


admin.site.register(Question, QuestionAdmin)
```

![自定义表单关联对象](/assets/images/django-tutorial/自定义表单关联对象.png)

可以看到，表单默认提供了3个选项输入框。

有一个小问题：这种方式占用了太多屏幕空间。Django还提供了一种表格式的展示方式，将`ChoiceInline`的声明修改为

```python
class ChoiceInline(admin.TabularInline):
    # ...
```

则选项输入框会以更紧凑的方式展示：

![自定义表单关联对象2](/assets/images/django-tutorial/自定义表单关联对象2.png)

#### 3.7.3 自定义列表
目前的问题列表只有标题这一列：

![问题列表](/assets/images/django-tutorial/问题列表2.png)

可以通过`list_display`属性自定义该列表展示的列：

```python
class QuestionAdmin(admin.ModelAdmin):
    # ...
    list_display = ('question_text', 'pub_date', 'was_published_recently')
```

![自定义问题列表](/assets/images/django-tutorial/自定义问题列表.png)

点击标题或日期列即可按该字段排序，第3列是`was_published_recently()`方法的返回值，不支持排序。可以在`was_published_recently()`方法上使用`display()`装饰器来改进这一点：

```python
class Question(models.Model):
    # ...
    @admin.display(
        boolean=True,
        ordering='pub_date',
        description='Published recently?'
    )
    def was_published_recently(self):
        now = timezone.now()
        return now - datetime.timedelta(days=1) <= self.pub_date <= now
```

另外，可以在`QuestionAdmin`中使用`list_filter`属性支持按字段过滤，`search_fields `属性将在顶部显示搜索框：

```python
class QuestionAdmin(admin.ModelAdmin):
    # ...
    list_filter = ['pub_date']
    search_fields = ['question_text']
```

![自定义问题列表2](/assets/images/django-tutorial/自定义问题列表2.png)

#### 3.7.4 自定义外观
Admin应用使用了Django自己的模板系统，可以使用模板来改变其外观。

修改mysite/settings.py，在`TEMPLATES`中增加一个`DIRS`选项：

```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [BASE_DIR / 'templates'],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```

`DIRS`是Django加载模板时的搜索路径列表。

在项目根目录下创建一个templates目录，并在其中创建一个admin目录，之后从Django源代码目录中的默认admin模板目录(django/contrib/admin/templates)将模板admin/base_site.html拷贝到该目录下。

注：Django源代码目录可以在Python shell中查看：

```python
>>> import django
>>> print(django.__path__)
['D:\\miniconda3\\envs\\django\\lib\\site-packages\\django']
```

之后，将`{{ site_header|default:_('Django administration') }}`替换为自定义的名称，例如 "Polls Administration" ：

```html
{% block branding %}
<h1 id="site-name"><a href="{% url 'admin:index' %}">Polls Administration</a></h1>
{% endblock %}
```

![自定义admin站点标题](/assets/images/django-tutorial/自定义admin站点标题.png)

通过这种方式即可覆盖默认的admin模板。

#### 3.7.5 自定义首页
如果想自定义admin首页的外观，按照上一节的方式自定义admin/index.html模板即可。
