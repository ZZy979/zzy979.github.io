---
title: 【Android】基础
date: 2018-07-10 10:33 +0800
categories: [Android]
tags: [android]
---
* Android：一个项目（应用）就是一个进程
* Android四大部件：Activity活动、Service服务、ContentProvider内容提供者、BroadcastReceiver广播接收器
* 必须掌握的基础知识：JDK 1.8, Maven
* 官方文档：<https://developer.android.google.cn/guide>
* AndroidManifest.xml：表单文件，所有资源在此注册
* activity_main.xml
  * 前端界面，放置控件，相当于Java Web应用中的jsp页面
  * xmlns：XML命名空间
  * context上下文：声明一个项目中所有资源路径的联系
  * Android Studio中Design界面能看到预览的方法：将主题设置为Light

![设置Android Studio主题](/assets/images/android-basis/设置Android Studio主题.png)

* MainActivity.java
  * 与activity_main.xml相关的后台代码，Activity相当于Java Web应用中的Servlet、Struts中的Action
  * onCreate()：参数savedInstanceState：绑定（Bundle）类型，保存实例状态（前一个活动页面的句柄，只针对当前app）。在Android中一个项目是一个进程，使用栈保存项目状态。切换到其他Activity时该Activity的状态被保存，切换回来时使用savedInstanceState恢复状态，不需重新启动应用。
  * setContentView()：设置视图
* R.java
  * R: 注册，所有资源全部在此注册（生成一个整数），自动生成，绝对不能修改！
* 组件
  * View视图：所有控件的基类，如标签TextView、按钮Button、编辑框EditText、图片ImageView等
  * ViewGroup视图组：容器或布局（Layout），也是View的子类
