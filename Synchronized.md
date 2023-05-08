# Synchroized 关键字 及锁机制优化

<!--more-->

Java中synchroized 加锁为单体锁 

## 锁与对象头

![image-20210622130125471](../images/Synchronized/image-20210622130125471.png)



java 中 Synchoronized 锁住的是的对象 

贯穿全文的为mark word 

![image-20210608144215440](../images/Synchronized/image-20210608144215440.png)

(cms_free 1.6后移出)

ark Word在64位虚拟机下，也就是占用64位大小即8个字节的空间.

内具体容包括：

- unused：未使用的
- hashcode：上文提到的**identity** hash code，本文出现的hashcode都是指identity hash code
- thread: 偏向锁记录的线程标识
- epoch: 验证偏向锁有效性的时间戳
- age：分代年龄
- biased_lock 偏向锁标志
- lock 锁标志
- pointer_to_lock_record 轻量锁lock record指针
- pointer_to_heavyweight_monitor 重量锁monitor指针



所有锁的状态如下

![image-20210608145725658](../images/Synchronized/image-20210608145725658.png)





### 对象内存布局查看工具-JOL

```text
<!-- https://mvnrepository.com/artifact/org.openjdk.jol/jol-core -->
<dependency>
  <groupId>org.openjdk.jol</groupId>
  <artifactId>jol-core</artifactId>
  <version>0.9</version>
</dependency>
```

测试程序如下

```

	@Test  
    public void test01() throws Exception{
        Object o1 = new Object();
        System.out.println(ClassLayout.parseInstance(o1).toPrintable());  // 不加锁 
    }
```

打印结果如下

![image-20210608114909417](../images/Synchronized/image-20210608114909417.png)

***打印结果是一道高级的面试题哦：Object obj = new Object()初始化出的obj对象，在内存中占用多少字节？\***大家还可尝试声明一个类，分别加上boolean、Boolean、int、Integer、数组、引用对象等成员变量，打印出的结果便可观看出该类型在Java中到底占多少字节。

修改为打印String 类型的对象

```
  @Test
    public void test01() throws Exception{
        Object o1 = new Object();
        String o2 = new String("fafafa");
        System.out.println(ClassLayout.parseInstance(o2).toPrintable());
    }
```

![image-20210608115200562](../images/Synchronized/image-20210608115200562.png)

一个含有属性的user类 

![image-20210608120439828](../images/Synchronized/image-20210608120439828.png)

一个object 类为16字节 ,  四个String 变量为16字节 

对象size: 32 字节 



8字节的mark



#### 对加synchronized 的方法进行对象内存查看

![image-20210608124234715](../images/Synchronized/image-20210608124234715.png)





![img](../images/Synchronized/v2-eca382c5b933670007c1f206f70e4c31_720w.jpg)



其中2 bit 的锁标志位 表示锁的状态, 1bit 的偏向锁标志位表示是否偏向 (下文位为重点锁膨胀过程 )

1. 当对象初始化后，还未有任何线程来竞争，此时为无锁状态。其中`锁标志位`为**01**，`偏向锁标志位`为**0**
2. 当有一个线程来竞争锁，锁对象第一次被线程获取时，`锁标志位`依然为**01**，`偏向锁标志位`会被置为1，此时锁进入偏向模式。同时，使用CAS操作将此获取锁对象的线程ID设置到锁对象的Mark Word中，持有偏向锁，下次再可直接进入。
3. 此时，线程B尝试获取锁，发现锁处于偏向模式，但Mark Word中存储的不是本线程ID。那么线程B使用CAS操作尝试获取锁，这时锁是有可能获取成功的，因为上一个持有偏向锁的线程不会主动释放偏向锁。如果线程B获取锁成功，则会将Mark Word中的线程ID设置为本线程的ID。但若线程B获取锁失败，则会执行下述操作。
4. 偏向锁抢占失败，表明锁对象存在竞争，则会先撤销偏向模式，`偏向锁标志位`重新被置为**0**，准备升级轻量级锁。首先将在当前线程的帧栈中开辟一块锁记录空间（Lock Record），用于存储锁对象当前的Mark Word拷贝。然后，使用CAS操作尝试把锁对象的Mark Word更新为指向帧栈中Lock Record的指针，CAS操作成功，则代表获取到锁，同时将`锁标志位`设置为**00**，进入轻量级锁模式。若CAS操作失败，则进入下述操作。
5. 刚一出现CAS竞争轻量级锁失败时，不会立即膨胀为重量级锁，而是采用**自旋**的方式，不断重试，尝试抢锁。JDK1.6中，默认开启自旋，自旋10次，可通过-XX:PreBlockSpin更改自旋次数。JDK1.6对于只能指定固定次数的自旋进行了优化，采用了**自适应的自旋**，重试机制更加智能。
6. 只有通过自旋依然获取不到锁的情况，表明锁竞争较为激烈，不再适合额外的CAS操作消耗CPU资源，则直接膨胀为重量级锁，`锁标志位`设置为**10**。在此状态下，所有等待锁的线程都必须进入阻塞状态。



![image-20210608124304713](../images/Synchronized/image-20210608124304713.png)

```java 
  @Test
    public  void test02() throws Exception{
        Object lock = new Object();
        System.out.println("枷锁时");
        synchronized (lock){
            //
            String layout = ClassLayout.parseInstance(lock).toPrintable();
            System.out.println(layout);
        }

        System.out.println("*********************释放锁后*");
        String layout2 = ClassLayout.parseInstance(lock).toPrintable();
        System.out.println(layout2);
    }
```



![image-20210608125223843](../images/Synchronized/image-20210608125223843.png) 

`synchronized`修饰代码块时，会产生`mointerenter`和`mointerexit`指令。那么，jvm是如何通过这两个指令来搞定加锁的呢？下面我们一步步跟踪openjdk源码中，如何实现的`mointerenter`和`mointerexit`。

```java
// 解释器的同步代码被分解出来，以便方法调用和同步快可以共享使用
// The interpreter's synchronization code is factored out so that it can
// be shared by method invocation and synchronized blocks.
//%note synchronization_3

//%note monitor_1 monitorenter同步锁加锁方法
IRT_ENTRY_NO_ASYNC(void, InterpreterRuntime::monitorenter(JavaThread* thread, BasicObjectLock* elem))
#ifdef ASSERT
  thread->last_frame().interpreter_frame_verify_monitor(elem);
#endif
  if (PrintBiasedLockingStatistics) { // 打印偏向锁的统计
    Atomic::inc(BiasedLocking::slow_path_entry_count_addr());
  }
  Handle h_obj(thread, elem->obj());
  assert(Universe::heap()->is_in_reserved_or_null(h_obj()),
         "must be NULL or an object");
  if (UseBiasedLocking) { // 如果开启了偏向模式
    // Retry fast entry if bias is revoked to avoid unnecessary inflation
    // 请快速重试进入，如果偏向锁被取消以避免不必要的膨胀
    ObjectSynchronizer::fast_enter(h_obj, elem->lock(), true, CHECK);
  } else {
    // 没开启偏向模式的，则调用slow_enter方法进入轻/重量级锁
    ObjectSynchronizer::slow_enter(h_obj, elem->lock(), CHECK);
  }
  assert(Universe::heap()->is_in_reserved_or_null(elem->obj()),
         "must be NULL or an object");
#ifdef ASSERT
  thread->last_frame().interpreter_frame_verify_monitor(elem);
#endif
IRT_END


//%note monitor_1  monitorexit同步锁的释放锁方法
IRT_ENTRY_NO_ASYNC(void, InterpreterRuntime::monitorexit(JavaThread* thread, BasicObjectLock* elem))
#ifdef ASSERT
  thread->last_frame().interpreter_frame_verify_monitor(elem);
#endif
  Handle h_obj(thread, elem->obj());
  assert(Universe::heap()->is_in_reserved_or_null(h_obj()),
         "must be NULL or an object");
  if (elem == NULL || h_obj()->is_unlocked()) {
    THROW(vmSymbols::java_lang_IllegalMonitorStateException());
  }
  ObjectSynchronizer::slow_exit(h_obj(), elem->lock(), thread);
  // Free entry. This must be done here, since a pending exception might be installed on
  // exit. If it is not cleared, the exception handling code will try to unlock the monitor again.
  elem->set_obj(NULL);
#ifdef ASSERT
  thread->last_frame().interpreter_frame_verify_monitor(elem);
#endif
IRT_END
```

### jdk源码中fast_enter 和 slow_enter方法

仔细阅读上文中对于锁膨胀过程的介绍 

```
// -----------------------------------------------------------------------------
// Monitor快速Enter/Exit的方法，解释器和编译器使用了一些汇编语言在其中。如果一下的函数被更改，请确保更新他们。实现方式对竟态条件及其敏感，务必小心。
//  Fast Monitor Enter/Exit
// This the fast monitor enter. The interpreter and compiler use
// some assembly copies of this code. Make sure update those code
// if the following function is changed. The implementation is
// extremely sensitive to race condition. Be careful.

void ObjectSynchronizer::fast_enter(Handle obj, BasicLock* lock, bool attempt_rebias, TRAPS) {
 if (UseBiasedLocking) {// 又判断了一遍是否使用偏向模式
    if (!SafepointSynchronize::is_at_safepoint()) {// 确保当前不在安全点
      // 偏向锁加锁：revoke_and_rebias
      BiasedLocking::Condition cond = BiasedLocking::revoke_and_rebias(obj, attempt_rebias, THREAD);
      if (cond == BiasedLocking::BIAS_REVOKED_AND_REBIASED) {
        return;
      }
    } else {
      assert(!attempt_rebias, "can not rebias toward VM thread");
      BiasedLocking::revoke_at_safepoint(obj);
    }
    assert(!obj->mark()->has_bias_pattern(), "biases should be revoked by now");
 }
 // 快速加锁未成功时，采用慢加锁的方式
 slow_enter (obj, lock, THREAD) ;
}

void ObjectSynchronizer::fast_exit(oop object, BasicLock* lock, TRAPS) {
  // 从下面这个断言遍可得知：偏向锁不会进入快锁解锁方法。
  assert(!object->mark()->has_bias_pattern(), "should not see bias pattern here");
  // displaced header是升级轻量级锁过程中，用于存储锁对象MarkWord的拷贝，官方为这份拷贝加了一个Displaced前缀。可参考：《深入理解Java虚拟机》第三版482页的介绍。
  // 如果displaced header是空，先前的加锁便是重量级锁
  // if displaced header is null, the previous enter is recursive enter, no-op
  markOop dhw = lock->displaced_header();
  markOop mark ;
  if (dhw == NULL) {
     // Recursive stack-lock. 递归堆栈锁
     // Diagnostics -- Could be: stack-locked, inflating, inflated. 断定应该是：堆栈锁、膨胀中、已膨胀（重量级锁）
     mark = object->mark() ;
     assert (!mark->is_neutral(), "invariant") ;
     if (mark->has_locker() && mark != markOopDesc::INFLATING()) {
        assert(THREAD->is_lock_owned((address)mark->locker()), "invariant") ;
     }
     if (mark->has_monitor()) {
        ObjectMonitor * m = mark->monitor() ;
        assert(((oop)(m->object()))->mark() == mark, "invariant") ;
        assert(m->is_entered(THREAD), "invariant") ;
     }
     return ;
  }

  mark = object->mark() ; // 锁对象头的MarkWord

  // 此处为轻量级锁的释放过程，使用CAS方式解锁（下述方法中的cmpxchg_ptr即CAS操作）。
  // 如果对象被当前线程堆栈锁定，请尝试将displaced header和锁对象中的MarkWord替换回来。
  // If the object is stack-locked by the current thread, try to
  // swing the displaced header from the box back to the mark.
  if (mark == (markOop) lock) {
     assert (dhw->is_neutral(), "invariant") ;
     if ((markOop) Atomic::cmpxchg_ptr (dhw, object->mark_addr(), mark) == mark) {
        TEVENT (fast_exit: release stacklock) ;
        return;
     }
  }

  ObjectSynchronizer::inflate(THREAD, object)->exit (true, THREAD) ;
}

// -----------------------------------------------------------------------------
// Interpreter/Compiler Slow Case
// 解释器/编译器慢加锁的case。常规操作，此时不需使用fast_enter的方式，因为一定是在解释器/编译器已经失败过了。
// This routine is used to handle interpreter/compiler slow case
// We don't need to use fast path here, because it must have been
// failed in the interpreter/compiler code.
void ObjectSynchronizer::slow_enter(Handle obj, BasicLock* lock, TRAPS) {
  markOop mark = obj->mark();
  assert(!mark->has_bias_pattern(), "should not see bias pattern here");

  if (mark->is_neutral()) {
    // 预期成功的CAS -- 替换标记的ST必须是可见的 <= CAS执行的ST。优先使用轻量级锁（又叫：自旋锁）
    // Anticipate successful CAS -- the ST of the displaced mark must
    // be visible <= the ST performed by the CAS.
    lock->set_displaced_header(mark);
    if (mark == (markOop) Atomic::cmpxchg_ptr(lock, obj()->mark_addr(), mark)) {
      TEVENT (slow_enter: release stacklock) ;
      return ;
    }
    // Fall through to inflate() ... 上面没成功，只能向下执行inflate()锁膨胀方法了
  } else
  if (mark->has_locker() && THREAD->is_lock_owned((address)mark->locker())) { //当前线程已持有锁
    assert(lock != mark->locker(), "must not re-lock the same lock");
    assert(lock != (BasicLock*)obj->mark(), "don't relock with same BasicLock");
    lock->set_displaced_header(NULL);
    return;
  }

#if 0
  // The following optimization isn't particularly useful.
  if (mark->has_monitor() && mark->monitor()->is_entered(THREAD)) {
    lock->set_displaced_header (NULL) ;
    return ;
  }
#endif

  // 对象头将再也不会被移到这个锁锁，所以是什么值并不重要，除非必须是非零的，以避免看起来像是重入锁，而且也不能看起来是锁定的。
  // 重量级锁的mrakword中除了锁标记位为10外，另外30位是：指向重量级锁的指针
  // The object header will never be displaced to this lock,
  // so it does not matter what the value is, except that it
  // must be non-zero to avoid looking like a re-entrant lock,
  // and must not look locked either.
  lock->set_displaced_header(markOopDesc::unused_mark());
  ObjectSynchronizer::inflate(THREAD, obj())->enter(THREAD);
}

// This routine is used to handle interpreter/compiler slow case
// We don't need to use fast path here, because it must have
// failed in the interpreter/compiler code. Simply use the heavy
// weight monitor should be ok, unless someone find otherwise.
void ObjectSynchronizer::slow_exit(oop object, BasicLock* lock, TRAPS) {
  fast_exit (object, lock, THREAD) ;
}
```

### jdk源码中inflate 方法 

同样是synchronized.cpp文件中的方法, 

```c
// Note that we could encounter some performance loss through false-sharing as
// multiple locks occupy the same $ line.  Padding might be appropriate.
// 注意：当多个锁并发使用同一 $=行时，错误的共享方式可能会导致一些性能损失。填充可能是合适的。


ObjectMonitor * ATTR ObjectSynchronizer::inflate (Thread * Self, oop object) {
  // Inflate mutates the heap ...
  // Relaxing assertion for bug 6320749.
  assert (Universe::verify_in_progress() ||
          !SafepointSynchronize::is_at_safepoint(), "invariant") ;

  for (;;) {
      const markOop mark = object->mark() ;
      assert (!mark->has_bias_pattern(), "invariant") ;

      // The mark can be in one of the following states:
      // *  Inflated     - just return 仅仅返回
      // *  Stack-locked - coerce it to inflated 轻量级锁，需强迫它膨胀
      // *  INFLATING    - busy wait for conversion to complete 膨胀中，需自旋等待转换完成
      // *  Neutral中立的 - aggressively inflate the object. 积极地使object发生膨胀
      // *  BIASED       - Illegal.  We should never see this 进入此方法必定不是偏向锁状态，直接忽略即可

      // CASE: inflated
      if (mark->has_monitor()) {
          ObjectMonitor * inf = mark->monitor() ;
          assert (inf->header()->is_neutral(), "invariant");
          assert (inf->object() == object, "invariant") ;
          assert (ObjectSynchronizer::verify_objmon_isinpool(inf), "monitor is invalid");
          return inf ;
      }

      // CASE: inflation in progress - inflating over a stack-lock.   锁膨胀正在进行中，膨胀的堆栈锁（轻量级锁）
      // Some other thread is converting from stack-locked to inflated.     其他线程正在从堆栈锁（轻量级锁）定转换为膨胀。
      // Only that thread can complete inflation -- other threads must wait.  只有那个线程才能完成膨胀——其他线程必须等待。
      // The INFLATING value is transient.                    INFLATING状态是暂时的
      // Currently, we spin/yield/park and poll the markword, waiting for inflation to finish. 并发地，我们 spin/yield/park和poll的markword，等待inflation结束。
      // We could always eliminate polling by parking the thread on some auxiliary list.  我们总是可以通过将线程停在某个辅助列表上来消除轮询。
      if (mark == markOopDesc::INFLATING()) {
         TEVENT (Inflate: spin while INFLATING) ;
         ReadStableMark(object) ;
         continue ;
      }

      // CASE: stack-locked 此时锁为：轻量级锁，需强迫它膨胀为重量级锁
      // Could be stack-locked either by this thread or by some other thread.  可能被此线程或其他线程堆栈锁定
      //
      // Note that we allocate the objectmonitor speculatively, _before_ attempting
      // to install INFLATING into the mark word.  We originally installed INFLATING,
      // allocated the objectmonitor, and then finally STed the address of the
      // objectmonitor into the mark.  This was correct, but artificially lengthened
      // the interval in which INFLATED appeared in the mark, thus increasing
      // the odds of inflation contention.
      // 我们大胆地分配objectmonitor，在此之前尝试将INFLATING状态先设置到mark word。
      // 我们先设置了INFLATING状态标记，然后分配了objectmonitor，最后将objectmonitor的地址设置到mark word中。
      // 这是正确的，但人为地延长了INFLATED出现在mark上的时间间隔，从而增加了锁膨胀的可能性。
      // 老外反复说了一堆重复的话，意思无非就是：markword设置状态INFLATING（结合上段对INFLATING处理的代码思考） -> 分配锁 -> markword设置状态INFLATED(膨胀重量级锁成功)
      //
      // We now use per-thread private objectmonitor free lists.
      // These list are reprovisioned from the global free list outside the
      // critical INFLATING...ST interval.  A thread can transfer
      // multiple objectmonitors en-mass from the global free list to its local free list.
      // This reduces coherency traffic and lock contention on the global free list.
      // Using such local free lists, it doesn't matter if the omAlloc() call appears
      // before or after the CAS(INFLATING) operation.
      // See the comments in omAlloc().

      if (mark->has_locker()) {
          ObjectMonitor * m = omAlloc (Self) ;
          // Optimistically prepare the objectmonitor - anticipate successful CAS
          // We do this before the CAS in order to minimize the length of time
          // in which INFLATING appears in the mark.
          m->Recycle();
          m->_Responsible  = NULL ;
          m->OwnerIsThread = 0 ;
          m->_recursions   = 0 ;
          m->_SpinDuration = ObjectMonitor::Knob_SpinLimit ;   // Consider: maintain by type/class

          markOop cmp = (markOop) Atomic::cmpxchg_ptr (markOopDesc::INFLATING(), object->mark_addr(), mark) ;
          if (cmp != mark) {
             omRelease (Self, m, true) ;
             continue ;       // Interference -- just retry
          }

          // We've successfully installed INFLATING (0) into the mark-word.
          // This is the only case where 0 will appear in a mark-work.
          // Only the singular thread that successfully swings the mark-word
          // to 0 can perform (or more precisely, complete) inflation.
          //
          // Why do we CAS a 0 into the mark-word instead of just CASing the
          // mark-word from the stack-locked value directly to the new inflated state?
          // Consider what happens when a thread unlocks a stack-locked object.
          // It attempts to use CAS to swing the displaced header value from the
          // on-stack basiclock back into the object header.  Recall also that the
          // header value (hashcode, etc) can reside in (a) the object header, or
          // (b) a displaced header associated with the stack-lock, or (c) a displaced
          // header in an objectMonitor.  The inflate() routine must copy the header
          // value from the basiclock on the owner's stack to the objectMonitor, all
          // the while preserving the hashCode stability invariants.  If the owner
          // decides to release the lock while the value is 0, the unlock will fail
          // and control will eventually pass from slow_exit() to inflate.  The owner
          // will then spin, waiting for the 0 value to disappear.   Put another way,
          // the 0 causes the owner to stall if the owner happens to try to
          // drop the lock (restoring the header from the basiclock to the object)
          // while inflation is in-progress.  This protocol avoids races that might
          // would otherwise permit hashCode values to change or "flicker" for an object.
          // Critically, while object->mark is 0 mark->displaced_mark_helper() is stable.
          // 0 serves as a "BUSY" inflate-in-progress indicator.


          // fetch the displaced mark from the owner's stack.
          // The owner can't die or unwind past the lock while our INFLATING
          // object is in the mark.  Furthermore the owner can't complete
          // an unlock on the object, either.
          markOop dmw = mark->displaced_mark_helper() ;
          assert (dmw->is_neutral(), "invariant") ;

          // Setup monitor fields to proper values -- prepare the monitor
          m->set_header(dmw) ;

          // Optimization: if the mark->locker stack address is associated
          // with this thread we could simply set m->_owner = Self and
          // m->OwnerIsThread = 1. Note that a thread can inflate an object
          // that it has stack-locked -- as might happen in wait() -- directly
          // with CAS.  That is, we can avoid the xchg-NULL .... ST idiom.
          m->set_owner(mark->locker());
          m->set_object(object);
          // TODO-FIXME: assert BasicLock->dhw != 0.

          // Must preserve store ordering. The monitor state must
          // be stable at the time of publishing the monitor address.
          guarantee (object->mark() == markOopDesc::INFLATING(), "invariant") ;
          object->release_set_mark(markOopDesc::encode(m));

          // Hopefully the performance counters are allocated on distinct cache lines
          // to avoid false sharing on MP systems ...
          if (ObjectMonitor::_sync_Inflations != NULL) ObjectMonitor::_sync_Inflations->inc() ;
          TEVENT(Inflate: overwrite stacklock) ;
          if (TraceMonitorInflation) {
            if (object->is_instance()) {
              ResourceMark rm;
              tty->print_cr("Inflating object " INTPTR_FORMAT " , mark " INTPTR_FORMAT " , type %s",
                (void *) object, (intptr_t) object->mark(),
                object->klass()->external_name());
            }
          }
          return m ;
      }

      // CASE: neutral
      // TODO-FIXME: for entry we currently inflate and then try to CAS _owner.
      // If we know we're inflating for entry it's better to inflate by swinging a
      // pre-locked objectMonitor pointer into the object header.   A successful
      // CAS inflates the object *and* confers ownership to the inflating thread.
      // In the current implementation we use a 2-step mechanism where we CAS()
      // to inflate and then CAS() again to try to swing _owner from NULL to Self.
      // An inflateTry() method that we could call from fast_enter() and slow_enter()
      // would be useful.

      assert (mark->is_neutral(), "invariant");
      ObjectMonitor * m = omAlloc (Self) ;
      // prepare m for installation - set monitor to initial state
      m->Recycle();
      m->set_header(mark);
      m->set_owner(NULL);
      m->set_object(object);
      m->OwnerIsThread = 1 ;
      m->_recursions   = 0 ;
      m->_Responsible  = NULL ;
      m->_SpinDuration = ObjectMonitor::Knob_SpinLimit ;       // consider: keep metastats by type/class

      if (Atomic::cmpxchg_ptr (markOopDesc::encode(m), object->mark_addr(), mark) != mark) {
          m->set_object (NULL) ;
          m->set_owner  (NULL) ;
          m->OwnerIsThread = 0 ;
          m->Recycle() ;
          omRelease (Self, m, true) ;
          m = NULL ;
          continue ;
          // interference - the markword changed - just retry.
          // The state-transitions are one-way, so there's no chance of
          // live-lock -- "Inflated" is an absorbing state.
      }

      // Hopefully the performance counters are allocated on distinct
      // cache lines to avoid false sharing on MP systems ...
      if (ObjectMonitor::_sync_Inflations != NULL) ObjectMonitor::_sync_Inflations->inc() ;
      TEVENT(Inflate: overwrite neutral) ;
      if (TraceMonitorInflation) {
        if (object->is_instance()) {
          ResourceMark rm;
          tty->print_cr("Inflating object " INTPTR_FORMAT " , mark " INTPTR_FORMAT " , type %s",
            (void *) object, (intptr_t) object->mark(),
            object->klass()->external_name());
        }
      }
      return m ;
  }
}
```

### 锁升级过程 

锁升级过程, 可以总结为: 无锁 -> 偏向锁 -> 轻量轻锁(自旋锁, 自适应自旋)



### 锁的状态

锁的状态总共有四种: 无锁状态、偏向锁、轻量级锁和重量级锁。随着锁的竞争，锁可以从偏向锁升级到轻量级锁，再升级的重量级锁（但是锁的升级是单向的，也就是说只能从低到高升级，不会出现锁的降级）。JDK 1.6中默认是开启偏向锁和轻量级锁的，我们也可以通过-XX:-UseBiasedLocking来禁用偏向锁。

#### 轻量级的加锁过程

（1）在代码进入同步块的时候， 如果同步对象锁状态为无所状态（所标志位为“01”状态 ，是否为偏向锁为"0") , 虚拟机首先将在当前线程的栈帧中建立一个名为锁记录（Lock Record）的空间，用于存储锁对象目前的Mark Word 的拷贝 ，官方称之为 Displaced Mark Word。 这时候线程堆栈与对象头的状态如图2.1所示 

（2）拷贝对象头中的Mark Word 复制到锁记录中 

  (3) 拷贝成功后，虚拟机将使用CAS操作尝试将对象的Mark Word 更新为指向Lock Record 的指针， 并将Lock record 里的owner 指针指向object mark word 。 如果更新成功， 则执行步骤（4），否则执行步骤（5)  

  (4) 如果这个更新操作成功了， 那么这个线程就拥有了该对象的锁， 并且对象Mark Word 的锁标志位设置位“00”，即表示此对象处于轻量级锁定状态， 这时候线程堆栈与对象头的状态如图2.2 所示 

（5）如果这个更新操作失败了 ，虚拟机首先会检查对象的Mark Word是否指向当前线程的栈帧，如果是就说明当前线程已经拥有了这个对象的锁，那就可以直接进入同步块继续执行。否则说明多个线程竞争锁，轻量级锁就要膨胀为重量级锁，所标志的状态值变为"10", Mark Word 中存储的就是指向重量级锁（互斥量）的指针，后面等待锁的线程也要进入阻塞状态。而当前线程便尝试使用自旋来获取锁，自旋就是为了不让线程阻塞，而采用循环去获取锁的过程

![image-20210608155231757](../images/Synchronized/image-20210608155231757.png)

![image-20210608161055240](../images/Synchronized/image-20210608161055240.png)



### 偏向锁

引入偏向锁是为了在无线程竞争的情况下尽量减少不必要的轻量级锁执行路径， 因为轻量级锁的获取及释放依赖多次CAS原子指令， 而偏向锁只要在置换ThreadID的时候依赖一次CAS原子指令（由于一旦出现多线程竞争的情况就必须撤销偏向锁，所以偏向锁的撤销操作的性能损耗必须小于节省下来的CAS原子指令的性能消耗）。上面说过，轻量级锁是为了在线程交替执行同步块时提高性能，而偏向锁则是在只有一个线程执行同步块时进一步提高性能。

1、偏向锁获取过程：

　　（1）访问Mark Word中偏向锁的标识是否设置成1，锁标志位是否为01——确认为可偏向状态。

　　（2）如果为可偏向状态，则测试线程ID是否指向当前线程，如果是，进入步骤（5），否则进入步骤（3）。

　　（3）如果线程ID并未指向当前线程，则通过CAS操作竞争锁。如果竞争成功，则将Mark Word中线程ID设置为当前线程ID，然后执行（5）；如果竞争失败，执行（4）。

　　（4）如果CAS获取偏向锁失败，则表示有竞争。当到达全局安全点（safepoint）时获得偏向锁的线程被挂起，偏向锁升级为轻量级锁，然后被阻塞在安全点的线程继续往下执行同步代码。

　　（5）执行同步代码。

2、偏向锁的释放：

　　偏向锁的撤销在上述第四步骤中有提到。偏向锁只有遇到其他线程尝试竞争偏向锁时，持有偏向锁的线程才会释放锁，线程不会主动去释放偏向锁。偏向锁的撤销，需要等待全局安全点（在这个时间点上没有字节码正在执行），它会首先暂停拥有偏向锁的线程，判断锁对象是否处于被锁定状态，撤销偏向锁后恢复到未锁定（标志位为“01”）或轻量级锁（标志位为“00”）的状态。

3、重量级锁、轻量级锁和偏向锁之间转换

**![Java 锁与对象头5](../images/Synchronized/af728337fcdbb13cb541787eac5a27371603856918316.png)**

### 其它优化

1. 适应性自旋
2. 锁粗化(Lock Coarsening): 就是将多次连接在一起的加锁、解锁操作合并为一次，将多个连锁的锁扩展成一个范围更大的锁

例



```
public class StringBufferTest{
	StringBuffer stringBuffer =  new StringBuffer();
	
	public void append(){
		stringBuffer.append("a");
		stringBuffer.append("b");
		stringBuffer.append("c");
	}
}
```

每次调用stringBuffer.append 方法都需要加锁和解锁，如果虚拟机检测到有一系列连串的对同一个对象加锁和解锁操作，就会将其合并成一次范围更大的加锁和解锁操作，即在第一次append方法时进行加锁，最后一次append方法结束后进行解锁。
