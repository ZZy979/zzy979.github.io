---
title: 【设计模式】7.外观
date: 2021-09-15 14:17:50 +0800
categories: [Design Pattern]
tags: [design pattern]
---
## 定义
外观(facade)模式提供了一个统一的接口，用来访问子系统中的一群接口

![外观模式示意图](/assets/images/design-pattern-facade/外观模式示意图.png)

## 解决的问题
* 降低了客户端与子系统之间的耦合度
* 简化了复杂子系统的使用

## 实现
假设有两个子系统：
```java
public class Subsystem1 {

    public void begin() {
        System.out.println("Subsystem1 begin");
    }

    public void end() {
        System.out.println("Subsystem1 end");
    }

}

public class Subsystem2 {

    public void begin() {
        System.out.println("Subsystem2 begin");
    }

    public void end() {
        System.out.println("Subsystem2 end");
    }

}
```

不使用外观模式的客户端：
```java
public class Client {

    public static void main(String[] args) {
        Subsystem1 subsystem1 = new Subsystem1();
        Subsystem2 subsystem2 = new Subsystem2();
        subsystem1.begin();
        subsystem2.begin();
        subsystem1.end();
        subsystem2.end();
    }

}
```

使用外观模式的实现方式：
```java
public class Facade {
    private Subsystem1 subsystem1;
    private Subsystem2 subsystem2;

    public Facade(Subsystem1 subsystem1, Subsystem2 subsystem2) {
        this.subsystem1 = subsystem1;
        this.subsystem2 = subsystem2;
    }

    public void call() {
        subsystem1.begin();
        subsystem2.begin();
        subsystem1.end();
        subsystem2.end();
    }

}

public class Client {

    public static void main(String[] args) {
        Subsystem1 subsystem1 = new Subsystem1();
        Subsystem2 subsystem2 = new Subsystem2();
        Facade facade = new Facade(subsystem1, subsystem2);
        facade.call();
    }

}
```

注：
* 每个子系统表示一个类的集合
* 对于子系统来说，外观类只是另一种客户端
* 外观模式本质上就是对多个子系统做了一层封装，将多个“子任务”组合成一个“复合任务”

## UML类图
![外观模式UML类图](/assets/images/design-pattern-facade/外观模式UML类图.png)

## 优点
* 降低了客户端与子系统之间的耦合度
* 简化了复杂子系统的使用

## 缺点
* 增加新的子系统需要修改外观类或客户端的代码，违反了开闭原则
* 不能很好地限制客户端使用子系统，如果做太多限制则减少了灵活性

## 应用场景
* 要为一个复杂的子系统提供一个简单的对外接口
* 客户端与多个子系统之间存在很大的依赖性
* 在层次化结构中，可以使用外观模式定义每一层的入口
