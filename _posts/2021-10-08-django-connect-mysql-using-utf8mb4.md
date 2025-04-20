---
title: 【Django】连接MySQL使用utf8mb4字符集
date: 2021-10-08 22:29:55 +0800
categories: [Django]
tags: [django, database, mysql, character encoding]
---
Django向MySQL中插入的字符串包含特殊Unicode字符时报错：
```
django.db.utils.OperationalError: (1366, "Incorrect string value: '\\xF0\\x9D\\x90\\xBF' for column 'abstract' at row 1")
```

该字符是一个4字节的Unicode字符，而MySQL的utf8编码最多只能存储3个字节的字符
```python
>>> b'\xF0\x9D\x90\xBF'.decode('utf8')
'𝐿'
```

在MySQL中要存储4个字节的Unicode字符必须使用utf8mb4字符集
首先要设置数据库或表的字符集为utf8mb4
```sql
CREATE DATABASE foo CHARACTER SET utf8mb4;
```

Django连接数据库的配置中也要指定charset选项：
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'OPTIONS': {
            'read_default_file': '.mylogin.cnf',
            'charset': 'utf8mb4',
        },
    }
}
```

增加该选项后即可正常存储4字节字符

charset参数将被传递给底层连接器mysqlclient，Django的MySQL后端默认给该参数赋值为utf8，而OPTIONS将覆盖默认值

参考代码`django.db.backends.mysql.base.DatabaseWrapper.get_connection_params()`：
```python
def get_connection_params(self):
    kwargs = {
        'conv': django_conversions,
        'charset': 'utf8',
    }
    settings_dict = self.settings_dict
    if settings_dict['USER']:
        kwargs['user'] = settings_dict['USER']
    # ....
    options = settings_dict['OPTIONS'].copy()
    # ...
    kwargs.update(options)
    return kwargs
```

参考文档：
* <https://docs.djangoproject.com/en/stable/ref/databases/#connecting-to-the-database>
* <https://mysqlclient.readthedocs.io/user_guide.html#functions-and-attributes>
