---
title: 【Android】SharedPreferences
date: 2018-07-26 16:12 +0800
categories: [Android]
tags: [android, shared preferences]
---
* SharedPreferences是Android平台上一个轻量级的存储类，用来保存应用的一些常用配置
* 其原理是通过Android系统生成一个**xml文件**保到：/data/data/包名/shared_prefs目录下，类似键值对的方式来存储数据。

## 获取SharedPreferences对象
(1) 获取默认设置集合：

```java
SharedPreferences sp = PreferenceManager.getDefaultSharedPreferences(this);
```

生成的XML文件名为"包名_preferences.xml"

(2) 获取命名的设置集合：

```java
SharedPreferences sp = getSharedPreferences("filename", MODE_PRIVATE);
```

生成的XML文件名为"filename.xml"，其中模式可以是MODE_PRIVATE, MODE_APPEND, MODE_WORLD_READABLE或MODE_WORLD_WRITEABLE

(3) 获取Activity类名的设置集合：

```java
SharedPreferences sp = getPreferences(MODE_PRIVATE);
```

生成的XML文件名为"Activity类名.xml"

## 写入数据
先使用edit()方法获取Editor对象，然后editor.putXXX()

```java
SharedPreferences.Editor editor = sp.edit();
editor.putString("StringValue", "str");
editor.putInt("IntValue", 123);
editor.putBoolean("BooleanValue", true);
editor.putFloat("FloatValue", 3.1415926f);
editor.putLong("LongValue", 0x7FFFFFFFFFFFFFFFL);
Set<String> set = new HashSet<>();
set.add("element1");
set.add("element2");
set.add("element3");
editor.putStringSet("StringSetValue", set);
// 必须要提交事务
// editor.commit();
editor.apply();
```

结果如下：

```xml
<?xml version='1.0' encoding='utf-8' standalone='yes' ?>
<map>
    <float name="FloatValue" value="3.1415925" />
    <boolean name="BooleanValue" value="true" />
    <long name="LongValue" value="9223372036854775807" />
    <set name="StringSetValue">
        <string>element2</string>
        <string>element1</string>
        <string>element3</string>
    </set>
    <string name="StringValue">str</string>
    <int name="IntValue" value="123" />
</map> 
```

![保存的XML文件](/assets/images/android-shared-preferences/保存的XML文件.png)

## 读取数据
getXXX()（SharedPreferences类的方法）

```java
String getString(String key, String defValue);
int getInt(String key, int defValue);
boolean getBoolean(String key, boolean defValue);
float getFloat(String key, float defValue);
long getLong(String key, long defValue);
Set<String> getStringSet(String key, Set<String> defValues);
Map<String, ?> getAll();
boolean contains(String key);
```

## PreferenceActivity
专门用于实现设置界面

![PreferenceActivity](/assets/images/android-shared-preferences/PreferenceActivity.png)

该类是ListActivity的子类，因此基本界面是一个列表，具体控件由XML文件定义：

```xml
<?xml version="1.0" encoding="utf-8"?>
<PreferenceScreen xmlns:android="http://schemas.android.com/apk/res/android">
    <PreferenceCategory android:title="第1组">
        <Preference
            android:key="preference"
            android:title="纯文本"
            android:summary="副标题" />
        <EditTextPreference
            android:key="editTextPreference"
            android:title="输入编辑框"
            android:summary="点击可输入文字"
            android:dialogTitle="请输入姓名" />
        <CheckBoxPreference
            android:key="checkBoxPreference"
            android:title="复选框"
            android:summary="点击可勾选或取消勾选复选框" />
        <SwitchPreference
            android:key="switchPreference"
            android:title="开关"
            android:summary="点击可打开或关闭开关" />
    </PreferenceCategory>
    <PreferenceCategory android:title="第2组">
        <ListPreference
            android:key="listPreference"
            android:title="单选列表"
            android:summary="点击可弹出单选列表对话框"
            android:dialogTitle="请选择性别"
            android:entries="@array/sexes"
            android:entryValues="@array/sexes" />
        <MultiSelectListPreference
            android:key="multiSelectListPreference"
            android:title="多选列表"
            android:summary="点击可弹出多选列表对话框"
            android:dialogTitle="请选择编程语言"
            android:entries="@array/languages"
            android:entryValues="@array/languages" />
        <RingtonePreference
            android:key="ringtonePreference"
            android:title="系统铃声"
            android:summary="点击可选择系统铃声" />
    </PreferenceCategory>
</PreferenceScreen>
```

标签的功能可对照上图。该文件preferences.xml位于res/xml目录下（需要新建），在Activity的onCreate()方法中只需一句调用`addPreferencesFromResource(R.xml.preferences);`即可。

PreferenceActivity相当于自动管理SharedPreferences数据，这些设置实际上被保存在/data/data/包名/shared_prefs/包名_preferences.xml文件中，下一次打开时自动读取出之前保存的设置数据。每一项设置对应一个名为key属性的数据，输入框、单选框列表->string，开关、单个复选框->boolean，复选框列表->string set

```xml
<?xml version='1.0' encoding='utf-8' standalone='yes' ?>
<map>
    <set name="multiSelectListPreference">
        <string>Java</string>
        <string>C++</string>
        <string>Visual Basic</string>
    </set>
    <string name="editTextPreference">ZZy</string>
    <boolean name="switchPreference" value="true" />
    <string name="listPreference">男</string>
    <boolean name="checkBoxPreference" value="true" />
</map>
```

![PreferenceActivity2](/assets/images/android-shared-preferences/PreferenceActivity2.png)

![请输入姓名](/assets/images/android-shared-preferences/请输入姓名.png)

![请选择性别](/assets/images/android-shared-preferences/请选择性别.png)

![请选择编程语言](/assets/images/android-shared-preferences/请选择编程语言.png)
