---
layout: post 
title: "mysql init-connect选项与字符集"
subTitle: 
heroImageUrl: 
date: 2011-1-20
tags: ["MySQL","mysql"]
keywords: 
---

init-connect='SET NAMES utf8'
SET character_set_client = x;
SET character_set_results = x;
SET character_set_connection = x;
这三个选项应该配置的是服务器端的，而我们设置的character_set_server=utf8，默认这三个选项就是utf8的，因此这个指定我觉得没有作用。

Note that the content of init_connect is not executed for users that have the SUPER privilege. This is done so that an erroneous value for init_connect does not prevent all clients from connecting. For example, the value might contain a statement that has a syntax error, thus causing client connections to fail. Not executing init_connect for users that have the SUPER privilege enables them to open a connection and fix the init_connect value.

通过python进行测试
conn=MySQLdb.connect(host='127.0.0.1',user='admin2',passwd='',db='test',charset='gb2312')

mysql> show grants for 'admin2'@'127.0.0.1';
+-----------------------------------------------------------------+
' Grants for admin2@127.0.0.1                                   '
+-----------------------------------------------------------------+
' GRANT USAGE ON *.* TO 'admin2'@'127.0.0.1'                    '
' GRANT SELECT, INSERT ON `test`.`test` TO 'admin2'@'127.0.0.1' '
+-----------------------------------------------------------------+
2 rows in set (0.00 sec)

但是测试脚本执行的结果仍然显示乱码，并且character_set_client、character_set_results为MySQLdb.connect连接方法中设置的参数，init-connect没有执行。猜测假如init-connect='SET NAMES utf8'按照文档中所说的在连接连上server之后，执行set操作，session级别的参数character_set_client、character_set_results应该为utf8

但是将init-connect改为"insert into test.test values('hello')"，执行结果显示插入了hello

将mysql的log开启之后发现，对于使用python下面的MySQLdb来说，其中set autocommit=0是MySQLdb默认的方式。
conn=MySQLdb.connect(host='127.0.0.1',user='admin2',passwd='',db='test',charset='gb2312')
<span style="color: #ff0000;">MySQLdb先执行init-connect的SET NAMES utf8，然后将charset='gb2312'解释为SET NAMES gb2312执行，所以使用不同语言的客户端的时候最好都强制对字符集进行指定或者深入调查清楚默认的行为。</span>
101118  0:27:52     1 Connect   admin2@127.0.0.1 on test
1 Query     SET NAMES utf8
1 Query     SET NAMES gb2312
1 Query     set autocommit=0
conn=MySQLdb.connect(host='127.0.0.1',user='admin2',passwd='',db='test')
101118  0:27:52     1 Connect   admin2@127.0.0.1 on test
1 Query     SET NAMES utf8
1 Query     set autocommit=0

另外两篇讨论字符集的问题：

自己写的《MySQL中latin1与中文》[http://www.themysql.com/?p=283
](http://www.themysql.com/?p=283)鸟哥写的《深入Mysql字符集设置》[http://www.laruence.com/2008/01/05/12.html](http://www.laruence.com/2008/01/05/12.html)