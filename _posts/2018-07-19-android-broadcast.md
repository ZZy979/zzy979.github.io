---
title: 【Android】Broadcast
date: 2018-07-19 09:17 +0800
categories: [Android]
tags: [android, broadcast]
---
* 广播是是一个全局的监听器，监听/接收Activity发出的广播消息，并作出响应
* 创建广播接收器类：

```java
public class MyBroadcastReceiver extends BroadcastReceiver {
    public MyBroadcastReceiver() {
        System.out.println("**每次广播都会实例化一个新的广播组件进行操作。");
    }

    @Override
    public void onReceive(Context context, Intent intent) {
        // TODO: This method is called when the BroadcastReceiver is receiving
        // an Intent broadcast.
    }
}
```

* 配置表单文件：

```xml
<receiver
    android:name=".MyBroadcastReceiver"
    android:enabled="true" >
    <intent-filter>
        <action android:name="com.zzy.action.MY_ACTION" />
    </intent-filter>
</receiver>
```

* `<intent-filter>`：决定广播接收器响应哪些动作的intent，设置intent动作的方法：
`new Intent(MY_ACTION)`或`intent.setAction(MY_ACTION)`
* 从Activity中启动广播：`sendBroadcast(new Intent(MY_ACTION))`
