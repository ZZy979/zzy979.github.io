---
title: Android Studio更改目标SDK版本
date: 2018-07-26 10:41 +0800
categories: [Android]
tags: [android, android studio]
---
Android Studio创建新工程时默认设置的目标SDK版本是最新SDK版本，目前是API 28。但是这个版本的SDK不能下载源码，编程时无法查看。以下是修改工程目标SDK版本的方法。

1.打开工程设置：File->Project Structure或View->Open Module Settings，在app->Properties选项卡中将Compile Sdk Version改为希望的版本（如API 26）

![修改Compile Sdk Version](/assets/images/android-studio-change-target-sdk-version/修改Compile Sdk Version.png)

2.在Flavor选项卡中将Target Sdk Version改为希望的版本

![修改Target Sdk Version](/assets/images/android-studio-change-target-sdk-version/修改Target Sdk Version.png)

3.如果添加了向下兼容包，重新构建工程时会报错，需要修改模块的build.gradle文件

![修改build.gradle文件前](/assets/images/android-studio-change-target-sdk-version/修改build.gradle文件前.png)

将上图中的28.0.0-alpha3改为26.1.0，并点击右上角提示的Sync Now同步工程，或点击Build->Make Project重新构建工程

![修改build.gradle文件后](/assets/images/android-studio-change-target-sdk-version/修改build.gradle文件后.png)

4.此时工程可以正常运行，并且可以查看Android的源代码。

