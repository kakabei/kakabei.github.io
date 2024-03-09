---
layout: post
title: 负载均衡和反向代理
date: 2020-08-18 16:12:15
categories: 系统设计
tags: 
excerpt: 负载均衡和反向代理
---


## 负载均衡

负载均衡是高可用网络基础架构的关键组件，通常用于将工作负载分布到多个服务器来提高网站、应用、数据库或其他服务的性能和可靠性。

#### 一个没有负载均衡的web架构

![](https://pic3.zhimg.com/80/v2-6a7f624cd5e776a6b0b05a051e65666e_720w.jpg)

#### 存在问题

1. 如果服务器宕机了，用户就无法访问到数据了。
2. 单位时间内请求人的比较，可能会超过它的处理极限，也可能请求不到数据。 

后端引入负载均衡器，可以缓解这个问题。通常要求所有后端服务器保证提供相同的内容，无论用户访问时昨到那了个服务响应。都能收到相同的内容。 

#### 负载均衡器的效用在于

* 防止请求进入不好的服务器
* 防止资源过载帮助
* 消除单一的故障点
* 水平扩展，提高性能和可用性，性价比高。

![](https://picb.zhimg.com/80/v2-6aa2607e04cc9d2f0d448f9fa80b2ae2_720w.jpg)

#### 负载均衡可以处理协议请求：

* http 
* https 
* tcp 
* udp 

其实负载均衡是一种解法的问题思路，和具体的协议没什么关系。特定协议可以自己写负载均衡器达到目的。 

负载均衡器如何选择要转发的后端服务器？

首先，要确保所和选择的服务器是健康的，一般会有一个健康服务器池（healthy pool）中进行选择。 

如何知道后端的服务是否健康，我猜负载均衡器会定时的检测后端的服务的状态，是否可用。　

#### 选择规则算法：

1. 轮询算法
2. 加权轮询算法
3. 随机算法
4. 加权随机算法
5. 哈希法
6. 一致性哈希
7. 最少连接算法  


负载均衡器还能帮助水平扩展，提高性能和可用性。使用商业硬件的性价比更高，并且比在单台硬件上垂直扩展更贵的硬件具有更高的可用性。

**水平扩展的缺陷：**

* 水平扩展引入了复杂度并涉及服务器复制

    * 服务器应该是无状态的:它们也不该包含像 session 或资料图片等与用户关联的数据。
    * session 可以集中存储在数据库或持久化缓存（Redis、Memcached）的数据存储区中。

*  缓存和数据库等下游服务器需要随着上游服务器进行扩展，以处理更多的并发连接。

#### nginx 负载均衡的策略：

1. 轮询（默认）： 每个请求按时间顺序逐一分配到不同的后端服务器，如果后端不通，则自动剔除。
2. weight  ： 指定轮询几率，weight和访问比率成正比， 用于后端服务器能不情况。 
3. ip_hash :  通过hash 用户访问IP，把用户指向同一个服务器上，这种方式会把用户固定在某一个服务器上，除非用户IP变了。 
4. fair （第三方）： 按后端服务嘎嘎的响应时间来分配请求，响应时间短的优先分配。 
5. url_hash (第三方) ：按访问url的hash结果来分配请求，使用每个url定向到一个（对应的）后端服务。 


#### 扩展阅读

* [浮动IP（FLOAT IP）](https://blog.csdn.net/readiay/article/details/53538085)
* [什么是负载均衡？https://zhuanlan.zhihu.com/p/32841479](https://zhuanlan.zhihu.com/p/32841479)
* [大神口中的服务器负载均衡到底是什么意思？https://cloud.tencent.com/developer/article/1480179](https://cloud.tencent.com/developer/article/1480179)
* [关于负载均衡的一切：总结与思考:https://www.cnblogs.com/xybaby/p/7867735.html](https://www.cnblogs.com/xybaby/p/7867735.html)


## 反向代理

反向代理是一种可以集中地调用内部服务，并提供统一接口给公共客户的 web 服务器。来自客户端的请求先被反向代理服务器转发到可响应请求的服务器，然后代理再把服务器的响应结果返回给客户端。

**带来的好处包括：**

* 增加安全性 - 隐藏后端服务器的信息，屏蔽黑名单中的 IP，限制每个客户端的连接数。
* 提高可扩展性和灵活性 - 客户端只能看到反向代理服务器的 IP，这使你可以增减服务器或者修改它们的配置。
* 本地终结 SSL 会话 - 解密传入请求，加密服务器响应，这样后端服务器就不必完成这些潜在的高成本的操作。 免除了在每个服务器上安装 X.509 证书的需要
*  压缩 - 压缩服务器响应.
*  缓存 - 直接返回命中的缓存结果.
*  静态内容 - 直接提供静态内容, HTML/CSS/JS图片视频等等

#### 扩展阅读

* [What is a Reverse Proxy vs. Load Balancer? https://www.nginx.com/resources/glossary/reverse-proxy-vs-load-balancer/](https://www.nginx.com/resources/glossary/reverse-proxy-vs-load-balancer/)
* [NGINX 架构 https://www.nginx.com/blog/inside-nginx-how-we-designed-for-performance-scale/](https://www.nginx.com/blog/inside-nginx-how-we-designed-for-performance-scale/)