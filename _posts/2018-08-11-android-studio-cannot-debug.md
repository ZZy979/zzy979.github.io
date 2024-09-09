---
title: Android Studio无法调试的问题
date: 2018-08-11 23:28 +0800
categories: [Android]
tags: [android, android studio]
---
今天在调试程序时突然发现无法进入调试，卡在waiting for debugger，显示Could not connect to remote process. Aborting debug session.网上查到的重启adb、关闭端口号占用等解决方法都没用，重启Android Studio、重启电脑也不行。后来查到要关闭Instant Run，打开设置，在Build, Execution, Deployment->Instant Run中取消第一行Enable Instant Run to...勾选居然就可以调试了，但再次勾选也没有出现之前的问题，不知道为什么...
