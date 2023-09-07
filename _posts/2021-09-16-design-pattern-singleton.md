---
title: 【设计模式】1.单例模式
date: 2021-09-16 14:29:13 +0800
categories: [Design Pattern]
tags: [design pattern]
---
## 定义
单例模式(singleton pattern)确保一个类只有一个实例，并提供该实例的全局访问点

使用一个私有构造函数、一个私有静态变量以及一个公有静态函数来实现

私有构造函数保证了不能通过构造函数来创建对象实例，只能通过公有静态函数返回唯一的私有静态变量

## 实现
### 线程不安全
```java
public class Singleton {
    private static Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null)
            instance = new Singleton();
        return instance;
    }
}
```

延迟实例化的好处：节约资源

存在的问题：如果两个线程同时进入`if (instance == null)`，则会有多个线程执行实例化语句

### 线程安全1
对`getInstance()`方法加锁

```java
public class Singleton {
    private static Singleton instance;

    private Singleton() {}

    public static synchronized Singleton getInstance() {
        if (instance == null)
            instance = new Singleton();
        return instance;
    }
}
```

存在的问题：第一次实例化之后只需读取`instance`，是并发安全的，而线程每次进入该方法都要加锁，性能较低

### 线程安全2
仅对实例化部分加锁+双重校验

```java
public class Singleton {
    private static volatile Singleton instance;

    private Singleton() {}

    public static synchronized Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null)
                    instance = new Singleton();
            }
        }
        return instance;
    }
}
```

注意：
* 为了避免多次实例化，synchronized块中需要进行二次校验，例如两个线程同时进入`if (instance == null)`，线程1先获得锁并实例化，之后线程2获得锁，如果不重新判断则又会实例化一次
* `instance`必须使用volatile关键字修饰，因为`instance = new Singleton()`实际分为三步执行：分配内存空间、初始化对象、将`instance`指向分配的内存地址；由于JVM可能进行指令重排，因此在多线程环境下可能获取到一个没有被初始化的实例，因此要使用volatile关键字避免指令重排

## UML类图
![单例模式UML类图](/assets/images/design-pattern-singleton/单例模式UML类图.png)

## 应用场景
* Logger类
* 配置类
* 以共享方式访问资源

## JDK
* java.lang.Runtime: getRuntime()
* java.lang.System: getSecurityManager()
