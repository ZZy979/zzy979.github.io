---
title: 【Android】Intent
date: 2018-07-17 15:40 +0800
categories: [Android]
tags: [android]
---
* Intent（意图）是一个完整的线程

```java
Intent intent = new Intent(SrcActivity.this, DstActivity.class)
startActivity(intent);
```

* startActivity()：单向传输数据，startActivityForResult()：双向传输数据，可以返回数据
* 类似于new Thread(...).start()
* 反射机制很重要的两个概念：Class表示类，ClassName.class实例化
* Activity使用的是栈结构，调用的关联关系使用Intent实现

## 应用
(1) 调用Activity

```java
Intent it = new Intent(SrcActivity.this, DstActivity.class);
it.putExtra("key", value);
startActivity(it);
```

被调用的Activity获取数据：

```java
Intent intent = super.getIntent();
String value = intent.getStringExtra("key");
```

可以调用同一个Activity的另一个实例：

```java
startActivity(new Intent(MainActivity.this, MainActivity.class));
```

(2) 打开网页（调用浏览器）

```java
Uri uri = Uri.parse("http://" + etWebsite.getText().toString());
Intent it = new Intent();
it.setAction(Intent.ACTION_VIEW);
it.setData(uri);
startActivity(it);
```

权限配置：

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

(3) 打电话（调用电话应用）

见[项目：电话拨号器]({% post_url 2018-07-10-android-project-call-phone %})

(4) 发短信（调用短信应用）

见[项目：短信发送]({% post_url 2018-07-10-android-project-send-sms %})

(5) 发送Email（调用邮件应用）

```java
Intent intent = new Intent(Intent.ACTION_SEND);
intent.setType("plain/text");
intent.putExtra(Intent.EXTRA_EMAIL, address);  //address是字符串数组
intent.putExtra(Intent.EXTRA_SUBJECT, subject);
intent.putExtra(Intent.EXTRA_TEXT, content);
startActivity(intent);
```

权限配置：

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

(6) 调用ContentProvider

俗成>约定>配置>编程
