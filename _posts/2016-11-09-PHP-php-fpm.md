---
layout: post
title: php-fpm重启
categories: PHP
description: php-fpm重启
keywords: Linux,命令,php
---

## 重启 php-fpm

我们在新安装扩展后，是需要重新php-fpm的，已使扩展生效。

最简单粗暴的重新php-fpm的方式是：

先找到php-fpm的进程号，kill 掉，再用/usr/local/php/sbin/php-fpm 这样启动。

其实还有更多温和的方法，就是使用信号。

- INT, TERM 立刻终止
- QUIT 平滑终止
- USR1 重新打开日志文件
- USR2 平滑重载所有worker进程并重新载入配置和二进制模块

示例：

php-fpm 关闭：
```shell
kill -INT `cat /usr/local/php/var/run/php-fpm.pid`
```
php-fpm 重启：
```shell
kill -USR2 `cat /usr/local/php/var/run/php-fpm.pid`
```