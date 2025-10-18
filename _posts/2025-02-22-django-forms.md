---
title: 【Django】表单
date: 2025-02-22 22:37:23 +0800
categories: [Django]
tags: [django, form]
render_with_liquid: false
---
Django中的表单(form)是处理用户输入数据的强大工具，可用于生成HTML表单、验证用户输入的数据以及显示错误信息。

官方文档：
* <https://docs.djangoproject.com/en/stable/topics/forms/>
* <https://docs.djangoproject.com/en/stable/ref/forms/>

## 1.基本用法
### 1.1 定义表单
表单与模型类似，模型的字段对应数据库字段，表单字段对应HTML表单元素（如`<input>`）。

Django表单的核心是`django.forms.Form`类，通过继承该类来自定义表单。例如，可以如下创建一个联系人表单：

```python
from django import forms

class ContactForm(forms.Form):
    name = forms.CharField(label='Your name', max_length=100)
    email = forms.EmailField(label='Your email')
    message = forms.CharField(label='Your message', widget=forms.Textarea)
```

表单类通常定义在应用目录下的forms.py文件中。

### 1.2 在视图中使用表单
为了处理表单，需要在视图中将其实例化。

```python
from django.shortcuts import render, redirect
from .forms import ContactForm

def contact_view(request):
    if request.method == 'POST':
        form = ContactForm(request.POST)
        if form.is_valid():
            # process the data in form.cleaned_data as required
            name = form.cleaned_data['name']
            email = form.cleaned_data['email']
            message = form.cleaned_data['message']
            # save to database or send email ...
            return redirect('success_page')
    else:
        form = ContactForm()
    return render(request, 'contact.html', {'form': form})
```

在视图函数中，可以通过`request.method`判断是显示表单还是处理表单提交。
* 如果是GET请求，表示首次访问这个页面，则创建一个空的表单（称为“未绑定的”表单），并将其放在模板上下文中进行渲染，以便给用户填写。
* 如果是POST请求，表示用户已经提交了表单，则再次创建一个表单实例，并使用请求中的数据填充（称为“绑定的”表单）。

表单的`is_valid()`方法会检查所有字段。如果所有字段都包含合法数据，则返回`True`并将表单数据放在`cleaned_data`属性中。此时可以处理数据（例如保存到数据库或发送邮件）并重定向到其他页面。否则，带着表单返回模板，这次HTML表单将使用之前提交的数据填充，同时可以显示错误信息，以便用户进行编辑和修正。

### 1.3 在模板中渲染表单
在模板中，可以使用`{{ form }}`渲染整个表单。

```html
<form action="/contact" method="post">
    {% csrf_token %}
    {{ form }}
    <button type="submit">Submit</button>
</form>
```

表单方法`as_div`、`as_p`、`as_ul`和`as_table`分别将表单渲染为`<div>`、`<p>`、`<ul>`和`<table>`元素。例如，`{{ form.as_div }}`将生成以下HTML：

```html
<div>
    <label for="id_name">Name:</label>
    <input type="text" name="name" maxlength="100" required id="id_name">
</div>
<div>
    <label for="id_email">Email:</label>
    <input type="email" name="email" maxlength="320" required id="id_email">
</div>
<div>
    <label for="id_message">Message:</label>
    <textarea name="message" cols="40" rows="10" required id="id_message">
    </textarea>
</div>
```

注意，其中不包含`<form>`标签和提交按钮，需要自己在模板中提供。

注：`{{ form }}`使用默认的渲染方式，取决于[模板后端](https://docs.djangoproject.com/en/stable/ref/settings/#std-setting-TEMPLATES-BACKEND)。

也可以遍历表单字段并手动渲染：

```html
{% for field in form %}
    <div class="fieldWrapper">
        {{ field.errors }}
        {{ field.label_tag }}
        {{ field }}
        {% if field.help_text %}
            <p class="help" id="{{ field.auto_id }}_helptext">
                {{ field.help_text|safe }}
            </p>
        {% endif %}
    </div>
{% endfor %}
```

## 2.表单字段
Django提供了多种表单字段类型，常用的包括：

| 字段类 | 默认元素 |
| --- | --- |
| `CharField` | `<input type="text">` |
| `EmailField` | `<input type="email">` |
| `IntegerField` | `<input type="number">` |
| `DateField` | `<input type="text">` |
 | `ChoiceField` | `<select>` |
 | `BooleanField` | `<input type="checkbox">` |

完整列表参考官方文档：<https://docs.djangoproject.com/en/stable/ref/forms/fields/>

示例：
```python
class SurveyForm(forms.Form):
    age = forms.IntegerField(label='Your age')
    gender = forms.ChoiceField(label='Your gender', choices=[('M', 'Male'), ('F', 'Female')])
    agree = forms.BooleanField(label='Do you agree?')
```

## 3.表单部件
部件(widget)用于控制表单字段在HTML中的渲染方式。可以通过表单字段的`widget`参数指定部件。例如：

```python
class ContactForm(forms.Form):
    name = forms.CharField(label='Your name', widget=forms.TextInput(attrs={'class': 'form-control'}))
    email = forms.EmailField(label='Your email', widget=forms.EmailInput(attrs={'class': 'form-control'}))
    message = forms.CharField(label='Your message', widget=forms.Textarea(attrs={'class': 'form-control'}))
```

完整列表参考官方文档：<https://docs.djangoproject.com/en/stable/ref/forms/widgets/>

## 4.表单验证
表单的`is_valid()`方法会自动验证用户输入的数据。也可以自定义表单验证过程。

官方文档：<https://docs.djangoproject.com/en/stable/ref/forms/validation/>

### 4.1 字段验证
可以通过覆盖`clean_<fieldname>()`方法对特定字段进行自定义验证。该方法会在将原始数据转换为正确的类型并完成初步验证之后被调用，返回值将替换`cleaned_data`中已有的值。例如：

```python
class ContactForm(forms.Form):
    name = forms.CharField(label='Your name', max_length=100)
    email = forms.EmailField(label='Your email')

    def clean_name(self):
        name = self.cleaned_data['name']
        if len(name) < 3:
            raise forms.ValidationError('Name must be at least 3 characters long.')
        return name
```

### 4.2 表单级验证
可以通过覆盖`clean()`方法对多个字段进行联合验证。该方法将在所有字段完成验证之后被调用。例如：

```python
class ContactForm(forms.Form):
    name = forms.CharField(label='Your name', max_length=100)
    email = forms.EmailField(label='Your email')

    def clean(self):
        cleaned_data = super().clean()
        name = cleaned_data.get('name')
        email = cleaned_data.get('email')
        if name and email and name.lower() in email.lower():
            raise forms.ValidationError('Name should not be part of the email.')
```

## 5.模型表单
模型表单是基于Django模型的表单。`ModelForm`类可以根据模型定义自动生成表单字段并保存到数据库。

官方文档：<https://docs.djangoproject.com/en/stable/topics/forms/modelforms/>

注：admin应用为（admin.py中注册的）模型提供了增删改查功能，就是使用模型表单实现的。参见`django.contrib.admin.options.ModelAdmin.get_urls()`。

### 5.1 定义模型表单
假设模型`Contact`定义如下：

```python
from django.db import models

class Contact(models.Model):
    name = models.CharField(max_length=100)
    email = models.EmailField()
    message = models.TextField()
```

则对应的模型表单定义为：

```python
from django.forms import ModelForm
from .models import Contact

class ContactForm(ModelForm):
    class Meta:
        model = Contact
        fields = ['name', 'email', 'message']
```

这大致等价于1.1节中定义的`ContactForm`。

将`fields`属性设置为特殊值`'__all__'`表示使用模型中的所有字段。

### 5.2 在视图中使用模型表单
可以从POST数据创建一个模型表单实例，调用`save()`方法将新的模型对象保存到数据库。例如：

```python
from django.shortcuts import render, redirect
from .forms import ContactForm

def contact_view(request):
    if request.method == 'POST':
        form = ContactForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('success_page')
    else:
        form = ContactForm()
    return render(request, 'contact.html', {'form': form})
```

也可以通过`instance`参数指定已有模型对象来对其进行编辑。例如：

```python
>>> c = Contact.objects.get(pk=1)
>>> form = ContactForm(instance=c)
>>> submitted_form = ContactForm(request.POST, instance=c)
>>> submitted_form.save()
```

## 6.表单集
表单集用于处理多个表单实例，常用于批量编辑或创建多个对象。

官方文档：<https://docs.djangoproject.com/en/stable/topics/forms/formsets/>
