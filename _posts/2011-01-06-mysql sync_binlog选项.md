---
layout: post 
title: "mysql sync_binlog选项"
subTitle: 
heroImageUrl: 
date: 2011-1-6
tags: ["MySQL"]
keywords: 
---

sync_binlog

If the value of this variable is greater than 0, the MySQL server synchronizes its binary log to disk (using fdatasync()) after every sync_binlog writes to the binary log. There is one write to the binary log per statement if autocommit is enabled, and one write per transaction otherwise. The default value of sync_binlog is 0, which does no synchronizing to disk - in this case, the server relies on the operating system to flush the binary log's contents from to time as for any other file. A value of 1 is the safest choice because in the event of a crash you lose at most one statement or transaction from the binary log. However, it is also the slowest choice (unless the disk has a battery-backed cache, which makes synchronization very fast).

[Group commit and XA ](http://www.mysqlperformanceblog.com/2006/05/19/group-commit-and-xa/)
Returning to post Group commit and real fsync I made several experiments:

I ran sysbench update_key benchmarks without -log-bin, with -log-bin, and with -log-bin and -innodb-support-xa=0 (default value is 1). Results (in transactions / sec)
<table border="1" cellspacing="0" cellpadding="1" width="100%">
<tbody>
<tr>
<td>threads</td>
<td>without -log-bin</td>
<td>-log-bin</td>
<td>-log-bin and
-innodb_support-xa=0</td>
</tr>
<tr>
<td>1</td>
<td>1218.68</td>
<td>614.94</td>
<td>1010.44</td>
</tr>
<tr>
<td>4</td>
<td>2686.36</td>
<td>667.77</td>
<td>1162.60</td>
</tr>
<tr>
<td>16</td>
<td>3993.59</td>
<td>666.14</td>
<td>1161.56</td>
</tr>
<tr>
<td>64</td>
<td>3630.55</td>
<td>665.18</td>
<td>1151.36</td>
</tr>
</tbody>
</table>
So we can see group commit is not only broken when XA is enabled but also if XA is disabled but binary log is enabled. Performance without XA can be twice as good as with XA if binary logs are enabled as Innodb will need to flush its log buffer only once. So, if you are using -log-bin with innodb tables it makes sense to set -innodb-support-xa=0