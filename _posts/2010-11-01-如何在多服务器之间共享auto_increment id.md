---
layout: post 
title: "如何在多服务器之间共享auto_increment id"
subTitle: 
heroImageUrl: 
date: 2010-11-1
tags: ["auto_increment","MySQL","mysql","原创"]
keywords: 
---

Morgan Tocker对多种做法进行了一个测试，[详细的测试结论](http://www.mysqlperformanceblog.com/2010/10/26/sharing-an-auto_increment-value-across-multiple-mysql-tables-revisited/)，[详细的测试代码](http://www.mysqlperformanceblog.com/wpcontent/uploads/2010/10/fulldisclosure.txt)，推荐使用下面两种方式之一：

第一种：
<pre>CREATE TABLE option1 (id int not null primary key auto_increment) engine=innodb;
# each insert does one operations to get the value:
INSERT INTO option1 VALUES (NULL);
# $connection->insert_id();</pre>
第二种：

    CREATE TABLE `Tickets64` (
      `id` bigint(20) unsigned NOT NULL auto_increment,
      `stub` char(1) NOT NULL default '',
      PRIMARY KEY  (`id`),
      UNIQUE KEY `stub` (`stub`)
    ) ENGINE=MyISAM
    REPLACE INTO Tickets64 (stub) VALUES ('a');
    SELECT LAST_INSERT_ID();

flickr采用了第二种方式，并且也发表一篇[日志](http://code.flickr.com/blog/2010/02/08/ticket-servers-distributed-unique-primary-keys-on-the-cheap/)，详细论述了其他的方案的缺陷包括第一种。
<div>针对flickr的日志进行简要意译，因为数据量庞大，然后对大表进行了切分，但是切分之后又属于同一个逻辑表，因此需要在多个表或者库之间共享auto_increment id，保证id的全局唯一性。使用GUID虽然可以保证唯一性，但是因为GUID太大，索引的效率等等会受到影响，因此这种方法不可取；使用方法1，使用一个全局的表来保存auto_increment id，使用这种方法当插入频繁的时候改表会增长的非常迅速，因此需要对它进行维护；使用方法2则可以避免这种维护，而且可以使用master-master结构来保证唯一id的高可用性。当然也可以选择第三方数据库来实现，比如使用PostgreSQL的sequence，但是我个人觉得这个事情其实完全可以用c++写一个服务，前后lock一下，不过高可用性等不太方面，起多个，这个不行了就用另外一个。</div>