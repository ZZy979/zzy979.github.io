---
title: 【暑期实习面经】字节跳动-穿山甲-后端开发（接受offer）
date: 2021-05-08 13:15:58 +0800
categories: [面经]
tags: [interview]
---
## 一面
2021年3月23日  
1小时  
自我介绍

线程有哪些状态  
什么是死锁，如何恢复和避免

算法题  
1.长度为n+1的数组，元素为1~n，有一个数字出现了两次，找出出现两次的数字  
要求：时间复杂度O(n)，空间复杂度O(1)  
方法1：sum(a) - sum(range(1, n + 1)) 问题：可能溢出  
方法2：xor(a) ^ xor(range(1, n + 1)) 其中xor表示将所有元素异或起来

2.判断一棵二叉树是否是另一棵二叉树的子树  
方法1：递归，**时间复杂度？** 极端情况：只有叶节点不同，O(mn)  
方法2：中序遍历+子串判断 **KMP算法**

3.**最长递增子序列**  
动态规划（没想出来）  
（力扣300）

简历项目  
**ES的底层实现**（不了解）

计网：TCP三次握手四次挥手，TCP和UDP的区别

数据库：MySQL存储引擎，索引的优缺点，MySQL索引类型（B+树、哈希，用B+树的好处，**为什么不用B树/红黑树？**），事务的四个特性

评价：还可以，要注重底层原理

## 二面
2021年3月28日  
1小时

自我介绍  
简历项目

编程  
1.Python yield语句  
2.字典的底层结构（不知道）（使用伪随机探测的散列表）  
3.Java的HashMap如何实现（数组+链表/红黑树）  
4.Python中如何创建一个进程（Process类），进程如何通信  
5.一个线程从数据库读取URL，10个线程负责爬取，如何设计（生产者消费者模式，信号量？）→Java线程池，创建的方法和核心参数  
6.垃圾收集算法CMS/G1（不知道）  
7.SpringBoot启动过程（不知道）

计网  
1.输入网址到显示页面的过程  
2.DNS解析之后如何根据IP地址找到主机（路由器路由功能，转发表）  
3.如何处理HTTP请求（Web服务器：匹配URL，调用视图函数，匹配不上的404）

数据库  
1.事务
```
ID ｜ AGE 
1 ｜ 10

A                      B
---------begin   -----------begin
select -> 10
                      update -> 20
                     -----------commit
select -> ?
-----------commit
```
可重复读，仍然是10  
此时A是否可以更新？有冲突？

2.索引
```
ID ｜ AGE ｜ NAME(uniq_key)
1 ｜ 10  ｜ zhangsan
```
主键索引和unique索引的性能差别？（都是B+树，性能一样）

3.主从同步（不了解）

操作系统  
用户态、内核态的区别，进程如何从用户态进入内核态（忘了）（系统调用、异常、外部设备中断）

编程  
1.给定一个无序数组，查找第一个缺失的正整数  
```
Example 1:
Input: 1,2,0 Output: 3
Example 2:
Input: 3,4,-1,1 Output: 2
Example 3:
Input: 7,8,9,11,12 Output: 1
```
散列表：时间复杂度O(n)，空间复杂度O(n)  
排序：时间复杂度O(nlog n)，空间复杂度O(1)  
还能优化空间吗→散列表的元素改成区间，难点是如何合并→bitset  
（力扣41原题！）

2.给定一个二叉树和一个数字n，判断二叉树中是否有一个路径上的数字之和等于给定的数字n  
如果要输出路径如何做（增加path参数）

## 三面
2021年3月31日  
30分钟

自我介绍

简历项目：数据处理  
1.导出原始数据时如何验证，写伪代码
```python
def valid(paper, validators):
    for key in validators:
        if not validators[key](paper[key]):
            return False
    return True

if __name__ == '__main__':
    paper = {'id': 1, 'title': 'foo', 'keywords': 'a;b;c', 'authors': 'a[A];b[B]'}
    validators = {
        'title': lambda s: len(s) > 0,
        'authors': lambda s: re.fullmatch(r'([A-Za-z]+\[[A-Za-z]+\];)*[A-Za-z]+\[[A-Za-z]+\]', s)
    }
    print(valid(paper, validators))
```

2.如何从MySQL导入图数据库，写伪代码
```
class Paper:
    id 
    title
    authors: str

class PaperVertex:
    id
    title
    authors: List[AuthorDO]

class EntityUtils:
    def paper2paperVertex(paper):
        authors = paper.authors.split(';')
        paperVertex
        vertex_id = g.addV().property(...).next()
        paper.vertex_id = vertex_id
```

3.ES的原理（不了解），如果让你设计会怎么设计（B+树）

反问

## HR面
2021年4月1日  
10分钟

自我介绍  
遇到过的困难  
职业规划  
业务方向：后端开发  
有没有面其他公司，拿到offer会怎么选择  
实习时间（出勤要求：每周4~5天，大小周）
