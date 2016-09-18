---
layout: post 
title: "使用innobackupex进行备份"
subTitle: 
heroImageUrl: 
date: 2010-11-5
tags: ["innobackupex","MySQL","Percona","原创","未完待续"]
keywords: 
---

<div id="_mcePaste">
<div id="_mcePaste">安装xtrabackup会自动安装innobackupex，innobackupex是使用perl包装的xtrabackup，下面是一些使用心得，仍然有几个问题没有解决：</div>
<div id="_mcePaste">①如何可以避免使用tar，因为如果需要备份的数据特别大的时候，使用tar解压需要很久</div>
<div id="_mcePaste">②nc的-d选项为什么会对nc的传输产生影响</div>
<div id="_mcePaste">nc需要加上-d选项，否则会出现截断现象，详细见(未解决)</div>
<div id="_mcePaste">http://topic.csdn.net/u/20101102/19/8e641dae-d4e1-45c6-8e6f-14b67b9500b0.html?56071</div>
<div id="_mcePaste">ssh root@host_name "( nc -d -l 80 > /data/backup.tar 2>/dev/null &)" && innobackupex-1.5.1 --throttle=500 --user=root --password=XXXXX --stream=tar --slave-info ./ '  nc host_name 80</div>
<div id="_mcePaste">出现下面的问题</div>
<div id="_mcePaste">innobackupex-1.5.1: Created backup directory /root</div>
<div id="_mcePaste">innobackupex-1.5.1: Error: Failed to stream 'backup-my.cnf': Inappropriate ioctl for device at /usr/bin/innobackupex-1.5.1 line 479.</div>
<div id="_mcePaste">当我把./改成/data时就ok了</div>
<div id="_mcePaste">ssh root@host_name "( nc -d -l 80 > /data/backup.tar 2>/dev/null &)" && innobackupex-1.5.1 --throttle=500 --user=root --password=XXXXX --stream=tar --slave-info /data '  nc host_name 80</div>
<div id="_mcePaste">innobackupex-1.5.1 -user=root -password=123 -stream=tar /u01/backup/2/ 2>/u01/backup/2.log 1>/u01/backup/2/2.tar</div>
<div id="_mcePaste">大约2分55秒。这里使用1>做标准输出重定向。</div>
<div id="_mcePaste">innobackupex-1.5.1 -user=root -password=123 /u01/backup/1/innobackup/ 2>/u01/backup/1/1.log</div>
<div id="_mcePaste">是将备份过程中的输出信息重定向到1.log</div>
<div id="_mcePaste">备份完成之后，需要执行两次prepare操作</div>
</div>