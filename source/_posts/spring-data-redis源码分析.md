title: spring data redis源码分析
tags:
  - spring-data-redis
categories:
  - 源码分析
date: 2016-01-09 13:24:00
---
既然涉及到源码分析，最好的方式就是去看开源出来的代码。


https://github.com/spring-projects/spring-data-redis

<!-- more -->

简单介绍下，Spring-data-redis为spring-data模块中对redis的支持部分，简称为“SDR”，提供了基于jedis客户端API的高度封装以及与spring容器的整合。

下面是spring data redis class diagram。

![](/images/1.jpg)


简单的说：

1.提供了一个高度封装的“RedisTemplate”类，进行了连接池自动管理。

如果你使用过jedisPool连接池，在数据操作之前，你需要pool.getResource()即从连接池中获取“链接资源”(Jedis),在操作之后，你需要(必须)调用pool.returnResource()将资源归还个连接池。但是，spring-data-redis中，我们似乎并没有直接操作pool，那么spring是如何做到pool管理的呢？？一句话：spring的“看门绝技”--callback。
 public <T> T execute(RedisCallback<T> action)：这个方法是redisTemplate中执行操作的底层方法，任何基于redisTemplate之上的调用(比如，valueOperations)最终都会被封装成RedisCallback，redisTemplate在execute方法中将会直接使用jedis客户端API进行与server通信，而且在如果使用了连接池，则会在操作之后执行returnSource。


2.同时针对jedis客户端中大量api进行了归类封装,将同一类型操作封装为operation接口

ValueOperations：简单K-V操作
SetOperations：set类型数据操作
ZSetOperations：zset类型数据操作
HashOperations：针对map类型的数据操作
ListOperations：针对list类型的数据操作

3.源码中你可以发现RedisSerializer接口，其实是针对数据的“序列化/反序列化”做了处理。

JdkSerializationRedisSerializer：POJO对象的存取场景，使用JDK本身序列化机制，将pojo类通过ObjectInputStream/ObjectOutputStream进行序列化操作，最终redis-server中将存储字节序列。是目前最常用的序列化策略。
StringRedisSerializer：Key或者value为字符串的场景，根据指定的charset对数据的字节序列编码成string，是“new String(bytes, charset)”和“string.getBytes(charset)”的直接封装。是最轻量级和高效的策略。
JacksonJsonRedisSerializer：jackson-json工具提供了javabean与json之间的转换能力，可以将pojo实例序列化成json格式存储在redis中，也可以将json格式的数据转换成pojo实例。因为jackson工具在序列化和反序列化时，需要明确指定Class类型，因此此策略封装起来稍微复杂。【需要jackson-mapper-asl工具支持】
OxmSerializer：提供了将javabean与xml之间的转换能力，目前可用的三方支持包括jaxb，apache-xmlbeans；redis存储的数据将是xml工具。不过使用此策略，编程将会有些难度，而且效率最低；不建议使用。【需要spring-oxm模块的支持】
    针对“序列化和发序列化”中JdkSerializationRedisSerializer和StringRedisSerializer是最基础的策略，原则上，我们可以将数据存储为任何格式以便应用程序存取和解析(其中应用包括app，hadoop等其他工具)，不过在设计时仍然不推荐直接使用“JacksonJsonRedisSerializer”和“OxmSerializer”，因为无论是json还是xml，他们本身仍然是String。
    如果你的数据需要被第三方工具解析，那么数据应该使用StringRedisSerializer而不是JdkSerializationRedisSerializer。
    如果你的数据格式必须为json或者xml，那么在编程级别，在redisTemplate配置中仍然使用StringRedisSerializer，在存储之前或者读取之后，使用“SerializationUtils”工具转换转换成json或者xml。



如果觉得还不错，赏点酒钱！

![](/images/aex068188cqwy9xbxa3oc07.png)
