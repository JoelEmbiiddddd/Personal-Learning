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



#### Java NIO中，有三大组件，分别是Buffer、Channel、Selector



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















### 通用API

**ChannelActive：** 客户端连接后触发该函数。

**ChannelRead：** 客户端向服务端往通道缓存发送数据后触发该函数，服务端调用该API读取数据。

**writeAndFlush：** 发送ByteBuf字节码数据给客户端。



**Netty字节缓冲区对象ByteBuf常见API** 

readByte()：读取缓冲区当前读指针位置的内容，然后readerIndex指针+1 

readerIndex()：获取当前数据包的开头读位置Index 

readerIndex(num)：将读指针位置设置为num 





### 发送方式

基于protostuff传输对象数据：需要自己定义解码器和编码器，并在其中调用框架API实现字节码和对象之间的序列化和反序列化。 

channel.pipeline().addLast()：给当前bootstrap添加编码器，解码器，处理类。



### 粘包和半包

半包数据：客户端发送的一包数据到服务端，接收时拆分成多包 

粘包数据：收到的数据包中结束符号标志后面还带有其它的数据。 因此需要在服务端自定义编码解码器来处理以上问题。 



### 事件通知机制

Netty使用了事件通知机制，因此它的IO是异步的。

 而要实现Netty同步请求，这里的方案是通过countDownlatch和future配合实现，当前客户端线程通过writeAndFlush发送消息后，使用countdownlatch将该线程阻塞，然后服务端响应回复后唤醒客户端线程，通过future.get()获取响应数据。整个流程如下：

1. 客户端通过writeAndFlush向缓冲区发送请求数据，并添加监听器，确保客户端发送成功。
2. 然后客户端在writeFuture.get中线程阻塞，等待服务端响应数据，从而实现同步。(基于latch.await实现)此处给latch设置了超时时间，没有完全阻塞，超时后计数还没有为0则返回false。
3. 服务端通过channelRead收到请求数据，并将响应结果发送给客户端。
4. 客户端在channelRead中收到响应数据后，从Future缓存中取出对应的future对象，将响应数据封装进去，并countDown()唤醒前面的请求线程。





## 网络操作抽象类Channel

一旦客户端成功连接服务端，就会新建一个 Channel 同该用户端进行绑定。常用的`Channel`接口有 `NioServerSocketChannel服务端`和`NioSocketChannel客户端`。





## ChannelPipeline：消息处理器链表

ChannelHandler 是消息的具体处理器，主要负责处理客户端/服务端接收和发送的数据。ChannelPipeline 则是包含了一个或多个 ChannelHandler 的链表。

使用 Netty 的时候，我们通常就只要写一些自定义的 `handler` 就可以了，我们定义的这些 handler 会组成一个 pipeline，用于处理 IO 事件，这个和我们平时接触的 `Filter` 或 `Interceptor` 表达的差不多是一个意思。

每个 Channel 内部都有一个 pipeline，pipeline 由多个 handler 组成，handler 之间的顺序是很重要的，因为 IO 事

件将按照顺序顺次经过 pipeline 上的 handler，这样每个 handler 可以专注于做一点点小事，由多个 handler 组合来完成一些复杂的逻辑。

### ChannelInboundHandlerAdapter

ChannelInboundHandlerAdapter：拦截处理入栈事件，需要区分客户端和服务端。用户自定义处理方法时，需要重写以下事件函数： 

1. channelRead：信道缓冲区收到数据后，进行读数据。 

2. channelActive：服务端监听到客户端连接，信道激活。 

3. channelInactive：服务端监听到客户端断开连接。



## 异步操作：Future和Promise

