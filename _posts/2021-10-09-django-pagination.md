---
title: 【Django】分页
date: 2021-10-09 16:28 +0800
categories: [Django]
tags: [django, pagination]
render_with_liquid: false
---
**分页**(pagination)用于将数据划分为多个页，从而可以在查看数据列表时使用“上一页/下一页”链接。

官方文档：
* <https://docs.djangoproject.com/en/stable/topics/pagination/>
* <https://docs.djangoproject.com/en/stable/ref/paginator/>

## 1.Paginator类
在Django中分页使用`Paginator`类，主要功能是将对象列表（可以是列表、元组、`QuerySet`等）划分为`Page`对象。

创建`Paginator`对象时指定对象列表和每一页包含的对象数量，即可访问每一页包含的对象。

示例：

```python
>>> from django.core.paginator import Paginator
>>> objects = ['john', 'paul', 'george', 'ringo']
>>> p = Paginator(objects, 2)
>>> p.count
4
>>> p.num_pages
2
>>> p.page_range
range(1, 3)

>>> page1 = p.page(1)
>>> page1
<Page 1 of 2>
>>> page1.object_list
['john', 'paul']
>>> page1.number
1
>>> len(page1)
2
>>> page1[1]
'paul'

>>> page2 = p.page(2)
>>> page2.object_list
['george', 'ringo']
>>> page2.has_next()
False
>>> page2.has_previous()
True
>>> page2.next_page_number()
Traceback (most recent call last):
...
django.core.paginator.EmptyPage: That page contains no results
>>> page2.previous_page_number()
1
>>> page2.start_index()
3
>>> page2.end_index()
4

>>> p.page(0)
Traceback (most recent call last):
...
django.core.paginator.EmptyPage: That page number is less than 1
>>> p.page(3)
...
django.core.paginator.EmptyPage: That page contains no results
```

注意：页码和对象索引都从1开始。

## 2.ListView使用分页
通用视图`ListView`提供了内置的分页方式，只需在视图类中添加`paginate_by`（页大小）属性即可：

```python
from django.views.generic import ListView

from myapp.models import Contact

class ContactListView(ListView):
    paginate_by = 2
    model = Contact
```

该属性将限制每一页的对象数量，并在返回的`context`中添加`paginator`, `page_obj`和`object_list`，分别表示使用的分页器、当前页对象和当前页包含的对象列表（`object_list`, `page_obj.object_list`以及原有的`<model>_list`都是同一个`QuerySet`）。从而可以在模板中添加“上一页/下一页”链接：

```html
{% for contact in page_obj %}
    {# Each "contact" is a Contact model object. #}
    {{ contact.full_name|upper }}<br>
    ...
{% endfor %}

<div class="pagination">
    <span class="step-links">
        {% if page_obj.has_previous %}
            <a href="?page=1">&laquo; first</a>
            <a href="?page={{ page_obj.previous_page_number }}">previous</a>
        {% endif %}

        <span class="current">
            Page {{ page_obj.number }} of {{ paginator.num_pages }}.
        </span>

        {% if page_obj.has_next %}
            <a href="?page={{ page_obj.next_page_number }}">next</a>
            <a href="?page={{ paginator.num_pages }}">last &raquo;</a>
        {% endif %}
    </span>
</div>
```

当前页码由GET参数`page`指定（默认为1），返回的`page_obj`就是指定页码对应的页，因此指向其他页的超链接只需增加`page`参数。

这一逻辑由`ListView`自动完成，不需要手动处理。

## 3.视图函数使用分页

```python
from django.core.paginator import Paginator
from django.shortcuts import render

from myapp.models import Contact

def listing(request):
    contact_list = Contact.objects.all()
    paginator = Paginator(contact_list, 25) # Show 25 contacts per page.
    page_number = request.GET.get('page')
    page_obj = paginator.get_page(page_number)
    return render(request, 'list.html', {'page_obj': page_obj})
```

## 4.示例：生成分页栏
`Paginator`类提供了一个方便的方法`get_elided_page_range()`返回页码序列，当页数过多时会自动使用省略号省略中间页，只保留开头、结尾和当前页左右的页码。

```python
>>> from django.core.paginator import Paginator
>>> p = Paginator(list(range(1000)), 20)
>>> list(p.get_elided_page_range(10))
[1, 2, '…', 7, 8, 9, 10, 11, 12, 13, '…', 49, 50]
```

其中`'…'`是类常量`Paginator.ELLIPSIS`。

利用该方法和Bootstrap样式在模板中生成分页栏：

```html
{% if is_paginated %}
    <ul class="pagination justify-content-center">
        {% if page_obj.has_previous %}
            <li class="page-item"><a href="?page={{ page_obj.previous_page_number }}" class="page-link">上一页</a></li>
        {% endif %}
        {% for p in page_range %}
            {% if p == paginator.ELLIPSIS %}
                <li class="page-item disabled"><a href="#" class="page-link">...</a></li>
            {% elif p == page_obj.number %}
                <li class="page-item active"><a href="?page={{ p }}" class="page-link">{{ p }}</a></li>
            {% else %}
                <li class="page-item"><a href="?page={{ p }}" class="page-link">{{ p }}</a></li>
            {% endif %}
        {% endfor %}
        {% if page_obj.has_next %}
            <li class="page-item"><a href="?page={{ page_obj.next_page_number }}" class="page-link">下一页</a></li>
        {% endif %}
    </ul>
{% endif %}
```

其中`page_range`需要在视图中手动添加到`context`：

```python
def get_context_data(self, **kwargs):
    context = super().get_context_data(**kwargs)
    page = context['page_obj']
    context['page_range'] = context['paginator'].get_elided_page_range(page.number)
    return context
```

注意：如果当前url包含其他的GET参数（例如搜索的关键词）则也需要添加到`context`并写在`href`属性中。

效果：

![分页栏](/assets/images/django-pagination/分页栏.png)
