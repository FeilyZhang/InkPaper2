title: "计算机网络应用层"
date: 2020-01-21 21:49:14 +0800
update: 2020-01-21 21:49:14 +0800
author: me
cover: "-/images/network.jpg"
tags:
    - Computer Network
preview: HTTP协议，即超文本传输协议(HyperText Transfer Protocol, HTTP)，是Web的核心，可以理解为是客户端与服务器在应用层数据传输上达成的共识，其规定了HTTP请求报文与响应报文的格式，从而服务器能够正确解析HTTP请求，客户端也可以正确解析HTTP响应。

---

# 一、HTTP协议

HTTP协议，即超文本传输协议(HyperText Transfer Protocol, HTTP)，是Web的核心，可以理解为是客户端与服务器在应用层数据传输上达成的共识，其规定了HTTP请求报文与响应报文的格式，从而服务器能够正确解析HTTP请求，客户端也可以正确解析HTTP响应。

**HTTP协议是可靠的、无状态协议。**具体解释如下

+ HTTP协议使用TCP协议作为传输层协议，由于TCP协议提供可靠的数据传输服务，因此，HTTP协议是可靠的协议；
+ HTTP服务器不存储任何有关客户状态的信息，这意味着客户端在短时间内的重复请求依然会得到服务器的完全响应，因此是无状态的。

## 1.1 非持续连接和持续连接

对于每一个请求/响应对，如果经单独的TCP连接发送，那么就属于非持续连接，若是经相同的TCP连接发送，则属于持续连接。HTTP在默认方式下使用持续连接，但也可以配置为非持续连接。

简单来说，非持续连接中，一个TCP连接在服务器发送完对象之后就立即关闭，也就是说对于客户端的每次请求，服务器都需要建立单独的TCP连接进行处理，更进一步而言，客户端与服务器建立的TCP连接，只分别传输了一个请求报文和一个响应报文。在此基础之上，我们更关注的是非持续连接中一个TCP连接自始(connect)至终(close)所花费的时间，这个时间几乎是客户端一次请求动作的作出到接收到整个文件的全部时间，应该如何定义呢？

在给出定义之前，我们先引入往返时间(Round-Trip Time, RTT)的概念：

> 往返时间(RTT)，是指一个短分组从客户到服务器然后再返回客户所花费的时间。


众所周知，一个TCP连接的建立，需要经过“三次握手”，即客户端先向服务器发送一个小的TCP报文段，然后服务器用一个小的TCP报文段作出响应，最后客户端向服务器返回确认。可见，前两次握手对应一次往返时间(RTT)，即先是客户端到服务器的端到端传输，再是服务器到客户端的端到端传输。第三次握手时，客户端一并向服务器发送请求报文，服务器随后在正式的TCP会话中响应，这就构成了另一个完整的RTT。此外，服务器对用户的响应，还包括服务器本地检索并传输(在本地传输，即HTML文档被推向往客户端的链路之前的数据传输)的时间。也就是说，**一个TCP连接或者客户端一次请求动作的作出到接收到整个文件的总的响应时间就是两个RTT加上服务器传输HTML文件的时间。**

之前定义过端到端时延，事实上，一个RTT就是两次端到端时延，那么总的响应时间其实就是四次端到端时延再加上服务器传输HTML文件的时间。

至此，非持续连接的缺陷显而易见，即

+ **资源浪费**。对于客户端的每次对象请求，服务器都要建立和维护一个全新的TCP连接，而每次TCP连接的建立，在客户端和服务器中都要分配TCP缓冲区和保持TCP变量，造成不必要的资源浪费。
+ **时延较高**。对于客户端的每次对象请求，客户端都要经受两倍的RTT交付时延，一个RTT时延用于创建TCP连接，另一个RTT时延用于请求和接收对象。

在采用HTTP/1.1持续连接的情况下，服务器在发送响应之后不关闭TCP连接而是保持该TCP连接的打开，以便后续的其余对象请求可以重复利用该TCP连接，这就意味着后续的对象请求减少了一半的时延，即减少到了一个RTT时延。那么该TCP连接会一直保持打开状态吗？并不会，如果一个TCP连接在一个可配置的时间间隔内未被利用，那么该连接就会被HTTP服务器关闭，从而资源得以释放。

HTTP的默认模式是使用带流水线的持续连接，而在HTTP 1.1基础上构建的HTTP/2允许在相同连接中进行多个请求和回答交错，并增加了在该连接中优化HTTP报文请求和回答的机制。

|            | 定义 | TCP连接| 时延 | 评价 |
|------------|------|--------|------|------|
| 非持续连接 | 每个请求/响应对，经单独TCP连接发送 | 用完即关 |  至少两倍RTT | 资源浪费、时延较高|
| 持续连接   | 每个请求/响应对，经相同TCP连接发送 | 重复利用 | 一个RTT | 资源利用率高、时延较低 |