---
title: 【Android】项目：备份短信
date: 2018-07-28 14:35 +0800
categories: [Android]
tags: [android]
---
* 短信结构：时间、类型（收/发）、内容、发送者/接收者号码
* 短信存储位置：/data/data/com.android.providers.telephony/databases/mmssms.db
* 数据表结构：sms(_id, thread_id, address, person, date, date_sent, protocol, read, status, type, reply_path_present, subject, body, service_center, locked, sub_id, error_code, creator, seen)
* 获取短信必须使用ContentProvider，uri=content://sms
* 权限：android.permission.READ_SMS

未解决的问题：短信太多会导致Activity跳转的时候闪退。在真实手机上测试时，共有2589条短信，可以备份成功，但读取时直接闪退。经调试发现闪退发生在调用startActivity()之后（可以正确读取）、被调用Activity的onCreate()方法执行之前。经试验，读取的短信条数≤1916时可以正常显示，≥1917就会闪退，原因未知。

![备份结果](/assets/images/android-project-sms-backup/备份结果.png)

![备份结果2](/assets/images/android-project-sms-backup/备份结果2.png)
