---
layout: post 
title: "关于mysql varchar长度"
subTitle: 
heroImageUrl: 
date: 2011-4-27
tags: ["MySQL","吹毛求疵"]
keywords: 
---

本文是对row_format为compact时，varchar长度的一个探讨。
《MySQL技术内幕 InnoDB存储引擎》中姜承尧提到MySQL varchar最大长度65535是指所有的varchar长度累加必须小于65535，这篇文章对这个进行一个探讨，发现65535中应该包含了所有字段的长度、变长字段长度标示位、NULL标示位的累计。在此感谢姜承尧的《MySQL技术内幕 InnoDB存储引擎》，对很多东西有了一个更清晰的了解。

注：下面的测试是
character_set=utf8
utf8下面一个字符占3个字节，因此最大是65535/3=21845，但是21845之后没有地方存储长度信息，所以单列的最大长度是21844

Every table (regardless of storage engine) has a maximum row size of 65,535 bytes。可以通过下面的测试有个了解。
<pre lang="sql">
show table status like "test";
Row_format:Compact

CREATE TABLE `test` (
  `c1` varchar(21844) COLLATE utf8_bin NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin

mysql> alter table test modify column c1 varchar(21844) not null; 
Query OK, 7 rows affected (0.07 sec)
Records: 7  Duplicates: 0  Warnings: 0

mysql> alter table test modify column c1 varchar(21845) not null; 
ERROR 1118 (42000): Row size too large. The maximum row size for the used table type, not counting BLOBs, is 65535. You have to change some columns to TEXT or BLOBs
mysql> select 21844*3+2;
+-----------+
' 21844*3+2 '
+-----------+
'     65534 '
+-----------+
1 row in set (0.00 sec)
</pre>
**因为null标示位占用了一个字节，所以可以去掉not null限制。**
<pre lang="sql">
mysql> alter table test modify c1 varchar(21844) COLLATE utf8_bin; 
Query OK, 7 rows affected (0.08 sec)
Records: 7  Duplicates: 0  Warnings: 0

mysql> alter table test add column i1 int not null;
ERROR 1118 (42000): Row size too large. The maximum row size for the used table type, not counting BLOBs, is 65535. You have to change some columns to TEXT or BLOBs

mysql> alter table test modify column c1 varchar(21843) not null;   
Query OK, 7 rows affected (0.08 sec)
Records: 7  Duplicates: 0  Warnings: 0

mysql> alter table test add column i1 int not null;              
Query OK, 7 rows affected (0.08 sec)
Records: 7  Duplicates: 0  Warnings: 0

mysql> alter table test add column i2 int not null; 
ERROR 1118 (42000): Row size too large. The maximum row size for the used table type, not counting BLOBs, is 65535. You have to change some columns to TEXT or BLOBs

mysql> select 21843*3+2+4;
+-------------+
' 21843*3+2+4 '
+-------------+
'       65535 '
+-------------+
1 row in set (0.00 sec)
</pre>
**这个时候去掉not null就不可以了**
<pre lang="sql">
mysql> alter table test modify column c1 varchar(21843);           
ERROR 1118 (42000): Row size too large. The maximum row size for the used table type, not counting BLOBs, is 65535. You have to change some columns to TEXT or BLOBs
</pre>
参照如下：
1，http://dev.mysql.com/doc/refman/5.0/en/column-count-limit.html
2，MySQL技术内幕 InnoDB存储引擎