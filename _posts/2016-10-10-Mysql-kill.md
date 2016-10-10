---
layout: post
title: MYSQL-5.6.x的kill进程的一些坑
categories: Mysql
description: MYSQL-5.6.x的kill进程的一些坑
keywords: Mysql,编码,数据库
---

1. 直接kill掉进程后，information_schema.TABLES里的数据不能及时同步更新(也就是说在第三方Mysql连接工具查看表状态时的数据)。

......
