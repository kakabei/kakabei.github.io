---
layout: post
title: 服务发现原理
date: 2022-01-25 16:12:15
categories: 系统设计  
tags: 工作经验
excerpt: 在分布式系统中，微服务多且杂，服务发现很好的解决了服务的之间的调用问题。
---


# 一、概述

在微服务分布式框架中，服务被拆分成很微小服务，各自独立。由多个台机器组成，各服务之间相互调用，有管理各种服务的需求。 服务发现机制可以降低服务管理的成本。

在服务发现之前，服务之间的调用，可能要在各服务代码中写死IP 或者 通过配置文件读出服务之间的IP。 

服务发现机制则一分两步:

1、服务注册。 在服务启动时，把自己的**IP和端口** 通过服务注册定入到服务管理平台中，以便提供给其他主调用。

2、服务发现。其他方调用时则要先通过服务平台拿到这个服务其中的一个**IP和端口**，然后再调用。

如图：

![](/assets/system-design/service-discovery-2022-09-23_11-53-40.png)

说明： 

1、 server b 把自己注册到  service registry  叫做 **服务注册**。

2、 server  a  从 service registry 发现 server b 的节点信息叫做 **服务发现**。

3、最后 server a 调用 server b。

# 二、服务注册

服务注册，简单的说是，就是把自己的IP和端口写到服务管理平台上形成一组服务，告诉所有其他的服务，想调用我，可以通过服务管理平台上的服务名找到我的IP和端口。 

服务注册的大概框架图：

![](/assets/system-design/service-discovery-2022-09-23_00-58-18.png)

一般一个服务会有多个 IP 和端口。 如：

 ```json 
 # user server 
[
	{ 
		"ip":127.0.0.1,
		"port":8800
	},
	{ 
		"ip":127.0.0.2,
		"port":8800
	},
	{ 
		"ip":127.0.0.3,
		"port":8800
	}
]

# live server 
[
	{ 
		"ip":127.0.1.1,
		"port":8801
	},
	{ 
		"ip":127.0.1.2,
		"port":8801
	},
	{ 
		"ip":127.0.1.3,
		"port":8801
	}
]

```

说明：

1、业务服务会向服务平台注册一个服务名，把自己的 IP和端口号写入。 

2、一般情况下，同一个服务，所有端口最好一样，方便管理。 

3、以现在docker 的技术，一般是一个实例只能跑一个业务服务。但也有企业了节约成本，也会在同一个docker 跑多个服务。

4、有的解决方案是在服务管理平台手动的写入服务名和对应的IP列表。 服务名要全局惟一。 

# 三、服务发现

服务发现，即当server a 想请求 serve b 时，应该如何找到合适的IP和端口呢？如：

user sever 这个服务 有三台机器，  live server  也有三台机器。 live sever1  想请求 user server  的业务接口。它就应该先请求服务管理平台。

服务管理平台返回了 user server 中的一个IP和端口。 （负载均衡后的结果）


![](/assets/system-design/service-discovery-2022-09-23_11-09-13.png)

上图说明：

1、 live server1 想请求 user server 中的接口。

2、live server1 要请求服务管理平台，服务管理平台返回了 user server 其中的一个IP 和 端口。 （user server2）

3、live server1 再请求 user server2

这样就完成一个服务的调用。 



# 四、活跃检测

服务发现机制，除了在微服务分布式系统中方便服务之间的调用后，还是功能就两个是活跃检测和负载均衡。

即服务管理平台会向注册的服务发送请求，确认服务是否可用，如果服务不可用或机器已挂机。即标记该 IP 不可用，或者直接摘机。 

另一种方式是注册的服务定时的向服务管理平台发送确认的状态。告诉服务管理平台，自己还是活跃的。

当时发现服务不通时，可以进行摘机。即把IP从列表中移除掉。

所以，服务提供方一般都要求两个机器以上，这样可以起到容灾的作用。 要求高一点，可以要求机器在不同机房。当一个机房出问题时，另一个机器的服务可以支持着服务。 

![](/assets/system-design/service-discovery-2022-09-23_01-04-49.png)



# 五、负载均衡

服务请求时，提供方的服务是无状态的，会存在多个IP，我们只需求访问其中一个IP即可。 所以，服务管理平台同时也起了负载均衡的作用，只返回一个合适的IP。

当然，也有的策略是返回多个 IP， 让调用方自己去选择那个IP访问。 

这里也导致一个问题，即所有请求都会预先请求服务管理平台。流量大的时侯会有瓶颈。解决的方案的是，请求到一个IP后，可以在本地缓存几秒钟，减少因流量峰值而带来服务管理平台的压力。

[负载均衡策略：http://blog.xyecho.com/load-balancing-and-reverse-proxy/](http://blog.xyecho.com/load-balancing-and-reverse-proxy/)

# 五、服务发现技术点

一套完整的服务发现平台，所需要的技术点：

1、 集群、分布式。 要应对大流量的服务，就一定要用到分布式集群。同时也起到容灾的作用。 

2、强一致性、数据同步。由于分布式， 所以数据的一定要同步到每一台机器上，同时也要保证每一台机器的上的数据是一致的。

3、高并发、高可用。服务请求都要预先请求一次服务管理平台，所以流量会比较大。不能因为服务发现的加入而导致整体的性能下降太多。

4、部署管理简单。 简单的管理和好的体验，会让更多开发者去运用它。 

# 六、开源的解决方案

有资源的企业会基于源开放的解决方案上做一些二次开发，以适用于本企业的研发文化。 github 上也有一些比较靠谱的解决方案。如：
`etcd` 、`zookeeper` 、`consul` 等。都具有安全稳定，高可用，高并发。强一致性的特点。 

1、`etcd`  : [https://github.com/etcd-io/etcd](https://github.com/etcd-io/etcd)

2、`zookeeper` : [https://github.com/apache/zookeeper](https://github.com/apache/zookeeper)

3、`consul` :[https://github.com/hashicorp/consul](https://github.com/hashicorp/consul)


---
1、[深入了解服务注册与发现 : https://zhuanlan.zhihu.com/p/161277955](https://zhuanlan.zhihu.com/p/161277955)

2、[服务注册与发现的原理和实现 : https://zhuanlan.zhihu.com/p/409154290](https://zhuanlan.zhihu.com/p/409154290)


