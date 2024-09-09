---
title: 【Android】保存和恢复状态
date: 2020-02-18 18:55 +0800
categories: [Android]
tags: [android]
---
* 官方文档：
  * <https://developer.android.google.cn/guide/components/activities/activity-lifecycle?hl=zh_cn#saras>
  * <https://developer.android.google.cn/topic/libraries/architecture/saving-states.html?hl=zh_cn>
* 参考：
  * <https://www.jianshu.com/p/c268300749ef>
  * <https://blog.csdn.net/growing_tree/article/details/53759564>

## 1.Activity

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    if (savedInstanceState == null) {
        // 初始化myData
    }
    else
        myData = savedInstanceState.getInt("myData"));

    // ...
} 

@Override
protected void onSaveInstanceState(Bundle outState) {
    outState.putInt("myData", myData);
    super.onSaveInstanceState(outState);
}
```

也可以在onRestoreInstanceState()方法中恢复状态，该方法将在onStart()之后调用

```java
@Override
protected void onRestoreInstanceState(Bundle savedInstanceState) {
    super.onRestoreInstanceState(savedInstanceState);
    myData = savedInstanceState.getInt("myData"));
}
```

## 2.Fragment

```java
@Override
public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
    View rootView = inflater.inflate(R.layout.fragment_my, container, false);

    if (savedInstanceState == null) {
        // 初始化myData
    }
    else
        myData = savedInstanceState.getInt("myData"));

    return rootView;
}

@Override
public void onSaveInstanceState(Bundle outState) {
    outState.putInt("myData", myData);
    super.onSaveInstanceState(outState);
}
```

也可以在onActivityCreated()方法中恢复状态，该方法将在onCreateView()之后调用

```java
@Override
public void onActivityCreated(Bundle savedInstanceState) {
    super.onActivityCreated(savedInstanceState);
    if (savedInstanceState != null)
        myData = savedInstanceState.getInt("myData"));
}
```

* Fragment的状态分为两类：自身状态和包含的View状态
* 当包含的View实现了onSaveInstanceState()方法且在布局文件中有id时，其状态将被自动保存和恢复，否则不会被自动保存，但可以手动实现
* 对于TextView，其mFreezesText决定了文本是否被保存（默认为false），可以在布局文件通过android:freezesText属性或在代码中通过TextView.setFreezesText()方法设置（见TextView.setFreezesText()的文档说明和TextView.onSaveInstanceState()源码）
* 即使将freezesText属性设置为true也只能保存文字内容，对于按钮（TextView的子类），其enabled, visibility等状态仍然不会被自动保存，需要在Activity或Fragment的onSaveInstanceState()方法中手动实现

参考：<https://blog.csdn.net/zephyr_g/article/details/53516568>

## 3.View

```java
@Override
protected Parcelable onSaveInstanceState() {
    Bundle bundle = new Bundle();
    bundle.putParcelable("superState", super.onSaveInstanceState());
    bundle.putInt("myData", myData); 
    return bundle;
}

@Override
protected void onRestoreInstanceState(Parcelable state) {
    Bundle bundle = (Bundle) state;
    super.onRestoreInstanceState(bundle.getParcelable("superState"));
    myData = bundle.getInt("myData");
}
```
