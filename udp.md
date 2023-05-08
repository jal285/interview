# 关于udp协议下socket



> UDP协议，也就是用户数据报协议（User Datagram Protocol），是一个简单的面向数据报的传输层协议。只在IP协议上增加了很少一点的功能，就是复用和分用，以及差错检测的功能。 

特点; 

- 无连接：也就是说发送之前不需要建立连接，直接发送就可以，和TCP协议相比就减少了三次握手四次挥手等时间的消耗。]
- 不可靠交付：也就是说，我们只管发送数据，对方收没收到不需要去管。
- 面向报文：只进行简单的添加首部数据，就直接封装成IP包发送了。
- 支持多对多：这里表示的就是单播多播广播机制。
- 没有拥塞控制

**数据格式**

在上面我们知道，UDP协议包只在I协议上增加了很少一点的功能，就是复用和分用，以及差错检测的功能。那添加的这些数据是什么样子的呢？

UDP协议分为首部字段和数据字段，其中首部字段只占用8个字节，分别是个占用两个字节的源端口、目的端口、长度和检验和。



![img](../images/udp/16ca48bdb1f17b27tplv-t2oaga2asx-watermark.awebp)



我们在这里对添加的数据字段分别进行一个解释说明：

（1）伪首部：伪首部其实是在校验和的时候添加的，起到一个辅助运算的作用。并不是真正的首部。校验和是为了检查报文中的数据是否有差错，如果没有差错，那么校验和之后的结果应该是1.

（2）源端口：源端口，这是为了收到对方的回音才使用的。

（3）目的端口：也就是UDP数据包的发送目的地。

（4）长度：数据长度

（5）检验和：检查是否数据出现了差错，如果有差错，那就丢弃。



## 三种通信模式

  1.单播：单台主机与单台主机之间的通信；

  2.广播：单台主机与网络中所有主机的通信；

  3.组播：单台主机与选定的一组主机的通信；





### 单播

```java
 单播是网络通信中最常见的，网络节点之间的通信 就好像是人们之间的对话一样。如果一个人对另外一个人说话，

 那么用网络技术的术语来描述就是“单播”，此时信息的接收和传递只在两个节点之间进行。

 1. 单播的优点：

     (1)服务器以及响应客户端的请求；

     (2)服务器能针对每个客户端的不同请求发送不同的响应，容易显示个性化服务；

 2. 单播的缺点：

     (1)服务器针对每个客户机发送数据流，服务器流量＝客户机数量×客户机流量；在客户数量大、每个客户机流量大的流媒体应用中服务器不堪重负；
 3. 应用场景：

    单播在网络中得到了广泛的应用，网络上绝大部分的数据都 是以单播的形式传输的。例如：收发电子邮件、游览网页时，必须与邮件服务器、

    服务器建立连接，此时使用的就是单播通信方式；
```





### 广播

  “广播”可以比方为：一个人通过广播喇叭对在场的全体说话(他才不管你是否乐意听)。换句话说: 广播是一台主机对某一个网络上的所有主机发送数据报包。

```java
这个网络可能是网络，也可能时子网，还有可能是所有子网。

广播有两类：本地广播和定向广播：

        定向广播：将数据报包发送到本网络之外的特定网络的所有主机，然而，由于互联网上的大部分路由器都不转发定向广播消息，所以这里不深入介绍了

        本地广播：将数据报包发送到本地网络的所有主机，IPv4的本地广播地址为“255.255.255.255”，路由器不会转发此广播；
1.广播的优点：

   (1)通信的效率高，信息一下子就可以传递到某一个网络上的所有主机。

   (2)由于服务器不用向每个客户端单独发送数据，所以服务器流量比较负载低；
2.广播的缺点：

   (1)非常占用网络的带宽；

   (2)缺乏针对性,也不管主机是否真的需要接收该数据, 就强制的接收数据；

3.应用场景：

   (1)有线电视就是典型的广播型网络
```





### 组播(又叫多播)

同时具有单播和广播的优点, 最具有发展前景 

   ”组播“可以比方为：你对着大街喊：”是男人的来一下，一人发一百块”，那么男的过来，女就不会过来,因为没有钱发她不理你(组播：其中所有的男生就是一个组)，

```java
 换句话说: 组播是一台主机向指定的一组主机发送数据报包，因为如果采用单播方式，逐个节点传输，有多少个目标节点，就会有多少次传送过程，这种方式显然效率 极低，是不可取  
 的；如果采用不区分目标、全部发送的广播方式，虽然一次可以传送完数据，但是显然达不到区分特定数据接收对象的目的，又会占用网络带宽。采用组播方式，既可以 实现一次传送所

 有目标节点的数据，也可以达到只对特定对象传送数据的目的。

 IP网络的组播一般通过组播IP地址来实现。组播IP地址就是D类IP地址，即224.0.0.0至239.255.255.255之间的IP地址。
 1.组播的优点：

    (1)具备广播所具备的所有优点；

(2)与单播相比，提供了发送数据报包的效率，与广播相比，减少了网络流量；

 2.组播的缺点：

    (1)与单播协议相比没有纠错机制，发生丢包错包后难以弥补，但可以通过一定的容错机制和QOS加以弥补；
```

关于组播的例子

```java

import java.io.IOException;
import java.net.DatagramPacket;
import java.net.InetAddress;
import java.net.MulticastSocket;
 
//客户端
public class MulticastSender
{
    public static void main(String[] args) throws IOException
    {
        int port = 8888;
        byte[] msg = "Connection successfully!!!".getBytes();
 
        InetAddress inetRemoteAddr = InetAddress.getByName("224.0.0.5");
 
        /*
         * Java UDP组播应用程序主要通过MulticastSocket实例进行通信,它是DatagramSocket的是一个子类,
         * 其中包含了一些额外的可以控制多播的属性.
         * 
         * 注意：
         * 
         * 多播数据报包实际上可以通过DatagramSocket发送,只需要简单地指定一个多播地址。
         * 我们这里使用MulticastSocket,是因为它具有DatagramSocket没有的能力
         */
        MulticastSocket client = new MulticastSocket();
 
        DatagramPacket sendPack = new DatagramPacket(msg, msg.length,
                inetRemoteAddr, port);
 
        client.send(sendPack);
 
        System.out.println("Client send msg complete");
 
        client.close();
 
    }
}
 
 
import java.io.IOException;
import java.net.DatagramPacket;
import java.net.InetAddress;
import java.net.MulticastSocket;
import java.util.Arrays;
 
//服务端
public class MulticastReceive
{
    public static void main(String[] args) throws IOException
    {
        InetAddress inetRemoteAddr = InetAddress.getByName("224.0.0.5");
 
        DatagramPacket recvPack = new DatagramPacket(new byte[1024], 1024);
 
        MulticastSocket server = new MulticastSocket(8888);
 
        /*
         * 如果是发送数据报包,可以不加入多播组; 如果是接收数据报包,必须加入多播组; 这里是接收数据报包,所以必须加入多播组;
         */
        server.joinGroup(inetRemoteAddr);
 
        System.out.println("---------------------------------");
        System.out.println("Server current start......");
        System.out.println("---------------------------------");
 
        while (true)
        {
            server.receive(recvPack);
 
            byte[] recvByte = Arrays.copyOfRange(recvPack.getData(), 0,
                    recvPack.getLength());
 
            System.out.println("Server receive msg:" + new String(recvByte));
        }
 
    }
}

```

