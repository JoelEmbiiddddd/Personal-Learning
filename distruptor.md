4、disruptor 不同等待策略；

5、为什么选择 disruptor；

6、介绍下 disruptor 的底层实现；

7、disruptor 的无锁体现在哪里；

8、disruptor 游标的技巧；

9、disruptor 如何解决 JDK 当时无法处理的伪共享；

5、介绍下 Disruptor，有哪些组件使用到了；

6、Disruptor 底层实现，为什么快；

3、disruptor 的使用场景；

3、介绍下 disruptor；

3、介绍下 disruptor；

7、介绍 disruptor 框架的特点；介绍下使用场景；



### Disruptor是什么？

Disruptor 是一个开源的高性能内存队列。Disruptor 提供的功能优点类似于 Kafka、RocketMQ 这类分布式队列，不过，其作为范围是 JVM(内存)。

它在无锁的情况下还能保证队列有界，并且还是线程安全的。



### Kafka 和 disruptor的区别是什么？

- **Kafka**：分布式消息队列，一般用在系统或者服务之间的消息传递，还可以被用作流式处理平台。
- **Disruptor**：内存级别的消息队列，一般用在系统内部中线程间的消息传递。



### Disruptor的等待策略都有哪些？

1. **BlockingWaitStrategy**：默认的策略，内部使用定时锁和condition来控制线程的唤醒。这个是最低效的策略，但是对CPU消耗最小，更能够提供一致性效果。
2. **SleepingWaitStrategy**：性能表现跟 **BlockingWaitStrategy**一样，但这种情况是对生产者最好。
3. **YieldingWaitStrategy**：YieldingWaitStrategy将自旋以等待序列增加到适当的值。在循环体内，将调用Thread.yield()以允许其他排队的线程运行。在要求极高性能且事件处理线数小于 CPU 逻辑核心数的场景中，推荐使用此策略。
4. **BusySpinWaitStrategy**：性能最好，适合用于低延迟的系统。在要求极高性能且事件处理线程数小于CPU逻辑核心数的场景中，推荐使用此策略；例如，CPU开启超线程的特性。



### 核心思想

- 环形数组：由于这个数组中的所有元素在初始化时一次性全部创建，因此这些元素的内存地址一般来说是连续的。这样做的好处是，当生产者不断往 RingBuffer 中插入新的事件对象时，这些事件对象的内存地址就能够保持连续，从而利用 CPU 缓存的局部性原理，将相邻的事件对象一起加载到缓存中，提高程序的性能。还可以避免频繁的内存分配和垃圾回收

- **「元素位置定位：」**数组长度2^n，通过位运算，加快定位的速度。下标采取递增的形式。不用担心index溢出的问题。index是long类型，即使100万QPS的处理速度，也需要30万年才能用完。

- **「无锁设计：」**每个生产者或者消费者线程，会先申请可以操作的元素在数组中的位置，申请到之后，直接在该位置写入或者读取数据，整个过程通过原子变量CAS，保证操作的线程安全

- **避免了伪共享问题**：CPU 缓存内部是按照 Cache Line（缓存行）管理的，一般的 Cache Line 大小在 64 字节左右。Disruptor 为了确保目标字段独占一个 Cache Line，会在目标字段前后增加了 64 个字节的填充（前 56 个字节和后 8 个字节），这样可以避免 Cache Line 的伪共享（False Sharing）问题。



### Disruptor的数据结构

框架使用**RingBuffer**来作为队列的数据结构，RingBuffer就是一个可自定义大小的环形数组。除数组外还有一个序列号(sequence)，用以指向下一个可用的元素，供生产者与消费者使用。



**Sequence**

mark：Disruptor通过顺序递增的序号来编号管理通过其进行交换的数据（事件），对数据(事件)的处理过程总是沿着序号逐个递增处理。

**「数组+序列号设计的优势是什么呢？」**

回顾一下HashMap，在知道索引(index)下标的情况下，存与取数组上的元素时间复杂度只有O(1)，而这个index我们可以通过序列号与数组的长度取模来计算得出，index=sequence % table.length。当然也可以用位运算来计算效率更高，此时table.length必须是2的幂次方。



### 使用场景

1. 经过测试，Disruptor的的延时和吞吐量都比ArrayBlockingQueue优秀很多，所以，当你在使用ArrayBlockingQueue出现性能瓶颈的时候，你就可以考虑采用Disruptor的代替。

2. Disruptor的最常用的场景就是“生产者-消费者”场景，对场景的就是“一个生产者、多个消费者”的场景，并且要求顺序处理。

   举个例子，我们从MySQL的BigLog文件中顺序读取数据，然后写入到ElasticSearch（搜索引擎）中。在这种场景下，BigLog要求一个文件一个生产者，那个是一个生产者。而写入到ElasticSearch，则严格要求顺序，否则会出现问题，所以通常意义上的多消费者线程无法解决该问题，如果通过加锁，则性能大打折扣

3. 停车场景。当汽车进入停车场时(A)，系统首先会记录汽车信息(B)。同时也会发送消息到其他系统处理相关业务(C)，最后发送短信通知车主收费开始(D)。在这个结构下，每个消费者拥有各自独立的事件序号Sequence，消费者之间不存在共享竞态。SequenceBarrier1监听RingBuffer的序号cursor，消费者B与C通过SequenceBarrier1等待可消费事件。SequenceBarrier2除了监听cursor，同时也监听B与C的序号Sequence，从而将最小的序号返回给消费者D，由此实现了D依赖B与C的逻辑。
