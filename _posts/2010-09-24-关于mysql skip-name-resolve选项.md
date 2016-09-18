---
layout: post 
title: "关于mysql skip-name-resolve选项"
subTitle: 
heroImageUrl: 
date: 2010-9-24
tags: ["MySQL","mysql","option"]
keywords: 
---

When a new thread connects to mysqld, mysqld will spawn a new thread to handle the request. This thread will first check if the hostname is in the hostname cache. If not the thread will call gethostbyaddr_r() and gethostbyname_r() to resolve the hostname.

If the operating system doesn't support the above thread-safe calls, the thread will lock a mutex and call gethostbyaddr() and gethostbyname() instead. Note that in this case no other thread can resolve other hostnames that is not in the hostname cache until the first thread is ready.

You can disable DNS host lookup by starting mysqld with --skip-name-resolve. In this case you can however only use IP names in the MySQL privilege tables.

If you have a very slow DNS and many hosts, you can get more performance by either disabling DNS lookop with --skip-name-resolve or by increasing the HOST_CACHE_SIZE define (default: 128) and recompile mysqld.

You can disable the hostname cache with --skip-host-cache. You can clear the hostname cache with FLUSH HOSTS or mysqladmin flush-hosts.

If you don't want to allow connections over TCP/IP, you can do this by starting mysqld with --skip-networking.

大致翻译如下：

当一个新连接连接mysql服务器时，mysql服务器会对此次连接的合法性进行判定，具体通过查询mysql.user表实现。mysql的权限设置将user和host（客户端的地址）联系起来，只有当两者都符合条件时才能进行下一步认证。

当 客户端连接的时候，客户端的地址假如不在mysql.host表中时，mysql服务器会调用gethostbyaddr和gethostbyname名 字进行解析（同步方法），或者gethostbyaddr_r和gethostbyname_r（异步）来解析客户端地址，这样会导致效率下降。

因此建议安装完毕之后从my.cnf中删除skip-name-resolve，然后在调用grant命令时全部写成ip地址。

假如以前的mysql.user表中host列存在host-name，设置skip-name-resolve时，会出现如下的warning：

[Warning] 'user' entry 'root@XXX.com' ignored in --skip-name-resolve mode.

删掉（因为已经改成ip认证的形式了）重启就ok了