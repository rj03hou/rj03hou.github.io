---
layout: post 
title: "关于MySQLdb的一些新发现（使用python连接mysql的库）"
subTitle: 
heroImageUrl: 
date: 2010-9-24
tags: ["MySQL","mysql","MySQLdb","Python","python"]
keywords: 
---

MySQLdb的文档时通过python的工具自动将注释生成的，所以文档的可读性不是很强。下面是通过其他的文章发现的两个比较好点的使用方式：

一、
cursor.execute("select id,ip,port from db limit 5")
for (id,ip,port) in cursor.fetchall():
print id

这样比row[0][0]...这种方式阅读性要好一点，并且代码比较简洁

二、

cursor = conn.cursor (MySQLdb.cursors.DictCursor)
cursor.execute ("SELECT name, category FROM animal")
result_set = cursor.fetchall ()
for row in result_set:
print "%s, %s" % (row["name"], row["category"])

效果同上，感觉相比而言更喜欢第一种，简洁可读性等都较高

三、

cursor有个属性rowcount返回受影响的行数，这个在文档中没有找到，但是实际测试的时候发现确实包含了这个属性，有兴趣的娃可以考证一下。

[官方文档地址](http://mysql-python.sourceforge.net/MySQLdb-1.2.2/public/MySQLdb-module.html)