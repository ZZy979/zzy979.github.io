---
title: 《Java核心技术》笔记 卷I 第1章 Java概述
date: 2024-07-13 11:54:07 +0800
categories: [Java, Core Java]
tags: [java]
---
## 1.2 Java“白皮书”的关键术语
Java的设计者编写了颇有影响力的“白皮书”，用来解释设计初衷和完成情况。他们还发布了一个简短的摘要，这个摘要按照下面11个关键术语进行组织：
1. 简单性(Simple)
2. 面向对象(Object-Oriented)
3. 分布式(Distributed)
4. 健壮性(Robust)
5. 安全性(Secure)
6. 体系结构中立(Architecture-Neutral)
7. 可移植性(Portable)
8. 解释性(Interpreted)
9. 高性能(High-Performance)
10. 多线程(Multithreaded)
11. 动态性(Dynamic)

注释：白皮书可以在 <https://www.oracle.com/java/technologies/language-environment.html> 找到。对于11个关键术语的概述参见 <https://horstmann.com/corejava/java-an-overview/7Gosling.pdf> 。

## 1.4 Java发展简史

| 版本 | 年份 | 语言新特性 |
| --- | --- | --- |
| 1.0 | 1996 | 语言本身 |
| 1.1 | 1997 | 内部类 |
| 1.2 | 1998 | `strictfp`修饰符 |
| 1.3 | 2000 | 无 |
| 1.4 | 2002 | 断言 |
| 5.0 | 2004 | 泛型类、 "for each" 循环、可变参数、自动装箱、元数据、枚举、静态导入 |
| 6 | 2006 | 无 |
| 7 | 2011 | 基于字符串的`switch`、菱形运算符、二进制字面值、异常处理增强 |
| 8 | 2014 | Lambda表达式、接口默认方法、流和日期/时间库 |
| 9 | 2017 | 模块、JShell、其他的语言和类库增强 |
| 11 | 2018 | 局部变量类型推断(`var`)，HTTP客户端，移除Java FX、JNLP、Java EE重叠和CORBA |
| 17 | 2021 | `switch`表达式、文本块、`instanceof`模式匹配、记录、密封类 |
