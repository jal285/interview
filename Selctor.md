# NIO选择器

## Selector是什么

![img](../images/Selctor/webp.webp)

NIO(非阻塞IO) 式通信模型中 ,使用的为单线程, 作为可以应对多连接数的模型 ,Selecotr起到的作用

就是:

- 作为一个员工去接收所有从客户(客户端)发送过来的全部数据
- 厂家(服务器端)处理完"请求业务"之后返回response给客户(端)

这里最重要的一点是: 当客户端通过socket建立完成后,会通知Thread接收其发送的信息,并将其运送到Selecotr, Selecotr会不断地遍历所有Socket，一旦有一个Socket建立完成，他会通知Thread，然后Thread处理完数据再返回给客户端——**这个过程是不阻塞的**，这样就能让一个Thread处理更多的请求了。 

大白话就是, 同时有多个客户想要购买厂家的商品. 他们将购买意愿和定金什么的交给给员工,员工负责将客户的需求带给厂家,也就是上头,一个员工可以同时和多个顾客保持联系 , 但会一个一个的记录顾客是否要买厂家的产品, 也即是单线程处理了多个请求. 最终客户得到商品也是一个一个的从员工拿到货 

## 应用

用一个线程处理多个的客户端连接,就会使用到NIO的Selector(选择器) . Selsector 能够检测多个注册的服务端通道上是否有事件发生,如果有,便获取事件然后针对其进行相应的处理. 这样就可以只用一个单线程去管理多个通道,也就是管理多个连接和请求

![image.png](../images/NIO%E7%BC%96%E7%A8%8BSelctor/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=.png)

在这种没有选择器的情况下,对应每个连接对应一个处理线程. 但是连接并不能马上就会发送信息,所以还会产生资源浪费

![image.png](../images/NIO%E7%BC%96%E7%A8%8BSelctor/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=.png)

只有在通道真正有读写事件发生时，才会进行读写，就大大地减少了系统开销，并且不必为每个连接都创建一个线程，不用去维护多个线程, 避免了多线程之间的上下文切换导致的开销

## 常用API

Selector 类是一个抽象类

![image.png](../images/NIO%E7%BC%96%E7%A8%8BSelctor/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=.png)

常用方法:

Selector.open() : //得到一个选择器对象

selector.select() : //阻塞 监控所有注册的通道,当有对应的事件操作时, 会将SelectionKey放入集合内部并返回事件数量

selector.select(1000): //阻塞 1000 毫秒，监控所有注册的通道,当有对应的事件操作时, 会将SelectionKey放入集合内部并返回

selector.selectedKeys() : // 返回存有SelectionKey的集合

SelectionKey

![image.png](../images/NIO%E7%BC%96%E7%A8%8BSelctor/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=.png)

常用方法

SelectionKey.isAcceptable(): 是否是连接继续事件

SelectionKey.isConnectable(): 是否是连接就绪事件

SelectionKey.isReadable(): 是否是读就绪事件

SelectionKey.isWritable(): 是否是写就绪事件

SelectionKey中定义的4种事件:

SelectionKey.OP_ACCEPT —— 接收连接继续事件，表示服务器监听到了客户连接，服务器可以接收这个连接了

SelectionKey.OP_CONNECT —— 连接就绪事件，表示客户端与服务器的连接已经建立成功

SelectionKey.OP_READ —— 读就绪事件，表示通道中已经有了可读的数据，可以执行读操作了（通道目前有数据，可以进行读操作了）

SelectionKey.OP_WRITE —— 写就绪事件，表示已经可以向通道写数据了（通道目前可以用于写操作）

## Selector 编码
