# Netty - EventLoop,EventLoopGroup

netty 的 EventloopGroup 其实就是线程池， 通过它来配置 接收连接 和 处理连接读写 的线程池大小。

EventLoop里则封装了类似selector的单线程模块

![img](../images/NettyEvent/2016-11-24-23-22-57.png)

最为重点关注画紫圈的地方, 也就是Boss EventLoopGroup, Worker EventLoopGroup, Channel Pipeline(Handler1 Handler2 Handler3……)

有几个关键点。

1. Eventloopsize 建议设置为 2 的次方，dispatch 使用位移，更快。
2. 侦听一个端口，只会绑定到 BossEventLoopGroup 中的一个 Eventloop，所以， BossEventLoopGroup 配置多个也无用。

## EventLoop

[参考](https://caorong.github.io/2016/12/24/head-first-netty-1/)

如果只使用 tcp 和 异步阻塞的话主要关心以下2个 EventLoop 

NioEventLoop - 基于java 原生nio

```
level-triggered （水平触发）
```

EpollEventLoop - native jni 直接调用 epoll, only work on linux

```
edge-triggered （边缘触发）更少的系统调用
C代码，更少GC，更少synchronized
暴露了更多的Socket配置参数
```

### 流程图

![img](../images/NettyEvent/2016-11-25-01-23-15.png)

关键点

1. 整个loop 干的事情就是 `select -> processIO -> runAllTask`
2. 这是一个死循环
3. 那么这个loop 如何自己优雅退出？ noway，只能通过外部添加 CloseTask， 比如添加到 MpscQueue
4. deadline 为 定时任务的触发时间，避免 select 阻塞, 让 定时任务不能及时执行。
5. 在select 这一步 解决 epollbug

关于解决 epoll bug的原理是 应当 “阻塞”的 select 变得不再阻塞。
所以只需要统计下 select 次数就行了

部分关键代码:

```java
for(;;){
    int selectedKeys = selector.select(timeoutMillis); // select with timeout
    selectCnt ++;
    // 我由于 select 阻塞 而等待了 timeoutMillis 毫秒， 说明， 我阻塞了，说明没有bug
    if (time - TimeUnit.MILLISECONDS.toNanos(timeoutMillis) >= currentTimeNanos) {
        selectCnt = 1;
    } else if (SELECTOR_AUTO_REBUILD_THRESHOLD > 0 &&
            selectCnt >= SELECTOR_AUTO_REBUILD_THRESHOLD) {
        // 在小于 timeoutMillis 毫秒的时间内 select 的次数超过了 阀值(512) 次
        rebuildSelector();
        selector = this.selector;

        selector.selectNow();// Select again
        selectCnt = 1;
        break;
    }
}
```

### other

#### Selector.wakeup()

java 的 Selector 在原生的 select api 之上 增加了个 Selector.wakeup()

她的目的是唤醒 阻塞在 `select()` 的线程, (通过写一个字节)

为什么要唤醒？ 什么时候需要唤醒？

```
1. 注册了新的 channel 或者事件。
2. channel 关闭， 取消注册。
3. 优先级更高的事件触发（如定时器事件）， 希望及时处理。
```

**原理**

Linux上利用pipe调用创建一个管道，Windows上则是一个loopback的tcp连接。这是因为win32的管道无法加入select的fd set，将管道或者TCP连接加入select fd set。

wakeup往管道或者连接写入一个字节，阻塞的select因为有I/O事件就绪，立即返回。可见，wakeup的调用开销不可忽视。

之前看到的 coolshell 也分析过 –> Java NIO类库Selector机制解析（[上](http://blog.csdn.net/haoel/archive/2008/03/27/2224055.aspx)，[下](http://blog.csdn.net/haoel/archive/2008/03/27/2224069.aspx)，[续](http://blog.csdn.net/haoel/archive/2008/05/04/2379586.aspx)）

### 类层次结构图

<img src="../images/NettyEvent/285763-20180207134611779-1494591619.png" alt="img" style="zoom:67%;" />

NioEventLoop 的类层次结构图还是比较复杂的, 不过我们只需要关注几个重要的点即可. 首先 NioEventLoop 的继承链如下:

```
NioEventLoop -> SingleThreadEventLoop -> SingleThreadEventExecutor -> AbstractScheduledEventExecutor
```

### 实例化过程

[参考](https://www.cnblogs.com/duanxz/p/3724395.html)

![img](../images/NettyEvent/285763-20180207144316310-262152843.png)

从上图可以看到, SingleThreadEventExecutor 有一个名为 **thread** 的 Thread 类型字段, 这个字段就代表了与 SingleThreadEventExecutor 关联的本地线程。在 SingleThreadEventExecutor 构造器中, 通过 **threadFactory.newThread** 创建了一个新的 Java 线程. 在这个线程中所做的事情主要就是调用 **SingleThreadEventExecutor.this.run()** 方法, 而因为 NioEventLoop 实现了这个方法, 因此根据多态性, 其实调用的是 **NioEventLoop.run()** 方法。

protected SingleThreadEventExecutor(
        EventExecutorGroup parent, ThreadFactory threadFactory, boolean addTaskWakesUp) {
    this.parent = parent;
    this.addTaskWakesUp = addTaskWakesUp;

```java
thread = threadFactory.newThread(new Runnable() {
    @Override
    public void run() {
        boolean success = false;
        updateLastExecutionTime();
        try {
            SingleThreadEventExecutor.this.run();
            success = true;
        } catch (Throwable t) {
            logger.warn("Unexpected exception from an event executor: ", t);
        } finally {
            // 省略清理代码
            ...
        }
    }
});
threadProperties = new DefaultThreadProperties(thread);
taskQueue = newTaskQueue();
}
```

### 两大功能

1、是作为 IO 线程, 执行与 Channel 相关的 IO 操作, 包括 调用 select 等待就绪的 IO 事件、读写数据与数据的处理等；

2、第二个任务是作为任务队列执行任务， 任务可以分为2类：

　　**2.1、普通task：**通过调用NioEventLoop的execute(Runnable task)方法往任务队列里增加任务，Netty有很多系统task，创建他们的主要原因是：当io线程和用户线程同时操作网络资源的时候，为了防止并发操作导致的锁竞争，将用户线程的操作封装成Task放入消息队列中，由i/o线程负责执行，这样就实现了**局部无锁化**。

　　**2.2、定时任务**：执行schedule()方法

依次看这些功能的源码吧：

**对比2.1：**上面2.1中的普通任务，NioEventLoop通过队列来保存任务。在 SingleThreadEventLoop 中, 又实现了任务队列的功能, 通过它, 我们可以调用一个 NioEventLoop 实例的 execute 方法来向任务队列中添加一个 task, 并由 NioEventLoop 进行调度执行。

```java
    protected Queue<Runnable> newTaskQueue() {
        return new LinkedBlockingQueue<Runnable>();
    }
```

添加任务的SingleThreadEventExecutor中的execute()方法：

[![复制代码](../images/NettyEvent/copycode.gif)](javascript:void(0);)

```
    @Override
    public void execute(Runnable task) {
        if (task == null) {
            throw new NullPointerException("task");
        }

        boolean inEventLoop = inEventLoop();
        if (inEventLoop) {
            addTask(task);
        } else {
            startThread();
            addTask(task);
            if (isShutdown() && removeTask(task)) {
                reject();
            }
        }

        if (!addTaskWakesUp) {
            wakeup(inEventLoop);
        }
    }
```

[![复制代码](../images/NettyEvent/copycode.gif)](javascript:void(0);)

上面2.1中的普通task的执行：

[![复制代码](../images/NettyEvent/copycode.gif)](javascript:void(0);)

```
    protected boolean runAllTasks() {
        fetchFromScheduledTaskQueue();
        Runnable task = pollTask();
        if (task == null) {
            return false;
        }

        for (;;) {
            try {
                task.run();
            } catch (Throwable t) {
                logger.warn("A task raised an exception.", t);
            }

            task = pollTask();
            if (task == null) {
                lastExecutionTime = ScheduledFutureTask.nanoTime();
                return true;
            }
        }
    }
```

[![复制代码](../images/NettyEvent/copycode.gif)](javascript:void(0);)

**对比2.2：**schedule（**定时**） 任务处理

除了通过 execute 添加普通的 Runnabl$e 任$务外, 我们还可以通过调用 eventLoop.scheduleXXX 之类的方法来添加一个定时任务。
EventLoop 中实现任务队列的功能在超类 `SingleThreadEventExecutor` 实现的, 而 schedule 功能的实现是在 `SingleThreadEventExecutor` 的父类, 即 `AbstractScheduledEventExecutor` 中实现的。
在 `AbstractScheduledEventExecutor` 中, 有以 scheduledTaskQueue 字段:

```
Queue<ScheduledFutureTask<?>> scheduledTaskQueue;
```

scheduledTaskQueue 是一个队列(Queue), 其中存放的元素是 **ScheduledFutureTask**. 而 **ScheduledFutureTask** 我们很容易猜到, 它是对 Schedule 任务的一个抽象。我们来看一下 `AbstractScheduledEventExecutor` 所实现的 **schedule** 方法吧：

[![复制代码](../images/NettyEvent/copycode.gif)](javascript:void(0);)

```
@Override
public  ScheduledFuture<?> schedule(Runnable command, long delay, TimeUnit unit) {
    ObjectUtil.checkNotNull(command, "command");
    ObjectUtil.checkNotNull(unit, "unit");
    if (delay < 0) {
        throw new IllegalArgumentException(
                String.format("delay: %d (expected: >= 0)", delay));
    }
    return schedule(new ScheduledFutureTask<Void>(
            this, command, null, ScheduledFutureTask.deadlineNanos(unit.toNanos(delay))));
}
```

[![复制代码](../images/NettyEvent/copycode.gif)](javascript:void(0);)

这是其中一个重载的 schedule, 当一个 Runnable 传递进来后, 会被封装为一个 **ScheduledFutureTask** 对象, 这个对象会记录下这个 Runnable 在何时运行、已何种频率运行等信息。
当构建了 **ScheduledFutureTask** 后, 会继续调用 另一个重载的 schedule 方法:

[![复制代码](../images/NettyEvent/copycode.gif)](javascript:void(0);)

```
<V> ScheduledFuture<V> schedule(final ScheduledFutureTask<V> task) {
    if (inEventLoop()) {
        scheduledTaskQueue().add(task);
    } else {
        execute(new OneTimeTask() {
            @Override
            public void run() {
                scheduledTaskQueue().add(task);
            }
        });
    }

    return task;
}
```

[![复制代码](../images/NettyEvent/copycode.gif)](javascript:void(0);)

在这个方法中, ScheduledFutureTask 对象就会被添加到 **scheduledTaskQueue** 中了。

#### 任务的执行

当一个任务被添加到 taskQueue 后, 它是怎么被 EventLoop 执行的呢?
让我们回到 NioEventLoop.run() 方法中, 在这个方法里, 会分别调用 **processSelectedKeys()** 和 **runAllTasks()** 方法, 来进行 IO 事件的处理和 task 的处理. **processSelectedKeys()** 方法我们已经分析过了, 下面我们来看一下 **runAllTasks()** 中到底有什么名堂吧。
runAllTasks 方法有两个重载的方法, 一个是无参数的, 另一个有一个参数的. 首先来看一下无参数的 runAllTasks：

```
protected boolean runAllTasks() {
    fetchFromScheduledTaskQueue();
    Runnable task = pollTask();
    if (task == null) {
        return false;
    }

    for (;;) {
        try {
            task.run();
        } catch (Throwable t) {
            logger.warn("A task raised an exception.", t);
        }

        task = pollTask();
        if (task == null) {
            lastExecutionTime = ScheduledFutureTask.nanoTime();
            return true;
        }
    }
}
```

我们前面已经提到过, EventLoop 可以通过调用 **EventLoop.execute** 来将一个 Runnable 提交到 taskQueue 中, 也可以通过调用 **EventLoop.schedule** 来提交一个 schedule 任务到 **scheduledTaskQueue** 中。在此方法的一开始调用的 **fetchFromScheduledTaskQueue()** 其实就是将 **scheduledTaskQueue** 中已经可以执行的(即定时时间已到的 schedule 任务) 拿出来并添加到 taskQueue 中, 作为可执行的 task 等待被调度执行。
它的源码如下:

```
private void fetchFromScheduledTaskQueue() {
    if (hasScheduledTasks()) {
        long nanoTime = AbstractScheduledEventExecutor.nanoTime();
        for (;;) {
            Runnable scheduledTask = pollScheduledTask(nanoTime);
            if (scheduledTask == null) {
                break;
            }
            taskQueue.add(scheduledTask);
        }
    }
}
```

接下来 **runAllTasks()** 方法就会不断调用 **task = pollTask()** 从 **taskQueue** 中获取一个可执行的 task, 然后调用它的 **run()** 方法来运行此 task。

> `注意`, 因为 EventLoop 既需要执行 IO 操作, 又需要执行 task, 因此我们在调用 EventLoop.execute 方法提交任务时, 不要提交耗时任务, 更不能提交一些会造成阻塞的任务, 不然会导致我们的 IO 线程得不到调度, 影响整个程序的并发量.

## NIO EventLoopGroup与Reactor线程模型的关系

Netty的线程模型并不是一成不变的，实际通过用户的启动配置参数来配置。

![img](../images/NettyEvent/285763-20180206174951748-683754909.png)

 在创建ServerBootstrap类实例前，先创建两个EventLoopGroup，它们实际上是两个独立的Reactor线程池，bossGroup负责接收客户端的连接，workerGroup负责处理IO相关的读写操作，或者执行系统task、定时task等。

用于接收客户端请求的线程池职责如下：

1. 接收客户端TCP连接，初始化Channel参数；
2. 将链路状态变更事件通知给ChannelPipeline；

处理IO操作的线程池职责如下：

1. 异步读取远端数据，发送读事件到ChannelPipeline；
2. 异步发送数据到远端，调用ChannelPipeline的发送消息接口；
3. 执行系统调用Task；
4. 执行定时任务Task，如空闲链路检测等；

通过调整两个EventLoopGroup的线程数、是否共享线程池等方式，Netty的Reactor线程模型可以在单线程、多线程和主从多线程间切换，用户可以根据实际情况灵活配置。 　

为了提高性能，Netty`在`很多地方采用了无锁化设计。例如在IO线程的内部进行串行操作，避免多线程竞争导致的性能下降。尽管串行化设计看上去CPU利用率不高，并发程度不够，但是通过调整NIO线程池的线程参数，可以同时启动多个串行化的线程并行运行，这种局部无锁化的设计相比一个队列——多个工作线程的模型性能更优。

它的设计原理如下

![img](../images/NettyEvent/285763-20180206222504732-771250884.png)

Netty的NioEventLoop读取到消息之后，调用ChannelPipeline的fireChannelRead方法，只要用户不主动切换线程，就一直由NioEventLoop调用用户的Handler，期间不进行线程切换。这种串行化的处理方式避免了多线程操作导致的锁竞争，从性能角度看是最优的。

 Netty多线程编程的最佳实践如下： 

1. 服务端创建两个EventLoopGroup，用于逻辑隔离NIO acceptor和NIO IO线程；
2. 尽量避免在用户Handler里面启动用户线程（解码后将POJO消息发送到后端业务线程除外）；
3. 解码要在NIO线程调用的解码Handler中进行，不要切换到用户线程中完成消息的解码；
4. 如果业务逻辑比较简单，没有复杂的业务逻辑计算，没有可能阻塞线程的操作如磁盘操作、数据库操作、网络操作等，可以直接在NIO线程中进行业务逻辑操作，不用切换到用户线程；
5. 如果业务逻辑比较复杂，不要在NIO线程上操作，应将解码后的POJO封装成Task提交到业务线程池中执行，以保证NIO线程被尽快释放，处理其他IO操作；

## EventLoopGroup

类层次结构图

<img src="../images/NettyEvent/285763-20180207114810685-668084313.png" alt="img" style="zoom: 67%;" />

EventLoopGroup (其实是MutithreadEvnentExecutorGroup) 内部维护一个类型为EventExecutor chidren数组, 

- 从类结构可知，NioEventLoopGroup是一个Schedule类型的线程池，线程池中的线程用数组存放，
  
    EventLoopGroup(其实是MultithreadEventExecutorGroup) 内部维护一个类型为 EventExecutor children 数组, 其大小是 nThreads, 这样就构成了一个线程池，线程池大小通过
  
    在实例化 NioEventLoopGroup 时, 如果指定线程池大小, 则 nThreads 就是指定的值, 反之是处理器核心数 * 2;

- MultithreadEventExecutorGroup 中会调用 newChild 抽象方法(抽象方法 newChild 是在 NioEventLoopGroup 中实现的, 它返回一个 NioEventLoop 实例)来初始化 children 数组,也是**在NioEventLoopGroup中调用NioEventLoop的构造函数来创建NioEventLoop**；

```java
 @Override
    protected EventLoop newChild(Executor executor, Object... args) throws Exception {
        return new NioEventLoop(this, executor, (SelectorProvider) args[0]);
    }
```

#### 主从线程池

![image-20210921204938232](../images/NettyEvent/image-20210921204938232.png)
