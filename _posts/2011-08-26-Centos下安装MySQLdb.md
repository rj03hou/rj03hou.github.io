---
layout: post 
title: "Centos下安装MySQLdb"
subTitle: 
heroImageUrl: 
date: 2011-8-26
tags: ["MySQL","Python"]
keywords: 
---

安装之前需要先安装
①MySQL-devel-VERSION.i386.rpm。The libraries and include files that are needed if you want to compile other MySQL clients, such as the Perl modules。如果不安装会出现"EnvironmentError: mysql_config not found"。
②MySQL-shared-VERSION.i386.rpm。This package contains the shared libraries (libmysqlclient.so*) that certain languages and applications need to dynamically load and use MySQL。如果不安装会出现"/usr/bin/ld: cannot find -lmysqlclient_r"

注意：
5.5.8的shared出现问题，导致一直提示"/usr/bin/ld: cannot find -lmysqlclient_r"，安装5.5.15之后就问题
x86对应32位版本,amd64对应x86_64版本

方法一：
wget -q http://peak.telecommunity.com/dist/ez_setup.py
python ez_setup.py
easy_install是由PEAK(Python Enterprise Application Kit)开发的setuptools包里带的一个命令，所以使用easy_install实际上是在调用setuptools来完成安装模块的工作。

easy_install MySQL-python

方法二：
下载MySQL-python-1.2.3.tar.gz;tar xfz MySQL-python-1.2.1.tar.gz;python ez_setup.py;python setup.py build;python setup.py install