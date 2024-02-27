---
title: 《Python基础教程》笔记 第7章 再谈抽象
date: 2024-01-24 22:49:50 +0800
categories: [Python, Beginning Python]
tags: [python, object-oriented programming, polymorphism, inheritance, class, attribute, method, multi-inheritance, mro]
---
创建自己的对象（和类）是Python非常核心的概念，以至于Python被称为**面向对象**的语言。在本章中，你将学习如何创建对象，以及多态、封装、方法、属性、超类和继承。

## 7.1 对象魔法
在**面向对象编程**(object-oriented programming)中，术语**对象**(object)大致意味着一系列数据（属性）以及一组访问和操作这些数据的方法。使用对象代替全局变量和函数最重要的好处：
* **多态**(polymorphism)：可以对不同类型的对象执行相同的操作。
* **封装**(encapsulation)：对外部隐藏对象工作原理的细节。
* **继承**(inheritance)：可以基于通用类创建专用类。

### 7.1.1 多态
术语**多态**意味着即使不知道变量指向的是哪种对象，仍然能够对其执行操作，且操作的行为随对象的类型而异。

例如，要统计一个元素在序列中的出现次数，可以编写一个函数，根据对象的类型选择对应的实现。

```python
# Don't do it like this ...
def count(s, x):
    if isinstance(s, list):
        return list_count(s, x)
    elif isinstance(s, str):
        return str_count(s, x)
    elif isinstance(s, tuple):
        return tuple_count(s, x)
    # ...
```

显然，这种方式既不灵活也不切实际。

更好的方法是让对象自己去处理这种操作。这正是多态（某种程度上还有封装）的用武之地。

#### 多态和方法
你收到一个对象，并不知道它是如何实现的——它可能是众多“形态”中的一种。你只知道可以对其执行某种操作（例如统计元素的出现次数），这就足够了。

```python
s.count(x)
```

像这样绑定到对象属性的函数称为**方法**(methods)。你已经见过字符串、列表和字典方法，其实也见过多态。

```python
>>> 'abc'.count('a')
1
>>> [1, 2, 'a'].count('a')
1
```

如果有一个变量`x`，你无需知道它是字符串还是列表就能调用`count()`方法。

下面来做个实验。标准库模块`random`包含一个名为`choice()`的函数，它从序列中随机选择一个元素。下面使用这个函数给变量提供一个值。

```python
>>> from random import choice
>>> x = choice(['Hello, world!', [1, 2, 'e', 'e', 4]])
>>> x.count('e')
2
```

变量`x`可能是字符串**或**列表。但关键在于你无需检查，只要`x`有一个名为`count()`、接受一个字符参数并返回整数的方法即可。对于包含该方法的自定义对象，也可以像字符串和列表一样使用。

注：在这个例子中，“多态”是指“一个函数可用于多种类型的参数”。在C++和Java中要实现类似的功能，要么使用泛型，要么使用接口或抽象类参数。而在Python中，使用普通函数定义即可。例如：

```python
def count(s, x):
    return s.count(x)
```

这个函数可用于任何具有名为`count`且接受一个参数的方法的对象（参数的类型甚至可以不同），否则会引发异常。这大致等价于C++的函数模板：

```cpp
template<class S, class T>
int count(const S& s, const T& x) {
    return s.count(x);
}
```

或者传基类指针/引用参数：

```cpp
template<class T>
class Seq {
public:
    virtual int count(const T& x) = 0;
};

template<class T>
int count(const Seq<T>& s, const T& x) {
    return s.count(x);
}
```

不同之处在于，如果对象`s`没有`count()`方法，C++会在编译时报错，而Python在运行时报错。

#### 多态的多种形式
每当需要对对象“执行某种操作”而无需知道对象的确切类型时，都是多态在起作用。这不仅适用于方法，内置运算符和函数也大量使用了多态。例如：

```python
>>> 1 + 2
3
>>> 'Fish' + 'license'
'Fishlicense'
```

加法运算符(`+`)既可用于数字，也可用于字符串（以及其他类型的序列）。为证明这一点，创建一个将两个对象相加的函数`add()`（等价于`operator`模块中的`add()`函数）：

```python
def add(x, y):
    return x + y
```

该函数可用于多种类型的参数。

```python
>>> add(1, 2)
3
>>> add('Fish', 'license')
'Fishlicense'
```

重点在于参数可以是**任何支持加法**的对象。

如果想编写一个打印对象长度信息的函数，唯一的要求是对象**有长度**（可对其调用`len()`）。

```python
def length_message(x):
    print('The length of', repr(x), 'is', len(x))
```

```python
>>> length_message('Fnord')
The length of 'Fnord' is 5
>>> length_message([1, 2, 3])
The length of [1, 2, 3] is 3
>>> length_message(42)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 2, in length_message
TypeError: object of type 'int' has no len()
```

该函数还使用了`repr()`，而`repr()`是多态的集大成者之一——可用于任何对象。

很多函数和运算符都是多态的，你编写的大多数函数也可能如此，即便你不是有意为之。事实上，唯一破坏多态的办法是使用`isinstance()`和`issubclass()`等函数做显式类型检查，应该尽可能避免这种方式。**重要的是对象是否按你希望的方式工作（是否有特定的方法和属性），而非它是否是正确的类型。**

注意：这里讨论的多态形式是Python编程方式的核心，有时称为**鸭子类型**(duck typing)。这个术语源自如下说法：“如果走起来像鸭子，叫起来像鸭子，那么它就是鸭子。”详见 <https://en.wikipedia.org/wiki/Duck_typing> 和官方文档[duck-typing](https://docs.python.org/3/glossary.html#term-duck-typing)。

注：Python的“多态”相当于C++和Java的泛型+多态。这两节的例子本质上都是泛型，而7.2.6节的例子才是真正的多态。由于Python的函数不需要声明参数类型，因此本身就是泛型的，从而将泛型与多态合二为一了。

### 7.1.2 封装
**封装**是指向外部隐藏不必要的细节。多态和封装这两个概念很像，因为它们都是**抽象的原则**。它们都能帮助你处理程序的组成部分，而无需关心不必要的细节，就像函数一样。

但封装不同于多态。多态让你无需知道对象的类型就能调用其方法，而封装让你无需知道对象的构造就能使用它。例如：

```python
>>> r = ClosedObject()
>>> r.set_name('Sir Robin')
>>> r.get_name()
'Sir Robin'
```

对象的**状态**(state)由其**属性**(attribute)描述，对象的方法可能修改这些属性。因此，对象就像是将一系列函数（方法）组合起来，使其能够访问一些变量（属性），属性能够在两次函数调用之间保持存储的值。

7.2.4节将更详细地讨论Python的封装机制。

### 7.1.3 继承
继承是另一种偷懒（褒义）的方式。如果已经有了一个类，想要创建一个很相似的类（可能只是添加了几个方法）。

例如，已经有了一个`Shape`类，它知道如何将自己绘制在屏幕上。现在想创建一个`Rectangle`类，它不仅知道如何将自己绘制在屏幕上，还会计算其面积。可以让`Rectangle` **继承** `Shape`，使得对`Rectangle`对象调用`draw()`方法时，将自动调用`Shape`类的这个方法（详见7.2.6节）。

## 7.2 类
### 7.2.1 类到底是什么
前面反复提到了**类**(class)，并将其用作**类型**(type)的同义词。从很多方面来说，这正是类的定义——一种对象。每个对象都属于特定的类，称为该类的**实例**(instance)。

例如，如果在窗外看到一只鸟，这只鸟就是“鸟类”的一个实例。“鸟类”是一个非常通用（抽象）的类，它有多个子类：你看到的那只鸟可能属于子类“云雀”。可以将“鸟类”视为所有鸟组成的集合，而“云雀”是其一个子集。“云雀”是“鸟类”的**子类**，而“鸟类”是“云雀”的**超类**。

注意：在英语日常交谈中，使用复数来表示类，例如 "birds" 和 "larks" 。在Python中，习惯上使用单数并将首字母大写，例如`Bird`和`Lark`（但内置类型都是小写，如`int`、`list`和`str`）。

在面向对象编程中，子类关系有很重要的含义。因为类是由其支持的方法定义的，类的所有实例都有该类的所有方法，因此子类的所有实例也必须都有这些方法。从而，定义子类只是定义**更多**方法或者**覆盖**(override)已有方法。

例如，`Bird`类支持`fly()`方法，而`Penguin`（`Bird`的子类）可能添加`eat_fish()`方法。创建`Penguin`类时，可能想覆盖超类的`fly()`方法，该方法应该什么都不做或者引发异常，因为企鹅不会飞。

### 7.2.2 创建自己的类
使用`class`语句定义一个类。例如：

[自定义类示例](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch07/person.py)

注意：在交互式解释器中定义类时，中间不能包含空行。

这个示例定义了一个名为`Person`的类，包含三个方法定义（类似于函数定义，只是位于`class`语句内）。`class`语句创建独立的命名空间（见7.2.5节）。下面创建几个实例看看。

```python
>>> foo = Person()
>>> bar = Person()
>>> foo.set_name('Luke Skywalker')
>>> bar.set_name('Anakin Skywalker')
>>> foo.greet()
Hello, world! I'm Luke Skywalker.
>>> bar.greet()
Hello, world! I'm Anakin Skywalker.
```

方法的`self`参数指向对象本身（类似于C++的`this`指针）。对`foo`调用`set_name()`和`greet()`方法时，`foo`本身被自动传递给第一个参数，习惯上将其命名为`self`。

方法通过`self`参数访问对象的属性。属性也可以在外部访问。

```python
>>> foo.name
'Luke Skywalker'
>>> bar.name = 'Yoda'
>>> bar.greet()
Hello, world! I'm Yoda.
```

提示：如果`foo`是`Person`的实例，可以将`foo.greet()`视为`Person.greet(foo)`的简写，但后者的多态性更低。另见9.2.2节。

### 7.2.3 属性、函数和方法
**属性**(attribute)是关联到对象的值，使用`object.attribute`的形式访问。

注：attribute和property是两个Python术语。大部分博客将前者翻译为“特性”、后者翻译为“属性”，而官方文档将前者翻译为“[属性](https://docs.python.org/zh-cn/3/glossary.html#term-attribute)”、后者翻译为“[特征属性](https://docs.python.org/zh-cn/3/library/functions.html#property)”。这里与官方文档保持一致。

实际上，方法和函数的区别正是`self`参数。方法（更准确地说是**绑定**方法(bound method)）的第一个参数绑定到它所属的实例，因此无需提供这个参数。当然也可以将属性绑定到一个普通函数，但这样就没有特殊的`self`参数了。

```python
>>> class C:
...     def foo(self):
...         print('I have a self!')
...
>>> def bar():
...     print("I don't...")
...
>>> c = C()
>>> c.foo()
I have a self!
>>> c.bar = bar
>>> c.bar()
I don't...
```

注意，有没有`self`参数并不取决于是否使用`instance.method()`这种方式调用方法。

注：而是取决于被调用的是绑定方法还是普通函数。调用绑定方法时，会将实例插入到参数列表前（即传递给`self`参数）。详见官方文档[method binding](https://docs.python.org/3/reference/datamodel.html#method-binding)。

> When an instance method object is called, the underlying function is called, inserting the class instance in front of the argument list.

```python
>>> c.foo
<bound method C.foo of <__main__.C object at 0x0000014B734E7FD0>>
>>> c.bar
<function bar at 0x0000014B734F79C0>
```

实际上，完全可以让另一个变量引用实例方法。

```python
>>> class Bird:
...     song = 'Squaawk!'
...     def sing(self):
...         print(self.song)
...
>>> bird = Bird()
>>> bird.sing()
Squaawk!
>>> birdsong = bird.sing
>>> birdsong()
Squaawk!
>>> birdsong
<bound method Bird.sing of <__main__.Bird object at 0x0000014B734FA490>>
```

尽管`birdsong()`看起来很像函数调用，但变量`birdsong`引用的是绑定方法`bird.sing`，这意味着它仍然能够访问`self`参数（即仍然绑定到实例`bird`）。

注：这类似于Java的方法引用`instance::method`。

### 7.2.4 再谈私有
默认情况下，可以从外部访问对象的属性。例如7.2.2节最后的示例。

有些程序员认为这违反了封装原则，而应该对外部**完全隐藏**对象的状态（即不可访问）。但为何要向外部隐藏属性？毕竟，如果直接使用`Person`类的`name`属性，就不需要`set_name()`和`get_name()`方法了。

关键是其他程序员可能不知道（也不应该知道）对象内部的情况。例如，`set_name()`方法可能包含其他操作（例如向管理员发送电子邮件），而直接修改`name`属性则什么都不会发生。为避免这类问题，可以使用**私有**(private)属性。私有属性不能从对象外部访问，而只能通过**访问器**(accessor)方法（例如`get_name()`和`set_name()`）来访问。

注意：第9章将介绍**特征属性**(property)，这是一种功能强大的访问器替代品。

Python并不直接支持私有属性，但可以通过一种技巧达到类似的效果。要让方法或属性成为私有的（不能从外部访问），只需让其名称**以两个下划线开头**。

```python
class Secretive:

    def __inaccessible(self):
        print("Bet you can't see me ...")

    def accessible(self):
        print('The secret message is:')
        self.__inaccessible()
```

```python
>>> s = Secretive()
>>> s.__inaccessible()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'Secretive' object has no attribute '__inaccessible'. Did you mean: 'accessible'?
>>> s.accessible()
The secret message is:
Bet you can't see me ...
```

然而，在类定义中，所有以双下划线开头的名字都会被“翻译”为在开头加上一个下划线和类名。只要知道这种幕后操作，就能从类外部访问私有方法，然而不应该这样做。

```python
>>> s._Secretive__inaccessible()
Bet you can't see me ...
```

注：这种机制称为**私有名称转换**(private name mangling)，见官方文档[Private name mangling](https://docs.python.org/3/reference/expressions.html#private-name-mangling)。

总之，无法确保别人不会访问对象的私有方法和属性，但这种名称修改方式是一个强烈的信号，表示**不应该**这样做。

如果不希望名称被修改，又想发出不要从外部访问的信号，可以使用单个下划线开头。这只是一种约定，但也有实际作用。例如，星号导入(`from module import *`)不会导入以一个下划线开头的名字。

注：有些语言支持多种成员变量可见性。Python没有提供这样的支持，不过从某种程度上说，以一个或两个下划线开头相当于两种不同的私有程度。

### 7.2.5 类命名空间
在`class`语句中的所有代码都是在一个特殊的命名空间中执行的——**类命名空间**(class namespace)。类的所有成员都可以访问这个命名空间。类定义其实就是要执行的代码段。例如，在类定义中不限于使用`def`语句。

```python
>>> class C:
...     print('Class C being defined...')
...
Class C being defined...
>>>
```

[类命名空间示例](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch07/member_counter.py)

```python
>>> m1 = MemberCounter()
>>> m1.init()
>>> MemberCounter.members
1
>>> m2 = MemberCounter()
>>> m2.init()
>>> MemberCounter.members
2
```

上述代码在类作用域内定义了一个变量（类属性），所有的实例都可以访问它。注意，这里手动调用`init()`来初始化实例，第9章将自动化这一过程（使用构造函数）。

每个实例都可以访问类作用域内的变量，就像方法一样。

```python
>>> m1.members
2
>>> m2.members
2
```

注意：Python的类属性并不是其他语言中的静态成员。如果在实例中给`members`属性赋值：

```python
>>> m1.members = 'Two'
>>> m1.members
'Two'
>>> m2.members
2
```

新值被写入`m1`的实例属性中，遮盖了类属性。这类似于第6章“遮盖问题”所讨论的函数局部变量与全局变量的关系。

### 7.2.6 指定超类
子类扩展了超类的定义。要指定超类，将其写在`class`语句中类名后面的圆括号中。

[超类示例](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch07/filters.py)

`Filter`是一个过滤序列的通用类，实际上它不会过滤掉任何东西。

```python
>>> f = Filter()
>>> f.init()
>>> f.filter([1, 2, 3])
[1, 2, 3]
```

`Filter`类的用处在于作为其他类的超类，例如从序列中过滤掉 "SPAM" 的`SPAMFilter`。

```python
>>> s = SPAMFilter()
>>> s.init()
>>> s.filter(['SPAM', 'SPAM', 'SPAM', 'SPAM', 'eggs', 'bacon', 'SPAM'])
['eggs', 'bacon']
```

注意`SPAMFilter`的定义中有两个要点：
* 以提供新定义的方式**覆盖**了`Filter`类的`init()`方法。
* 直接从`Filter`类**继承**了`filter()`方法的定义，因此无需重新编写。

### 7.2.7 深入探讨继承
要确定一个类是否是另一个类的子类，可以使用内置函数`issubclass()`。

```python
>>> issubclass(SPAMFilter, Filter)
True
>>> issubclass(Filter, SPAMFilter)
False
```

要知道一个类的超类（基类），可以访问其特殊属性`__bases__`。

```python
>>> SPAMFilter.__bases__
(<class '__main__.Filter'>,)
>>> Filter.__bases__
(<class 'object'>,)
```

要确定一个对象是否是一个类（或其子类）的实例，可以使用`isinstance()`。

```python
>>> isinstance(s, SPAMFilter)
True
>>> isinstance(s, Filter)
True
>>> isinstance(s, str)
False
>>> isinstance('SPAM', str)
True
```

可以看到，`s`是`SPAMFilter`类的直接实例，也是`Filter`类的间接实例，因为`SPAMFilter`是`Filter`的子类。换句话说，所有的`SPAMFilter`对象都是`Filter`对象。

如果想知道一个对象属于哪个类，可以使用`__class__`属性。

```python
>>> f.__class__
<class '__main__.Filter'>
>>> s.__class__
<class '__main__.SPAMFilter'>
>>> 'abc'.__class__
<class 'str'>
>>> str.__class__
<class 'type'>
```

注意：在Python 3中，也可以使用内置函数`type()`获取对象的类型。另见9.1节。

### 7.2.8 多个超类
属性`__bases__`的复数形式暗示了一个类的超类可能有多个。为了说明，下面创建几个类。

[多个超类示例](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch07/talking_calculator.py)

子类`TalkingCalculator`本身什么都不做，其所有行为都是从超类继承的。

```python
>>> tc = TalkingCalculator()
>>> tc.calculate('1 + 2 * 3')
>>> tc.talk()
Hi, my value is 7
```

这叫做**多重继承**(multiple inheritance)，是一个功能强大的工具。然而，除非万不得已，否则应避免使用多重继承，因为某些情况下会导致不可预见的麻烦。

使用多重继承时，务必注意一点：如果多个超类有同名方法，必须注意`class`语句中超类的顺序，因为位于前面的类的方法会覆盖位于后面的类的方法。因此，在前面的示例中，如果`Calculator`类也有`talk()`方法，它将覆盖`Talker`类的`talk()`方法（使其不可访问）。而如果反转超类的顺序，即`class TalkingCalculator(Talker, Calculator)`，会使得`Talker`类的`talk()`方法是可访问的。

如果多个超类拥有公共的超类（例如`D`继承`B`和`C`，而`B`和`C`继承`A`，叫做**菱形继承**），查找特定方法或属性时访问超类的顺序称为**方法解析顺序**(method resolution order, MRO)，使用的算法非常复杂。所幸它运行得很好，因此无需担心。

注：详见[Python多继承的MRO和构造函数问题]({% post_url 2020-10-15-python-multi-inheritance-mro-constructor %})。

### 7.2.9 接口和内省
**接口**(interface)这一概念与多态有关。处理多态对象时，你只关心接口（协议(protocol)）——对外暴露的方法和属性。在Python中，不用显式地指定对象必须包含哪些方法才能用作参数，而是假定对象能够完成你要求它完成的任务（即前面提到的“鸭子类型”）。如果不能，程序将失败。

然而，也可以使用`hasattr()`函数检查对象是否有特定的方法或属性。

```python
>>> hasattr(tc, 'talk')
True
>>> hasattr(tc, 'fnord')
False
```

`getattr()`用于获取对象的属性，如果不存在则返回指定的默认值或引发`AttributeError`。

```python
>>> getattr(tc, 'talk')()
Hi, my value is 7
>>> print(getattr(tc, 'fnord', None))
None
```

`setattr()`用于设置对象的属性。

```python
>>> setattr(tc, 'name', 'Mr. Gumby')  # equivalent to tc.name = 'Mr. Gumby'
>>> tc.name
'Mr. Gumby'
```

注：使用这三个函数可以很容易地实现**反射**功能。在Python中，使用`type()`、`isinstance()`、`hasattr()`等方式在运行时获取类型信息的能力叫做**内省**(introspection)。

要查看对象中存储的所有值，可以访问其`__dict__`属性。如果要确定对象是由什么组成的，可以查看`inspect`模块。

### 7.2.10 抽象基类
在历史上的大部分时间，Python几乎都只依赖于鸭子类型（假设对象都能完成其工作，偶尔使用`hasattr()`）。很多其他语言（例如Java和Go）都采用显式指定接口的思想。最终，Python通过引入`abc`模块提供了官方解决方案。这个模块为所谓的**抽象基类**(abstract base class)提供了支持。一般而言，抽象类是不能（至少不应该）实例化的类，其职责是定义子类应该实现的一组抽象方法。下面是一个简单的示例：

```python
from abc import ABC, abstractmethod

class Talker(ABC):
    @abstractmethod
    def talk(self):
        pass
```

形如`@xxx`的叫做**装饰器**(decorator)，将在第9章详细介绍。这里使用`@abstractmethod`将方法标记为抽象的——必须在子类中实现。

抽象类（即包含抽象方法的类）不能实例化。

```python
>>> Talker()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: Can't instantiate abstract class Talker with abstract method talk
```

下面创建一个子类：

```python
class Knigget(Talker):
    def talk(self):
        print('Ni!')
```

这是抽象类的主要用途，或许也是`isinstance()`的唯一合理用途：如果先检查给定的实例确实是`Talker`对象，就能相信它实现了`talk()`方法。

```python
>>> k = Knigget()
>>> isinstance(k, Talker)
True
>>> k.talk()
Ni!
```

本着鸭子类型的精神，只要实现了`talk()`方法，即便不是`Talker`的子类，**应该**依然能够通过类型检查。因此创建另一个类。

```python
class Herring:
    def talk(self):
        print('Blub.')
```

然而，它并不是`Talker`对象。

```python
>>> h = Herring()
>>> isinstance(h, Talker)
False
```

为此，可以将`Herring` **注册**为`Talker`的“虚拟子类”，这样所有的`Herring`对象都将被视为`Talker`对象。

```python
>>> Talker.register(Herring)
<class '__main__.Herring'>
>>> isinstance(h, Talker)
True
>>> issubclass(Herring, Talker)
True
```

然而，这种做法存在一个缺点：直接继承抽象类提供的保障没有了。

```python
class Clam:
    pass
```

```python
>>> Talker.register(Clam)
<class '__main__.Clam'>
>>> issubclass(Clam, Talker)
True
>>> c = Clam()
>>> isinstance(c, Talker)
True
>>> c.talk()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'Clam' object has no attribute 'talk'
```

换句话说，应该将`isinstance()`返回`True`视为一种**意图**(intent)表达。在这里，`Clam` **旨在**(intended to)成为`Talker`。本着鸭子类型的精神，我们相信它能完成工作（调用`talk()`方法），但可悲的是它失败了。

标准库提供了很多有用的抽象基类，例如[collections.abc](https://docs.python.org/3/library/collections.abc.html#module-collections.abc)模块。

## 7.3 关于面向对象设计的一些思考
* 将相关的东西放在一起。如果一个函数操作一个全局变量，最好将它们作为一个类的属性和方法。
* 不要让对象之间过于亲密。方法应该只关心自己实例的属性。让其他实例管理自己的状态。
* 慎用继承，尤其是多重继承。继承有时很有用，但有些情况下可能带来不必要的复杂性。要正确地使用多继承很难，debug更难。
* 保持简单。让方法短小。根据经验，应确保大多数方法都能在30秒内读完（并理解）。对于其他方法，尽可能将其篇幅控制在一页或一屏内。
