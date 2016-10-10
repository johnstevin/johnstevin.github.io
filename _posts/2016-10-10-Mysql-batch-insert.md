---
layout: post
title: Mysql中INSERT INTO (field1,field2,...) SELECT (field1,field2,...) FROM table语句执行过程中，INNODB引擎会自动创建事物，当终止进程或kill时，会发生rollback。
categories: Mysql
description: Mysql中INSERT INTO (field1,field2,...) SELECT (field1,field2,...) FROM table语句执行过程中，INNODB引擎会自动创建事物，当终止进程或kill时，会发生rollback。
keywords: Mysql,编码,数据库
---

Mysql中INSERT INTO (field1,field2,...) SELECT (field1,field2,...) FROM table语句执行过程中，INNODB引擎会自动创建事物，当终止进程或kill时，会发生rollback。

表现在当你kill后，SHOW PROCESSLIST状态一直为killed僵尸状态，直到rollback完毕。
