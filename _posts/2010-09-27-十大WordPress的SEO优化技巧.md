---
layout: post 
title: "十大WordPress的SEO优化技巧"
subTitle: 
heroImageUrl: 
date: 2010-9-27
tags: ["SEO","wordpress"]
keywords: 
---

[WordPress](http://www.williamlong.info/?tags=WordPress)系统本身，默认安装的情况下使用默认模板，实际上对搜索引擎并不友好，并没有针对搜索引擎进行很好的设计，下面我介绍一些技巧和方法可以使得WordPress能否对搜索引擎更为友好。

**1、文章URL链接结构的优化**

Permalink里面要包含postname.一般的服务器都支持mod_rewrite功能，使用这个功能可以优化Permalink（永久链接），在Option-Permalink里的Common options里进行设置，我比较倾向于使用/%year%/%monthnum%/%postname%.html这种链接结构，一来链接目录只有两级，利于索引，二来这种链接结构和Blogspot和Movable Type的链接结构一致，比较利于系统平滑迁移或切换。postname使用英文，如果是写英文Blog的话，系统会自动将标题的post slug做为postname.
<div id="_mcePaste">%year%</div>
<div id="_mcePaste">The year of the post, four digits, for example 2004</div>
<div id="_mcePaste">%monthnum%</div>
<div id="_mcePaste">Month of the year, for example 05</div>
<div id="_mcePaste">%day%</div>
<div id="_mcePaste">Day of the month, for example 28</div>
<div id="_mcePaste">%hour%</div>
<div id="_mcePaste">Hour of the day, for example 15</div>
<div id="_mcePaste">%minute%</div>
<div id="_mcePaste">Minute of the hour, for example 43</div>
<div id="_mcePaste">%second%</div>
<div id="_mcePaste">Second of the minute, for example 33</div>
<div id="_mcePaste">%postname%</div>
<div id="_mcePaste">A sanitized version of the title of the post (post slug field on Edit Post/Page panel). So "This Is A Great Post!" becomesthis-is-a-great-post in the URI (see Using only %postname%). Starting Permalinks with %postname% is strongly not recommended for performance reasons..</div>
<div id="_mcePaste">%post_id%</div>
<div id="_mcePaste">The unique ID # of the post, for example 423</div>
<div id="_mcePaste">%category%</div>
<div id="_mcePaste">A sanitized version of the category name (category slug field on New/Edit Category panel). Nested sub-categories appear as nested directories in the URI. Starting Permalinks with %category% is strongly not recommended for performance reasons.</div>
<div id="_mcePaste">%tag%</div>
<div id="_mcePaste">A sanitized version of the tag name (tag slug field on New/Edit Tag panel). Starting Permalinks with %tag% is strongly not recommended for performance reasons</div>
<div id="_mcePaste">%author%</div>
<div id="_mcePaste">A sanitized version of the author name. Starting Permalinks with %author% is strongly not recommended for performance reasons</div>
%year%The year of the post, four digits, for example 2004%monthnum%Month of the year, for example 05%day%Day of the month, for example 28%hour%Hour of the day, for example 15%minute%Minute of the hour, for example 43%second%Second of the minute, for example 33%postname%A sanitized version of the title of the post (post slug field on Edit Post/Page panel). So "This Is A Great Post!" becomesthis-is-a-great-post in the URI (see Using only %postname%). Starting Permalinks with %postname% is strongly not recommended for performance reasons..%post_id%The unique ID # of the post, for example 423%category%A sanitized version of the category name (category slug field on New/Edit Category panel). Nested sub-categories appear as nested directories in the URI. Starting Permalinks with %category% is strongly not recommended for performance reasons.%tag%A sanitized version of the tag name (tag slug field on New/Edit Tag panel). Starting Permalinks with %tag% is strongly not recommended for performance reasons%author%A sanitized version of the author name. Starting Permalinks with %author% is strongly not recommended for performance reasons

**2、文章Post Slug的优化**

文章标题中最好包含文章最关键的关键字，不要使用一些没有意义的标题，对于英文Blog来讲，最好启用一个名叫[SEO Slugs](http://www.vretoolbar.com/news/seo-slugs-wordpress-plugin)的插件，该插件能够自动将post slug中的the、in等"没用"的单词删除，有利于SEO.

**3、文章Title的优化**

WordPress默认的Title是"博客名-文章名"，这对SEO很不好，我觉得应该使用"文章名-博客名"的形式，建议安装一个名叫[All in One SEO Pack](http://wp.uberdose.com/2007/03/24/all-in-one-seo-pack/)的插件，可以自动将Title进行优化，并增加Descriptions和Keywords的Meta.

**4、robots.txt的优化**

在博客根目录下放置一个robots.txt的文件，可以指定搜索引擎只收录指定的内容。 对于WordPress来说，有一些地址是不应该被搜索引擎索引的，比如后台程序、日志文件、FEED地址等，一个针对WordPress的robots.txt的例子如下：

User-agent: *
Disallow: /wp-
Disallow: /feed/
Disallow: /comments/feed
Disallow: /trackback/

**5、Sitemap的优化**

对于Google搜索引擎来讲，使用[Sitemap](http://www.williamlong.info/archives/327.html)可以让搜索引擎更为有效的进行索引，安装一个名叫[Sitemap Generator](http://www.arnebrachhold.de/2005/06/05/google-sitemaps-generator-v2-final)的插件可以自动完成Google Sitemap的生成，然后将这个地址提交到[Google Webmaster](http://www.google.com/webmasters/)即可。

**6、防止垃圾留言评论**

垃圾留言评论会影响Blog在搜索引擎中的表现，因此需要安装一个自动过滤垃圾留言评论的的插件，推荐使用[Akismet](http://akismet.com/)。

**7、相关文章**

通过tag的标记来实现相关文章，不过我建议使用WordPress 2.3里面的tag系统来实现，那样效率会更高一些。

**8、搜索引擎来源的优化**

安装一个名叫[Landing sites](http://theundersigned.net/2006/06/landing-sites-11/)的插件，可以让那些从搜索引擎搜索过来的用户体验更好，通过这个插件能够选择显示给用户搜索关键字相关的文章。

**9、不要轻易做变动**

不要总是草率的变动自己的域名、博客名、链接结构、链接地址等，早期应该做全局的规划，中途进行大的变动是非常不明智的。

**10、更新你的博客**

记着经常更新，并且写出高质量的内容，这才是SEO中最关键的地方，写出高质量的文章，将会更容易实现SEO的目标。

下面是一些其他中文的WordPress SEO优化技巧文章：

[SEO for WordPress 完全指南](http://www.osxcn.com/wordpress/seo-for-wordpress.html)

[10步实现WordPress搜索引擎优化](http://blogunion.org/wordpress/wordpress-tips/wordpress-seo.html)

下面是一些英文的WordPress SEO优化技巧文章：

[Top Ten WordPress SEO Tips](http://www.moon-blog.com/2008/02/top-ten-wordpress-seo-tips.html)

[Search Engine Optimization (SEO) Tips for Wordpress 2.0](http://www.solostream.com/2006/01/16/savvy-search-engine-optimization-seo-tips-for-wordpress-20/)

[8 simple SEO tips for blogs](http://www.johntp.com/2006/04/21/8-simple-seo-tips-for-blogs/)

[Optimize Wordpress for Search Engines](http://www.geek-notes.com/wordpress/25/optimize-wordpress-for-search-engines/)

转自：[月光博客](http://www.williamlong.info/archives/1050.html)

因为[我自己现在的博客](http://www.themysql.com)就是按照这个来进行优化，所以就贴在了这里