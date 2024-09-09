---
title: 【Android】关于androidTest的问题
date: 2018-07-22 22:10 +0800
categories: [Android]
tags: [android]
---
在运行Android测试用例时报错：

```
Conflict with dependency 'com.android.support:support-annotations' in project ':app'. Resolved versions for app (26.1.0) and test app (27.1.1) differ. See https://d.android.com/r/tools/test-apk-dependency-conflicts.html for details.
```

原因是项目的SDK版本为26.1.0，而测试用例版本为27.1.1
* 解决方法1：将项目的SDK版本改为27.1.1或以上
* 解决方法2：修改模块的（不是工程的）build.gradle文件（位于项目目录下的app文件夹中），在android{...}中增加以下内容：

```
configurations.all {
  resolutionStrategy.force 'com.android.support:support-annotations:26.1.0'
}
```

然后同步项目（File->Sync Project with Gradle Files或直接点击右上角的提示）即可

参考：<https://www.jianshu.com/p/b6f4baa4e9e4>

![修复androidTest问题](/assets/images/android-test-problem/修复androidTest问题.png)
