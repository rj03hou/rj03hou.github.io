---
layout: post 
title: "InnoDB Tablespaces 表空间"
subTitle: 
heroImageUrl: 
date: 2010-11-21
tags: ["innodb","MySQL","mysql"]
keywords: 
---

文章转自：[http://www.linbuluo.com/?p=91](http://www.linbuluo.com/?p=91)

**A tablespace consists of multiple files and/or raw disk partitions.**

(注：如果**innodb**设置成共享表空间，则所有的**元数据**，表数据，索引数据，及事务的undo数据都存放在同一个表空间(一个或多个数据文件)；

也可设置成独享表空间，每个表的数据和索引存放在一个单独的.ibd文件中，该文件包括每个表的表数据，索引数据和事务undo数据)
file_name:file_size[:autoextend[:max:max_file_size]]
• A file/partition is a collection of segments.
• A segment consists of fixed-length pages.
• The page size is always 16KB in uncompressed tablespaces, and 1KB-16KB in compressed tablespaces (for both data and index).

[![](innodb_tablespace-300x215.jpg "innodb_tablespace")](http://www.themysql.com/wp-content/uploads/2010/11/innodb_tablespace.jpg)

from:《InnoDB Internals: InnoDB File Formats and Source Code Structure》

下面这段来自mysql fm。from:[http://dev.mysql.com/doc/refman/5.0/en/innodb-file-space.html](http://dev.mysql.com/doc/refman/5.0/en/innodb-file-space.html)

<a name="innodb-file-space"></a>13.2.11.2. File Space Management

The data files that you define in the configuration file form the `InnoDB` tablespace. The files （innodb_data_file_path=ibdata1:1G;ibdata2:1G;ibdata3:1G;ibdata4:1G;ibdata5:1G ）are logically concatenated to form the tablespace. There is no striping in use. Currently, you cannot define where within the tablespace your tables are allocated. However, in a newly created tablespace, `InnoDB` allocates space starting from the first data file.

The tablespace consists of database pages with a default size of 16KB. The pages are grouped into extents of size 1MB (64 consecutive pages). The "files" inside a tablespace are called _segments_ in `InnoDB`. The term "rollback segment" is somewhat confusing because it actually contains many tablespace segments.

When a segment grows inside the tablespace, `InnoDB` allocates the first 32 pages to it individually. After that,`InnoDB` starts to allocate whole extents to the segment. `InnoDB` can add up to 4 extents at a time to a large segment to ensure good sequentiality of data.

Two segments are allocated for each index in `InnoDB`. One is for nonleaf nodes of the B-tree, the other is for the leaf nodes. The idea here is to achieve better sequentiality for the leaf nodes, which contain the data.

Some pages in the tablespace contain bitmaps of other pages, and therefore **a few extents in an `InnoDB`tablespace cannot be allocated to segments as a whole**, but only as individual pages.

下面这段来自一个网友的总结：

[![](tablespace2-300x300.jpg "tablespace2")](http://www.themysql.com/wp-content/uploads/2010/11/tablespace2.jpg)

1.  在配置文件中可以配置InnoDB的表空间<sup>[1]</sup>，一般格式如下(共享表空间)：
datadir = /opt/mysql/data
innodb_data_file_path=ibdata1:1G;ibdata2:1G;ibdata3:1G;ibdata4:1G;ibdata5:1G
2.  完整的表空间，会被分成如下结构供给InnoDB使用。最小单位是page，每个page为16K；64个连续的page组成一个extent；多个extent和page构成一个segment。Segment初始时InnoDB会为它分配32个pages，之后根据需要会将extent分配给segment，单次最多会分配4个extends给segment。<sup>[1]</sup>
3.  具体的，InnoDB中一个索引（B-tree）由两个segment组成。其中，所有的叶子节点（leaf nodes）存放在一个segment中，所有的非叶子节点（nonleaf nodes）存放在一个segment中。<sup>[1]</sup>
4.  一个存放记录(row)的page，由page header、page trailer、page body组成。如下图:<sup>[2]</sup>
[![](http://www.themysql.com/wp-content/uploads/2010/11/page_struct.png "page_struct")](page_struct.png)

from:[http://www.orczhou.com/index.php/2009/08/image-innodb-tablespace/](http://www.orczhou.com/index.php/2009/08/image-innodb-tablespace/)