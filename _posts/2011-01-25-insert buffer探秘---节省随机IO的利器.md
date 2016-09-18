---
layout: post 
title: "insert buffer探秘---节省随机IO的利器"
subTitle: 
heroImageUrl: 
date: 2011-1-25
tags: ["innodb","insert buffer","MySQL","mysql","performance"]
keywords: 
---

<span style="font-size: small;">原文出处：[http://www.mysqlperformanceblog.com/2009/01/13/some-little-known-facts-about-innodb-insert-buffer/](http://www.mysqlperformanceblog.com/2009/01/13/some-little-known-facts-about-innodb-insert-buffer/)</span>

<span style="font-size: small;">中文翻译：[http://hi.baidu.com/fishhust/blog/item/b9ffb3806ccaebde9023d919.html](http://hi.baidu.com/fishhust/blog/item/b9ffb3806ccaebde9023d919.html)</span>

<span style="font-size: small;">Despite being standard Innodb feature forever Insert Buffers remains some kind of mysterious thing for a lot of people, so let me try to explain thing a little bit.</span>

<span style="font-size: small;">Innodb uses insert buffer to "cheat" and not to update index leaf pages when at once but "buffer" such updates so several updates to the same page can be performed with single sweep.---<span style="color: #ff0000;">innodb使用insert buffer忽悠数据库:它并不马上更新索引的叶子页,而是把若干对同一页面的更新缓存起来,一次性更新</span>. Insert buffer only can work for non-unique keys because until the merge is performed it is impossible to check if the value is unique.----<span>insert buffer只能用在非唯一索引中,因为列的唯一性通过索引来保证,所以在索引没有被更新前,数据库并不知道索引是否是唯一的.如果用在唯一索引中,在insert merge的过程中,可能就会导致唯一键冲突.这和insert delayed是一个道理,而insert delayed可能导致冲突</span></span>

<span style="font-size: small;">Insert buffer is allocated in the Innodb system table space. Even though it is called "buffer" similar to "doublewrite buffer" it is really the space in the tablepace. ---<span>insert buffer和doublewrite buffer一样,都不是内存buffer,而是保存在磁盘上,公共的表空间中.</span>Though it can be cached in the buffer pool same as other pages. This property allows insert buffer to survive transaction commits and even MySQL restarts. ----<span>存储在硬盘上的好处是,即使机器重启,这个buffer也不会丢失.</span>Really it may take **weeks** before the given index page is merged, though usually it is much sooner than that.-----<span>insert buffer的数据真正merge到数据库可能会经过很长的时间</span></span>

<span style="font-size: small;">There are two ways of insert buffer merge is happening. First is on demand merge - if accessed page contains unmerged records in insert buffer the merge is performed before page is made available. This means insert buffer can slow down read operations.----<span>如果页面含有没有被merge的记录,那么这页将被标记为不可用,所以对unmerged 页面的读取操作,必须先等待merge操作完成,然后才能进行.这会降低读取的速度.另一方面,对页面的读取操作,会触发insert buffer的merge操作.</span></span>

<span style="font-size: small;">The other way insert buffer is merged is by background thread. There are very little merges happening if system is loaded and merge process becomes more active if system is idle. ----<span>另一种情况是,如果系统负载很重,insert buffer的merge操作只会缓慢的进行;反之,当系统没有什么负载时,merge操作会很活跃.</span>This behavior can cause interesting results, like you had system lightly used and have very little IO activity, but when you remove the load from the system completely you see high IO load which goes for hours even after all buffer pool dirty pages are completed. ----<span>dirty pages的刷新操作也很消耗资源,通常在关闭数据库的时候,你会看到IO很重,mysql正是在purge dirty pages,但是也许在merge insert buffer,同样,如果系统一点负载也没有,但是mysql很活跃,也不要奇怪,它可能正在merge insert buffer</span>.This can be very surprising.</span>

<span style="font-size: small;">Stats about Innodb Insert Merge Buffer are available in **SHOW INNODB STATUS** output:</span>

`
<span style="font-family: 新宋体; font-size: small;">-------------------------------------
INSERT BUFFER AND ADAPTIVE HASH INDEX
-------------------------------------
Ibuf: size 7545, free list len 3790, seg size 11336<span>,==这里size+free=seg size,表示总共的insert buffer的大小,他们的单位都是page
</span>8075308 inserts, 7540969 merged recs, 2246304 merges
</span>`

<span style="font-size: small;">The "seg size" is a full allocated size of segment in pages. So in this case it is about 180MB
The "free list" is number of pages which are free - containing no unmerged records. The <span style="color: #ff0000;">"size" is size (in pages) of insert buffer which is not merged.--<span>size里的是没有被merge的脏数据</span></span></span>

<span style="font-size: small;">The fact size is in pages is not really helpful because depending on the row size there can be different number of rows in the insert buffer --<span>insert buffer中的数据行大小不一致,所以不能根据size的大小来估计行的数量</span> and it is rows we see in performance stats, for example to understand when insert buffer merge will be completed.</span>

<span style="font-size: small;"><span style="color: #ff0000;">The "inserts" is number of inserts to insert buffer since start and number of merged records is number of records which were merged to their appropriate page locations since start.</span> ----<span>单位都是行.</span> So we know in this case insert buffer has grown 534339 records since start.---<span>inserts减去已经merge的即可</span> There is a temptation to use this number as count of unmerged rows in insert buffer but this would not be correct - <span style="color: #ff0000;">insert buffer may not be empty at the start. So you can only tell insert buffer has at least this number of records. For the same reason do not get scared if you see more merged records than inserted.--------<span>上面的3个值,都是mysql启动的时候开始计数的,从mysql启动以来,共插入了</span></span><span style="font-family: 新宋体; color: #000000;">8075308条,merge了7540969条,merge了2246304次.所以说,从mysql启动开始到现在,insert buffer中的数据增加了8075308-7540969=534339条.但是由于mysql启动的时候,insert buffer可能不是空的,所以现在insert buffer的数据应该>=534339条.</span></span>

<span style="font-size: small;">The value of 2246304 merges shows us there was about 3 records merged for each merge operation, meaning insert buffer could in theory reduce IO needed to update leaf pages 3 times.--<span style="font-family: 新宋体;">7540969/2246304=3.36,说明每次merge操作节省了大约3被的IO</span></span>

<span style="font-size: small;">As I mentioned Insert buffer merge can take quite a while - with 100 records merged per second we're looking at least 5343 seconds or 1.5 hours on this server... and there are insert buffers which are 10x and 100x larger than this.</span>

<span style="font-size: small;">Innodb unfortunately offers no control for insert buffer while it surely would be benefiting for different workloads and hardware configuration. For example there is very good question if insert buffer really makes sense for SSD because saving random IO is not so much needed for these devices.</span>