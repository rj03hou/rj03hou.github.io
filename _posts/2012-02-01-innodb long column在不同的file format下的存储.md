---
layout: post 
title: "innodb long column在不同的file format下的存储"
subTitle: 
heroImageUrl: 
date: 2012-2-1
tags: ["MySQL"]
keywords: 
---

innodb long column在不同的file format下的存储：
innodb_file_format=Antelope，ROW_FORMAT=COMPACT或者REDUNDANT时，保存前768个字节，剩下的会保存在overflow page中。
innodb_file_format=Barracuda，table is created with ROW_FORMAT=DYNAMIC or ROW_FORMAT=COMPRESSED， long column values are stored fully off-page, and the clustered index record contains only a 20-byte pointer to the overflow page.

facebook的Mark Callaghan进行了测试，补充了官方文档，一个page中可以存储下两行时可以不会将数据保存到overflow page中。（因为facebook被墙了，因此将文章贴在下面了）
How many pages does InnoDB for tables with large columns?
by Mark Callaghan on Friday, January 20, 2012 at 9:20pm

I am confused by the descriptions of when InnoDB uses external pages for long columns. There have been a few good blog posts about this but the official documentation isn't clear to me when it describes the old (Antelope) and new (Barracuda) file formats. I don't think I am alone in being confused.

The main point is that despite what the documentation might say and I think it is ambiguous, rows with a large column can be stored inline. By large column I mean one that is greater than 768 bytes. I prefer to have rows not use external pages because that can increase the number of disk reads required to fetch a row. I prefer the Dynamic row format versus the Compact row format because there is no benefit for my workload in storing the prefix of a large column inline. Queries in my workload select do not select a prefix of large columns and I suspect that the database will be smaller when the Dynamic format is used but I have yet to confirm that (a test is in progress).

I tested this for uncompressed InnoDB tables using the default page size of 16kb and a table with two columns (int and text). I inserted 131072 rows into the table and then ran a modified version of innochecksum to count the number of pages by type. I also looked at the size of the per-table ibd files. The InnoDB file format is Barracuda and the tables use row_format=Compact. The table was loaded four times using:
length(text) == 500 for all rows. This used 5376 pages total and 0 external pages.
length(text) == 1000 for all rows. This used 9984 pages total and 0 external pages.
length(text) == 6000 for all rows. This used 67072 pages total and 0 external pages. There were 2 rows per page as that used about 12kb on each page. This makes it clear that InnoDB can store large columns inline in leaf pages without using external pages as long as the size of the row is less than half the size of a page.
length(text) == 9000 for all rows. This used 160768 pages. Each row requires one overflow page. Innochecksum reports that there were 131072 pages with type FIL_PAGE_TYPE_BLOB, 7792 pages with type FIL_PAGE_TYPE_INDEX and 22383 pages with type FIL_PAGE_TYPE_ALLOCATED. This makes it clear that InnoDB uses a separate external page per large column as there were 131072 rows with one large column each and the same number of external pages.
I repeated the test using row_format=Dynamic and the same number of pages is used for the first three sizes for the text column: 500. Fewer pages were used for the Dynamic row format because a prefix of the large column is not stored inline and the number of FIL_PAGE_TYPE_INDEX pages is 2147 versus 7792. The number of FIL_PAGE_TYPE_ALLOCATED pages was also much less (2147 versus 22383) but I have yet to explain the reason for that.