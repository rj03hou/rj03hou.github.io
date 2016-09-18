---
layout: post 
title: "mysql中idle connections的影响"
subTitle: 
heroImageUrl: 
date: 2010-12-21
tags: ["MySQL","原创"]
keywords: 
---

Yves Trudeau先是在mysqlhighperformance上发表了一篇文章指出idle-connections对mysql性能的影响，后来下面的回复中有人指出造成这种影响的是原因是作者是以debug模式编译的mysql，Yves Trudeau意识到之后又发了一篇，使用正式版进行测试，测试的结论是影响相对较小，贴上两次的图（右边的是debug版本，左边的是标准版），感谢作者这种孜孜追求：

[![](NOTPM_vs_idle_conn-1023x578-300x169.png "debug测试结果")](http://www.themysql.com/wp-content/uploads/2010/12/NOTPM_vs_idle_conn-1023x578.png)[![](NOTPM_vs_idle_conn_v2-1023x578-300x169.png "标准版测试结果")](http://www.themysql.com/wp-content/uploads/2010/12/NOTPM_vs_idle_conn_v2-1023x578.png)

dubug版本原文：[http://www.mysqlperformanceblog.com/2010/12/17/impact-of-the-number-of-idle-connections-in-mysql/](http://www.mysqlperformanceblog.com/2010/12/17/impact-of-the-number-of-idle-connections-in-mysql/)

[ ](http://www.mysqlperformanceblog.com/2010/12/17/impact-of-the-number-of-idle-connections-in-mysql/)正式版原文：[http://www.mysqlperformanceblog.com/2010/12/20/impact-of-the-number-of-idle-connections-in-mysql-version-2/](http://www.mysqlperformanceblog.com/2010/12/20/impact-of-the-number-of-idle-connections-in-mysql-version-2/)

作者：**Yves Trudeau**

** **[ ](http://www.mysqlperformanceblog.com/2010/12/17/impact-of-the-number-of-idle-connections-in-mysql/)