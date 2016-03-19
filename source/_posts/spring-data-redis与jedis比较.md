title: spring data redis与jedis比较
tags:
  - spring-data-redis
  - jedis
categories:
  - 随笔
date: 2016-01-09 11:21:00
---
redis客户端实现方式有两种，一种是直接调用jedis来实现，一种是使用spring data redis,做了一些封装来调用。

<!-- more -->

#spring data redis:


*  对具体redis客户端做了封装，客户端可在jedis,jredis,rjc等Java客户端中做出选择和切换
*  用template对调用做了封装，省去了建立连接，释放连接等繁琐代码。
*  对对象的序列化也可自由选择工具。

*  提供对spring cache的支持，可用注解实现Cache —— 但是无法设定缓存失效时间。 （这块我没怎么验证过）

#通过对spring data redis的源码分析，具体提供了一些功能:

* 连接池自动管理，提供了一个高度封装的“RedisTemplate”类
* 针对jedis客户端中大量api进行了归类封装,将同一类型操作封装为operation接口
* 提供了对key的“bound”(绑定)便捷化操作API，可以通过bound封装指定的key，然后进行一系列的操作而无须“显式”的再次指定Key
* 将事务操作封装，有容器控制。
* 针对数据的“序列化/反序列化”，提供了多种可选择策略。
* 基于设计模式，和JMS开发思路，将pub/sub的API设计进行了封装，使开发更加便捷。  
* spring-data-redis中，并没有对sharding提供良好的封装，如果你的架构是基于sharding，那么你需要自己去实现，这也是sdr和jedis相比，唯一缺少的特性。

#jedis:

* connection管理缺乏自动化，connection-pool的设计缺少必要的容器支持。
* 数据操作需要关注“序列化”/“反序列化”，因为jedis的客户端API接受的数据类型为string和byte，对结构化数据(json,xml,pojo等)操作需要额外的支持。
* 事务操作纯粹为硬编码
* pub/sub功能，缺乏必要的设计模式支持，对于开发者而言需要关注的太多。
* jedis最大优势就是sharded，比如Masater/Slaver。这也是我为啥项目发展到一定地步，我需要把spring-data-redis换成jedis
