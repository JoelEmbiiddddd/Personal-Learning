**不错的参考资料：**[聊聊Netty那些事儿之从内核角度看IO模型 (qq.com)](https://mp.weixin.qq.com/s/zAh1yD5IfwuoYdrZ1tGf5Q)



## IO模型



### BIO模型

服务阻塞点①： ServerSocket通过调用 accept( )方法，阻塞并返回一个 java.net.Socket 对象。只有当前客户端写完数据close后，服务端才会对下一个客户端进行处理。 

服务阻塞点②：BufferedReader#readLine方法是阻塞的，读取一行时只有碰到换行、回车、buffer溢出或者是客户端线程直接close关闭资源，最终才会返回null。 如果直接使用InputStream#read方法也会一直阻塞等待，除非客户端发来消息。



### NIO模型

Selector.select()：阻塞直到至少有一个通道在已注册的事件上就绪(设置为非阻塞的方式①可以指定阻塞时间，超过后不管是否就绪直接返回②通过wakeup方法直接唤醒selector)。

服务端单线程通过选择器Selector监测处理Channel多个通道，监听四种事件：连接就绪(客户端事件)、接受就绪(服务端)、读就绪(服务端读取客户端通道发送数据)、写就绪。每次事件读取完成后，都需要把事件剔除，防止下次重复读取事件。

ServerSocketChannel ：服务端通道，监听的作用。调用 accept( )监听客户端连接的通道(非阻塞模式，accept拿不到连接则返回null)。 

SocketChannel ：用于客户端与服务端之间相互读写数据。



#### Java NIO中的三大组件



Buffer、Channel、Selector



**Buffer**

一个 Buffer 本质上是内存中的一块，我们可以将数据写入这块内存，之后从这块内存获取数据。

Java中定义了几个Buffer的实现，用的最多的就是ByteBuffer。

Buffer有几个参数：capacity、limit、position

在写操作下，每往里边添加一个字符，position就+1，limit=capacity

在读操作下，则position从0开始记，limit则代表写入的长度。



**Channel**

所有的 NIO 操作始于通道，通道是数据来源或数据写入的目的地，Java NIO中提供了4种通道，分别是FileChannel(文件通道)、DataGramChannel（用于UDP连接）、SocketChannel（TCP客户端连接）、ServerSocketChannel（TCP服务端，用于监听某个端口传进来的请求）。

类似 IO 中的流，用于读取和写入，读操作的时候将 Channel 中的数据填充到 Buffer 中，而写操作时将 Buffer 中的数据写入到 Channel 中。



**Selector**

多路复用实现的方法，在Java NIO中，一般涉及到

1. 开启Selector。
2. 将Channel注册到Selector中，注册的参数有4种二进制方式，有通道可以读取，可以往通道写入数据，成功建立TCP连接，接受TCP连接。我们可以同时监听一个 Channel 中的发生的多个事件，比如我们要监听 ACCEPT 和 READ 事件，那么指定参数为二进制的 000**1**000**1** 即十进制数值 17 即可。

​	3. 调用 select() 方法获取通道信息。用于判断是否有我们感兴趣的事件已经发生了。



# Netty

`Netty`是一个异步的、基于事件驱动的网络应用框架，用以快速开发高性能、高可靠性的网络`IO`程序。 主要针对在`TCP`协议下，面向`Client`端的高并发应用，或者`Peer-to-Peer`场景下的大量数据持续传输的应用。`Netty`本质是一个`NIO`框架，适用于服务器通讯相关的多种应用场景。

1. 异步的
2. 事件驱动的
3. 针对TCP协议下，面向Client的
4. 或者是peer2peer的大量数据持续传输的应用
5. NIO框架



需要掌握的：

1. Netty的流程和各个组件是如何进行工作的。
2. Netty高效的原因
   - Reactor线程模型
   - 内存管理
   - 零拷贝
3. 如何自定义协议通信





## 原生Java NIO存在的问题

1. 原有的NIO库很复杂，开展使用很不方便。
2. 需要使用者自己去写一个工具包，开发难度很大。比如说粘包处理、断连重连，失败缓存，网络拥塞。
3. `JDK NIO`的`Bug`：例如臭名昭著的`Epoll Bug`，它会导致`Selector`空轮询，最终导致`CPU100%`。



## Netty模型

Netty基于Reactor多线程模型进行了一定的改进，其中主从Reactor中都有多个Reactor，可以大幅提高Netty的吞吐量。

1. 增加了BossGroup来维护多个主Reactor，主Reactor还是只关注连接的Accept；增加了WorkGroup来维护多个从Reactor，从Reactor将接收到的请求交给Handler进行处理。
2. 在主Reactor中接收到Accept事件，获取到对应的SocketChannel，Netty会将它进一步封装成NIOSocketChannel对象，，这个对象里边包含有所对象的信息。
3. 之后Netty就会将这个封装后的对象注册到WorkerGroup中的从Reactor中。
4. 当WorkerGroup中的从Reactor监听到事件后，就会将之交给与此Reactor对应的Handler进行处理。



### 具体流程

1. Netty会抽象出2个线程池，BossGroup专门用来接收客户端的连接，WorkerGroup专门负责网络的读写。
2. BossGroup和WorkerGroup的类型都是NioEventLoopGroup类型。
3. NioEventLoop相当于一个事件循环组，这个组里边包含有多个事件循环，每个事件循环都是NioEventLoop。
4. 每个NioEventLoop相当于是一个不断循环处理任务的线程，每个NioEventLoop都有一个selector。
5. 每个BossNioEventLoop循环三件事
   - 轮询Accept事件
   - 处理Accept事件，与cilent建立连接，生成NioSocketChannel，并将其注册到某个worker的NioEventLoop的selector上。
   - 处理任务队列
6. 每个Worker NioEventLoop循环三件事：
   - 轮询 NioSocketChannel是否有read，write事件。
   - 处理事件
   - 处理任务队列中的任务。
7. 在Worker NioEventLoop处理业务的时候，会使用pipeline，pipeline中存在有多个ChannelHandler，每个ChannelHandler都保存了这个ChannelHandlerContext的所有上下文信息。



### 对应关系

1. `Netty`抽象出两组线程池，`BossGroup`专门负责接收客户端连接，`WorkerGroup`专门负责网络读写操作。
2. `NioEventLoop`表示一个不断循环执行处理任务的线程，每个`NioEventLoop`都有一个`Selector`，用于监听绑定在其上的`socket`网络通道。
3. `NioEventLoop`内部采用串行化设计，从消息的**读取->解码->处理->编码->发送**，始终由`IO`线程`NioEventLoop`负责

NioEventLoopGroup`下包含多个`NioEventLoop

- 每个`NioEventLoop`中包含有一个`Selector`，一个`taskQueue`
- 每个`NioEventLoop`的`Selector`上可以注册监听多个`NioChannel`
- 每个`NioChannel`只会绑定在唯一的`NioEventLoop`上
- 每个`NioChannel`都绑定有一个自己的`ChannelPipeline`



## 异步模型

Netty`中的`I/O`操作是异步的，包括`Bind、Write、Connect`等操作会简单的返回一个`ChannelFuture。

调用者并不能立刻获得结果，实际Netty是使用了Future-listener机制，用户可以方便的主动获取或者通过通知机制获得`IO`操作结果。



## 主要组件



### Bootstrap，ServerBootStrap

一个`Netty`应用通常由一个`Bootstrap`开始，主要作用是配置整个`Netty`程序，串联各个组件，`Netty`中`Bootstrap`类是客户端程序的启动引导类，`ServerBootstrap`是服务端启动引导类。



### Future、ChannelFuture

`Netty`中所有的`IO`操作都是异步的，不能立刻得知消息是否被正确处理。但是可以过一会等它执行完成或者直接注册一个监听，具体的实现就是通过`Future`和`ChannelFutures`，他们可以注册一个监听，当操作执行成功或失败时监听会自动触发注册的监听事件



### Channel

1. `Netty`网络通信的组件，能够用于执行网络`I/O`操作。
2. `Channel`提供异步的网络`I/O`操作(如建立连接，读写，绑定端口)，异步调用意味着任何`I/O`调用都将立即返回，并且不保证在调用结束时所请求的`I/O`操作已完成。
3. 调用立即返回一个`ChannelFuture`实例，通过注册监听器到`ChannelFuture`上，可以`I/O`操作成功、失败或取消时回调通知调用方。



### Selector

1. `Netty`基于`Selector`对象实现`I/O`多路复用，通过`Selector`一个线程可以监听多个连接的`Channel`事件。
2. 当向一个`Selector`中注册`Channel`后，`Selector`内部的机制就可以自动不断地查询(`Select`)这些注册的`Channel`是否有已就绪的`I/O`事件(例如可读，可写，网络连接完成等)，这样程序就可以很简单地使用一个线程高效地管理多个`Channel`



### ChannelHandler

1. `ChannelHandler`是一个接口，处理`I/O`事件或拦截`I/O`操作，并将其转发到其`ChannelPipeline`(业务处理链)中的下一个处理程序。
2. 我们经常需要自定义一个`Handler`类去继承`ChannelInboundHandlerAdapter`，然后通过重写相应方法实现业务逻辑。



### Pipeline和ChannelPipeline

1. ChannelPipeline是一个Handler的集合，它负责处理和拦截`inbound`或者`outbound`的事件和操作，相当于一个贯穿`Netty`的链。
2. `hannelPipeline`实现了一种高级形式的拦截过滤器模式，使用户可以完全控制事件的处理方式，以及`Channel`中各个的`ChannelHandler`如何相互交互
3. 在`Netty`中每个`Channel`都有且仅有一个`ChannelPipeline`与之对应



### ChannelHandlerContext

1. 保存`Channel`相关的所有上下文信息，同时关联一个`ChannelHandler`对象
2. 即`ChannelHandlerContext`中包含一个具体的事件处理器`ChannelHandler`，同时`ChannelHandlerContext`中也绑定了对应的`pipeline`和`Channel`的信息，方便对`ChannelHandler`进行调用。



### EventLoopGroup、EventLoop、NioEventLoopGroup

1. EventLoopGroup管理着多个EventLoop，每个EventLoop维护着一个Selector实例。
2. 常一个服务端口即一个`ServerSocketChannel`对应一个`Selector`和一个`EventLoop`线程。`BossEventLoop`负责接收客户端的连接并将`SocketChannel`交给`WorkerEventLoopGroup`来进行`IO`处理



### 编解码器

**编码器：** 将数据转化为二进制

**解码器：** 将二进制转化为数据

`Netty`的组件设计：`Netty`的主要组件有`Channel`、`EventLoop`、`ChannelFuture`、`ChannelHandler`、`ChannelPipe`等

`ChannelHandler`可以充当入站和出站数据的应用程序逻辑的容器，例如，实现ChannelInboundHandler接口或ChannelOutBoundHandler接口，就可以接收入站事件和数据，这些数据会被业务逻辑处理。



1. 当`Netty`发送或者接受一个消息的时候，就将会发生一次数据转换。入站消息会被解码：从字节转换为另一种格式(比如`java`对象)；如果是出站消息，它会被编码成字节。
2. Netty提供了一系列实用的编解码器看，全都实现了ChannelInboundHandler或者是ChannelOutboundHandler接口。

**常见的编码器有**ByteToMessageDecoder

**常见的解码器有**ReplayingDecoder



在我们的Gateway网关中，所使用的是`HttpObjectDecoder`：一个`HTTP`数据的解码器，和`HttpObjectEncoder`：一个Http数据的编码器

