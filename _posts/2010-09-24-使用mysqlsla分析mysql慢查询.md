---
layout: post 
title: "使用mysqlsla分析mysql慢查询"
subTitle: 
heroImageUrl: 
date: 2010-9-24
tags: ["MySQL","mysql","mysqlsla","slow log"]
keywords: 
---

mysqlsla是hackmysql.com推出的一款MySQL的日志分析工具，功能异常强大，可以用来替代mysql自带的slow log分析工具。分析服务器的slow log对于服务器优化非常重要。

官方地址:[http://hackmysql.com/mysqlsla](http://hackmysql.com/mysqlsla)

**安装方法：**

Alternatively, you can bypass the installer and simply wget http://hackmysql.com/scripts/mysqlsla-2.03, then chmod +x and run it.

直接wget http://hackmysql.com/scripts/mysqlsla-2.03下载就可以直接运行，不过因为mysqlsla是用perl写的，因此需要perl环境。

**使用方法：**

通过下面几个常用参数，对于mysqlsla的使用就会有一个初步的了解，具体的其他功能再需要的时候可以通过官方文档，详细了解。

详细的参数列表：[http://hackmysql.com/mysqlsla_documentation#meta-filter](http://hackmysql.com/mysqlsla_documentation#meta-filter)

--log-type (-lt) TYPE LOGS

Parse MySQL LOGS of TYPE. Default none. TYPE must be either slow, general, binary, msl or udl. LOGS is a space-separated list of MySQL log files.

通过这个参数来制定log的类型，主要有slow、general、binary等，分析slow log时通过制定为slow

--sort META

Sort queries according to META. Default t_sum for slow and msl logs, c_sum for all others. META is any meta-property name; see [mysqlsla v2 Filters](http://hackmysql.com/mysqlsla_filters).

mysqlsla currently does not check that the meta-property name META actually exists. Therefore, if a non-existent meta-property name is given, mysqlsla will print copious errors.

mysqlsla通过这个参数来制定使用什么参数来对分析结果进行排序，默认是按照t_sum来进行排序。

_t_sum_: Total t

总的消耗时间

_c_sum_: Total number of times SQL statement appears in log，按照sql语句执行的总时间来进行排序。

_c_sum_p_: Percentage that c_sum constitutes of grand total c_sum for all SQL statements in log。

sql语句执行时间占总执行时间的百分比。

详细的sort参数可以在下面这个地方查看。

[http://hackmysql.com/mysqlsla_filters#setting_the_filters](http://hackmysql.com/mysqlsla_filters#setting_the_filters)

--top N

After [sorting](http://hackmysql.com/mysqlsla_documentation#sort) display only the top N queries. Default 10.

显示的sql数，默认是10条

下面不常用的参数简单说明一下：

--statement-filter (-sf) CONDTIONS

通过这个参数过滤sql语句的类型，比如select、update、drop等等

--grep PATTERN

grep statements for PATTERN and keep only those which match. Default none.

PATTERN is a [Perl regular expressions](http://perldoc.perl.org/perlre.html) pattern without m//.

--meta-filter (-mf) CONDTIONS

通过这个参数对结果进行过滤，比如执行次数占总次数10%以上

mysqlsla --log-type slow slow.log --meta-filter "c_sum_p>5"

**结果说明：**

** **

**Count         : 4.22k  (1.01%)**

Time          : 8526 s total, 2.018466 s avg, 2 s to 11 s max  (0.61%)

95% of Time : 8024 s total, 2 s avg, 2 s to 2 s max

Lock Time (s) : 0 total, 0 avg, 0 to 0 max  (0.00%)

95% of Lock : 0 total, 0 avg, 0 to 0 max

Rows sent     : 1 avg, 0 to 1 max  (0.01%)

Rows examined : 0 avg, 0 to 0 max  (0.00%)

Count, sql的执行次数及占总的slow log数量的百分比.
Time, 执行时间, 包括总时间, 平均时间, 最小, 最大时间, 时间占到总慢sql时间的百分比.
95% of Time, 去除最快和最慢的sql, 覆盖率占95%的sql的执行时间.
Lock Time, 等待锁的时间.
95% of Lock , 95%的慢sql等待锁时间.
Rows sent, 结果行统计数量, 包括平均, 最小, 最大数量.
Rows examined, 扫描的行数量.
Database, 属于哪个数据库
Users, 哪个用户,IP, 占到所有用户执行的sql百分比
Query abstract, 抽象后的sql语句
Query sample, sql语句

下面这篇文章写的也不错，也可以看看

[http://www.517sou.net/Article/mysqlsla.aspx](http://www.517sou.net/Article/mysqlsla.aspx)

突然感觉蛋有点疼--到此为止吧