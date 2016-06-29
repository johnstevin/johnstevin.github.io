---
layout: post
title: Docker常用命令(一)
categories: Tools
description: Docker一些简单常用的命令
keywords: docker，开发，工具
---

提交更新并创建副本

<code>
sudo docker commit -m "test commit" -a "Stevin.John" 17ab11e81414 johnstevin/centos:testv1
<code>

镜像推送到Docker Hub上面

<code>
sudo docker push johnstevin/centos
<code>

使用Dockerfile来生成镜像

<code>
sudo docker build -t="johnstevin/centos:testv2" /path/to/Dockerfile
<code>

导出镜像到本地文件

<code>
sudo docker save -o centos_v7_testv3.tar johnstevin/centos:testv3
<code>

导入本地文件到镜像，将导入镜像以及其相关的元数据信息（包括标签等）

<code>
sudo docker load < centos_v7_testv3.tar
<code>

启动已经终止的容器containers

<code>
sudo docker start -i e4942b75734d
<code>

终止正在运行的容器

<code>
sudo docker stop <CONTAINERS>
<code>

查看运行的容器

<code>
sudo docker ps -a
<code>