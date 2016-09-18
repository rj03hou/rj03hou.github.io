---
layout: post 
title: "sysbench找不到libmysqlclient_r.so.15解决方法"
subTitle: 
heroImageUrl: 
date: 2010-9-27
tags: ["Linux","MySQL","MySQL-shared-compat","sybench"]
keywords: 
---

运行sysbench时，提示类似下面的error:

./sysbench: error while loading shared libraries: libmysqlclient_r.so.15: cannot open shared object file: No such file or directory

安装对应版本的：

RedHat Enterprise Linux 5 RPM (AMD64, Dynamic client libraries (including 3.23.x libraries)) (10 Jul 2007, 3.1M)

注：AMD64位对应x86_64

5.0.45对应的是MySQL-shared-compat-5.0.45-0.rhel5.x86_64.rpm

安装Percona时，安装Percona-Server-shared-51-5.1.50-rel11.4.111.rhel5.x86_64.rpm后，依然提示找不到libmysqlclient_r.so.15，卸载之后安装对应版本的MySQL-shared-compat-5.1.50-1.rhel5.x86_64.rpm，就好了。

在percona的[trac](https://bugs.launchpad.net/percona-xtradb/+bug/386054)上可以看到此前这个被举报为一个bug，按照下面的状态显示应该已经被fixed的，不知道因为什么原因又出来。

另外如果是编译安装的MySQL，可以参考下面的做法：

/usr/local/sysbench/bin/sysbench: error while loading shared libraries: libmysqlclient_r.so.15: cannot open shared object file: No such file or directory找不到连接库，解决方法：①在/usr/local/mysql55/lib/目录下首先执行ln -s libmysqlclient.so.18.0.0 libmysqlclient_r.so.15②编辑/etc/lld.so.conf文件运行ldconfig(ldconfig的作用使得/etc/ld.so.conf文件的变更生效)
或者export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/mysql55/lib长期有效的添加到.bash_profileLD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/libexport LD_LIBRARY_PATH