---
layout: post 
title: "redis和mongodb比较"
subTitle: 
heroImageUrl: 
date: 1970-1-1
tags: ["mongodb","NoSQL","NoSQL","redis"]
keywords: 
---

原文：[MongoDB and Redis: a different interpretation of what's wrong with Relational DBs](http://antirez.com/post/MongoDB-and-Redis.html)
作者：antirez（redis作者）
Working to Redis is a good feeling for me: it's not something about money, or deadlines, or customers not agreeing with me, but about trying to do my 2 cents in order to help the field to go forward. It's a joy to work to things you love, especially if you have the feeling that you don't want to win: even if a few ideas of your work will be useful for another experiment or implementation it is already worth it. It's like science, what matters is to know more, to find better solutions to problems, and so on.

So I check other work on other key value databases, and suggest this databases to people interested in something different than Redis. For instance this MongoDB slides are good and worth a look. MongoDB seems an interesting project to me, and the interesting thing is how Redis and MongoDB try to solve the same problem in theory but with a very different analysis of it.

Both the projects are about there is something wrong if we use an RDBMS for all the kind of works. Not all the problems look like a nail but too much databases look like an hammer, the slide says, and indeed it's a colorful imagine to communicate. But it is remarkable how, in response to the same non-nail problems, this two tools taken different paths.
MongoDB

Before to continue I want to spend some word about how MongoDB works. The idea is to have objects, that are actually a sum of named fields with values. A Mongo DB object looks like this:
Name: Salvatore
Surname: Sanfilippo
Foo: yes
Bar: no
age: 32
That is, actually, very similar to an RDBMS table. Then you can run interesting queries against your object collections:
db.collection.find.({'Name':'John'}) # Finds all Johns
db.collection.find.({$where:'this.age >= 6 && this.age <= 20'})
You can have indexes in given fields, like in RDBMS, and can sort your queries against some field, order it, get a range using LIMIT, and so on. Basically the data model is the same as an RDBMS, so the MongoDB developers main idea is the following, in my opinion:
What's wrong with RDBMS when used for (many) tasks that don't need all this complexity? They are bloated, thus slow and a pain to replicate, shard, ... But the data model is right, to have tables and index and run complex queries against data.
The Redis path

Redis tries to solve non nail problems too indeed. But in a different way: what Redis provides are data structures much more similar to the data structures you find in a computer science book, liked lists, Sets, and server side operations against this kind of values. Programming with Redis is just like doing everything with Lists and Hashes inside memory with your favorite dynamic programming language, but the dataset is persistent and of course not as fast as accessing directly to your PC's memory (there is a networking layer in the middle).

So what's the Redis point of view?
What's wrong with RDBMS when used for (many) tasks that don't need all this complexity? The data model: non scalable, time complexity hard to predict, and can't model many common problems well enough.
I expect that in a few years what was the real problem with RDBMS is going to be very clear, even if now it can look confusing enough and there are different alternatives and it is very hard to meter the relative value of the different solutions proposed. This kind of changes appear to be very fast, with all the key-value hype growing every week, but actually it's going to take much more time before to start considering RDBMS alternatives as conceptually mature as we look at RDBMS today.