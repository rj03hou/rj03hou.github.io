---
layout: post 
title: "MySQL Trigger的一个bug"
subTitle: 
heroImageUrl: 
date: 2011-12-12
tags: ["MySQL"]
keywords: 
---

一、故障原因

使用trigger在特定的场景（场景描述见Bug重现以及原因分析）下触发了mysql的bug，官方已经确认bug还没有修复，目前5.1以上版本都存在。

Bug地址http://bugs.mysql.com/bug.php?id=53079

二、解决方案

1.将binlog_format修改为mixed或者row-based（可以在线修改）。

2.从应用角度分析，摒弃trigger这种实现方式。

三、Bug重现以及原因分析

下面是从bug中摘取的，描述的很清晰，我这里就不做翻译了，已经进行了测试了和bug描述的一致。

Description:

When there is an UPDATE and an INSERT trigger on same table, and both triggers perform an

INSERT into a second table which has an auto_inc column, replication fails with a DUP KEY

error on the UPDATE statement if the transactions are committed in a different order than

the statements are executed. This is because the binary log contains a SET INSERT_ID

statement before the UPDATE, and that ID is used by the trigger (not by the UPDATE

statement itself).

How to repeat:

Create the following two tables and two triggers in a replicated environment, using

InnoDB engine and 5.1.45 distribution.

master [localhost] {msandbox} (test) > CREATE TABLE `a` (

->   `id` int(10) unsigned NOT NULL auto_increment,

->   `c1` varchar(255) NOT NULL,

->   PRIMARY KEY  (`id`)

-> ) ENGINE=InnoDB ;

Query OK, 0 rows affected (0.00 sec)

master [localhost] {msandbox} (test) > CREATE TABLE `b` (

->   `id` int(10) unsigned NOT NULL auto_increment,

->   `a_id` int(10) unsigned NOT NULL,

->   `op` enum('i','u','d') default 'i',

->   `t` timestamp NOT NULL default CURRENT_TIMESTAMP,

->   PRIMARY KEY  (`id`)

-> ) ENGINE=InnoDB ;

Query OK, 0 rows affected (0.00 sec)

master [localhost] {msandbox} (test) > delimiter ;;

master [localhost] {msandbox} (test) > create trigger a_ai after insert on a for each row

begin insert into b (a_id, op) values (NEW.id, 'i'); end;;

Query OK, 0 rows affected (0.01 sec)

master [localhost] {msandbox} (test) > create trigger a_au after update on a for each row

begin insert into b (a_id, op) values (NEW.id, 'u'); end;;

Query OK, 0 rows affected (0.00 sec)

I will upload the full log of reproducing the bug in a file, but the gist of it is:

On master, open 2 connections and run insert and update on table `a` in the following

order:

1: INSERT

1: COMMIT

2:     BEGIN

2:     UPDATE

1: BEGIN

1: INSERT

1: COMMIT

2:     COMMIT

The interleaving of statements causes the order of statements in the binary log to differ

from the order in which the TRIGGERs executed on the master (the master executes IUI but

the slave executes IIU), and therefor the UPDATE fails with a DUPKEY error because the

binary log included a SET INSERT_ID before the UPDATE statement.

The slave will execute both inserts into the `a` table, and the fail when it executes the

update, claiming a duplicate key error for primary key value "2".

Suggested fix:

Do not include SET INSERT_ID before an UPDATE, even if that UPDATE fires a TRIGGER which

generates a new auto_inc value. That was the behavior in 5.0.87, and replication did not

break (though the master and slave did get out of sync).