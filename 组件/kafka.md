[Kafka 连环20问 (qq.com)](https://mp.weixin.qq.com/s/DM3z8aM0CqAzWGwXEIzBhQ)

### 消息队列的作用是什么？

1. 通过异步处理提高系统性能。
2. 削峰/限流
3. 降低系统的耦合性



### AMQP是什么？

是一种提供统一消息服务的应用层标准的消息队列协议。基于此协议的客户端与消息中间件可传递消息，并不受客户端/中间件同产品，不同的开发语言等条件的限制



### 什么是死信队列？

死信可以看作消费者不能处理收到的消息，也可以看作消费者不想处理收到的消息，还可以看作不符合处理要求的消息。

- 消息内包含的消息内容无法被消费者解析，**为了确保消息的可靠性而不被随意丢弃**
- **超过既定的重试次数**之后将消息投入死信队列



### Kafka中如何实现死信队列

重试越多次重新投递的时间就越久，并且需要设置一个上限，超过投递次数就进入死信队列





### Kafka的两大应用场景

1. 消息队列：建立实时流数据通道。
2. 数据处理：可以构建实时的流数据处理程序来转换或处理数据流。



### 和其他的消息队列相对，kafka的优势在哪？

1. **极致的性能：**基于 Scala 和 Java 语言开发，设计中大量使用了批量处理和异步的思想，最高可以每秒处理千万级别的消息。
2. **生态系统兼容性好：** 客户端语言丰富：支持Java、.Net、PHP、Ruby、Python、Go等多种语言；



### 队列模型和发布-订阅模型的区别是什么

使用队列作为消息通信的载体，一条消息只能被一个消费者使用。



发布订阅模型使用的是 **主题**作为消息通信的载体，类似于广播模式，只有订阅的人才能收到。



### Kafka的概念

1. 生产者
2. 消费者
3. 代理（Broker）：可以看作是一个独立的kafka的实例。每个broker又包含有2个重要的概念
   - Topic：Producer发布主题，consumer订阅主题。
   - Partition：对应于消息队列中的队列。 一个Topic可以有多个Partition，一个Topic有多个broker。
4. Controller：Broker的领导者，每个Kafka集群任意时刻都只能有一个Controller。
   - 每个Broker上的分区副本信息。
   - 每个分区的Leader副本信息。



### Kafka的零拷贝

1. Kafka 把所有的消息都存放在单独的文件里，在消息投递时直接通过 `Sendfile` 方法发送文件，减少了上下文切换，因此大大提高了性能。

2. Producer生产的数据持久化到broker，采用mmap文件映射，实现顺序的快速写入。

   Customer从broker读取数据，采用sendfile，将磁盘文件读到OS内核缓冲区后，直接转到socket buffer进行网络发送。



### Kafka高性能

1. **顺序读写：**顺序读写不需要硬盘磁头的寻道时间，只需很少的扇区旋转时间，所以速度远快于随机读写
2. **零拷贝：** 分别用了sendfile和mmap。
3. **批量发送读取：**它在消息投递时会将消息缓存起来，然后批量发送。消费端在消费消息时，也不是一条一条处理的，而是批量进行拉取，提高了消息的处理速度。
4. **数据压缩：**Kafka还支持对消息集合进行压缩，`Producer`可以通过`GZIP`或`Snappy`格式对消息集合进行压缩压缩的好处就是减少传输的数据量，减轻对网络传输的压力。
5. **分区机制：** kafka中的topic中的内容可以被分为多partition存在，每个partition又分为多个段segment，所以每次操作都是针对一小部分做操作，很轻便，并且增加`并行操作`的能力



### kafka的高可用性

1. **备份机制：**Kafka会尽量将所有的Partition以及各Partition的副本均匀地分配到整个集群的各个Broker上。
2. **ISR机制：** ISR中所有副本都跟上了Leader，通常只有ISR里的成员才可能被选为Leader。
3. **ACK机制：**producer的消息发送确认机制，ack=0，ack=1，ack=all，其中ack=all的可用性是最高的。
4. **故障恢复机制：**首先需要在集群所有Broker中选出一个Controller，负责各Partition的Leader选举以及Replica的重新分配。当出现Leader故障后，Controller会将Leader/Follower的变动通知到需为此作出响应的Broker。







### Kafka的多副本机制了解吗

分区（Partition）中的多个副本之间会有一个叫做 leader 的家伙，其他副本称为 follower。我们发送的消息会被发送到 leader 副本，然后 follower 副本才能从 leader 副本中拉取消息进行同步。

生产者和消费者只与Leader副本交互。它们的存在只是为了保证消息存储的安全性。



**存在的好处**

1. Kafka 通过给特定 Topic 指定多个 Partition, 而各个 Partition 可以分布在不同的 Broker 上, 这样便能提供比较好的并发能力。
2. Partition 可以指定对应的 Replica 数, 这也极大地提高了消息存储的安全性, 提高了容灾能力。



### Zookeeper在Kakfa中的作用是什么？

1. **Broker的注册：**有一个节点专门用来进行Broker服务器记录的节点。当Broker启动后，都会到zookeeper上注册。
2. **Topic注册：**在 Kafka 中，同一个**Topic 的消息会被分成多个分区**并将其分布在多个 Broker 上，**这些分区信息及与 Broker 的对应关系**也都是由 Zookeeper 在维护。
3. **负载均衡**



### Kafka如何保证消息的消费顺序

每次添加消息到 Partition(分区) 的时候都会采用尾加法，即使用一个偏移量。 **Kafka 只能为我们保证 Partition(分区) 中的消息有序。**因此，我们可以用：

1. 1个 Topic 只对应一个 Partition。
2. （推荐）发送消息的时候指定 key/Partition。



### Kafka如何保证消息不丢失



#### 生产者丢失消息的情况

发送`send`后，因为网络问题没有发送过去。可以使用生产者的重试机制。



####  消费者丢失消息的情况

当消费者拉取到了某个分区的某个消息后，消费者会自动提交offset，但这里会出问题，就是拿到后消费者就会挂掉。那么就可以关闭自动提交offset，每次真正消费完消息后再手动提交offset。



#### Kafka弄丢了消息

kafka对于这个问题有3个参数

1. **acks**：默认值为1，表示如果leader副本接收后就算成功发送。 如果配置acks=all表示所有ISR列表的副本全部收到消息后，生产者才会接收到来自服务器的响应.。

2. **设置 replication.factor >= 3**：保证每个 分区(partition) 至少有 3 个副本

   



### 如何保证消息不重复消费

**kafka 出现消息重复消费的原因：**

1. 服务端侧已经消费的数据没有成功提交 offset（根本原因）。
2. Kafka 侧 由于服务端处理业务时间长或者网络链接等等原因让 Kafka 认为服务假死，触发了分区 rebalance。



**解决办法：**

1. 最佳方法：消费消息服务做幂等校验，比如Redis的Set，MySQL的主键等天然的幂等功能。
2. 关闭自动提交offset功能。那么什么时候开启是最好的？
   - 处理完消息再提交：依旧有消息重复消费的风险，和自动提交一样
   - 拉取到消息即提交：会有消息丢失的风险。允许消息延时的场景，一般会采用这种方式。



### Kafka的副本机制

在Kafka中，**追随者副本是不对外提供服务的。**这就是说，任何一个追随者副本都不能响应消费者和生产者的读写请求。所有的请求都必须由领导者副本来处理，或者说，所有的读写请求都必须发往领导者副本所在的Broker，由该Broker负责处理。

追随者副本不处理客户端请求，它唯一的任务就是从领导者副本**「异步拉取」**消息，并写入到自己的提交日志中，从而实现与领导者副本的同步。

当领导者副本挂掉了，或者说领导者副本所在的Broker宕机时，Kafka依托于ZooKeeper提供的监控功能能够实时感知到，并立即开启新一轮的领导者选举，从追随者副本中选一个作为新的领导者。老Leader副本重启回来后，只能作为追随者副本加入到集群中。

这样子的好处**方便实现单调读**





###  kafka的主从机制描述下（ISR机制）

- **ISR(In-Sync Replication):** 一个 Partition 的中所有能够正常通信的 Partitions组成 ISR
- **OSR：** 一个Partition中的所有不能够正常通信的Partitions组成的OSR。
- AR(Assigned Replication): 所有 partition的合集称为 AR。ISR 是 AR 的一个子集。



Leader维护ISR列表，Follower从leader同步数据有一定些延迟，超过阈值，这个Follower就会被踢出ISR，加入OSR列表中。此外新加入的Follower也会先加入到OSR中。倘若该副本后面慢慢地追上了Leader的进度，那么它是能够重新被加回ISR的。

每个relica（leader和follower）都有HW，leader和follower各自负责更新自己的HW的状态。**对于leader新写入的消息，只有等所有ISR中的Follower都同步后，才能被消费者给消费。**这样就保证了如果 leader 所在的 broker 失效，该消息仍然可以从新选举的 leader 中获取。对于来自内部 broker 的读取请求，没有 HW 的限制。

由此可得：Kafka 的复制机制既不是完全的同步复制，也不是单纯的异步复制



### Leader选举策略

只有 ISR 中的 Partition 才能被选举成 Leader, 不使用多数派机制，只要有一个 Follower 存在就可以被选举为 Leader。当所有的 Follower 都失败时，默认会选举第一个恢复的 Partition作为新的 Leader partition。在选举时不会参考各Partition中 LEO 和 HW 的位置。



## Kafka的高可用性

消息备份机制、ISR、消息应答确认机制、LEO和HW



### 消息备份机制

Kafka的每个分区（Partition）都有一个副本集合AR，每个副本集合包含一个Leader副本，及0个以上的Follower副本。生产者将消息发送对应Partition的Leader副本，Follower副本从Leader副本同步消息，Kafka的Leader机制在保障数据一致性的同时，也降低了消息备份的复杂度。
同一个Partition的副本不会存储在同一个Broker上，Kafka会尽量将所有的Partition以及其各个副本均匀地分配在整个集群中，这样既做好了负载均衡，又提高了容错能力。



### ISR

在消息同步期间，Follower副本相对于Leader副本具有一定程度的滞后，Leader副本负责维护和跟踪ISR集合中所有Follower副本的滞后状态，并将ISR的变更同步至ZooKeeper。当Follower副本落后太多或失效时，Leader副本会将它从ISR集合中剔除，被移除ISR的Follower可以继续发送FetchRequest请求，尝试再次跟上Leader并重新进入ISR。追赶上Leader副本的判定标准是，此Follower副本的LEO不小于Leader副本的HW。


通常只有在ISR集合中的副本才有资格被选举为新的Leader。



### 消息应答确认机制

ack=-1/all ，就要所有ISR中的副本都写入才行



### LEO和HW

在消息的追加过程，提到了日志偏移量，Kafka的每个副本对象有两个重要的偏移量属性：

- LEO（log end offset），即日志末端位移，指向副本日志中下一条消息的偏移量（即下一条消息的写入位置）
- HW（High Watermark），高水位线，指已同步ISR中Follower副本的偏移量标识

所有高水位线以下的消息都是备份过的，消费者仅可以消费各个分区Leader高水位线以下的消息，所以Leader的HW值是由ISR中所有备份的LEO最小值决定的




## Kafka分区Leader副本选举

同一个分区，同一个Broker节点中不允许出现多个副本，当分区的Leader节点发生功能故障时，其中一个Follower节点就会成为新的Leader节点。
