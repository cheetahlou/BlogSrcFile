---
title: HashMap源码浅探
date: 2019-09-03 16:53:00
tags: 
- HashMap
- 源码

---

## HashMap源码浅探

本文将会持续更新HashMap源码的探秘之旅

### 底层数据结构

 HashMap的底层数据结构主要由一个数组 ，数组元素为Entry链表，当链表元素插入超过8个将转化为红黑树（treeifyBin）。详见以下方法分解。

### hash方法

```java
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```

*^ 表示位运算符 异或 ，即 0 1不同为1，相同为0*

对象的hashCode存放于对象头的Mark Word中，在64位虚拟机中占 31 bit，如`100 0010 0011 1000 1000 0011 1100 0000`。

HashMap中的hash变量不同于对象的hashCode，而是先调用hashCode()之后，再用hashCode和它的右移16位之后的二进制进行异或操作，为什么要这么做呢？这样做可以打乱原来的hashCode，让hashCode的所有位都参与运算，后续确定数组位置的方法是hash再对数组大小取模，同样采用**位运算与 &** ，index = h & (n- 1)，其中n为数组大小，因为数组大小（HashMap容量）总是为2的k次幂，即二进制表示为 `0001 0000`...，所以(2^k - 1) 二进制为`0000 1111` ，和 h 做按位与，正常思维是 h % n ，尽量分布均匀减少碰撞，然后容量是2的k次幂的话可以使用二进制的**位运算与**提高运算效率。h % n = h & (n - 1) = h & (2^k - 1)

### put方法

作用： 	put()添加一个键值对到HashMap中



### resize 扩容方法

JDK8之前resize方法的多线程下死循环问题：

```java
void resize(int newCapacity) {
	...
	transfer(newTable);//将旧数组中的元素转移到新数组上来
	...
}


```

JDK8后对该问题的修改（Oracle不认为这是一个问题，表示如果要线程安全你们就去用ConcurrentHashMap呀）

先看源码：

```

```



### QA

Q：为什么HashMap的容量要是2的n次幂 2^n，

A：一 是因为在计算元素的数组索引位置时，使用位运算用hash和容量n-1做逻辑与 ，二 是因为扩容时只要判断新hash高位是否为1（用hash & oldCap == 0判断如 0001 0011 & 0001 0000）因为当容量为2的n次幂时二进制表示为0001 0000···，确定旧链表中元素在新数组中的索引位置newTable[i]是保持不变( 还是增加一个旧容量的大小 newTable[i+oldCap]。



### 思考 & 举一反三

hashMap中大量运用了位运算符，有时两数之间的运算和判断用位运算性能会更好，因为机器码二进制直接进行运算不用转进制，不过也会带来一定可读性的丢失，还需慎用。