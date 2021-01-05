## 图解HTTP



TCP/IP 指的是互联网所用到的协议族，而不是两个协议





REST:

#### REST是什么

`REST`即表述性状态传递（英文：`Representational State Transfer`，简称`REST`）是`Roy Fielding`博士在2000年他的博士论文中提出来的一种软件架构风格。表述性状态转移是一组架构约束条件和原则
 所以简而言之，`REST`是一种风格
 `REST`通常基于使用`HTTP`，`URI`，和`XML`（标准通用标记语言下的一个子集）以及`HTML`（标准通用标记语言下的一个应用）这些现有的广泛流行的协议和标准。`REST` 通常使用 `JSON` 数据格式。

#### RESTful是什么

那么既然`REST`是一种风格，`RESTful`又是什么呢？
 `RESTful`是满足这些约束条件和原则的应用程序或设计

主要是从三个方面来看

- 资源
  资源就是一个实体，你可以用一个`URI`（统一资源定位符）指向它，每种资源对应一个特定的`URI`。要获取这个资源，访问它的`URI`就可以，因此`URI`就成了每一个资源的地址或独一无二的识别符
- 表现层
  把资源呈现出来的形式叫表现层，可以用`HTML`格式、`XML`格式、`JSON`格式表现，`REST`中常用的是`JSON`格式
- 状态转化
  访问一个网站，就代表了客户端和服务器的一个互动过程。在这个过程中，势必涉及到数据和状态的变化。
  `HTTP`协议，是一个无状态协议。这意味着，所有的状态都保存在服务器端。因此，如果客户端想要操作服务器，必须通过某种手段，让服务器端发生"状态转化"。而这种转化是建立在表现层之上的，所以就是"表现层状态转化"。
  客户端用到的手段，只能是HTTP协议。
  1.**GET**用来获取资源
  2.**POST**用来新建资源（也可以用于更新资源）
  3.**PUT**用来更新资源
  4.**DELETE**用来删除资源。

综合上面的解释，我们总结一下什么是`RESTful`架构：
 （1）每一个`URI`代表一种资源；
 （2）客户端和服务器之间，传递这种资源的某种表现层；
 （3）客户端通过四个`HTTP`动词，对服务器端资源进行操作，实现"表现层状态转化"。

#### 设计误区

- `URI`不应该包含动词，因为它是一个资源，动词应该放在`HTTP`协议中
- `URI`不应该包含版本号，因为不同的版本，可以理解成同一种资源的不同表现形式，所以应该采用同一个`URI`

#### REST的优点

- `URI`具有很强可读性的，具有自描述性
- 充分利用 `HTTP` 协议本身语义
- 无状态，在调用一个接口（访问、操作资源）的时候，可以不用考虑上下文，不用考虑当前状态，极大的降低了复杂度
- `HTTP` 本身提供了丰富的内容协商手段，无论是缓存，还是资源修改的乐观并发控制，都可以以业务无关的中间件来实现



参考链接：https://www.jianshu.com/p/c85baf7e12a8

### 2.8 使用Cookie的状态管理

Cookie 会根据从服务器端发送的响应报文内的一个叫做 **Set-Cookie** 的 首部字段信息，通知客户端保存 Cookie。当下次客户端再往该服务器 发送请求时，客户端会自动在请求报文中加入 Cookie 值后发送出去。

```http
Response Headers:

Access-Control-Allow-Credentials: true
Access-Control-Allow-Origin: https://segmentfault.com
Connection: keep-alive
Content-Length: 4
Content-Type: application/octet-stream
Date: Mon, 21 Sep 2020 09:46:00 GMT
Set-Cookie: io=VViLyL3uyfpR3tRowckp; Path=/; HttpOnly

Request Headers:
Cookie: PHPSESSID=web1~5nl63jiot6ifaiura65t0rck25; sf_remember=8f8ed94792bbd89ab27ea61062de1eea; io=VViLyL3uyfpR3tRowckp
```

