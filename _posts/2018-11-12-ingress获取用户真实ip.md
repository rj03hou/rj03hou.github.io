---
layout: single
title: ingress如何获取用户真实ip
description: ingress如何获取用户真实ip
headline: ingress如何获取用户真实ip
categories: aws,k8s,ingress
headline:
tags: [aws,k8s,ingress]
comments: true
published: true

---

### 配置文件

在aws上使用kops部署的k8s集群，获取用户真实ip的过程

通过观察aws上ingress-nginx的生成的配置文件

```nginx
#--------修改remote-addr
72     real_ip_header      X-Forwarded-For;
73
74     real_ip_recursive   on;
75
76     set_real_ip_from    0.0.0.0/0;

#--------修改$the_real_ip
172     # The following is a sneaky way to do "set $the_real_ip $remote_addr"
173     # Needed because using set is not allowed outside server blocks.
174     map '' $the_real_ip {
175
176         default          $remote_addr;
177
178     }

1564             proxy_set_header X-Real-IP              $the_real_ip;
1565
1566             proxy_set_header X-Forwarded-For        $the_real_ip;
```

上面配置文件的一个解释说明，

### 第一部分

修改remote-addr，nginx从连接ip和x-forwarded-for头部来进行遍历，先判断连接ip，然后再从右向左遍历x-forwarded-for；如果发现ip在set_real_ip_from范围内，继续往前搜索，直到第一个不在set_real_ip_from范围内的ip。

而0.0.0.0/0表示的所有ip，所以在默认配置下，会持续的向右遍历，直到最后一个ip，所以这个时候用户伪造的x-forwarded-for，伪造的最左边的ip就会被修改为remote-addr。

在`real_ip_recursive on`的情况下，`128.22.189.11`、`192.168.2.100`都出现在set_real_ip_from中,仅仅`222.11.158.67`没出现,那么这个IP就被认为是用户的IP地址，并且赋值到`remote_addr`变量。

在`real_ip_recursive off`或者不设置的情况下,`192.168.2.100`出现在了`set_real_ip_from`中会被排除掉，其它的IP地址便认为是用户的ip地址。

### 第二部分

将remote_addr覆盖到X-Real-IP和X-Forwarded-For的header。



### 现象

通过上面的默认配置，用户伪造X-Forwarded-For，获取的X-Real-IP，就是最左边伪造的ip；



### 如何解决

通过修改nginx configmap，在data section下面设置proxy-real-ip-cidr:内网ip和k8s

实际上会更新set_real_ip_from，第一个真实ip（连接到CDN，CDN会将连接ip追加到X-Forwarded-For中），所以这个时候获取到的X-Real-IP就是用户的真实ip。

同事提出了另外一个解决方案就是使用PROXY Protocol，为什么不能用这个原因，需要配合real-ip-header设置为proxy_protocol；而这个时候获取的ip就变成了CDN的ip了。



### 几个有意思的现象
1.如果set_real_ip_from 0.0.0.0/0，意味所有的ip都是信任，所以现象X-Real-IP，X-Forwarded-For最左边的一个；
2.如果set_real_ip_from 不包括127.0.0.1，在本机curl，会发现X-Real-IP就是127.0.0.1；经过测试也说明了nginx在real_ip_header X-Forwarded-For，修改remote_addr是从连接ip和x-forwarded-for头部来进行遍历，先判断连接ip，然后再从右向左遍历x-forwarded-for；



### 参照

1. [nginx-controller configmap](https://github.com/kubernetes/ingress-nginx/blob/master/docs/user-guide/nginx-configuration/configmap.md)
2. [PROXY Protocol](https://github.com/nginxinc/kubernetes-ingress/tree/master/examples/proxy-protocol)