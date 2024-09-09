---
title: 【Android】使用Room连接SQLite数据库
date: 2020-02-10 01:08 +0800
categories: [Android]
tags: [android, database]
---
* Room是Google官方提供的数据库框架（相当于Android版的Spring Data JPA），官方强烈建议使用Room而不是SQLite API
* Room在SQLite上提供了一个抽象层，可以更方便地访问和操作数据库，大大减少代码量
* 官方文档：<https://developer.android.google.cn/training/data-storage/room>
* 官方示例：<https://github.com/android/architecture-components-samples/tree/master/PersistenceContentProviderSample>
* 参考：
  * <https://blog.csdn.net/wuditwj/article/details/84381112>
  * <http://www.mamicode.com/info-detail-2762152.html>
  * <https://www.jianshu.com/p/145f5e959dfb>

1.在app/build.gradle中添加依赖

```
implementation 'android.arch.persistence.room:runtime:1.1.1'
annotationProcessor 'android.arch.persistence.room:compiler:1.1.1' 
```

2.定义实体类

```java
@Entity
public class User {
    @PrimaryKey
    public int uid;

    @ColumnInfo(name = "first_name")
    public String firstName;

    @ColumnInfo(name = "last_name")
    public String lastName;
}
```

3.定义DAO（即Repository）

```java
@Dao
public interface UserDao {
    @Query("SELECT * FROM user")
    List<User> getAll();

    @Query("SELECT * FROM user WHERE uid IN (:userIds)")
    List<User> loadAllByIds(int[] userIds);

    @Query("SELECT * FROM user WHERE first_name LIKE :first AND " +
           "last_name LIKE :last LIMIT 1")
    User findByName(String first, String last);

    @Insert
    void insertAll(User... users);

    @Delete
    void delete(User user);
}
```

4.定义数据库

由于创建RoomDatabase实例的成本相当高，因此官方推荐使用单例模式

```java
@Database(entities = {User.class}, version = 1)
public abstract class SampleDatabase extends RoomDatabase {
    /** The only instance */
    private static SampleDatabase sInstance;

    public static synchronized SampleDatabase getInstance(Context context) {
        if (sInstance == null) {
            sInstance = Room
                    .databaseBuilder(context.getApplicationContext(), SampleDatabase.class, "database-name")
                    .build();
        }
        return sInstance;
    }

    public static void closeInstance() {
        if (sInstance != null) {
            sInstance.close();
            sInstance = null;
    }

    public abstract UserDao userDao();

}
```
