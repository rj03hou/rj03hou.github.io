---
layout: post 
title: "使用redis-py访问Redis"
subTitle: 
heroImageUrl: 
date: 2012-9-10
tags: ["NoSQL","Python"]
keywords: 
---

**使用redis-py访问Redis**

**一、安装**
在[官方网站](https://github.com/andymccurdy/redis-py)下载redis-py
执行python setup.py install进行安装

**二、使用**
<pre lang="python">
import redis
r=redis.Redis(host="10.75.18.59",port=7001,db=0)
r.info()
{'pubsub_channels': 0, 'used_memory_peak_human': '5.28M', 'aof_buffer_length': 0, 'bgrewriteaof_in_progress': 0, 'aof_current_size': 0, 'connected_slaves': 0, 'aof_base_size': 0, 'uptime_in_days': 0, 'multiplexing_api': 'epoll', 'lru_clock': 507056, 'last_save_time': 1347247101, 'redis_version': '2.4.15', 'redis_git_sha1': 0, 'gcc_version': '4.1.2', 'connected_clients': 3, 'redis_release': 1, 'keyspace_misses': 0, 'used_memory': 5544616, 'vm_enabled': 0, 'used_cpu_user_children': 0.0, 'aof_pending_rewrite': 0, 'used_memory_peak': 5536016, 'role': 'master', 'total_commands_processed': 106, 'latest_fork_usec': 0, 'loading': 0, 'used_memory_rss': 5935104, 'total_connections_received': 10, 'pubsub_patterns': 0, 'aof_enabled': 1, 'used_cpu_sys': 0.01, 'used_memory_human': '5.29M', 'used_cpu_sys_children': 0.0, 'blocked_clients': 0, 'used_cpu_user': 0.02, 'client_biggest_input_buf': 0, 'arch_bits': 64, 'mem_fragmentation_ratio': 1.07, 'expired_keys': 0, 'evicted_keys': 0, 'bgsave_in_progress': 0, 'client_longest_output_list': 0, 'mem_allocator': 'jemalloc-3.0.0', 'total_read_requests': 0, 'total_write_requests': 0, 'process_id': 5415, 'uptime_in_seconds': 741, 'changes_since_last_save': 0, 'redis_git_dirty': 0, 'pending_aofbuf_length': 0, 'keyspace_hits': 0}
r.info()["used_memory_human"]
'3.04G'
#使用完成之后需要手工调用disconnect释放资源，虽然connection的析构函数中会断开连接，但是类似这种连接最好是手工进行释放，更好的是放在finally中确保释放，防止对服务器端产生影响。
r.connection_pool.disconnect()
</pre>