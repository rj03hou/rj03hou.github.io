---
layout: post 
title: "Facebook的Online Schema Change"
subTitle: 
heroImageUrl: 
date: 2010-9-29
tags: ["MySQL","mysql","online chema change","未完待续"]
keywords: 
---

在下面这个文章中对Facebook的Online schema change做了概要性的描述：

作者: **[Fenng](http://www.dbanotes.net/)** ' 可以转载, 但必须以超链接形式标明文章原始出处和作者信息及[版权声明](http://creativecommons.org/licenses/by/2.5/cn/)
网址: [http://www.dbanotes.net/opensource/facebook_mysql_online_schema_change.html](http://www.dbanotes.net/opensource/facebook_mysql_online_schema_change.html)

Facebook针对Online schema change在下面网址做出了详细的描述：
[http://www.facebook.com/notes/mysql-at-facebook/online-schema-change-for-mysql/430801045932](http://www.facebook.com/notes/mysql-at-facebook/online-schema-change-for-mysql/430801045932)

代码可以在下面这个地方下载：
[http://bazaar.launchpad.net/~mysqlatfacebook/mysqlatfacebook/tools/annotate/head:/osc/OnlineSchemaChange.php](http://bazaar.launchpad.net/~mysqlatfacebook/mysqlatfacebook/tools/annotate/head:/osc/OnlineSchemaChange.php)

Some ALTER TABLE statements take too long form the perspective of some MySQL users. The [fast index create](http://dev.mysql.com/doc/innodb-plugin/1.0/en/innodb-create-index.html "http://dev.mysql.com/doc/innodb-plugin/1.0/en/innodb-create-index.html") feature for the InnoDB plugin in MySQL 5.1 makes this less of an issue but this can still take minutes to hours for a large table and for some MySQL deployments that is too long.

A workaround is to perform the change on a slave first and then promote the slave to be the new master. But this requires a slave located near the master. MySQL 5.0 added support for triggers and some replication systems have been built using triggers to capture row changes. Why not use triggers for this? The [openarkkit](http://code.google.com/p/openarkkit/ "http://code.google.com/p/openarkkit/") toolkit did just that with oak-online-alter-table. We have published our version of an online schema change utility (OnlineSchemaChange.php aka OSC).

当对线上的大表的schema进行的操作，比如说增删数据列时，会对整个表加上排他锁，而阻塞其他的操作，针对这种情况，目前更多的做法是先在slave上对schema进行更改，然后将slave切换成master，这种做法相对比较麻烦，而且需要slave的延时很少。

Facebook针对这种情况推出了Online Schema Change，先将表copy出来，然后进行更改。

看了一半，后面的看的不太懂，打算对这个再深入看下，先保留着。