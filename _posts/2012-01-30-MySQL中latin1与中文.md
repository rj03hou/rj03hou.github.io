---
layout: post 
title: "MySQL中latin1与中文"
subTitle: 
heroImageUrl: 
date: 2012-1-30
tags: ["MySQL","mysql"]
keywords: 
---

跟同事讨论latin1与中文的关系，调查了之后将结果总结如下:
不同的字符集编码了不同的字符，latin1中包含有191个可打印字符，其余是控制字符或者扩展的欧洲特殊字符；不包含中文字符。

之所以可以在latin1中保存和显示中文字符请看下面的示例以及说明：
latin1和ascii都可以存储中文(ascii就不列举)
<pre lang="sql">CREATE TABLE `user` (
`name` varchar(100) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1
set names latin1;
insert into user values("我们");
mysql>select * from user;
+--------+
' name   '
+--------+
' 我们   '
+--------+
1 row in set (0.00 sec)</pre>
使用set names latin1之后，character_set_client、character_set_connection、character_set_results都将设置为ascii编码，不会做任何编码转化(编码转化会涉及到重新编码，因为不同的字符集对同一个字符的编码不一样；以及新字符集无法编码旧字符集中字符的问题)。

如果改成set names utf8，则insert的时候，character_set_connection->column character set时，会将utf8编码强行转化成latin1，转化时超出latin1编码范围的字符全部转化成0x3F（latin1中的?）。

参照：[深入MySQL字符集设置](http://www.laruence.com/2008/01/05/12.html "深入Mysql字符集设置")