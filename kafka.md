# kafka

### 1.简要

分布式流平台。

基于zookeeper的分布式消息系统

高吞吐量，高性能，实时，高可靠

消费模式为拉取的模式

kafka消费是存储在磁盘中。默认存7天

### 2.基本概念

![image-20200827213157137](C:\Users\lc\AppData\Roaming\Typora\typora-user-images\image-20200827213157137.png)

topic：由1到多个Partitions组成

partition：实际消息存储单位，一个topic可以分布到多个broker上，一个topic可以分为多个partition，每个partition是一个有序队列。

producer：生产者

consumer：消费者

broker：一个kafka服务器就是一个broker

consumer  group：由多个consumer组成，消费者组内每个消费者负责消费不同分区的数据。**一个分区（partition）只有由一个组内消费者消费。消费者组间互不影响**

replica：副本，为保证集群中某个节点故障时，节点上partition的数据不丢失，kafka提供副本机制，一个topic的每个分区都有若干个副本。一个leader和若干个follower。

leader：每个分区多个副本的主，生产者发送数据的对象以及消费者消费数据的对象都是leader。

follower:每个分区多个副本中的从，实时从leader中同步数据，leader故障时，某个follower会成为新的leader

**zookeeper作用：**为kafka集群存储集群信息。为消费者存储消费到的位置信息（offset（0.9之间存在zookeeper，0.9之后存在kafka（消费者拉取消息每次都要同zookeeper交互，影响并发量）））。

### 3.kafka客户端

AdminClient API 允许管理和检测topic broker以及其他kafka对象

Producer API发布消息到1个或多个topic

Consumer API订阅一个或多个topic

Stream API 高效地将输入流转换到输出流

Connector API 从一些源系统或应用程序中拉取数据到kafka

**AdminClient API**

![image-20200820155531004](C:\Users\lc\AppData\Roaming\Typora\typora-user-images\image-20200820155531004.png)

```java
    /**
     * 设置AdminClient
     * @return
     */
    public static AdminClient adminClient(){
        Properties properties=new Properties();
        properties.setProperty(AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG,"120.77.241.51:9092");
        AdminClient adminClient=AdminClient.create(properties);
        return  adminClient;
    }
```

**Producer发送**

**异步发送**

**同步发送**，接受到发送结果再继续发送下一条信息。

***

**异步发送回调**

![image-20200820192630960](C:\Users\lc\AppData\Roaming\Typora\typora-user-images\image-20200820192630960.png)

### 4.生产者发送原理

![image-20200820194028971](C:\Users\lc\AppData\Roaming\Typora\typora-user-images\image-20200820194028971.png)

![image-20200820193354719](C:\Users\lc\AppData\Roaming\Typora\typora-user-images\image-20200820193354719.png)

批量发送：不是有一条消息就发送一条与时间阈值以及缓冲区阈值有关。

### 5.消息传递保障

kafka提供三种保障，最多一次，最少一次，正好一次

**最一次**

发送一次

**最少一次**

发送后，等待响应，没有收到响应则会重发。

**正好一次**

使用唯一id+消息发送，broker端作去重。

### 6.消费者

![image-20200823011233837](C:\Users\lc\AppData\Roaming\Typora\typora-user-images\image-20200823011233837.png)

![image-20200823011246826](C:\Users\lc\AppData\Roaming\Typora\typora-user-images\image-20200823011246826.png)

##### 多线程消费

①每个线程单独创建一个消费者，同步阻塞，消费失败可回滚。

![image-20200823013726988](C:\Users\lc\AppData\Roaming\Typora\typora-user-images\image-20200823013726988.png)

②消费者复用，消费者拿到消息，再分配给多线程处理

可以在一个线程池创建一个消费者。但是消费者并不能确定消息是否能够真正处理成功。异步非阻塞

![image-20200823013749565](C:\Users\lc\AppData\Roaming\Typora\typora-user-images\image-20200823013749565.png)

##### 手动指定offset起始位置

kafka不会随意丢弃消息

##### 限流

手动暂停或重启消费。

##### 新成员加入组

![image-20200823015544888](C:\Users\lc\AppData\Roaming\Typora\typora-user-images\image-20200823015544888.png)

kafka在平衡partition和consumer的关系的时候，会要求所有consumer成员断开，然后在重连。为了，会给每个成员颁布一个generation。

##### 组成员崩溃

![image-20200823020034559](C:\Users\lc\AppData\Roaming\Typora\typora-user-images\image-20200823020034559.png)

##### 提交位移

![image-20200823020054649](C:\Users\lc\AppData\Roaming\Typora\typora-user-images\image-20200823020054649.png)

### 7.kafka stream

处理分析存储在kafka数据的客户端程序库

通过state store可以实现高效状态操作

支持源于processor和高层抽象DSL

**关键词**

流及流处理器

流处理拓扑

源处理器及sink处理器

![image-20200823205244277](C:\Users\lc\AppData\Roaming\Typora\typora-user-images\image-20200823205244277.png)

### 8.Connect

![image-20200823210325741](C:\Users\lc\AppData\Roaming\Typora\typora-user-images\image-20200823210325741.png)

用来与其他中间件建立流式通道，支持流式和批量处理集成

### 9.底层原理

kafka吞吐量大的原因？速度快？

日志顺序读写和快速检索

partition机制

批量发送接收机数据压缩机制

通过sendfile实现零拷贝原则（重要原因）

##### 9.1日志

kafka日志是以partition为单位进行保存

日志目录格式为Topic名称+数字

日志文件格式是一个日志条目序列

每条日志消息有四字节整形与N字节消息组成

**日志分段**

每个partition的日志会分为N个大小相等的segment

每个segment中消息数量不一定相等

每个partition只支持顺序读写（顺序读写时磁盘的读写效率可能会比内存要快）

![image-20200823215621997](C:\Users\lc\AppData\Roaming\Typora\typora-user-images\image-20200823215621997.png)

**segment存储结构**

partition会将消息添加到最后个segment上

当segment达到一定阈值会flush到磁盘上，只有刷新到磁盘上，才能被consumer读到，consumer是到磁盘读取的

segment文件分为index和data

![image-20200823215934081](C:\Users\lc\AppData\Roaming\Typora\typora-user-images\image-20200823215934081.png)

消息内容是存在log文件中的，通过读消息头，取出消息内容的长度

**日志读**

在存储的数据中找出segment文件（通过offset找到segmentlist中的某一个segment）

通过全局offset计算出segment中的offset（）

通过index中的offset寻找具体数据内容（）

**日志写**

日志运行串行的追加消息到文件最后

当日志文件达到阈值则滚动到新文件

##### 9.2零拷贝



![image-20200823220542107](C:\Users\lc\AppData\Roaming\Typora\typora-user-images\image-20200823220542107.png)

![image-20200823220718183](C:\Users\lc\AppData\Roaming\Typora\typora-user-images\image-20200823220718183.png)

##### 9.3消费者与消费者组

消费者组是消费的单位

单个partition只能由消费者组中某一个消费者消费

单个消费者可以消费多个partition

##### 9.4producer客户端

![image-20200823221112533](C:\Users\lc\AppData\Roaming\Typora\typora-user-images\image-20200823221112533.png)

 send方法并不是直接发送到kafka服务端，在创建producer工作线程的时候，会开启一个守护线程，轮询发送消息到kafka，send方法传递的消息，会先追加到队列，然后由守护线程发送到kafka。

##### 9.5kafka如何保证顺序性（业务级别）

kafka只支持单partition有序

使用kafka key+offset可以做到业务有序

##### 9.6topic删除

异步线程+延迟操作

![image-20200824002147784](C:\Users\lc\AppData\Roaming\Typora\typora-user-images\image-20200824002147784.png)

### 10.kafka架构

##### 10.1工作流程及存储机制

kafka中的消息是以topic进行分类的，生产者生产消息，消费者消费消息都是面向topic的。

partition是消息的实际存储单位，每个partition对应于一个**topic名称+partition序号**的文件夹，文件夹包含.log和.index文件，相互一一对应，以当前segment的第一条消息的offset命名。

为防止log文件过大导致数据定位效率低，kafka采用**分片****和**索引**机制。每个partition分为多个segment，每个segment对应一个.index和.log。

.index文件存储大量的索引信息，指向offset对应的消息实体在.log文件中的物理偏移位置。

消费者组中的每个消费者都会记录自己消费到了哪个offset。

![image-20200827221724738](C:\Users\lc\AppData\Roaming\Typora\typora-user-images\image-20200827221724738.png)

##### 10.2 分区的原则

没有知名partition的但是有key的情况下，将key的hash值与topic的partition数取余

也没有key的情况下，第一次发送的时候生成一个随机数，对partition总数取余，然后每次递增轮询。

##### 10.3消息可靠性投递

消息发送给leader时，待所有的副本同步完成才回复ack,另外，leader维护一个动态的isr（和leader保持同步的follower集合），如果某一个副本长时间没有同步完成，则会被从集合中剔除。时间阈值由指定参数指定。

**低版本中：**通信延迟时间长的、同步数据差距大的会被剔除isr。

**新版本中：**只会剔除通信延迟时间长的。因为剔除数据差距大的会导致频繁操作isr，频繁操作zk。

**acks配置：**

0：生产者不等待ack,一直发消息。

1：只等待leader写完，就继续发送。

-1：leader写完，isr中的follower同步完成，ack，然后继续重发。

当极限情况，isr中只剩leader，leader挂掉，也会丢失数据。

当leader未ack就挂掉导致数据重发，会出现幂等性问题。

**数据一致性问题：**

LEO：每个副本最大的offset

HW：消费者可见的最大offset，isr中最小的LEO。（消费一致性）

leader故障，从isr中选举一个新的leader，为保证存储数据一致性，命令其他副本截取大于HW的部分，然后从新的leader同步数据。

##### 10.4producer到server消息投递模式

at least once，至少一次，对应ack级别设置为-1，不丢数据，可能会重复消费。

at most once，最多一次，对应ack级别设置为0.

exactly once，正好一次。在至少一次的基础上，对消息去重。对消息设置唯一id。

##### 10.5kafka消费者

**消费模式：**拉模式,服务端没有数据时，会让消费者等待一段时间再来拉取

**分区分配策略:**一个消费者组有多个消费者，一个topic也有多个partition。确定哪个partition由哪个消费者消费

两种策略：**轮询（RoundRobin）和**

**轮询：**将消费者组订阅的所有topic当作一个整体，将所有的partition按hash值排序，轮询分配给消费者组内的消费者。**需要保证消费者组内的消费者订阅的是同一topic**

**range（默认）：**将单个topic内的partition均分给订阅topic的消费者，先多后少。会导致消费者消费消息不对等问题严重。

当消费者个数发生变化时，会触发**重新分配**

##### 10.6offset

消费者组+topic+partition唯一确定一个offset

0.9版本之前同时保存在zk和本地，之后只保存在本地。

##### 10.7高效读取

1.顺序写磁盘

2.零拷贝技术

##### 10.8zookeeper作用

kafka集群中leader、controller选举

##### 10.9事务

0.11开始起引入事务，保证kafka在exactly one的基础上，生产和消费可以跨分区和跨会话，要么全成功，要么全失败。

**producer事务：**

引入一个全局唯一transaction id（客户端设置），

**consumer事务：**

##### 10.10消息发送流程

![image-20200831165951802](C:\Users\lc\AppData\Roaming\Typora\typora-user-images\image-20200831165951802.png)

Kafka 的 Producer 发送消息采用的是异步发送的方式。在消息发送的过程中，涉及到了
两个线程——main 线程和 Sender 线程，以及一个线程共享变量——RecordAccumulator。
main 线程将消息发送给 RecordAccumulator， Sender 线程不断从 RecordAccumulator 中拉取
消息发送到 Kafka broker。

batch.size： 只有数据积累到 batch.size 之后， sender 才会发送数据。
linger.ms： 如果数据迟迟未达到 batch.size， sender 等待 linger.time 之后就会发送数据。

### 11.面试题

##### kafka中的ISR、HW、LEO

##### kafka中的分区器、序列化器、拦截器以及执行顺序。

##### kafka消息发送流程



##### 11.1Kafka Exactly Once实现原理

https://www.zybuluo.com/tinadu/note/949867

at least once +幂等性保证实现exactly once

**幂等性：**

为了实现Producer的幂等语义，Kafka引入了`Producer ID`（即`PID`）和`Sequence Number`。每个新的Producer在初始化的时候会被分配一个唯一的PID，该PID对用户完全透明而不会暴露给用户。

对于每个PID，该Producer发送数据的每个<Topic, Partition>都对应一个从0开始单调递增的SequenceNumber。

类似地，Broker端也会为每个<PID, Topic, Partition>维护一个序号，并且每次Commit一条消息时将其对应序号递增。对于接收的每条消息，如果其序号比Broker维护的序号（即最后一次Commit的消息的序号）大一，则Broker会接受它，否则将其丢弃：

- 如果消息序号比Broker维护的序号大一以上，说明中间有数据尚未写入，也即乱序，此时Broker拒绝该消息，Producer抛出`InvalidSequenceNumber`
- 如果消息序号小于等于Broker维护的序号，说明该消息已被保存，即为重复消息，Broker直接丢弃该消息，Producer抛出`DuplicateSequenceNumber`

上述设计解决了0.11.0.0之前版本中的两个问题：

- Broker保存消息后，发送ACK前宕机，Producer认为消息未发送成功并重试，造成数据重复
- 前一条消息发送失败，后一条消息发送成功，前一条消息重试后成功，造成数据乱序

但是producer挂掉重启后会发生变化。同时不同的partition也具有不同的主键，所有**无法保证跨分区跨会话**的exactly once。另外，它并**不能保证写操作的原子性**——即多个写操作，要么全部被Commit要么全部不被Commit。

kafka从0.11版本引入事务支持，来保证跨分区跨会话的exactly  once。

引入一个全局唯一的transactionID（重启不变）,将 pid和transactionID绑定。当producer重启后，通过transactionID来获取原来的PID。

另外，为了保证新的Producer启动后，旧的具有相同`Transaction ID`的Producer即失效，每次Producer通过`Transaction ID`拿到PID的同时，还会获取一个单调递增的epoch。由于旧的Producer的epoch比新Producer的epoch小，Kafka可以很容易识别出该Producer是老的Producer并拒绝其请求。

有了`Transaction ID`后，Kafka可保证：

- 跨Session的数据幂等发送。当具有相同`Transaction ID`的新的Producer实例被创建且工作时，旧的且拥有相同`Transaction ID`的Producer将不再工作。
- 跨Session的事务恢复。如果某个应用实例宕机，新的实例可以保证任何未完成的旧的事务要么Commit要么Abort，使得新实例从一个正常状态开始工作。

需要注意的是，上述的事务保证是从Producer的角度去考虑的。从Consumer的角度来看，该保证会相对弱一些。尤其是不能保证所有被某事务Commit过的所有消息都被一起消费，因为：

- 对于压缩的Topic而言，同一事务的某些消息可能被其它版本覆盖
- 事务包含的消息可能分布在多个Segment中（即使在同一个Partition内），当老的Segment被删除时，该事务的部分数据可能会丢失
- Consumer在一个事务内可能通过seek方法访问任意Offset的消息，从而可能丢失部分消息
- Consumer可能并不需要消费某一事务内的所有Partition，因此它将永远不会读取组成该事务的所有消息

![image-20200831213837692](C:\Users\lc\AppData\Roaming\Typora\typora-user-images\image-20200831213837692.png)