---
title: 【Android】ContentProvider内容提供者
date: 2018-07-18 11:18 +0800
categories: [Android]
tags: [android, content provider]
---
* 参考：<https://blog.csdn.net/carson_ho/article/details/76101093>
* 用于进程间进行数据交互和共享，即跨进程通信；也适用于进程内通信。

![ContentProvider](/assets/images/android-content-provider/ContentProvider.png)

* 功能：
  * (1) 接收用户需要查询其他应用程序的的uri
  * (2) 将获取的uri传送该数据源应用程序
  * (3) 如果uri存在，则用户可以获得数据源应用程序的数据
  * (4) 如果uri保存在，则不能获取数据源应用程序数据
  * (5) 根据获取的uri完成对数据源应用程序的CURD
## URI
* Uniform Resource Identifier，统一资源标识符，用于唯一标识 ContentProvider和其中的数据
* 系统提供的URI：
  * <https://blog.csdn.net/kingnity/article/details/52505984>
  * <https://blog.csdn.net/lo5sea/article/details/38308513>
* 自定义URI格式：content://授权信息/表名/记录
* 例如自定义CURD操作的4中URI：

```
content://com.zzy.myprovider/insert——插入
content://com.zzy.myprovider/delete——删除
content://com.zzy.myprovider/update——更新
content://com.zzy.myprovider/query——查询
```

* 获取URI对象：Uri uri = Uri.parse("content://授权信息/表名/记录")

## MIME数据类型
包含两部分的字符串，用于指定某个扩展名的文件用某种应用程序来打开
* ContentProvider根据URI返回MIME类型：String getType(Uri uri)
* 每种MIME类型由两部分组成：类型+子类型，如text/html, text/css, text/xml, application/pdf
* 单条记录：vnd.android.cursor.item/自定义
* 多条记录（集合）：vnd.android.cursor.dir/自定义

## ContentProvider类
* 4个核心方法：insert(), delete(), query(), update()
* 2个其他方法：onCreate()--第一次访问该ContentProvider时由系统进行调用, getType()--返回当前Uri所代表数据的MIME类型

```java
public class MyContentProvider extends ContentProvider {
    public MyContentProvider() {
    }
    @Override
    public boolean onCreate() {
        // TODO: Implement this to initialize your content provider on startup.
        return false;
    }
    @Override
    public Uri insert(Uri uri, ContentValues values) {
        // TODO: Implement this to handle requests to insert a new row.
    }
    @Override
    public int delete(Uri uri, String selection, String[] selectionArgs) {
        // TODO: Implement this to handle requests to delete one or more rows.
    }
    @Override
    public Cursor query(Uri uri, String[] projection, String selection,
                        String[] selectionArgs, String sortOrder) {
        // TODO: Implement this to handle query requests from clients.
    }
    @Override
    public int update(Uri uri, ContentValues values, String selection,
                      String[] selectionArgs) {
        // TODO: Implement this to handle requests to update one or more rows.
    }
    @Override
    public String getType(Uri uri) {
        // TODO: Implement this to handle requests for the MIME type of the data at the given URI
    }
}
```

* 该组件是为应用程序提供另一个项目的数据，必须在表单文件中配置：

```xml
<provider
    android:name=".MyContentProvider"
    android:authorities="com.zzy.myprovider"
    android:exported="true" />
```

## ContentResolver类
统一管理不同ContentProvider间的操作
* ContentResolver类提供了与ContentProvider类相同名字和作用的4个方法：insert(), delete(), query(), update()
* 使用方法：

①获取URI对象：

```java
Uri uri = Uri.parse("content://com.zzy.myprovider/query");
```

注意URI路径中的"query"是自定义的，用于在ContentProvide类中使用匹配器得到返回码，不是指定调用query()方法

②在Activity中获取ContentResolver对象：

```java
ContentResolver resolver = getContentResolver();
```

③调用ContentResolver的query()方法：

```java
Cursor cursor = resolver.query(uri, null, null, null, null);
```

该调用将通过解析uri得到授权信息指定的ContentProvide，然后调用该provider的query()方法

Android提供了3个用于辅助ContentProvide的工具类：ContentUris, UriMatcher和ContentObserver

## ContentUris类
用于操作URI
* 核心方法：withAppendedId()--向URI追加一个id，parseId()--从URL中获取ID

## UriMatcher类
在ContentProvider中注册URI、根据URI匹配ContentProvider中对应的数据表
* 初始化

```java
UriMatcher matcher = new UriMatcher(UriMatcher.NO_MATCH);
```

* 在ContentProvider中注册URI：

```java
matcher.addURI("com.zzy.myprovider", "table", URI_CODE);
// 若URI资源路径=content://com.zzy.myprovider/table ，则返回注册码URI_CODE
```

* 根据URI匹配URI_CODE：

```java
public String getType(Uri uri) {
    switch(matcher.match(uri)) {
        case URI_CODE:
            return tableName;
            break;
    }
}
```

## ContentObserver类
内容观察者，用于观察Uri引起ContentProvider中的数据变化、通知外界（即访问该数据访问者）。当ContentProvider中的数据发生变化（增、删、改）时，就会触发该ContentObserver类。
* 注册内容观察者：getContentResolver().registerContentObserver(uri);
* 当该URI的ContentProvider数据发生变化时，通知外界（即访问该ContentProvider数据的访问者）：getContext().getContentResolver().notifyChange(uri, null);
* 解除观察者：getContentResolver().unregisterContentObserver(uri);

## 应用
无论是同一进程还是其他进程，都可以使用ContentResolver通过ContentProvider规定的URI访问数据。典型代码：

```java
ContentResolver resolver = getContentResolver();
Uri uri = Uri.parse("content://com.zzy.myprovider/query");
Cursor cursor = resolver.query(uri, null, null, null, null);
```
