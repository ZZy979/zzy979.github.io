---
title: 【Django】视图
date: 2020-03-29 20:59 +0800
categories: [Django]
tags: [django]
---
## 1.视图
**视图**（view, MVC中的V）用于处理请求并返回特定功能的Web页面，用Python函数或类方法表示。

官方文档：<https://docs.djangoproject.com/en/stable/topics/http/views/>

Django实际上采用的是MTV (Model-Template-View)模式，与Spring MVC的对应关系如下：

| 概念 | Spring | Django |
| --- | --- | --- |
| Model | 实体类 | 模型 |
| View | HTML页面 | 模板 |
| Controller | Controller类 | URLconf+视图 |

应用目录下的views.py定义应用中使用的视图，每个视图接受一个`request`参数和可选的（URL模式中定义的）路径参数，返回一个包含页面内容的`HttpResponse`对象，或者抛出异常（例如`Http404`）。

## 2.基于函数的视图
第一个参数为`request`，如果URL模式中包含路径参数则将其名称作为其他参数。

### 2.1 GET和POST参数
从`request`参数（`django.http.request.HttpRequest`类型的对象）获取
* GET参数：`request.GET['name']`，返回一个字符串，有多个值时取最后一个，如果没有名为`name`的参数则产生`KeyError`
  * 例如`a=1&b=2&b=x` → `{'a': ['1'], 'b': ['2', 'x']}`，则`request.GET['a'] == '1'`, `request.GET['b'] == 'x'`
* POST参数：`request.POST[key]`，返回一个字符串，如果没有名为`name`的参数则产生`KeyError`

### 2.2 返回纯文本的响应

```python
def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
```

对应的URL模式定义：

```python
path('', views.index, name='index')
```

### 2.3 返回HTML模板

```python
def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    # context是一个字典，将HTML模板中的变量映射为Python对象
    context = {'latest_question_list': latest_question_list}
    template = loader.get_template('polls/index.html')
    return HttpResponse(template.render(context, request))
```

注意：
* Django模板系统默认的模板目录是应用下的templates，因此`"polls/index.html"`表示的实际路径是mysite/polls/templates/polls/index.html
* templates目录下还要加一层polls目录的原因是为了避免两个不同应用中有相同名字的模板，内层的polls目录相当于模板的命名空间。
* 渲染模板时提供了一个`context`，其中包含了可以在HTML模板标签中访问的变量。

### 2.4 捷径render()
相当于加载模板+渲染+返回`HttpResponse`。

```python
def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    context = {'latest_question_list': latest_question_list}
    return render(request, 'polls/index.html', context)
```

### 2.5 产生404错误

```python
def detail(request, question_id):
    try:
        question = Question.objects.get(pk=question_id)
    except Question.DoesNotExist:
        raise Http404('Question does not exist')
    return render(request, 'polls/detail.html', {'question': question})
```

### 2.6 捷径get_object_or_404()
使用`get()`获取对象，如果对象不存在则抛出`Http404`。

```python
def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/detail.html', {'question': question})
```

捷径`get_list_or_404()`：使用`filter()`获取对象列表，如果列表为空则抛出`Http404`。

### 2.7 重定向
返回`HttpResponseRedirect`。

```python
return HttpResponseRedirect(reverse('polls:results', args=(question.id,)))
```

注意：这里使用`reverse()`函数避免URL硬编码，`"polls:results"`对应URL模式`"/polls/<int:pk>/results/"`，代入参数`question.id=3`，则`reverse()`函数返回`"/polls/3/results/"`。

良好的Web开发实践：
* **在服务器端修改数据的请求都应使用POST方法。**
* **处理完POST请求后都应重定向**，以避免用户点击后退按钮时表单被重复提交。

## 3.基于类的视图
官方文档：
* <https://docs.djangoproject.com/en/stable/topics/class-based-views/>
* <https://docs.djangoproject.com/en/stable/ref/class-based-views/>

继承`django.views.View`类，并在HTTP方法名对应的方法中定义响应逻辑：

```python
from django.http import HttpResponse
from django.views import View

class MyView(View):
    def get(self, request):
        # <view logic>
        return HttpResponse('result')
```

对应的URL模式定义：

```python
path('', views.IndexView.as_view(), name='index')
```

URL模式中的路径参数通过`self.kwargs`获取，GET和POST参数通过`self.request`获取（同函数视图的第一个参数）。

## 4.通用视图
编写视图时有一些非常常见的模式，例如“根据id查询得到一个实体类并展示详细信息”、“根据某些条件查询得到一个实体类列表并展示这个列表”等。Django通用视图封装了这些固定的逻辑，从而帮助减少编写重复的代码。编写Django应用时应首先考虑使用通用视图。

官方文档：
* <https://docs.djangoproject.com/en/stable/topics/class-based-views/generic-display/>
* <https://docs.djangoproject.com/en/stable/ref/class-based-views/>

### 4.1 详情视图
`django.views.generic.DetailView`：展示一个对象详情页面。

URL路径参数：`pk`，要展示的对象id，例如`'question/<int:pk>/'`。

类属性

| 属性 | 描述 |
| --- | --- |
| `template_name` | 返回的模板名字，默认为`"<app_name>/<model_name>_detail.html"`，例如`"polls/question_detail.html"` |
| `queryset` | 指定对象查询范围 |
| `model` | 要展示的对象所属的模型类，`model = Foo`等价于`queryset = Foo.objects.all()` |
| `context_object_name` | 指定对象的名字 |

方法

| 方法 | 描述 |
| --- | --- |
| `get_queryset()` | 返回对象的查询范围（然后在这个范围内查找`id=pk`的对象），如果指定了类属性`queryset`则直接使用它，否则默认的范围是全部对象，即`model.objects.all()` |
| `get_object()` | 返回要显示的（一个）对象，可以通过覆盖该方法来显示任意的对象，默认为`get_queryset().get(pk=pk)`，如果不存在则产生`Http404` |
| `get_context_object_name()` | 返回要显示的对象的名字（在HTML模板中使用），如果指定了类属性`context_object_name`则直接使用它，否则返回实体类名的小写形式 |
| `get_context_data()` | 返回渲染模板的`context`字典，默认为`{'object': obj}`，其中`obj`是`get_object()`的返回值；如果`get_context_object_name()`不为空则也将其添加到`context`中，对应的值也是`obj` |

注意：如果覆盖`get_context_data()`方法来添加其他自定义的对象，要先调用超类方法来获取超类添加的对象，如下：

```python
def get_context_data(self, **kwargs):
    context = super().get_context_data(**kwargs)
    context['foo'] = bar
    return context
```

方法调用过程：

```
View.as_view.view()  # 入口
  View.dispatch()
    BaseDetailView.get()  # 真正的响应逻辑，对应HTTP的GET方法
      SingleObjectMixin.get_object()  # 获取要显示的对象
        SingleObjectMixin.get_queryset()  # 获取对象查找范围
      SingleObjectMixin.get_context_data()  # 获取渲染模板的context
        SingleObjectMixin.get_context_object_name()  # 获取对象的名字
      TemplateResponseMixin.render_to_response()  # 渲染模板并返回响应
```

源代码：[django/views/generic/detail.py](https://github.com/django/django/blob/main/django/views/generic/detail.py)

例如：

```python
class DetailView(generic.DetailView):
    model = Question
    template_name = 'polls/detail.html'

    def get_queryset(self):
        """Excludes any questions that aren't published yet."""
        return Question.objects.filter(pub_date__lte=timezone.now())
```

对应的URL模式定义：

```python
path('<int:pk>/', views.DetailView.as_view(), name='detail')
```

则在polls/detail.html中可以使用`object`或`question`访问该视图返回的对象。

### 4.2 列表视图
`django.views.generic.ListView`：展示一个对象列表。

类属性

| 属性 | 描述 |
| --- | --- |
| `template_name` | 返回的模板名字，默认为`"<app_name>/<model_name>_list.html"`，例如`"polls/question_list.html"` |
| `queryset` | 指定对象查询范围 |
| `model` | 要展示的对象所属的模型类，`model = Foo`等价于`queryset = Foo.objects.all()` |
| `ordering` | 字段名列表，指定结果排序方式 |
| `context_object_name` | 指定对象列表的名字 |
| `paginate_by` | 指定分页的页大小（见[分页]({% post_url 2021-10-09-django-pagination %})） |

方法

| 方法 | 描述 |
| --- | --- |
| `get_queryset()` | 返回对象的查询范围，如果指定了类属性`queryset`则直接使用它，否则默认的范围是全部对象，即`model.objects.all()` |
| `get_context_object_name()` | 返回要显示的对象列表的名字（在HTML模板中使用），如果指定了类属性`context_object_name`则直接使用它，否则返回实体类名的小写形式+`"_list"` |
| `get_context_data()` | 返回渲染模板的`context`字典 |

`get_context_data()`方法返回的字典默认包含的键：
* `'object_list'`：`get_queryset()`返回的对象列表`obj_list`，如果`get_context_object_name()`不为空则也将其添加到`context`中，对应的值也是`obj_list`
* `'is_paginated'`：`bool`值，结果是否分页
* `'paginator'`：`django.core.paginator.Paginator`类型的对象或`None`
* `'page_obj'`：`django.core.paginator.Page`类型的对象或`None`

方法调用过程：

```
View.as_view.view()  # 入口
  View.dispatch()
    BaseListView.get()  # 真正的响应逻辑，对应HTTP的GET方法
      MultipleObjectMixin.get_queryset()  # 获取对象查找范围
      MultipleObjectMixin.get_context_data()  # 获取渲染模板的context
        MultipleObjectMixin.get_context_object_name()  # 获取对象列表的名字
        MultipleObjectMixin.paginate_queryset()  # 查询结果分页
      TemplateResponseMixin.render_to_response()  # 渲染模板并返回响应
```

源代码：[django/views/generic/list.py](https://github.com/django/django/blob/main/django/views/generic/list.py)

例如：

```python
class IndexView(generic.ListView):
    template_name = 'polls/index.html'
    context_object_name = 'latest_question_list'

    def get_queryset(self):
        """Return the last five published questions
        (not including those set to be published in the future).
        """
        return Question.objects.filter(pub_date__lte=timezone.now()).order_by('-pub_date')[:5]
```

对应的URL模式定义：

```python
path('', views.IndexView.as_view(), name='index')
```

则在polls/index.html中可以使用`object_list`或`latest_question_list`访问该视图返回的对象列表。

注意：`ListView`做的事情是查询对象列表、将结果保存在`context`中、使用`context`渲染模板，而不是生成HTML页面中的列表，使用的模板还是需要自己提供。

### 4.3 编辑视图
<https://docs.djangoproject.com/en/stable/ref/class-based-views/generic-editing/>

* `django.views.generic.FormView` 显示一个表单，错误时重新显示表单和错误信息；成功时重定向到新URL
* `django.views.generic.CreateView` 显示一个模型表单，用于创建并保存一个对象
* `django.views.generic.UpdateView` 显示一个模型表单，用于编辑一个已有对象
* `django.views.generic.DeleteView` 显示确认页面并删除一个已有对象

完整示例：<https://github.com/ZZy979/library_management/blob/master/library/views.py>
* 视图：`BookCreateView`、`BookUpdateView`和`BookDeleteView`
* 模板：book_form.html和book_confirm_delete.html
