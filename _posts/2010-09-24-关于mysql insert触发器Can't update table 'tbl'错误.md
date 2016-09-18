---
layout: post 
title: "关于mysql insert触发器Can't update table 'tbl'错误"
subTitle: 
heroImageUrl: 
date: 2010-9-24
tags: ["MySQL","mysql","trigger"]
keywords: 
---

Can't update table 'tbl' in stored function/trigger because it is already used by statement which invoked this stored function/trigger

链接：[http://forums.mysql.com/read.php?99,122354,122354#msg-122354](http://forums.mysql.com/read.php?99,122354,122354#msg-122354)

**原因：**

when you insert a record mysql is doing some lock stuff. you can't insert/update/delete rows of the same table where you insert.. because then the trigger would called again and again.. ending up in a recursion。

别人测试的结果是类似这样的触发器在MSSQL, Oracle, DB2, PostgreSQL都没有问题，应该是mysql的一个缺陷，估计会在以后得版本中完成，因为担心陷入死循环，完全可以暴力一点判断再该条语句引发的第三次对该表的操作时，终止该操作，然后给一个waring。mysql目前的做法就是传统的"一个ip出问题，然后封锁一个网段"的做法，mysql因为触发器刚刚推出，再一步一步的完善中。

I think the reason is because a table-level lock is issued while the trigger is running, meaning, no modifications allowed to the calling table by the trigger.

这是另外一个人提出的假设，他假设mysql现在使用触发器时使用的table lock

下面是另外一个人的回帖，对这个是否产生的recursion进行了分析，并给出了测试语句。

What's that stuff about recursion?

CREATE TABLE a (
id int(11) NOT NULL auto_increment primary key,
ref int,
updttime datetime
);

DELIMITER '

CREATE TRIGGER buildref AFTER INSERT ON a
FOR EACH ROW BEGIN
UPDATE a SET a.ref = NEW.id, a.updttime = NOW() WHERE a.id = NEW.id;
END;
'

DELIMITER ;

insert into a values(0,0,0) should work!

What's so recursive about that? The record is created and an update should occur.
There is no way to invoce AFTER INSERT again, because it's not INSERT but UPDATE.
What's wrong with that? It runs in Oracle and Informix. Can't find any reason why this should not work!

**解决方法：**

I know this is an old post but I stumbled on the answer and thought I would share. During the insert/update you have access to the NEW object which contains all of the fields in the table involved. If you do a before insert/update and edit the field(s) that you want to change in the new object it will become a part of the calling statement and not be executed as a separately (eliminating the recursion)
ex.
create trigger test
before update on test
for each row
set NEW.updateTime = NOW();

delimiter '

说白了，就是对该记录的操作将after改成before。

我另外做了一下的测试：

测试版本包括5.0.45和5.1.49

create table test(id serial,value1 int,value2 int);

delimiter '

CREATE TRIGGER t1 AFTER INSERT ON test

FOR EACH ROW BEGIN

update test set value2=value1 where id=NEW.id;

END;

'

delimiter ;

Can't update table 'test' in stored function/trigger because it is already used by statement which invoked this stored function/trigger

CREATE TRIGGER t1 BEFORE INSERT ON test

FOR EACH ROW BEGIN

SET NEW.value2 =NEW.value1;

END;

'

delimiter ;

success

CREATE TRIGGER t1 BEFORE INSERT ON test

FOR EACH ROW BEGIN

update test set value2=value1 where id=NEW.id-1;

END;

'

delimiter ;

ERROR 1442 (HY000): Can't update table 'test' in stored function/trigger because it is already used by statement which invoked this stored function/trigger.

因为trigger刚刚加入mysql不久还在剧烈更新中，因此最后再每次添加触发器的时候都要进行详细的测试，能用就OK。