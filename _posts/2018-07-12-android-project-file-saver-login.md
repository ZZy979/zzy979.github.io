---
title: 【Android】项目：使用文件保存用户信息
date: 2018-07-12 11:21 +0800
categories: [Android]
tags: [android, file access]
---
* 数据文件位置：data/data/包名/files/info.txt
* 打开/创建文件：
  * 直接指定全路径：

  ```java
  File f = new File("data/data/com.zzy.filesaverlogin/files", "info.txt");
  ```

  如果写成new File("data/data/com.zzy.filesaverlogin/files/info.txt")则保存失败？

  * 通过Context上下文类的getFilesDir()方法获取文件的路径：

  ```java
  File f = new File(context.getFilesDir(), "info.txt");
  ```

  * 保存在缓存目录：context.getCacheDir()
  * 使用Context类的方法：FileOutputStream fos = context.openFileOutput("info.txt", Context.MODE_PRIVATE);

    该方法可用的几种模式：
    * (1) 私有：Context.MODE_PRIVATE，文件权限为-rw-rw----
    * (2) 可读：Context.MODE_WORLD_READABLE，文件权限为-rw-rw-r--
    * (3) 可写：Context.MODE_WORLD_WRITEABLE，文件权限为-rw-rw--w-
    * (4) 共有（可读+可写）：Context.MODE_WORLD_READABLE + Context.MODE_WORLD_WRITEABLE，文件权限为-rw-rw-rw-

    注意“可读”、“可写”是对任何其他应用来说的，一般数据文件都应为私有
* 写文件：

```java
FileOutputStream fos = new FileOutputStream(f);
fos.write(string.getBytes());
fos.close();
```

* 读文件：

```java
FileInputStream fis = new FileInputStream(file);
BufferedReader br = new BufferedReader(new InputStreamReader(fis));
String str = br.readLine();
```

![登录界面](/assets/images/android-project-file-saver-login/登录界面.png)
