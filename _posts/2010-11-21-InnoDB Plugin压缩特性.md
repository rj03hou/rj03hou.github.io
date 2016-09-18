---
layout: post 
title: "InnoDB Plugin压缩特性"
subTitle: 
heroImageUrl: 
date: 2010-11-21
tags: ["innodb plugin","MySQL","mysql","原创"]
keywords: 
---

innodb plugin的压缩特性是从5.1版本开始出现的一个特性，最初是由google开发出来，然后开源，mysql吸收的。

The usual (uncompressed) size of InnoDB data pages is 16KB. Beginning with the InnoDB Plugin, you can use the attributes `ROW_FORMAT=COMPRESSED`, `KEY_BLOCK_SIZE`, or both in the `CREATE TABLE` and `ALTER TABLE`statements to enable table compression. Depending on the combination of option values, InnoDB attempts to compress each page to 1KB, 2KB, 4KB, 8KB, or 16KB.

通过上面这段官方文档的理解就是page在解压之后16kb，这个不会发生变化，key_block_size可以解释为压缩的程度，mysql将page压缩到多大。

压缩会大幅度的节省磁盘空间，压缩之后空间为原来的1/4不等，具体于表结构有很大关系，varchar、char等压缩比会较高，压缩使用的是[zlib library](http://www.zlib.net/)中的[LZ77](http://zh.wikipedia.org/wiki/LZ77%E4%B8%8ELZ78)算法。

压缩适合于当数据的insert、update操作较少，select操作较多并且io为瓶颈的一种场景，也可以用来取代archive引擎和myisam的压缩。

使用起来相对很简单：

CREATE TABLE name (column1 INT PRIMARY KEY) ENGINE=InnoDB
ROW_FORMAT=COMPRESSED
KEY_BLOCK_SIZE=4;

下面这张表格主要说明了ROW_FORMAT和key_block_size的具体含义。其中其中innodb默认的innodb_file_format是antelope，innodb plugin中可以指定为Barracuda，其中Barracuda和antelope的主要区别是在ROW_FORMAT上的区别。

**Meaning of `CREATE TABLE` and `ALTER TABLE` Options**
<table border="1" summary="Meaning of CREATE TABLE and ALTER TABLE Options"><colgroup> <col></col> <col></col> <col></col> </colgroup>
<thead>
<tr>
<th>Option</th>
<th>Usage</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td>`ROW_FORMAT=​REDUNDANT`</td>
<td>Storage format used prior to MySQL 5.0.3</td>
<td>Less efficient than `ROW_FORMAT=COMPACT`; for backward compatibility</td>
</tr>
<tr>
<td>`ROW_FORMAT=​COMPACT`</td>
<td>Default storage format since MySQL 5.0.3</td>
<td>Stores a prefix of 768 bytes of long column values in the clustered index page, with the remaining bytes stored in an overflow page</td>
</tr>
<tr>
<td>`ROW_FORMAT=​DYNAMIC`</td>
<td>Available only with `innodb_file​_format=Barracuda`</td>
<td>Store values within the clustered index page if they fit; if not, stores only a 20-byte pointer to an overflow page (no prefix)</td>
</tr>
<tr>
<td>`ROW_FORMAT=​COMPRESSED`</td>
<td>Available only with `innodb_file​_format=Barracuda`</td>
<td>Compresses the table and indexes using zlib to default compressed page size of 8K bytes; implies`ROW_FORMAT=DYNAMIC`</td>
</tr>
<tr>
<td>`KEY_BLOCK_​SIZE=_<code>n`_</code></td>
<td>Available only with `innodb_file​_format=Barracuda`</td>
<td>Specifies compressed page size of 1, 2, 4, 8 or 16K bytes; implies `ROW_FORMAT=DYNAMIC` and`ROW_FORMAT=COMPRESSED`</td>
</tr>
</tbody>
</table>
那天有同事担心压缩之后假如性能不能接受能否改回未压缩版本，可以使用alter table row_format=compact，前提是innodb_strict_mode为0，否则提示ERROR 1005 (HY000): Can't create table 'test.#sql-4d7e_1' (errno: 1478)类似的错误。改了之后在show create table的时候还会看到key_block_size，但是此时key_block_size已经被ignore了。推荐的方式重新建一张表，然后insert into select * from。
[另外一篇orczhu写的关于innodb plugin压缩特性的文章](http://www.orczhou.com/index.php/2010/03/innodb-plugin-compression/)，写的非常不错