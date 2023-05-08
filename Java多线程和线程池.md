# 关于多线程实战（并发）

<style>
     img{
        box-shadow:  0px 0px 10px  rgba(0,0,0,.5);
                 /*图片添加阴影*/ 
    }
</style>

## 多线程

 使用多线程的目的在于提升效率 ，但同时也应该保证线程安全，所以在使用多线程的时候应该保证线程同步 

> 线程同步是为了确保线程安全，所谓线程安全指的是多个线程对同一资源进行访问时，有可能产生数据不一致问题，导致线程访问的资源并不是安全的。如果多线程序运行结果和单线程运行的结果是一样的，且相关变量的值与预期值一样，则是线程安全的。

### 线程同步

1. **临界区**（Critical section）：通过对多线程的**串行化**来访问公共资源或一段代码，速度快，适合控制数据访问。在任何时候只允许一个线程访问共享资源，如果有多个线程访问，那么当有一个线程进入后，其他试图访问共享资源的线程将会被挂起，并且等到进入临界区的线程离开，临界在被释放后，其他线程才可以抢占。
2. **互斥量**（Mutex）：为协调对一个共享资源的单独访问而设计。互斥量只有一个，只有拥有互斥量的线程，才有权限去访问系统的公共资源，保证资源不会被多个线程访问。互斥不仅能实现同一个应用程序的公共资源安全共享，还能实现不同应用程序的公共资源安全共享。  比如 Java 中的 synchronized 关键词和各种 Lock 都是这种机制。  
3. **信号量**（Semphore）：为控制一个具有有限数量的用户资源而设计。它允许多个线程在同一时刻去访问同一个资源，但一般需要限制同一时刻访问此资源的最大线程数目。
4. **事件**(Event)：用来通知线程有一些事件已发生，从而启动后继任务的开始。

### 创建多线程--继承Thread类和实现Runnable接口的区别

 java中我们想要实现多线程常用的有两种方法，继承Thread 类和实现Runnable 接口

java是单继承，所以在继承Thread类的时候需要创建实例对象去完成任务，在创建多个对象时，虽然同时创建了等量的线程，但各自线程完成的任务数量是相等的

而实现Runnable的时候，是将一定量的任务交给多个对象和线程去共同完成，即实例化多个Thread，创建多个Thread去共同执行任务

一个类只能继承一个父类，存在局限；一个类可以实现多个接口。在实现Runnable接口的时候调用Thread的Thread(Runnable run)或者Thread(Runnablerun,String name)构造方法创建进程时，使用同一个Runnable实例，建立的多线程的实例变量也是共享的；但是通过继承Thread类是不能用一个实例建立多个线程，故而实现Runnable接口适合于资源共享；当然，继承Thread类也能够共享变量，能共享Thread类的static变量；  

抽象来讲，可以看作接口和继承的问题

#### 继承Thread 类

```java
package test;
/**
 * 继承thread类
 * @author hongxin
 *
 */
public class testThread extends Thread{
    private int count=5;
    private String name;
    public testThread(String name) {
       this.name=name;
    }
    public void run() {
        for (int i = 0; i < 5; i++) {
            System.out.println(name + "运行  count= " + count--);
            try {
                sleep((int) Math.random() * 10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

    }
}

package test;

public class main1 {
    public static void main(String[] args) {
        testThread mTh1=new testThread("A");
        testThread mTh2=new testThread("B");
        mTh1.start();
        mTh2.start();
    }
}
```

![image-20210317003814657](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210317003814657.png)

线程1和2之间的变量是不能共享的，每次count--都有各自的变量和结果

#### Runnable接口

```java
package test;

/**
 * 实现runnable接口
 * @author hongxin
 *
 */
public class testRunnable implements Runnable{
    private int count=15;
    @Override
    public void run() {
          for (int i = 0; i < 5; i++) {
              System.out.println(Thread.currentThread().getName() + "运行  count= " + count--);
                try {
                    Thread.sleep((int) Math.random() * 10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
    }

}


package test;

public class main2 {
    public static void main(String[] args) {
        testRunnable mTh = new testRunnable();
            new Thread(mTh, "C").start();//同一个mTh，但是在Thread中就不可以，如果用同一个实例化对象mt，就会出现异常   
            new Thread(mTh, "D").start();
            new Thread(mTh, "E").start();
    }

}
```

![image-20210317004039162](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210317004039162.png)

三个不同的线程之间的变量是共享的，每次count--得到的都是上一个线程运行结果运算后的

### 线程创建和线程终止

每个进程有一个进程ID ,每个线程也有一个线程ID, 进程ID在整个系统是唯一的, 但线程ID不同, 线程ID 是只有在它所属的进程上下文中才有意义

进程ID 使用pid_t数据类型来表示的, 是一个非负整数 ,线程ID是用pthread_t 数据类型来表示的, 实现的时候可以用一个结构来表示pthread_t 数据类型 , 所以可医治的操作系统实现不能把它作为整数处理. 因此必须使用一个函数来对两个线程ID进行比较 

```c
#include <pthread.h>
int pthread_equal(pthread_t tidl,pthread tid2);
                                                            返回值: 若相等, 返回非0数值; 否则,返回0
```

![image-20210601091348548](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210601091348548.png)

用结构表示pthread_t 数据类型的后果是不能用同一种可移植的方式打印该数据类型的值, 在程序调试过程中打印线程ID是非常有用的, 而在其他情况下通常不需要打印线程ID, 最坏的情况是, 有可能出现不可移植的调试代码, 

线程可以通过调用pthread_self函数获得自身的线程ID 

```c
#include<pthread.h>
pthread_t pthread_self(void);
//返回值: 调用线程的进程ID
```

当线程需要识别以线程ID 作为标识的数据结构式, pthread_self 函数可以与pthread_equal 函数一起使用, 例如, 主线程可能把工作任务放在一个队列中, 用线程ID 来控制每个工程线程处理那些作业, 如图11-1 所示, 主线陈把新的作业放道一个工作队列中, 由3个工作线程组成的线程池从队列中移出作业.  主线程不允许每个线程任意处理从队列顶端取出的作业,而是由主线程控制作业的分配 , 主线程会在每个待处理作业的结构中放置处理改作业的线程ID, 每个工作线程智能移出标有自己线程ID 的作业

![image-20210601114822588](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210601114822588.png)

#### 线程创建

![image-20210601134314961](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210601134314961.png)

当pthread_create成功返回时, 新创建线程的线程ID 会被设置成tidp指向的内存单元 

attr 参数用于定制各种不同的线程属性 . 现在我们将它设置为NULL, 创建一个具有默认属性的线程 

```java
#include "apue.h"
#include <pthread.h>

void printids(const char *s){
        pid_t pid;
        pthread_t tid;

        pid = getpid();
        tid = pthread_self();
        printf("%s pid %lu tid %lu (0x%lx) \n",s, (unsigned    long) pid, (unsigned long) tid, (unsigned long) tid);
}


void * thr_fn(void *arg){
    printids("new thread: ");
    return ((void *)0 );
}
int main(void ){
    int err;
    err = pthread_create(&ntid, NULL,thr_fn,NULL);
    if(err!=0)
        err)exut(err,"can't create thread");
        printids("main thread: ");
        sleep(1);
        exit(0);
}
```

![image-20210601141223032](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210601141223032.png)

![image-20210601141237185](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210601141237185.png)

![image-20210601141255718](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210601141255718.png)

Java 中有三种线程创建方式, 分别为Runnable 接口的run方法, 继承Tread类并重写run的方法, 使用FutureTask 方式 

其中前两种方法, 任务没有返回值 , 使用FutureTask 则可以带有返回值, 即任务函数可以定义 string 等类别 

下面看三种实现

![image-20210606083951302](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210606083951302.png)

注意: 创建完thread对象后该线程并没有被启动执行, 直到调用start 方法后才真正启动了线程 

其实调用start 方法后线程并没有马上执行而是处于就绪状态, 这个就绪状态是指该线程已经获取除CPU 资源外的其他资源, 等待获取CPU资源后才会真正处于运行状态 . 一旦run方法执行完毕, 该线程就处于终止状态

![image-20210606084714737](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210606084714737.png)

![image-20210606084733172](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210606084733172.png)

![image-20210606084751103](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210606084751103.png)

如上代码中的CallerTask 类实现了Callable 接口的call() 方法. 在main函数内首先创建了一个FutrueTask 对象(构造函数为CallerTask 的实例) , 然后使用创建的FutureTask 对象作为任务创建了一个线程并且启动它, 最后通过futureTask.get() 等待任务执行完毕并返回结果 

小结: 使用继承方式的好处是方便传参, 你可以在子类里面添加成员变量, 通过set方法设置参数或者通过构造函数进行传递, 而如果使用Runnable方式, 则只能使用主线程里面被声明为final的变量, 不好的方法是Java不支持多继承, 如果继承了Thread类, 那么子类不能再继承其他类, Runnable则没有这个限制. 前两种方式都没办法拿到任务的返回结果, 但是Futuretask方式可以 .

#### 线程终止

如果进程中的任意线程调用了exit、_eixt，那么整个进程就会终止。 与此相类似， 如果默认的动作是终止进程， 那么，发送到线程的信号就会终止整个进程

单个线程可以通过3种方式退出，因此可以在不终止整个进程的情况下，停止它的控制流

![image-20210601153956246](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210601153956246.png)

![image-20210601154004277](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210601154004277.png)

![image-20210601154011163](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210601154011163.png)

![image-20210601154023940](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210601154023940.png)

可以看到， 当一个线程通过调用pthread_exit退出或者简单地从启动例中返回时，进程中的其它线程可以通过调用pthread_join函数获得该线程的退出状态

pthread_create 和 pthread_exit含函数的无类型指针参数可以传递的值不止一个，这个指针可以传递包含复杂信息的结构的地址，但是注意，这个结构所使用的内存在调用者完成调用以后必须仍然是有效的 。例如， 在调用线程的栈上分配了该结构，那么其他的线程在使用这个结构时内存内容可能已经改变了。又如，线程在自己的栈上分配了一个结构，然后把指向这个结构的指针传给pthread_exit，那么调用pthread_join的线程试图使用该结构时，这个栈可能已经被撤销， 这块内存也已另作他用 

第二个线程中的内容可能会覆盖第一个线程中的内容， 

![image-20210601164532483](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210601164532483.png)

![image-20210601164542688](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210601164542688.png)

![image-20210601164559657](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210601164559657.png)

![image-20210601164609353](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210601164609353.png)

![image-20210601164630645](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210601164630645.png)

> 以上图片来自《java并发编程之美》

#### 线程清理处理程序

```c
#include "apue.h"
#include <pthread.h>

void cleanup(void *arg){
    printf("cleanup: %s\n",(char*)arg);
}

void * thr_fn1(void *arg){
    printf("thread 1 start\n");
    pthread_cleanup_push(cleanup,"thread 1 first handler");
    pthread_cleanup_push(cleanup,"thread 1 second handler");
    printf("thread 1 push complete\n");
    if(arg) return((void*)1);
    pthread_cleanup_pop(0);
    pthread_cleanup_pop(0);
}

void * thr_fn2(void * arg)
{
    printf（"thread 2  start\n");
    pthread_cleanup_push(cleanup,"thread 2 first handler");
    pthread_cleanup_push(cleanup,"thread 2 second handler");    
    printf("thread 2 ");
    if(arg) pthread_exit((void *)2);
    pthread_cleanup_pop(0);
    pthread_coeanup_pop(0);
    pthread_exit((void* )2);    
}

int main(void ){
    int err;
    pthread_t    tidl;
    void         *tret;

    err = pthread_create(&tidl,NULL,thr_fnl,(void*) 1);
    if(err != 0)
        err_exit(err, "can't create thread 1 ");
    err = pthread_create(&tid2,NULL,thr_fn1,(void*)1);
    if(err != 0)
        err_eixt(err,"can't create thread 2 ");
    err = pthread_join(tid1,&tret);
    if(err != 0)
        err_eixt(err,"can't join with thread 1 ");
    printf("thread 1 exit code %ld\n",(long)tret );
    err = pthread_join(tid2,&tret);
    if(err != 0)
        err_eixt(err,"can't join with thread 2 ");
    printf("thread 2 exit code %ld\n",(long)tret );
    exit(0);
}   // 图 11-5 线程清理处理程序
```

![image-20210601215621394](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210601215621394.png)

从输出结果可以看到两个线程都正确地启动和退出了， 但是只有第二个线程的清除处理程序被调用了， 因此，如果线程是    通过它的启动例程中返回而终止的话，它的清理程序就不会被调用 。还要注意， 清理处理程序是按照与它们安装时相反的顺序被调用的 。 ![image-20210601220343888](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210601220343888.png)

### wait 和 waitpid

当一个进程正常或异常终止时, 内核就向其父进程发送SIGCHLD 信号 , 因为子进程终止是个异步事件(这可以在父进程运行的任何时候发生) , 所以这种信号也是内核向父进程发的异步通知 . 父进程可以选择忽略该信号 ,或者提供一个该信号发生时即被调用执行的函数(信号处理程序). 对于这种信号的系统默认动作是忽略它 

调用wait 和waitpid 进程可能会发生什么 

- 如果所有子进程都还运行 , 则阻塞
- 如果一个进程已种种, 正等待父进程获取其终止状态, 则取得该子进制的终止状态立即返回
- 如果它没有任何子进程, 则立即出错返回

![image-20210531155708053](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210531155708053.png)

![image-20210531155715264](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210531155715264.png)

POSIX 定义了waitpid函数以提供 :如果有一个进程有几个子进程,如果要等待一个指定的进程终止, wait就返回

witpid 函数中 pid参数的作用解释如下 

![image-20210531215559088](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210531215559088.png)

waitpid 函数返回终止子进程的进程ID

witipid 函数返回终止子进程的进程ID,并将该子进程的状态村法国在由static 指向的存储单元中 . 对于wait . 其唯一的出错是调用进程没有子进程(函数调用被一个信号中断时, 也可能返回另一种出错 ) 

options 参数使我们能进一步控制waitpid 操作. 此参数或者是0, 或者是图 8-7 中常量换位或运算的结果 

![image-20210531221008058](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210531221008058.png)

 僵死进程 , 如果一个进程fork 一个子进程 ,但不要它等待子进程终止, 也不希望子进程处于僵死进程直到父进程终止, 实现这一要求的诀窍需要fork两次

![image-20210531222659213](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210531222659213.png)

![image-20210531222727325](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210531222727325.png)

## 线程池

属于并发包中java.util.concurrent  ，在Java中使用多线程操作时一般使用自定义线程池进行多线程的创建，方便使用和管理 

### 介绍

线程池主要解决两个问题:

- 执行大量异步任务时线程池能够提供较好的性能. 在不使用线程池时,每当需要执行异步任务时直接new一个线程来运行,而线程的创建和销毁时需要开销的. 线程池里面提供的线程是可复用的, 不需要每次执行异步任务时都重新创建和销毁线程. 

- 线程池提供了一种资源限制和管理的手段, 比如可以限制线程的个数, 动态新增线程等, 每个ThreadPoolExecutor 也保留了一些基本的统计数据, 比如当前线程池完成的任务数目等
  
    除此之外, 线程池也提供了许多可调参数和可扩展性接口, 以满足不同情景的需要, 我们可以使用更方便的Executors的工厂方法, 比如newCachedThreadPool(线程池线程个数最多可达Integer.MAX_VALUE. 线程自动回收) , newFixedThreadPool(固定大小的线程池) 和newSingleThreadExecutor(单个线程)等来创建线程池 , 也可以自定义线程池 

### 类图

下面类图中 Executors其实是个工具类, 里面提供了很多静态方法, 这些方法根据用户选择返回不同的线程池实例. ThreadPoolExecutor

继承了AbstractExecutorService , 成员变量ctl 是一Integer的原子变量, 用来记录线程池状态和线程池中线程个数, 类似于ReentrantReadWriteLock 使用一个变量来保存两种信息 

<img src="../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210624094104678.png" alt="image-20210624094104678" style="zoom:200%;" />

### 源码

```java
/**
     * Creates a new {@code ThreadPoolExecutor} with the given initial
     * parameters and default thread factory and rejected execution handler.
     * It may be more convenient to use one of the {@link Executors} factory
     * methods instead of this general purpose constructor.
     *
     * @param corePoolSize the number of threads to keep in the pool, even
     *        if they are idle, unless {@code allowCoreThreadTimeOut} is set
     * @param maximumPoolSize the maximum number of threads to allow in the
     *        pool
     * @param keepAliveTime when the number of threads is greater than
     *        the core, this is the maximum time that excess idle threads
     *        will wait for new tasks before terminating.
     * @param unit the time unit for the {@code keepAliveTime} argument
     * @param workQueue the queue to use for holding tasks before they are
     *        executed.  This queue will hold only the {@code Runnable}
     *        tasks submitted by the {@code execute} method.
     * @throws IllegalArgumentException if one of the following holds:<br>
     *         {@code corePoolSize < 0}<br>
     *         {@code keepAliveTime < 0}<br>
     *         {@code maximumPoolSize <= 0}<br>
     *         {@code maximumPoolSize < corePoolSize}
     * @throws NullPointerException if {@code workQueue} is null
     */
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), defaultHandler);
    }
```

请明白上述参数值, 参考注释 

1.**corePoolSize**：the number of threads to keep in the pool, even if they are idle, unless {@code allowCoreThreadTimeOut} is set

（核心线程数大小：不管它们创建以后是不是空闲的。线程池需要保持 corePoolSize 数量的线程，除非设置了 allowCoreThreadTimeOut。）

2.**maximumPoolSize**：the maximum number of threads to allow in the pool。

（最大线程数：线程池中最多允许创建 maximumPoolSize 个线程。）

3.**keepAliveTime**：when the number of threads is greater than the core, this is the maximum time that excess idle threads will wait for new tasks before terminating。

（存活时间：如果经过 keepAliveTime 时间后，超过核心线程数的线程还没有接受到新的任务，那就回收。）

4.**unit**：the time unit for the {@code keepAliveTime} argument

（keepAliveTime 的时间单位。）

5.**workQueue**：the queue to use for holding tasks before they are executed. This queue will hold only the {@code Runnable} tasks submitted by the {@code execute} method。

（存放待执行任务的队列：当提交的任务数超过核心线程数大小后，再提交的任务就存放在这里。它仅仅用来存放被 execute 方法提交的 Runnable 任务。所以这里就不要翻译为工作队列了，好吗？不要自己给自己挖坑。）

6.**threadFactory**：the factory to use when the executor creates a new thread。

（线程工程：用来创建线程工厂。比如这里面可以自定义线程名称，当进行虚拟机栈分析时，看着名字就知道这个线程是哪里来的，不会懵逼。）

7.**handler** ：the handler to use when execution is blocked because the thread bounds and queue capacities are reached。

（拒绝策略：当队列里面放满了任务、最大线程数的线程都在工作时，这时继续提交的任务线程池就处理不了，应该执行怎么样的拒绝策略。）

7 个参数介绍完了，我希望当面试官问你自定义线程池可以指定哪些参数的时候，你能回答的上来。

[参考](https://blog.csdn.net/xuemengrui12/article/details/78543543)

Executors

```java
Executor：一个接口，其定义了一个接收Runnable对象的方法executor，其方法签名为executor(Runnable command)
ExecutorService：是一个比Executor使用更广泛的子类接口，其提供了生命周期管理的方法，以及可跟踪一个或多个异步任务执行状况返回Future的方法
AbstractExecutorService：ExecutorService执行方法的默认实现
ScheduledExecutorService：一个可定时调度任务的接口
ScheduledThreadPoolExecutor：ScheduledExecutorService的实现，一个可定时调度任务的线程池
ThreadPoolExecutor：表示一个线程池，可以通过调用Executors的静态工厂方法来创建一个拥有特定功能的线程池并返回一个ExecutorService对象
```

### 几个线程池策略

1. 固定线程池执行程序
   
    –创建一个线程池，该线程池可重用固定数量的线程来执行任意数量的任务。如果在所有线程都处于活动状态时提交了其他任务，则它们将在队列中等待，直到某个线程可用为止。它最适合大多数现实生活中的用例。
   
   ```jva
   `ThreadPoolExecutor executor = (ThreadPoolExecutor) Executors.newFixedThreadPool(``10``);`
   ```

2. 缓存线程池执行程序
   
    –创建一个线程池，该线程池可根据需要创建新线程，但在可用时将重用以前构造的线程。如果任务长时间运行，请勿使用此线程池。如果线程数超出系统可以处理的范围，则可能导致系统瘫痪。
   
   ```java
   `ThreadPoolExecutor executor = (ThreadPoolExecutor) Executors.newCachedThreadPool();`
   ```

3. 计划线程池执行程序
   
    –创建一个线程池，该线程池可以计划命令在给定延迟后运行或定期执行。
   
   ```java
   `ThreadPoolExecutor executor = (ThreadPoolExecutor) Executors.newScheduledThreadPool(``10``);`
   ```

4. 单线程池执行程序
   
    –创建单线程以执行所有任务。当您只有一个任务要执行时，请使用它。
   
   ```java
   `ThreadPoolExecutor executor = (ThreadPoolExecutor) Executors.newSingleThreadExecutor();`
   ```

5. 工作窃取线程池执行程序
   
    –创建一个线程池，该线程池维护着足以支持给定并行度级别的线程。这里的并行度是指在多处理器机器中在单个时间点将用于执行给定任务的最大线程数。
   
   ```java
   `ThreadPoolExecutor executor = (ThreadPoolExecutor) Executors.newWorkStealingPool(``4``);`
   ```

#### 注意

阿里巴巴开发手册中禁止使用Excutors 创建线程池，因为线程数最大数是Integer.MAX_VALUE，可能会创建数量非常多的线程，甚至OOM。

所以使用 ThreadPoolExecutor 方法 手动构建线程池

#### 任务拒绝策略

当线程池满了之后，将会执行拒绝策略。线程池提供了4种拒绝策略。如下：

| 拒绝策略                | 说明                                                                |
| ------------------- | ----------------------------------------------------------------- |
| AbortPolicy         | 当线程池满时，将会拒绝接收新任务，并抛出RejectedExecutionException。如果没有指定拒绝策略，此为默认策略  |
| CallerRunsPolicy    | 当线程池满时，将会使用任务提交者所在的线程来执行这个任务。线程池本身会丢弃掉这个任务。                       |
| DiscardPolicy       | 当线程池满时，将会默默地丢弃掉这个任务。tips:在实际开发当中，如果任务不影响业务，则可以采用此策略，否则断然不可采取这个策略。 |
| DiscardOldestPolicy | 当线程池满时，将会默默地丢弃掉队列最前端的任务。然后执行提交的任务。tips:和以上一样                      |

#### 关闭线程池

关闭线程池是很有必要的。当应用程序需要退出时，可以通过注册回调函数来关闭线程池。如果我们暴力关闭应用程序的话，会导致正在执行的任务和队列中的任务丢失。在企业工程中，这一点千万注意。 线程池提供了两种关闭方法：

| 关闭方式          | 说明                                                                 |
| ------------- | ------------------------------------------------------------------ |
| shutdown()    | 当调用此方法时，线程池状态会变成SHUTDOWN状态，此时线程池将不会再接收新的任务，但以及接收的任务会执行完毕。          |
| shutdownNow() | 当调用此方法时，线程池状态会变成STOP状态，此时线程池将不会再接收新的任务，并且会中断所有正在执行中的任务以及丢弃掉队列中的任务。 |

在实际工程中，具体采取哪种方式，应该根据实际情况来抉择。如果任务对业务有影响，则应当选择shutdown()，否则可以视情况选择shutdownNow()。

#### 线程数设置策略

在Java应用中，线程属于稀有资源。那么线程数是设置的越大越好么？非也。在计算机体系中，如果想让性能发挥极致，应该是各个子系统之间的合理配置使用。对于线程数而言也是如此。要想合理的设置线程数，就必须首先分析人物的特性。可以从以下几个角度来分析：

- 任务的性质：CPU密集型、IO密集型、混合型。
- 任务的优先级：高、中、低。
- 任务的执行时间：长、中、短。
- 任务的依赖性：是否依赖其他资源，比如数据库连接。

我们可以根据任务的不同特性来综合考虑线程数的设置。一般而言。如果是CPU密集型，则应该分配尽可能小的线程数：通常情况下，可以设置为CPU核数 + 1；如果是IO密集型，则线程并不总是在执行任务，则应该分配尽可能大的线程数：通常情况下，可以设置为2 * CPU核数；如果是混合型，则可以将任务拆分成一个CPU密集型和一个IO密集型，只要两个任务执行的时间不会相差太大，则性能会比串行执行的效率要高，如果拆分后任务执行相差的时间过大，则没有必要拆分。

不符合上述场景时

![image-20210623232700515](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210623232700515.png)

#### 线程池的监控

通过以上的介绍，如何用好线程池应该不是问题。对于一个完善的应用而言，应当还要有良好的监控能力，以便在任务执行出现问题时，可以快速的定位、分析、解决问题。
 ThreadPoolExecutor提供了一些基本且好用的方法来监控线程池的运行情况：

| 方法名                     | 说明              |
| ----------------------- | --------------- |
| getTaskCount()          | 线程池队列中需要执行的任务数量 |
| getCompletedTaskCount() | 线程池中已经执行完成的任务数量 |
| getActiveCount()        | 线程池中正在执行任务的线程数量 |

如果我们想要更加全面的监控线程池的运行状态以及任务的执行过程。可以继承ThreadPoolExecutor来自定义线程池。

### public void execute(Runnable command)

execute 方法的作用是提交任务 command 到线程池进行执行。 用户线程提交任务到线 

程池的模型图如图 8-2 所示。

![image-20210626124915317](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210626124915317.png)

​          

从该图可以看出， ThreadPoo!Executor 的实现实际是一个生产消费模型， 当用户添加任务到线程池时相当于生产者生产元素， workers 线程工作集中的线程直接执行任务或者从任务队列里面获取任务时则相当于消费者消费元素。 

## 并发教程

### wait() 函数

线程通知和等待  调用该方法的线程会被阻塞挂起, 直到发生下面几件事情之一才返回: 

- 其他线程调用了该共享对象的notif() 或者notifyAll() 方法:
- 其它线程调用了该线程的interrupt()方法, 该线程抛出InterruptedException 异常返回 

需要注意的是,如果调用wait()方法的线程没有事先获取该对象的监视器锁,则调用wait() 方法时调用线程会抛出IllegalMonitorStateException 异常 

注意: 如果线程含有多个共享变量锁时, 只要调用其中一个共享变量的 wait()方法就会挂起当前线程

#### 一个线程怎样获取一个共享变量的监视器锁

1. 执行synchronized 同步代码块时, 使用该共享变量作为参数 

```
synchroized(共享便量){
// doSomething 
}    
```

2. 调用该共享变量的方法,并且该方法使用了synchronized 修饰 

```
synchronized void add(int a,int b){
    //doSomething 
}
```

另外需要注意的是，一个线程可以从挂起状态变为可以运行状态（ 就是被唤醒）， 即使该线程没有被其他线程调用 notify （）、 notifyAll()方法进行通知，或者被中断，或者等 待超时，这就是所谓的虚假唤醒

虽然虚假唤醒在应用实践中很少发生,但要防患于未然 ,做法就是不停 去测试该线程被唤醒的条件是否满足，不满足则继续等待，也就是说在一个循环中调用wait() 方法进行防范. 推出循环的条件是满足了唤醒该线程的条件

```
synchroized(obj){
    while(条件不满足){
        obj.wait();     
    }
}
```

如上代码是经典的调用共享变量wait() 方法的实例 

下面从一个简单的生产者和消费者例子来加深理解 

```
//生产线程
synchronized(queue){
    //消费队列满,则等待队列架空闲
    while(queue.size()==MAX_SIZE){
        tyr{
            //挂起当前线程, 并释放通过同步块获取的queue上的锁, 让消费者线程可以获取该锁, 然后获取队列里
            的元素 
            queue.wait();
        }catch(Exception ex{
            ex.printStackTrace();
        }
    }
    //空闲则生成元素, 并通知消费者线程
    queue.add(ele);
    queue.notifyAll();
}

//消费者线程
synchroized(queue)[
    //消费队列为空
    while(queue.sie()==0){
        try {
                 //挂起当前线程,并释放通过同步块获取的queue上的锁,让生产者线程可以获取该锁,将生产元素放入
                 队列 
                 queue.wait();
        }
    }
    //消费元素, 并通知唤醒生产者线程
    queue.take();
    queue.notifyAll();
]
```

上面代码中假如生产者线程A首先通过synchronized 获取到了queue上的锁, 那么后续所有企图生产元素的线程和消费线程将会在获取该监视器锁的地方被阻塞挂起. 线程A获取锁后发现当前队列已满会调用queue.wait()方法阻塞自己, 然后释放获取新的queue上的锁, 如果不释放, 由于其它生产者线程和所有消费者线程都已经被阻塞挂起, 而线程A也被挂起, 这就处于了死锁状态. 。所以这里线程A 挂起自己后释放共享便量上的锁, 就是为了打破死锁必要条件之一的持有并等待原则.  线程A释放锁后, 其它生产者线程和所有消费者线程中会有一个线程来获取queue上的锁而进入同步块, 这就打破了死锁状态 

另外需要注意的是, 当前线程调用共享变量的wait()方法后只会释放当前共享变量上的锁, 如果当前线程还持有其它共享变量的锁, 则这些锁是不会被释放的 

```java
//创建资源
    private static volatile Object resourceA = new Object() ;
    private static volatile Object resourceB = new Object() ;
    public static void main(String[] args) throws InterruptedException{
        //创建线程
        Thread threadA = new Thread(new Runnable(){
            @Override
            public void run() {
                try{
                    //获取resourceA 共享资源的监视器锁
                    synchronized(resourceA){
                        System.out.println("threadA get resourceA");
                        //获取resourceB共享资源的监视器
                        synchronized(resourceB){
                            System.out.println("threadA get resourceB to lock");
                            //线程A阻塞, 并释放获取到的resourceA的锁
                            System.out.println("threadA release resourceA lock");
                            //调用wait阻塞自己 
                            resourceA.wait();
                        }
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

        });
        //创建线程
        Thread threadB = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    //休眠1s, 让线程A先获取资源锁
                    Thread.sleep(1000);

                    //获取resourceA共享资源的监视器锁
                    synchronized (resourceA){
                        System.out.println("threadB get resourceA lock");
                        System.out.println("threadB try get resourceB lock...");
                        //获取resouceB共享资源监视器锁
                        synchronized (resourceB){
                            System.out.println("threadB get resouceB lock");
                            //线程B阻塞, 并示范获取到resourceA的锁
                            System.out.println("threadB release resourceA lock");
                            resourceA.wait();
                        }
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
        threadA.start();
        threadB.start();

        //等待两个线程结束
        threadA.join();
        threadB.join();
        System.out.println("main over");

    }
```

输出结果![image-20210609135945403](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210609135945403.png)

线程B获取resourceB lock被阻塞, 因为线程A挂起自己后并没有释放获取到resourceB上的锁 

证明了当线程调用共享对象的wait()方法时, 当前线程只会释放**当前共享对象的锁**, 当前线程持有的其它共享对象的监视器锁并不会

被释放 

再举一个例子说明 : 当一个线程调用共享对象的wait() 方法被阻塞挂起后 , 如果其它线程中断了该线程,则该线程会放出InterruptedException 异常并返回

```java
 @Test
    public void  Interrupte() throws InterruptedException {

        //创建线程
        Thread threadA = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    System.out.println("-------begin------");
                    //阻塞当前线程
                    synchronized (obj){
                        obj.wait();
                    }
                    System.out.println("------end-------");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });

        threadA.start();
        //避免直接中断threada
        Thread.sleep(1000);
        System.out.println("-----begin interrupt threadA----");
        threadA.interrupt();
        System.out.println("-----end interrupt threadA");
    }
```

输出结果如下 

![image-20210609175522704](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210609175522704.png)

#### wait(long timeout)函数

多了一个超时参数,  挂起后, 在timeout ms 时间内没有被其它线程调用notify()或者notifyAll()方法唤醒,该函数还是会因为超时而返回,wait(0) 和wait效果一样 , 注意: 传递负的timeout 则会抛出IllegalArgumentException 异常 

#### wait(long timeout, int nanos) 函数

 在其内部调用的是wait(long timeout) 函数, 如下代码只有在nanos > 0时才使参数timeout 递增1 

### join

等待线程执行终止

### Synchronized

![image-20210622130110001](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210622130110001.png)

内存语义

![image-20210622230357366](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210622230357366.png)

**对象级锁**是机制，当我们要同步**非静态方法**或者**非静态代码块**，使得只有一个线程就可以在类的给定实例执行的代码块。应该始终这样做**以确保实例级数据线程安全**。

```java
public class DemoClass
{
    public synchronized void demoMethod(){}
}

or

public class DemoClass
{
    public void demoMethod(){
        synchronized (this)
        {
            //other thread safe code
        }
    }
}

or

public class DemoClass
{
    private final Object lock = new Object();
    public void demoMethod(){
        synchronized (lock)
        {
            //other thread safe code
        }
    }
}
```

即使用synchronized关键字  或者加lock

**类级别锁定**可防止多个线程`synchronized`在运行时进入该类的所有可用实例中的任何一个块中。这意味着，如果在运行时有100个实例`DemoClass`，则一次只能在一个实例中的一个线程上执行一个线程`demoMethod()`，而所有其他实例将被其他线程锁定。

应该始终进行类级别锁定，**以使静态数据线程安全**。我们知道[**static**](https://howtodoinjava.com/java/keywords/java-static-keyword/)关键字将方法的数据关联到类级别，因此请使用对静态字段或方法的锁定使其在类级别上。

### volatile 关键字

volatile一般用在多个线程访问同一个变量时，对该变量进行唯一性约束，volatile保证了变量的可见性，不能保证原子性。

禁止JVM 的指令重排, 保证在多线程环境下 线程可以获得初始化的实例

![image-20210622213203295](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210622213203295.png)

使用volatile 关键字的例子

![image-20210622230721664](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210622230721664.png)

这里的volatile和synchronized 使用是等价的, 都解决了value的内存可见性问题, 但是前者是独占锁, 同时只能有一个线程调用get() 方法, 其他调用线程会被阻塞, 同时会存在线程上下文切换和线程重新调度的开销, 这也是使用锁方式不好的地方. 而后者是非阻塞算法, 不会造成线程上下文切换的开销

但并非所有情况使用它们都是等价的, volatile 虽然提供了可见性保证, 但不保证操作的原子性 

使用volatile 关键字的时间 

- 写入变量值不依赖变量的当前值时, 因为如果依赖当前值, 将是获取--计算--写入三步操作, 这三步操作不是原子性的, 而volatile不保证原子性 
- 读写变量值时没有加锁. 因为加锁本身已经保证了内存可见性, 这时候不需要把变量声明为volatile

### 原子性操作

 原子性操作, 是指一系列操作时, 这些操作要么全执行, 要么全部不执行, 不存在只执行其中一部分的情况. 在设计计数器时一般都先读取当前值, 然后+1, 再更新. 这个过程是读--改--写的过程, 如果不能保证这个过程是原子性的, 那么就会出现线程安全问题.  

> juc并发包中有原子类

### CAS算法

比较和交换实例

传统的锁定机制，例如在Java中使用synched关键字，被称为是锁定或多线程的乐观锁 

CAS操作有3个参数：

1. 必须替换值的存储位置V
2. 线程上次读取的旧值A
3. 新值B应该写在V上
- CAS+ 失败重试: CAS是乐观锁的一种实现方式 所谓乐观锁就是, 每次不加锁而是假设没有冲突去完成某项操作, 如果因为冲突失败就重试, 直到成功为止. **虚拟机采用CAS配上失败重试的方式保证更新操作的原子性**
- TLAB: 为每一个线程与现在Eden区分配一块儿内存, JVM 在给线程中对象分配内存时, 首先在TLAB分配 ,当对象大于TLAB中的剩余内存或TLAB的内存已用尽时, 再采用上述的CAS进行内存分配

JUC包提供了一系列原子性操作类, 这些类都是使用非阻塞算法CAS实现的, 相比使用锁实现原子性操作这在性能上有很大提高. 原子性操作类的原理都大致相同.

### AQS

抽象同步队列(AbstractQueueSynchronizer) 简称AQS , 它是实现同步器的基础组件, 并发包中锁的底层就是使用AQS 实现的, 另外, 大多数开发者可能永远不会直接使用AQS,但是知道其原理对于架构设计还是很有帮助的 ,下图为AQS类图结构

![image-20210624094430356](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210624094430356.png)

![image-20210624094736977](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210624094736977.png)

时间原因不继续深记 , 详解观看并发之美第六章, 二小节(6.2) 

### JUC

#### AtomicLong 类

public class AtomicLong extends Number implements java.io.Serializable {

```java
private static final long serialVersionUID = 1927816293512124184L;

// setup to use Unsafe.compareAndSwapLong for updates 
// 存放Unsafe实例
private static final Unsafe unsafe = Unsafe.getUnsafe();
// 存放变量value的偏移量
private static final long valueOffset;

/**
 * Records whether the underlying JVM supports lockless
 * compareAndSwap for longs. While the Unsafe.compareAndSwapLong
 * method works in either case, some constructions should be
 * handled at Java level to avoid locking user-visible locks.
 * 判断JVM是否支持Long类型无锁CAS
 */
static final boolean VM_SUPPORTS_LONG_CAS = VMSupportsCS8();

/**
 * Returns whether underlying JVM supports lockless CompareAndSet
 * for longs. Called only once and cached in VM_SUPPORTS_LONG_CAS.
 */
private static native boolean VMSupportsCS8();

static {
    try {
        valueOffset = unsafe.objectFieldOffset
            (AtomicLong.class.getDeclaredField("value"));
    } catch (Exception ex) { throw new Error(ex); }
}
//实际变量值
private volatile long value;

/**
 * Creates a new AtomicLong with the given initial value.
 *
 * @param initialValue the initial value
 */
public AtomicLong(long initialValue) {
    value = initialValue;
}
.... //源码并不完整 
}
```

![image-20210623095200919](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210623095200919.png)

![image-20210623095211091](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210623095211091.png)

AtomicLong类 是原子性递增或者递减类, 其内部使用Unsafe 实现

![image-20210623125110255](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210623125110255.png)

#### LongAddr

JDK8 新增原子类操作 , 在高并发场景下使用 

![image-20210623125045718](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210623125045718.png)

上图为使用LongAdder

![image-20210623133313711](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210623133313711.png)

重点研究longAccumulate 的代码逻辑 , 这是cells数组被初始化和扩容的地方 

```java
/**
 * Handles cases of updates involving initialization, resizing,
 * creating new Cells, and/or contention. See above for
 * explanation. This method suffers the usual non-modularity
 * problems of optimistic retry code, relying on rechecked sets of
 * reads.
 *
 * @param x the value
 * @param fn the update function, or null for add (this convention
 * avoids the need for an extra field or function in LongAdder).
 * @param wasUncontended false if CAS failed before call
 */
final void longAccumulate(long x, LongBinaryOperator fn,
                          boolean wasUncontended) {
    int h;
    if ((h = getProbe()) == 0) {
        ThreadLocalRandom.current(); // force initialization
        h = getProbe();
        wasUncontended = true;
    }
    boolean collide = false;                // True if last slot nonempty
    for (;;) {
        Cell[] as; Cell a; int n; long v;
        if ((as = cells) != null && (n = as.length) > 0) {
            if ((a = as[(n - 1) & h]) == null) {
                if (cellsBusy == 0) {       // Try to attach new Cell
                    Cell r = new Cell(x);   // Optimistically create
                    if (cellsBusy == 0 && casCellsBusy()) {
                        boolean created = false;
                        try {               // Recheck under lock
                            Cell[] rs; int m, j;
                            if ((rs = cells) != null &&
                                (m = rs.length) > 0 &&
                                rs[j = (m - 1) & h] == null) {
                                rs[j] = r;
                                created = true;
                            }
                        } finally {
                            cellsBusy = 0;
                        }
                        if (created)
                            break;
                        continue;           // Slot is now non-empty
                    }
                }
                collide = false;
            }
            else if (!wasUncontended)       // CAS already known to fail
                wasUncontended = true;      // Continue after rehash
            else if (a.cas(v = a.value, ((fn == null) ? v + x :
                                         fn.applyAsLong(v, x))))
                break;
            else if (n >= NCPU || cells != as)
                collide = false;            // At max size or stale
            else if (!collide)
                collide = true;
            else if (cellsBusy == 0 && casCellsBusy()) {
                try {
                    if (cells == as) {      // Expand table unless stale
                        Cell[] rs = new Cell[n << 1];
                        for (int i = 0; i < n; ++i)
                            rs[i] = as[i];
                        cells = rs;
                    }
                } finally {
                    cellsBusy = 0;
                }
                collide = false;
                continue;                   // Retry with expanded table
            }
            h = advanceProbe(h);
        }
        //初始化Cellll数组
        else if (cellsBusy == 0 && cells == as && casCellsBusy()) {
            boolean init = false;
            try {                           // Initialize table
                if (cells == as) {
                    Cell[] rs = new Cell[2];
                    rs[h & 1] = new Cell(x);
                    cells = rs;
                    init = true;
                }
            } finally {
                cellsBusy = 0;
            }
            if (init)
                break;
        }
        else if (casBase(v = base, ((fn == null) ? v + x :
                                    fn.applyAsLong(v, x))))
            break;                          // Fall back on using base
    }
}
```

额外的等有空再补

### 面试题

#### 什么是上下文切换

多线程编程中一般线程的个数都大于CPU 核心的个数, 而一个CPU核心在任意时刻只能被一个线程使用, 为了让这些线程都能够得到有效执行, CPU采取的策略是为每个线程分配事件片并轮转的形式. 当一个线程的事件片用完的时候就会重新处于就绪状态让给其他线程使用, 这个过程就属于一次上下文切换 

概括来说就是: 当前任务在执行完CPU 时间片切换到另一个任务之前先保存自己的状态, ,以便下次再切换回这个任务时, 可以再加载这个任务的状态 . **任务从保存到加载的过程就是一次上下文切换**

上下文切换通常是计算密集型的. 也就是说,它需要相当可观的处理器时间, 在每秒几十上百次的切换中, 每次切换都需要纳秒级的时间. 所以, 上下文切换对系统来说意味着消耗大量的CPU 时间, 事实上, 可能是操作系统中时间消耗最大的操作 

LInux 相比与其他操作(包括其他类Unix系统) 有很多的有点, 其中有一项就是, 其上下文切换和模式切换的时间消耗非常少 .

#### sleep() 方法和wait() 方法区别和共同点

- 两者最主要的区别在于：**`sleep()` 方法没有释放锁，而 `wait()` 方法释放了锁** 。
- 两者都可以暂停线程的执行。
- `wait()` 通常被用于线程间交互/通信，`sleep() `通常被用于暂停执行。
- `wait()` 方法被调用后，线程不会自动苏醒，需要别的线程调用同一个对象上的 `notify() `或者 `notifyAll()` 方法。`sleep() `方法执行完成后，线程会自动苏醒。或者可以使用 `wait(long timeout)` 超时后线程会自动苏醒。

#### 调用start()方法时会执行run方法, 为什么我们不能直接调用run方法

new 一个 Thread，线程进入了新建状态。调用 `start()`方法，会启动一个线程并使线程进入了就绪状态，当分配到时间片后就可以开始运行了。 `start()` 会执行线程的相应准备工作，然后自动执行 `run()` 方法的内容，这是真正的多线程工作。 但是，直接执行 `run()` 方法，会把 `run()` 方法当成一个 main 线程下的普通方法去执行，并不会在某个线程中执行它，所以这并不是多线程工作。

**总结： 调用 `start()` 方法方可启动线程并使线程进入就绪状态，直接执行 `run()` 方法的话不会以多线程的方式执行。**

### CountDownLatch

CountDownLatch**与JDK 1.5一起引入，并与**java.util.concurrent包中的**其他并发实用程序（如CyclicBarrier，Semaphore，****[ConcurrentHashMap](https://howtodoinjava.com/java/multi-threading/best-practices-for-using-concurrenthashmap/)****和****[BlockingQueue）](https://howtodoinjava.com/java/multi-threading/how-to-use-blockingqueue-and-threadpoolexecutor-in-java/)****一起引入**。此类**使Java线程可以等待，直到其他线程集完成**其任务。例如，应用程序的主线程要等待，直到负责启动框架服务的其他服务线程完成了所有服务的启动。0CountDownLatch的工作原理是使用线程数初始化计数器，每次线程完成执行时，计数器都会递减。当count达到零时，表示所有线程已完成其执行，并且等待闩锁的线程将恢复执行。

![倒数计时](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/CountdownLatch_example.png)

```java
//Constructs a CountDownLatch initialized with the given count.
public CountDownLatch(int count) {...}
```

此**计数本质上是**闩锁应等待**的线程数**。该值只能设置一次，并且CountDownLatch**没有**提供**其他机制来重置此count**。

与CountDownLatch的第一次交互是与主线程进行的，goind等待其他线程。启动其他线程后，该主线程必须立即调用**CountDownLatch.await（）**方法。该执行将在await（）方法上停止，直到其他线程完成其执行为止。

其他N个线程必须具有闩锁对象的引用，因为它们将需要通知CountDownLatch对象它们已完成其任务。该通知是通过以下方法完成的：**CountDownLatch.countDown（）** ; 每次方法调用都会使构造函数中设置的初始计数减少1。因此，当所有N个线程都调用此方法时，计数达到零，并且允许主线程通过await（）方法恢复执行。

### 线程同步(Unix 高级编程三)

synchronized 关键字

同步代码块【 synchronized(inner){  //需要进行访问控制的代码块    } 】     同步方法 【public synchronized void method（）】 ，关键部分是一次仅执行一个线程的线程，该线程持有同步部分的锁。

**sync**关键字有助于编写应用程序的[并发](https://howtodoinjava.com/java-concurrency-tutorial/)部分，以保护此块中的共享资源

同步避免了由于共享内存视图不一而导致的内存一致性错误

**当一个线程可以修改的变量, 其它线程也可以读取或者修改的时候, 我们就需要对这些线程进行同步, 确保它们在访问变量的存储内容时不会访问到无效的的值.** 

![image-20210605232425464](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210605232425464.png)

线程A读取变量然后给这个变量赋予一个新的数值, 但写操作需要两个存储周期, 当线程B在这两个存储器写周期中间读取这个变量时, 它就会的到并不一致的值![image-20210605232742078](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210605232742078.png)

为了解决获取值不同的问题, 线程不得不使用锁, 同一时间只允许一个线程访问该变量. 图11-8描述了这种同步, 如果线程B希望读取变量, 它首先要获取锁. 同样, 当线程A更新变量时, 也需要获取同样的这把锁 . 这样, 线程B在线程A释放锁以前就不能读取变量 

![image-20210605232900872](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210605232900872.png)

两个或多个线程识图在同一时间修改同一变量时, 也需要进行同步. 考虑变量增量操作的情况 ,增量操作通常分解为以下3 步 

(1) 从内存单元读入寄存器 

(2) 在寄存器中对变量做增量操作

(3) 把新的值写回内存单元 

![image-20210605233311129](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210605233311129.png)

如果两个线程试图几乎在同一时间对同一个变量做增量操作而不进行同步的话 , 结果就可能出现不一致, 变量的值可能增加1, 也可能增加2, 具体增加1还是2 要取决于第二个线程开始操作时获取的数值 ,如果第二个线程执行第1步 要比第一个线程执行第3步要早, 第二个线程读到的值与第一个线程一样,为变量加1, 然后写回去, 事实上没有实际的效果, 总的来说变量只增加了1 

如果修改操作是原子操作, 那么就不存在竞争. 在上面例子中, 如果增加1只需要一个存储器周期, 那么就没有竞争存在  

在顺序一致环境中, 可以把数据修改操作解释为运行线程的顺序执行步骤 . 可以把这样的操作描述为 "线程A对变量增加了1,然后线程B对变量增加了1, 所以变量的值就比原来的大 2" . 这两个线程的任何操作顺序都不可能让变量出现除了上述值以外的其他值 

除了计算机体系结构意外,程序使用变量的方式也会引起竞争, 也会导致不一致的情况发生, 比如, 我们可能对某个变量+1, 然后基于这个值做出就决定,因为这个增量操作步骤和这个决定步骤的组合并非原子操作, 所以就给不一致情况的出现提供了可能 

#### 互斥量

可以使用pthread 的互斥接口来保护数据 , 确保同一时间只有一个线程访问数据. 互斥量(mutex) 从本质上说是一把锁, 在访问共享资源前对互斥量进行设置(加锁) , 在访问完成后释放(解锁)互斥量. 对互斥量进行加锁 以后, 任何其他试图再次对互斥量加锁的线程都会被阻塞直到当前线程释放该互斥锁 .如果释放互斥量时有一个以上的线程阻塞, 那么所有该锁上的阻塞线程都会变成可运行状态, 第一个变为运行的线程就可以对互斥量加锁, 其它线程就会看到互斥量依然是锁着的, 只能回去再次等待它重新变为可用, 在这种方式下, 每次只有一个线程可以向前执行 .

线程要设计成遵守相同数据访问规则的 .![image-20210606094036442](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210606094036442.png)

要用默认的属性初始化互斥量, 只需把attr 设为NULL,

对互斥量进行加锁 ,需要调用pthread_mutex_lock ,如果互斥量已经上锁,调用线程将阻塞直到互斥量被解锁. 对互斥量解锁, 需要调用pthread_mutex_unlock 

![image-20210606140413101](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210606140413101.png)

线程不希望被阻塞, 可以使用pthread_mutex_trylock 尝试对互斥量进行加锁 

![image-20210606140643207](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210606140643207.png)

#### 避免死锁

如果线程试图对同一个互斥量加锁两次, 自身就会陷入死锁状态 

还有其他不太明显的方式也能产生死锁. 例如, 程序中使用一个以上的互斥量时, 如果允许一个线程一直占有第一个互斥量, 并且在试图锁住第二个互斥量时处于阻塞状态, 但是拥有第二个互斥量的线程也在试图锁住第一个互斥量 . 因为两个线程都在相互请求另一个线程拥有的资源, 所以这两个线程都无法向前运行, 于是产生死锁 

可以通过仔细控制互斥量加锁的顺序来避免思路的发生. 例如,假设需要对两个互斥量A和B同时加锁 ,如果所有线程总是在对互斥量B加锁之前锁住互斥量A, 那么使用这两个互斥量就不会产生死锁(当然在其他的资源上仍可能出现死锁) 类似地, 如果所有的线程总是在锁住互斥量A之前锁住了互斥量B,那么也不会发生死锁. 可能出现的死锁只会发生在一个线程试图锁住另一个线程以相反的顺序锁住的互斥量 .

有时候, 应用程序的结构使得对互斥量进行排序是很苦难的. 如果涉及了太多的锁和数据结构 ,可用的函数并不能吧它转换成简单的层次, 那么就需要采用另外的方法. 在这种情况下, 可以先释放占有的锁, 然后过一段时间再试.这种情况可以使用pthread_mutex_trylock接口避免死锁, 如果不能获取锁, 可以先释放已经占有的锁, 做好清理工作 ,然后过一段时间再重新试. 

实例 

![image-20210606160003696](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210606160003696.png)

![image-20210606160015520](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210606160015520.png)

![image-20210606160024654](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210606160024654.png)

![image-20210606160041364](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210606160041364.png)

#### 读写锁

与互斥量相似, 不过读写锁允许更高的并行性 . 互斥量要么是锁住状态, 要么就是不加锁状态, 而且一次只有一个线程可以对其加锁 . 读写锁有3种状态: 读模式下加锁状态, 写模式下加锁状态, 不加锁状态 . 一次只有一个线程可以占有写模式的读写锁, 但是多个线程可以同时占有读模式的读写锁 .

当读写锁是写加锁状态时, 在这个锁被解锁之前, 所有试图对这个锁加锁的线程都会被阻塞 .当读写锁在读加锁状态时, 所有试图以读模式对它进行加锁的线程都可以得到访问权, 但是任何希望以写模式对此锁进行加锁的线程都会被阻塞, 直到所有线程释放它们的读锁为止 . 虽然各操作系统对读写锁的实现各不相同, 但当读写锁处于读模式锁住的状态, 而这时有一个线程试图以写模式获取锁时, 读写锁通常会阻塞随后的读模式锁请求 . 这样可以避免读模式锁长期占用, 而等待的写模式锁请求一直得不到满足 .

读写锁非常适于对数据结构读的次数远大于写的情况 . 当读写锁在写模式下时, 它所保护的数据结构就可以被安全地修改, 因为一次只有一个线程可以在写模式下拥有这个锁 . 当读写锁在该模式下时,只要线程先获取了读模式下的读写锁 该锁保护的数据结构就可以被多个获得读模式锁的线程读取 . 

读写锁也叫共享互斥锁(shared-exclusive lock) . 当读写锁是读模式锁住时,就可以说成是以共享模式锁住的. 当它是写模式锁住的时候, 就可以说成是以互斥模式锁住的 . 

与互斥量相比, 读写锁在**使用之前**必须初始化  ,在释放它们底层的内存之前必须销毁 

![image-20210606221840566](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210606221840566.png)

读写锁通过调用pthread_rwlock_init 进行初始化 . 如果希望读写锁有默认的属性,可以传一个null指针给attr 

Single UNIX Specification 在XSI扩展中定义了PTHREAD_rWLOCK_INITIALIZER 常量, 如果默认属性就足够的话, 可以用它对静态分配的读写锁进行初始化 

![image-20210606222429944](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/image-20210606222429944.png)

各种实现可能会对共享模式下可获取的读写锁的次数进行限制 ,所以需要检查 pthread_rwlock_rdlock 的返回值 ,即使pthread_rwlock_wrlock 和 pthread_rwlock_unlock 有错误返回, 而且从技术上来讲, 在调用函数时应该总是检查错误返回, 但是如果锁设计合理的话 ,就不需要检查他们. 错误返回值的定义只是针对不正确使用读写锁的情况(如未经初始化的锁 ), 或者试图获取已拥有的锁从而可能产生死锁的情况. 但是需要注意, 有些特定的实现可能会定义另外的错误返回. 

Single UNIX Specification 还定义了读写锁原语的条件版本 

```
#include<pthread.h>
int pthread_rwlock_tryrdlock(pthread_rwlock_t, *rwlock);
int pthread_rwlock_trywdlock(pthread_rwlock_t, *rwlock);
                                        两个函数的返回值; 若成功, 返回0; 否则, 返回错误编号 
```

可以获取锁时, 这两个函数返回 0.,否则, 它们返回错误EBUSY , 这两个函数可以用于我们前面讨论的遵守某种锁层次但还不能完全避免死锁的情况 .

#### 带有超时的读写锁

#### 条件变量

#### 自旋锁

自旋锁（spin lock）与互斥锁（mutex）类似，任时刻只有一个线程能够获得锁。当一个线程在获取锁的时候，如果锁已经被其它线程获取，那么该线程将循环等待，然后不断的判断锁是否能够被成功获取，直到获取到锁才会退出循环。

在获取锁的过程中，线程一直处于活跃状态。因此与mutex不同，spinlock不会导致线程的状态切换(用户态->内核态)，一直处于户态，即线程一直都是active的；不会使线程进入阻塞状态，减少了不必要的上下文切换，执行速度快。

由于自旋时不释放CPU，如果持有自旋锁的线程一直不释放自旋锁，那么等待该自旋锁的线程会一直浪费CPU时间。因此，自旋锁主要适用于被持有时间短，线程不希望在重新调度上花过多时间的情况。

```java
//循环检查锁状态,并尝试获取,直到成功 
while(true):
    locked=get_lock(); 
    if locked==false; 
    locked=true; 

//上锁后执行相关任务 
do_something 
//执行关闭,释放锁 
locked=false 
```

将查询锁(get_lock)和设置锁(locked)设置true

```java
//get_and_set(locked) 是一个
while(get_and_set(locked)==false)
    continue 
do_something 

locked=false; 
```

如此, 就实现了一个简单的自旋锁 

那么现在的问题是如何实现`get_and_set(locked)`这个原子操作？这里有两个常用的方法可以使用：`TAS（test and set）`和`CAS （compare and swap）`。

1. `TAS`：一个TAS指令包括两个子步骤，把给定的内存地址设置为1，然后返回之前的旧值。

2. `CAS`：CAS指令需要三个参数，一个内存位置(V)、一个期望旧值(A)、一个新值(B)。过程如下：
   
   a. 比较内存V的值是否与A相等？  
   b. 如果相等，则用新值B替换内存位置V的旧值  
   c. 如果不相等，不做任何操作。  
   d. 无论哪个情况，CAS都会把内存V原来的值返回。

#### 屏障

### Semaphore

Semaphore 通常我们叫它信号量， 可以用来控制同时访问特定资源的线程数量，通过协调各个线程，以保证合理的使用资源。

可以把它简单的理解成我们停车场入口立着的那个显示屏，每有一辆车进入停车场显示屏就会显示剩余车位减1，每有一辆车从停车场出去，显示屏上显示的剩余车辆就会加1，当显示屏上的剩余车位为0时，停车场入口的栏杆就不会再打开，车辆就无法进入停车场了，直到有一辆车从停车场出去为止。

**使用场景**

通常用于那些资源有明确访问数量限制的场景， 常用于限流 

比如：数据库连接池，同时进行连接的线程有数量限制，连接不能超过一定的数量，当连接达到了限制数量后，后面的线程只能排队等前面的线程释放了数据库连接才能获得数据库连接。

比如：停车场场景，车位数量有限，同时只能容纳多少台车，车位满了之后只有等里面的车离开停车场外面的车才可以进入。

#### 常用方法

```java
acquire()  
获取一个令牌，在获取到令牌、或者被其他线程调用中断之前线程一直处于阻塞状态。

acquire(int permits)  
获取一个令牌，在获取到令牌、或者被其他线程调用中断、或超时之前线程一直处于阻塞状态。

acquireUninterruptibly() 
获取一个令牌，在获取到令牌之前线程一直处于阻塞状态（忽略中断）。

tryAcquire()
尝试获得令牌，返回获取令牌成功或失败，不阻塞线程。

tryAcquire(long timeout, TimeUnit unit)
尝试获得令牌，在超时时间内循环尝试获取，直到尝试获取成功或超时返回，不阻塞线程。

release()
释放一个令牌，唤醒一个获取令牌不成功的阻塞线程。

hasQueuedThreads()
等待队列里是否还存在等待线程。

getQueueLength()
获取等待队列里阻塞的线程数。

drainPermits()
清空令牌把可用令牌数置为0，返回清空令牌的数量。

availablePermits()
返回可用的令牌数量。
```

#### 实现原理

**(1)、Semaphore初始化。**

```java
Semaphore semaphore=new Semaphore(2);
```

1、当调用new Semaphore(2) 方法时，默认会创建一个非公平的锁的同步阻塞队列。

2、把初始令牌数量赋值给同步队列的state状态，state的值就代表当前所剩余的令牌数量。

**初始化完成后同步队列信息如下图：**

![img](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/v2-5f78e28f05d2a656fa534b5a816c45ac_720w.jpg)

 **（2）获取令牌**

```java
semaphore.acquire();
```

1、当前线程会尝试去同步队列获取一个令牌，获取令牌的过程也就是使用原子的操作去修改同步队列的state ,获取一个令牌则修改为state=state-1。

2、 当计算出来的state<0，则代表令牌数量不足，此时会创建一个Node节点加入阻塞队列，挂起当前线程。

3、当计算出来的state>=0，则代表获取令牌成功。

源码： 

```java
/**
     *  获取1个令牌
     */
    public void acquire() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }
```

```java
/**
     * 共享模式下获取令牌，获取成功则返回，失败则加入阻塞队列，挂起线程
     * @param arg
     * @throws InterruptedException
     */
    public final void acquireSharedInterruptibly(int arg)
            throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        //尝试获取令牌，arg为获取令牌个数，当可用令牌数减当前令牌数结果小于0,则创建一个节点加入阻塞队列，挂起当前线程。
        if (tryAcquireShared(arg) < 0)
            doAcquireSharedInterruptibly(arg);
    }
```

```java
/**
     * 1、创建节点，加入阻塞队列，
     * 2、重双向链表的head，tail节点关系，清空无效节点
     * 3、挂起当前节点线程
     * @param arg
     * @throws InterruptedException
     */
    private void doAcquireSharedInterruptibly(int arg)
        throws InterruptedException {
        //创建节点加入阻塞队列
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            for (;;) {
                //获得当前节点pre节点
                final Node p = node.predecessor();
                if (p == head) {
                    int r = tryAcquireShared(arg);//返回锁的state
                    if (r >= 0) {
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        failed = false;
                        return;
                    }
                }
                //重组双向链表，清空无效节点，挂起当前线程
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```

**线程1、线程2、线程3、分别调用semaphore.acquire(),整个过程队列信息变化如下图：**

![img](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/v2-698639f5f06e35b648ad7f1eb41b04b0_720w.jpg)

 **(3)、释放令牌**

```java
 semaphore.release();
```

当调用semaphore.release() 方法时

1、线程会尝试释放一个令牌，释放令牌的过程也就是把同步队列的state修改为state=state+1的过程

2、释放令牌成功之后，同时会唤醒同步队列中的一个线程。

3、被唤醒的节点会重新尝试去修改state=state-1 的操作，如果state>=0则获取令牌成功，否则重新进入阻塞队列，挂起线程。

**源码：**

```java
 /**
     * 释放令牌
     */
    public void release() {
        sync.releaseShared(1);
    }
```

```java
/**
     *释放共享锁，同时会唤醒同步队列中的一个线程。
     * @param arg
     * @return
     */
    public final boolean releaseShared(int arg) {
        //释放共享锁
        if (tryReleaseShared(arg)) {
            //唤醒所有共享节点线程
            doReleaseShared();
            return true;
        }
        return false;
    }
```

```java
 /**
     * 唤醒同步队列中的一个线程
     */
    private void doReleaseShared() {
        for (;;) {
            Node h = head;
            if (h != null && h != tail) {
                int ws = h.waitStatus;
                if (ws == Node.SIGNAL) {//是否需要唤醒后继节点
                    if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))//修改状态为初始0
                        continue;
                    unparkSuccessor(h);//唤醒h.nex节点线程
                }
                else if (ws == 0 &&
                         !compareAndSetWaitStatus(h, 0, Node.PROPAGATE));
            }
            if (h == head)                   // loop if head changed
                break;
        }
    }
```

**继上面的图，当我们线程1调用semaphore.release(); 时候整个流程如下图：**

![img](../images/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/v2-03aa28cc53dfe6820d46d8517c1e1e59_720w.jpg)
