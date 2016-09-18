---
layout: post 
title: "linux下export无效的原因"
subTitle: 
heroImageUrl: 
date: 2010-12-2
tags: ["Linux","linux"]
keywords: 
---

<div>export的用途是将自定义变量转成环境变量，这样该变量可以继续再<span style="color: #ff0000;">子程序</span>中使用。1、执行脚本时是在一个子shell环境运行的，脚本执行完后该子shell自动退出；2、一个shell中的系统环境变量才会被复制到子shell中（用export定义的变量）；3、一个shell中的系统环境变量只对该shell或者它的子shell有效，该shell结束时变量消失（并不能返回到父shell中）。3、不用export定义的变量只对该shell有效，对子shell也是无效的。</div>
定义下面两个脚本：
s1
<pre>#!/bin/sh
name1="123"
export name1
echo $name1</pre>
s2
<pre>#!/bin/sh
echo $name1</pre>
<div>上面执行sh s2的时候失败的原因，是因为s2不是s1的子程序</div>
可以通过下面几种方法实现s2中正常显示结果：
一、将s1进行修改
<pre>#!/bin/sh
name1="123"
export name1
echo $name1
sh s2</pre>
再次执行sh s1，就可以按照预期显示了。
二、在shell中export name1=123，然后执行sh s2也可以正常显示。
三、使用source命令，source s1;sh s2。source命令是这样解释的：Read and execute commands from filename in the current shell environment and return the exit status of the last command executed  from  filename.也就是source命令执行是在当前shell执行，因此s2就成了s1的子shell。