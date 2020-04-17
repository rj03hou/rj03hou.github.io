---
layout: single
title: aws guardduty简介
description: aws guardduty简介
headline: aws guardduty简介
categories: aws security
headline: 
tags: [aws, security]
comments: true
published: true
---



#### 介绍

![product-page-diagram-Amazon-GuardDuty_how-it-works](https://d1.awsstatic.com/Products/product-name/diagrams/product-page-diagram-Amazon-GuardDuty_how-it-works.4370200b49eddc34d3a55c52c584484ceb2d532b.png)

GuardDuty 对来自多个 AWS 数据源（ AWS CloudTrail、Amazon VPC Flow Logs 和 DNS 日志）的数百亿事件进行分析，来进行威胁检测，监控恶意活动和未经授权的行为，是一种事后的检测发现机制。集成了威胁情报、异常检测和机器学习功能。

#### FAQ

**问：要使 Amazon GuardDuty 正常运行，必须启用 AWS CloudTrail、VPC 流日志和 DNS 日志吗？**

不需要，Amazon GuardDuty 可以直接从 AWS CloudTrail、VPC 流日志和 AWS DNS 日志中提取独立的数据流。您不需要管理 Amazon S3 存储桶策略，也不需要修改收集和存储日志的方式。GuardDuty 的权限作为服务相关角色进行管理，因此您可以随时通过禁用 GuardDuty 进行撤消。这样，您无需复杂的配置即可轻松启用该服务，还可以避免 AWS IAM 权限修改或 S3 存储桶策略更改对该服务的运行产生影响。而且，GuardDuty 还得以非常高效并且近乎实时地使用大量数据，不影响您的账户或工作负载的性能或可用性。

**问：Amazon GuardDuty 会管理或保存我的日志吗？**

不会，Amazon GuardDuty 不会管理或保存您的日志。GuardDuty 使用的所有数据都会被近乎实时地分析，然后丢弃。这让 GuardDuty 不仅经济高效，还能降低数据残留的风险。如需传输和保存日志，您应该直接使用 AWS 的日志记录和监控服务，这些服务可以提供功能全面的传输和保存选项。

**问：Amazon GuardDuty 可以检测哪些风险？**

Amazon GuardDuty 内置的检测技术是专门针对云服务开发和优化的技术。其检测算法由 AWS 安全团队进行维护和不断改进。主要检测类别包括：

- **侦察** — 表明攻击者在进行侦察的活动，例如 API 活动异常、VPC 内部端口扫描、登录请求失败模式异常或未被阻止的来自已知不良 IP 地址的端口探测。
- **实例盗用** — 表明存在实例盗用的活动，例如加密货币挖矿、恶意使用域名生成算法 (DGA)、出站拒绝服务活动、网络流量异常高、网络协议异常、与已知恶意 IP 进行出站实例通信、外部 IP 地址使用 Amazon EC2 临时凭证以及使用 DNS 造成数据外泄。
- **账户盗用** — 表明存在账户盗用的常见形式包括：来自异常位置或匿名代理的 API 调用、试图禁用 AWS CloudTrail 日志记录功能、实例或基础设施异常启动、在异常区域部署基础设施以及来自已知恶意 IP 地址的 API 调用。

#### 风险类型

发现的威胁很详细，详情可以看[aws文档](https://docs.aws.amazon.com/zh_cn/guardduty/latest/ug/guardduty_finding-types-active.html)

1. UnauthorizedAccess:EC2/SSHBruteForce，可以看到哪个ip在哪些时间对机器进行ssh暴力攻击，可能的原因是安全组设置不合理，或者将不该包含公网ip的机器包含了公网ip；
1. UnauthorizedAccess:IAMUser/ConsoleLogin，aws账号被异常登录，比如从以前从未用过的客户端或异常位置调用了 ConsoleLogin API；
1. Behavior:EC2/NetworkPortUnusual，EC2 实例在异常服务器端口上与远程主机通信；
1. CryptoCurrency:EC2/BitcoinTool.B!DNS和CryptoCurrency:EC2/BitcoinTool.B，表示服务器可能存在挖矿行为；