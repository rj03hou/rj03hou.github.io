
---
layout: single
title: aws ec2 bandwidth benchmark(network performance)
description: aws ec2 bandwidth benchmark(network performance)
headline:
categories: aws
headline:
tags: [aws]
comments: true
published: true
---


# 一 结论

1. vpc内部和vpc-peering对于aws没有区别；aws内部只用sdn做了一个隔离而已；
2. 不同的instance type，network bandwidth差距很大，特别是是否支持ENA之后（结论三，t3.xlarge支持ENA网络增强）；
3. t3.xlarge TCP，波动性比较强，从5Gbits/sec - 2.17 Gbits/sec之间波动；
   t2.xlarge TCP，从最初1.15Gbits/sec持续下降，稳定在0.74Gbits/sec；
   t2.micro，TCP，从最初的1.07Gbits/sec，稳定在0.064Gbits/sec；

# 一 测试方法

iperf，主要是验证cloudonaut上的结论是否可以作为参照

```bash
yum install iperf3.x86_64 -y
#server端
iperf3 -s -p 8080
#client端，-i 每隔一定的时间输出一下结果，-P多少个线程，-u udp的意思，默认是tcp，-p server端口，-t测试时间
iperf3 -c SERVER_IP -u -P 10 -i 5 -t 300 -V -p 8080
```



# 二 详情

## 1. 测试同一个VPC内部的网络，选择两种以上的机型

```bash
#t2.micro，TCP，从最初的1.07Gbits/sec，稳定在0.064Gbits/sec
Starting Test: protocol: TCP, 10 streams, 131072 byte blocks, omitting 0 seconds, 300 second test
[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
[  4]   0.00-5.00   sec   144 MBytes   241 Mbits/sec   45    210 KBytes
[  6]   0.00-5.00   sec   160 MBytes   268 Mbits/sec   42    393 KBytes
[  8]   0.00-5.00   sec  37.1 MBytes  62.2 Mbits/sec  125   69.9 KBytes
[ 10]   0.00-5.00   sec  37.2 MBytes  62.4 Mbits/sec  113   78.7 KBytes
[ 12]   0.00-5.00   sec  34.4 MBytes  57.8 Mbits/sec  113   87.4 KBytes
[ 14]   0.00-5.00   sec  36.8 MBytes  61.8 Mbits/sec  118   61.2 KBytes
[ 16]   0.00-5.00   sec  42.3 MBytes  71.0 Mbits/sec  119   87.4 KBytes
[ 18]   0.00-5.00   sec  41.1 MBytes  69.0 Mbits/sec  107   87.4 KBytes
[ 20]   0.00-5.00   sec  39.6 MBytes  66.5 Mbits/sec  118   52.4 KBytes
[ 22]   0.00-5.00   sec  37.7 MBytes  63.3 Mbits/sec  123   69.9 KBytes
[SUM]   0.00-5.00   sec   610 MBytes  1.02 Gbits/sec  1023
- - - - - - - - - - - - - - - - - - - - - - - - -
[  4] 295.00-300.00 sec  6.57 MBytes  11.0 Mbits/sec    1    245 KBytes
[  6] 295.00-300.00 sec  11.8 MBytes  19.8 Mbits/sec    2    297 KBytes
[  8] 295.00-300.00 sec  2.45 MBytes  4.11 Mbits/sec    9   78.7 KBytes
[ 10] 295.00-300.00 sec  2.45 MBytes  4.11 Mbits/sec    8   69.9 KBytes
[ 12] 295.00-300.00 sec  2.39 MBytes  4.01 Mbits/sec   11   43.7 KBytes
[ 14] 295.00-300.00 sec  2.93 MBytes  4.91 Mbits/sec    6    105 KBytes
[ 16] 295.00-300.00 sec  2.51 MBytes  4.21 Mbits/sec    6   96.1 KBytes
[ 18] 295.00-300.00 sec  2.39 MBytes  4.01 Mbits/sec    7   96.1 KBytes
[ 20] 295.00-300.00 sec  2.27 MBytes  3.81 Mbits/sec    8   61.2 KBytes
[ 22] 295.00-300.00 sec  1.67 MBytes  2.81 Mbits/sec    8   43.7 KBytes
[SUM] 295.00-300.00 sec  37.5 MBytes  62.8 Mbits/sec   66
- - - - - - - - - - - - - - - - - - - - - - - - -
Test Complete. Summary Results:
[ ID] Interval           Transfer     Bandwidth       Retr
[  4]   0.00-300.00 sec  4.30 GBytes   123 Mbits/sec  1367             sender
[  4]   0.00-300.00 sec  4.30 GBytes   123 Mbits/sec                  receiver
[  6]   0.00-300.00 sec  4.30 GBytes   123 Mbits/sec  1322             sender
[  6]   0.00-300.00 sec  4.29 GBytes   123 Mbits/sec                  receiver
[  8]   0.00-300.00 sec  1.06 GBytes  30.3 Mbits/sec  2661             sender
[  8]   0.00-300.00 sec  1.06 GBytes  30.3 Mbits/sec                  receiver
[ 10]   0.00-300.00 sec  1.05 GBytes  30.2 Mbits/sec  2688             sender
[ 10]   0.00-300.00 sec  1.05 GBytes  30.1 Mbits/sec                  receiver
[ 12]   0.00-300.00 sec  1.06 GBytes  30.2 Mbits/sec  2659             sender
[ 12]   0.00-300.00 sec  1.05 GBytes  30.2 Mbits/sec                  receiver
[ 14]   0.00-300.00 sec  1.11 GBytes  31.7 Mbits/sec  2671             sender
[ 14]   0.00-300.00 sec  1.11 GBytes  31.7 Mbits/sec                  receiver
[ 16]   0.00-300.00 sec  1.06 GBytes  30.5 Mbits/sec  2711             sender
[ 16]   0.00-300.00 sec  1.06 GBytes  30.5 Mbits/sec                  receiver
[ 18]   0.00-300.00 sec  1.08 GBytes  31.0 Mbits/sec  2687             sender
[ 18]   0.00-300.00 sec  1.08 GBytes  31.0 Mbits/sec                  receiver
[ 20]   0.00-300.00 sec  1.06 GBytes  30.5 Mbits/sec  2723             sender
[ 20]   0.00-300.00 sec  1.06 GBytes  30.5 Mbits/sec                  receiver
[ 22]   0.00-300.00 sec  1.11 GBytes  31.9 Mbits/sec  2666             sender
[ 22]   0.00-300.00 sec  1.11 GBytes  31.9 Mbits/sec                  receiver
[SUM]   0.00-300.00 sec  17.2 GBytes   492 Mbits/sec  24155             sender
[SUM]   0.00-300.00 sec  17.2 GBytes   492 Mbits/sec                  receiver
CPU Utilization: local/sender 2.1% (0.1%u/2.0%s), remote/receiver 9.2% (0.9%u/8.3%s)

```



```bash
#t2.xlarge tcp，从1.15G持续下降，稳定在0.74
[  4] 295.00-300.00 sec  48.8 MBytes  81.8 Mbits/sec   38    280 KBytes
[  6] 295.00-300.00 sec  56.2 MBytes  94.4 Mbits/sec   68    192 KBytes
[  8] 295.00-300.00 sec  48.8 MBytes  81.8 Mbits/sec   61    280 KBytes
[ 10] 295.00-300.00 sec  50.0 MBytes  83.9 Mbits/sec   43    175 KBytes
[ 12] 295.00-300.00 sec  43.8 MBytes  73.4 Mbits/sec   39    131 KBytes
[ 14] 295.00-300.00 sec  36.2 MBytes  60.8 Mbits/sec   49   87.4 KBytes
[ 16] 295.00-300.00 sec  40.0 MBytes  67.1 Mbits/sec   46    175 KBytes
[ 18] 295.00-300.00 sec  36.2 MBytes  60.8 Mbits/sec   46    140 KBytes
[ 20] 295.00-300.00 sec  33.8 MBytes  56.6 Mbits/sec   46    149 KBytes
[ 22] 295.00-300.00 sec  50.0 MBytes  83.9 Mbits/sec   43    105 KBytes
[SUM] 295.00-300.00 sec   444 MBytes   744 Mbits/sec  479
- - - - - - - - - - - - - - - - - - - - - - - - -
Test Complete. Summary Results:
[ ID] Interval           Transfer     Bandwidth       Retr
[  4]   0.00-300.00 sec  3.29 GBytes  94.3 Mbits/sec  4293             sender
[  4]   0.00-300.00 sec  3.28 GBytes  94.0 Mbits/sec                  receiver
[  6]   0.00-300.00 sec  3.30 GBytes  94.5 Mbits/sec  4206             sender
[  6]   0.00-300.00 sec  3.29 GBytes  94.2 Mbits/sec                  receiver
[  8]   0.00-300.00 sec  3.30 GBytes  94.4 Mbits/sec  4136             sender
[  8]   0.00-300.00 sec  3.29 GBytes  94.2 Mbits/sec                  receiver
[ 10]   0.00-300.00 sec  3.29 GBytes  94.1 Mbits/sec  4285             sender
[ 10]   0.00-300.00 sec  3.28 GBytes  93.9 Mbits/sec                  receiver
[ 12]   0.00-300.00 sec  3.31 GBytes  94.8 Mbits/sec  4333             sender
[ 12]   0.00-300.00 sec  3.30 GBytes  94.6 Mbits/sec                  receiver
[ 14]   0.00-300.00 sec  2.67 GBytes  76.4 Mbits/sec  4068             sender
[ 14]   0.00-300.00 sec  2.66 GBytes  76.2 Mbits/sec                  receiver
[ 16]   0.00-300.00 sec  2.67 GBytes  76.5 Mbits/sec  3998             sender
[ 16]   0.00-300.00 sec  2.66 GBytes  76.2 Mbits/sec                  receiver
[ 18]   0.00-300.00 sec  2.54 GBytes  72.7 Mbits/sec  4121             sender
[ 18]   0.00-300.00 sec  2.53 GBytes  72.5 Mbits/sec                  receiver
[ 20]   0.00-300.00 sec  2.52 GBytes  72.1 Mbits/sec  4287             sender
[ 20]   0.00-300.00 sec  2.51 GBytes  71.8 Mbits/sec                  receiver
[ 22]   0.00-300.00 sec  3.33 GBytes  95.2 Mbits/sec  4309             sender
[ 22]   0.00-300.00 sec  3.32 GBytes  95.0 Mbits/sec                  receiver
[SUM]   0.00-300.00 sec  30.2 GBytes   865 Mbits/sec  42036             sender
[SUM]   0.00-300.00 sec  30.1 GBytes   863 Mbits/sec                  receiver
CPU Utilization: local/sender 3.1% (0.0%u/3.0%s), remote/receiver 17.0% (1.2%u/15.8%s)

```



```bash
#t3.xlarge TCP，测试时间10分钟，波动性比较强，观察最低到2.17 Gbits/sec
Starting Test: protocol: TCP, 10 streams, 131072 byte blocks, omitting 0 seconds, 300 second test
[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
[  4]   0.00-5.00   sec   313 MBytes   525 Mbits/sec  140    262 KBytes
[  6]   0.00-5.00   sec   329 MBytes   552 Mbits/sec  149    385 KBytes
[  8]   0.00-5.00   sec   287 MBytes   482 Mbits/sec  183    271 KBytes
[ 10]   0.00-5.00   sec   184 MBytes   308 Mbits/sec  130    245 KBytes
[ 12]   0.00-5.00   sec   180 MBytes   302 Mbits/sec  113    367 KBytes
[ 14]   0.00-5.00   sec   185 MBytes   310 Mbits/sec   69    385 KBytes
[ 16]   0.00-5.00   sec   191 MBytes   319 Mbits/sec  106    481 KBytes
[ 18]   0.00-5.00   sec   432 MBytes   724 Mbits/sec  222    306 KBytes
[ 20]   0.00-5.00   sec   198 MBytes   331 Mbits/sec   60    306 KBytes
[ 22]   0.00-5.00   sec   420 MBytes   704 Mbits/sec  179    376 KBytes
[SUM]   0.00-5.00   sec  2.66 GBytes  4.56 Gbits/sec  1351
- - - - - - - - - - - - - - - - - - - - - - - - -
Test Complete. Summary Results:
[ ID] Interval           Transfer     Bandwidth       Retr
[  4]   0.00-600.00 sec  25.7 GBytes   367 Mbits/sec  11576             sender
[  4]   0.00-600.00 sec  25.7 GBytes   367 Mbits/sec                  receiver
[  6]   0.00-600.00 sec  25.1 GBytes   359 Mbits/sec  11041             sender
[  6]   0.00-600.00 sec  25.1 GBytes   359 Mbits/sec                  receiver
[  8]   0.00-600.00 sec  24.7 GBytes   354 Mbits/sec  10864             sender
[  8]   0.00-600.00 sec  24.7 GBytes   354 Mbits/sec                  receiver
[ 10]   0.00-600.00 sec  37.1 GBytes   530 Mbits/sec  18291             sender
[ 10]   0.00-600.00 sec  37.0 GBytes   530 Mbits/sec                  receiver
[ 12]   0.00-600.00 sec  36.4 GBytes   521 Mbits/sec  17987             sender
[ 12]   0.00-600.00 sec  36.4 GBytes   521 Mbits/sec                  receiver
[ 14]   0.00-600.00 sec  23.4 GBytes   335 Mbits/sec  12663             sender
[ 14]   0.00-600.00 sec  23.4 GBytes   335 Mbits/sec                  receiver
[ 16]   0.00-600.00 sec  38.6 GBytes   553 Mbits/sec  13080             sender
[ 16]   0.00-600.00 sec  38.6 GBytes   553 Mbits/sec                  receiver
[ 18]   0.00-600.00 sec  23.0 GBytes   330 Mbits/sec  12549             sender
[ 18]   0.00-600.00 sec  23.0 GBytes   330 Mbits/sec                  receiver
[ 20]   0.00-600.00 sec  36.2 GBytes   518 Mbits/sec  16072             sender
[ 20]   0.00-600.00 sec  36.2 GBytes   518 Mbits/sec                  receiver
[ 22]   0.00-600.00 sec  35.3 GBytes   506 Mbits/sec  15482             sender
[ 22]   0.00-600.00 sec  35.3 GBytes   506 Mbits/sec                  receiver
[SUM]   0.00-600.00 sec   305 GBytes  4.37 Gbits/sec  139605             sender
[SUM]   0.00-600.00 sec   305 GBytes  4.37 Gbits/sec                  receiver
CPU Utilization: local/sender 15.5% (0.2%u/15.3%s), remote/receiver 56.5% (3.1%u/53.5%s)
```

**Enabling Enhanced Networking with the Elastic Network Adapter (ENA) on Linux Instances**

CentOS Linux 7 x86_64 HVM EBS ENA 1805_01-b7ee8a69-ee97-4a49-9e68-afaee216db2e-ami-77ec9308.4 (ami-8e8847f1)，默认ENA就是打开的，因此可以达到5Gbits/sec，查看官方文档

Select EC2 instances support cluster networking when launched into a common cluster placement group. A cluster placement group provides low-latency networking between all instances in the cluster. **The bandwidth an EC2 instance can utilize depends on the instance type and its networking performance specification. Inter instance traffic within the same region can utilize up to 5 Gbps for single-flow and up to 25 Gbps for multi-flow traffic in each direction (full duplex).** Traffic to and from S3 buckets in the same region can also utilize all available instance aggregate bandwidth. When launched in a placement group, instances can utilize up to 10 Gbps for single-flow traffic and up to 25 Gbps for multi-flow traffic. Network traffic to the Internet is limited to 5 Gbps (full duplex). Cluster networking is ideal for high performance analytics systems and many science and engineering applications, especially those using the MPI library standard for parallel programming.

```bash
#t3.xlarge tcp enable ENA
# ethtool -i ens5
driver: ena
version: 1.2.0k
firmware-version:
expansion-rom-version:
bus-info: 0000:00:05.0
supports-statistics: yes
supports-test: no
supports-eeprom-access: no
supports-register-dump: no
supports-priv-flags: no
```



## 2. 测试跨vpc的网络

```bash
#ap-northeast-1a下面的两个不同的vpc
[  4] 295.00-300.00 sec  31.2 MBytes  52.4 Mbits/sec   77   78.7 KBytes
[  6] 295.00-300.00 sec  31.2 MBytes  52.4 Mbits/sec   67   69.9 KBytes
[  8] 295.00-300.00 sec  32.5 MBytes  54.5 Mbits/sec   58    105 KBytes
[ 10] 295.00-300.00 sec  35.0 MBytes  58.7 Mbits/sec   66    131 KBytes
[ 12] 295.00-300.00 sec  30.0 MBytes  50.3 Mbits/sec   72   52.4 KBytes
[ 14] 295.00-300.00 sec  63.8 MBytes   107 Mbits/sec   62    114 KBytes
[ 16] 295.00-300.00 sec  31.2 MBytes  52.4 Mbits/sec   63    105 KBytes
[ 18] 295.00-300.00 sec  35.0 MBytes  58.7 Mbits/sec   49    140 KBytes
[ 20] 295.00-300.00 sec  72.5 MBytes   122 Mbits/sec   69    122 KBytes
[ 22] 295.00-300.00 sec  78.8 MBytes   132 Mbits/sec   69    122 KBytes
[SUM] 295.00-300.00 sec   441 MBytes   740 Mbits/sec  652
- - - - - - - - - - - - - - - - - - - - - - - - -
Test Complete. Summary Results:
[ ID] Interval           Transfer     Bandwidth       Retr
[  4]   0.00-300.00 sec  2.47 GBytes  70.6 Mbits/sec  5958             sender
[  4]   0.00-300.00 sec  2.46 GBytes  70.4 Mbits/sec                  receiver
[  6]   0.00-300.00 sec  2.27 GBytes  65.0 Mbits/sec  6435             sender
[  6]   0.00-300.00 sec  2.26 GBytes  64.8 Mbits/sec                  receiver
[  8]   0.00-300.00 sec  2.25 GBytes  64.4 Mbits/sec  6410             sender
[  8]   0.00-300.00 sec  2.24 GBytes  64.1 Mbits/sec                  receiver
[ 10]   0.00-300.00 sec  2.44 GBytes  69.8 Mbits/sec  6126             sender
[ 10]   0.00-300.00 sec  2.43 GBytes  69.5 Mbits/sec                  receiver
[ 12]   0.00-300.00 sec  2.35 GBytes  67.3 Mbits/sec  6211             sender
[ 12]   0.00-300.00 sec  2.34 GBytes  67.0 Mbits/sec                  receiver
[ 14]   0.00-300.00 sec  4.34 GBytes   124 Mbits/sec  7982             sender
[ 14]   0.00-300.00 sec  4.33 GBytes   124 Mbits/sec                  receiver
[ 16]   0.00-300.00 sec  2.26 GBytes  64.7 Mbits/sec  6522             sender
[ 16]   0.00-300.00 sec  2.25 GBytes  64.4 Mbits/sec                  receiver
[ 18]   0.00-300.00 sec  2.46 GBytes  70.6 Mbits/sec  5985             sender
[ 18]   0.00-300.00 sec  2.46 GBytes  70.3 Mbits/sec                  receiver
[ 20]   0.00-300.00 sec  4.72 GBytes   135 Mbits/sec  7383             sender
[ 20]   0.00-300.00 sec  4.71 GBytes   135 Mbits/sec                  receiver
[ 22]   0.00-300.00 sec  4.65 GBytes   133 Mbits/sec  7528             sender
[ 22]   0.00-300.00 sec  4.65 GBytes   133 Mbits/sec                  receiver
[SUM]   0.00-300.00 sec  30.2 GBytes   865 Mbits/sec  66540             sender
[SUM]   0.00-300.00 sec  30.1 GBytes   863 Mbits/sec                  receiver
CPU Utilization: local/sender 3.5% (0.1%u/3.5%s), remote/receiver 12.7% (1.0%u/11.8%s)


#两个不同的vpc，使用vpc-peering连接起来，并且跨az，ap-northeast-1a到ap-northeast-1c，数据和同一个vpc内是相同的
- - - - - - - - - - - - - - - - - - - - - - - - -
[  4] 295.00-300.00 sec  40.0 MBytes  67.1 Mbits/sec  122   87.4 KBytes
[  6] 295.00-300.00 sec  41.2 MBytes  69.2 Mbits/sec  134   87.4 KBytes
[  8] 295.00-300.00 sec  50.0 MBytes  83.9 Mbits/sec  101   52.4 KBytes
[ 10] 295.00-300.00 sec  47.5 MBytes  79.7 Mbits/sec  107   61.2 KBytes
[ 12] 295.00-300.00 sec  45.0 MBytes  75.5 Mbits/sec  109   78.7 KBytes
[ 14] 295.00-300.00 sec  38.8 MBytes  65.0 Mbits/sec  135   87.4 KBytes
[ 16] 295.00-300.00 sec  47.5 MBytes  79.7 Mbits/sec  116   52.4 KBytes
[ 18] 295.00-300.00 sec  47.5 MBytes  79.7 Mbits/sec  104    122 KBytes
[ 20] 295.00-300.00 sec  43.8 MBytes  73.4 Mbits/sec  110   78.7 KBytes
[ 22] 295.00-300.00 sec  41.2 MBytes  69.2 Mbits/sec  122   69.9 KBytes
[SUM] 295.00-300.00 sec   442 MBytes   742 Mbits/sec  1160
- - - - - - - - - - - - - - - - - - - - - - - - -
Test Complete. Summary Results:
[ ID] Interval           Transfer     Bandwidth       Retr
[  4]   0.00-300.00 sec  3.43 GBytes  98.1 Mbits/sec  6560             sender
[  4]   0.00-300.00 sec  3.42 GBytes  97.9 Mbits/sec                  receiver
[  6]   0.00-300.00 sec  2.78 GBytes  79.7 Mbits/sec  5757             sender
[  6]   0.00-300.00 sec  2.77 GBytes  79.4 Mbits/sec                  receiver
[  8]   0.00-300.00 sec  3.07 GBytes  87.9 Mbits/sec  5315             sender
[  8]   0.00-300.00 sec  3.06 GBytes  87.7 Mbits/sec                  receiver
[ 10]   0.00-300.00 sec  3.31 GBytes  94.7 Mbits/sec  5934             sender
[ 10]   0.00-300.00 sec  3.30 GBytes  94.4 Mbits/sec                  receiver
[ 12]   0.00-300.00 sec  2.96 GBytes  84.7 Mbits/sec  6740             sender
[ 12]   0.00-300.00 sec  2.95 GBytes  84.4 Mbits/sec                  receiver
[ 14]   0.00-300.00 sec  2.98 GBytes  85.4 Mbits/sec  6616             sender
[ 14]   0.00-300.00 sec  2.98 GBytes  85.2 Mbits/sec                  receiver
[ 16]   0.00-300.00 sec  2.91 GBytes  83.4 Mbits/sec  5395             sender
[ 16]   0.00-300.00 sec  2.91 GBytes  83.2 Mbits/sec                  receiver
[ 18]   0.00-300.00 sec  2.98 GBytes  85.5 Mbits/sec  5370             sender
[ 18]   0.00-300.00 sec  2.98 GBytes  85.2 Mbits/sec                  receiver
[ 20]   0.00-300.00 sec  3.10 GBytes  88.8 Mbits/sec  6310             sender
[ 20]   0.00-300.00 sec  3.09 GBytes  88.6 Mbits/sec                  receiver
[ 22]   0.00-300.00 sec  2.81 GBytes  80.5 Mbits/sec  5799             sender
[ 22]   0.00-300.00 sec  2.80 GBytes  80.2 Mbits/sec                  receiver
[SUM]   0.00-300.00 sec  30.3 GBytes   869 Mbits/sec  59796             sender
[SUM]   0.00-300.00 sec  30.3 GBytes   866 Mbits/sec                  receiver
CPU Utilization: local/sender 3.2% (0.0%u/3.2%s), remote/receiver 22.0% (2.0%u/20.1%s)
```



## 3. 机型对比

4C 16G，推荐使用

|           |                  | EBS OPT | ENA  | Network performance |
| --------- | ---------------- | ------- | ---- | ------------------- |
| t3.xlarge | $0.2176 per Hour | Yes     | Yes  | Moderate            |
| t2.xlarge | $0.2432 per Hour | *       | *    | Moderate            |
| m5.xlarge | $0.248 per Hour  | Yes     | Yes  | High                |

## 4. AQ

1. is the bandwidth limit is dynamic because the ec2 instance is share in the physical matchine? does aws have some thing like cgroup to make sure that one ec2 can't use all the network.

   It is not dynamic. It is static and stable, this is the main reason why we limit networking throughput according to the instance type - to make sure that each instance can go up to the max throughput for that instance type regardless the other tenants (instances). The underlying hardware is able to manage the traffic from all the guests, even when they are using their full capacities.

2. How to check whether the network bandwidth is botterneck?
   **Correct. For example, if you only see packet loss/retransmission to a certain instance inside the VPC. That packet loss/retransmission is not seen in any other instances within the same VPC, so it might be some issue on the instance. If it is not OS/application issues, the NIC might be overloaded.**

   it depends on how you VPC is organized. Do you have an IGW, NAT Gateway or Nat Instance to go to the Internet. IGW and NAT gateway will scale out automatically, so you might not face any bottleneck. In case you are using a NAT instance, you can run "netstat" for example and see the amount of connection as well as "free -m" to see if you running out of memory.

   The same way, you can run traceroute or ping to see packet loss or high latency and where that is coming from.

   Finally, VPC flow logs can show you the amount of flows at a certain time-frame.

   Once, again it depends on how you build your VPC - the central point.

   if the latency now is very high, how can I make sure is the bandwidth problem?

   You will need to use the same approach for traditional networking, by running "netstat", checking the number of active connections, number of drops and retransmission (ifconfig).

   You can use some other tools such as "iftop" that will show all the in-flight connections.

   Correct, especially if you are seeing packet loss/retransmission.

3. **what is the key metric to alert?**
   丢包率irate(node_netstat_Tcp_RetransSegs[5m])/(irate(node_netstat_Tcp_OutSegs[5m])+irate(node_netstat_Tcp_InSegs[5m]))*100

   bandwidth：networkin+networkout

4. why the bandwidth is down from time.
   T2 instance generally use a shared environment and tend to get throttled quite often.... so if you looking for gaurenteed network performance, I would recommend our enhanced networking instances

5. if we enable t3.xlarge and m5.xlarge can we have dedicate network bandwidth?
   Any performance-sensitive workloads that require minimal variability and dedicated Amazon EC2 to Amazon EBS traffic, such as production databases or business applications, should use volumes that are attached to an EBS-optimized instance or an instance with 10 Gigabit network connectivity. EC2 instances that do not meet this criteria offer no guarantee of network resources. The only way to ensure sustained reliable network bandwidth between your EC2 instance and your EBS volumes is to launch the EC2 instance as EBS-optimized or choose an instance type with 10 Gigabit network connectivity. To see which instance types include 10 Gigabit network connectivity, see Amazon EC2 Instance Types. For information about configuring EBS-optimized instances, see Amazon EBS–Optimized Instances.

6. **the bandwidth is according to the ec2 instance, the vpc-peering not be bottleneck?**
   Correct. VPC Peerings works in the same way (bandwidth) than AWS internal networking. Transferring data from 2 EC2s within the same VPC, or between VPCs via VPC peering should have the same transfer rate.

7. the loadbalance before the ec2 instance, the loadbalance's bandwidth has limit?

   The Load Balancer's Networking DOES have a limit. However, ELB/ALB will scale out before the limit is reached.

   I can not give you the limit, because it depends on the node is in use. For example, when we pre-warm the ELB, we will have more powerful ELBs (with more powerful NICs).

8. when I enable ENA, the baseline is not change, but the limit is change, is that right?
   the throughput can go up to 25 Gbps. This is not the burst... If you have an application that can work steadily with 25Gbps, it will work fine.

9. why udp only to 10Mbits/sec and tcp to 60Mbits/sec

   from an AWS perspective there is no difference. We don't do any shaping or QoS regarding protocol. I have not run the same test on my side, but I would say that TCP can scale (Window Scaling) easier than UDP. Besides that, internally, MTU is 9001 so that TCP can use it more efficiently.



## 参照

[Azure 中使用 iPerf 进行网络带宽测试](https://docs.azure.cn/zh-cn/articles/azure-operations-guide/virtual-network/aog-virtual-network-iperf-bandwidth-test)

[The Floodgates Are Open – Increased Network Bandwidth for EC2 Instances](https://aws.amazon.com/blogs/aws/the-floodgates-are-open-increased-network-bandwidth-for-ec2-instances/)

[EC2 Network Performance Cheat Sheet](https://cloudonaut.io/ec2-network-performance-cheat-sheet/)
