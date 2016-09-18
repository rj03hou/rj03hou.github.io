---
layout: post 
title: "编译安装xtrabackup"
subTitle: 
heroImageUrl: 
date: 2011-6-25
tags: ["MySQL","Percona"]
keywords: 
---

xtrabackup是percona公司创建并维护的项目，提供innodb引擎的在线备份。
xtrabackup提供了两种命令行工具：
xtrabackup：用于备份InnoDB引擎的数据（不会备份myisam比如mysql权限相关表等，也不会自动copy frm文件）；
innobackupex：一个perl脚本，在执行过程中会调用xtrabackup命令，用该命令即可以备份InnoDB，也可以备份MyISAM/copy frm文件。

1，下载xtrabackup源代码,www.percona.com
2，解压xtrabackup源代码，在解压之后的根目录下执行。注意
①不能进入到utils目录中
②不能使用sh来执行，因为ubuntu下sh默认使用的是dash而不是bash
③最后代表mysql的版本号，执行时要指定与之匹配的MySQL数据库版本。
④解压之后目录下就已经存在innobackupex脚本，innobackupex脚本调用xtrabackup命令进行备份，因此实际上是在编译安装xtrabackup

/win/xtrabackup-1.6# ./utils/build.sh 5.1

因为是编译安装，因此包依赖关系需要一点点的处理
bzr: command not found
apt-get install bzr

BUILD/autorun.sh: line 41: aclocal: command not found
apt-get install autoconf

BUILD/autorun.sh: line 44: libtoolize: command not found
apt-get install libtool

命令执行完成之后，xtrabackup就可以用了，一般默认编译后，该文件保存在xtrabackup-1.6/mysql-5.1.56/storage/innobase/xtrabackup/目录下，当然也可以通过find命令查找xtrabackup的具体路径。

为方便使用创建两个软连接：
<pre lang="sh">
ln -s /usr/local/xtrabackup-1.6/innobackupex /bin/innobackupex
ln -s /usr/local/xtrabackup-1.6/mysql-5.1.56/storage/innobase/xtrabackup/xtrabackup_51 /bin/xtrabackup_51
</pre>

下来就可以使用了，下面的脚本是一个简单的mysql全量备份脚本

<pre lang="sh">
#!/bin/sh
#The location to save backup file,the location should be dedicated to backup,when backup finish it should be move to other safe place.
backup_loc="/data/dbbak"
innobackupex --user=root --password=123 --throttle=100 --slave-info $backup_loc
if [ $? -ne 0 ]
then
    exit
fi
backup_name=`ls  $backup_loc ' grep 20`
innobackupex --apply-log --ibbackup=xtrabackup_51 /data/dbbak/$backup_name >/data/dbbak/xtrabackup.log 2>&1
tar -cjf $backup_name.tar.bz2 ./$backup_name/
#copy...
#rm -rf $backup_loc
</pre>