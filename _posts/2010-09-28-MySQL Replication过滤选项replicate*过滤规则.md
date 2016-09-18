---
layout: post 
title: "MySQL Replication过滤选项replicate*过滤规则"
subTitle: 
heroImageUrl: 
date: 2010-9-28
tags: ["MySQL","mysql","replication"]
keywords: 
---

本文为官方文档[How Servers Evaluate Replication Filtering Rules](http://dev.mysql.com/doc/refman/5.1/en/replication-rules.html)的精简意译版，有什么疑问欢迎讨论。

mysql从库的过滤选项replicate*分为两类，Database-Level和Table-Level。其中Database-Level受到`binlog_format的影响；Table-Level指不受到binlog_format的影响，关于这个下文会详细介绍。`

If any --replicate-rewrite-db options were specified, they are applied before the --replicate-* filtering rules are tested.

replicate-rewrite-db选项会在其他选项生效之前执行。

官方文档中Database-Level筛选流程如下：

![Evaluation of Database-Level Filtering Rules           in Replication](replication-filtering-db-rules.png)

假如过滤选项中包含Database-Level选项，则首先按照Database-Level过滤，过滤完毕之后再按照Table-Level选项进行过滤。

官方文档中Table-Level选项过滤流程如下：

![Evaluation of Table-Level Filtering Rules in           Replication](replication-filtering-tbl-rules.png)

其中do-db和do-table的优先级高于ignore-db和ignore-table

本着尽信书不如无书以及文档中的"破绽"，我做了下面的测试：
<div id="_mcePaste">测试一：</div>
<div id="_mcePaste">①</div>
<div id="_mcePaste">增加下面选项时</div>
<div id="_mcePaste">Replicate_Do_DB: test1</div>
<div id="_mcePaste">Replicate_Ignore_Table: test1.test1_1</div>
<div id="_mcePaste">master</div>
<div id="_mcePaste">test1.test1_1 test1.test1_2</div>
<div id="_mcePaste">slave</div>
<div id="_mcePaste">no tables</div>
<div id="_mcePaste">在master上执行use test1;drop table test1_1;</div>
<div id="_mcePaste">在slave上执行show slave status\G未报错</div>
<div id="_mcePaste">在master上执行use test1;drop table test1_2;</div>
<div id="_mcePaste">在slave上执行show slave status\G报错</div>
<div id="_mcePaste">结论：说明Replicate_Ignore_Table起作用（文档中的流程图不一致）</div>
<div id="_mcePaste">②</div>
<div id="_mcePaste">选项同上，并且表结构都有</div>
<div id="_mcePaste">在master上执行use test1;insert into test1_1 values()，查看从库未同步（文档中的流程图不一致）</div>
<div id="_mcePaste">在master上执行use test1;insert into test1_2 values()，查看从库同步</div>
<div id="_mcePaste">测试二：</div>
<div id="_mcePaste">Replicate_Do_DB: test1</div>
<div id="_mcePaste">Replicate_Do_Table: test2.test2_1</div>
<div id="_mcePaste">Replicate_Ignore_Table: test1.test1_1</div>
<div id="_mcePaste">主从表结构都相同，都是</div>
<div id="_mcePaste">test1.test1_1</div>
<div id="_mcePaste">test1.test1_2</div>
<div id="_mcePaste">test2.test2_1</div>
<div id="_mcePaste">test2.test2_2</div>
<div id="_mcePaste">在主库上执行use test2;insert into test2_1 values()，查看从库未同步</div>
<div id="_mcePaste">也就是说Replicate_Do_DB未匹配并且不包含replicate-ignore-db时丢弃（文档中的流程图不一致）</div>
<div id="_mcePaste">测试三：</div>
<div id="_mcePaste">Replicate_Do_DB: test1</div>
<div id="_mcePaste">Replicate_Ignore_DB: test2</div>
<div id="_mcePaste">主从表结果都相同，都是</div>
<div id="_mcePaste">test1.test1_1</div>
<div id="_mcePaste">test1.test1_2</div>
<div id="_mcePaste">test2.test2_1</div>
<div id="_mcePaste">test2.test2_2</div>
<div id="_mcePaste">test3.test3_1</div>
<div id="_mcePaste">在主库上执行use test3;insert into test3_1 values()，查看从库未同步（文档中的流程图不一致）</div>
<div id="_mcePaste">通过上面的测试得出结论：</div>
<div id="_mcePaste">①当选项中包含Replicate_Do_DB时，没有match到的语句全部丢弃了，也就是说包含Replicate_Do_DB，Replicate_Ignore_DB没有起作用</div>
<div id="_mcePaste">②mysql官方文档中的流程图不准确</div>
<div>③当Replicate_Do_DB选项匹配时，会进一步的通过Replicate_Do_Table和Replicate_Ignore_Table进行筛选。</div>
<div>文档中提到的一些注意事项：</div>
<div>①In general, to make it easier to determine what effect an option set will have, it is recommended that you avoid mixing "do" and "ignore" options, or wildcard and nonwildcard options.</div>
<div>避免混合使用do和ignore以及通配符和非通配符</div>
<div>②First, as a preliminary condition, the slave checks whether statement-based replication is enabled. If so, and the statement occurs within a stored function, the slave executes the statement and exits. If row-based replication is enabled, the slave does not know whether a statement occurred within a stored function on the master, so this condition does not apply.</div>
<div>Note</div>
<div>For statement-based replication, replication events represent statements (all changes making up a given event are associated with a single SQL statement); for row-based replication, each event represents a change in a single table row (thus a single statement such as UPDATE mytable SET mycol = 1 may yield many row-based events). When viewed in terms of events, the process of checking table options is the same for both row-based and statement-based replication.</div>
<div>当binlog-format为row-level的时候，执行行所引起的触发器、存储过程、UDF等是不会执行，因为此时mysql不能确定引发的sql语句是否已经写入到binlog中。当binlog-format为row-level时，一条语句会产生很多binlog event。</div>
<div>最后mysql文档中举了一个例子：</div>
<div>
<div>Suppose that we have two tables mytbl1 in database db1 and mytbl2 in database db2 on the master, and the slave is running with the following options (and no other replication filtering options):</div>
<div>replicate-ignore-db = db1</div>
<div>replicate-do-table  = db2.tbl2</div>
<div>Now we execute the following statements on the master:</div>
<div>USE db1;</div>
<div>INSERT INTO db2.tbl2 VALUES (1);</div>
<div>The results on the slave vary considerably depending on the binary log format, and may not match initial expectations in either case.</div>
<div>Statement-based replication.  The USE statement causes db1 to be the default database. Thus the --replicate-ignore-db option matches, and the INSERT statement is ignored. The table options are not checked.</div>
<div>Row-based replication.  The default database has no effect on how the slave reads database options when using row-based replication. Thus, the USE statement makes no difference in how the --replicate-ignore-db option is handled: the database specified by this option does not match the database where the INSERT statement changes data, so the slave proceeds to check the table options. The table specified by --replicate-do-table matches the table to be updated, and the row is inserted.</div>
<div>当binlog-format为statement 时，replicate-do-db等会按照use语句进行过滤，这个时候假如使用类似着这样的操作，user test1;update test2.test2_1等会出现过滤结果跟猜测的不一致的情况发生。</div>
</div>
[阿飞](http://www.themysql.com)原创文章，欢迎转载，转载请保留此句话。