---
title: 【设计模式】9.桥接
date: 2021-09-22 20:11:21 +0800
categories: [Design Pattern]
tags: [design pattern]
---
## 定义
桥接(bridge)模式将抽象部分与它的实现部分分离，使它们可以独立变化

桥接模式将继承关系转化成关联关系，降低了类之间的耦合度，减少了系统中类的数量

## 解决的问题
当类层次结构的变化有两个维度，一个维度的变化会引起另一个维度进行相应的变化，使得系统扩展起来非常困难

桥接模式将这两个维度分离（将抽象部分与实现部分分离），使其相互独立，从而实现两个部分可以独立变化，使扩展变得简单

## 示例1：消息系统
考虑一个发送消息的业务功能，消息类型分为普通消息和加急消息，发送方式分为短信和邮件

加急消息可以监控处理进度

不使用模式的实现方式

消息接口`Message`定义了发送消息的方法：
```java
public interface Message {
    void send(String message, String user);
}
```

普通消息两种发送方式的实现类如下：
```java
public class CommonMessageSMS implements Message {
    @Override
    public void send(String message, String user) {
        System.out.printf("短信发送普通消息 \"%s\" 给 %s\n", message, user);
    }
}

public class CommonMessageEmail implements Message {
    @Override
    public void send(String message, String user) {
        System.out.printf("邮件发送普通消息 \"%s\" 给 %s\n", message, user);
    }
}
```

加急消息接口增加了监控处理进度的方法（仅示意）：
```java
public interface UrgencyMessage extends Message {
    Object watch(int messageId);
}
```

加急消息对应的两种实现类如下：
```java
public class UrgencyMessageSMS implements UrgencyMessage {
    @Override
    public void send(String message, String user) {
        message = "加急：" + message;
        System.out.printf("短信发送加急消息 \"%s\" 给 %s\n", message, user);
    }

    @Override
    public Object watch(int messageId) {
        return null;
    }
}

public class UrgencyMessageEmail implements UrgencyMessage {
    @Override
    public void send(String message, String user) {
        message = "加急：" + message;
        System.out.printf("邮件发送加急消息 \"%s\" 给 %s\n", message, user);
    }

    @Override
    public Object watch(int messageId) {
        return null;
    }
}
```

客户端：
```java
public class Client {

    public static void main(String[] args) {
        Message message = new CommonMessageSMS();
        message.send("Hello, world!", "张三");

        message = new CommonMessageEmail();
        message.send("Hello, world!", "张三");

        message = new UrgencyMessageSMS();
        message.send("Hello, world!", "张三");

        message = new UrgencyMessageEmail();
        message.send("Hello, world!", "张三");
    }

}
```

输出结果：
```
短信发送普通消息 "Hello, world!" 给 张三
邮件发送普通消息 "Hello, world!" 给 张三
短信发送加急消息 "加急：Hello, world!" 给 张三
邮件发送加急消息 "加急：Hello, world!" 给 张三
```

当前系统的UML类图如下：

![消息系统UML类图](/assets/images/design-pattern-bridge/消息系统UML类图.png)

目前有2种消息类型、2种发送方式，因此有2×2=4个实现类

如果要增加特急消息，则要增加一个接口和2种发送方式对应的2个实现类

如果再增加一种发送方式，则要增加3种消息类型对应的3个实现类

如果要修改消息处理逻辑则要修改所有3×3=9个实现类

|  | 普通消息 | 加急消息 | 特急消息 |
| --- | --- | --- | --- |
| 短信 |
| 邮件 |
| 站内通知 |

可见这种实现方式扩展和修改都非常困难，因为消息类型和发送方式两个维度混合在一起，一个维度的变化会影响另一个维度

## 示例2：绘制图形
一个绘制图形的业务功能可以绘制的形状包括正方形、方形和圆形，每种形状有白色、灰色和黑色三种颜色

|  | 正方形 | 长方形 | 圆形 |
| --- | --- | --- | --- |
| 白色 |
| 灰色 |
| 黑色 |

如果为每种颜色的每种形状都提供一个类，则增加三角形就要增加对应的三种颜色；增加红色就要增加对应的四种形状

## 实现
桥接模式将抽象部分与实现部分分离
* Abstraction（抽象类）：抽象部分的接口，定义跟具体业务相关的方法，包含一个实现部分的对象引用
* RefinedAbstraction：扩展抽象部分的接口，定义跟实际业务相关的方法
* Implementor：实现部分的接口，提供基本操作，供Abstraction调用
* ConcreteImplementor：具体实现类

```java
public abstract class Abstraction {
    protected Implementor impl;

    public Abstraction(Implementor impl) {
        this.impl = impl;
    }

    public void operation() {
        impl.operationImpl();
    }
}

public class RefinedAbstraction extends Abstraction {
    public RefinedAbstraction(Implementor impl) {
        super(impl);
    }

    public void otherOperation() {
    }
}

public interface Implementor {
    public void operationImpl();
}

public class ConcreteImplementor implements Implementor {
    @Override
    public void operationImpl() {
    }
}
```

## UML类图
![桥接模式UML类图](/assets/images/design-pattern-bridge/桥接模式UML类图.png)

## 重新实现示例
使用桥接模式重新实现消息系统的示例

Abstraction=消息

RefinedAbstraction=普通消息、加急消息

Implementor=发送器

ConcreteImplementor=短信发送器、邮件发送器

抽象部分：消息
```java
public abstract class AbstractMessage {
    protected Sender sender;

    public AbstractMessage(Sender sender) {
        this.sender = sender;
    }

    public void sendMessage(String message, String user) {
        sender.send(message, user);
    }
}
```

实现部分：发送器
```java
public interface Sender {
    void send(String message, String user);
}

public class SMSSender implements Sender {
    @Override
    public void send(String message, String user) {
        System.out.printf("短信发送普通消息 \"%s\" 给 %s\n", message, user);
    }
}

public class EmailSender implements Sender {
    @Override
    public void send(String message, String user) {
        System.out.printf("邮件发送普通消息 \"%s\" 给 %s\n", message, user);
    }
}
```

扩展抽象部分：具体类型的消息
```java
public class CommonMessage extends AbstractMessage {
    public CommonMessage(Sender sender) {
        super(sender);
    }
}

public class UrgencyMessage extends AbstractMessage {
    public UrgencyMessage(Sender sender) {
        super(sender);
    }

    @Override
    public void sendMessage(String message, String user) {
        super.sendMessage("加急：" + message, user);
    }

    Object watch(int messageId) {
        return null;
    }
}
```

客户端：
```java
public class Client {

    public static void main(String[] args) {
        Sender smsSender = new SMSSender();
        AbstractMessage message = new CommonMessage(smsSender);
        message.sendMessage("Hello, world!", "张三");

        message = new UrgencyMessage(smsSender);
        message.sendMessage("Hello, world!", "张三");

        Sender emailSender = new EmailSender();
        message = new CommonMessage(emailSender);
        message.sendMessage("Hello, world!", "张三");

        message = new UrgencyMessage(emailSender);
        message.sendMessage("Hello, world!", "张三");
    }

}
```

输出结果：
```
短信发送普通消息 "Hello, world!" 给 张三
短信发送普通消息 "加急：Hello, world!" 给 张三
邮件发送普通消息 "Hello, world!" 给 张三
邮件发送普通消息 "加急：Hello, world!" 给 张三
```

重写后系统的UML类图如下：

![重写后消息系统UML类图](/assets/images/design-pattern-bridge/重写后消息系统UML类图.png)

此时要增加一种消息类型，只需增加一个`AbstractMessage`的子类； 要增加一种发送方式，只需增加一个`Sender`接口的实现类

由于桥接模式将继承关系转化为关联（组合）关系，使得“消息类型”和“发送方式”两个维度分离开，从而两个维度的变化是相互独立的，同时使得类的数量由m*n降到m+n

客户端使用时只需根据实际需要组合两个部分的实现类即可

## 优点
* 分离抽象和实现部分，使它们可以独立变化，提高了系统的灵活性和可扩展性，符合开闭原则
* 将继承关系转化成关联关系，降低了类之间的耦合度，减少了系统中类的数量

## 缺点
* 增加系统的理解与设计难度
* 要求正确识别出系统中两个独立变化的维度，因此使用范围具有一定的局限性

## 应用场景
* 类层次结构存在两个独立变化的维度，且这两个维度都需要扩展
* 不希望因为继承导致类的数量急剧增加

## JDK
JDBC
