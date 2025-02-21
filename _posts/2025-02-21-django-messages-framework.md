---
title: 【Django】消息框架
date: 2025-02-21 16:29:25 +0800
categories: [Django]
tags: [django]
render_with_liquid: false
---
在web应用中，经常需要在处理完一个表单或其他用户输入后，向用户显示一条一次性的通知消息。为此，Django提供了消息框架，可以在请求中临时存储消息，并在响应或后续请求中获取并显示。

官方文档：<https://docs.djangoproject.com/en/stable/ref/contrib/messages/>

## 1.配置
### 1.1 启用消息框架
`startproject`命令创建的默认设置文件已经包含了启用消息框架需要的所有配置：

```python
INSTALLED_APPS = [
    ...
    'django.contrib.messages',
]

MIDDLEWARE = [
    ...
    'django.contrib.messages.middleware.MessageMiddleware',
]

TEMPLATES = [
    {
        ...
        'OPTIONS': {
            'context_processors': [
                ...
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```

### 1.2 存储后端
消息框架可以使用不同的存储后端。Django提供了三个内置的存储类：`SessionStorage`、`CookieStorage`和`FallbackStorage`（默认），可以通过`MESSAGE_STORAGE`设置指定。例如：

```python
MESSAGE_STORAGE = "django.contrib.messages.storage.cookie.CookieStorage"
```

### 1.3 消息级别
消息有一个级别（类似于Python日志模块），可用于对消息进行过滤。内置级别包括`DEBUG`、`INFO`、`SUCCESS`、`WARNING`和`ERROR`，可以从`django.contrib.messages`导入。

可以使用`MESSAGE_LEVEL`设置指定消息的最小级别，小于这个级别的消息将被忽略。

### 1.4 消息标签
消息标签是消息级别的小写形式字符串表示（如`'info'`），也可以在视图中添加额外的标签。标签存储在字符串中，用空格分隔。消息标签通常用作CSS类以自定义消息样式。

## 2.使用消息
### 2.1 添加消息
在视图中，调用`add_message()`函数添加一条消息。例如：

```python
from django.contrib import messages

messages.add_message(request, messages.INFO, "Hello world.")
```

每种级别都有对应的快捷方法（类似于日志API），例如`messages.info()`。

### 2.2 显示消息
在模板中，可以使用`messages`变量获取消息：

```html
{% if messages %}
    <ul class="messages">
    {% for message in messages %}
        <li{% if message.tags %} class="{{ message.tags }}"{% endif %}>{{ message }}</li>
    {% endfor %}
    </ul>
{% endif %}
```

在视图中，可以使用`get_messages()`：

```python
from django.contrib.messages import get_messages

storage = get_messages(request)
for message in storage:
    do_something_with_the_message(message)
```

当消息被迭代后，就会在响应处理完成时清除。
