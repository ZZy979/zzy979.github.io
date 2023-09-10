---
title: 【设计模式】5.建造者
date: 2021-09-10 15:31:42 +0800
categories: [Design Pattern]
tags: [design pattern]
---
## 定义
建造者(builder)模式（也称为生成器模式）用于封装对象的构造过程，允许按步骤构造

最典型的实例：`StringBuilder`

## 解决的问题
封装了对象的构造过程，降低了创建对象的复杂度，方便用户创建复杂的对象

## 实现
假设`Product`类有`a`和`b`两个部件：
```java
public class Product {
    private int a, b;

    public int getA() { return a; }
    public void setA(int a) { this.a = a; }
    public int getB() { return b; }
    public void setB(int b) { this.b = b; }
}
```

生成器接口`Builder`定义了构造各部件的方法，并提供一个获取构造完成的对象的方法：
```java
public interface Builder {
    void buildPartA();
    void buildPartB();
    Product getProduct();
}
```

生成器的实现类实现各部件的构造操作，并负责组装`Product`对象：
```java
public class ConcreteBuilder implements Builder {
    private Product product = new Product();

    @Override
    public void buildPartA() {
        product.setA(1);
    }

    @Override
    public void buildPartB() {
        product.setB(2);
    }

    @Override
    public Product getProduct() {
        return product;
    }

}
```

`Director`类（指导者）使用`Builder`构造`Product`对象，将构造对象的各步骤集中在一个方法中：
```java
public class Director {
    private Builder builder;

    public Director(Builder builder) {
        this.builder = builder;
    }

    public Product buildProduct() {
        builder.buildPartA();
        builder.buildPartB();
        return builder.getProduct();
    }
}
```

客户端使用`Director`获取对象：
```java
public class Client {
    public static void main(String[] args) {
        Builder builder = new ConcreteBuilder();
        Director director = new Director(builder);
        Product product = director.buildProduct();
        // do something with product
    }
}
```

## UML类图
![建造者模式UML类图](/assets/images/design-pattern-builder/建造者模式UML类图.png)

## 优点
* 将对象本身与构造过程解耦，可以使用相同的构造过程得到不同的产品
* 允许按步骤构造复杂对象，使构造过程更加清晰
* 增加新的生成器实现类无需修改现有代码，符合开闭原则

## 缺点
* 建造者模式创建的对象需要具备共性（具有相似的组成部分），否则不适合使用，使用范围受限
* 如果产品的内部变化复杂，可能会导致需要定义很多生成器实现类来适应这种变化，导致系统变得很庞大

## 应用场景
* 需要构造的对象有复杂的内部结构，这些对象具备共性
* 可以使用相同的构造过程得到不同的产品（例如：生成HTML文件和XML文件都包含文件头、文件体和文件尾三部分）

## JDK
* java.lang.StringBuilder
* java.lang.StringBuffer
* java.nio.ByteBuffer
