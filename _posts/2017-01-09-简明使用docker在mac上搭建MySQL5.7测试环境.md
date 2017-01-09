---
layout: single
title: 简明使用docker在mac上搭建MySQL5.7测试环境
description: 简明使用docker在mac上搭建MySQL5.7测试环境
headline:
categories: mysql
headline:
tags: [docker,  mysql]
comments: true
published: true
---

MySQL5.7已经GA很久了，但是很多人还没有进行尝鲜，也没有在本地做一些功能上的测试研究；本文主要介绍了如何使用docker在mac下，快速搭建一个MySQL或者Percona的5.7功能测试环境。

### 1.mac下docker安装

下载安装Docker for Mac

[https://download.docker.com/mac/stable/Docker.dmg](https://download.docker.com/mac/stable/Docker.dmg)

如果之前安装过virtualbox，需要升级virtualbox或者卸载virtualbox

卸载virtualbox流程: 进入Applications目录，右键点击virtualbox，选择move to trash，另外需要重启系统

### 2.docker image创建

获取Percona5.7的 image，如果打算测试官方5.7，则docker pull mysql:5.7就可以。

```shell
bash> docker pull percona:5.7
```

### 3.启动container（image是镜像，container是实际运行的instance）

设置mysql的root密码为空，并启动docker

```shell
docker run --name percona57 -e MYSQL_ALLOW_EMPTY_PASSWORD=yes -d percona:5.7
```

设置mysql的root密码，并启动docker

```shell
docker run --name percona57 -e MYSQL_ROOT_PASSWORD=yourpasswd -d percona:5.7
```

可以通过docker ps查看镜像

percona的image中包含了很多选项，上面仅仅演示最简单的，详细的可以查看[官方说明](https://hub.docker.com/_/percona/)

### 4.进入container并且连接mysql进行各种功能测试

```bash
docker exec -it percona57 bash
root@46cce1dfe38c:/# mysql
```



是不是很简单，祝玩的愉快。

