---
title: 【Android】SQLite数据库
date: 2018-07-18 11:18 +0800
categories: [Android]
tags: [android, database]
---
* 官方文档：<https://developer.android.google.cn/training/data-storage/sqlite.html>

  注意：官方提供了一个叫作Room的工具，并强烈建议使用Room而不是SQLite API，见[使用Room连接SQLite数据库]({% post_url 2020-02-10-android-connect-to-sqlite-with-room %})
* SQLite：在Android平台上集成的一个嵌入式关系型数据库
* SQLite最大的特点是你可以把各种类型的数据保存到任何字段中，而不用关心字段声明的数据类型是什么。另外，SQLite在解析CREATE TABLE语句时，会忽略CREATE TABLE语句中跟在字段名后面的数据类型信息。

## SQLiteOpenHelper类

```java
public class PersonSQLiteOpenHelper extends SQLiteOpenHelper {
    public PersonSQLiteOpenHelper(Context context) {
        super(context, "person.db", null, 1);
        // 创建或打开数据库
        // 当版本号改变时调用onUpgrade()
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        // 初次创建数据库的操作
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        // 更新数据库的操作，如添加、修改、删除关系表
    }
}
```

* 用于初次使用软件时创建数据库、关系表，通过对数据库进行版本管理实现软件更新时更新数据库以及对数据库进行增删改查操作。
* 该类是抽象类，要使用必须继承它并实现以下两个方法：

```java
void onCreate(SQLiteDatabase db);
```

初次生成数据库时被调用，可以生成数据库表结构及添加一些应用使用到的初始化数据

```java
void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion);
```

在数据库的版本发生变化时会被调用，可以更新数据库表结构。

* 构造器：

```java
SQLiteOpenHelper(Context context, String name, CursorFactory factory, int version)
```
      
如果数据库存在，它就会打开；如果不存在，就会创建这个数据库。
如果version大于当前版本号则调用onUpgrade()，如果小于当前版本号则调用onDowngrade()。

* 获取可操作数据库的方法：

```java
SQLiteDatabase getWritableDatabase();
SQLiteDatabase getReadableDatabase();
```

* 创建的数据库位于data/data/包名/databases/数据库.db

## SQLiteDatabase 类：表示打开的数据库对象
* 使用getWritableDatabase()或getReadableDatabase()方法获得
* 执行SQL语句：

```java
void execSQL(String sql, Object[] bindArgs)
Cursor rawQuery(String sql, String[] selectionArgs)
```

例：
* 插入一条记录：`db.execSQL("INSERT INTO Person VALUES(?,?)", new Object[]{ name, telephone });`
* 根据姓名查询所有的人：`Cursor cursor = db.rawQuery("SELECT * FROM Person WHERE Name=?", new String[]{ name });`

* 注意：为避免SQL注入问题，应使用"?"占位符和Object数组传入参数。

  SQL注入攻击：SQL语句是`"SELECT * FROM Person WHERE Name='"+name+"'"`，而用户输入的name是`"';<其他SQL语句>;--"`，则可以执行任何攻击者希望的SQL语句，如`name="';DELETE FROM Person;--"`，最后的`"--"`将注释掉最后一个单引号。
* 最后使用close()方法关闭数据库
* 该类还专门提供了对应于添加、删除、更新、查询的操作方法：insert(), delete(), update()和query()。例如：

```java
ContentValues values = new ContentValues();
values.put("Name", name);
values.put("Telephone", telephone);
db.insert("Person", null, values);
```

对于熟悉SQL语法的程序员而言，直接使用execSQL()和rawQuery()即可。

* 定义事务：beginTransaction(), endTransaction()，在这两个调用之间加入执行SQL语句的方法

## Cursor接口：游标
* 常用方法：
  * boolean moveToFirst();    移动到第一行，返回移动否成功，下同
  * boolean moveToLast();    移动到最后一行
  * boolean moveToNext();    移动到下一行
  * boolean moveToPrevious();    移动到上一行
  * int getCount();    获得结果集中的行数
  * int getColumnIndex(String columnName);    根据列（属性）名获取列号
  * int getInt(int columnIndex);    根据列号（从0开始）获取整型属性值，其他类型类似
* 典型代码：

```java
SQLiteDatabase db = helper.getReadableDatabase();
Cursor cursor = db.rawQuery("SELECT * FROM Person", null);
List<Person> persons = new ArrayList<>();
while (cursor.moveToNext()) {
    int id = cursor.getInt(0);
    // 或cursor.getInt(cursor.getColumnIndex("ID"))
    String name = cursor.getString(1);
    String telephone = cursor.getString(2);
    persons.add(new Person(id, name, telephone));
}
cursor.close();
db.close();
```
