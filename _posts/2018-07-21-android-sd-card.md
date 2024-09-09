---
title: 【Android】SD卡
date: 2018-07-21 13:56 +0800
categories: [Android]
tags: [android, sd card]
---
* SD卡：Android的外部存储空间
* 使用DOS命令创建SD卡：mksdcard 2048M D:\sdcard.img

  其中mksdcard.exe位于[SDK安装目录]\emulator
* 检测SD卡是否存在：

```java
if (Environment.getExternalStorageState().equals(Environment.MEDIA_MOUNTED))
    // SD卡存在
```

* 向SD卡写文件：

```java
File sdCardDir = Environment.getExternalStorageDirectory();     // 获取SD卡目录
File saveFile = new File(sdCardDir, "itcast.txt");
FileOutputStream outputStream = new FileOutputStream(saveFile);
```

需要的权限（第一个权限似乎并不需要，Android Studio还会报错）：

```xml
<!-- 在SDCard中创建与删除文件权限 -->
<uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS" />
<!-- 往SDCard写入数据权限 -->
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<!-- 从SDCard读取数据权限 -->
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
```

Genymotion模拟器测试结果：

![SD卡文件](/assets/images/android-sd-card/SD卡文件.png)

其中legacy是一个指向mnt/shell/emulated/0的链接，实际的绝对路径：mnt/shell/emulated/0/itcast.txt

![SD卡文件2](/assets/images/android-sd-card/SD卡文件2.png)

真实手机（Android 6.0）测试结果：

![真实手机测试结果](/assets/images/android-sd-card/真实手机测试结果.jpg)

因为WRITE_EXTERNAL_STORAGE和READ_EXTERNAL_STORAGE权限在Android 6.0之上的系统除了在表单文件中配置，还要在代码中动态申请！

```java
private void verifyStoragePermissions() {
   //检查是否有写权限
   int check = ActivityCompat.checkSelfPermission(this,
         Manifest.permission.WRITE_EXTERNAL_STORAGE);
   if (check != PackageManager.PERMISSION_GRANTED) {
      String[] permissions = new String[]{ Manifest.permission.WRITE_EXTERNAL_STORAGE,
            Manifest.permission.READ_EXTERNAL_STORAGE };
      ActivityCompat.requestPermissions(this, permissions, 1);
   }
}
```

再次测试结果如下：

安装时提示的权限申请是表单文件中静态配置的：

![安装时提示权限申请](/assets/images/android-sd-card/安装时提示权限申请.jpg)

运行时提示的权限申请是代码中动态申请的：

![运行时提示权限申请](/assets/images/android-sd-card/运行时提示权限申请.jpg)

成功写入文件：

![成功写入文件](/assets/images/android-sd-card/成功写入文件.jpg)
