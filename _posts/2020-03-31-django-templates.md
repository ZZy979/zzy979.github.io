---
title: 【Django】模板
date: 2020-03-31 16:04 +0800
categories: [Django]
tags: [django]
render_with_liquid: false
---
## 1.模板
官方文档：
* <https://docs.djangoproject.com/en/stable/topics/templates/>
* <https://docs.djangoproject.com/en/stable/ref/templates/>

视图返回的HTML页面可以通过使用Django模板标签像JSP页面一样动态生成HTML内容。

默认存放目录：应用目录下的templates目录（需手动创建），其中再创建一个和应用名相同的目录，例如mysite/polls/templates/polls/foo.html，在代码中的引用方式：`"polls/foo.html"`。

admin应用使用的模板：[django/contrib/admin/templates](https://github.com/django/django/tree/main/django/contrib/admin/templates)。

## 2.Django模板语法
官方文档：
* <https://docs.djangoproject.com/en/stable/topics/templates/#the-django-template-language>
* <https://docs.djangoproject.com/en/stable/ref/templates/language/>

模板使用字典`context`渲染，渲染过程包括：
* 在`context`中查找**变量**并替换为具体的值
* 执行**标签**

### 2.1 变量
变量输出`context`中的一个值，语法为`{{ var }}`。

例如，如果`context`为`{'first_name': 'John', 'last_name': 'Doe'}`，则

```html
<p>My first name is {{ first_name }}. My last name is {{ last_name }}.</p>
```

渲染为

```html
<p>My first name is John. My last name is Doe.</p>
```

变量还支持字典、属性、下标和函数调用（只能是无参），但都是点式语法，函数调用也不加括号。

| 变量语法 | 等价的Python语法 |
| --- | --- |
| `{{ my_dict.key }}` | `my_dict[key]` |
| `{{ my_object.attribute }}` | `my_object.attribute` |
| `{{ my_list.0 }}` | `my_list[0]` |
| `{{ my_callable }}` | `my_callable()` |

变量可以用在HTML标签的属性中，例如：

```html
<input id="choice{{ forloop.counter }}" type="radio" name="choice" value="{{ choice.id }}">
```

### 2.2 标签
标签提供渲染过程中的处理逻辑，有些标签也像变量一样产生文本，语法为`{% tag arg1 arg2 ... %}`。

产生文本的标签也可以用在HTML标签的属性中，例如

```html
<a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a>
```

内置标签完整列表：<https://docs.djangoproject.com/en/stable/ref/templates/builtins/#built-in-tag-reference>

#### 2.2.1 for
遍历一个可迭代对象。例如：

```html
<ul>
    {% for question in latest_question_list %}
        <li>{{ question.question_text }}</li>
    {% endfor %}
</ul>
```

使用`forloop.counter`变量获取循环计数器的值（从1开始）：

```html
{% for choice in question.choice_set.all %}
    <input id="choice{{ forloop.counter }}" type="radio" name="choice" value="{{ choice.id }}">
    <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br>
{% endfor %}
```

其他可用的变量见 <https://docs.djangoproject.com/en/stable/ref/templates/builtins/#for> 。

#### 2.2.2 if, elif和else
条件分支。如果`if`后的条件是一个变量名，则为真的条件是存在且布尔值为真。条件中也可以使用比较运算符和逻辑运算符。例如：

```html
{% if athlete_list %}
    Number of athletes: {{ athlete_list|length }}
{% elif athlete_in_locker_room_list %}
    Athletes should be out of the locker room soon!
{% else %}
    No athletes.
{% endif %}
```

在条件表达式中也可以使用过滤器：

```html
{% if athlete_list|length > 1 %}
    Team: {% for athlete in athlete_list %} ... {% endfor %}
{% else %}
    Athlete: {{ athlete_list.0.name }}
{% endif %}
```

#### 2.2.3 url
引用URLconf中的URL模式并代入参数，返回不包含域名的绝对路径。例如：`{% url 'polls:vote' question.id %}`返回`"/polls/123/vote"`。

#### 2.2.4 static
<!-- 由于添加了render_with_liquid: false，这里不能使用post_url标签，只能使用文章的绝对链接 -->

产生静态文件的绝对URL路径，见[静态文件](/posts/django-static-files/)。

#### 2.2.5 include
包含另一个模板。

#### 2.2.6 block和extends
实现模板继承。利用模板继承可以在基模板中包含一些公共部分，在子模板中覆盖`block`标签。

官方文档：<https://docs.djangoproject.com/en/stable/ref/templates/language/#template-inheritance>

基模板中的`block`标签表示的是“有这么一块内容，但不知道该写什么”，类似于抽象类的抽象方法表示的是“有这么一个功能，但不知道该怎么实现”。

### 2.3 过滤器
过滤器用于转换变量或标签参数的值，语法为`{{ var|filter }}`或`{{ var|filter:arg }}`。

内置过滤器完整列表：<https://docs.djangoproject.com/en/stable/ref/templates/builtins/#built-in-filter-reference>

### 2.4 注释

```
{# this won't be rendered #}
```
