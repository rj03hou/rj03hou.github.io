---
layout: single
title: 如何使用vpc peering和vpc endpoint service打通两个aws vpc之间的服务
description: 如何使用vpc peering和vpc endpoint service打通两个aws vpc之间的服务
headline: 如何使用vpc peering和vpc endpoint service打通两个aws vpc之间的服务
categories: aws vpc
headline: 
tags: [aws, vpc]
comments: true
published: true
---


实际的场景下，因为隔离的原因，会划分成多个账号，或者同一个账号划分成多个VPC，这个时候就会遇到VPC之间服务互相访问的问题，目前经常使用的两种方式，vpc peering和vpc endpoint service。区别主要vpc-peering等同于将两个vpc网络全部打通，只能依赖于security group进行安全隔离了，安全性不如vpc endpoint service，这个只是将某个service暴露给受限的vpc。

### VPC-Peering

**规则：默认不打通，除非必要才打通**

以不同账户A和B为例子

#### 一 登陆B账号

Services->VPC->Peering Connections->create peering connection

| 参数                        | 设置                                                         |
| --------------------------- | ------------------------------------------------------------ |
| Peering connection name tag | connect=A账户名                                              |
| VPC (Requester)*            | 选择要连接的VPC，选择我们之前创建的VPC                       |
| Account                     | Another account                                              |
| Account ID*                 | 找到需要连接的Account的维护者，让其登陆A，点击右上角的账户名，下拉列表中选择My Account，可以看到 Account Id，输入到刚才的输入框中 |
| VPC (Accepter)*             | Account A登陆其账户获取到到需要连接的VPC-id，输入进去        |

点击 Create Peering Connection

#### 二 账户A登陆

Services->VPC->Peering Connections

选择B发起的Peering Connection，Actions->Accept Request->Yes Accept->close

#### 三 修改路由表

**分别修改路由表**（两个账户都需要分别操作，方法相同，只是参数换成对方的参数）

账户B

Services->VPC->Route Tables找到以private-ap开头的route table

点击下方的Routes，点击Edit，点击add another route

| 参数        | 值                                       |
| ----------- | ---------------------------------------- |
| Destination | 输入A的VPC的CIDR值，                     |
| Target      | 选择刚才创建的VPC Peering，一般以pcx开头 |

点击save

####  四 修改安全组

安全规则：白名单模式，开通需要访问的端口，其他deny**

分别登陆A/B账户根据需求放开一些端口的访问

service->vpc->Security Groups

因为Security Groups的规则中没有配置所以虽然两个VPC现在是联通的，但是之间是无法访问的

我们全部使用k8s集群，而集群暴露服务只能通过Service来暴露，比如Service的暴露方式推荐选择internal的loadbalance，而通过k8s创建的internal的loadbalance会自动创建一个security group

可以在这个security group中进行限制，loadbalance默认的Inbound的source是0.0.0.0/0等于允许所有的连接

比如A访问B的GRPC 6379，则只修改B的loadbalance的安全组，允许来自A的网段TCP连接自己的6379端口。

可以先在双方以**nodes**开头的安全组中增加一个条目，⚠️所有的安全组都必须标注清楚description，对方可以访问LB之后，需要删除该条测试安全组。

| 字段       | 值                           | 描述       |
| ---------- | ---------------------------- | ---------- |
| Type       | All ICMP - IPv4              |            |
| Protocol   | ICMP (1)                     | 默认会出来 |
| Port Range | ALL                          | 默认会出来 |
| Source     | A账号就填B的网段；反之B填A； |            |

填完之后，互相登陆**nodes**一台机器，互相ping一下，确认一下是否可以ping通，如果可以ping通，就说明vpc-peering打通了。

### Interface VPC Endpoints (AWS PrivateLink)

An interface VPC endpoint (interface endpoint) enables you to connect to services powered by AWS PrivateLink. These services include some AWS services, services hosted by other AWS customers and Partners in their own VPCs (referred to as *endpoint services*), and supported AWS Marketplace Partner services. The owner of the service is the *service provider*, and you, as the principal creating the interface endpoint, are the *service consumer*.

大概的步骤如下：

1. 创建NLB；
2. 创建vpc endpoint service；
3. 在vpc endpoint service中添加白名单；
4. 在其他的账号中创建vpc endpoint；注意安全组，可以vpc endpoint理解成一个service的入口；
5. 在vpc endpoint service中接受这个连接；
6. 过一会连接可用之后，直接vpc endpoint所在的vpc都可以访问到；



Q:ip冲突怎么解决？
The design takes into consideration that provider and consumer accounts could be using the same VPC CIDR and therefore that is not an issue

Q:别的account搜索不到endpoint service
连接之前需要先加白名单才能连接

https://docs.aws.amazon.com/vpc/latest/userguide/endpoint-service.html#vpce-endpoint-service-availability-zones

Q:don't understand "To create an interface endpoint to an AWS service using the console"
Remember that all of our services are accessing via public endpoints, but you can optionally create an interface endpoint if you don't wish to access that same service via the internet. That is the use case for interface endpoint with AWS services
eg, the service of aws ec2 is refer to the ec2 aws api.

Q:if you want to connect other account's aws service, eg rds, you must create nlb/endpoint service, right? now we using vpc peering, but I think we can use nlb/endpoint service too.
No, Endpoint Services are only for services that you host as the customer and want other AWS accounts/consumers to connect to it
The nlb target type only instance/ip/lambda function, don't have other, so can't using endpoint service.
Correct, the existing configuration doesn't allow creating an Endpoint service for an AWS service

Q:private dns

必须own需要在public的dns里面进行verify