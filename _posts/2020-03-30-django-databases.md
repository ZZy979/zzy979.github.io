---
title: 【Django】数据库
date: 2020-03-30 00:48 +0800
categories: [Django]
tags: [django, database]
---
## 1.配置数据库
官方文档：
* <https://docs.djangoproject.com/en/stable/ref/databases/>
* <https://docs.djangoproject.com/en/stable/ref/settings/#databases>

settings.py中的`DATABASES`指定项目使用的数据库。

### 1.1 SQLite

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}
```

### 1.2 MySQL
支持版本：MySQL 5.6及以上

安装连接驱动：[mysqlclient](https://mysqlclient.readthedocs.io/)（推荐）

```shell
pip install mysqlclient
```

或[MySQL Connector/Python](https://dev.mysql.com/doc/connector-python/en/)

```shell
pip install mysql-connector-python
```

设置文件：

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'db_name',
        'USER': 'username',
        'PASSWORD': 'password',
        'HOST': '192.168.0.1',
        'PORT': '3306'
    }
}
```

以上方法直接将密码暴露在设置文件中，如果把项目上传到GitHub则是不安全的。

更安全的方式是使用MySQL配置文件，这也是Django官方文档中推荐的方式：

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'OPTIONS': {
            'read_default_file': os.path.join(BASE_DIR, 'my.ini'),
        },
    },
}
```

其中my.ini的内容如下：

```ini
[client]
host = 192.168.0.1
port = 3306
user = username
password = password
database = db_name
default-character-set = utf8
```

该文件可以放在任何地方，这里将其放在工程根目录下（注意路径中不能包含中文，否则MySQL会读取失败），并将其在.gitignore中排除，这样就保护了密码安全。

原理：`OPTIONS`指定了MySQL连接驱动（如mysqlclient）的连接参数`read_default_file`而不是`host`, `user`, `password`等，相当于将`mysql -h 192.168.0.1 -u username -p password -D db_name`改为`mysql --defaults-file=/path/to/my.ini`

## 2.多数据库
官方文档：<https://docs.djangoproject.com/en/stable/topics/db/multi-db/>

### 2.1 设置文件
在`DATABASES`设置中增加一个数据库连接配置并自定义一个别名。例如：

```python
DATABASES = {
    'default': {
        'NAME': 'app_data',
        'ENGINE': 'django.db.backends.postgresql',
        'USER': 'postgres_user',
        'PASSWORD': 's3krit'
    },
    'users': {
        'NAME': 'user_data',
        'ENGINE': 'django.db.backends.mysql',
        'USER': 'mysql_user',
        'PASSWORD': 'priv4te'
    }
}
```

### 2.2 数据库路由器
数据库路由器是定义了`db_for_read()`, `db_for_write()`, `allow_relation()`, `allow_migrate()`四个方法的类，用于决定每个模型使用哪个数据库。

下面的示例定义了users数据库路由器：user应用的模型使用别名为`users`的数据库；其他情况返回`None`，表示该路由器不做决定，由后续路由器来决定。如果没有其他路由器则默认使用`default`。

```python
class UsersRouter:
    route_app_labels = {'user'}

    def db_for_read(self, model, **hints):
        if model._meta.app_label in self.route_app_labels:
            return 'users'
        return None

    def db_for_write(self, model, **hints):
        if model._meta.app_label in self.route_app_labels:
            return 'users'
        return None

    def allow_relation(self, obj1, obj2, **hints):
        if obj1._meta.app_label in self.route_app_labels \
                or obj2._meta.app_label in self.route_app_labels:
            return True
        return None

    def allow_migrate(self, db, app_label, model_name=None, **hints):
        if db == 'users':
            return app_label in self.route_app_labels
        return None
```

假设上面的类定义在{项目根目录}/mysite/routers.py中，则在设置文件中增加：

```python
DATABASE_ROUTERS = ['mysite.routers.UsersRouter']
```

也可以在查询时指定使用的数据库：

```python
u = User.objects.using('users').get(username='fred')
```

### 2.3 测试
在`TestCase`类中增加类属性：

```python
databases = {'default', 'users'}
```
