# ZooKeeper数据结构和实际操作

<!--more-->

### 介绍

ZooKeeper 主要为大型分布式计算提供开源的分布式配置服务、同步服务和命名注册。曾经是 Hadoop 项目中的一个子项目，用来控制集群中的数据，目前已升级为独立的顶级项目。很多场景下也用它作为 Service 发现服务解决方案。  
ZooKeeper 是基于 CP 来设计的，即任何时刻对 ZooKeeper 的访问请求能得到一致的数据结果，同时系统对网络分割具备容错性，但是它不能保证每次服务请求的可用性。从实际情况来分析，在使用 ZooKeeper 获取服务列表时，如果 zookeeper 正在选主，或者 ZooKeeper 集群中半数以上机器不可用，那么将无法获得数据。所以说，ZooKeeper 不能保证服务可用性。

### 用途

在分布式架构中往往伴随 CAP 的理论。因为分布式的架构，不再使用传统的单机架构，多机为了提供可靠服务所以需要冗余数据因而会存在分区容忍性Ｐ。  
冗余数据的同时会在复制数据的同时伴随着可用性 A 和强一致性 C 的问题。是选择停止可用性达到强一致性还是保留可用性选择最终一致性。通常选择后者。其中 Zookeeper 和 Eureka 分别是注册中心 CP AP 的两种的实践。

## 数据结构

ZooKeeper维护一个类似文件系统的数据结构（如下官方示意图），每一个子目录项（如app1）都被称作为znode（目录节点），和文件系统一样，我们能够自由的对一个znode进行CRUD，也可以在znode下进行子znode的CRUD，唯一不同的是，znode是可以存储数据的。

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/99abd357a8de42b8b8725d4ab61611a6~tplv-k3u1fbpfcp-watermark.awebp)

### Znode类型分为三类

- 持久节点（persistent node）节点会被持久化。
- 临时节点  -e（ephemeral node），客户端断开连接后，ZooKeeper会自动删除临时节点。
- 顺序节点 -s（sequential node），每次创建顺序节点时，ZooKeeper都会在路径后面自动添加上10位的数字，从1开始，最大是2147483647（2^32-1） 
    每个顺序节点都有一个单独的计数器，并且单调递增的，由ZooKeeper的leader实例维护。

Znode实际上有四种形式，默认是persistent。

（1）PERSISTENT 持久节点：如 create/test/a "hello" ，通过 create 参数指定为持久节点

（2）PERSISTENT_SEQUENTIAL（持久顺序节点/s0000000001），通过 create -s 参数指定为顺序节点

（3）EPHEMERAL 临时节点，通过 create -e 参数指定为顺序节点

（4）EPHEMERAL_SEQUENTIAL（临时顺序节点/s0000000001），通过 create -s -e 参数指定为临时及顺序节点![图片](/images/Zookeeper/beeaf70f45fc4a85b882cbaf6db26872tplv-k3u1fbpfcp-watermark.awebp)

ZooKeeper3.5.x中引

（1）container 节点用来存放子节点，如果container节点中的子节点为0 ，则container节点在未来会被服务器删除，定时任务默认60秒执行一次。

（2）ttl 节点默认禁用，需要通过配置开启，如果ttl 节点没有子节点，或者 ttl 节点在 指定的时间内没有被修改则会被服务器删除。

[实例参考](https://juejin.cn/post/6959754013323919391)

### Zxid

![file](../images/Zookeeper/17245d953b547968tplv-t2oaga2asx-watermark.awebp)

以下变化都会产生一个集群全局的唯一的事务Id， Zxid（`Zookeeper Transaction Id `）

- 任何的客户端连接到`Server`
- 任何的客户端断开与`Server`连接
- 任何的Znode节点被创建`create`、修改`set`、删除`delete`，`rmr`

Zxid是一个64位的数字，高32位表示纪元，从1开始，每次选举出一个新的`leader`,就会递增1；低32位是当前纪元维护的单调递增的数字，同样从1开始。

### Znode  属性

![image-20211201184828693](../images/Zookeeper/image-20211201184828693.png)

`cZxid` ：创建的事务标识。

`ctime`：创建的时间戳

`mZxid`：修改的事务标识，每次修改操作（`set`）后都会更新`mZxid`和`mtime`。

`mtime`：修改的时间戳

`pZxid`：直接子节点最后更新的事务标识，子节点有变化（创建`create`、修改`set`、删除`delete`，`rmr`）时，都会更新`pZxid`。

`cversion` ：直接子节点的版本号。当子节点有变化（创建`create`、修改`set`、删除`delete`，`rmr`）时，`cversion` 的值就会增加1。

`dataVersion`  ：节点数据的版本号，每次对节点进行修改操作（`set`）后，`dataVersion`的值都会增加1（即使设置的是相同的数据）。

`aclVersion`  ：节点ACL的版本号，每次节点的ACL进行变化时，`aclVersion` 的值就会增加1。

`ephemeralOwner`：当前节点是临时节点（`ephemeral node` ）时，这个`ephemeralOwner`的值是客户端持有的`session` id。

`dataLength`：节点存储的数据长度，单位为 B （字节）。

`numChildren`：直接子节点的个数。

​    

```
[zk: localhost:2181(CONNECTED) 12] get -s /

cZxid = 0x0
ctime = Thu Jan 01 08:00:00 CST 1970
mZxid = 0x0
mtime = Thu Jan 01 08:00:00 CST 1970
pZxid = 0x2
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 2
```

默认根节点 / 和 /zookeeper 存在， cZxid 是 0x0

### 监视（Watch)

ZooKeeper支持 Watch。客户端可以在znode上设置 Watch。

znode更改时，将触发并删除监视。触发监视后，客户端会收到一个数据包，说明znode已更改。

如果客户端与其中一个ZooKeeper服务器之间的连接断开，则客户端将收到本地通

**3.6.0中的新增功能：**  

客户端还可以在znode上设置永久性的递归监视，这些监视在触发时不会删除，并且会以递归方式触发注册znode以及所有子znode的更改。

支持 Watch的 客户端命令：

- `stat path [watch]`
- `ls path [watch]`
- `ls2 path [watch]`
- `get path [watch]`

监视某个节点

```
[zk: localhost:2181(CONNECTED) 16] get  -w /test
hello
```

更改节点的数据， 就可以看到WATCHER 通知

```
[zk: localhost:2181(CONNECTED) 21] set  /test "hhhhhh"

WATCHER::

WatchedEvent state:SyncConnected type:NodeDataChanged path:/test
```

## Zookeeper分布式锁

### 公平锁和可重入锁

最经典的分布式锁是可重入的公平锁 

公平锁：独占锁的一种， 有先来后到之说，公平

重入锁：一把独占锁，可以被多次锁定， 可重入 

### ZooKeeper 分布式锁的原理

(一)ZooKeeper的每一个节点，都是一个天然的顺序发号器

在每一个节点下面创建临时顺序节点（EPHEMERAL_SEQUENTIAL）类型，新的子节点后面，会加上一个次序编号，而这个生成的次序编号，是上一个生成的次序编号加一。

例如，有一个用于发号的节点“/test/lock”为父亲节点，可以在这个父节点下面创建相同前缀的临时顺序子节点，假定相同的前缀为“/test/lock/seq-”。第一个创建的子节点基本上应该为/test/lock/seq-0000000000，下一个节点则为/test/lock/seq-0000000001，依次类推

![在这里插入图片描述](../images/Zookeeper/20210309155957390.png)

​                                            `Zookeeper 临时顺序节点的天然的发号器作用`

(二) ZooKeeper节点的递增有序性，可以确保锁的公平

一个ZooKeeper分布式锁，首先需要创建一个父节点，尽量是持久节点（PERSISTENT类型），然后每个要获得锁的线程，都在这个节点下创建个临时顺序节点。由于ZK节点，是按照创建的次序，依次递增的。

为了确保公平，可以简单的规定：编号最小的那个节点，表示获得了锁。所以，每个线程在尝试占用锁之前，首先判断自己是排号是不是当前最小，如果是，则获取锁。

(三) Zookeeper 的节点监听机制 

每个线程抢占锁之前, 先尝试创建自己的ZNode. 同样, 释放锁的时候, 就需要删除创建的Znode . 创建成功后，如果不是排号最小的节点，就处于等待通知的状态。等谁的通知呢？不需要其他人，只需要等前一个Znode的通知就可以了。前一个Znode删除的时候，会触发Znode事件，当前节点能监听到删除事件，就是轮到了自己占有锁的时候。第一个通知第二个、第二个通知第三个，击鼓传花似的依次向后。

ZooKeeper的节点监听机制，能够非常完美地实现这种击鼓传花似的信息传递。具体的方法是，每一个等通知的Znode节点，只需要监听（linsten）或者监视（watch）排号在自己前面那个，而且紧挨在自己前面的那个节点，就能收到其删除事件了。只要上一个节点被删除了，就进行再一次判断，看看自己是不是序号最小的那个节点，如果是，自己就获得锁。

另外，ZooKeeper的内部优越的机制，能保证由于网络异常或者其他原因，集群中占用锁的客户端失联时，锁能够被有效释放。一旦占用Znode锁的客户端与ZooKeeper集群服务器失去联系，这个临时Znode也将自动删除。排在它后面的那个节点，也能收到删除事件，从而获得锁。正是由于这个原因，在创建取号节点的时候，尽量创建临时znode节点，

(四)ZooKeeper的节点监听机制, 能避免羊群效应 

ZooKeeper这种首尾相接，后面监听前面的方式，可以避免羊群效应。所谓羊群效应就是一个节点挂掉，所有节点都去监听，然后做出反应，这样会给服务器带来巨大压力，所以有了临时顺序节点，当一个节点挂掉，只有它后面的那一个节点才做出反应。

## 分布式锁的抢占过程

![在这里插入图片描述](../images/Zookeeper/20210309105355106.png)

假设客户端A抢先一步, 对zk发起了加分布式锁的请求,这个加锁请求是用到了zk中的一个特殊的概念, 叫做"临时顺序节点". 

简单来说 就是直接在"my_lock"这个锁节点下，创建一个顺序节点，这个顺序节点有zk内部自行维护的一个节点序号。

### 客户端A发起一个加锁请求

第一个客户端来搞一个顺序节点 ,zk内部会给起名位 :xxx-000001, 第二个: xxx-000002. 从一开始数字一次递增 , 由zk维护这个顺序 

就是直接在"my_lock"这个锁节点下，创建一个顺序节点，这个顺序节点有zk内部自行维护的一个节点序号。

就是直接在"my_lock"这个锁节点下，创建一个顺序节点，这个顺序节点有zk内部自行维护的一个节点序号。

![加锁请求生成一个临时顺序节点](../images/Zookeeper/20210309105443497.png)

A创建完一个顺序节点后会查询"my_lock" 这个锁节点下的所有子节点, 这些子节点是按照序号排列的, A**将会判断自身在所有子节点内的位置**, 会根据其所在的序号进行请求加锁, 在这里A是第一个, 所以也是第一个请求加锁的

![在这里插入图片描述](../images/Zookeeper/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NyYXp5bWFrZXJjaXJjbGU=,size_16,color_FFFFFF,t_70.png)

### 客户端B过来排队

A加锁后, B过来想要加锁,会重复上述A的过程, 

![在这里插入图片描述](../images/Zookeeper/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NyYXp5bWFrZXJjaXJjbGU=,size_16,color_FFFFFF,t_70.png)

从上图可以看出B将会在"my_lock"下创建新的子节点, zk内部会维护其序号为"2"

查询"my_lock"锁节点下的所有子节点,按序号顺序排列, 类似于

![img](../images/Zookeeper/1599f548c182bd5c6a8e4b47a93a71be.png)

同时检查自己创建的顺序节点是不是集合中的第一个, 明显不是,此时第一个是客户端A创建的顺序节点, 所以加锁失败

### 客户端B开启监听客户端A

加锁失败了以后，客户端B就会通过ZK的API对他的顺序节点的上一个顺序节点加一个监听器。zk天然就可以实现对某个节点的监听。

B的上一个顺序节点及A创建的那个顺序节点, 客户端会对A创建的顺序节点加一个监听器 

加锁失败了以后，客户端B就会通过ZK的API对他的顺序节点的上一个顺序节点加一个监听器。zk天然就可以实现对某个节点的监听。

![在这里插入图片描述](../images/Zookeeper/2021030911010510.png)

在A释放锁之后, A将会把自己在zk里创建的顺序节点, 001 删除 , 删除后zk会负责通知监听这个节点的监视器,此时被通知节点将会知道被监视节点释放

客户端B重新尝试获取锁, "my_lock"节点下的子节点集合中只剩下B对应节点序号, 所以B将是集合中的第一个顺序节点, 此时可以加锁,完成加锁,运行后续业务代码,完成业务后再释放锁

## 实现

使用ZooKeeper实**现分布式锁**的算法，有以下几个要点：

（1）**一把分布式锁通常使用一个Znode节点表示**；如果锁对应的Znode节点不存在，首先创建Znode节点。这里假设为“/test/lock”，代表了一把需要创建的分布式锁。

（2）抢占锁的所有客户端，使用锁的Znode节点的子节点列表来表示；如果某个客户端需要占用锁，则在“/test/lock”下创建一个临时有序的子节点。

这里，所有临时有序子节点，尽量共用一个有意义的子节点前缀。

比如，如果子节点的前缀为“/test/lock/seq-”，则第一次抢锁对应的子节点为“/test/lock/seq-000000000”，第二次抢锁对应的子节点为“/test/lock/seq-000000001”，以此类推。

再比如，如果子节点前缀为“/test/lock/”，则[第一次抢锁对]()应的子$节点为$“/test/lock/000000000”，第二次抢锁对应的子节点为“/test/lock/000000001”，以此类推，也非常直观。

（3）如果判定客户端是否占有锁呢？
很简单，客户端创建子节点后，需要进行判断：自己创建的子节点，是否为当前子节点列表中序号最小的子节点。如果是，则认为加锁成功；如果不是，则监听前一个Znode子节点变更消息，等待前一个节点释放锁。

（4）一旦队列中的后面的节点，获得前一个子节点变更通知，则开始进行判断，判断自己是否为当前子节点列表中序号最小的子节点，如果是，则认为加锁成功；如果不是，则持续监听，一直到获得锁。

（5）获取锁后，开始处理业务流程。完成业务流程后，删除自己的对应的子节点，完成释放锁的工作，以方面后继节点能捕获到节点变更通知，获得分布式锁。

> 定义Lock接口, 加锁和解锁方法

## 搭建Zookeeper集群

docker 拉取3.5.8版本zookeeper image 

```
docker pull zookeeper:3.5.8
```

使用如下命令启动Zookeeper docker

```
docker run -p 8080:8080 --name zookeeper-standalone --restart always -d zookeeper:3.5.8
```

直接进入zookeeper服务

```
zkCli.sh -server 127.0.0.1:2181 
```

### 集群模式

为了避免一个个的启动，使用docker-compose的方式来启动Zookeeper集群 

创建一个docker-compose.yml 文件

```
version: '3.1'

services:
  zoo1:
    image: zookeeper:3.5.8
    restart: always
    hostname: zoo1
    ports:
      - 2181:2181
    environment:
      ZOO_MY_ID: 1
      ZOO_SERVERS: server.1=0.0.0.0:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=zoo3:2888:3888;2181

  zoo2:
    image: zookeeper:3.5.8
    restart: always
    hostname: zoo2
    ports:
      - 2182:2181
    environment:
      ZOO_MY_ID: 2
      ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=0.0.0.0:2888:3888;2181 server.3=zoo3:2888:3888;2181

  zoo3:
    image: zookeeper:3.5.8
    restart: always
    hostname: zoo3
    ports:
      - 2183:2181
    environment:
      ZOO_MY_ID: 3
      ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=0.0.0.0:2888:3888;2181
```

上面以副本模式启动Zookeeper ，同时会告诉 Docker 运行三个 Zookeeper 容器：zoo1、zoo2、zoo3，并分别将本地的 2181, 2182, 2183 端口绑定到对应的容器的 2181 端口上。

ZOO_MY_ID 和 ZOO_SERVERS 是搭建 Zookeeper 集群需要设置的两个环境变量, 其中 ZOO_MY_ID 表示 Zookeeper 服务的 id, 它是1-255 之间的整数, 必须在集群中唯一。ZOO_SERVERS 是Zookeeper 集群的主机列表。

在docker-compose.yml 文件当前目录下运行下面命令：     

```
COMPOSE_PROJECT_NAME=zookeeper_cluster docker-compose up -d
```

会出现三个副本容器

 ![](D:\program\writing\images\2022-03-26-16-51-08-image.png)

进入zookeeper_cluster_zoo1_1 中使用 `zkServer.sh status`  查看zookeeper状态 

```
 zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: follower
```

可知该节点是Follower节点,依次类推，查看其余两个容器启动状态

```
usr/docker# docker exec -it zookeeper_cluster_zoo2_1  /bin/sh
# zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: follower


/usr/docker# docker exec -it zookeeper_cluster_zoo3_1  /bin/sh
# zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: leader
```

由上面信息可以知道我们创建了一个主节点leader，两个从节点Follower

在zookeeper_cluster_zoo3_1 容器中查看配置文件

```
 cat /conf/zoo.cfg
dataDir=/data
dataLogDir=/datalog
tickTime=2000
initLimit=5
syncLimit=2
autopurge.snapRetainCount=3
autopurge.purgeInterval=0
maxClientCnxns=60
standaloneEnabled=true
admin.enableServer=true
server.1=zoo1:2888:3888;2181
server.2=zoo2:2888:3888;2181
server.3=0.0.0.0:2888:3888;2181
```

可以看到Zookeeer节点的配置信息
