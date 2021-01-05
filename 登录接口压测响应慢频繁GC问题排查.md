## 登录接口压测响应慢频繁GC问题排查

2020.5.22

最近项目组针对几个较重要的接口进行了几十个小时的压测，发现登录接口的压测呈现了一种响应慢且越来越慢的趋势，CPU 也居高不下

## 压测情况

查看CPU占用情况如图所示：  

![image.png](https://i.loli.net/2020/05/22/ezKgLu4nWBNUAqM.png)    

  找到对应服务包是鉴权服务(auth)：        

![image.png](https://i.loli.net/2020/05/22/IervEGFCANRMDnB.png)

  持续运行3小时的CPU占用曲线图：        ![img](https://i.loli.net/2020/05/22/enyW2VqcRhgLd5k.png)       **结论：sc-auth包中的登录接口，响应时间长，占用CPU较高，需要优化。**

## 排查思路

业务场景很简单，账号密码鉴权登录接口。

### 先排查为什么CPU占用率高

从top命令的结果发现。pid为15082的java进程CPU利用持续占用过高，达到了惊人的222.3%

![image.png](https://i.loli.net/2020/05/22/D18l2EMowAO4bSR.png)

### 定位问题线程

使用`ps -mp pid -o THREAD,tid,time`命令查看该进程的线程情况，发现该进程的15805到15099线程占用率都比较高

![image.png](https://i.loli.net/2020/05/22/pXADVdJRZ2qztHl.png)

查看问题线程堆栈


```bash
# 将线程id转换为16进制
[root@app-03 ~]# printf "%x\n" 15085
3aed
[root@app-03 root]$ printf "%x\n" 15089
3af1
```


jstack查看线程堆栈信息
jstack命令打印线程堆栈信息，命令格式(pid为进程id，tid为线程)：`jstack pid |grep tid`

```bash
[xx@app-03 root]$ jstack 15082 | grep 3af1
"Gang worker#3 (Parallel GC Threads)" os_prio=0 tid=0x00007ff54c05f800 nid=0x3af1 runnable 
[xx@app-03 root]$ jstack 15082 | grep 3aed
"Gang worker#0 (Parallel GC Threads)" os_prio=0 tid=0x00007ff54c05a000 nid=0x3aed runnable
```


从上面可以看出，这些都是GC的线程。那么可以推断，很有可能就是内存不够导致GC不断执行。接下来我们就需要查看
gc 内存的情况

查看Spring Admin监控中的JVM信息发现parNew和CMS 的GC都非常频繁，GC时间也很长，达到了一个多小时，而且几乎压测账号的每次请求都会导致一次CMS GC，起初没有分析我们怀疑是图中鲜红的Non-Heap Memory占用过高的问题

![image.png](https://i.loli.net/2020/05/22/ghqxHEvns2tz9aU.png)



jstat -gc 查看垃圾回收统计

![image.png](https://i.loli.net/2020/05/22/AfipGeo5jSNVUv7.png)

> - **S0C：**第一个幸存区的大小
> - **S1C：**第二个幸存区的大小
> - **S0U：**第一个幸存区的使用大小
> - **S1U：**第二个幸存区的使用大小
> - **EC：**伊甸园区的大小
> - **EU：**伊甸园区的使用大小
> - **OC：**老年代大小
> - **OU：**老年代使用大小
> - **MC：**方法区大小
> - **MU：**方法区使用大小
> - **CCSC:**压缩类空间大小
> - **CCSU:**压缩类空间使用大小
> - **YGC：**年轻代垃圾回收次数\*
> - **YGCT：**年轻代垃圾回收消耗时间
> - **FGC：**老年代垃圾回收次数\*
> - **FGCT：**老年代垃圾回收消耗时间
> - **GCT：**垃圾回收消耗总时间

从其中的**YGC**、**FGC**等列可以看到 每秒钟大约有4、5次FGC和6次YGC

要分析具体的GC情况，首先可以通过增加JVM配置参数`-XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintHeapAtGC`来查看更详细GC日志。再用以下的heap dump定量分析实时进程内存情况。(*Heap Dump*也叫堆转储*文件*，是一个Java进程在某个时间点上的内存快照，常用作分析内存泄漏)

### `jstack` 和 `jmap` 分析进程堆栈和内存状况

我这边选择导出heap dump文件`heapdump2020-05-21-20-21-live795350237996816401.hprof.gz`到本地用MAT工具进行内存分析（mat下载地址 https://www.eclipse.org/mat/downloads.php，使用可参考[MAT使用-jvm内存溢出问题分析定位](https://www.jianshu.com/p/82b25cf8cfde?utm)）

最终在这里定位了问题具体的原因和进一步定位到具体代码

使用MAT File -> open heap dump...打开并解析dump文件，

总览结果，可以看到

![image.png](https://i.loli.net/2020/05/22/nLHBeDlsaJVv7x3.png)

 **Actions > The Histogram** (查看堆栈中java类 对象[Objects]个数、[Shallow Heap]individual objects此类对象占用大小、[Retained Heap]关联对象占用大小) -> Retained Heap倒叙排序(重点关注内存占用高对象) -> **Merge Shortest Paths to GC roots -> exclude all phantom/weak/soft etc.reference(排除所有虚弱软引用) ->查看剩余未被回收的强引用对象占用原因 & GC Roots。**

![image.png](https://i.loli.net/2020/05/22/aBsFLZez2gPiqhk.png)

Merge到GC roots之后之后，排除所有虚弱软引用

![image.png](https://i.loli.net/2020/05/22/Wirmx2UcAQeOHDK.png)

进入Overall 的 Leak Suspects 

![image.png](https://i.loli.net/2020/05/22/qNvtzmRuCBZs8GH.png)

再进入Details查看详情

Accumulated Objects in Dominator Tree*（以当前问题对象为根的Dominator支配树）

![image.png](https://i.loli.net/2020/05/22/XcsrT8R5kYHLWV6.png)

线程栈情况，太棒了，可以直接看到此线程在运行的代码具体类方法和行号

![image.png](https://i.loli.net/2020/05/22/Pq6v9TbkjrLxaQN.png)

### 问题代码

下面直接到问题源代码处

![image.png](https://i.loli.net/2020/05/22/xgcr7TEeznifDdh.png)

### 验证猜想和修复验证

使用新账号不停调用登录验证问题，果然接口响应时间都在几十毫秒左右，对比之前的压测账号，可以发现明显响应快，但也呈一个持续微幅增大的趋势，和我们之前猜想符合。

![image.png](https://i.loli.net/2020/05/22/eHoSfd6N7GwpUtF.png)

下图为跑了几十个小时的压测账号的响应时间统计，都在1.5s以上

![image.png](https://i.loli.net/2020/05/22/BTLCIS36gJ85VWz.png)

修改问题代码之后的压测结果、CPU占用率和GC情况如下，可以看到，响应时间都在几十毫秒100s单用户成功请求4329次，比之前提升了10倍性能，非常快：

![image.png](https://i.loli.net/2020/05/22/kBqrhpfH9s4VMYy.png)

![image.png](https://i.loli.net/2020/05/22/I7C6hHPJ9rWqu31.png)

![image.png](https://i.loli.net/2020/05/22/veGHwZPqDYsayAI.png)



电脑本地模拟Set大对象FGC：

```shell
{Heap before GC invocations=25 (full 4):
 PSYoungGen      total 818176K, used 91111K [0x000000076b200000, 0x00000007a9500000, 0x00000007c0000000)
  eden space 727040K, 0% used [0x000000076b200000,0x000000076b200000,0x0000000797800000)
  from space 91136K, 99% used [0x0000000799580000,0x000000079ee79d10,0x000000079ee80000)
  to   space 145920K, 0% used [0x00000007a0680000,0x00000007a0680000,0x00000007a9500000)
 ParOldGen       total 265728K, used 252614K [0x00000006c1600000, 0x00000006d1980000, 0x000000076b200000)
  object space 265728K, 95% used [0x00000006c1600000,0x00000006d0cb1bc0,0x00000006d1980000)
 Metaspace       used 76389K, capacity 77814K, committed 77848K, reserved 1116160K
  class space    used 10007K, capacity 10271K, committed 10280K, reserved 1048576K
2020-05-29T19:37:05.672+0800: [Full GC (Ergonomics) [PSYoungGen: 91111K->0K(818176K)] [ParOldGen: 252614K->257705K(587776K)] 343726K->257705K(1405952K), [Metaspace: 76389K->76389K(1116160K)], 0.1815747 secs] [Times: user=0.84 sys=0.00, real=0.18 secs] 
Heap after GC invocations=25 (full 4):
 PSYoungGen      total 818176K, used 0K [0x000000076b200000, 0x00000007a9500000, 0x00000007c0000000)
  eden space 727040K, 0% used [0x000000076b200000,0x000000076b200000,0x0000000797800000)
  from space 91136K, 0% used [0x0000000799580000,0x0000000799580000,0x000000079ee80000)
  to   space 145920K, 0% used [0x00000007a0680000,0x00000007a0680000,0x00000007a9500000)
 ParOldGen       total 587776K, used 257705K [0x00000006c1600000, 0x00000006e5400000, 0x000000076b200000)
  object space 587776K, 43% used [0x00000006c1600000,0x00000006d11aa5a8,0x00000006e5400000)
 Metaspace       used 76389K, capacity 77814K, committed 77848K, reserved 1116160K
  class space    used 10007K, capacity 10271K, committed 10280K, reserved 1048576K
}
```

dev set大小 9w =  byte[]占用大小 43MB

推算 压测环境  Set大小 36w =  byte[]占用大小 170M

### 复盘总结

排查思路：

- 观察到压测报告中接口慢同时伴随着CPU高占用，且都出现在同一接口中，伴随情况大概率指向同一问题，先排查易排查的CPU高占用问题

- 确定到具体java服务后通过jstat、jps、jinfo等JDK命令观察运行情况
- Spring Admin观察JVM运行和内存占用情况，有明显的频繁FGC问题，考虑大对象占用
- 用GC日志和heap dump分析当时的具体线程和对象的内存占用情况，找到大对象对应的线程和执行程序代码
- 快速浏览接口逻辑代码 找到可能产生大对象的问题代码（或者dump文件分析时候可以直接看到项目代码位置）
- 本地debug调试，启动时加上JVM配置参数分析复现和定位问题
- 修改代码本地运行观察GC频率，提交更改，观察接口响应时间、CPU占用率、GC频繁情况、内存占用等综合情况，验证是否解决问题

中间走了一些弯路，比如看到Non-heap Memory占用很高以为是元空间配置-XX:*MetaspaceSize*不够，jstat -gc的情况也显示元空间占用率很高，但是后来发现服务器上的配置 -XX:*MetaspaceSize* = 256m其实够用，也问询了一下写这段代码的同事，他也说是去年一次业务上的改造要兼容多客户端登录不被踢的奇怪需求，后来没有这个需求后代码也没有还原导致一直运行。

过程中还用到了压测工具Gatling的使用，轻量级的压测工具也比较容易上手。



#### 附上压测demo

自己简单写了一个压测脚本登录单接口单用户压测一定时间

```scala
package computerdatabase.advanced

import io.gatling.core.Predef._
import io.gatling.http.Predef._
import scala.concurrent.duration._

class AuthLoginPressureTest extends Simulation {

  val httpConf = http
    .baseUrl("https://xxx.com/api")
  //注意这里,设置提交内容type
  val contentType = Map(
    "Content-Type" -> "application/x-www-form-urlencoded",
    "Authorization" -> "Basic aW9jOmlvYw=="
  )
  //持续时长
  val duration = 100
  //声明scenario
  val scn = scenario("form Scenario")
    .during(duration){
      exec(http("authLogin_test") //http 请求name
        .post("/auth/oauth/token") //post地址, 真正发起的地址会拼上上面的baseUrl http://computer-database.gatling.io/computers
        .headers(contentType)
        .formParam("username", "zzz") //form 表单的property name = name, value=Beautiful Computer
        .formParam("password", "123456")
      )
    }
  setUp(scn.inject(atOnceUsers(1)).protocols(httpConf))
}
```



### 参考资料

[线上java程序CPU占用过高问题排查](https://blog.csdn.net/u010862794/article/details/78020231)：https://blog.csdn.net/u010862794/article/details/78020231

[MAT使用-jvm内存溢出问题分析定位](https://www.jianshu.com/p/82b25cf8cfde?utm)：https://www.jianshu.com/p/82b25cf8cfde?utm

[Gatling官方文档](https://gatling.io/docs/current/general/scenario)



```bash
{Heap before GC invocations=220 (full 158):
 par new generation   total 235968K, used 1493K [0x00000000c0000000, 0x00000000d0000000, 0x00000000d0000000)
  eden space 209792K,   0% used [0x00000000c0000000, 0x00000000c0175638, 0x00000000ccce0000)
  from space 26176K,   0% used [0x00000000ce670000, 0x00000000ce670000, 0x00000000d0000000)
  to   space 26176K,   0% used [0x00000000ccce0000, 0x00000000ccce0000, 0x00000000ce670000)
 concurrent mark-sweep generation total 786432K, used 92911K [0x00000000d0000000, 0x0000000100000000, 0x0000000100000000)
 Metaspace       used 99710K, capacity 101056K, committed 102400K, reserved 1138688K
  class space    used 12031K, capacity 12260K, committed 12544K, reserved 1048576K
2020-07-29T18:09:47.791+0800: [Full GC (Metadata GC Threshold) 2020-07-29T18:09:47.791+0800: [Heap Dump (before full gc): Dumping heap to /app/sc-event-center/logs/java_pid3966.hprof.259 ...
Heap dump file created [156996397 bytes in 6.942 secs]
, 6.9421021 secs]2020-07-29T18:09:54.733+0800: [CMS: 92911K->92910K(786432K), 0.2455402 secs]2020-07-29T18:09:54.979+0800: [Heap Dump (after full gc): Dumping heap to /app/sc-event-center/logs/java_pid3966.hprof.260 ...
Heap dump file created [155464284 bytes in 7.033 secs]
, 7.0332201 secs] 94405K->92910K(1022400K), [Metaspace: 99710K->99710K(1138688K)], 14.2212031 secs] [Times: user=14.02 sys=0.20, real=14.22 secs] 
Heap after GC invocations=221 (full 159):
 par new generation   total 235968K, used 0K [0x00000000c0000000, 0x00000000d0000000, 0x00000000d0000000)
  eden space 209792K,   0% used [0x00000000c0000000, 0x00000000c0000000, 0x00000000ccce0000)
  from space 26176K,   0% used [0x00000000ce670000, 0x00000000ce670000, 0x00000000d0000000)
  to   space 26176K,   0% used [0x00000000ccce0000, 0x00000000ccce0000, 0x00000000ce670000)
 concurrent mark-sweep generation total 786432K, used 92910K [0x00000000d0000000, 0x0000000100000000, 0x0000000100000000)
 Metaspace       used 99710K, capacity 101056K, committed 102400K, reserved 1138688K
  class space    used 12031K, capacity 12260K, committed 12544K, reserved 1048576K
}

从JDK8开始，任何GC，都会默认打印GC Cause，所以看到上面的Full GC是因为Metadata GC Threshold触发的，也就是Metaspace committed的内存加上这次要分配的内存达到了MetaspaceSize的阈值。
```

JVM参数：

```
打印GC日志详细
-XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintHeapAtGC

追踪类加载和类卸载的情况
-XX:TraceClassLoading -XX:TraceClassUnloading

GC前后生成heapdump文件
-XX:+HeapDumpBeforeFullGC -XX:+HeapDumpAfterFullGC

```

Metaspace committed的内存加上这次要分配的内存达到了MetaspaceSize的阈值，Metaspce触发的GC都是Full GC

![元空间满时jstat gc情况.png](https://i.loli.net/2020/07/29/xw92JFe1MtPaRdo.png)