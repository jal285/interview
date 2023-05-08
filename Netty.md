---

---

# Netty

<!--more-->

1. Netty基于NIO(NIO 是一种同步非阻塞的I/O模型 ) 实现的异步通信框架. 使用Netty 可以极大地简化并简化了 TCP 和 UDP 套接字服务器等网络编程,并且性能以及安全性等很多方面都非常优秀。
2. 我们平常经常接触的 Dubbo、RocketMQ、Elasticsearch、gRPC、Spark、Elasticsearch 等等热门开源项目都用到了 Netty。
3. 大部分微服务框架底层涉及到网络通信的部分都是基于 Netty 来做的，比如说 Spring Cloud 生态系统中的网关 Spring Cloud Gateway。
4. Netty是异步事件驱动的框架，网络IO模型采用的是NIO。异步事件驱动框架体现在所有的I/O操作是异步的， 所有的`IO`调用会立即返回 ，并不保证调用成功与否，但是调用会返回`ChannelFuture`，`netty`会通过`ChannelFuture`通知你调用是成功了还是失败了亦或是取消了。
5. Netty的异步事件驱动基于底层的NIO实现

## 同步和异步

从操作系统角度来说，网络IO的数据拷贝主要分为两个阶段，一是数据准备阶段，二是数据从内核（网卡）拷贝到用户中。

同步IO指的是数据从内核拷贝到用户时。发起该请求的线程会自己来拷贝数据（表现为线程阻塞拷贝）。

PS：一旦涉及到网络 IO必定会发生数据拷贝的阻塞（此阻塞非彼阻塞，这里的阻塞形容的是拷贝数据相对于CPU的速度来说是非常耗时的，看起来像线程阻塞了一样。我们常说的阻塞是线程挂起并让出CPU），只不过阻塞发生在其他的地方（自己来拷贝就是同步的，别人帮我拷贝就是异步的），因为IO必定会用到CPU，即使是零拷贝。

## 阻塞和非阻塞

阻塞和非阻塞主要描述的是网络IO数据拷贝的第一个阶段。

阻塞指的是线程一直等待数据准备好，期间什么都不干，但是会让出CPU，这样其他线程可以执行（CPU的利用率比较高），数据准备好之后自己来拷贝数据。

非阻塞指的是在第一阶段，发起网络IO请求的时候会立即返回去干别的事情，但是会不断地进行询问数据是否准备好，这种方式称之为轮询。由于CPU要处理更多的系统调用（每次询问都是系统调用），这种模型的CPU利用率低。

同步IO包括阻塞IO、非阻塞IO和IO多路复用

### 为什么选用非阻塞（NIO）

在多线程时一般使用线程池，线程的创建和回收成本较低。 在大量线程的情况下，使用BIO的话会容易造成大量请求的结果同时返回， 激活大量的阻塞线程使系统负载压力过大而崩溃，而使用NIO的话就可以根据数据准备情况来返回数据给用户 

## Netty中的NIO 与Netty的异步事件有什么关系

没有关系 

Netty是异步事件驱动的框架，网络IO模型采用的是`JAVA NIO`（同步非阻塞IO）。异步事件驱动框架体现在所有的I/O操作是异步的，所有的IO调用会立即返回，并不保证调用成功与否，但是调用会返回ChannelFuture，netty会通过ChannelFuture通知你调用是成功了还是失败了亦或是取消了。两个或两个以上的对象或事件不同时存在或发生(或多个相关事物的发生无需等待其前一事物的完成)

所以所谓的异步是针对用户而言的，用户使用Channel进行IO操作，会立即返回。但是这个IO操作的任务是提交给了Netty的NIO底层去进行处理，所以我们说Netty的异步事件驱动与Netty底层基于NIO（同步非阻塞）是不矛盾的。

## 是什么

1. Netty 是一个 **基于 NIO** 的 client-server(客户端服务器)框架，使用它可以快速简单地开发网络应用程序。
2. 它极大地简化并优化了 TCP 和 UDP 套接字服务器等网络编程,并且性能以及安全性等很多方面甚至都要更好。
3. **支持多种协议** 如 FTP，SMTP，HTTP 以及各种二进制和基于文本的传统协议。

官方总结; Netty 成功地找到了一种在不妥协可维护性`和`性能的情况下实现易于开发,性能, 稳定性和灵活性的方法 

## 不直接使用NIO 的原因

### NIO 原理

<img src="../images/Netty/image-20210620173647171.png" alt="image-20210620173647171" style="zoom: 50%;" />

结合上面的接口交互图可知，接口 B 通过网络返回数据给调用方(接口 A)这一过程，对应底层实现就是网卡接收到返回数据后，通过自身的 DMA（直接内存访问）将数据拷贝到内核缓冲区，这一步不需要 CPU 参与操作，也就是把原先 CPU 等待的事情交给了底层网卡去处理，这样 **CPU 就可以专注于我们的应用程序即接口内部的逻辑运算**。

<img src="../images/Netty/image-20210620173730332.png" alt="image-20210620173730332" style="zoom: 50%;" />

NIO 的编程模型复杂而且存在一些BUG, 并且对编程功底要求比较高, 下面是一个典型的使用NIO 进行编程的案例 

![img](../images/Nettymd/c9b2752b36c3e1399ab8f09d82b066e6.png)

NIO实现不完整,也具有实现比较复杂的原因, 所以Netty采用NIO作为框架技术支持

### NIO的一个特性

Netty的传输快其实也是依赖了NIO的一个特性——零拷贝

一般我们的数据如果需要从IO读取到堆内存，中间需要经过Socket缓冲区，也就是说一个数据会被拷贝两次才能到达他的的终点，如果数据量大，就会造成不必要的资源浪费。
 Netty针对这种情况，使用了NIO中的另一大特性——零拷贝，当他需要接收数据的时候，他会在堆内存之外开辟一块内存，数据就直接从IO读到了那块内存中去，在netty里面通过ByteBuf可以直接对这些数据进行直接操作，从而加快了传输速度。

## 传统的阻塞式通信流程(BIO)

**早期的 Java 网络相关的 API(`java.net`包) 使用 Socket(套接字)进行网络通信，不过只支持阻塞函数使用。**

要通过互联网进行通信，至少需要一对套接字：

1. 运行于服务器端的 Server Socket。
2. 运行于客户机端的 Client Socket

Socket 网络通信过程如下图所示：

<img src="../images/Netty/image-20210612165901610.png" alt="image-20210612165901610" style="zoom: 80%;" />

**Socket 网络通信过程简单来说分为下面 4 步：**

1. 建立服务端并且监听客户端请求
2. 客户端请求，服务端和客户端建立连接
3. 两端之间可以传递数据
4. 关闭资源

对应到服务端和客户端的话，是下面这样的。

**服务器端：**

1. 创建 `ServerSocket` 对象并且绑定地址（ip）和端口号(port)：` server.bind(new InetSocketAddress(host, port))`
2. 通过 `accept()`方法监听客户端请求
3. 连接建立后，通过输入流读取客户端发送的请求信息
4. 通过输出流向客户端发送响应信息
5. 关闭相关资源

**客户端：**

1. 创建`Socket` 对象并且连接指定的服务器的地址（ip）和端口号(port)：`socket.connect(inetSocketAddress)`
2. 连接建立后，通过输出流向服务器端发送请求信息
3. 通过输入流获取服务器响应的信息
4. 关闭相关资源

## 为什么用Netty

Netty具有以下有点, 而且相比直接使用JDK自带的NIO相关的API 来说更加易用

- 同一的API , 支持多种传输类型, 阻塞和非阻塞的 . 
- 简单而强大的线程模型
- 自带编码解码器解决TCP粘包/拆包问题
- 自带各种协议栈
- 真正的无连接数据包套接字支持
- 比直接使用Java核心API 有更高的吞吐量、更低的延迟、更低的资源消耗和更少的内存复制
- 安全性不错， 有完整的SSL/TLS以及StartTLS 支持
- 社区活跃
- 成熟稳定，经历了大型项目的使用和考验，而且很多开源项目都使用到了Netty，比如Dubbo、RocketMQ 

## 应用场景

NIO可以做的事情， 使用Netty都可以做并且做的更好 

Netty 主要用来做网络通信： 

1. **作为 RPC 框架的网络通信工具** ： 我们在分布式系统中，不同服务节点之间经常需要相互调用，这个时候就需要 RPC 框架了。不同服务节点的通信是如何做的呢？可以使用 Netty 来做。比如我调用另外一个节点的方法的话，至少是要让对方知道我调用的是哪个类中的哪个方法以及相关参数吧！
2. **实现一个自己的 HTTP 服务器** ：通过 Netty 我们可以自己实现一个简单的 HTTP 服务器，这个大家应该不陌生。说到 HTTP 服务器的话，作为 Java 后端开发，我们一般使用 Tomcat 比较多。一个最基本的 HTTP 服务器可要以处理常见的 HTTP Method 的请求，比如 POST 请求、GET 请求等等。
3. **实现一个即时通讯系统** ： 使用 Netty 我们可以实现一个可以聊天类似微信的即时通讯系统，这方面的开源项目还蛮多的，可以自行去 Github 找一找。
4. **实现消息推送系统** ：市面上有很多消息推送系统都是基于 Netty 来做的。
5. ......

## 那些开源项目用到了Netty

实际上有很多很多优秀的项目用到了 Netty,Netty 官方也做了统计，统计结果在这里：https://netty.io/wiki/related-projects.html 。

## 介绍一下Netty的核心组件

![img](../images/Nettymd/0c8871a17950bfbc7bd351fd8d3a2340.png)

### Bytebuf （字节容器）

**网络通信最终都是通过字节流进行传输的。 ByteBuf 就是Netty提供的一个字节容器， 其内部是一个字节数组。** 当我们通过Netty传输数据时， 就是通过ByteBuf 进行的 

- ByteBuf是一个存储字节的容器，最大特点就是**使用方便**，它既有自己的读索引和写索引，方便你对整段字节缓存进行读写，也支持get/set，方便你对其中每一个字节进行读写，他的数据结构如下图所示：

![img](https:////upload-images.jianshu.io/upload_images/1089449-b1ec677f253b692a.png?imageMogr2/auto-orient/strip|imageView2/2/w/558/format/webp)

ByteBuf数据结构

我们可以将ByteBuf看作是Netty对Java NIO 提供了ByteByffer 字节容器的封装和抽象 

ByteBuffer 这个类使用起来过于复杂和繁琐 ,所以使用ByteBuf

他有三种使用模式：

1. Heap Buffer 堆缓冲区
     堆缓冲区是ByteBuf最常用的模式，他将数据存储在堆空间。

2. Direct Buffer 直接缓冲区
   
    直接缓冲区是ByteBuf的另外一种常用模式，他的内存分配都不发生在堆，jdk1.4引入的nio的ByteBuffer`类`允许jvm通过本地方法调用分配内存，这样做有两个好处
   
   - 通过免去中间交换的内存拷贝, 提升IO处理速度; 直接缓冲区的内容可以驻留在垃圾回收扫描的堆区以外。
   - DirectBuffer 在 -XX:MaxDirectMemorySize=xxM大小限制下, 使用 Heap 之外的内存, GC对此”无能为力”,也就意味着规避了在高负载下频繁的GC过程对应用线程的中断影响.

3. Composite Buffer 复合缓冲区
     复合缓冲区相当于多个不同ByteBuf的视图，这是netty提供的，jdk不提供这样的功能。

除此之外，他还提供一大堆api方便你使用，在这里我就不一一列出了，具体参见[ByteBuf字节缓存](https://links.jianshu.com/go?to=https%3A%2F%2Fwaylau.gitbooks.io%2Fessential-netty-in-action%2Fcontent%2FCORE%20FUNCTIONS%2FBuffers.html)。

- Codec
     Netty中的编码/解码器，通过他你能完成字节与pojo、pojo与pojo的相互转换，从而达到自定义协议的目的。
     在Netty里面最有名的就是HttpRequestDecoder和HttpResponseEncoder了。

### Bootstarp 和ServerBootstrap (启动引导类)

Bootstrap 是客户端的启动引导类/辅助类 ， 

```java
        EventLoopGroup group = new NioEventLoopGroup();
        try {
            //创建客户端启动引导/辅助类：Bootstrap
            Bootstrap b = new Bootstrap();
            //指定线程模型
            b.group(group).
                    ......
            // 尝试建立连接
            ChannelFuture f = b.connect(host, port).sync();
            f.channel().closeFuture().sync();
        } finally {
            // 优雅关闭相关线程组资源
            group.shutdownGracefully();
        }
```

ServerBootStrap 客户端的启动引导类/辅助类 ，具体使用方法如下

```java
  // 1.bossGroup 用于接收连接，workerGroup 用于具体的处理
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            //2.创建服务端启动引导/辅助类：ServerBootstrap
            ServerBootstrap b = new ServerBootstrap();
            //3.给引导类配置两大线程组,确定了线程模型
            b.group(bossGroup, workerGroup).
                   ......
            // 6.绑定端口
            ChannelFuture f = b.bind(port).sync();
            // 等待连接关闭
            f.channel().closeFuture().sync();
        } finally {
            //7.优雅关闭相关线程组资源
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
```

1.Bootstrap 通常使用 connect() 方法远程连接到远程的主机和端口, 作为一个Netty TCP 协议通信中的客户端, 另外, Bootstrap 也可以通过bind()方法绑定本地的要给端口, 作为 UDP 协议通信种的一端

2.ServerBootstrap 通常使用bind() 方法绑定本地的端口上, 然后等待客户端的连接

3. Bootstrap 只需要配置一个线程组 一 EventLoopGroup , 而 ServerBootstrap 需要配置两个线程组 -EventLoopGroup , 一个用于接受连接, 一个用于具体的IO 处理 

## Channel (网络操作抽象类)

channel是访问I/O服务的导管. I/O 可以分为广义的两大类别: File I/O和Stream I/O  

相应地有两种类型的通道  文件(file)和套接字(socket) 通道 

Channel接口时Netty对网络操作抽象类. 通过Channel 我们可以进行I/O操作 

客户端成功连接服务端后就会成功新建一个channel 同该用户端进行绑定 

```java
 public Channel doConnect(InetSocketAddress inetSocketAddress) throws ExecutionException, InterruptedException {
        CompletableFuture<Channel> completableFuture = new CompletableFuture<>();
        Bootstrap bootstrap = new Bootstrap();
        bootstrap.connect(inetSocketAddress).addListener((ChannelFutureListener)future->{
            if(future.isSuccess()){
                completableFuture.complete(future.channel());
            }else{
                throw  new IllegalStateException();
            }
        });
        return completableFuture.get();
    }
```

比较常用的Channel 接口实现类是 : 

- `NioServerSocketChannel`（服务端）
- `NioSocketChannel`（客户端）

这两个Channel 可以和BIO 编程模型中的ServerSocket 以及Socket 两个概念对应上

### EventLoop(事件循环)

EventLoop  接口可以说是Netty中最核心的概念了 

《Netty 实战》这本书是这样介绍它的：

> `EventLoop` 定义了 Netty 的核心抽象，用于处理连接的生命周期中所发生的事件。

EventLoop的主要作用实际上就是负责监听网络事件并调用事件处理器进行相关I/O操作《读写》的处理 

### Channel 和 EventLoop 的关系

Channel 为Netty网络操作(读写等操作) 抽象类, EventLoop 负责处理注册到其上的Channel 的I/O 操作, 两者配合进行I/O操作

## 线程模型

[参考](https://zhuanlan.zhihu.com/p/92193290)

Java的NIO采用的是Reactor 线程模型中的单Reactor单线程模型（前台和服务员是一个人，全程为顾客服务，可以服务多个人） 

Netty的NIO 采用的是主从Reacto模型、是Reactor多线程模型 

### 三种线程模型

常见的Reactor线程模型有以下三种

1. Reactor单线程模型；
2. Reactor多线程模型；
3. 主从Reactor多线程模型；

管理线程池 

#### 1、Reactor单线程模型

Reactor单线程模型，指的是所有的I/O操作都在同一个NIO线程上面完成，NIO线程的职责如下：

1. 作为NIO服务端，接收客户端的TCP连接；
2. 作为NIO客户端，向服务端发起TCP连接；
3. 读取通信对端的请求或者应答消息；
4. 向通信对端发送消息请求或者应答消息；

Reactor线程是个多面手，负责多路分离套接字，Accept新连接，并分派请求到处理器链中。该模型 适用于处理器链中业务处理组件能快速完成的场景。不过，这种单线程模型不能充分利用多核资源，所以实际使用的不多。

![img](../images/Netty/285763-20180123121112287-1483895090.png)

对于一些小容量应用场景，可以使用单线程模型，但是对于高负载、大并发的应用却不合适，主要原因如下：

1. 一个NIO线程同时处理成百上千的链路，性能上无法支撑。即便NIO线程的CPU负荷达到100%，也无法满足海量消息的编码、解码、读取和发送；
2. 当NIO线程负载过重之后，处理速度将变慢，这会导致大量客户端连接超时，超时之后往往进行重发，这更加重了NIO线程的负载，最终导致大量消息积压和处理超时，NIO线程会成为系统的性能瓶颈；
3. 可靠性问题。一旦NIO线程意外跑飞，或者进入死循环，会导致整个系统通讯模块不可用，不能接收和处理外部信息，造成节点故障。

为了解决这些问题，演进出了Reactor多线程模型，下面我们一起学习下Reactor多线程模型。

#### Reactor多线程模型

Reactor多线程模型与单线程模型最大区别就是有一组NIO线程处理I/O操作，它的特点如下：

1. 有一个专门的NIO线程--acceptor新城用于监听服务端，接收客户端的TCP连接请求；
2. 网络I/O操作--读、写等由一个NIO线程池负责，线程池可以采用标准的JDK线程池实现，它包含一个任务队列和N个可用的线程，由这些NIO线程负责消息的读取、解码、编码和发送；
3. 1个NIO线程可以同时处理N条链路，但是1个链路只对应1个NIO线程，防止发生并发操作问题。

![img](../images/Netty/285763-20180123121127162-471886539.png)

在绝大多数场景下，Reactor多线程模型都可以满足性能需求；但是，在极特殊应用场景中，一个NIO线程负责监听和处理所有的客户端连接可能会存在性能问题。例如百万客户端并发连接，或者服务端需要对客户端的握手信息进行安全认证，认证本身非常损耗性能。这类场景下，单独一个Acceptor线程可能会存在性能不足问题，为了解决性能问题，产生了第三种Reactor线程模型--主从Reactor多线程模型。

#### 主从Reactor 多线程模型

特点是：服务端用于接收客户端连接的不再是1个单独的NIO线程，而是一个独立的NIO线程池。Acceptor接收到客户端TCP连接请求处理完成后（可能包含接入认证等），将新创建的SocketChannel注册到I/O线程池（sub reactor线程池）的某个I/O线程上，由它负责SocketChannel的读写和编解码工作。

Acceptor线程池只用于客户端的登录、握手和安全认证，一旦链路建立成功，就将链路注册到后端subReactor线程池的I/O线程上，有I/O线程负责后续的I/O操作。

第三种模型比起第二种模型，是将Reactor分成两部分，mainReactor负责监听server socket，accept新连接，并将建立的socket分派给subReactor。subReactor负责多路分离已连接的socket，读写网 络数据，对业务处理功能，其扔给worker线程池完成。通常，subReactor个数上可与CPU个数等同。

![img](../images/Netty/285763-20180123121145006-1931312241.png)

```java
/**
     * Netty主从线程模型代码
     * @param port
     */
    public void bind3(int port) {
        EventLoopGroup acceptorGroup = new NioEventLoopGroup();
        NioEventLoopGroup ioGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap b = new ServerBootstrap();
            b.group(acceptorGroup, ioGroup)
                .channel(NioServerSocketChannel.class)
                .childHandler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    protected void initChannel(SocketChannel ch) throws Exception {
                        ch.pipeline().addLast("http-codec", new HttpServerCodec());
                        ch.pipeline().addLast("aggregator", new HttpObjectAggregator(65536));
                        ch.pipeline().addLast("http-chunked", new ChunkedWriteHandler());
                        //后面代码省略
                    }
                });

            Channel ch = b.bind(port).sync().channel();
            ch.closeFuture().sync();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            acceptorGroup.shutdownGracefully();
            ioGroup.shutdownGracefully();
        }
    }
```

## 制作服务器

![image-20210929211642278](../images/Netty/image-20210929211642278.png)

### NIO示例

示例

服务端实现步骤:

1. 打开一个服务端通道

2. 绑定对应的端口号

3. 通道默认是阻塞的，需要设置为非阻塞

4. 创建选择器

5. 将服务端通道注册到选择器上,并指定注册监听的事件为OP_ACCEPT

6. 检查选择器是否有事件

7. 获取事件集合

8. 判断事件是否是客户端连接事件SelectionKey.isAcceptable()

9. 得到客户端通道,并将通道注册到选择器上, 并指定监听事件为OP_READ

10. 判断是否是客户端读就绪事件SelectionKey.isReadable()

11. 得到客户端通道,读取数据到缓冲区

12. 给客户端回写数据

13. 从集合中删除对应的事件, 因为防止二次处理.

代码实现:

```java
package com.lagou.selector;

import java.io.IOException;

import java.net.InetSocketAddress;

import java.nio.ByteBuffer;

import java.nio.channels.SelectionKey;

import java.nio.channels.Selector;

import java.nio.channels.ServerSocketChannel;

import java.nio.channels.SocketChannel;

import java.nio.charset.StandardCharsets;

import java.util.Iterator;

import java.util.Set;

/**

* 服务端-选择器

*/

public class NIOSelectorServer {

  public static void main(String[] args) throws IOException,

InterruptedException {

    //1. 打开一个服务端通道

    ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();

    //2. 绑定对应的端口号

    serverSocketChannel.bind(new InetSocketAddress(9999));

    //3. 通道默认是阻塞的，需要设置为非阻塞

    serverSocketChannel.configureBlocking(false);

    //4. 创建选择器

    Selector selector = Selector.open();

    //5. 将服务端通道注册到选择器上,并指定注册监听的事件为OP_ACCEPT

    serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

    System.out.println("服务端启动成功...");

    while (true) {

      //6. 检查选择器是否有事件

      int select = selector.select(2000);

      if (select == 0) {

        continue;

     }

      //7. 获取事件集合 Set Iterator

      while (iterator.hasNext()) {

        //8. 判断事件是否是客户端连接事件SelectionKey.isAcceptable()

        SelectionKey key = iterator.next();

        //9. 得到客户端通道,并将通道注册到选择器上, 并指定监听事件为OP_READ

        if (key.isAcceptable()) {

          SocketChannel socketChannel = serverSocketChannel.accept();

          System.out.println("客户端已连接......" + socketChannel);

          //必须设置通道为非阻塞, 因为selector需要轮询监听每个通道的事件

          socketChannel.configureBlocking(false);

          //并指定监听事件为OP_READ

          socketChannel.register(selector, SelectionKey.OP_READ);

       }

        //10. 判断是否是客户端读就绪事件SelectionKey.isReadable()

        if (key.isReadable()) {

          //11.得到客户端通道,读取数据到缓冲区

          SocketChannel socketChannel = (SocketChannel) key.channel();

          ByteBuffer byteBuffer = ByteBuffer.allocate(1024);

          int read = socketChannel.read(byteBuffer);

          if (read > 0) {

 System.out.println("客户端消息:" +

                new String(byteBuffer.array(), 0, read,

StandardCharsets.UTF_8));

            //12.给客户端回写数据

            socketChannel.write(ByteBuffer.wrap("没

钱".getBytes(StandardCharsets.UTF_8)));

            socketChannel.close();

         }

       }

        //13.从集合中删除对应的事件, 因为防止二次处理.

        iterator.remove();

     }

   }

 }

}
```

下面详细介绍HttpServer的搭建 

在一个channel中处理完整的Http请求 , 在netty处理channel的时候, 只处理消息是FullHttpRequest的Channel, 这样我们就能在一个ChannelHandler中处理一个完整的Http请求了。

Netty服务器的搭建只需要两个类——一个是启动类，负责启动（BootStrap）和main方法，一个是ChannelHandler，负责具体的业务逻辑

启动类:

```java
package com.dz.netty.http;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.handler.codec.http.HttpObjectAggregator;
import io.netty.handler.codec.http.HttpRequestDecoder;
import io.netty.handler.codec.http.HttpResponseEncoder;

/**
 * Created by RoyDeng on 17/7/20.
 */
public class HttpServer {

    private final int port;

    public HttpServer(int port) {
        this.port = port;
    }

    public static void main(String[] args) throws Exception {
        if (args.length != 1) {
            System.err.println(
                    "Usage: " + HttpServer.class.getSimpleName() +
                            " <port>");
            return;
        }
        int port = Integer.parseInt(args[0]);
        new HttpServer(port).start();
    }

    public void start() throws Exception {
        ServerBootstrap b = new ServerBootstrap();
        NioEventLoopGroup group = new NioEventLoopGroup();
        b.group(group)
                .channel(NioServerSocketChannel.class)
                .childHandler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    public void initChannel(SocketChannel ch)
                            throws Exception {
                        System.out.println("initChannel ch:" + ch);
                        ch.pipeline()
                                .addLast("decoder", new HttpRequestDecoder())   // 1
                                .addLast("encoder", new HttpResponseEncoder())  // 2
                                .addLast("aggregator", new HttpObjectAggregator(512 * 1024))    // 3
                                .addLast("handler", new HttpHandler());        // 4
                    }
                })
                .option(ChannelOption.SO_BACKLOG, 128) // determining the number of connections queued
                .childOption(ChannelOption.SO_KEEPALIVE, Boolean.TRUE);

        b.bind(port).sync();
    }
}
```

### Netty 启动一个服务器

[参考](https://hanelalo.github.io/2021/03/20/%E6%8E%98%E9%87%91Netty%E5%B0%8F%E5%86%8C%E5%AD%A6%E4%B9%A0-%E7%AE%80%E6%98%93IM%E5%AE%9E%E6%97%B6%E8%81%8A%E5%A4%A9%E7%B3%BB%E7%BB%9F/)

```java
public class NettyServer {
  public static void main(String[] args) {

    ServerBootstrap serverBootstrap = new ServerBootstrap();
    /** handle connection event */
    NioEventLoopGroup boss = new NioEventLoopGroup();
    /** handle io event, business operation */
    NioEventLoopGroup worker = new NioEventLoopGroup();
    serverBootstrap
        .group(boss, worker)
        // switch io model, general Nio, you can use OIO, but why use netty ?
        .channel(NioServerSocketChannel.class)
        .childHandler(
            new ChannelInitializer<NioSocketChannel>() {
              @Override
              protected void initChannel(NioSocketChannel ch) {
                System.out.println("netty ...");
              }
            });
    ChannelFuture bind = serverBootstrap.bind(8000);
    bind.addListener(
        future -> {
          if (future.isSuccess()) {
            System.out.println("bind port sucess");
          } else {
            System.out.println("bind port failed");
          }
        });
  }
}
```

Netty 将处理网络连接和做数据读写分成了两个线程池来实现，`boss` 用来处理网络连接事件，`worker` 用来做业务处理，`channel()` 方法用来指定创建的 Channle 类型，如果不指定，则自动根据操作系统选择， Windows 一般都会是 `NioSocketChannel`，`childHandler()` 方法用来指定接收到请求的回调处理逻辑，`bind()` 用来绑定端口。

### Netty启动一个客户端

```java
public class NettyClient {

  public static void main(String[] args) {
    Bootstrap bootstrap = new Bootstrap();
    NioEventLoopGroup group = new NioEventLoopGroup();
    bootstrap
        .group(group)
        .channel(NioSocketChannel.class)
        .handler(
            new ChannelInitializer<NioSocketChannel>() {
              @Override
              protected void initChannel(NioSocketChannel ch) throws Exception {
                // 业务处理
              }
            })
        .connect("127.0.0.1", 8000);
  }
```

注意这里使用的不再是 `SeverBootstrap`，而是 `Bootstrap`，因为作为客户端来说，不像服务端一样需要处理多个连接，`group()` 和服务端类似，只不过可以理解为绑定的线程都是用来做业务处理，`handler()` 和服务端的 `childHandler` 类似，用来绑定业务处理逻辑代码，`connect()` 指定服务端地址和端口，建立连接。

## 自定义协议

### 协议格式

- 魔数
  
    4 个字节，也就是一个 int 值，二进制数据包最前面几个字节为固定数字，称为魔数，不屑这一个魔数的化，万一绑定的是 80 端口，可能就直接当成 Http 协议解析了，这就像 Java 的 Class 文件里面开头就有一个 `Coffe(没记错的话)` 的魔数，主要是为了和其他协议区分开。

- 协议版本
  
    1 个字节，还是很大了，最大 128 呢，一个协议要是真的更新这么多个版本，那肯定是个垃圾，当前虽然是一个屁用没有的字段，但是既然要模仿，那就模仿得像一点，得体现出现代自定义协议必备的可扩展性。

- 命令类型
  
    1 个字节，因为是自定义协议，一般都是针对某些系统之间的特定协议，所以完全可以指定一些命令类型，比如用来标识当前的数据请求时登录还是其他的什么请求。

- 主体数据长度
  
    4 个字节，也就是一个 int 值，用来标识真正的请求二进制主体数据的大小，也就是主题数据的 byte 数组的长度

- 主体数据

### 协议数据抽象

```java
@Data
public abstract class Packet {

  /** 协议版本 */
  private byte version = 1;
  /** 需要序列化成 json 的原因，虽然这个字段没用，但是还是得显式定义 */
  private String command;

  /**
   * 获取命令
   */
  public abstract byte getCommand();
}
```

## 序列化实现

```java
/**
 * 序列化
 */
public interface Serializer {

  Serializer DEFAULT = new JSONSerializer();

  /**
   * 序列化算法
   * @return byte
   */
  byte getSerializerAlgorithm();

  /**
   * 序列化
   */
  byte[] serialize(Object object);

  /**
   * 反序列化
   */
  <T> T deserialize(Class<T> clazz, byte[] bytes);
}
```

```java
public class JSONSerializer implements Serializer {

  private final ObjectMapper objectMapper;

  public JSONSerializer(){
    objectMapper = new ObjectMapper();
  }

  @Override
  public byte getSerializerAlgorithm() {
    return SerializerAlgorithm.JSON;
  }

  @Override
  public byte[] serialize(Object object) {
    try {
      return objectMapper.writeValueAsString(object).getBytes(StandardCharsets.UTF_8);
    } catch (JsonProcessingException e) {
      e.printStackTrace();
    }
    return new byte[]{};
  }

  @Override
  public <T> T deserialize(Class<T> clazz, byte[] bytes) {
    try {
      return objectMapper.readValue(new String(bytes), clazz);
    } catch (IOException e) {
      e.printStackTrace();
    }
    return null;
  }

}
```

序列化的实现，首先将 Java 对象转成 json 字符串，这里使用的是 jackson，然后才转成了二进制数组。

这样做是因为之前看了 Dubbo 和 Feign 的区别时，知道 Dubbo 是和 Java 这门语言强绑定的，因为直接将对象序列化为二进制，而 Feign 是先序列化为 json 字符串，然后再将 json 字符串序列化为二进制，这两种方法好处坏处都有， Dubbo 性能更高，因为序列化和反序列化都比 Feign 少了一步 json 转化，但是也因此和 Java 这门语言强绑定，也就是通信的两端都必须是 Java 实现，而 Feign 虽然因为转换成 json 导致性能不如 Dubbo，但是也因为是转换成了 json，解开了对语言的绑定，毕竟现在各种语言都有很多 json 工具的。

​    

## ChannelPipeline

处理或拦截**Channel**入站事件和出站操作的ChannelHandler的列表 . **ChannelPipeline** 实现了一种高级形式的 **拦截过滤器** 模式, 使用户可以完全控制事件的处理方式以及管道中的 **ChannelHandler** 如何相互交互 

管道的创建
每个通道都有自己的管道，并在创建新通道时自动创建。
事件如何在管道中流动
下图描述了ChannelPipeline ChannelHandler通常如何处理 I/O 事件。 I/O 事件由ChannelInboundHandler或ChannelOutboundHandler处理，并通过调用ChannelHandlerContext定义的事件传播方法ChannelHandlerContext.fireChannelRead(Object)例如ChannelHandlerContext.fireChannelRead(Object)和ChannelHandlerContext.write(Object)转发到其最近的处理程序。

### Handler的执行顺序

下图描述了ChannelPipeline ChannelHandler通常如何处理 I/O 事件。 I/O 事件由ChannelInboundHandler或ChannelOutboundHandler处理，并通过调用ChannelHandlerContext定义的事件传播方法ChannelHandlerContext.fireChannelRead(Object)例如ChannelHandlerContext.fireChannelRead(Object)和ChannelHandlerContext.write(Object)转发到其最近的处理程序

![image-20211002124434593](../images/Netty/image-20211002124434593.png)

入站事件由入站处理程序按自下而上的方向处理(逆序处理)，如图左侧所示。 入站处理程序通常处理由图底部的 I/O 线程生成的入站数据。 入站数据通常通过实际的输入操作（例如SocketChannel.read(ByteBuffer)从远程对等方读取。 如果入站事件超出顶部入站处理程序，它将被静默丢弃，或者在需要您注意时记录。
出站事件由出站处理程序按自上而下的方向处理，如图右侧所示。 出站处理程序通常会生成或转换出站流量，例如写入请求。 如果出站事件超出了底部出站处理程序，则由与Channel关联的 I/O 线程处理。 I/O 线程经常执行实际的输出操作，例如SocketChannel.write(ByteBuffer) 。

例如，假设我们创建了以下管道：

```
   ChannelPipeline p = ...;
   p.addLast("1", new InboundHandlerA());
   p.addLast("2", new InboundHandlerB());
   p.addLast("3", new OutboundHandlerA());
   p.addLast("4", new OutboundHandlerB());
   p.addLast("5", new InboundOutboundHandlerX());
```

> 在上面的示例中，名称以Inbound开头的类表示它是一个入站处理程序。 名称以Outbound开头的类表示它是一个出站处理程序。
> 在给定的示例配置中，当事件进入时，处理程序评估顺序为 1、2、3、4、5。 当事件出站时，顺序为ChannelPipeline 。在此原则之上， ChannelPipeline跳过某些处理程序的评估以缩短堆栈深度：
> 3 和 4 没有实现ChannelInboundHandler ，因此入站事件的实际评估顺序将是：1、2 和 5。
> 1 和 2 没有实现ChannelOutboundHandler ，因此出站事件的实际评估顺序将是： ChannelOutboundHandler和 3。
> 如果 5 同时实现ChannelInboundHandler和ChannelOutboundHandler ，则入站和出站事件的评估顺序可能分别为 125 和 543。

将事件转发到下一个处理程序   

处理程序必须调用ChanneHandleContenx的事件传播方法才能将事件转发到其下一个处理程序 , 这些方法包括

入站事件传播方法：

```java
ChannelHandlerContext.fireChannelRegistered()
ChannelHandlerContext.fireChannelActive()
ChannelHandlerContext.fireChannelRead(Object)
ChannelHandlerContext.fireChannelReadComplete()
ChannelHandlerContext.fireExceptionCaught(Throwable)
ChannelHandlerContext.fireUserEventTriggered(Object)
ChannelHandlerContext.fireChannelWritabilityChanged()
ChannelHandlerContext.fireChannelInactive()
ChannelHandlerContext.fireChannelUnregistered()
```

出站事件传播方法：

```java
ChannelHandlerContext.bind(SocketAddress, ChannelPromise)
ChannelHandlerContext.connect(SocketAddress, SocketAddress, ChannelPromise)
ChannelHandlerContext.write(Object, ChannelPromise)
ChannelHandlerContext.flush()
ChannelHandlerContext.read()
ChannelHandlerContext.disconnect(ChannelPromise)
ChannelHandlerContext.close(ChannelPromise)
ChannelHandlerContext.deregister(ChannelPromise)
```

以下示例显示了事件传播通常是如何完成的：

```java
   public class MyInboundHandler extends ChannelInboundHandlerAdapter {
        @Override
       public void channelActive(ChannelHandlerContext ctx) {
           System.out.println("Connected!");
           ctx.fireChannelActive();
       }
   }

   public class MyOutboundHandler extends ChannelOutboundHandlerAdapter {
        @Override
       public void close(ChannelHandlerContext ctx, ChannelPromise promise) {
           System.out.println("Closing ..");
           ctx.close(promise);
       }
   }
```

以上来自官方文档 

Netty中，可以注册多个handler。ChannelInboundHandler按照注册的先后顺序执行；ChannelOutboundHandler按照注册的先后顺序逆序执行，如下图所示，按照注册的先后顺序对Handler进行排序，request进入Netty后的执行顺序为：

![image-20211002125222642](../images/Netty/image-20211002125222642.png)

### 建立管道

用户应该在管道中有一个或多个ChannelHandler来接收 I/O 事件（例如读取）和请求 I/O 操作（例如写入和关闭）。 例如，典型的服务器将在每个通道的管道中具有以下处理程序，但您的里程可能会因协议和业务逻辑的复杂性和特征而异：
协议解码器 - 将二进制数据（例如ByteBuf ）转换为 Java 对象。
协议编码器 - 将 Java 对象转换为二进制数据。
业务逻辑处理程序 - 执行实际的业务逻辑（例如数据库访问）。

## Selector

selector 是一个使用 nonblocking io 的通用封装

```java
Selector selector = Selector.open();
ServerSocketChannel servChannel = ServerSocketChannel.open();
servChannel.configureBlocking(false);
// 建立一个server socket，到本地端口9999， backlog 1024
servChannel.socket().setReuseAddress(true);
servChannel.socket().bind(new InetSocketAddress(9999), 1024);
// selector 关心 server 上的 ACCEPT 事件
servChannel.register(selector, SelectionKey.OP_ACCEPT); 

while (start) {
  try {
    // 阻塞等待 直到有IO事件可读(系统IO事件队列不为空)
    selector.select();
    // 获取 事件 以及 事件所对应的 channel (client server 的连接)
    Set<SelectionKey> selectedKeys = selector.selectedKeys();
    Iterator<SelectionKey> it = selectedKeys.iterator();
    SelectionKey key = null;
    while (it.hasNext()) {
      key = it.next();
      it.remove();
      try {
        if (key.isValid()) {
          // OP_ACCEPT 事件 表示有个新client 完成了三次握手。连接上了本服务器
          if (key.isAcceptable()) {
            // Accept the new connection
            ServerSocketChannel ssc = (ServerSocketChannel) key.channel();
            SocketChannel sc = ssc.accept();
            sc.configureBlocking(false);
            // 将该连接的可读事件 注册到 selector， 到时候他发起请求的时候，我会收到新事件
            sc.register(selector, SelectionKey.OP_READ);
          }
          // OP_READ 事件 说明 client 发的数据已经发到了系统缓冲区，server 可以去读了。
          if (key.isReadable()) {
            SocketChannel sc = (SocketChannel) key.channel();
            // 分配用户台空间, 将数据从内核态 拷贝到 用户态
            ByteBuffer readBuffer = ByteBuffer.allocate(4);
            int readBytes = sc.read(readBuffer);
            if (readBytes > 0) {
              // 切换读写模式 详见下面的图, 表示自己目前可以读 [position, limit]
              readBuffer.flip();
              byte[] bytes = new byte[readBuffer.remaining()];
              // 将buffer 数据拷贝到 bytes 数组
              // 如果这里只收到一半的数据怎么办？
              String body = new String(bytes, "UTF-8");
              System.out.println(body);
              // 将 read的数据 写回去
              ByteBuffer writeBuffer = ByteBuffer.allocate(bytes.length);
              writeBuffer.put(bytes);
              writeBuffer.flip();
              sc.write(writeBuffer);
            } else if (readBytes < 0) {
              // 对端链路关闭
              key.cancel();
              sc.close();
            } else
              ;
          }
        }
      } catch (Exception e) {
        if (key != null) {
          key.cancel();
          if (key.channel() != null)
            key.channel().close();
        }
      }
    }
  } catch (Exception e) {
    throw e;
  }
}
```

代码量长

下面基于Netty

### netty echo server

```java
public class Netty4Demo {
  public class EchoHandler extends SimpleChannelInboundHandler {
    @Override
    public void channelRead0(ChannelHandlerContext ctx, Object msg) throws Exception {
      // 为什么 这里可以强转String？
      String in = (String) msg;
      System.out.print(in);
      // 将数据写回
      ctx.writeAndFlush(in);
    }
  }

  public void run() throws Exception {
    EventLoopGroup acceptGroup = new NioEventLoopGroup(1); // 指定 Acceptor 线程池大小
    EventLoopGroup ioGroup = new NioEventLoopGroup(1); // 指定 NIO线程池大小
    try {
      ServerBootstrap b = new ServerBootstrap(); // 创建 ServerBootstrap 对象，他是Netty 用于启动NIO 服务端的辅助启动类，目的是降低服务端的开发复杂度。
      b.group(acceptGroup, ioGrou).channel(
          NioServerSocketChannel.class) // 指定使用 java 的NioServerSocketChannel
          .childHandler(new ChannelInitializer<SocketChannel>() { // 创建 IOThread 的 pipeline
            @Override
            public void initChannel(SocketChannel ch) throws Exception {
              ch.pipeline()
                .addLast(new StringDecoder())
                .addLast(new StringEncoder()); // 添加echo Handler
                .addLast(new EchoHandler())
            }
          }).option(ChannelOption.SO_BACKLOG, 128)          // server socket config backlog 设置为 128
          .childOption(ChannelOption.SO_KEEPALIVE, true); // client socket config 设置 keepalive = true
      // 绑定端口，开始接收进来的连接
      ChannelFuture f = b.bind(9999).sync(); // 同步等待绑定本地端口

      // 等待服务器  socket 关闭 。
      // 在这个例子中，这不会发生，但你可以优雅地关闭你的服务器。
      f.channel().closeFuture().sync();
    } finally {
      // 释放两个线程池
      acceptGroup.shutdownGracefully();
      ioGrou.shutdownGracefully();
    }
  }

  public static void main(String[] args) throws Exception {
    new Netty4Demo().run();
  }
  }
```
