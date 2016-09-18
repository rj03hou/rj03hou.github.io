---
layout: post 
title: "crontab中运行python程序出错，提示ImportError: No module named解决全过程"
subTitle: 
heroImageUrl: 
date: 2010-9-24
tags: ["crontab","Linux","Python","python"]
keywords: 
---

将一个python脚本放入crontab执行时，提示如下错：
ImportError: No module named hashlib
但是在shell中直接执行时没有任何问题，google之后，得到线索是PYTHONPATH的问题，PYTHONPATH会决定python查找lib的路径。
在服务器上面echo $PYTHONPATH的时候没有任何路径
继续调查发现最终影响的是sys.path
分别输出了两种场景中的sys.path

shell:
[root@ short_task]# python
Python 2.6.2 (r262:71600, Aug  7 2009, 18:39:16)
[GCC 4.1.2 20080704 (Red Hat 4.1.2-44)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import sys
>>> print sys.path
['', '/usr/local/lib/python2.6/site-packages/setuptools-0.6c5-py2.6.egg', '/usr/local/lib/python2.6/site-packages/MySQL_python-1.2.2-py2.6-linux-x86_64.egg', '/home/houjw/short_task', '/home/bonny/sqlLib', '/usr/local/lib/python26.zip', '/usr/local/lib/python2.6', '/usr/local/lib/python2.6/plat-linux2', '/usr/local/lib/python2.6/lib-tk', '/usr/local/lib/python2.6/lib-old', '/usr/local/lib/python2.6/lib-dynload', '/usr/local/lib/python2.6/site-packages']

crontab:
于是修改脚本，查看当脚本在crontab执行时的syspath是多少
[root@ short_task]# less get_email_hash.log
['/home/houjw/short_task', '/usr/lib64/python24.zip', '/usr/lib64/python2.4', '/usr/lib64/python2.4/plat-linux2', '/usr/lib64/python2.4/lib-tk', '/usr/lib64/python2.4/lib-dynloa
d', '/usr/lib64/python2.4/site-packages', '/usr/lib64/python2.4/site-packages/Numeric', '/usr/lib64/python2.4/site-packages/gtk-2.0', '/usr/lib/python2.4/site-packages']
Traceback (most recent call last):
File "/home/houjw/short_task/get_email_hash.py", line 7, in ?
import hashlib
ImportError: No module named hashlib

然后研究了一下sys.patch的生成方式：
A list of strings that specifies the search path for modules. Initialized from the environment variable PYTHONPATH, plus an installation-dependent default.
这个不仅与PYTHONPATH有关系，而且与installation-dependent default有关系，这个估计与python的安装有关系，而且通过上面的sys.path输出发现机器上安装了两个python2.4和2.6，说明crontab中用到的是2.4，而shell中用到的是2.6，hashlib正好是在2.5的时候加入python的，所以2.4就没有找到。

通过cat crontab发现crontab中的PATH变量首先发现的是2.4的python

于是问题就得到了解决，在crontab中使用/usr/loca/bin/python XXX.python而不是python XXX.python或者将XXX.python修改为可执行文件，在python头部#!/usr/local/bin/python