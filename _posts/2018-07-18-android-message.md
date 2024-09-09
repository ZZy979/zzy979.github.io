---
title: 【Android】消息机制
date: 2018-07-18 11:17 +0800
categories: [Android]
tags: [android]
---
* 参考：<https://www.cnblogs.com/shakinghead/p/8057674.html>
* 4个主要的类：Message, Handler, MessageQueue, Looper

## 1.Message
封装消息
* 属性（公有成员）：

| 属性 | 含义 |
| --- | --- |
| int what | 操作种类 |
| Object obj | 传递的数据 |
| int arg1 | 整型数据1 |
| int arg2 | 整型数据2 |

* what一般通过类的静态常量域自己定义，例如`private static final int SET = 1;`
* 构造Message对象：

```java
Message msg = new Message(what, arg1, arg2, obj);
```

或：

```java
Message msg = new Message(what, obj);
```

或：

```java
Message msg = new Message();
msg.what = SET;
msg.obj = "data";
```

## 2.Handler
操作消息
* 构造对象，定义消息的处理操作：

```java
Handler handler = new Handler() {
    @Override
    public void handleMessage(Message msg) {
        switch (msg.what) {
            case SET:
                // ...
                break;
        }
    }
};
```

* 发送消息：handler.sendMessage(msg)
* 获取消息对象：handler.obtainMessage()或handler.obtainMessage(what, arg1, arg2, obj)

**与new Message()的区别**：obtainMessage()是从消息池中拿来一个Message，不需要另开辟空间，可以循环利用；new需要重新申请，效率低。

**内存泄漏**：如果直接在MainActivity中向上面那样定义匿名Handler类的对象，IDE会警告可能发生内存泄漏：

```
This Handler class should be static or leaks might occur (anonymous android.os.Handler)
```

参考：<https://blog.csdn.net/banxiali/article/details/51494842>

解决方法：将匿名Handler类单独定义为一个**静态类**，在handleMessage()方法中使用switch(msg.what)判断消息种类，从而执行不同的操作。这样可以把不同功能的Handler集中在一个类中。

```java
private static class NonLeakHandler extends Handler {
    private WeakReference<MainActivity> mWeakReference;

    public NonLeakHandler(MainActivity reference) {
        mWeakReference = new WeakReference<>(reference);
    }

    @Override
    public void handleMessage(Message msg) {
        MainActivity activity = mWeakReference.get();
        if (activity == null)    // the referenced object has been cleared
            return;
        // do something
    }
}
```

## 3.MessageQueue
消息队列，存放所有Handler发送的消息。

## 4.Looper
消息通道，是消息队列的“管家”，将消息从消息队列中一条条取出，并分派到Handler的handleMessage()方法中。

![Message、Handler和Looper](/assets/images/android-message/Message、Handler和Looper.png)

在使用Handler处理Message时，实际上都需要依靠一个Looper通道完成。当用户取得一个Handler对象时实际上都是通过Looper完成的，在一个Activity类之中，会自动帮助用户启动好的Looper对象，而如果是在一个用户定义的类中，则需要用户手工使用Looper类中的若干方法之后才可以正常的启动Looper对象。
