---
title: 【设计模式】4.抽象工厂
date: 2021-09-16 14:49:36 +0800
categories: [Design Pattern]
tags: [design pattern]
---
## 定义
抽象工厂(abstract factory)模式提供多个接口，用于创建**一系列相关的对象**（这些对象是相关的，必须一起创建出来），工厂方法模式只用于创建一个对象

## 解决的问题
解决了工厂方法模式的缺点，每个子工厂类可以创建多种产品

## 实现
假设`ProductA`和`ProductB`接口各有两种实现类：
```java
public interface ProductA {
}
  
public class ConcreteProductA1 implements Product {
}
  
public class ConcreteProductA2 implements Product {
}

public interface ProductB {
}
  
public class ConcreteProductB1 implements Product {
}
  
public class ConcreteProductB2 implements Product {
}
```

抽象工厂类定义了创建`ProductA`和`ProductB`的两个抽象方法，由子工厂类决定分别实例化哪个实现类：
```java
public abstract class AbstractFactory {
    public abstract ProductA createProductA();
    public abstract ProductB createProductB();
}

public class ConcreteFactory1 extends AbstractFactory {

    @Override
    public ProductA createProductA() {
        return new ConcreteProductA1();
    }

    @Override
    public ProductB createProductB() {
        return new ConcreteProductB1();
    }

}

public class ConcreteFactory2 extends AbstractFactory {

    @Override
    public ProductA createProductA() {
        return new ConcreteProductA2();
    }

    @Override
    public ProductB createProductB() {
        return new ConcreteProductB2();
    }

}
```

创建“一系列”对象的概念体现在客户端，客户端分别调用这两个方法创建出两个对象，因此这两个对象具有“相关性”：
```java
public class Client {
    public static void main(String[] args) {
        AbstractFactory factory = new ConcreteFactory1();
        ProductA productA = factory.createProductA();
        ProductB productB = factory.createProductB();
        // do something with productA and productB
    }
}
```

抽象工厂模式包含多个创建对象的方法，实际上是多个工厂方法模式的组合，单独看每一个方法就是工厂方法模式

## UML类图
![抽象工厂模式UML类图](/assets/images/design-pattern-abstract-factory/抽象工厂模式UML类图.png)

## 优点
同工厂方法

（注：添加新的实现类是指`ProductA`添加一个实现类`ConcreteProductA3`，不是添加一种新的产品种类`ProductC`）

## 缺点
难以支持添加新的产品种类（例如`ProductC`），此时需要修改所有工厂类，违背了开闭原则

（注：抽象工厂模式对于新的实现类符合开闭原则，对于新的产品种类不符合开闭原则，这一特性称为开闭原则的倾斜性）

## 应用场景
系统有多个种类的产品，每个种类只需要一种实现类

## JDK
* javax.xml.parsers.DocumentBuilderFactory: newDocumentBuilder()
* javax.xml.transform.TransformerFactory: newTransformer()和newTemplates()
* javax.xml.xpath.XPathFactory: newXPath()
