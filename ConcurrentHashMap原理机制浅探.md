---
title: AQS原理机制浅探
date: 2019-11-18 16:14:00
tags: 
- AQS
- ReentrantLock
---



Segment的get操作实现非常简单和高效。先经过一次再散列，然后使用这个散列值通过散 列运算定位到Segment，再通过散列算法定位到元素，代码如下。



```java
public V get(Object key) {
int hash = hash(key.hashCode());
return segmentFor(hash).get(key, hash);
}

```



get操作的高效之处在于整个get过程不需要加锁，除非读到的值是空才会加锁重读。我们 知道HashTable容器的get方法是需要加锁的，那么ConcurrentHashMap的get操作是如何做到不 加锁的呢？原因是它的get方法里将要使用的共享变量都定义成volatile类型，如用于统计当前 Segement大小的count字段和用于存储值的HashEntry的value。定义成volatile的变量，能够在线 程之间保持可见性，能够被多线程同时读，并且保证不会读到过期的值，但是只能被单线程写 （有一种情况可以被多线程写，就是写入的值不依赖于原值），在get操作里只需要读不需要写 共享变量count和value，所以可以不用加锁。之所以不会读到过期的值，是因为根据Java内存模 型的happen before原则，对volatile字段的写入操作先于读操作，即使两个线程同时修改和获取 volatile变量，get操作也能拿到最新的值，这是用volatile替换锁的经典应用场景。

