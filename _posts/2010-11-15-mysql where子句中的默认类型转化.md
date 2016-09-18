---
layout: post 
title: "mysql where子句中的默认类型转化"
subTitle: 
heroImageUrl: 
date: 2010-11-15
tags: ["MySQL","mysql","原创"]
keywords: 
---

别人碰到了一个问题，然后我闲来无事对这个问题了进行了刨根问底。
`
CREATE TABLE `table1` (
`id` varchar(20) collate utf8_bin NOT NULL default '',
`from` varchar(128) collate utf8_bin NOT NULL default '',
`time` timestamp NOT NULL default CURRENT_TIMESTAMP on update CURRENT_TIMESTAMP,
PRIMARY KEY  (`id`),
KEY `id_source` (`id`,`from`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin
`
针对上面的表，有一个很简单的语句select id, from from table1 where id=325381768;执行居然花费了40多秒钟，有人会奇怪为什么id用varchar，这个是历史原因，这篇日志主要是为了从根本上说明这条sql为什么会慢。
`
xxx.host/fb>explain select id, from from table1 where id=325381768;
+----+-------------+-----------------+-------+-------------------+-----------+---------+------+----------+--------------------------+
' id ' select_type ' table           ' type  ' possible_keys     ' key       ' key_len ' ref  ' rows     ' Extra                    '
+----+-------------+-----------------+-------+-------------------+-----------+---------+------+----------+--------------------------+
'  1 ' SIMPLE      ' reg_from ' index ' PRIMARY,id_source ' id_source ' 448     ' NULL ' **** ' Using where; Using index '
+----+-------------+-----------------+-------+-------------------+-----------+---------+------+----------+--------------------------+
1 row in set (0.00 sec)
`
虽然explain的结果是使用到了索引，但是结果执行结果确时间很长。
`
xxx.host/fb>select id, from from table1  where cast(id as signed)=325381768;
+-----------+-------------+
' id        ' from '
+-----------+-------------+
' 325381768 ' S_SCREG;    '
+-----------+-------------+
1 row in set (51.81 sec)`

` `

`xxx.host/fb>select id, from from table1  where id=cast(325381768 as char);
+-----------+-------------+
' id        ' from '
+-----------+-------------+
' 325381768 ' S_SCREG;    '
+-----------+-------------+
1 row in set (0.00 sec)
`
从上面这个对比试验中可以看出，mysql是将id默认转化成了int型，然后才进行的比较，问题的症结就在这里，因为进行了转化操作所以就不能再使用索引了，那为什么explain的结果仍然是使用了索引，因为索引中包含了需要select出来的两个字段，因此mysql query优化器就选择了使用id_source索引。

通过观察Handler_read_first的状态值就可以确认mysql对id_source索引进行了一个full scan。
`
xxx.host/fb>show status like "Handler_read_first";
+--------------------+-------+
' Variable_name      ' Value '
+--------------------+-------+
' Handler_read_first ' 2     '
+--------------------+-------+
1 row in set (0.02 sec)`

`xxx.host/fb>select id, from from table1  where id=325381768;
+-----------+-------------+
' id        ' from '
+-----------+-------------+
' 325381768 ' S_SCREG;    '
+-----------+-------------+
1 row in set (43.02 sec)

`

`xxx.host/fb>show status like "Handler_read_first";
+--------------------+-------+
' Variable_name      ' Value '
+--------------------+-------+
' Handler_read_first ' 3     '
+--------------------+-------+
1 row in set (0.02 sec)
`
最后必须落脚于文档，在文档里面找到"Comparison operations result in a value of 1 (TRUE), 0 (FALSE), or NULL. These operations work for both numbers and strings. Strings are automatically converted to numbers and numbers to strings as necessary."证实了上面的猜测。
文档地址：http://dev.mysql.com/doc/refman/5.0/en/comparison-operators.html#operator_equal