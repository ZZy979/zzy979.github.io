---
title: Django实践——用户登录
date: 2020-04-20 19:29:45 +0800
categories: [Django]
tags: [django]
render_with_liquid: false
---
Django默认提供了一个[用户认证系统](https://docs.djangoproject.com/en/3.0/topics/auth/default/)，但主要面向admin应用中用户对各种实体类的增删改查权限，比较复杂，本项目实现一个较简单的用户登录功能。

## 第1步：创建项目及应用

```shell
django-admin startproject DjangoLoginDemo
cd DjangoLoginDemo
python manage.py startapp login
```

将login应用添加到设置文件中，在DjangoLoginDemo/settings.py的`INSTALLED_APPS`中添加一行`'login.apps.LoginConfig'`

## 第2步：创建模型（实体类）
①在login/models.py中创建`User`类：

```python
import hashlib

from django.db import models


class User(models.Model):
    """用户实体类"""
    username = models.CharField(max_length=32, unique=True)
    password = models.CharField(max_length=40)

    def __str__(self):
        return self.username

    @classmethod
    def encrypt_password(cls, password):
        """使用SHA-1加密密码，返回长度为40的加密后的字符串。"""
        return hashlib.sha1(password.encode()).hexdigest()
```

其中用户密码使用SHA-1加密并转换为十六进制字符串，长度为40个字符。

②将模型应用到数据库中（数据库使用默认的SQLite）：

```shell
python manage.py makemigrations login
python manage.py migrate
```

等待执行完后就在数据库中创建了一个对应的表login_user（除此之外还创建了Django认证系统相关的表，与本项目无关）

![创建数据表](/assets/images/django-login-demo/创建数据表.png)

## 第3步：创建HTML模板
在login目录下创建templates目录，在templates目录下再创建一个login目录，在其中创建base.html, index.html, register.html, login.html四个文件。base.html作为基模板被其他模板继承，内容如下；另外三个分别作为主页、注册页面和登录页面，内容暂时为空。

```html
<!DOCTYPE html>
<html lang="zh-hans">
<head>
    <meta charset="UTF-8">
    <title>{% block title %}{% endblock %}</title>
    {% block script %}{% endblock %}
</head>
<body>
<div id="content">
    {% block header %}<h1>Django Login Demo</h1>{% endblock %}
    <hr>
    {% block content %}{% endblock %}
    <hr>
    <footer>
        <small>Author: <a href="https://github.com/ZZy979">ZZy</a></small>
    </footer>
</div>
</body>
</html>
```

## 第4步：实现视图逻辑
计划实现的URL及其对应的视图逻辑如下：
* index/：主页，如果未登录则显示“登录”和“注册”链接，否则显示欢迎信息
* register/：如果是GET方法则表示用户点击了“注册”链接，直接返回register.html；如果是POST方法则表示用户提交了注册表单，验证用户填写的信息，如果成功则重定向到首页，否则返回注册页面并显示错误信息
* login/：如果是GET方法则表示用户点击了“登录”链接，直接返回login.html；如果是POST方法则表示用户提交了登录表单，如果登录成功则通过Django提供的session机制将用户名保存在session中，并重定向到首页，否则返回登录页面并显示错误信息
* logout/：用户注销，从session中删除用户名，并重定向到首页

注：以上的URL模式省略了应用名前缀"login/"

①login/views.py内容如下：

```python
from django.http import HttpResponseRedirect
from django.shortcuts import render, redirect
from django.urls import reverse
from django.views.decorators.http import require_http_methods

from login.models import User


def index(request):
    return render(request, 'login/index.html', {'message': request.GET.get('message')})


@require_http_methods(['GET', 'POST'])
def register(request):
    if request.method == 'GET':
        return render(request, 'login/register.html')
    username = request.POST['username']
    password = User.encrypt_password(request.POST['password'])
    if User.objects.filter(username=username).exists():
        return render(request, 'login/register.html', {'error_message': '用户名已存在'})
    User.objects.create(username=username, password=password)
    return HttpResponseRedirect(reverse('login:index') + '?message=注册成功')


@require_http_methods(['GET', 'POST'])
def login(request):
    if request.method == 'GET':
        return render(request, 'login/login.html')
    username = request.POST['username']
    password = User.encrypt_password(request.POST['password'])
    try:
        user = User.objects.get(username=username)
        if user.password == password:
            request.session['username'] = user.username
            return redirect('login:index')
        else:
            return render(request, 'login/login.html', {'error_message': '用户名或密码错误'})
    except User.DoesNotExist:
        return render(request, 'login/login.html', {'error_message': '用户名或密码错误'})


def logout(request):
    if 'username' in request.session:
        del request.session['username']
    return HttpResponseRedirect(reverse('login:index') + '?message=注销成功')
```

②在login目录下创建文件urls.py，表示login应用包含的URL模式，内容如下：

```python
from django.urls import path

from login import views

app_name = 'login'
urlpatterns = [
    path('index/', views.index, name='index'),
    path('register/', views.register, name='register'),
    path('login/', views.login, name='login'),
    path('logout/', views.logout, name='logout'),
]
```

③在DjangoLoginDemo/urls.py的`urlpatterns`中添加一行`path('login/', include('login.urls'))`，最终该文件内容如下：

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('login/', include('login.urls')),
]
```

## 第5步：编写前端HTML模板
①index.html

```html
{% extends 'login/base.html' %}

{% block title %}首页{% endblock %}

{% block script %}
{% if message %}
    <script>alert('{{ message }}')</script>
{% endif %}
{% endblock %}

{% block content %}
{% if request.session.username %}
    <nav>
        <a href="{% url 'login:logout' %}">注销</a>
    </nav>
    <p>欢迎，{{ request.session.username }}！</p>
{% else %}
    <nav>
        <a href="{% url 'login:login' %}">登录</a> |
        <a href="{% url 'login:register' %}">注册</a>
    </nav>
    <p>欢迎！</p>
{% endif %}
{% endblock %}
```

②register.html

```html
{% extends 'login/base.html' %}

{% block title %}用户注册{% endblock %}

{% block header %}<h1>用户注册</h1>{% endblock %}

{% block content %}
{% if error_message %}<p style="color: red">{{ error_message }}</p>{% endif %}
<form action="{% url 'login:register' %}" method="post">
    {% csrf_token %}
    <label for="username-input">用户名</label>
    <input id="username-input" type="text" name="username"
           pattern="[0-9A-Za-z_]+" maxlength="32"
           placeholder="仅字母、数字和下划线" required>
    <br>
    <label for="password-input">密码</label>
    <input id="password-input" type="password" name="password" maxlength="32" required>
    <br>
    <input type="submit" value="注册">
</form>
{% endblock %}
```

③login.html

```html
{% extends 'login/base.html' %}

{% block title %}用户登录{% endblock %}

{% block header %}<h1>用户登录</h1>{% endblock %}

{% block content %}
{% if error_message %}<p style="color: red">{{ error_message }}</p>{% endif %}
<form action="{% url 'login:login' %}" method="post">
    {% csrf_token %}
    <label for="username-input">用户名</label>
    <input id="username-input" type="text" name="username"
           pattern="[0-9A-Za-z_]+" maxlength="32" required>
    <br>
    <label for="password-input">密码</label>
    <input id="password-input" type="password" name="password" maxlength="32" required>
    <br>
    <input type="submit" value="登录">
</form>
{% endblock %}
```

至此代码部分完成，运行服务器：

```shell
python manage.py runserver 0.0.0.0:8000
```

访问主页：http://127.0.0.1:8000/login/index/

## 功能截图
①主页

![主页](/assets/images/django-login-demo/主页.png)

②注册

![注册页面](/assets/images/django-login-demo/注册页面.png)

![注册成功](/assets/images/django-login-demo/注册成功.png)

③登录

![登录页面](/assets/images/django-login-demo/登录页面.png)

![登录成功](/assets/images/django-login-demo/登录成功.png)

④注销

![注销成功](/assets/images/django-login-demo/注销成功.png)

## 项目目录结构
![目录结构](/assets/images/django-login-demo/目录结构.png)

GitHub仓库地址：[DjangoLoginDemo](https://github.com/ZZy979/DjangoLoginDemo)
