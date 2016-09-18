---
layout: post 
title: "mysql跳过大量replication错误的几种方法"
subTitle: 
heroImageUrl: 
date: 2011-11-3
tags: ["MySQL"]
keywords: 
---

mysql跳过大量replication错误的几种方法(错误大家都不想见到，但是见到了也得想办法处理)：

1，使用pt-slave-restart脚本
地址：http://www.percona.com/doc/percona-toolkit/pt-slave-restart.html
推荐使用这种方法，有诸多优点：快速，可以使用error-numbers选项跳过指定的error code

2，使用set sql_slave_skip_counter
使用watch -n 1 "mysql -hxxx -uxx -pxx -e'set global sql_slave_skip_counter=1;start slave'"
但是有诸多缺点，跟pt-slave-restart相比，pt的优点就是他的缺点，因为他-n的最小单位是秒，因此速度较慢。
注意：可能有人会对这个错误产生疑问，提示一下set global sql_slave_skip_counter=1在mysql replication运行正常的情况下会报错误，提示无法执行。
另外使用sql_slave_skip_counter的时候会在error log中记录每一条跳过的语句，可以通过分析error log来调查原因或者进行修补。

3，使用mysql variable slave_skip_errors
缺点：不是Dynamic的，不能动态修改，必须重启服务器
参数说明：Normally, replication stops when an error occurs on the slave. This gives you the opportunity to resolve the inconsistency in the data manually. This variable tells the slave SQL thread to continue replication when a statement returns any of the errors listed in the variable value.

4，使用slave_exec_mode
是Dynamic的，可以动态修改，但是只能跳过duplicate-key and no-key-found errors
参数说明：Controls whether IDEMPOTENT or STRICT mode is used in replication conflict resolution and error checking. IDEMPOTENT mode causes suppression of duplicate-key and no-key-found errors. Beginning with MySQL 5.1.23-ndb-6.2.14 and MySQL 5.1.24, this mode should be employed in multi-master replication, circular replication, and some other special replication scenarios. STRICT mode is the default, and is suitable for most other cases.