---
title: AQS原理机制浅探
date: 2019-11-18 16:14:00
tags: 
- AQS
---

## AQS原理机制（with源码）



## 源码探AQS原理机制

AQS(AbstractQueuedSynchronizer)是concurrent包下用来构建锁或其他同步组件（信号量、事件等）的基础框架类。 JDK中许多并发工具类的内部实现都依赖于AQS，如ReentrantLock, Semaphore, CountDownLatch等等。 

**自定义同步器在实现时只需要实现共享资源state的获取与释放方式即可**，至于具体线程等待队列的维护（如获取资源失败入队/唤醒出队等），AQS已经在顶层实现好了。自定义同步器实现时主要实现以下几种方法：

- isHeldExclusively()：该线程是否正在独占资源。只有用到condition才需要去实现它。
- tryAcquire(int)：独占方式。尝试获取资源，成功则返回true，失败则返回false。
- tryRelease(int)：独占方式。尝试释放资源，成功则返回true，失败则返回false。
- tryAcquireShared(int)：共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
- tryReleaseShared(int)：共享方式。尝试释放资源，如果释放后允许唤醒后续等待结点返回true，否则返回false。

todo:按调用顺序分方法拷贝出await()、signal()等源码添加阅读中文注释，包括方法头上 1. 2. 3.步总结做了什么操作，然后在方法内关键代码行上也添加阅读注释，参考如下

![image-20191208014057909](https://i.imgur.com/v4SH0J6.png)

Condition原理：

await():

signal():

确认持有独占锁后调用enq()将等待队列中的Node节点移动到同步器同步队列中，并且通过LockSupport.unpark(node.thread)唤醒节点中的线程参与同步队列中获取同步状态(即锁)的竞争中，同时await()中因不再满足判断不在同步队列中条件，退出while循环，acquireQueued()成功获取到同步状态（锁）后继续往下执行，从await()方法返回,原线程不再阻塞继续往下执行

```java

        public final void await() throws InterruptedException {
            if (Thread.interrupted())
                throw new InterruptedException();
            Node node = addConditionWaiter();
            int savedState = fullyRelease(node);
            int interruptMode = 0;
            //signal()移动到同步队列后不满足条件退出while循环
            while (!isOnSyncQueue(node)) {
                LockSupport.park(this);
                if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
                    break;
            }
            //acquireQueued()成功获取到同步状态（锁）后继续往下执行，从await()方法返回,原线程不再阻塞继续往下执行
            if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
                interruptMode = REINTERRUPT;
            if (node.nextWaiter != null) // clean up if cancelled
                unlinkCancelledWaiters();
            if (interruptMode != 0)
                reportInterruptAfterWait(interruptMode);
        }
```

CLH自旋锁能确保无饥饿性，提供先来先服务的公平性。互斥锁资源被占用时会引起调用者睡眠，而自旋锁不会 会一直循环获取。

列举JDK中几种常见使用了AQS的同步组件：

- ReentrantLock: 使用了AQS的**独占**获取和释放,用state变量记录某个线程获取独占锁的次数,获取锁时+1，释放锁时-1，在获取时会校验线程是否可以获取锁。
- Semaphore: 使用了AQS的**共享**获取和释放，用state变量作为计数器，只有在大于0时允许线程进入。获取锁时-1，释放锁时+1。用作控制并发的数量，如数据库连接池等需要限制数量的场景，如大小为10的线程池限制DB连接数最大为3`private static Semaphore semaphore = new Semaphore(3);`。
- CountDownLatch: 使用了AQS的**共享**获取和释放，用state变量作为计数器，在初始化时指定。只要state还大于0，获取共享锁会因为失败而阻塞，直到计数器的值为0时，共享锁才允许获取，所有等待线程会被逐一唤醒。

ReentrantLock独占模式获取锁过程：

tryAcquire()尝试进行加锁

先getState判断状态，如果状态为0即为无锁状态。

​    如果为公平锁，则先判断是否需要排队，判断依据为head节点的下一个非取消（waitStatus<=0）节点不为null且节点中的thread线程不为null。如果不需要排队返回false，`if (!hasQueuedPredecessors() && compareAndSetState(0, acquires))`，取反true则直接尝试CAS变更state状态值加锁，如果加锁成功则设置独占线程为当前线程。并且tryAcquire()返回true表示加锁成功，不需要再进行下面的入队排队等

需要排队时不用进行&&后面的CAS操作，不执行ifelse中代码而是继续往下判断state状态

如果状态不为0，即为加锁状态，则判断当前拥有独占锁的线程是否为当前线程

​	如果是，则将state值再加1（因为此时持有锁，所以直接 `setState(c)` 不用CAS）。这也是为什么ReentrantLock可重入，相应的重入锁释放锁过程每次将state减1，减为0时完全释放锁。

​	如果不是，则直接返回false表示加锁失败，进行入队排队。



```java
    /**
     * Creates and enqueues node for current thread and given mode.
     *
     * @param mode Node.EXCLUSIVE for exclusive, Node.SHARED for shared
     * @return the new node
     */
    private Node addWaiter(Node mode) {
        Node node = new Node(mode);

        for (;;) {
            Node oldTail = tail;
            if (oldTail != null) {
                //有尾节点，即队列已经初始化
                //设置node的前节点为tail
                node.setPrevRelaxed(oldTail);
                if (compareAndSetTail(oldTail, node)) {
                    //CAS替换tail为当前node，成功之后将旧tail节点的next指针指向当前节点，注意这两步之间可能有其他线程成为新的tail节点，此时next指针还未加上，所以比如释放锁时要从tail往前prev遍历才不会漏掉节点，enq()方法中同样如此
                    oldTail.next = node;
                    return node;
                }
            } else {
                //如果此时tail为空则初始化队列，将头尾节点都设为空Node，之后继续循环进入if判断
                initializeSyncQueue();
            }
        }
    }

/**
 * Initializes head and tail fields on first contention.
 * 当第一次争夺锁时初始化同步队列，也就是说，当第一个线程进来时队列未初始化，也不会初始化，因为此时没有竞争，
 * 看代码料想第一个之后的后续直接获取到锁的线程也是不会初始化同步队列。
 * 初始化队列时头尾节点都设为空Node
 */
private final void initializeSyncQueue() {
    Node h;
    if (HEAD.compareAndSet(this, null, (h = new Node())))
        tail = h;
}
```

自旋获取锁，竞争失败后线程park()进入WAITING状态：

```java

    /*
     * Various flavors of acquire, varying in exclusive/shared and
     * control modes.  Each is mostly the same, but annoyingly
     * different.  Only a little bit of factoring is possible due to
     * interactions of exception mechanics (including ensuring that we
     * cancel if tryAcquire throws exception) and other control, at
     * least not without hurting performance too much.
     */

    /**
     * Acquires in exclusive uninterruptible mode for thread already in
     * queue. Used by condition wait methods as well as acquire.
     *
     * @param node the node
     * @param arg the acquire argument
     * @return {@code true} if interrupted while waiting
     */
    final boolean acquireQueued(final Node node, int arg) {
        //中断标记
        boolean interrupted = false;
        try {
            for (;;) {
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) {
                    //此处为提高性能，如果前节点为head则尝试获取一次锁，有可能锁已经被释放还没通知
                    setHead(node);
                    p.next = null; // help GC
                    return interrupted;
                }
                if (shouldParkAfterFailedAcquire(p, node))
                    //确认前面有排队的SIGNAL节点后当前线程park，等待唤醒
                    interrupted |= parkAndCheckInterrupt();
            }
        } catch (Throwable t) {
            cancelAcquire(node);
            if (interrupted)
                //响应中断
                selfInterrupt();
            throw t;
        }
    }

    /**
     * Checks and updates status for a node that failed to acquire.
     * Returns true if thread should block. This is the main signal
     * control in all acquire loops.  Requires that pred == node.prev.
     *
     * @param pred node's predecessor holding status
     * @param node the node
     * @return {@code true} if thread should block
     * 返回线程是否需要被阻塞等待，清除中间已取消的节点，确认前节点状态为signal时返回true表示当前Node的线程可以被park等待后续唤醒
     * 方法中可能改变pred的waitStatus但不会改变当前node的等待状态
     */
    private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
        //注意！是pred前节点的等待状态 用以判断前节点状态如果是signal的话，当前线程可以安心park等待唤醒。
        // 初始化等待队列后第一次进入该方法时pred的waitStatus一定是初始化值0，由下一个入队的节点来更改为SIGNAL，
        // 其他节点同理都由后一个节点来更新前节点的等待状态
        int ws = pred.waitStatus;
        if (ws == Node.SIGNAL)
            //前节点为SIGNAL状态，则返回当前Node可以被park线程
            /*
             * This node has already set status asking a release
             * to signal it, so it can safely park.
             */
            return true;
        if (ws > 0) {
            /*
             * Predecessor was cancelled. Skip over predecessors and
             * indicate retry.
             */
            //前节点为已取消状态，则循环清除前面所有的已取消节点，后再retry
            do {
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            pred.next = node;
        } else {
            /*
             * waitStatus must be 0 or PROPAGATE.  Indicate that we
             * need a signal, but don't park yet.  Caller will need to
             * retry to make sure it cannot acquire before parking.
             * 进入else时ws一定为初始化状态0或者PROPAGATE，更改前节点等待状态在下一次循环时确认拿不到锁后对当前线程执行park
             * 有可能的情况比如有线程占用锁正在运行，head节点为队列初始化节点，本次将head的ws改为SIGNAL后，下次acquireQueued()方法
             * 循环有可能刚好锁被释放当前节点直接CAS获取到锁不再第二次进入本方法，即本来要park的现在CAS一次后有可能直接获取锁
             */
            pred.compareAndSetWaitStatus(ws, Node.SIGNAL);
        }
        return false;
    }
```



### **问答:**

Q：为什么需要SIGNAL状态

在共享唤醒中，多线程并发争夺唤醒权，必然存在一个cas的过程。也就是需要一个从有状态值cas到0的过程。所以要存在这样的一个状态值，最后就是SIGNAL了。从另外一个角度来看，节点一旦进入取消状态就不可恢复，因此需要存在一个不同的状态用来表示该节点需要唤醒，这也就是signal。

Q：为什么需要PROPAGATE状态

在共享唤醒中，所有的节点都不断的抢夺唤醒权是没有意义而且浪费的。同时需要一个与初始状态不同的状态用来表达多线程竞争唤醒权的结果。因为从SIGNAL到0是表示唤醒权被某一个线程抢夺完成，因此需要有一个额外的状态可以用来通知其他竞争线程可以停止竞争了。所以就有了 PROPAGATE状态。

参考：https://zhuanlan.zhihu.com/p/112371628



### AQS源码阅读总结：

自旋处做if else判断，首次进入更改满足条件（做初始化什么的），第二次进入循环时满足条件进入if判断。或某一次更改判断条件后之后的循环中进入另一段逻辑。如addWaiter()。

思考为什么需要这一步判断，这一步判断的条件会在哪些情况下出现。假想+验证，比如要取消获取锁的时候，先自己假想一下队列需要做什么动作，Node需要做什么动作。

LockSupport.park()方法是使线程进入无限等待（WAITING状态），当调用unpark()或notify()时解除等待进入运行态（RUNNABLE）等待CPU时间片调度

![线程的状态转换](https://imgedu.lagou.com/add0a9690e0c4196b483f1ff65f77622.jpg)



源码阅读总结：

养成读源码时同时写总览Walkthrough的好习惯，写出1,2,3点简要说明过程，如

```
/*
 * Walkthrough:
 * 1. If write lock held by another thread, fail.
 * 2. Otherwise, this thread is eligible for
 *    lock wrt state, so ask if it should block
 *    because of queue policy. If not, try
 *    to grant by CASing state and updating count.
 *    Note that step does not check for reentrant
 *    acquires, which is postponed to full version
 *    to avoid having to check hold count in
 *    the more typical non-reentrant case.
 * 3. If step 2 fails either because thread
 *    apparently not eligible or CAS fails or count
 *    saturated, chain to version with full retry loop.
 */
```



**参考文章：**

[AbstractQueuedSynchronizer源码解读](https://www.cnblogs.com/micrari/p/6937995.html)

[AbstractQueuedSynchronizer 原理分析 - 独占/共享模式]( [http://www.tianxiaobo.com/2018/05/01/AbstractQueuedSynchronizer-%E5%8E%9F%E7%90%86%E5%88%86%E6%9E%90-%E7%8B%AC%E5%8D%A0-%E5%85%B1%E4%BA%AB%E6%A8%A1%E5%BC%8F/](http://www.tianxiaobo.com/2018/05/01/AbstractQueuedSynchronizer-原理分析-独占-共享模式/) )

[Java并发之AQS详解]([http://ifeve.com/java%e5%b9%b6%e5%8f%91%e4%b9%8baqs%e8%af%a6%e8%a7%a3/#more-44669](http://ifeve.com/java并发之aqs详解/#more-44669))

