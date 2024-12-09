---
layout: post
title: 【Cloud Native】09 service mesh
categories: [CloudNative, K8S]
tags: []
---

## 0 文档

reference:

- [k8s学习圣经](https://www.cnblogs.com/crazymakercircle/p/17304541.html)

## 1 概念

service mesh，服务网格。核心思想，就是对微服务的provider，进行架构解耦。也就是说，将业务服务和支持服务，进行解耦。

支持服务：流量代理、负载均衡、性能跟踪、日志、配置、链路跟踪、服务治理（限流、降级、熔断）。

主要分为两种模式：
- sidecar边车模式
- proxy代理模式

## 2 sidecar边车模式

sidecar，三轮摩托车的副座。sidercar服务，就是支持服务。

sidecar服务，与业务服务生命周期相同，一同创建和退出。
sidecar服务与业务服务绑定在一起部署，部署在业务程序旁边，这样通信没有明显io。

sidecar，也称sidekick。

优势：
- 非侵入式架构，服务治理等支持服务与业务服务解耦。

劣势：
- web服务，会增加请求次数
- 管理这些接口、依赖，会带来额外工作量

## 3 代理模式

sidecar拦截到业务服务的流量，以及业务服务的响应，进行转发。类似于Nginx，也就是一个反向代理的角色

sidercar还可以实现上面我们说的，性能监控、服务治理这些。

总的来说，sidecar是一种模式，一种角色，不是一个具体的组件。Istio是框架，Envoy是组件。

## 4 实现框架 Istio

Istio, Conduit, Envoy, Linkerd

|对比内容|Spring cloud|Istio|
|--|--|--|
| 配置中心、注册中心、服务发现、路由规则| nacos | Pilot(领航员) |
| 策略控制（如限流、熔断）| sentinel | Mixer |
| 支持组件 | | Envoy（使节） | 

![istio原理示意图](/assets/images/Cloud-Native/05-service-mesh/1-istio框架示意.png)


文档：
- [Service Mesh框架选型对比分析：Linkerd、Envoy、Istio、Conduit](http://www.mark-to-win.com/tutorial/50497.html)

### 4.1 Istio架构

Istio，基于现有的kubenete服务，proxy, kubelet，来获取网络信息（endpoint, cluster ip, pod ip）。
拥有istio-proxy的pod，所有的流量将被istio-proxy接管，但不会取代kube-proxy
而且istio-proxy会在处理每一个请求时，都将信息传给控制平面，这样，控制平面可以更好地观察、控制所有流量（精细化部署、流量治理）。
但是，拥有了istio-proxy，ingress就被取代了。

控制平面：Istiod
数据平面：Envoy


- [9 张图带你搞懂 Istio](https://cloud.tencent.com/developer/article/1757881)
- [Istio官方部署架构文档](https://istio.io/latest/docs/ops/deployment/architecture/)
- [Kiali——Istio Service Mesh 的可观察性工具](https://cloudnative.to/blog/kiali-the-istio-service-mesh-observability-tool/)

### 4.2 Istio运转流程

![istio框架示意图](/assets/images/Cloud-Native/05-service-mesh/1-istio框架示意2.png)

|步骤|说明|
|--|--|
|1, sidecar自动注入 | - 在Kubernetes场景下创建 Pod时，Kube-apiserver调用管理平面组件的 Sidecar-lnjector服务，<br/>- 然后会自动修改应用程序的描述信息yaml并注入Sidecar。<br/>- 业务服务无感知 |
|2, 流量拦截 | - 在 Pod 初始化时，设置iptables规则，将所有流量路由到 istio-proxy(envoy)上<br/>- 业务服务无感知 |
|3, 服务发现 | - envoy调用控制平面的pilot服务注册接口，注册服务<br/>- 这样，服务发现接口，也能发现这个服务 |
|4, 负载均衡 | - 控制平面Pilot可以对负载均衡进行配置，当然，有默认值。 |
|5, 流量治理 | - envoy路由时，会根据从控制平面（Pilot，Mixer）获取流量规则，进行流量分发。 |
|6, 访问安全 | - envoy通信时，会根据从控制平面（Pilot，Citadel）获取证书、密钥实现双向认证。 |
|7, 服务遥测 | - envoy通信时，所有请求都会上报控制平面（Mixer），这样控制平面可以将信息传给监控后端（如有）。 |
|8, 策略执行 | - envoy通信时，所有请求都会先检查（Mixer）的流量控制规则，这样可以实现（限流、熔断、降级、灰度发布（滚动、蓝绿、金丝雀）。 |
|9, 外部访问 | - Istio有Gateway，用来对接、控制外部访问。 |

### 4.3 Istio组件介绍

|组件|说明|
|--|--|
|Pilot| 服务注册中心、数据平面下发规则|
|Mixer（不是必须）| 分为：Policy（提供流量治理）、Telemetry（提供数据上报、日志搜集）|
|Citadel（不是必须）| 主要用来实现Istio的认证授权相关|
|Galley（不是必须）| 负责配置管理、配置复用等配置相关|
|Sidecar-Injector| 负责自动注入Sidecar|
|Proxy(Envoy)| envoy代理，是Istio框架中，唯一与数据平面流量交互的组件。<br/>envoy容器中有两个进程：<br/>- pilot-agent: 生成、启动、重载、监控envoy<br/>- envoy: 上面说的，流量代理、控制面交互等|
|Ingress-Gateway| 负责与网格外服务交互<br/>注：流量同样先经过Pilot|
