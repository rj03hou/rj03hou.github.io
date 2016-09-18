---
layout: post 
title: "Mysql调优中两个重要参数table_cache和key_buffer_size"
subTitle: 
heroImageUrl: 
date: 2010-10-12
tags: ["MySQL"]
keywords: 
---

<div>
<div id="_mcePaste">题记：</div>
<div id="_mcePaste">本文根据我自己的一点经验，讨论了Mysql服务器优化中两个非常重要的参数，分别是table_cache，key_buffer_size。</div>
<div id="_mcePaste">table_cache指示表高速缓存的大小。当Mysql访问一个表时，如果在Mysql表缓冲区中还有空间，那么这个表就被打开并放入表缓冲区，这样做的好处是可以更快速地访问表中的内容。一般来说，可以通过查看数据库运行峰值时间的状态值Open_tables和Opened_tables，用以判断是否需要增加table_cache的值，即如果open_tables接近table_cache的时候，并且Opened_tables这个值在逐步增加，那就要考虑增加这个值的大小了。</div>
<div id="_mcePaste">在mysql默认安装情况下，table_cache的值在2G内存以下的机器中的值默认时256到512，如果机器有4G内存,则默认这个值是2048，但这决意味着机器内存越大，这个值应该越大，因为table_cache加大后，使得mysql对SQL响应的速度更快了，不可避免的会产生更多的死锁（dead lock），这样反而使得数据库整个一套操作慢了下来，严重影响性能。所以平时维护中还是要根据库的实际情况去作出判断，找到最适合你维护的库的table_cache值，有人说："性能优化是一门艺术"，这话一点没错。大凡艺术品，大都是经过千锤百炼，精雕细琢而成。</div>
<div id="_mcePaste">这里还要说明一个问题，就是table_cache加大后碰到文件描述符不够用的问题，在mysql的配置文件中有这么一段提示</div>
<div id="_mcePaste">Quotation</div>
<div id="_mcePaste">"The number of open tables for all threads. Increasing this value increases the number of file descriptors that mysqld requires.</div>
<div id="_mcePaste">Therefore you have to make sure to set the amount of open files allowed to at least 4096 in the variable "open-files-limit" in" section [mysqld_safe]"</div>
<div id="_mcePaste">说的就是要注意这个问题，一想到这里，部分兄弟可能会用ulimit -n 作出调整，但是这个调整实际是不对的，换个终端后，这个值又会回到原始值，所以最好用sysctl或者修改/etc/sysctl.conf文件，同时还要在配置文件中把open_files_limit这个参数增大，对于4G内存服务器，相信现在购买的服务器都差不多用4G的了，那这个这个open_files_limit至少要增大到4096，如果没有什么特殊情况，设置成8192就可以了。</div>
<div id="_mcePaste">下面说说key_buffer_size这个参数，key_buffer_sizeO表示索引缓冲区的大小，严格说是它决定了数据库索引处理的速度，尤其是索引读的速度。根据网络一些高手写的文章表示可以检查状态值Key_read_requests和Key_reads，即可知道key_buffer_size设置是否合理。比例key_reads / key_read_requests应该尽可能的低，至少是1:100，1:1000更好，虽然我还没有找到理论的依据，但是，我在自己维护的几台实际运行良好的库做过的测试后表明，这个比值接近1：20000，这从结果证明了他们说这话的正确性，我们不妨用之。</div>
<div id="_mcePaste">后记：</div>
<div id="_mcePaste">我前面说过，性能优化是一件细活，影响mysql性能的因素很多，本文中只是选取了其中我认为比较重要的两个参数，期待和网友一起探讨更多mysql性能优化的技术。</div>
</div>
<div>题记：     本文根据我自己的一点经验，讨论了Mysql服务器优化中两个非常重要的参数，分别是table_cache，key_buffer_size。
table_cache指示表高速缓存的大小。当Mysql访问一个表时，如果在Mysql表缓冲区中还有空间，那么这个表就被打开并放入表缓冲区，这样做的好处是可以更快速地访问表中的内容。一般来说，可以通过查看数据库运行峰值时间的状态值Open_tables和Opened_tables，用以判断是否需要增加table_cache的值，即如果open_tables接近table_cache的时候，并且Opened_tables这个值在逐步增加，那就要考虑增加这个值的大小了。
在mysql默认安装情况下，table_cache的值在2G内存以下的机器中的值默认时256到512，如果机器有4G内存,则默认这个值是2048，但这决意味着机器内存越大，这个值应该越大，因为table_cache加大后，使得mysql对SQL响应的速度更快了，不可避免的会产生更多的死锁（dead lock），这样反而使得数据库整个一套操作慢了下来，严重影响性能。所以平时维护中还是要根据库的实际情况去作出判断，找到最适合你维护的库的table_cache值，有人说："性能优化是一门艺术"，这话一点没错。大凡艺术品，大都是经过千锤百炼，精雕细琢而成。
这里还要说明一个问题，就是table_cache加大后碰到文件描述符不够用的问题，在mysql的配置文件中有这么一段提示Quotation"The number of open tables for all threads. Increasing this value increases the number of file descriptors that mysqld requires.Therefore you have to make sure to set the amount of open files allowed to at least 4096 in the variable "open-files-limit" in" section [mysqld_safe]"说的就是要注意这个问题，一想到这里，部分兄弟可能会用ulimit -n 作出调整，但是这个调整实际是不对的，换个终端后，这个值又会回到原始值，所以最好用sysctl或者修改/etc/sysctl.conf文件，同时还要在配置文件中把open_files_limit这个参数增大，对于4G内存服务器，相信现在购买的服务器都差不多用4G的了，那这个这个open_files_limit至少要增大到4096，如果没有什么特殊情况，设置成8192就可以了。

下面说说key_buffer_size这个参数，key_buffer_sizeO表示索引缓冲区的大小，严格说是它决定了数据库索引处理的速度，尤其是索引读的速度。根据网络一些高手写的文章表示可以检查状态值Key_read_requests和Key_reads，即可知道key_buffer_size设置是否合理。比例key_reads / key_read_requests应该尽可能的低，至少是1:100，1:1000更好，虽然我还没有找到理论的依据，但是，我在自己维护的几台实际运行良好的库做过的测试后表明，这个比值接近1：20000，这从结果证明了他们说这话的正确性，我们不妨用之。
后记：   我前面说过，性能优化是一件细活，影响mysql性能的因素很多，本文中只是选取了其中我认为比较重要的两个参数，期待和网友一起探讨更多mysql性能优化的技术。

<span style="font-family: Arial, 'Liberation Sans', 'DejaVu Sans', sans-serif; line-height: 18px; border-collapse: collapse; font-size: 14px;">

`innodb_buffer_pool_size` is the setting that controls the size of the memory buffer that InnoDB uses to cache indexes _and_ data. It's an important performance option.

See the [manual page](http://dev.mysql.com/doc/refman/5.0/en/innodb-parameters.html#sysvar%5Finnodb%5Fbuffer%5Fpool%5Fsize) for the full explanation. The [MySQL Performance Blog](http://www.mysqlperformanceblog.com/2007/11/03/choosing-innodb_buffer_pool_size/) also has an article about how to choose a proper size for it.

key_buffer_size主要用于myisam引擎，innodb引擎不用设置该选项。

</span>

</div>
<div>转自：[http://www.askwan.com/post/4/](http://www.askwan.com/post/4/)</div>
<div></div>