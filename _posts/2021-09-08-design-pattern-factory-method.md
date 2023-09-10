---
title: 【设计模式】3.工厂方法
date: 2021-09-08 15:06:41 +0800
categories: [Design Pattern]
tags: [design pattern]
---
## 定义
工厂方法(factory method)模式在父工厂类中定义了一个创建对象的接口，由子工厂类决定实例化哪个类

## 解决的问题
将实例化操作推迟到子工厂类，解决了简单工厂模式的缺点

## 实现
假设`Product`接口有两种实现类：
```java
public interface Product {
}
  
public class ConcreteProduct1 implements Product {
}
  
public class ConcreteProduct2 implements Product {
}
```

父工厂类定义了一个工厂方法，由子工厂类决定实例化`Product`的哪个实现类：
```java
public abstract class Factory {
    public abstract Product createProduct();
}

public class ConcreteFactory1 extends Factory {
    @Override
    public Product createProduct() {
        return new ConcreteProduct1();
    }
}

public class ConcreteFactory2 extends Factory {
    @Override
    public Product createProduct() {
        return new ConcreteProduct2();
    }
}
```

客户端：
```java
public class Client {
    public static void main(String[] args) {
        Factory factory = new ConcreteFactory1();
        Product product = factory.createProduct();
        // do something with product
    }
}
```

## UML类图
![工厂方法模式UML类图](/assets/images/design-pattern-factory-method/工厂方法模式UML类图.png)

## 优点
* 降低耦合，客户端不需要依赖具体实现类
* 符合开闭原则，在添加新的实现类时只需添加新的子工厂类
* 符合单一职责原则，每个子工厂类只负责创建一种实现类
* 不使用静态工厂方法，可以形成基于继承的类层次结构

## 缺点
* 每增加一个新的产品类就要增加一个新的子工厂类，类的个数会越来越多，增加系统的复杂性，同时会增加编译时间，给系统带来额外开销
* 客户端要改用另一种产品类仍然需要修改实例化子工厂类的代码
* 每个子工厂类只能创建一种产品

## 应用场景
客户端不知道具体实现类的类名，只知道对应的工厂类

## JDK
java.util.concurrent.ThreadFactory
