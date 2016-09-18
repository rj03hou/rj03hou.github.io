---
layout: post 
title: "更改Innodb 数据页大小优化MySQL"
subTitle: 
heroImageUrl: 
date: 2010-11-24
tags: ["innodb","MySQL","mysql"]
keywords: 
---

<div id="_mcePaste">
<div id="_mcePaste">作者：吴炳锡　来源：http://www.mysqlsupport.cn/ 联系方式： wubingxi#gmail.com</div>
<div></div>
<div>我们知道Innodb的数据页是16K,而且是一个硬性的规定，系统里没更改的办法，希望将来MySQL也能也Oracle一样支持多种数据页的大小。</div>
<div id="_mcePaste">但实际应用中有时16K显的有点大了，特别是很多业务在Oracle或是SQL SERVER运行的挺好的情况下迁到了MySQL上发现IO增长太明显的情况下，就会想到更改数据页大小了。</div>
<div id="_mcePaste">实际上innodb的数据页大小也是可以更改的，只是需要在源码层去更改，然后重新rebuild一下MySQL.</div>
<div id="_mcePaste">更改办法：</div>
<div id="_mcePaste">(以MySQL-5.1.38源码为例）</div>
<div id="_mcePaste">位置在storage/innobase/include/univ.i ，在univ.i中查找：UNIV_PAGE_SIZE</div>
<div id="_mcePaste">/*</div>
<div id="_mcePaste">DATABASE VERSION CONTROL</div>
<div id="_mcePaste">========================</div>
<div id="_mcePaste">*/</div>
<div id="_mcePaste">/* The universal page size of the database */</div>
<div id="_mcePaste">#define UNIV_PAGE_SIZE          (2 * 8192) /* NOTE! Currently, this has to be a</div>
<div id="_mcePaste">power of 2 */</div>
<div id="_mcePaste">/* The 2-logarithm of UNIV_PAGE_SIZE: */</div>
<div id="_mcePaste">#define UNIV_PAGE_SIZE_SHIFT 14</div>
<div id="_mcePaste">/* Maximum number of parallel threads in a parallelized operation */</div>
<div id="_mcePaste">#define UNIV_MAX_PARALLELISM 32</div>
<div id="_mcePaste">UNIV_PAGE_SIZE就是数据页大小，默认的是16K. 后面的备注里标明，该值是可以设置必须为2的次方。对于该值可以设置成4k,8k,16k,32K,64K，在大也没意义了。</div>
<div id="_mcePaste">同时更改了UNIV_PAGE_SIZE后需要更改 UNIV_PAGE_SIZE_SHIFT 该值是2的多少次方为UNIV_PAGE_SIZE，所以设置数据页分别情况如下：</div>
<div id="_mcePaste">#define UNIV_PAGE_SIZE_SHIFT 12  if UNIV_PAGE_SIZ=4K</div>
<div id="_mcePaste">#define UNIV_PAGE_SIZE_SHIFT 13  if UNIV_PAGE_SIZ=8K</div>
<div id="_mcePaste">#define UNIV_PAGE_SIZE_SHIFT 15  if UNIV_PAGE_SIZ=32K</div>
<div id="_mcePaste">例子：</div>
<div id="_mcePaste">更改innodb的数据页为8K,相应修改为：</div>
<div id="_mcePaste">/*</div>
<div id="_mcePaste">DATABASE VERSION CONTROL</div>
<div id="_mcePaste">========================</div>
<div id="_mcePaste">*/</div>
<div id="_mcePaste">/* The universal page size of the database */</div>
<div id="_mcePaste">#define UNIV_PAGE_SIZE          8192   /* NOTE! Currently, this has to be a</div>
<div id="_mcePaste">power of 2 */</div>
<div id="_mcePaste">/* The 2-logarithm of UNIV_PAGE_SIZE: */</div>
<div id="_mcePaste">#define UNIV_PAGE_SIZE_SHIFT 13</div>
<div id="_mcePaste">/* Maximum number of parallel threads in a parallelized operation */</div>
<div id="_mcePaste">#define UNIV_MAX_PARALLELISM 32</div>
<div id="_mcePaste">重新编译，然后测试测试，再测试。Good luck!</div>
<div><span style="font-family: Verdana, Helvetica, sans-serif; line-height: 20px; font-size: small;">后补：
对于该值的修改如果存在担心可以用 grep -r "UNIV_PAGE_SIZE" * 在innobase那个目录 看看调用的位置去肯定一下。欢迎拍砖。目前我测试没问题。可以用于生产。</span></div>
<div><span style="font-family: Verdana, Helvetica, sans-serif; line-height: 20px; font-size: small;">这篇文章很给力，打算下一个阶段的时候用实际的环境测试一下page size对实际应用的影响。</span></div>
</div>