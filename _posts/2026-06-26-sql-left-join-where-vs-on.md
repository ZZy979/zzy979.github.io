---
title: 【SQL】LEFT JOIN查询中WHERE和ON的区别
date: 2026-06-26 14:37 +0800
categories: [Database]
tags: [sql, left join]
---
## 1.问题
假设有下面的用户表和订单表：

```sql
CREATE TABLE Users (
    user_id INT,
    name VARCHAR(50),
    age INT,
    PRIMARY KEY (user_id)
);

CREATE TABLE Orders (
    order_id INT,
    user_id INT,
    create_time DATETIME,
    PRIMARY KEY (order_id)
);
```

测试数据如下：

```sql
INSERT INTO Users VALUES
(1, 'Alice', 30),
(2, 'Bob', 35),
(3, 'Cindy', 25);

INSERT INTO Orders VALUES
(100, 1, '2026-06-01 00:00:00'),
(101, 3, '2026-06-02 00:00:00');
```

现在需要查询**年龄≥30且在2026年6月1日无订单的用户姓名**，预期结果只包含 "Bob" 。

### 错误写法1
下面的SQL是错误的：

```sql
SELECT DISTINCT Users.name
FROM Users
LEFT JOIN Orders ON Users.user_id = Orders.user_id
WHERE Users.age >= 30
    AND DATE(Orders.create_time) = '2026-06-01'
    AND Orders.order_id IS NULL;
```

实际查询结果为空。

## 2.核心原理
为了弄清楚上述问题产生的原因，需要理解SQL查询的逻辑执行顺序：
1. `FROM`（包括`JOIN`）
2. `ON`（连接条件，生成临时表）
3. `WHERE`（过滤行）
4. `SELECT`

优化器实际执行时可能会调整执行顺序，但最终结果必须与逻辑语义一致。

逻辑上，**WHERE子句是在LEFT JOIN之后执行的**。MySQL文档的[JOIN Clause](https://dev.mysql.com/doc/refman/8.4/en/join.html)一节有如下说明：

> Generally, the ON clause serves for conditions that specify how to join tables, and the WHERE clause restricts which rows to include in the result set.

也就是说，`ON`指定连接条件，`WHERE`指定结果集的过滤条件。如果`WHERE`条件中包含右表(`Orders`)的字段，会破坏`LEFT JOIN` “保留左表全部记录”的语义，实际上把它变成了`INNER JOIN`。

在前面的SQL中，问题出在`AND DATE(Orders.create_time) = '2026-06-01'`这一行。根据`ON`指定的条件，`LEFT JOIN`的结果包含以下3行记录，但均由于不同原因被过滤掉：

| name | age | order_id | create_time | 过滤原因 |
| --- | --- | --- | --- | --- |
| Alice | 30 | 100 | 2026-06-01 00:00:00 | ❌订单id不是NULL |
| Bob | 35 | NULL | NULL | ❌订单创建日期不是2026-06-01 |
| Cindy | 25 | 101 | 2026-06-02 00:00:00 | ❌年龄小于30 |

注：可以通过删除`WHERE`子句得到上面的中间结果

```sql
SELECT * FROM Users LEFT JOIN Orders ON Users.user_id = Orders.user_id;
```

在SQL中，任何包含`NULL`的表达式结果都是`NULL`（`IS NULL`/`IS NOT NULL`除外）。当`Orders.create_time`为`NULL`时，`DATE()`和`=`均返回`NULL`。而`NULL`值在`WHERE`过滤中被视为`false`，因此Bob那一行也被过滤掉，导致最终结果为空。

实际上，`DATE(Orders.create_time) = '2026-06-01'`和`Orders.order_id IS NULL`是互斥条件，不可能同时满足。

### 正确写法
正确写法是将右表条件移至`ON`子句：

```sql
SELECT DISTINCT Users.name
FROM Users
LEFT JOIN Orders ON Users.user_id = Orders.user_id
    AND DATE(Orders.create_time) = '2026-06-01'
WHERE Users.age >= 30
    AND Orders.order_id IS NULL;
```

```
+------+
| name |
+------+
| Bob  |
+------+
```

这样订单创建时间是连接条件而不是过滤条件，`LEFT JOIN`结果如下，因此能得到正确结果 "Bob" 。

| name | age | order_id | create_time | 过滤原因 |
| --- | --- | --- | --- | --- |
| Alice | 30 | 100 | 2026-06-01 00:00:00 | ❌订单id不是NULL |
| Bob | 35 | NULL | NULL | ✅保留 |
| Cindy | 25 | NULL | NULL | ❌年龄小于30 |

虽然条件`Users.age >= 30`的本意是对用户进行前置过滤，但`WHERE`子句是在`LEFT JOIN`完成后统一执行的，不会在`JOIN`前后分别执行一部分。不过，现代SQL优化器可能会做“谓词下推”优化，即在执行计划中将部分过滤条件提前执行。例如，`Users.age >= 30`可能会被下推到扫描`Users`表时执行以加快查询，但这不会改变最终的语义结果。

### 错误写法2
对于`LEFT JOIN`，左表(`Users`)的过滤条件放在`WHERE`和`ON`中的效果完全不同。下面是另一种错误写法：

```sql
SELECT DISTINCT Users.name
FROM Users
LEFT JOIN Orders ON Users.user_id = Orders.user_id
    AND Users.age >= 30
    AND DATE(Orders.create_time) = '2026-06-01'
WHERE Orders.order_id IS NULL;
```

```
+-------+
| name  |
+-------+
| Bob   |
| Cindy |
+-------+
```

结果 "Cindy" 也被返回了。根本原因是`ON`子句仅决定如何匹配右表，无法起到过滤左表的作用。

| name | age | order_id | create_time | 过滤原因 |
| --- | --- | --- | --- | --- |
| Alice | 30 | 100 | 2026-06-01 00:00:00 | ❌订单id不是NULL |
| Bob | 35 | NULL | NULL | ✅保留 |
| Cindy | 25 | NULL | NULL | ✅保留（错误） |

## 3.总结
三种写法的对比如下表所示：

| 写法 | 左表条件位置 | 右表条件位置 | 查询结果 |
| --- | --- | --- | --- |
| 错误写法1 | WHERE | WHERE | 空集 ❌ |
| 正确写法 | WHERE | ON | Bob ✅ |
| 错误写法2 | ON | ON | Bob,Cindy ❌ |

下表总结了在`LEFT JOIN`查询中将过滤条件放在`ON`和`WHERE`子句中的区别：

| 条件位置 | 对左表过滤 | 对右表过滤 |
| --- | --- | --- |
| ON | ❌ 不过滤左表记录，只影响连接匹配 | ✅ 不影响左表保留 |
| WHERE | ✅ 会过滤左表记录 | ❌ 会破坏LEFT JOIN语义 |

经验法则：
* 左表过滤条件放在`WHERE`
* 右表过滤条件放在`ON`
* 匹配判断(`IS NULL`)放在`WHERE`
