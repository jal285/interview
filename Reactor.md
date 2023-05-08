# Reactor 模型

在高性能的I/O设计中，有两个著名的模型：Reactor模型和Proactor模型，其中Reactor模型用于同步I/O，而Proactor模型运用于异步I/O操作。

想要了解两种模型，需要了解一些IO、同步异步的基础知识，[点击查看](https://mp.weixin.qq.com/s?__biz=MzUyNzgyNzAwNg==&mid=2247483941&idx=1&sn=97628f4d69d8607badf39bfeb7557457&scene=21#wechat_redirect)

## 响应式编程

### 什么是响应式编程？它和传统的编程方式有什么区别？

响应式编程基于同步I/O

响应式可以简单的理解为收到某个事件或通知后采取的一系列动作，如上文中所说的响应操作系统的网络数据通知，然后以**回调的方式**处理数据。

传统的命令式编程主要由：顺序、分支、循环 等控制流来完成不同的行为

响应式编程的特点是：

- **以逻辑为中心转换为以数据为中心**
- **从命令式到声明式的转换**

### 响应流

响应流的特点

- **响应流**必须是无阻塞的。
- **响应流**必须是一个数据**流**。
- 它必须可以异步执行。
- 并且它也应该能够处理背压。

### Java Reactor 框架使用和详解

已经涵盖并实现了非阻塞NIO流 

#### Mono

[参考](https://developer.51cto.com/art/202008/624995.htm)

https://segmentfault.com/a/1190000024499748

```
Mono.just()
```

`Mono` 是一个发出(emit)`0-1`个元素的`Publisher<T>`,可以被`onComplete`信号或者`onError`信号所终止。

![image-20210622202401236](../images/Reactor/image-20210622202401236.png)

这里就不翻译了，整体和`Flux`差不多，只不过这里只会发出0-1个元素。也就是说不是有就是没有。象`Flux`一样，我们来看看`Mono`的演化过程以帮助理解。

 传统数据处理

```java
public ClientUser currentUser () {
    return isAuthenticated ? new ClientUser("felord.cn", "reactive") : null;
}
```

直接返回符合条件的对象或者`null`。

Optional的处理方式

```java
public Optional<ClientUser> currentUser () {
    return isAuthenticated ? Optional.of(new ClientUser("felord.cn", "reactive"))
            : Optional.empty();
}
```

这个`Optional`我觉得就有反应式的那种味儿了，当然它并不是反应式。当我们不从返回值`Optional`取其中具体的对象时，我们不清楚里面到底有没有，但是`Optional`是一定客观存在的,不会出现**NPE**问题。

 反应式数据处理

```java
public Mono<ClientUser> currentUser () {
    return isAuthenticated ? Mono.just(new ClientUser("felord.cn", "reactive"))
            : Mono.empty();
}
```

和`Optional`有点类似的机制，当然`Mono`不是为了解决**NPE**问题的，它是为了处理响应流中单个值（也可能是`Void`）而存在的。

#### fkux

## Java CompletableFuture

Java CompletableFuture 是 Java 8 强大的函数式异步编程辅助类

CompletableFuture 的一个优点是它是真正的异步，它允许您从调用者线程和 API 异步运行您的任务，例如`thenXXX`允许您在结果可用时对其进行处理。另一方面，`CompletableFuture`并不总是非阻塞的。例如，当您运行以下代码时，它会在默认情况下异步执行`ForkJoinPool`：

```java
CompletableFuture.supplyAsync(() -> {
    try {
        Thread.sleep(1000);
    }
    catch (InterruptedException e) {

    }

    return 1;
});
```

很明显，执行任务的`Thread`in`ForkJoinPool`最终会被阻塞，这意味着我们不能保证调用是非阻塞的。

另一方面，`CompletableFuture`公开 API 允许您使其真正非阻塞。

例如，您始终可以执行以下操作：

```java
public CompletableFuture myNonBlockingHttpCall(Object someData) {
    var uncompletedFuture = new CompletableFuture(); // creates uncompleted future

    myAsyncHttpClient.execute(someData, (result, exception -> {
        if(exception != null) {
            uncompletedFuture.completeExceptionally(exception);
            return;
        }
        uncompletedFuture.complete(result);
    })

    return uncompletedFuture;
}
```

如您所见，`CompletableFuture`未来的 API为您提供了在需要时完成执行的方法`complete`和`completeExceptionally`方法，而不会阻塞任何线程。

### 基本使用

在大部分情况下我们可能会用一个线程池去执行这些异步任务。`CompletableFuture.complete()`、`CompletableFuture.completeExceptionally`只能被调用一次。但是我们有两个后门方法可以重设这个值:`obtrudeValue`、`obtrudeException`，但是使用的时候要小心，因为`complete`已经触发了客户端，有可能导致客户端会得到不期望的结果。

#### 创建CompletableFuture对象

`CompletableFuture.completedFuture`是一个静态辅助方法，用来返回一个已经计算好的`CompletableFuture`。

```
public static <U> CompletableFuture<U> completedFuture(U value)
```

```java
/***
     * 四个异步方法
     * @param runnable

     * @return
     */
    public static CompletableFuture<Void> runAsync(Runnable runnable) {
        return null;
    }

    public static CompletableFuture<Void> runAsync(Runnable runnable, Executor executor) {
        return null;
    }

    public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier) {
        return null;
    }

    public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor) {
        return null;
    }
```

以`Async`结尾并且没有指定`Executor`的方法会使用`ForkJoinPool.commonPool()`作为它的线程池执行异步代码。

`runAsync`方法也好理解，它以`Runnable`函数式接口类型为参数，所以`CompletableFuture`的计算结果为空。

`supplyAsync`方法以`Supplier<U>`函数式接口类型为参数,`CompletableFuture`的计算结果类型为`U`。

方法的参数类型都是函数式接口, 所以可以使用lambda表达式实现异步任务

```java
public static void main(String[] args) {
    CompletableFuture<String> future = CompletableFuture.supplyAsync(()->{
        //计算任务;
        return ".00";
    });
}
```

#### 计算结果完成时的处理

当`CompletableFuture`的计算结果完成，或者抛出异常的时候，我们可以执行特定的`Action`。主要是下面的方法：

```java
public CompletableFuture<T>     whenComplete(BiConsumer<? super T,? super Throwable> action)
public CompletableFuture<T>     whenCompleteAsync(BiConsumer<? super T,? super Throwable> action)
public CompletableFuture<T>     whenCompleteAsync(BiConsumer<? super T,? super Throwable> action, Executor executor)
public CompletableFuture<T>     exceptionally(Function<Throwable,? extends T> fn)
```

可以看到`Action`的类型是`BiConsumer<? super T,? super Throwable>`，它可以处理正常的计算结果，或者异常情况。
方法不以`Async`结尾，意味着`Action`使用相同的线程执行，而`Async`可能会使用其它的线程去执行(如果使用相同的线程池，也可能会被同一个线程选中执行)。

注意这几个方法都会返回`CompletableFuture`，当`Action`执行完毕后它的结果返回原始的`CompletableFuture`的计算结果或者返回异常。

`exceptionally`方法返回一个新的CompletableFuture，当原始的CompletableFuture抛出异常的时候，就会触发这个CompletableFuture的计算，调用function计算值，否则如果原始的CompletableFuture正常计算完后，这个新的CompletableFuture也计算完成，它的值和原始的CompletableFuture的计算的值相同。也就是这个`exceptionally`方法用来处理异常的情况。

下面一组方法虽然也返回CompletableFuture对象，但是对象的值和原来的CompletableFuture计算的值不同。当原先的CompletableFuture的值计算完成或者抛出异常的时候，会触发这个CompletableFuture对象的计算，结果由`BiFunction`参数计算而得。因此这组方法兼有`whenComplete`和转换的两个功能。

```
public <U> CompletableFuture<U>     handle(BiFunction<? super T,Throwable,? extends U> fn)
public <U> CompletableFuture<U>     handleAsync(BiFunction<? super T,Throwable,? extends U> fn)
public <U> CompletableFuture<U>     handleAsync(BiFunction<? super T,Throwable,? extends U> fn, Executor executor)
```

同样，不以`Async`结尾的方法由原来的线程计算，以`Async`结尾的方法由默认的线程池`ForkJoinPool.commonPool()`或者指定的线程池`executor`运行。

#### 转换

[参考](https://colobu.com/2016/02/29/Java-CompletableFuture/)
