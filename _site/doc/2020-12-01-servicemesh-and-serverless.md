Service Mesh和Serverless 

 刘俊海 《service mesh 微服务架构设计》 京东：https://item.jd.com/12730788.html

第10章  笔记 时间： 2020-12-01

### Serverless 简介

Serverless中文可以翻译为**“无服务器架构”**

Serverless不是指不需要服务器，主要是强调在Serverless下，业务不再需要固定的服务器资源，而是按需分配相应的服务资源。当业务请求比较多时，会多分配相应的服务实例；当业务请求比较少甚至为零时，可以不分配任何服务器资源。

Serverless架构和之前的架构相比，最大的差异是：业务服务不再是固定的常驻进程，而是真正按需启动和关闭的服务实例。

Serverless架构可以提供如下两点优势:

1. 按需使用的资源管理。
2. 简化业务运维复杂度。

在简化微服务管理复杂度上，Serverless和Service Mesh的目标是一致的，都是将微服务通信和服务治理相关的非功能需求从业务中剥离，对于Serverless是交给Serverless框架负责处理，对于Service Mesh是下沉到底层，成为通信基础设施的一部分。

 ### Serverless基础架构

在Serverless中，按需执行的代码片断称为**“函数”**，它是Serverless资源管理和调度的基本单位，因此Serverless有时也被称为FaaS（Function As A Service，函数即服务）。

Serverless架构大体由如下几个部分组成：

1. 函数管理。
2. 事件触发器。
3. 函数的路由和伸缩管理。

Serverless的发展历程和当前现状：

- 2014年11月，AWS发布了新产品Lambda，开启了全新的Serverless时代。 
- 2016年Google、Microsoft Azure相继推出自己的Cloud Function服务，
- 2017年国内公有云提供商阿里云和腾讯云也分别推出了各自的Serverless产品。
-  Google联合IBM、Pivotal、Redhat和SAP，一起推出了Knative，目的是实现Serverless标准化。

### Knative

Knative是一个以Kubernetes和Istio为基础的Serverless开源架构方案。

目的是解决Kubernetes环境下Serverless应用的构建、部署和运行。Knative和以往Serverless解决方案最大的不同是，Knative不仅管理函数，它的定位是管理所有的工作负载，除了Serverless关注的函数外，还包括普通的微服务，因此Knative强调的是通过相应的机制，用户不用再关注应用的伸缩和运维。

 架构上，Knative由**Build**、**Serving**和**Eventing** 3个核心组件组成。

- Build负责构建工作，把用户定义的函数和应用构建成容器镜像；
- Serving负责计算工作，具体包含Serverless实例的路由和访问，以及Serverless实例的按需伸缩；
- Eventing负责事件工作，自动完成事件的绑定和触发执行。
       

Knative Serving的主要概念如下：
1. Route：工作负载的路由规则，对应Istio的VirtualService。
2. Configuration：应用的最新配置，对应Kubernetes的Deployment。
3. Revison：每次对工作负载进行修改的快照，对应Istio的Subset。
4. Service：对工作负载的整个生命周期进行管理。

Knative的Serving系统做的事情和Kubernetes、Istio大体相同，但通过提供更高的抽象，屏蔽了Kubernetes、Istio的细节，自动帮应用管理好Deployment、VirtualService以及autoscaling之间的关系。

基于Kubernetes的Serverless产品和解决方案市场上已经有不少，比如Fission、Funktion、Kubeless等，但同时依赖Kubernetes和Istio这两个重量级的项目，Knative还是第一个。

Knative中为什么采用Kubernetes和Istio这么重的通信解决方案？Kubernetes已经成为容器调度和生命周期管理的事实标准，可以看成云原生时代的Linux操作系统；Istio虽然产生时间不长，但已经显现出强大的发展势头，当前在ServiceMesh和Kubernetes生态中的地位已经不可撼动，可以看成云原生时代的TCP/IP协议。

架构的角度看，这也是Unix设计哲学的体现——**整个系统采用模块化设计，每个模块只聚焦一件事情，并且做好做到极致**。