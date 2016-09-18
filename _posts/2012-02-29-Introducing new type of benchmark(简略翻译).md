---
layout: post 
title: "Introducing new type of benchmark(简略翻译)"
subTitle: 
heroImageUrl: 
date: 2012-2-29
tags: ["MySQL"]
keywords: 
---

<div>

原文地址：[http://www.mysqlperformanceblog.com/2012/02/25/introducing-new-type-of-benchmark/](http://www.mysqlperformanceblog.com/2012/02/25/introducing-new-type-of-benchmark/)

一篇写的很好的文章，让测试更加接近现实。

Traditionally the most benchmarks are focusing on throughput. We all get used to that, and in fact in our benchmarks, sysbench and tpcc-mysql, the final result is also represents the throughput (transactions per second in sysbench; NewOrder transactions Per Minute in tpcc-mysql). However, like Mark Callaghan mentioned [in comments](http://www.mysqlperformanceblog.com/2012/02/23/percona-server-vs-mysql-on-intel-320-ssd/comment-page-1/#comment-891509), response time is way more important metric to compare.

传统的benchmark都主要考察的是TPS，忽略了另外一个重要的纬度：响应时间。
<div>I want to pretend that we pioneered (not invented, but started to use widely) a benchmark methodology when we measure not the final throughput, but rather periodic probes (i.e. every 10 sec).
It allows us to draw "stability" graphs, like this one</div>
<div>![alt](Image.png)</div>
<div>where we can see not only a final result, but how the system behaves in dynamic.</div>
<div>我提倡在测试时，我们不应该只记录TPS，同时应该间隔一段时间记录响应时间，这样可以绘制出一幅图，通过它可以观察稳定性。</div>
**What's wrong with existing benchmarks?**
<div>Well, all benchmarks are lie, and focusing on throughput does not get any closer to reality.</div>
<div>现有的benchmark都存在问题，他们只关注TPS，跟实际有差距。</div>
Benchmarks, like sysbench or tpcc-mysql, start N threads and try to push the database as much as possible, bombarding the system with queries with no pause.
<div>That rarely happens in real life. There are no systems that are pushed to 100% load all time.</div>
<div>benchmark软件比如说sysbench、tpcc-mysql，都是开启N个线程，然后"不停"的向数据库发送请求，在现实中很少见，很有有系统会不停的发送请求。</div>
<div>So, **how we can model it?** There are different theories, and the one which describes user's behavior, is [Queueing theory](http://en.wikipedia.org/wiki/Queueing_theory). In short we can assume that users send requests with some **arrival rate** (which can be different in the different part of day/week/month/year though). And what is important for an end user is **response time**, that is how long the user has to wait on results. E.g. when you go to your Facebook profile or Wikipedia page, you expect to get response within second or two.</div>
<div>我们应该如何建模？有很多的理论，其中一种模仿用户的行为，队列理论。用户发送请求都是存在一定的"arrival rate"（这个在不同的时间段是不一样）。对与用户最重要的，就是请求的响应时间。</div>
<div>How we should change the benchmark to base on this model ?</div>
<div>按照这种模型，我们应该如何改变我们的benchmark？</div>
<div>
There are my working ideas:</div>

*   Benchmark starts N working threads, but they all are idle until asked to handle a request
*   For example, if our target is arrival rate 2 queries per second, then exponential distribution will give us following intervals (in sec) between events: 0.61, 0.43, 1.55, 0.18, 0.01, 0.76, 0.09, 1.26, ...&nbsp;

Or if we represent graphically (point means even arrival):
[![alt](Image(1).png)](http://www.mysqlperformanceblog.com/wp-content/uploads/2012/02/arrival.png)
As you see interval is far from being strict 0.5 sec, but 0.5 is the mean of this random generation function. On the graph you see 20 events arrived within 9 seconds.
*   Transactions from the queue are handled by one of free threads, or are waiting in the queue until one of threads are ready. The time waiting in the queue is added to a total response time.
*   As a result we measure 95% or 99% response times.
<div>我的一些想法就是：启动起来N个线程，按照泊松分布将请求发送到队列中，N个线程开始处理队列中的请求，当请求不能被处理时，就等待，最后的response time从进入队列中开始计算。</div>
<div>What does it give to us? It allows to see:</div>

*   What is the response time we may expect having a given arrival rate
*   What is the optimal number of working threads (the one that provides best response times)
<div>按照这样的做法，我们可以通过测试得出：</div>
<div>1.按照某个arrival rate的时候的响应时间</div>
<div>2.最佳的working threads</div>
<div>**
**</div>
<div>**When it is useful?**</div>
At this moment I am looking to answer on questions like:
- When we add additional node to a cluster (e.g. XtraDB Cluster), how does it affect the response time ?
- When we put a load to two nodes instead of three nodes, will it help to improve the response time ?
- Do we need to increase number of working threads when we add nodes ?

Beside cluster testing, it will also help to see an affect of having a side on the server. For example, the famous problem with [DROP TABLE performance](http://www.mysqlperformanceblog.com/2011/04/20/drop-table-performance/). Does DROP TABLE, running in separate session, affect a response time of queries that handle user load ? The same for mysqldump, how does it affect short user queries ?
<div>In fact I have a prototype based on sysbench. It is there [lp:~vadim-tk/sysbench/inj-rate/](https://code.launchpad.net/~vadim-tk/sysbench/inj-rate). It works like a regular sysbench, but you need to specify the additional parameter `tx-rate`, which defines an expected arrival rate (in transactions per second).</div>
<div>事实上，我已经做了原型出来（地址如上），和普通的sysbench使用没有两样，通过增加一个参数tx-rate（这个参数定义了期望的arrival rate，TPS）</div>
There are some results from my early experiments. Assume we have an usual sysbench setup, and we target an arrival rate as 3000 transactions per second (regular sysbench transactions). We vary working threads from 1 to 128.

There are results for 16-128 threads (the result is 99% response time, taken every 10 sec. the less is better)
[![alt](Image(2).png)](http://www.mysqlperformanceblog.com/wp-content/uploads/2012/02/rt-16-128-1.png)
<div>We can see that 16 threads give best 99% response time (15.74ms final), 32 threads: 16.75 ms, 64 threads: 25.14ms.
And with 128 threads we have pretty terrible unstable response times, with 1579.91ms final.
That means that 16-32 threads is probably best number of concurrently working threads (for this kind of workload and this arrival rate).</div>
<div>我使用它进行了一次并发测试，发现并发数量在16-32之间时，响应时间最低。</div>
Ok, but what happens if we have not enough working threads? You can see it from following graph (1-8 threads):
[![alt](Image(3).png)](http://www.mysqlperformanceblog.com/wp-content/uploads/2012/02/rt-1-8-1.png)
The queue piles up, waiting time grows, and the final response time grows linearly up to ~30 sec, where benchmark stops, because the queue is full.

I am looking for your comments, do you find it useful?

[Follow @VadimTk](https://twitter.com/VadimTk)

</div>
&nbsp;