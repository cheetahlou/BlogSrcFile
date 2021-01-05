---
title: MESI缓存一致性协议笔记
date: 2020-03-11 16:14:00
tags: 
- MESI volatile
typora-root-url: assets
---

## MESI缓存一致性协议笔记

缓存一致性问题主要指

![img](https://pic1.zhimg.com/80/v2-326ce3092f7c80cb4b20177724cdc8e4_720w.jpg)



### CPU切换状态阻塞解决-存储缓存（Store Bufferes）

比如你需要修改本地缓存中的一条信息，那么你必须将I（无效）状态通知到其他拥有该缓存数据的CPU缓存中，并且等待确认。等待确认的过程会阻塞处理器，这会降低处理器的性能。应为这个等待远远比一个指令的执行时间长的多。

#### Store Bufferes

为了避免这种CPU运算能力的浪费，Store Bufferes被引入使用。处理器把它想要写入到主存的值写到缓存，然后继续去处理其他事情。当所有失效确认（Invalidate Acknowledge）都接收到时，数据才会最终被提交。
这么做有两个风险

#### Store Bufferes的风险   重排序

第一、就是处理器会尝试从存储缓存（Store buffer）中读取值，但它还没有进行提交。这个的解决方案称为Store Forwarding，它使得加载的时候，如果存储缓存中存在，则进行返回。
第二、保存什么时候会完成，这个并没有任何保证。

```java
value = 3；

void exeToCPUA(){
  value = 10;
  isFinsh = true;
}
void exeToCPUB(){
  if(isFinsh){
    //value一定等于10？！
    assert value == 10;
  }
}
```

试想一下开始执行时，CPU A保存着finished在E(独享)状态，而value并没有保存在它的缓存中。（例如，Invalid）。在这种情况下，value会比finished更迟地抛弃存储缓存。完全有可能CPU B读取finished的值为true，而value的值不等于10。

**即isFinsh的赋值在value赋值之前。**

这种在可识别的行为中发生的变化称为***重排序***（reordings）。注意，这不意味着你的指令的位置被恶意（或者好意）地更改。

它只是意味着其他的CPU会读到跟程序中写入的顺序不一样的结果。



### 内存屏障

针对store buffer和invalidate queue这两个优化带来的问题，我们又提供了内存屏障作为解决方案。内存屏障交给了编写程序的人的手里，利用它就可以规避上面提到的问题。

内存屏障分为写屏障和读屏障，编写程序时可以在期望的地方加入内存屏障。写屏障会强制cpu清空store buffer的内容，也就是将所有的变更都写入cache，随后变更也就写入了内存，使其对其他cpu可见；读屏障会强制cpu执行invalidate queue中的所有invalidate操作，使自身的cache内容失效，从而使cpu从内存或者其他cpu中获取最新的cache数据。



### 自己总结整理的问答：

1、为什么要有Invalidate Queue失效队列？

A：缓解大量的invalidate消息。不马上收到消息就进行失效操作

2、为什么要有Store Buffer存储缓存？

A：CPU计算速度以及寄存器存储速度比高速缓存要快，原本写入缓存操作还要等待先发送invalidate消息并等待其他CPU返回Ack 确认消息，浪费了性能，现在可以写了本地Store Buffer就直接返回继续处理。

3、为什么并发情况下会有重排序，当CPU0先将不在cache line中的前面a写到 store buffer再发出read invalidate消息，后一条语句变量b是E独占状态又同时收到其他CPU1先读取后变量时，会直接写入Cache Line中并返回给CPU1，CPU1先读到后面新的b，之后才收到read invalidate失效a的消息，本应先到的失效消息因为store buffer的异步写入晚到了。CPU0对a的写入走的是Store Buffer(有延迟)，而对b的写入走的是Cache，因此b比a先在Cache中生效，导致CPU1读到`b=1`时，a还存在于Store Buffer中。

4、什么时候数据从CPU内写到store buffer？

A：当缓存行状态不是独占状态的时候，包括Cache line上没有该数据地址，将写入到Store Buffer并发出`Read Invalidate a`消息

5、什么时候数据从store buffer写到Cache line？

A：当收到其他CPU发出的对同数据地址的read response消息

6、什么时候数据会从Cache Line写回内存？

A：当该缓存行状态为M修改态（持有最新数据），并且地址数据将被替换成其他时，执行write back写回内存

7、什么时候CPU会处理Invalidate Queue中的失效消息？

A：暂时还未了解到具体处理时机，

8、内存屏障具体作用？

内存屏障smp_mb有两个作用，1，标记store buffer，在处理之后的写请求之前需要把store buffer中的数据apply到cache，2，标记invalidate queue，在加载之后的数据之前把invalidate queue中的消息都处理掉。smp_wmb写屏障，只会标记store buffer，smp_rmb读屏障，只会标记invalidate queue。

所谓重排序和可见性问题，其实都可以归为重排序（顺序）问题，可见性是store buffer和invalidate queue引起的顺序问题，写的时候先写了后面独占态的变量到cache line，读的时候没有先处理invalidate queue中的消息直接读了本地cache line中的旧值。所以内存屏障也分为写屏障和读屏障两种。

Store Buffer和Invalidate Queue设计思路：化同步为异步（非独占写时先写到store buffer， 处理失效消息时先添加到Invalidate Queue就直接返回invalid ack，再异步去），先返回ack确认消息，大幅减少等待耗时提高吞吐量。

参考链接：

[并发研究之CPU缓存一致性协议(MESI)](https://www.cnblogs.com/yanlong300/p/8986041.html)（https://www.cnblogs.com/yanlong300/p/8986041.html）

Cache一致性和内存模型：https://wudaijun.com/2019/04/cpu-cache-and-memory-model/

维基百科：MESI协议：https://en.wikipedia.org/wiki/MESI_protocol

