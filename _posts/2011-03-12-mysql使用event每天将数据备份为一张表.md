---
layout: post 
title: "mysql使用event每天将数据备份为一张表"
subTitle: 
heroImageUrl: 
date: 2011-3-12
tags: ["DBA Tools","DBA Tools","MySQL","mysql","原创"]
keywords: 
---

主要思路是先创建一个存储过程，然后创建event每天定时执行存储过程。

**一、创建日期表存储过程**
<pre lang="sql">DELIMITER //
CREATE PROCEDURE proc_create_table_daily(table_name varchar(100))
begin
declare str_date char(8);
declare create_sql varchar(100);
declare rename_sql varchar(100);
set str_date=date_format(now(),"%Y%m%d");
select str_date;
set create_sql=concat('create table ',table_name,'_tmp like ',table_name,";");
select create_sql;
set rename_sql=concat('rename table ',table_name,' to ',table_name,'_',str_date,',',table_name,'_tmp to ',table_name,";");
select rename_sql;
set @create_sql=create_sql;
prepare p1 from @create_sql;
execute p1;
deallocate prepare p1;
set @rename_sql=rename_sql;
prepare p2 from @rename_sql;
execute p2;
deallocate prepare p2;
end//</pre>
**二、创建event定时创建表**
<pre lang="sql">DELIMITER //
DROP EVENT IF EXISTS event_create_table_daily//

CREATE EVENT event_create_table_daily
ON SCHEDULE EVERY 1 day
STARTS '2011-03-12 01:00:01'
ENABLE
DO
BEGIN
call proc_create_table_daily('event_test');
END//
DELIMITER ;</pre>
**在写event过程中遇到的一些问题：**

1.  查看存储过程：show procedure status
2.  查看event：show events
3.  prepare(预处理)execute stmt using @var,只能跟@var变量,declare和传入的变量不行!!!
4.  ON SCHEDULE EVERY 1 Hour STARTS '2011-03-11 18:11:30'意思就是从start时间开始，每过一个小时执行一次，starts时间也会执行
5.  存储过程的变量分为两种：全局变量@str_data，session中一直有效；declare str_date char(8)局部变量，使用的时候不需要添加@符号
6.  如果event是ON SCHEDULE at now()当event执行结束后使用[ON COMPLETION [NOT] PRESERVE] 来决定是否保存event