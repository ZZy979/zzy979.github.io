---
title: 【Django】认证系统
date: 2020-04-30 22:22 +0800
categories: [Django]
tags: [django, authentication]
---
Django默认提供了一个认证系统，该系统提供认证(authentication)和授权(authorization)两种功能。认证即用户登录，授权用于控制用户权限。

官方文档：<https://docs.djangoproject.com/en/stable/topics/auth/>

Django默认的认证系统：<https://docs.djangoproject.com/en/stable/topics/auth/default/>

## 1.User对象
`User`模型是认证系统的核心，既可以表示注册的用户，也可以表示admin应用的用户。只有`user.is_staff==True`的用户可以登录admin应用。

用户模型类：`django.contrib.auth.models.User`，对应的数据库表为`auth_user`。

### 1.1 创建用户

```python
from django.contrib.auth.models import User
user = User.objects.create_user('john', 'lennon@thebeatles.com', 'johnpassword')
```

Django将自动对密码进行加密。

### 1.2 创建超级用户

```shell
python manage.py createsuperuser
```

超级用户的`is_superuser`字段为`True`，默认拥有所有权限。

### 1.3 修改密码
在命令行修改：

```shell
python manage.py changepassword username
```

在代码中修改：

```python
user.set_password('new password')
user.save()
```

也可以在admin应用的表单中修改。

另外，Django提供了用于修改密码的内置视图和表单，见“认证视图”一节。

### 1.4 认证用户

```python
authenticate(request, username, password)
```

相当于一次数据库查询，如果用户存在且密码正确则返回一个`User`对象，否则返回`None`。

### 1.5 自定义User模型
添加字段：<https://docs.djangoproject.com/en/stable/topics/auth/customizing/#extending-the-existing-user-model>

替换`User`模型类：<https://docs.djangoproject.com/en/stable/topics/auth/customizing/#substituting-a-custom-user-model>

## 2.Web请求认证
Django使用session保存认证信息，`request.user`表示当前用户，`request.user.is_authenticated`表示是否已登录。

### 2.1 用户登录

```python
login(request, user)
```

登录用户，将用户信息保存在session中。

```python
from django.contrib.auth import authenticate, login

def login_view(request):
    username = request.POST['username']
    password = request.POST['password']
    user = authenticate(request, username=username, password=password)
    if user is not None:
        login(request, user)
        # Redirect to a success page.
    else:
        # Return an 'invalid login' error message.
```

### 2.2 用户注销

```python
logout(request)
```

注销用户，将session中的用户数据移除。

```python
from django.contrib.auth import logout

def logout_view(request):
    logout(request)
    # Redirect to a success page.
```

## 3.权限和授权
Django内置了权限系统，用于给特定的用户或一组用户授予权限。

admin应用使用权限控制用户对各种实体类的增删改查权限，也可以自定义用户权限。

### 3.1 权限
每个权限与一个`ContentType`关联，有`codename`和`name`两个字段，分别表示权限名称和人类可读名称。

`ContentType`表示一个应用中的一个模型，即`(app_label, model)`。权限由`(app_label, code_name)`唯一标识。

例如，polls应用中的`Choice`模型对应的`ContentType`为`('polls', 'choice')`，该模型的`add_choice`权限表示为`"polls.add_choice"`（因此同一个应用下不同模型的权限的`code_name`不能重复）。

权限模型类：`django.contrib.auth.models.Permission`，对应的数据表：`auth_permission`，但很少直接使用该模型类。

### 3.2 默认权限
在执行`migrate`命令时，Django将自动为每个模型创建四个默认权限（假设模型名称为`Foo`）：`add_foo`, `change_foo`, `delete_foo`, `view_foo`。

### 3.3 用户组
用户组是一种批量控制用户权限的方式，组内用户自动拥有该用户组的所有权限，每个用户可以属于任意数量的用户组。

模型类：`django.contrib.auth.models.Group`，对应的数据表：`auth_group`。

用户所属的用户组字段：`user.groups`，多对多外键。

除了权限，用户组也可以用于对用户进行分类，赋予用户某些“标签”。例如，可以创建一个Admin用户组，属于该组的用户是管理员，否则是普通用户（这里的“管理员”是自己的Web应用的概念，与Django的admin应用无关）。

### 3.4 授予权限
可以给单个用户授予权限，也可以给用户组授予权限。

用户的权限字段：`user.user_permissions`，多对多外键。

用户组的权限字段：`group.permissions`，多对多外键。

检查用户是否有某个权限：`user.has_perm(perm)`，其中`perm`是一个字符串，格式为`"<app_label>.<code_name>"`。

### 3.5 自定义权限
在模型的`Meta`类中定义权限：<https://docs.djangoproject.com/en/stable/topics/auth/customizing/#custom-permissions>

在代码中创建权限：

```python
from myapp.models import BlogPost
from django.contrib.auth.models import Permission
from django.contrib.contenttypes.models import ContentType

content_type = ContentType.objects.get_for_model(BlogPost)
permission = Permission.objects.create(
    codename='can_publish',
    name='Can Publish Posts',
    content_type=content_type,
)
```

注意：
* 权限是和模型绑定的，表示模型的某种增删改查操作。例如，上面例子中`BlogPost`模型的`can_publish`权限表示“可以添加`BlogPost`”。
* 如果需要区分用户类型时不涉及模型的增删改查操作，那么使用用户组即可（即判断用户是否属于某个用户组），不需要使用权限。

## 4.限制登录的用户
### 4.1 限制登录后才能访问
（1）原始方式

检查`request.user.is_authenticated`。

（2）基于函数的视图

使用`@login_required`装饰器：

```python
login_required(redirect_field_name='next', login_url=None)
```

如果用户已登录则正常执行视图；否则重定向到`login_url`（默认为`settings.LOGIN_URL`），同时将当前URL作为GET参数`redirect_field_name`（默认为`"next"`），用于登录成功后返回当前URL（这个逻辑需要自己在登录视图中实现）。

示例：

```python
from django.contrib.auth.decorators import login_required

@login_required
def my_view(request):
    ...
```

注：根据Python的装饰器语法，上述代码等价于

```python
def my_view(request):
    ...

my_view = login_required(my_view)
```

注意上面的写法中`@login_required`最后没有括号，因此是`login_required()`函数本身作为装饰器，被装饰函数`my_view`作为`login_required()`函数的参数。

该装饰器也可以带参数，此时并不是直接将`my_view`传给`login_required()`函数，而是传给`login_required()`函数返回的另一个装饰器：

```python
@login_required(login_url='/login')
def my_view(request):
    ...
```

等价于

```python
def my_view(request):
    ...

my_view = login_required(login_url='/login')(my_view)
```

查看`login_required()`函数的[源代码](https://github.com/django/django/blob/main/django/contrib/auth/decorators.py)可以看到：如果第一个参数（被装饰函数）不为`None`则直接返回装饰后的函数（第一种写法），否则返回一个新的装饰器（第二种写法）。

```python
def login_required(function=None, redirect_field_name=REDIRECT_FIELD_NAME, login_url=None):
    actual_decorator = ...
    if function:
        return actual_decorator(function)
    return actual_decorator
```

（3）基于类的视图

使用`LoginRequiredMixin`混入类。示例：

```python
from django.contrib.auth.mixins import LoginRequiredMixin

class MyView(LoginRequiredMixin, View):
    login_url = '/login/'
    redirect_field_name = 'next'
```

注意：混入类必须在继承列表的最前边，不能同时使用多个混入类。

### 4.2 限制用户权限
（1）基于函数的视图

使用`@permission_required`装饰器：

```python
permission_required(perm, login_url=None, raise_exception=False)
```

示例：

```python
from django.contrib.auth.decorators import permission_required

@permission_required('polls.add_choice')
def my_view(request):
    ...
```

其中`perm`参数是一个字符串，格式为`"<app_label>.<code_name>"`；也可以是权限列表，此时用户必须有所有权限。
* 如果用户有指定的权限则正常执行视图。
* 否则如果`raise_exception==True`则抛出`PermissionDenied`异常，返回403。
* 否则重定向到`login_url`（默认为`settings.LOGIN_URL`）。

（2）基于类的视图

使用`PermissionRequiredMixin`混入类。示例：

```python
from django.contrib.auth.mixins import PermissionRequiredMixin

class MyView(PermissionRequiredMixin, View):
    permission_required = 'polls.add_choice'
    # Or multiple of permissions:
    # permission_required = ('polls.view_choice', 'polls.change_choice')
```

### 4.3 自定义限制条件
（1）基于函数的视图

使用`@user_passes_test`装饰器：

```python
user_passes_test(test_func, login_url=None, redirect_field_name='next')
```

`test_func`是接收`User`参数、返回布尔值的函数，如果返回`True`则允许访问，否则重定向到登录页面。

示例：

```python
from django.contrib.auth.decorators import user_passes_test

def email_check(user):
    return user.email.endswith('@example.com')

@user_passes_test(email_check)
def my_view(request):
    ...
```

（2）基于类的视图

使用`UserPassesTestMixin`混入类，并覆盖`test_func()`或`get_test_func()`方法。示例：

```python
from django.contrib.auth.mixins import UserPassesTestMixin

class MyView(UserPassesTestMixin, View):
    def test_func(self):
        return self.request.user.email.endswith('@example.com')
```

### 4.4 重定向未授权的用户
`LoginRequiredMixin`, `PermissionRequiredMixin`和`UserPassesTestMixin`都是`AccessMixin`类的子类。

`AccessMixin`类用于在基于类的视图中控制当访问被拒绝时的行为。主要属性：`login_url`, `redirect_field_name`, `raise_exception`, `permission_denied_message`。
* 对于未登录的用户：如果`raise_exception==False`（默认）则重定向到登录页面；否则抛出`PermissionDenied`异常，返回403。
* 对于已登录但未授权的用户（没有权限或不满足自定义条件）：抛出`PermissionDenied`异常，返回403。

## 5.认证视图
Django提供了用于登录、注销和密码管理的内置视图及表单（没有模板）。

### 5.1 使用视图
Django提供的认证视图的URLconf为`django.contrib.auth.urls`，使用`include()`函数将其包含到自己的URLconf：

```python
urlpatterns = [
    path('accounts/', include('django.contrib.auth.urls')),
    ...
]
```

将会包含以下URL模式：

| URL模式 | 名字 | 视图 |
| --- | --- | --- |
| `accounts/login/` | `login` | `LoginView` |
| `accounts/logout/` | `logout` | `LogoutView` |
| `accounts/password_change/` | `password_change` | `PasswordChangeView` |
| `accounts/password_change/done/` | `password_change_done` | `PasswordChangeDoneView` |
| `accounts/password_reset/` | `password_reset` | `PasswordResetView` |
| `accounts/password_reset/done/` | `password_reset_done` | `PasswordResetDoneView` |
| `accounts/reset/<uidb64>/<token>/` | `password_reset_confirm` | `PasswordResetConfirmView` |
| `accounts/reset/done/` | `password_reset_complete` | `PasswordResetCompleteView` |

这些视图都是基于类的，可以通过继承来自定义。

视图详细信息及使用示例：<https://docs.djangoproject.com/en/stable/topics/auth/default/#all-authentication-views>

内置表单：<https://docs.djangoproject.com/en/stable/topics/auth/default/#module-django.contrib.auth.forms>
