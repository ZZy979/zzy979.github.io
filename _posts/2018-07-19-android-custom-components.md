---
title: 【Android】自定义控件
date: 2018-07-19 09:26 +0800
categories: [Android]
tags: [android, gui]
---
* 官方文档：<https://developer.android.google.cn/guide/topics/ui/custom-components>
* 参考：
  * <https://www.jianshu.com/p/cb93ab95acbf>
  * <https://www.jianshu.com/p/705a6cb6bfee>
* 继承View类或其子类
* 覆盖onDraw()方法以自定义控件怎么“画”

```java
public class MyView extends View {
    public MyView(Context context) {
        super(context);
    }

    @Override
    protected void onDraw(Canvas canvas) {

    }
}
```

* 添加自定义控件的方法：
  * 静态添加：在布局文件中

  ```xml
  <com.example.MyView
      android:layout_width="match_parent"
      android:layout_height="wrap_content" />
  ```

  * 动态添加：在代码中

  ```java
  myView1 = new MyView(this);
  linearLayout1.addView(myView1);
  ```

* 自定义属性

(1) 在{工程目录}/app/src/main/res/values/attrs.xml中声明自定义属性

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="MyView">
        <attr name="number" format="integer" />
        <attr name="text" format="string" />
    </declare-styleable>
</resources>
```

(2) 在构造器中获取自定义属性的值

```java
TypedArray a = context.obtainStyledAttributes(attrs, R.styleable.MyView, defStyleAttr, defStyleRes);
number = a.getInteger(R.styleable.MyView_number, 0);
text = a.getBoolean(R.styleable.MyView_text, "");
a.recycle();
```

(3) 在xml中使用自定义属性

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <com.example.MyView
        android:id="@+id/myView1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:number="1"
        app:text="abc" />
 
</LinearLayout>
```

* [保存和恢复状态]({% post_url 2020-02-18-android-saving-states %})
