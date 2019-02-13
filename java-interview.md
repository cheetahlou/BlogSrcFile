---
title: java-interview
date: 2019-02-01 23:37:00
tags: 
---

﻿

***
 1. Java中线程的sleep()和wait()有什么区别
 - 1、这两个方法来自不同的类。分别是：sleep来自Thread类，而wait来自Object类。
  sleep是Thread的静态类方法，谁调用的谁去睡觉，即使在a线程里调用了b的sleep方法，实际上还是a去睡觉，要让b线程睡觉要在b的代码中调用sleep。

  <!--more-->

***
 - 2、最主要是**sleep方法没有释放锁**，而wait方法释放了锁，使得其他线程可以使用同步控制块或者方法。
	**sleep不出让系统资源**；wait是进入线程等待池等待，出让系统资源，其他线程可以占用CPU。一般wait不会加时间限制，因为如果wait线程的运行资源不够，再出来也没用，要**等待其他线程调用notify**/notifyAll唤醒等待池中的所有线程，才会进入就绪队列等待OS分配系统资源。sleep(milliseconds)可以用时间指定使它自动唤醒过来，如果时间不到只能调用interrupt()强行打断。
Thread.Sleep(0)的作用是“触发操作系统立刻重新进行一次CPU竞争”。
 - 3、使用范围：wait，notify和notifyAll只能在同步控制方法或者同步控制块里面使用，而sleep可以在任何地方使用 
   synchronized(x){ 
      x.notify() 
     //或者wait() 
   }
- 4、sleep必须捕获异常，而wait，notify和notifyAll不需要捕获异常

 2. 将一个字符串逆序，不要使用反转函数

        String message = "he saw a racecar";
        StringBuilder rev = new StringBuilder();
        for (int i = message.length() - 1; i >= 0; i--) 
            rev.append(message.charAt(i));
        System.out.println(rev.toString());

***
 3. **forward和redirect的区别**
forward方式：
```
request.getRequestDispatcher("/somePage.jsp").forward(request, response); 
```
   redirect方式：

```response.sendRedirect("/somePage.jsp"); ```

　　forward是服务器内部重定向，程序收到请求后重新定向到另一个程序，客户机并不知道；redirect则是服务器收到请求后发送一个状态头给客户，客户将再请求一次，这里多了两次网络通信的来往。当然forward也有缺点，就是forward的页面的路径如果是相对路径就会有些问题了。forward会将request state ,bean等等信息带往下一个jsp。redirect是送到client端后再一次request,所以资料不被保留.使用forward你就可以用getAttribute()来取的前一个jsp所放入的bean等等资料。

　　在网上看到一些帖子，总结了一些区别，可以从以下几个方面来看：

   1、从地址栏显示来说

　　forward是服务器请求资源,服务器直接访问目标地址的URL,把那个URL的响应内容读取过来,然后把这些内容再发给浏览器.浏览器根本不知道服务器发送的内容从哪里来的,所以它的地址栏还是原来的地址.

　　redirect是服务端根据逻辑,发送一个状态码,告诉浏览器重新去请求那个地址.所以地址栏显示的是新的URL.所以redirect等于客户端向服务器端发出两次request，同时也接受两次response。

   2、从数据共享来说

　　forward:转发页面和转发到的页面可以共享request里面的数据.
　　redirect:不能共享数据.

　　redirect不仅可以重定向到当前应用程序的其他资源,还可以重定向到同一个站点上的其他应用程序中的资源,甚至是使用绝对URL重定向到其他站点的资源.forward,方法只能在同一个Web应用程序内的资源之间转发请求.forward 是服务器内部的一种操作.redirect 是服务器通知客户端,让客户端重新发起请求.所以,你可以说 redirect是一种间接的请求, 但是你不能说"一个请求是属于forward还是redirect "
   3、从运用地方来说

　　forward:一般用于用户登陆的时候,根据角色转发到相应的模块.

　　redirect:一般用于用户注销登陆时返回主页面和跳转到其它的网站等.

   4、从效率来说
　　forward:高.
　　redirect:低.

***
 4. **Java自动类型转换**
	自动类型转换遵循下面的规则：

		1.若参与运算的数据类型不同，则先转换成同一类型，然后进行运算。
	

		3.所有的浮点运算都是以双精度进行的，即使仅含float单精度量运算的表达式，也要先转换成double型，再作运算。
		
		4.char型和short型参与运算时，必须先转换成int型。
		
		5.在赋值运算中，赋值号两边的数据类型不同时，需要把右边表达式的类型将转换为左边变量的类型。如果右边表达式的数据类型长度比左边长时，将丢失一部分数据，这样会降低精度。
		下图表示了类型自动转换的规则： 
![自动类型转换](http://uploadfiles.nowcoder.com/images/20150917/415611_1442458661106_F4A62FDD254F710F39378C754ED65E61) 
***
 2. **声明二维数组的几种写法**
```
    int a[][];//二维数组其实就是数组成员是一维数组的一维数组
    int []b[];//这5个都可以看成艺术组为元素的数组
    int[] c[];
    int [][]d;
    int[] []e;
```
***
 - **什么是 AOP?**

	面向方面的编程 `(Aspect-oriented programming)`,
它可以运行期动态代理实现在不修改源代码的情况下给程序动态统一添加功能的一种技术. 比如检测某个模块的运行时间. 加入额外的功能 (introduce)
下面是专业的所法 :
利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率
主要的功能是：**日志记录，性能统计，安全控制，事务处理，异常处理**等等。

***
