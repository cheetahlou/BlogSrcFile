---
title: AQS原理机制浅探
date: 2019-11-18 16:14:00
tags: 
- AQS
---

## AQS原理机制（with源码）



## 源码探AQS原理机制

AQS(AbstractQueuedSynchronizer)是concurrent包下用来构建锁或其他同步组件（信号量、事件等）的基础框架类。 JDK中许多并发工具类的内部实现都依赖于AQS，如ReentrantLock, Semaphore, CountDownLatch等等。 

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



**参考文章：**

[AbstractQueuedSynchronizer源码解读](https://www.cnblogs.com/micrari/p/6937995.html)

[AbstractQueuedSynchronizer 原理分析 - 独占/共享模式]( [http://www.tianxiaobo.com/2018/05/01/AbstractQueuedSynchronizer-%E5%8E%9F%E7%90%86%E5%88%86%E6%9E%90-%E7%8B%AC%E5%8D%A0-%E5%85%B1%E4%BA%AB%E6%A8%A1%E5%BC%8F/](http://www.tianxiaobo.com/2018/05/01/AbstractQueuedSynchronizer-原理分析-独占-共享模式/) )