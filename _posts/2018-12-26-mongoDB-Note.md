---
layout:     post
title:      mongoDB复习
date:       2018-12-26 12:12:00
summary:    mongoDb与node.js结合学习
categories: jekyll pixyll
---
为什么开发者喜欢MongoDB
> 无论系统需要单个还是多个节点，mongodb都可以提供高性能，如果你经过关系型数据库的伸缩困境，那么使用MongoDB就可以避免这样的困境，但是并非每个人都需要**伸缩性操作**，如果你需要的就是单台数据库服务器，那么为啥还要使用MongoDB呢? ----**直观的数据模型**。



### 索引
> 索引通过构建可以很容易进行排序的查找索引来改善经常执行的查询的性能。集合中_id属性可以自动建立索引，因为通过id查找条目是一种常见的方式。

### 分片
> 分片是对数据的大集合进行切片的过程，这种大集合九二一被划分到集合中的多个MongoDb服务器。每个Mongodb服务器可以被认为是一个分片。这提供了利用多台服务器来支持针对大系统大量请求的好处。因此，它提供了数据库的横向拓展。你应该观察数据的大小和将要访问它的请求的数量，以确定是否对集合分片和分多少片。

### 复制
> 复制是对在集合中的多个MongoDb实例上的数据进行拷贝的过程，在考虑数据库的可靠性方面，应实现复制，以确保关键数据的备份副本始终是随时可用的。


### mongoDB in action
concept:
 - 索引机制和查询优化、mongoDB内部的文本搜索引擎
 - WiredTiger存储引擎和可插拔存储  这是MongoDB v3之后独有的特性
 - 复制，使用MongoDB 高可用策略以及伸缩性内容
 - 分片sharding存储、Mongo水平伸缩特性


大部分开发者使用的是面向对象编程语音，他们希望找到更好的数据库用于映射，使用MongoDB，语音中定义的对象可以原样持久化保存，减少了对象映射的复杂性。

{% highlight sql %}
SELECT * FROM posts
INNER JOIN posts_tags ON posts.id = posts_tags.post_id
INNER JOIN tags ON posts_tags.tag_id = tags.id
WHERE tags.text = 'politics' AND posts.vote_count >10;


db.posts.find({'tags':"politics",'vote_count':{'$gt':10}});
{% endhighlight %}

#### 索引
** ad hoc查询的一个关键元素就是查找在创建数据库时还不知道的值，随着数据库中添加的文档数据越来越多，查询值得成本变得越来越高，因此需要一种高效得方式来搜索数据。 理解数据库索引最好的方法就是类比：许多书籍都有索引，包括关键字和页码。**
MongoDB的索引就是使用了B-树(平衡树)数据结构。B-树索引也引用了大量使用于许多关系型数据库中，对于不同的查询做了优化，包括**范围扫描和条件子句查询**。但是新的引擎已经支持日志结构合并-树(LSM)。

** 使用MongoDB，每个集合我们可以创建64个索引。这些索引也可以在其他关系数据库中找到：升序、降序、复合键、哈希、文本以及地理空间索引。因为MongoDB和绝大多数关系型数据库RDBMS使用了相同的索引数据结构，所以管理这些系统的建议都是类似的。**
