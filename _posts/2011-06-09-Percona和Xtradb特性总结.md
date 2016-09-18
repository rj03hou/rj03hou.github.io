---
layout: post 
title: "Percona和Xtradb特性总结"
subTitle: 
heroImageUrl: 
date: 2011-6-9
tags: ["MySQL","Percona"]
keywords: 
---

**Response Time Distribution**
query_response_time_stats设置是否开启，默认[0]
通过query_response_time_range_base设置range_base
(range_base ^ n; range_base ^ (n+1)]
通过SELECT * from INFORMATION_SCHEMA.QUERY_RESPONSE_TIME或SHOW QUERY_RESPONSE_TIME;查看
FLUSH QUERY_RESPONSE_TIME;
FLUSH does two things:
Clears the collected times from the QUERY_RESPONSE_TIME table
Reads the value of query_response_time_range_base and uses it to set the range base for the table
表类型momory因此对性能的损耗应该十分有限

**Support of Multiple Page Sizes**
innodb_page_size
Dynamic Variable	 No
###EXPERIMENTAL###: The universal page size of the database. Changing for an existing database is not supported. Use at your own risk!

**Fast Shutdown**
Some InnoDB/XtraDB threads which perform various background activities are in the sleep state most of the time. They only wake up every few seconds to perform their tasks. They also check whether the server is in the shutdown phase, and if not, they go to the sleep state again. That means there could be a noticeable delay (up to 10 seconds) after a shutdown command and before all InnoDB/XtraDB threads actually notice this and terminate. This is not a big problem for most production servers, because a shutdown of a heavily loaded server normally takes much longer than 10 seconds.
后台运行的处于sleep状态的线程，只有在wake up的时候才会检查是否处于关闭状态，xtradb直接关闭。

**Count InnoDB Deadlocks**
It adds a new global status variable (innodb_deadlocks) showing the number of deadlocks.*
innodb_deadlocks

**Slow Query Log**
log_slow_filter这个variable可以将更多的类型输入到slow log中
<table>
<tbody>
<tr>
<th>Value</th>
<th>Explanation</th>
</tr>
<tr>
<td>qc_miss</td>
<td>The query was not found in the query cache.</td>
</tr>
<tr>
<td>full_scan</td>
<td>The query performed a full table scan.</td>
</tr>
<tr>
<td>full_join</td>
<td>The query performed a full join (a join without indexes).</td>
</tr>
<tr>
<td>tmp_table</td>
<td>The query created an implicit internal temporary table.</td>
</tr>
<tr>
<td>tmp_table_on_disk</td>
<td>The query's temporary table was stored on disk.</td>
</tr>
<tr>
<td>filesort</td>
<td>The query used a filesort.</td>
</tr>
<tr>
<td>filesort_on_disk</td>
<td>The filesort was performed on disk.</td>
</tr>
</tbody>
</table>
更多的特性参照官方说明：[http://www.percona.com/docs/wiki/percona-server:features:start](http://www.percona.com/docs/wiki/percona-server:features:start)

&nbsp;