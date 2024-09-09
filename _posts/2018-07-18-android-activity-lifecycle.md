---
title: 【Android】Activity生命周期
date: 2018-07-18 15:02 +0800
categories: [Android]
tags: [android, activity]
---

![Activity生命周期](/assets/images/android-activity-lifecycle/Activity生命周期.png)

Android中进程的5种状态：
1. 当前进程
2. 后台Activity
3. Service
4. 后台进程
5. 空进程

如果内存空间不够则从下往上依次删除
Service只是临时删除，保留句柄，可以运行时再恢复

官方文档：<https://developer.android.google.cn/guide/components/activities/activity-lifecycle>

Android中生命周期方法都是由系统调用，系统会根据当前Activity的不同状态回调相应的方法。

1.Activity的生命周期：
* onCreat()：当Activity第一次被创建时回调的方法，加载当前Activity的布局或者初始化View或者加载数据到集合等
* onStart()：当Activity能够被用户看到时调用的方法
* onResume()：当Activity能够与用户交互时回调的方法，或者称为获取用户焦点时回调的方法
* onPause()：当Activity失去用户焦点时回调的方法（不能与用户交互，暂停方法）启动了其它的Activity时就会回调
* onStop()：当Activity完全被遮挡的时候回调的方法
* onRestart()：当Activity重新被启动时回调的方法
* onDestroy()：当Activity被销毁时回调的方法，或由于配置变更（例如设备旋转或多窗口模式），系统暂时销毁 Activity

2.Activity切换时生命周期调用的顺序（A->B）：

Activity切换：

```
A-----onPause()
B-----onCreate()
B-----onStart()
B-----onResume()
A-----onStop()
```

点击Back键返回：

```
B-----onPause()
A-----onRestart()
A-----onStart()
A-----onResume()
B-----onStop()
B-----onDestroy()
```

3.Activity中finish()和onDestroy()的区别：
* finish()方法用于**主动**结束一个Activity的生命周期；而onDestory()方法则是Activity的一个生命周期方法，在一个Activity对象被销毁之前，Android系统会调用该方法，用于释放此Activity之前所占用的资源。
* finish()会调用到onDestroy()方法
