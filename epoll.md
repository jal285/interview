# epoll

## epoll的用途;

epoll用来替换select，epoll最大的好处是它不会随着监听fd 数目的增长而降低效率。在内核中的select视线中，它是采用轮询来处理的，轮询的fd数目越多， 自然耗时越多

fd 代表发生事件的文件描述符

epoll是Linux 特有的I/O 服用函数。它在实现和使用上与select、poll有很大差异。首先，epoll使用一组函数来完成任务，而不是单个函数。其次，epoll把用户关心的文件描述符上的事件放在内核里的一个事件表中，从而无需像select和poll那样每次调用都要重复传入文件描述符集或事件集。但epoll需要使用一个额外的文件描述符，来唯一标识内核中的这个事件表。这个文件描述符使用epoll\_create函数来创建。

    //通过拷贝的方式，每次循环都需要重复传入文件描述符集
    nready=poll(client,maxi+1,-1);

<!---->

    //用户关心的文件描述符上的事件放在内核里的一个事件表中，通过epollfd标识
    //这个事件表
    nready=epoll_wait(epollfd,&*events.begin(),static_cast<int>(events.size()),-1);

epoll\_create函数

int epoll\_create(int size);

作用：

该函数生成一个epoll专用的文件描述符，返回的文件描述符将用作其它所有epoll系统调用的第一个参数，以指定要访问的内核事件表。

参数：

size参数现在并不起作用，只是给内核一个提示，告诉它事件表需要多大。（曾经表示在这个epoll fd上能关注的最大fd数）

epoll\_ctl函数

int epoll\_ctl(int epfd, int op, int fd, struct epoll\_event \*event)

作用：

该函数用于控制某个epoll文件描述符上的事件，可以注册事件，修改事件，删除事件。

参数：

epfd：由 epoll\_create 生成的epoll专用的文件描述符；
op：要进行的操作例如注册事件，可能的取值EPOLL\_CTL\_ADD 注册、EPOLL\_CTL\_MOD修改、EPOLL\_CTL\_DEL 删除
fd：关联的文件描述符，加入epoll\_ctl进行管理

event：事件，对文件描述符所感兴趣的事件，指向epoll\_event的指针；

返回值

如果调用成功返回0,不成功返回-1

说明

    typedef union epoll_data {
    void *ptr;
    int fd;
    __uint32_t u32;
    __uint64_t u64;
    } epoll_data_t;

    struct epoll_event {
    __uint32_t events; //感兴趣的事件是可读还是可写
    epoll_data_t data; /* User data variable */这是epoll高效的地方，数据类型是上面的那个共用体   
    };

events可以是以下几个宏的集合：

EPOLLIN：触发该事件，表示对应的文件描述符上有可读数据。(包括对端SOCKET正常关闭)；
EPOLLOUT：触发该事件，表示对应的文件描述符上可以写数据；
EPOLLPRI：表示对应的文件描述符有紧急的数据可读（这里应该表示有带外数据到来）；
EPOLLERR：表示对应的文件描述符发生错误；
EPOLLHUP：表示对应的文件描述符被挂断；
EPOLLET： 将EPOLL设为边缘触发(Edge Triggered)模式，这是相对于水平触发(Level Triggered)来说的。
EPOLLONESHOT：只监听一次事件，当监听完这次事件之后，如果还需要继续监听这个socket的话，需要再次把这个socket加入到EPOLL队列里。
epoll\_wait函数

int epoll\_wait(int epfd, struct epoll\_event \* events, int maxevents, int timeout);

说明：

epoll\_wait函数如果检测到事件，就将所有就绪的事件从内核事件表（由epfd参数指定）中复制它的第二个参数events指向的数组中。这个数组只用于输出epoll\_wait检测到的就绪事件，而不像select和poll的数组参数那样既用于传入用户注册的事件，又用于输出内核检测到的就绪事件。这就极大地提高了应用程序索引就绪文件描述符的效率。

参数：
epfd:由epoll\_create 生成的epoll专用的文件描述符；
epoll\_event:用于回传待处理事件的数组；
maxevents:每次能处理的事件数；

timeout:等待I/O事件发生的超时值；-1相当于阻塞，0相当于非阻塞。一般用-1即可返回发生事件数。

返回值：

如果函数调用成功，返回对应I/O上已准备好的文件描述符数目

## epoll关键

epoll 是一种IO多路复用技术，可以非常高效的处理数以百万计的socket句柄，比起select和poll效率高很多。

首先要调用epoll\_create建立一个epoll对象。参数size是内核保证能够正确处理的最大句柄数，多于这个最大数时内核可不保证效果。epoll\_ctl可以操作上面建立的epoll，可以添加、移除socket句柄。epoll\_wait在调用时，在给定的timeout时间内，当在监控的所有句柄中有事件发生时，就返回。

> Epoll高效主要体现在以下三个方面：
>
> ①从上面的调用方式就可以看出epoll比select/poll的一个优势：select/poll每次调用都要传递所要监控的所有fd给select/poll系统调用（这意味着每次调用都要将fd列表从用户态拷贝到内核态，当fd数目很多时，这会造成低效）。而每次调用epoll\_wait时（作用相当于调用select/poll），不需要再传递fd列表给内核，因为已经在epoll\_ctl中将需要监控的fd告诉了内核（epoll\_ctl不需要每次都拷贝所有的fd，只需要进行增量式操作）。所以，在调用epoll\_create之后，内核已经在内核态开始准备数据结构存放要监控的fd了。每次epoll\_ctl只是对这个数据结构进行简单的维护。
>
> ② 此外，内核使用了slab机制，为epoll提供了快速的数据结构：
>
> 在内核里，一切皆文件。所以，epoll向内核注册了一个文件系统，用于存储上述的被监控的fd。当你调用epoll\_create时，就会在这个虚拟的epoll文件系统里创建一个file结点。当然这个file不是普通文件，它只服务于epoll。epoll在被内核初始化时（操作系统启动），同时会开辟出epoll自己的内核高速cache区，用于安置每一个我们想监控的fd，这些fd会以红黑树的形式保存在内核cache里，以支持快速的查找、插入、删除。这个内核高速cache区，就是建立连续的物理内存页，然后在之上建立slab层，简单的说，就是物理上分配好你想要的size的内存对象，每次使用时都是使用空闲的已分配好的对象。
>
> ③ epoll的第三个优势在于：当我们调用epoll\_ctl往里塞入百万个fd时，epoll\_wait仍然可以飞快的返回，并有效的将发生事件的fd给我们用户。这是由于我们在调用epoll\_create时，内核除了帮我们在epoll文件系统里建了个file结点，在内核cache里建了个红黑树用于存储以后epoll\_ctl传来的fd外，还会再建立一个list链表，用于存储准备就绪的事件，当epoll\_wait调用时，仅仅观察这个list链表里有没有数据即可。有数据就返回，没有数据就sleep，等到timeout时间到后即使链表没数据也返回。所以，epoll\_wait非常高效。而且，通常情况下即使我们要监控百万计的fd，大多一次也只返回很少量的准备就绪fd而已，所以，epoll\_wait仅需要从内核态copy少量的fd到用户态而已。那么，这个准备就绪list链表是怎么维护的呢？当我们执行epoll\_ctl时，除了把fd放到epoll文件系统里file对象对应的红黑树上之外，还会给内核中断处理程序注册一个回调函数，告诉内核，如果这个fd的中断到了，就把它放到准备就绪list链表里。所以，当一个fd（例如socket）上有数据到了，内核在把设备（例如网卡）上的数据copy到内核中后就来把fd（socket）插入到准备就绪list链表里了。
>
> 如此，一颗红黑树，一张准备就绪fd链表，少量的内核cache，就帮我们解决了大并发下的fd（socket）处理问题。
>
> 1.执行epoll\_create时，创建了红黑树和就绪list链表。
>
> 2.执行epoll\_ctl时，如果增加fd（socket），则检查在红黑树中是否存在，存在立即返回，不存在则添加到红黑树上，然后向内核注册回调函数，用于当中断事件来临时向准备就绪list链表中插入数据。
>
> 3.执行epoll\_wait时立刻返回准备就绪链表里的数据即可。

最后看看epoll独有的两种模式LT和ET。无论是LT和ET模式，都适用于以上所说的流程。区别是，LT模式下，只要一个句柄上的事件一次没有处理完，会在以后调用epoll\_wait时次次返回这个句柄，而ET模式仅在第一次返回。
这件事怎么做到的呢？当一个socket句柄上有事件时，内核会把该句柄插入上面所说的准备就绪list链表，这时我们调用epoll\_wait，会把准备就绪的socket拷贝到用户态内存，然后清空准备就绪list链表，最后，epoll\_wait干了件事，就是检查这些socket，如果不是ET模式（就是LT模式的句柄了），并且这些socket上确实有未处理的事件时，又把该句柄放回到刚刚清空的准备就绪链表了。所以，非ET的句柄，只要它上面还有事件，epoll\_wait每次都会返回。而ET模式的句柄，即使socket上的事件没有处理完，也是不会次次从epoll\_wait返回的。
