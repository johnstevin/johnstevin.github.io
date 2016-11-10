---
layout: post
title: Linux内存清理/释放命令
categories: Linux
description: linux内存清理/释放命令
keywords: Linux,命令
---

## 查看内存使用

```shell
free -m
```

## 优化内存

- cache释放：

To free pagecache:

```shell
echo 1 > /proc/sys/vm/drop_caches
```

To free dentries and inodes:

```shell
echo 2 > /proc/sys/vm/drop_caches
```

To free pagecache, dentries and inodes:

```shell
echo 3 > /proc/sys/vm/drop_caches
```

说明，释放前最好sync一下，防止丢数据。

因为LINUX的内核机制，一般情况下不需要特意去释放已经使用的cache。这些cache起来的内容可以增加文件以及的读写速度。

## vmstat：查看系统负载

## iostat：查看IO负载

## pstree：查看进程

```shell
pstree -p 3020（进程号）
```