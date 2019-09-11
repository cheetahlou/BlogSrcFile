---
title: RocketMQ使用和运行机制原理
date: 2019-09-11 14:44:00
tags: 
- 中间件
- RocketMQ

---

## RocketMQ使用和运行机制原理

### RocketMQ特点和优势：

####  支持事务消息：

​	注意事务型消息并非分布式事务，只是为了保证本地事务与消息发送的一致性。通过Offset下标方式找到消息进行修改第一阶段的Prepare消息的状态

#### 支持消息重试：

​	最好提供一种定时重试机制，减小Broker的压力

####  支持定时消息：

​	支持一定时间精度的定时消息，如5s，10s，1m等，不支持任意时间精度的定时消息



### RocketMQ架构和原理：

  NameServer集群用来作寻址路由，从Broker集群中读取可用的Broker地址返回给生产者集群或消费者集群，Broker集群主要负责消息的生产和消费，即消息队列

####  顺序消息： 

​	Producer 单线程顺序发送，且发送到同一个队列，这样 Consumer 就可以按照 Producer 发送的顺序去消费。消息按顺序即消息都在同一条队列中，使用单线程单队列 ，或自定义selector，使其每次都返回同一条队列（比如每次都返回List<MessageQueue> mqs的第一条队列）

####  如何避免消息重复：

​	RocketMQ本身并不保证消息不重复，如网络异常等情况，所以需要在业务端进行去重 保证消息消费的幂等性

####  优先级消息：

​	制定不同的topic来表示优先级，指定某几个队列来传递高优先级消息，指定几个队列来传递低优先级消息。

####  消息堆积：

​	值得注意的是，消息堆积并不一定是坏事，堆积过多才会有风险。

如果消息堆积过多，可以**临时加消费组扩容消费**。 

消息中间件除了异步解耦作用之外，另一个重要的作用就是挡住前端过来的数据洪峰作缓冲，保证后端系统的稳定性，这就要求具备一定的堆积能力

1. 消息堆积到内存Buffer中 
2. 消息堆积到持久化存储系统中，例如 DB，KV 存储，文件记录形式。
  当消息不能在内存 Cache 命中时，要不可避免的访问磁盘，会产生大量读 IO，读 IO 的吞吐量直接决定了
  消息堆积后的访问能力。



### 与其他消息中间件对比：

![img_5a295d3e7094050428c129136e33fafa.png](https://yqfile.alicdn.com/img_5a295d3e7094050428c129136e33fafa.png)

项目开源主页：https://github.com/alibaba/RocketMQ

参考资料： [官方RocketMQ原理简介](http://gd-rus-public.cn-hangzhou.oss-pub.aliyun-inc.com/attachment/201604/08/20160408165024/RocketMQ_design.pdf)：(http://gd-rus-public.cn-hangzhou.oss-pub.aliyun-inc.com/attachment/201604/08/20160408165024/RocketMQ_design.pdf)