---
title: 【Android】Adapter适配器
date: 2018-07-23 10:01 +0800
categories: [Android]
tags: [android, adapter]
---
* 用于将各种数据以合适的形式显示在View中给用户看，为ListView, Spinner等控件设置数据
* 每行都是一个布局

## 1.ArrayAdapter
构造器：

(1)

```java
public ArrayAdapter(Context context, int resource, List<T> objects);
public ArrayAdapter(Context context, int resource, T[] objects);
```

使用系统提供的布局，resource为布局ID。

ListView可使用android.R.layout.simple_list_item_1（只有一个TextView）

Spinner可使用R.layout.support_simple_spinner_dropdown_item（只有一个TextView，需要添加v7兼容包）、android.R.layout.simple_spinner_item或android.R.layout.simple_spinner_dropdown_item（不需要兼容包）

其他的系统提供的布局：
* simple_list_item1: 单独的一行文本框
* simple_list_item2: 有两个文本框组成
* simple_list_item_checked: 每项都是由一个已选中的列表项
* simple_list_item_multiple_choice: 都带有一个复选框
* simple_list_item_single_choice: 都带有一个单选框

(2)

```java
public ArrayAdapter(Context context, int resource, int textViewResourceId, List<T> objects);
public ArrayAdapter(Context context, int resource, int textViewResourceId, T[] objects);
```

使用自定义的布局，resource为自定义布局ID，如R.id.my_layout，对应res/layout下的布局文件my_layout.xml。

注意：使用自定义的布局必须指定文本框组件的ID（第3个参数），即数据objects放到哪里，否则程序会停止运行。

## 2.SimpleAdapter
构造器：

```java
public SimpleAdapter(Context context, List<? extends Map<String, ?>> data,
            int resource, String[] from, int[] to);
```

* data: Map<String, Object>的列表，每一个Map对应一行，有多个<"key", value>对
* resource: 自定义布局ID，同上
* from: Map中"key"的所有取值集合
* to: 显示数据的控件ID集合

如果from[i]=="key"，则数据data[i].get("key")将被放入控件to[i]
