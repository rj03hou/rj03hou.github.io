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

### 四 websocket连接断开

Server: nginx/1.12.2
chart-version: nginx-ingress-1.27.0 0.26.1

**两种场景下会断开重连**，1.ingress变更导致ng reload 2.ws连接的service因为deployment修改引起的service重新发布；

1. 修改ingress 比如修改path对应的backend，增加path等；修改和ws不相关的ingress也会reload；
  kubectl edit ingress ws-bhexb-com -n preview
  kubectl logs nginx-ingress-controller-76b94-hj27w nginx-ingress-controller|grep reload

  I0508 04:15:31.588682       6 controller.go:134] Configuration changes detected, backend reload required.
  I0508 04:15:31.863294       6 controller.go:150] Backend successfully reloaded.

  2020-05-08 12:19:30
  {"pong":1588911570891}
  Connection is already closed.

2. ingress不变，deployment发生变化；不会引起realod；
比如replica数量，比如request/limit调整，比如发布代码调整image等；
修改request没有引发reload；
修改ws连接的pod导致ws断开连接；



解决方法：

1. 针对长连接的nginx进行拆分；
2. 针对长连接的service前面增加一个gateway用于连接保持；