---
layout: post
title: 腾讯云容器团队内部Istio 笔记
date: 2020-10-16 16:12:15
categories: 学习笔记
tags: 系统设计 service-mesh
excerpt: 腾讯云容器团队内部Istio专题分享
---

文章来源： [腾讯云容器团队内部Istio专题分享](https://www.servicemesher.com/blog/istio-the-king-of-service-mesh/)
作者：钟华

微服务架构是更为复杂的分布式系统，它给运维带来了更多挑战, 这些挑战主要包括资源的有效管理和服务之间的治理, 如:

新的分布式系统:微服务架构 带来了新挑战， 资源的有效管理和服务之间的治理。 其实分布式系统本质上都有这样的挑战，但微服务架构快速发展，让这一些问更加突出了。

* 服务注册, 服务发现
* 服务伸缩
* 健康检查
* 快速部署
* 服务容错: 断路器, 限流, 隔离舱, 熔断保护, 服务降级等等
* 认证和授权
* 灰度发布方案
* 服务调用可观测性, 指标收集
* 配置管理

在很多微服务架构中，都是通过架构api、sdk。做入侵式的开发，在架构中屏蔽了底层网络的复杂性，提供服务注册发现、服务RPC通信、服务配置管理、服务负载均衡、路由限流、容错、服务监控及治理、服务发布及升级等通用能力。

比较典型的产品有:
* 分布式RPC通信框架: COBRA, WebServices, Thrift, GRPC 等
* 服务治理特定领域的类库和解决方案: Hystrix, Zookeeper, Zipkin, Sentinel 等
* 对多种方案进行整合的微服务框架: SpringCloud、Finagle、Dubbox 等

service mesh 提供了一种 **Sidecar 模式**

这其实不是什么新东西，以前叫做agent, 在两个节点之前加一个代理。 微服务的大部分需要解决的问题，在这sidecar 实现。提供服务注册发现、服务RPC通信、服务配置管理、服务负载均衡、路由限流、容错、服务监控及治理、服务发布及升级等通用能力

Linkerd的CEO Willian Morgan给出的Service Mesh的定义:

> A Service Mesh is a dedicated infrastructure layer for handling service-to-service communication. It’s responsible for the reliable > delivery of requests through the complex topology of services that comprise a modern, cloud native application. In practice, the
> Service Mesh is typically implemented as an array of lightweight network proxies that are deployed alongside application code,
> without the application needing to be aware.

关键字：基础设施层　轻量级网络代理　对应用程序透明

第二代 Service Mesh　在数据平面的基础上添加了控制平面。

国内Service Mesh 发展情况：

* 蚂蚁金服开源SOFAMesh：
    * https://github.com/alipay/sofa-mesh
    * 从istio fork
    * 使用Golang语言开发全新的Sidecar，替代Envoy
    * 为了避免Mixer带来的性能瓶颈，合并Mixer部分功能进入Sidecar
    * Pilot和Citadel模块进行了大幅的扩展和增强
    * 扩展RPC协议: SOFARPC/HSF/Dubbo

* 华为:
    * go-chassis: https://github.com/go-chassis/go-chassis golang 微服务框架, 支持istio平台
    * mesher: https://github.com/go-mesh/mesher mesh 数据面解决方案
    * 国内首家提供Service Mesh公共服务的云厂商
目前(2019年1月)公有云Istio 产品线上已经支持申请公测, 产品形态比较完善
腾讯云 TSF:
基于 Istio、envoy 进行改造
支持 Kubernetes、虚拟机以及裸金属的服务
对 Istio 的能力进行了扩展和增强, 对 Consul 的完整适配
对于其他二进制协议进行扩展支持
唯品会
OSP (Open Service Platform)
新浪:
Motan: 是一套基于java开发的RPC框架, Weibo Mesh 是基于Motan