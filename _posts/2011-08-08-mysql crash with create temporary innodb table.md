---
layout: post 
title: "mysql crash with create temporary innodb table"
subTitle: 
heroImageUrl: 
date: 2011-8-8
tags: ["MySQL"]
keywords: 
---

mysql5.5中create tempory table with innodb engine会造成mysqld crash掉
mysql@>create temporary table test1(id int) engine=innodb;
Query OK, 0 rows affected (0.00 sec)

mysql@>show create table test1;
ERROR 2006 (HY000): MySQL server has gone away
No connection. Trying to reconnect...
Connection id:    1
Current database: fb

ERROR 1146 (42S02): Table 'fb.test1' doesn't exist

error log中的信息如下：
110808 16:20:35  InnoDB: Database was not shut down normally!
InnoDB: of table 110808 16:20:38 [Warning] Invalid (old?) table or database name '#sql4619_1_0'
"shm".<result 2 when explaining filename '#sql4619_1_0'>.
110808 16:20:38  InnoDB: 1.1.4 started; log sequence number 154250481442
110808 16:20:38 [Note] Recovering after a crash using fb-bin
110808 16:20:38 [Note] Starting crash recovery...
110808 16:20:38 [Note] Crash recovery finished.
110808 16:20:38  InnoDB: Error: table 110808 16:20:38 [Warning] Invalid (old?) table or database name '#sql4619_1_0'
`shm`.<result 2 when explaining filename '#sql4619_1_0'> does not exist in the InnoDB internal

如果engine=myisam的话就没有问题，如果把tmp_dir由tmpfs修改为物理磁盘的话也不会有问题，更深层的原因现在还不知道，已经有人发现了这个问题，并且作为一个bug提交了。

mysql的temporary table是在线程存活期中有效的，当线程close掉的时候，temporary table会被drop掉，另外创建一个同名的temporary table会将同名的非temporary隐藏起来，线程结束之后显影。另外因为是跟线程关联的，所以多个线程的临时表名称可以冲突。

翻译来自这边文章：http://ronaldbradford.com/blog/tag/tmpfs/
mysql bug：http://bugs.mysql.com/bug.php?id=26662