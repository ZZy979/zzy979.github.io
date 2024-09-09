---
title: Android Studio减少APK大小
date: 2018-07-26 10:03 +0800
categories: [Android]
tags: [android, android studio]
---
使用Android Studio创建一个默认的Hello World项目，不添加其他任何代码，生成的APK已经有1.5 MB：

![HelloWorld代码](/assets/images/android-studio-reduce-apk-size/HelloWorld代码.png)

![修改前APK大小](/assets/images/android-studio-reduce-apk-size/修改前APK大小.png)

主要原因是工程里自动添加了v7的向下兼容包，为了使低版本（4.0以下）的Android能够使用高版本（4.0及以上）的特性。如果我们用不上这个兼容包就可以将其删除：

1.打开工程设置：File->Project Structure或View->Open Module Settings，在app->Dependencies选项卡中删除com.android.support:appcompat-v7:28.0.0-alpha3和com.android.support.constraint:constraint-layout:1.1.2，自动重新构建工程后开始报错。

![删除依赖](/assets/images/android-studio-reduce-apk-size/删除依赖.png)

2.将MainActivity的父类由AppCompatActivity改为Activity，导入android.app.Activity包，移除android.support.v7.app.AppCompatActivity包，此时仍然报错。
	
![修改MainActivity父类前](/assets/images/android-studio-reduce-apk-size/修改MainActivity父类前.png)

![修改MainActivity父类后](/assets/images/android-studio-reduce-apk-size/修改MainActivity父类后.png)

3.修改布局文件res/layout/activity_main.xml，将默认的TextView删除，根布局改为LinearLayout并删除无用的XML命名空间xmlns:app：

![修改根布局前](/assets/images/android-studio-reduce-apk-size/修改根布局前.png)

![修改根布局后](/assets/images/android-studio-reduce-apk-size/修改根布局后.png)

4.修改资源文件res/values/styles.xml：

![修改资源文件前](/assets/images/android-studio-reduce-apk-size/修改资源文件前.png)

![修改资源文件后](/assets/images/android-studio-reduce-apk-size/修改资源文件后.png)

主题的parent属性也可以改为android:Theme.Material.Light.DarkActionBar，更好看一些

5.点击Build->Make Project重新构建工程，发现此时构建工程比原来快多了。重新生成APK文件，只有82.3 KB。

![修改后APK大小](/assets/images/android-studio-reduce-apk-size/修改后APK大小.png)

以上是从现有工程中删除依赖包的方法，更简单的方法是新建工程时不要勾选向后兼容选项，则新建的项目不会自动添加向下兼容包。

![新建工程取消勾选向后兼容](/assets/images/android-studio-reduce-apk-size/新建工程取消勾选向后兼容.png)
