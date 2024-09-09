---
title: 【Android】项目：电话拨号器
date: 2018-07-10 10:47 +0800
categories: [Android]
tags: [android]
---
* 要使用系统API拨号需要授权：在表单文件AndroidManifest.xml中添加：
<uses-permission android:name="android.permission.CALL_PHONE" />
* [AndroidManifest.xml权限大全]({% post_url 2018-07-18-android-manifest-permissions %})
* 拨号代码：

```java
Intentintent = new Intent();    // 意图要干什么
intent.setAction(Intent.ACTION_CALL);
intent.setData(Uri.parse("tel:" + number));
startActivity(intent);
```

![电话拨号器-输入界面](/assets/images/android-project-call-phone/电话拨号器-输入界面.png)

![电话拨号器-拨号界面](/assets/images/android-project-call-phone/电话拨号器-拨号界面.png)
