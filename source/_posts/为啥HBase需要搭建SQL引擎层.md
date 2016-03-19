title: 为啥HBase需要搭建SQL引擎层
tags:
  - HBase
categories:
  - 读书笔记
date: 2016-02-12 21:51:00
---
现有的SQL解决方案通常都不是水平可伸缩的，因此当数据量变大时会遇到障碍。但是这样的情况，随着NoSQL的出现已经得到很大程度的缓解，并且随着NoSQL技术的完善与成熟，这种情况将会从根本上解决。

<!-- more -->

我们知道NoSQL区别于关系型数据库的一点就是NoSQL不使用SQL作为查询语言，至于为何在NoSQL数据存储HBase上提供SQL接口，有如下原因：

1.使用诸如SQL这样易于理解的语言，使人们能够更加轻松地使用HBase。

2.使用诸如SQL这样更高层次的语言来编写，减少了编写的代码量。

3.执行查询时，在数据访问与运行时执行之间加上SQL这样一层抽象可以进行大量优化。例如，对于GROUP BY查询来说，利用HBase中协同处理器，聚合可以在服务器上进行，而不必在客户端，这么做会极大减少客户端与服务器之间传输的数据量。此外，也可以在客户端并行执行GROUP BY，这是根据行健的范围来截断扫描而实现的。通过并行执行，结果会更快的返回。所有这些优化无需用户参与，只需执行查询即可。


<B>基于HBase的SQL引擎实现</B>

现阶段业内有一些关于HBase SQL引擎层的尝试，已经有一些比较稳定的解决方案和现实。

1.Hive整合HBase

Hive与HBase的整合功能从Hive0.6.0版本已经开始出现，利用两者对外的API接口互相通信，通信主要依靠hive_hbase-handler.jar工具包（Hive Storage Handlers）。由于HBase有一次比较大的版本变动，所以并不是每个版本的Hive都能和现有的HBase版本进行整合，所以在使用过程中特别注意的就是两者版本的一致性。

2.Phoenix

Phoenix由Salesforce.com开源，是构建在Apache HBase之上的一个SQL中间层，可以让开发者在HBase上执行SQL查询。Phoenix 完全使用Java编写，代码位于Github上，并且提供了一个客户端可嵌入的JDBC驱动。对于10w到100w行的简单查询来说，Phoenix要胜于Hive。

3.Kundera

Kundera 是一个JPA2.0兼容的NoSQL数据存储的对象映射框架。Kundera基于现有类库构建，封装出简易的API，其主要特性有：

1）支持交叉数据存储持久性，这意味着用户可以在不同的数据存储使用单一方法存储和获取相关实体。
2)能够很好地管理事务，同时支持EntityTransaction和Java Transaction API（JPA）。
3) 兼容JPA2.0，严格使用JPA注释对象映射到数据存储表。
4) 目前支持的NoSQL服务器包括: HBase,MongoDB,Redis,Neo4j等。

还有其它一些解决方案，例如：Lealone,hbase-sql,Impala等，要么不成熟，要么停止更新了，要么具有局限性。读者对其感兴趣，可以自行去了解。
