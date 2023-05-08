# java 异步请求

不谈需求就使用框架的程序员不是好程序员 

<!--more-->

首先是异步的流程图 

![一文带你彻底了解Java异步编程](../images/Java%E5%BC%82%E6%AD%A5/217a9f67c7890a907a736620bc627eb0.png)

异步场景 , IO密集型 , 一般场景也可使用其做优化 

  其次异步的原理简述:   ` 异步基于回调`

## 使用RxJava 异步的效果

降低接口的平均响应时长  

CPU负载情况也会有下降, 但CPU使用率并无影响(一般情况下异步化后cpu的利用率会有所提高, 但要看具体的业务场景)

CPU LoadAverage 是指：一段时间内处于可运行状态和不可中断状态的进程平均数量。(可运行分为正在运行进程和正在等待 CPU 的进程；**不可中断则是它正在做某些工作不能被中断比如等待磁盘 IO、网络 IO 等**)

而我们的服务业务场景大部分都是 IO 密集型业务，功能实现很多需要依赖底层接口，会进行频繁的 IO 操作。

![image-20210620110601649](../images/Java%E5%BC%82%E6%AD%A5/image-20210620110601649.png)

  nio 模式带来的好处 

- 提升QPS(用更少的线程资源来实现更高的并发能力)
- 降低CPU负荷, 提高利用率

## 异步网络模型

[参考](https://caorong.github.io/2016/12/24/head-first-netty-1/)

异步也就是非阻塞IO,  在java 中由于 jdk1.4 之后才被引入所以被称之为 Nio (New IO)

![img](../images/Java%E5%BC%82%E6%AD%A5/nio-pic.png)

由图可知， 实现异步的原理是 系统帮你给每一个连接(channel) 都维护了一个 `buffer`。
将 client 发来的数据暂存到 `buffer` 中, 待 java 进程需要的时候一次性做一次内存拷贝，在 java 进程中使用。

当然， 写出也是同样的 写到 `buffer` 中，当 `buffer` 满 或者 手动调用 `flush` 写出去。

当使用noblocking socket时在创建socketchannel后都需要做以下步骤

- 当创建socketChannel时,需要轮询判断是否有新的连接(socket)进来
- 连接进来后，由于我们不知道 client 何时能发完数据， 所以我们需要维护一个列表 `socketChannelList` 定期去轮询列表，看是否有数据可读。

把上面的代码提取称一个通用模块 

在 [berkeley socket](https://en.wikipedia.org/wiki/Berkeley_sockets) 中这个模块被称为 select, poll, epoll. (后面2个是加强版)

而在java中， java 在他们之上抽象成 Selector
