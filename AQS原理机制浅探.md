---
title: AQS原理机制浅探
date: 2019-11-18 16:14:00
tags: 
- AQS
---

## AQS原理机制（with源码）



## 源码探AQS原理机制

AQS(AbstractQueuedSynchronizer)是concurrent包下用来构建锁或其他同步组件（信号量、事件等）的基础框架类。 JDK中许多并发工具类的内部实现都依赖于AQS，如ReentrantLock, Semaphore, CountDownLatch等等。 









**参考文章：**

[AbstractQueuedSynchronizer源码解读](https://www.cnblogs.com/micrari/p/6937995.html)

[AbstractQueuedSynchronizer 原理分析 - 独占/共享模式]( [http://www.tianxiaobo.com/2018/05/01/AbstractQueuedSynchronizer-%E5%8E%9F%E7%90%86%E5%88%86%E6%9E%90-%E7%8B%AC%E5%8D%A0-%E5%85%B1%E4%BA%AB%E6%A8%A1%E5%BC%8F/](http://www.tianxiaobo.com/2018/05/01/AbstractQueuedSynchronizer-原理分析-独占-共享模式/) )