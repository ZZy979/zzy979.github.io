---
title: Android Studio修改模拟器存储目录
date: 2025-05-05 13:25:34 +0800
categories: [Android]
tags: [android, android studio]
---
Android Studio默认将模拟器(AVD)数据存储在$HOME/.android/avd目录中，在Windows系统中会占用大量的C盘空间。可以通过设置环境变量指定存储目录，从而节省C盘空间。

## 环境变量
Android Studio最常用的环境变量如下表所示。

| 环境变量 | 描述 | 默认值 | 旧版本 |
| --- | --- | --- | --- |
| `ANDROID_HOME` | Android SDK安装目录 | | `ANDROID_SDK_ROOT` |
| `ANDROID_USER_HOME` | Android SDK工具用户偏好设置目录 | `$HOME/.android` | `ANDROID_SDK_HOME` |
| `ANDROID_EMULATOR_HOME` | 模拟器配置目录 | `$ANDROID_USER_HOME` | `$ANDROID_SDK_HOME/.android` |
| `ANDROID_AVD_HOME` | AVD数据文件目录 | `$ANDROID_EMULATOR_HOME/avd` | `$ANDROID_SDK_HOME/.android/avd` |

如果希望将模拟器数据存储在 D:\Android\.android 目录中，只需将环境变量`ANDROID_USER_HOME`设置为该目录：

```shell
set ANDROID_USER_HOME=D:\Android\.android
```

对于旧版本（4.3及以前）的Android Studio，应该将环境变量`ANDROID_SDK_HOME`设置为.android目录的上级目录：

```shell
set ANDROID_SDK_HOME=D:\Android
```

之后重启Android Studio即可。

## 参考
[Environment variables | Android Studio](https://developer.android.google.cn/tools/variables)
