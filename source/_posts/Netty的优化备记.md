title: Netty的优化备记

---

1.当心“日志隐形杀手”

通常情况下，大家都知道不能在Netty的I/O线程上做执行时间不可控的操作，例如访问数据库、发送Email等。但是有个常用但是非常危险的操作却容易被忽略，那便是记录日志。

通常，在生产环境中，需要实时打印接口日志，其它日志处于ERROR级别，当推送服务发生I/O异常之后，会记录异常日志。如果当前磁盘的WIO比较高，可能会发生写日志文件操作被同步阻塞，阻塞时间无法预测。这就会导致Netty的NioEventLoop线程被阻塞，Socket链路无法被及时关闭、其它的链路也无法进行读写操作等。

<!-- more -->

以最常用的log4j为例，尽管它支持异步写日志（AsyncAppender），但是当日志队列满之后，它会同步阻塞业务线程，直到日志队列有空闲位置可用，相关代码如下：

        synchronized (this.buffer) {
              while (true) {
                int previousSize = this.buffer.size();
                if (previousSize < this.bufferSize) {
                  this.buffer.add(event);
                  if (previousSize != 0) break;
                  this.buffer.notifyAll(); break;
                }
                boolean discard = true;
                if ((this.blocking) && (!Thread.interrupted()) && (Thread.currentThread() != this.dispatcher)) //判断是业务线程
                {
                  try
                  {
                    this.buffer.wait();//阻塞业务线程
                    discard = false;
                  }
                  catch (InterruptedException e)
                  {
                    Thread.currentThread().interrupt();
                  }
        }
类似这类BUG具有极强的隐蔽性，往往WIO高的时间持续非常短，或者是偶现的，在测试环境中很难模拟此类故障，问题定位难度非常大。这就要求读者在平时写代码的时候一定要当心，注意那些隐性地雷。

2.TCP参数优化

常用的TCP参数，例如TCP层面的接收和发送缓冲区大小设置，在Netty中分别对应ChannelOption的SO_SNDBUF和SO_RCVBUF，需要根据推送消息的大小，合理设置，对于海量长连接，通常32K是个不错的选择。

另外一个比较常用的优化手段就是软中断，如图所示：如果所有的软中断都运行在CPU0相应网卡的硬件中断上，那么始终都是cpu0在处理软中断，而此时其它CPU资源就被浪费了，因为无法并行的执行多个软中断。

大于等于2.6.35版本的Linux kernel内核，开启RPS，网络通信性能提升20%之上。RPS的基本原理：根据数据包的源地址，目的地址以及目的和源端口，计算出一个hash值，然后根据这个hash值来选择软中断运行的cpu。从上层来看，也就是说将每个连接和cpu绑定，并通过这个hash值，来均衡软中断运行在多个cpu上，从而提升通信性能。

3.JVM参数

最重要的参数调整有两个：

-Xmx:JVM最大内存需要根据内存模型进行计算并得出相对合理的值；
GC相关的参数: 例如新生代和老生代、永久代的比例，GC的策略，新生代各区的比例等，需要根据具体的场景进行设置和测试，并不断的优化，尽量将Full GC的频率降到最低。

4.内存池

推送服务器承载了海量的长链接，每个长链接实际就是一个会话。如果每个会话都持有心跳数据、接收缓冲区、指令集等数据结构，而且这些实例随着消息的处理朝生夕灭，这就会给服务器带来沉重的GC压力，同时消耗大量的内存。

最有效的解决策略就是使用内存池，每个NioEventLoop线程处理N个链路，在线程内部，链路的处理时串行的。假如A链路首先被处理，它会创建接收缓冲区等对象，待解码完成之后，构造的POJO对象被封装成Task后投递到后台的线程池中执行，然后接收缓冲区会被释放，每条消息的接收和处理都会重复接收缓冲区的创建和释放。如果使用内存池，则当A链路接收到新的数据报之后，从NioEventLoop的内存池中申请空闲的ByteBuf，解码完成之后，调用release将ByteBuf释放到内存池中，供后续B链路继续使用。



如果觉得还不错，赏点酒钱！
![](/images/aex068188cqwy9xbxa3oc07.png)
