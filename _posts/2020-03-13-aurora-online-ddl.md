---
layout: single
title: AWS aurora online DDL
description: AWS aurora online DDL
headline: AWS aurora online DDL
categories: aurora,aws
headline: 
tags: [aurora,aws]
comments: true
published: true
---



## FastDDL

不过FastDDL根据文档只在lab mode下生效，但是lab mode默认是关闭的，根据[aws文档](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Managing.FastDDL.html)不建议在生产环境下使用lab mode。

Aurora支持添加字段的时候，支持fast DDL，不过有如下限制:

- **Fast DDL only supports adding nullable columns, without default values, to the end of an existing table.**

- Fast DDL does not support partitioned tables.

- Fast DDL does not support InnoDB tables that use the REDUNDANT row format.

- If the maximum possible record size for the DDL operation is too large, fast DDL is not used. A record size is too large if it is greater than half the page size. The maximum size of a record is computed by adding the maximum sizes of all columns. For variable sized columns, according to InnoDB standards, extern bytes are not included for computation.

  Note

  The maximum record size check was added in Aurora 1.15.



只支持add column，其他的见mysql：https://dev.mysql.com/doc/refman/5.7/en/innodb-online-ddl-operations.html

所以修改字段的类型，还是会rebuild table，并且会锁表，不允许进行其他dml

官方的一篇博客的测试结果，可以看到效果相当明显

![img](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2017/04/04/TableComparison.png)



**原理**

In Aurora, when a user issues a DDL statement:

1. The database updates the INFORMATION_SCHEMA system table with the new schema. In addition, the database timestamps the operation, records the old schema into a new system table (Schema Version Table), and propagates this change to read replicas.

That’s it for the synchronous part of the operation. Done.

Then, on subsequent DML operations, we check to see if the affected data page has a pending schema operation. That’s easily done by comparing the log sequence number (LSN) timestamp for the page with the LSN timestamp of schema changes. If needed, we then update the page to the new schema before applying the DML statement. This operation follows the same upgrade process for redo-undo record pages as everything else. And any I/Os are piggybacked on top of user activity.

You have to be careful about only upgrading the page on DML operations, because upgrades can cause page splits. We need to deal with upgrades on our Aurora Replicas too, and they’re not allowed to change any data. For SELECT statements, we change the memory image of the buffer being passed back to MySQL. This way, it always sees the latest schema, even though the underlying storage is a mix of old and new schema formats.



**index变更**

**Table 14.11 Online DDL Support for Index Operations**

| Operation                            | In Place | Rebuilds Table | Permits Concurrent DML | Only Modifies Metadata |
| ------------------------------------ | -------- | -------------- | ---------------------- | ---------------------- |
| Creating or adding a secondary index | Yes      | No             | Yes                    | No                     |
| Dropping an index                    | Yes      | No             | Yes                    | Yes                    |
| Renaming an index                    | Yes      | No             | Yes                    | Yes                    |
| Adding a `FULLTEXT` index            | Yes*     | No*            | No                     | No                     |
| Adding a `SPATIAL` index             | Yes      | No             | No                     | No                     |
| Changing the index type              | Yes      | No             | Yes                    | Yes                    |

alter table tb_order_archive ADD security_type tinyint(4) DEFAULT 1 NOT NULL, 	;

[Altering Tables in Amazon Aurora Using Fast DDL](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Managing.FastDDL.html)

[Amazon Aurora Under the Hood: Fast DDL](https://aws.amazon.com/blogs/database/amazon-aurora-under-the-hood-fast-ddl/)

[MySQL Online DDL Operations](https://dev.mysql.com/doc/refman/5.7/en/innodb-online-ddl-operations.html#online-ddl-index-operations)

https://www.vividcortex.com/blog/three-things-that-differentiate-amazon-aurora-from-mysql



## pt-query-digest

https://aws.amazon.com/cn/blogs/china/pt-query-digest-rds-mysql-slow-searchnew/