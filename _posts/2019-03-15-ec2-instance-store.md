---
layout: single
title: ec2-instance-store
description: ec2-instance-store
headline: ec2-instance-store
categories: aws,ec2
headline: 
tags: [aws,ec2]
comments: true
published: true
---

每个EC2上都带了一个instance store，instance store区别于ebs，就是机器上的本地盘（通过lsblk可以看到），根据instance type挂载的大小是不一样的；相比ebs是免费的，但是存在一个问题：

The data in an instance store persists only during the lifetime of its associated instance. If an instance reboots (intentionally or unintentionally), data in the instance store persists. However, data in the instance store is lost under any of the following circumstances:

- The underlying disk drive fails
- The instance stops
- The instance terminates

我们就遇到这个问题，我们的prometheus server在开始的部署的时候就使用了instance store，并且挂载到了/data目录下，修改了/etc/fstab进行挂载

```
/dev/nvme1n1                              /data                ext4    defaults        0 0
```

随后我们的prometheus server没有响应，于是ops想要重启这台机器（但是实际的操作是先关闭了，然后再启动），结果发现启动之后无法登录，Instance Status Checks也失败。

联系aws support之后，support发现问题所在：

```
实例并没有正常开机进入系统，但是进入emergency mode，并且有无法挂载/data的console讯息
===
...
[    4.715756] EXT4-fs (nvme1n1): VFS: Can't find ext4 filesystem
[[1;31mFAILED[0m] Failed to mount /data.
...
Welcome to emergency mode! After logging in, type "journalctl -xb" to view
system logs, "systemctl reboot" to reboot, "systemctl default" or ^D to
try again to boot into default mode.
===
```

再support的指导下做了如下操作：

1. 将老机器关机，将老机器的root ebs deattach；

2. 将老机器的root ebs attach到另外一台机器上，然后mount到比如/data目录下；

3. 登录另外的机器，编辑/data/etc/fstab注释掉原来对/data的挂载；
4. 将root ebs deattach，然后attach到老的机器；启动老的机器；

但是之前的/data中的数据全部丢失。



[官方文档](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/InstanceStorage.html)

