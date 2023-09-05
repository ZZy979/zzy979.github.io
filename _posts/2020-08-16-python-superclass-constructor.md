---
title: 关于超类构造函数的问题
date: 2020-08-16 21:27:42 +0800
categories: [Python]
tags: [python]
---
在Python中，构造函数可以被继承，但不能重载。 如果子类没有定义构造函数，则自动继承超类的构造函数； 如果子类定义了构造函数，则应当调用超类的构造函数，但不必是第一行代码，否则将缺失超类构造函数中设置的属性。

例如：

```python
class A:

    def __init__(self, x=0):
        self.x = x


class B(A):

    def __init__(self, x=0, y=1):
        super().__init__(x)
        self.y = y


class C(B):
    pass
```

其中，`C`类继承了`B`类的构造函数`__init__(x=0, y=1)`；如果`B`类的构造函数中不调用`A`类的构造函数，则`B`类的对象无法访问`x`属性

在Java中，构造函数不能被继承，但可以重载。 如果子类没有定义构造器，则编译器将自动添加一个无参构造器； 如果子类构造器中没有显式调用超类构造器，则默认调用超类的无参构造器，如果超类没有无参构造器，则编译器将报错；如果子类构造器显式调用了超类构造器，则必须是第一行代码。

例如：

```java
class A {
   private int x;

   public A(int x) {
      this.x = x;
   }

   public A() {
      this(0);
   }

}

class B extends A {
   private int y;

   public B(int x, int y) {
      super(x);
      this.y = y;
   }

   public B(int y) {
      // 默认调用super()
      this.y = y;
   }

   public B() {
      this(1);
   }

}

class C extends B {
} 
```

其中，`C`类只有一个编译器自动生成的无参构造器，并且自动调用`B`类的无参构造器。

如果`A`类没有无参构造器，则`B(y)`构造器必须显式调用`A`类构造器，且必须是第一行代码，否则编译器将报错。

如果`B`类没有无参构造器，则`C`类必须定义至少一个构造器并显式调用`B`类构造器，否则编译器将报错。
