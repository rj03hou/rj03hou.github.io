---
layout: post 
title: "fedora&ubuntu下安装synergy"
subTitle: 
heroImageUrl: 
date: 2011-5-12
tags: ["fedoral","Linux","linux","tools"]
keywords: 
---

synergy是一款在多个平台下共享鼠标键盘的软件。下面描述如何在fedora下面配置synergy。主要参照军辉的[我的工作模式](http://blog.renren.com/blog/227366242/711287907)
本来想去官方网站上下载1.4.2进行安装的时候，发现了一大堆的依赖关系，因此使用了使用yum安装了synergy-plus-1.3.4-6.fc14.i686，对应的windows版的[下载地址](http://code.google.com/p/synergy-plus/downloads/list?can=1&q=)；ubuntu下使用apt-get安装。

服务器端配置(fedora和ubuntu下的配置相同，默认没有synergy.conf，因此需要手工创建)：
1，配置/etc/synergy.conf，将linux-name替换成fedora主机的host-name，windows-name也替换成windows下的名称。**注意linux-name和windows-name后的:**。

section: screens
linux-name:
windows-name:
end
section: links
windows-name:
right = linux-name
linux-name:
left = windows-name
end

2，在fedora的Firewall Configuration-Other Ports中添加24800 TCP，否则会出现
failed to connect to server: Timed out

在windows的客户端：
Other Computer's Host Name:这个地方一定要填写服务器的IP地址，否则会提示address not found

Options中的Advanced选项中填写的host-name要在服务器的configure文件中配置，否则会提示Server refused client with name "XXX"  A client with name "XXX" is not in the map。