---
layout: post 
title: "windows本地可以使用xampp搭建mysql测试环境"
subTitle: 
heroImageUrl: 
date: 2010-9-24
tags: ["MySQL","mysql","phpMyAdmin","xampp"]
keywords: 
---

在windows上面搭建测试环境，可以使用xampp来迅速搭建，然后使用web管理工具phpMyAdmin来管理MySQL。[xampp下载地址](http://www.apachefriends.org/zh_cn/xampp.html)。

安装的时候需要将xampp安装在、根目录下，否则启动Apache时会出现下面的错误（使用xampp crontrl 启动Apache的时候提示busy，查看端口80也没有被占用）：

"xampp apache serverroot must be a valid directory"

xampp包括了安装十分便捷，解压之后执行setup_xampp.bat就可以安装，安装包括了Apache、php、MySQL、phpMyAdmin。

正常情况下安装完毕之后，打开这个页面http://localhost/phpmyadmin/，会出现错误，提示mysql无法连接。解决方法，进入到xampp解压目录（绿色的也就是安装目录），xampp\phpMyAdmin，编辑config.inc.php文件，将$cfg['Servers'][$i]['host'] = 'localhost';改成 $cfg['Servers'][$i]['host'] = '127.0.0.1′;因为mysql连接字符串中假如是localhost的话就会使用socket文件（linux的连接方式来连接），改成127.0.0.1之后，使用tcp进行连接。

就可以方便快捷的在windows本地使用mysql了。

也可以独立安装，使用过程中遇到其他问题，可以参照xampp的[官方QA](http://www.apachefriends.org/zh_cn/faq-xampp-windows.html)