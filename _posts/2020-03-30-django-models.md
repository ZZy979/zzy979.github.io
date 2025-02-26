---
title: 【Django】模型
date: 2020-03-30 00:52 +0800
categories: [Django]
tags: [django]
---
## 1.模型
**模型**（model, MVC中的M）对应一个数据库表，相当于实体类。

官方文档：<https://docs.djangoproject.com/en/stable/topics/db/models/>

### 1.1 定义模型
应用目录下的models.py定义应用中使用的模型，每个模型都是`django.db.models.Model`的子类，类属性（`Field`类的实例）对应数据库中的字段，默认情况下属性名即为字段名，主键（id字段）将被自动添加。

例如：

```python
from django.db import models

class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')

class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

相当于创建了以下两个表：

```
Question(id, question_text, pub_date)
Choice(id, question_id, choice_text, votes)
```

### 1.2 修改模型
初次创建模型后，使用`makemigrations`命令创建模型更改记录（migration，对应数据库表结构的修改）：

```shell
python manage.py makemigrations polls
```
可使用`sqlmigrate`命令查看migration对应的SQL：

```shell
python manage.py sqlmigrate polls 0001
```

使用`migrate`命令将更改应用到数据库：

```shell
python manage.py migrate
```

**修改模型的三个步骤**：
* 修改models.py中的模型定义
* 运行`python manage.py makemigrations`，为更改创建migration
* 运行`python manage.py migrate`，将更改应用到数据库

### 1.3 CRUD
模型类本身充当DAO，模型类本身及其`objects`属性（`django.db.models.manager.Manager`类型的对象）提供CRUD接口，查询语法见第3节。

### 1.4 admin应用
官方文档：<https://docs.djangoproject.com/en/stable/ref/contrib/admin/>

Django默认提供了一个admin应用，可通过UI进行模型的增删改查操作（admin应用会根据每个模型的字段自动生成表单，如下图所示）。

![admin应用](/assets/images/django-models/admin应用.png)

创建管理员用户：

```shell
python manage.py createsuperuser
```

在admin.py中注册想要管理的模型：

```python
admin.site.register(Question)
```

访问 <http://127.0.0.1:8000/admin/> 并使用管理员用户登录后进入管理界面。

#### ModelAdmin选项
可通过继承`admin.ModelAdmin`类来自定义admin应用中某个模型的展示界面

```python
class BookAdmin(admin.ModelAdmin):
    list_display = ('title', 'author', 'publisher')
    list_filter = ['tag']
    search_fields = ['title']

admin.site.register(Book, BookAdmin)
```

列表界面常用选项：
* `list_display`：字段名列表，指定显示的字段（默认只有一列显示`__str__()`的结果）
* `list_filter`：字段名列表，指定右侧显示过滤器的字段
* `search_fields`：字段名列表，指定显示搜索框的字段
* `list_per_page`：指定列表界面每页显示的条目数，默认为100
* `ordering`：字段名列表，指定对象的排序方式，字段名前加`"-"`表示倒序排序，例如`['-pub_date', 'author']`表示先按`pub_date`倒序排序、再按`author`升序排序

详情界面常用选项：
* `fields`：字段名称列表，指定表单中包含的字段
* `exclude`：字段名称列表，指定表单中排除的字段
* `fieldsets`：控制表单中的字段分组显示
* `readonly_fields`：字段名列表，指定表单中只读的字段
* `raw_id_fields`：字段名列表，表单中的外键字段默认显示为`<select>`或`<select multiple>`标签（此时下拉列表需要查询全部关联对象，当数据量很大时会非常慢）；`raw_id_fields`中指定的（外键）字段将改为`<input>`标签，内容为逗号分隔的外键id，从而省去大量数据库查询

### 1.5 为模型提供初始数据
<https://docs.djangoproject.com/en/stable/howto/initial-data/>

### 1.6 模型继承
<https://docs.djangoproject.com/en/stable/topics/db/models/#model-inheritance>

* 如果只是使用父模型保存公共字段，父模型不会被单独使用，则使用抽象基类
* 如果要继承现有模型，每个模型有自己的数据库表，则使用多表继承
* 如果只是改变模型类的行为，而不改变模型字段，则使用代理模型

### 1.7 Meta类
<https://docs.djangoproject.com/en/stable/ref/models/options/>

每个模型类都可以定义一个名为`Meta`的内部类，用于指定元数据（例如数据库表名、人类可读名称、排序方式等）。

例如：

```python
from django.db import models

class Ox(models.Model):
    horn_length = models.IntegerField()

    class Meta:
        ordering = ["horn_length"]
        verbose_name_plural = "oxen"
```

常用选项：
* `db_table`：数据库表名
* `verbose_name`：人类可读名称
* `verbose_name_plural`：名称复数形式
* `ordering`：字段列表，指定查询时的默认排序方式，添加`"-"`前缀表示倒序排序，例如`ordering = ['-pub_date', 'author']`
* `permissions`：二元组列表，自定义权限，二元组格式为`(permission_code, human_readable_permission_name)`

## 2.字段
官方文档：
* <https://docs.djangoproject.com/en/stable/topics/db/models/#fields>
* <https://docs.djangoproject.com/en/stable/ref/models/fields/>

### 2.1 Field类通用属性
* `db_column`：数据库字段名，若未指定则使用field的名字
* `primary_key`：该字段是否为主键，如果一个模型未指定主键则Django将自动添加主键
* `blank`：如果为`True`则**表单**允许该字段为空，默认为`False`
* `null`：如果为`True`则将空值存储为`NULL`（**数据库**允许该字段为空），默认为`False`
* `default`：字段默认值，可以是一个值或`callable`对象
* `choices`：字段可选值，相当于枚举类型（见“choices和枚举类型”）
* `db_index`：是否为该字段创建索引，默认`False`
* `unique`：如果为`True`则该字段的值唯一，此时自动创建`UNIQUE`索引
* `verbose_name`：人类可读名称（对于普通字段作为第一个位置参数，对于外键字段作为关键字参数）

注：字段默认均为非空，即SQL类型后添加`NOT NULL`。要允许字符串类型的字段为空则将`null`设置为`True`即可；要允许整型、日期等类型的字段为空，即要想实现表单中留空则数据库中存储为`NULL`，则将`blank`和`null`均设置`True`。例如：

```python
foo = CharField(max_length=32)  # foo VARCHAR(32) NOT NULL
bar = CharField(max_length=32, null=True)  # bar VARCHAR(32) NULL
```

### 2.2 常用字段类型

| Field子类 | 等价SQL类型(MySQL) |
| --- | --- |
| `CharField(max_length=n)` | `VARCHAR(n)` |
| `TextField()` | `LONGTEXT` |
| `IntegerField()` | `INT` |
| `BigIntegerField()` | `BIGINT` |
| `AutoField()` | `INT AUTO_INCREMENT` |
| `DecimalField(max_digits=m, decimal_places=n)` | `DOUBLE(m, n)` |
| `DateField(auto_now=False, auto_now_add=False)` | `DATE` |
| `DateTimeField(auto_now=False, auto_now_add=False)` | `DATETIME` |

### 2.3 外键
官方文档：
* <https://docs.djangoproject.com/en/stable/topics/db/models/#relationships>
* <https://docs.djangoproject.com/en/stable/ref/models/fields/#module-django.db.models.fields.related>

查询语法：<https://docs.djangoproject.com/en/stable/topics/db/queries/#related-objects>

#### 2.3.1 多对一

```python
ForeignKey(to, on_delete)
```

例如，一个`Question`对应多个`Choice`，则在`Choice`类中定义：

```python
question = models.ForeignKey(Question, on_delete=models.CASCADE)
```

* `choice.question`是与`choice`关联的`Question`对象
* `choice.question_id`是与`choice`关联的`Question`的id
* `question.choice_set.all()`是与`question`关联的所有`Choice`对象

可通过`related_name`属性覆盖`"choice_set"`名称，设置为`'+'`则不创建反向关联。

`on_delete`指定当外键关联的对象被删除时执行的动作，可选值（都定义在`django.db.models`模块）：
* `CASCADE`：级联删除
* `PROTECT`：阻止删除，抛出`ProtectedError`异常
* `SET_NULL`：置为`NULL`，要求外键字段的`null=True`
* `SET_DEFAULT`：置为默认值，外键字段必须指定了`default`属性
* `SET(value)`：置为指定值，如果`value`是函数则使用函数返回值

递归关系（一个模型和自己有多对一的关系）：

```python
models.ForeignKey('self', on_delete=models.CASCADE)
```

#### 2.3.2 多对多

```python
ManyToManyField(to)
```

例如：一个`Author`对应多个`Paper`，一个`Paper`也对应多个`Author`，则在`Author`类中定义：

```python
papers = models.ManyToManyField(Paper)
```

* `author.papers.all()`是与`author`关联的所有`Paper`对象
* `paper.author_set.all()`是与`paper`关联的所有`Author`对象

#### 2.3.3 一对一

```python
OneToOneField(to)
```

例如：`User`和`Reader`一一对应，则在`Reader`类中定义：

```python
user = models.OneToOneField(User)
```

* `reader.user`是与`reader`关联的`User`对象
* `user.reader`是与`user`关联的`Reader`对象

### 2.4 choices和枚举类型
<https://docs.djangoproject.com/en/stable/ref/models/fields/#choices>

`Field.choices`属性是二元组`(value, name)`的列表，二元组的第一项是字段的真实值，第二项是人类可读名称，例如：

```python
class Student(models.Model):
    YEAR_IN_SCHOOL_CHOICES = [
        ('FR', 'Freshman'),
        ('SO', 'Sophomore'),
        ('JR', 'Junior'),
        ('SR', 'Senior'),
        ('GR', 'Graduate'),
    ]
    year_in_school = models.CharField(max_length=2, choices=YEAR_IN_SCHOOL_CHOICES, default='FR')
```

对于每一个设置了`choices`的字段，Django会自动为模型添加一个`get_foo_display()`方法，用于获取该字段当前值的人类可读名称：

```python
>>> s = Student(year_in_school='GR')
>>> s.get_year_in_school_display()
'Graduate'
```

另外Django还专门为`choices`提供了枚举类型（内部使用Python的`enum`模块实现）：

```python
class Student(models.Model):
    class YearInSchool(models.TextChoices):
        FRESHMAN = 'FR', 'Freshman'
        SOPHOMORE = 'SO', 'Sophomore'
        JUNIOR = 'JR', 'Junior'
        SENIOR = 'SR', 'Senior'
        GRADUATE = 'GR', 'Graduate'

    year_in_school = models.CharField(max_length=2, choices=YearInSchool.choices, default=YearInSchool.FRESHMAN)
```

枚举类的`values`, `labels`, `choices`, `names`属性分别为值列表、标签（人类可读名称）列表、二元组列表（同上）和枚举成员名称列表，枚举成员的`label`属性为对应的标签

```python
class Suit(models.IntegerChoices):
    SPADE = 1, 'Spade'
    HEART = 2, 'Heart'
    CLUB = 3, 'Club'
    DIAMOND = 4, 'Diamond'
```

```python
>>> Suit.values
[1, 2, 3, 4]
>>> Suit.labels
['Spade', 'Heart', 'Club', 'Diamond']
>>> Suit.choices
[(1, 'Spade'), (2, 'Heart'), (3, 'Club'), (4, 'Diamond')]
>>> Suit.names
['SPADE', 'HEART', 'CLUB', 'DIAMOND']
>>> suit = Suit.SPADE
>>> suit.label
'Spade'
```

## 3.查询
官方文档：<https://docs.djangoproject.com/en/stable/topics/db/queries/>

下面以`Question`和`Choice`两个模型为例（`q`表示`Question`类的对象，`c`表示`Choice`类的对象）。

### 3.1 创建对象

```python
q = Question(question_text="What's new?", pub_date=datetime.now())
q.save()
```

捷径：`create()`，以上两行代码等价于

```python
q = Question.objects.create(question_text="What's new?", pub_date=datetime.now())
```

批量创建：`bulk_create(objs, batch_size=None)`

### 3.2 更新对象
假设`q`已被保存到数据库

```python
q.question_text = "What's up?"
q.save()
```

* `q.id`为空时`q.save()`执行`INSERT`语句；`q.id`非空时`q.save()`执行`UPDATE`语句。
* 设置外键的值：`c.question = q`

### 3.3 删除对象

```python
q.delete()
```

### 3.4 查询对象
查询的本质是通过模型类的`Manager`构造一个`QuerySet`。`QuerySet`相当于一个`SELECT`语句，每个模型类都有一个默认的`Manager`，即`objects`属性（注意是类属性，通过模型类访问，而不是模型类的对象）。

QuerySet API参考：<https://docs.djangoproject.com/en/stable/ref/models/querysets/>

查询全部对象：`Question.objects.all()`

查询记录数量：`Question.objects.count()`

条件过滤：
* `filter(**kwargs)`返回一个新的`QuerySet`，包含满足指定条件的对象
* `exclude(**kwargs)`返回一个新的`QuerySet`，包含不满足指定条件的对象
* `get(**kwargs)`返回满足条件的单个对象，如果不存在则产生`Question.DoesNotExist`异常

注意：`QuerySet`是惰性求值的，构造的过程不执行任何数据库查询，只有最终获取对象时才执行查询。

`kwargs`格式：`field__lookuptype=value`

完整语法：<https://docs.djangoproject.com/en/stable/ref/models/querysets/#field-lookups>

举例：

（1）基本查询

①查询主键(id)为1的问题

```python
Question.objects.filter(id=1)
Question.objects.get(pk=1)
```

等价于

```sql
SELECT * FROM Question WHERE id = 1
```

②查询票数小于1的选项

```python
Choice.objects.filter(votes__lt=1)
```

等价于

```sql
SELECT * FROM Choice WHERE votes < 1
```

③查询以 "What" 开头的问题

```python
Question.objects.filter(question_text__startswith='What')
```

等价于

```sql
SELECT * FROM Question WHERE question_text LIKE 'What%'
```

④查询包含 "up" 的问题

```python
Question.objects.filter(question_text__contains='up')
```

等价于

```sql
SELECT * FROM Question WHERE question_text LIKE '%up%'
```

⑤查询2019年发布的问题

```python
Question.objects.get(pub_date__year=2019)
```

等价于

```sql
SELECT * FROM Question WHERE YEAR(pub_date) = 2019
```

（2）连接查询

`Question`和`Choice`是一对多的关系，`q.choice_set`是一个包含`q`关联的所有`Choice`的`QuerySet`，`c.question`是`c`关联的`Question`。

①查询`q`关联的所有选项

```python
q.choice_set.all()
```

等价于

```sql
SELECT Choice.*
FROM Choice INNER JOIN Question
ON Choice.question_id = Question.id
WHERE Question.id = q.id
```

②为`q`创建一个名为 "Not much" 、票数为0的选项

```python
c = q.choice_set.create(choice_text='Not much', votes=0)
```

等价于

```sql
INSERT INTO Choice VALUES (NULL, 'Not much', 0, q.id)
```

③查询`c`关联的问题

```python
c.question
```

等价于

```sql
SELECT * FROM Question WHERE id = c.question_id
```

④查询2019年发布的问题的所有选项

```python
Choice.objects.filter(question__pub_date__year=2019)
```

等价于

```sql
SELECT Choice.*
FROM Choice INNER JOIN Question
ON Choice.question_id = Question.id
WHERE YEAR(Question.pub_date) = 2019
```

⑤查询`q`关联的以 "Just hacking" 开头的选项

```python
c = q.choice_set.filter(choice_text__startswith='Just hacking')
```

等价于

```sql
SELECT * FROM Choice WHERE Choice.question_id = q.id AND Choice.choice_text LIKE 'Just hacking%'
```

## 4.使用现有数据库表
官方文档：<https://docs.djangoproject.com/en/stable/ref/models/options/#managed>

在模型的内部类`Meta`中指定`managed = False`，则Django不对该模型类进行创建表等操作，此时应明确定义主键字段。

```python
class Foo(models.Model):
    id = models.AutoField(primary_key=True)
    # ...

    class Meta:
        managed = False
```

在单元测试中需要手动创建表

```python
from django.db import connections
from django.test import TestCase

from foo.models import Foo

class FooTests(TestCase):
    @classmethod
    def setUpClass(cls):
        with connection.schema_editor() as editor:
            editor.create_model(Foo)
        super().setUpClass()
```

如果要使用其他数据库则将`connection`改为`connections['xxx']`，`xxx`为`DATABASES`设置中定义的数据库别名。
