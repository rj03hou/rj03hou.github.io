---
layout: single
title: websocket遇到的三个问题
description: websocket遇到的三个问题
headline: websocket遇到的三个问题
categories: nginx,websocket
headline: 
tags: [nginx,websocket]
comments: true
published: true
---



websocket基础不在这里赘述，简单来说，websocket分为握手阶段和数据传输阶段，握手阶段使用http协议，握手结束之后使用tcp协议，可以发送文本也可以发送二进制，通信比较高效，常用于持续的数据传输，比如交易所行情数据/棋牌类游戏的持续推送等。



###  一 invalid Upgrade header问题

有个服务一直 在报下面的问题：
Time 2019-03-20 10:48:40.322.
1|ERROR||||||o.s.w.s.s.s.DefaultHandshakeHandler:296|Handshake failed due to invalid Upgrade header: null)

检查nginx ingress controller的配置文件，发现ingress nginx controller针对所有的path都有如下设置：

    # Allow websocket connections
    proxy_set_header                        Upgrade           $http_upgrade;
    proxy_set_header                        Connection        $connection_upgrade;
然后测试确认，如果针对websocket的path，用户没有在request header中增加如下的header，则就会出现上面的错误：

 Connection: Upgrade
 Upgrade: websocket

继而对比nginx的access log和应用的异常日志，发现时间和次数吻合，基本上确认这个；

下一步检查nginx的access log，发现这些错误都来自于同一个ip；则在waf中将这个ip禁用一段时间；check监控，发现错误完全消失。

### 二  websocket ingress nginx报503错误

奇怪的是，这次在nginx access log和error log中没有发现任何线索；（因为default的access log是默认被关闭的，找了没有找到如何打开）

后来在网上search之后，发现一般的原因都是因为svc名字或者端口写错

### 三 websocket接口报502错误

详情见[cloudfront遇到的502问题](https://rj03hou.github.io/aws,cloudfront/cloudfront%E9%81%87%E5%88%B0%E7%9A%84502%E9%97%AE%E9%A2%98/)