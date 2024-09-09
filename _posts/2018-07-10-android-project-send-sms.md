---
title: 【Android】项目：短信发送
date: 2018-07-10 15:59 +0800
categories: [Android]
tags: [android]
---
* 权限设置：

```xml
<uses-permission android:name="android.permission.SEND_SMS" />
```

* 发送短信代码：

```java
Intent intent = new Intent();
SmsManager smsManager = SmsManager.getDefault();
smsManager.sendTextMessage(number, null, content, null, null);
```

* Android短信内容长度限制为75个汉字，如果超过长度则拆分，分段发送：

```java
ArrayList<String> contents = smsManager.divideMessage(content);
for (String str : contents)
    smsManager.sendTextMessage(number, null, str, null, null);
```

![短信发送-输入界面](/assets/images/android-project-send-sms/短信发送-输入界面.png)

![短信发送-短信界面](/assets/images/android-project-send-sms/短信发送-短信界面.png)
