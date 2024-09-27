---
layout: post
title: 【Cloud Native】01 云原生原理与演进
categories: [CloudNative, K8S学习圣经]
tags: [云原生, 服务网格]
---

## 0 文档

reference:

- [k8s学习圣经](https://www.cnblogs.com/crazymakercircle/p/17304541.html)

## 1 什么是云原生

### 1.1 历史

云原生之所以解释不清楚，是因为云原生没有确切的定义，云原生一直在发展变化之中，解释权不归某个人或组织所有。

#### 1.1.1 2013年

Pivotal公司的Matt Stine于2013年首次提出云原生（Cloud Native）的概念。

#### 1.1.2 2015年

2015年，云原生刚推广时，Matt Stine在《迁移到云原生架构》一书中定义了符合云原生架构的几个特征：

- 符合 12 因素应用、
- 面向微服务架构、
- 自敏捷架构、
- 基于API协作、
- 扛脆弱性；

> Pivotal 推出过 Pivotal Cloud Foundry 云原生应用平台和 Spring 开源 Java 开发框架，成为云原生应用架构中先驱者和探路者。
> Pivotal 是云原生应用平台第一股，2018 年在纽交所上市，2019 年底被 VMWare 以 27 亿美元收购，加入到 VMware 新的产品线 Tanzu。

到了 2015 年 Google 主导成立了云原生计算基金会（CNCF），开始围绕云原生的概念打造云原生生态体系，起初CNCF对云原生的定义包含以下三个方面：

- 应用容器化(software stack to be Containerized)
- 面向微服务架构(Microservices oriented)
- 应用支持容器的编排调度(Dynamically Orchestrated)

#### 1.1.3 2017年

2017年, 云原生应用的提出者之一的Pivotal在其官网上将云原生的定义概况为DevOps、持续交付、微服务、容器这四大特征，这也成了很多人对 Cloud Native的基础印象。

![01-cloud-native-4-characters](/assets/images/Cloud-Native/01-basic/01-cloud-native-4-characters.jpeg)

## 2 云原生发展历史时间轴

### 2.1 从微服务到服务网格

|标致| 描述|
|:---|:---|
|微服务|马丁大师在2014年定义了微服务|
|Kubernetes|- 从2014年6月由Google宣布开源<br/>- 到2015年7月发布1.0这个正式版本并进入CNCF基金会<br/>- 再到2018年3月从CNCF基金会正式毕业，迅速成为容器编排领域的标准<br/>- 是开源历史上发展最快的项目之一|
|Linkerd|- Scala语言编写，运行在JVM中，Service Mesh名词的创造者<br/>- 2016年01月15号，0.0.7发布<br/>- 2017年01月23号，加入CNCF组织<br/>- 2017年04月25号，1.0版本发布|
|Envoy|- envoy 是一个开源的服务代理，为云原生设计的程序，由C++语言编程<br/>- 2016年09月13号，1.0发布<br/>- 2017年09月14号，加入CNCF组织|
|Istio|- Google、IBM、Lyft发布0.1版本<br/>- Istio是开源的微服务管理、保护和监控框架。Istio为希腊语，意思是”起航“。|

### 2.2 微服务架构的问题

#### 2.2.1 问题

1. 网络通信问题: 最初是为了业务而写代码，比如登录功能、支付功能等，到后面会发现要解决网络通信的问题。
   - SpringCloud "侵入式框架": 在业务代码里面加上spring cloud maven依赖，加上spring cloud组件注解，写配置，打成jar的时候还必须要把非业务的代码也要融合在一起，称为“侵入式框架”；
2. 跨语言问题: 微服务中的服务支持不同语言开发，也需要维护不同语言和非业务代码的成本；
3. 升级问题：因为Spring Cloud是“代码侵入式的框架”，这时候版本的升级就注定要让非业务代码一起，一旦出现问题，再加上多语言之间的调用，工程师会非常痛苦；

其实我们到目前为止应该感觉到了，服务拆分的越细，只是感觉上轻量级解耦了，但是维护成本却越高了，那么怎么办呢？

#### 2.2.2 问题解决思路

本质上是要解决服务之间通信的问题，不应该将非业务的代码融合到业务代码中，也就是从客户端发出的请求，要能够顺利到达对应的服务，这中间的网络通信的过程要和业务代码尽量无关。

服务通信无非就是服务发现、负载均衡、版本控制等等。

在很早之前的单体架构中，其实通信问题也是需要写在业务代码中的，那时候怎么解决的呢？

![02-communicate-01](/assets/images/Cloud-Native/01-basic/02-communicate-01.png)

把网络通信，流量转发等问题放到了计算机网络模型中的TCP/UDP层，也就是非业务功能代码下沉，把这些网络的问题下沉到计算机网络模型当中，也就是网络七层模型

![02-communicate-02](/assets/images/Cloud-Native/01-basic/02-communicate-02.png)

> 网络七层模型：应用层、表示层、会话层、传输层、网络层、数据链路层、物理层

所以，我们是否也可以把每个服务配置一个代理，所有通信的问题都交给这个代理去做？

就好比大家熟悉的nginx,haproxy其实它们做反向代理把请求转发给其他服务器,也就为 Service Mesh的诞生和功能实现提供了一个解决思路。

#### 2.2.3 SideCar 旁车模式（边车模式）

Sidecar模式是一种将应用功能从应用本身剥离出来作为单独进程的方式。

SideCar降低了与微服务架构相关的复杂性，并且提供了负载平衡、服务发现、流量管理、电路中断、遥测、故障注入等基础特性。

服务业务代码与Sidecar绑定在一起，每个服务都配置了一个Sidecar代理，每个服务所有的流量都经过sidecar, sidecar帮我们屏蔽了通信的细节，我的业务开发人员只需要关注业务就行了，而通信的事情交给sidecar处理

sidecar是为了通用基础设施而设计，可以做到与公司框架技术无侵入性。

该模式允许我们向应用无侵入添加多种功能，避免了为满足第三方组件需求而向应用添加额外的配置代码。


很多公司借鉴了Proxy模式，推出了Sidecar的产品， 比如像：Netflix的Prana，蚂蚁金服的SofaMesh。

2014年 Netflix发布的Prana

2015年 唯品会发布local proxy

2016年 Twitter的基础设施工程师发布了第一款Service Mesh项目：Linkerd (所以下面介绍Linkerd)

### 2.3 互联网服务架构历史

#### 2.3.1 单体服务时代

2010年前，大多数服务业务单一且简单，采用典型的单机+数据库模式，所有的功能都写在一个应用里并进行集中部署。

![05-单体服务01.png](/assets/images/Cloud-Native/01-basic/05-单体服务01.png)

论坛业务、聊天室业务、邮箱业务全部都耦合在一台小型机上面，所有的业务数据也都存储在一台数据库上。

随着应用的日益复杂与多样化，开发者对系统的容灾，伸缩以及业务响应能力有了更高的要求：

- 如果小型机和数据库中任何一个出现故障，整个系统都会崩溃，
- 若某个板块的功能需要更新，那么整个系统都需要重新发布，

如何保障可用性的同时快速响应业务的变化，需要将系统进行拆分，将上面的应用拆分出多个子应用。

![05-单体服务02.png](/assets/images/Cloud-Native/01-basic/05-单体服务02.png)

应用垂直拆分解决了应用发布的问题，但是随着用户数量的增加，单机的计算能力依旧是杯水车薪

用户量越来越大，同时也要进行高可用保证， 大部分系统，需要进行横向扩展（水平扩展）

在接入层，引入负载均衡组件，有了负载均衡之后，架构图如下

![05-单体服务03.png](/assets/images/Cloud-Native/01-basic/05-单体服务03.png)

阿里巴巴在2008提出去“IOE”，也就是IBM小型机、Oracle数据库，EMC存储，全部改成集群化负载均衡架构，在2013年支付宝最后一台IBM小型机下线.

优点：应用跟应用解耦，系统容错提高了，也解决了独立应用发布的问题，同时可以水平扩展来提供应用的并发量

> more doc:
> - [小型机](https://zh.wikipedia.org/wiki/%E5%B0%8F%E5%9E%8B%E8%AE%A1%E7%AE%97%E6%9C%BA)
> - [小型机历史](https://developer.aliyun.com/article/769926)

#### 2.3.2 微服务时代

分布式微服务时代

微服务是在2012年提出的概念，微服务的希望的重点是一个服务只负责一个独立的功能。

拆分原则，任何一个需求不会因为发布或者维护而影响到不相关的服务，一切可以做到独立部署运维。

- 比如传统的“用户中心”服务，对于微服务来说，需要根据业务再次拆分，可能需要拆分成“买家服务”、“卖家服务”、“商家服务”等。

典型代表：

- Spring Cloud，相对于传统分布式架构，SpringCloud使用的是HTTP作为RPC远程调用，配合上注册中心Eureka和API网关Zuul，可以做到细分内部服务的同时又可以对外暴露统一的接口，让外部对系统内部架构无感，此外Spring Cloud的config组件还可以把配置统一管理。

> - [马丁大师对微服务的定义](https://martinfowler.com/articles/microservices.html)
> - [spring cloud地址](https://spring.io/projects/spring-cloud)

微服务真正定义的时间是在2014年

> - The term "Microservice Architecture" has sprung up over the last few years to describe a particular way of designing software applications as suites of independently deployable services. While there is no precise definition of this architectural style, there are certain common characteristics around organization around business capability, automated deployment, intelligence in the endpoints, and decentralized control of languages and data.
> - 大概意思：可独立部署服务，服务会越来越细

集群部署多了，这些重复的功能无疑会造成资源浪费，所以会把重复功能抽取出来，名字叫"XX服务（Service）"

![06-micro-services](/assets/images/Cloud-Native/01-basic/06-micro-services.png)

为了解决服务跟服务如何相互调用，需要一个程序之间的通信协议，所以就有了远程过程调用（RPC），作用就是让服务之间的程序调用变得像本地调用一样的简单

#### 2.3.3 Service Mesh 服务网格

服务网格和微服务的本质区别，主要在于 单体服务进行 业务服务和 非业务基础 服务进行解耦

非业务基础 服务，交给基础框架进行 管理和控制

非业务基础 （sidecar）服务 + 服务治理 中间件 （Pilot/mixor），组成一个网络结构， 被称之为服务网格

如下图所示

![07-service-mesh](/assets/images/Cloud-Native/01-basic/07-service-mesh.png)

为啥叫做服务网格，主要 从架构层面看起来跟网格很像，

如果每一个格子都是一个sidecar数据单位（单元格），然后sidecar进行彼此通信，所以这些单元格 组成 **数据面**

这些 sidecar数据单位（单元格），由统一的控制/配置中间组件（类似nacos注册中心）进行配置和控制， 那些控制组件 组成 了 **控制面**

宏观上 ，`服务网格 =数据面 + 控制面`

特点：

- **基础设施**：服务网格是一种处理服务之间通信的基础设施层。
- **支撑云原生**：服务网格尤其适用于在云原生场景下帮助应用程序在复杂的服务间可靠地传递请求。
- **网络代理**：在实际使用中，服务网格一般是通过一组轻量级网络代理来执行治理逻辑的。
- **对应用透明**：轻量网络代理与应用程序部署在一起，但应用感知不到代理的存在，还是使用原来的方式工作。


## 3 2018 年云原生被CNCF重新定义

到了 2018 年，随着近几年来云原生生态的不断壮大，所有主流云计算供应商都加入了该基金会，

CNCF 基金会中的会员以及容纳的项目越来越多，该定义已经限制了云原生生态的发展，CNCF 为云原生进行了重新定位。

2018年6月，CNCF正式对外公布了更新之后的云原生的定义（包含中文版本）v1.0版本，当前版本为[v1.1版本](https://github.com/cncf/toc/blob/main/DEFINITION.md)。

云原生定义（v1.1版本）：

- 云原生技术有利于各组织在公有云、私有云和混合云等新型动态环境中，构建和运行可弹性扩展的应用。
- **云原生的代表技术包括容器、服务网格、微服务、[不可变基础设施](https://glossary.cncf.io/immutable-infrastructure/)和[声明式API](https://kubernetes.io/zh-cn/docs/concepts/extend-kubernetes/api-extension/custom-resources/#declarative-apis)**。
- 这些技术能够构建容错性好、易于管理和便于观察的松耦合系统。结合可靠的自动化手段，云原生技术使工程师能够轻松地对系统作出频繁和可预测的重大变更。
- 云原生计算基金会（CNCF）致力于培育和维护一个厂商中立的开源生态系统，来推广云原生技术。
- 我们通过将最前沿的模式民主化，让这些创新为大众所用。

五个代表技术

![04-cloud-native-defination-latest](/assets/images/Cloud-Native/01-basic/04-cloud-native-defination-latest.png)

服务网格

![04-cloud-native-defination-latest](/assets/images/Cloud-Native/01-basic/04-service-mesh.png)

不可变基础设施

![04-cloud-native-defination-latest](/assets/images/Cloud-Native/01-basic/04-immutable-infrastructure.png)

声明式API

![04-cloud-native-defination-latest](/assets/images/Cloud-Native/01-basic/04-declarative-api.png)

## 4 关于 CNCF

CNCF基金会是 2015年由Linux基金会发起了一个 The Cloud Native Computing Foundation（CNCF）基金组织，CNCF基金会的成立标志着云原生正式进入高速发展轨道，Google、Cisco、Docker各大厂纷纷加入，并逐步构建出围绕 Cloud Native 的具体工具，而云原生这个的概念也逐渐变得更具体化。

CNCF最初对云原生定义是也是狭窄的，当时把云原生定位为容器化封装+自动化管理+面向微服务。

CNCF的目的不一样，他成立的目的是希望打破云巨头的垄断，实际上是希望通过容器和k8s，将提供底层资源的云服务商变得无差异化。

这主要因为CNCF基金会在当时的核心拳头软件就是k8s，因此在概念定义上主要是围绕着容器编排建立起来的生态。这也是为什么我们感觉CNCF 定义云原生的时候就是在说容器生态。

CNCF 是非营利性 Linux 基金会的一部分。

[CNCF官网](https://www.cncf.io/)介绍：

CNCF is the open source, vendor-neutral hub of cloud native computing, hosting projects like Kubernetes and Prometheus to make cloud native universal and sustainable.

> CNCF是开源的、厂商中立的云原生计算中心，托管Kubernetes和Prometheus等项目，使云原生具有普遍性和可持续性。

从 2015 年 Google 牵头成立 CNCF 以来，云原生技术开始进入公众的视线并取得快速的发展，到 2018 年包括 Google、AWS、Azure、Alibaba Cloud 等大型云计算供应商都加入了云原生基金会 CNCF，云原生技术也从原来的应用容器化发展出包括容器、Service Mesh、微服务、不可变基础设施、Serverless、FaaS 等众多技术方向。

### 4.1 CNCF 优秀项目

[CNCF项目全景图](https://landscape.cncf.io/)

|优秀项目| 描述|
|:----|:----|
|Kubernetes|- Kubernetes 是世界上最受欢迎的**容器编排平台**也是第一个 CNCF 项目。<br/>- Kubernetes 帮助用户构建、扩展和管理应用程序及其动态生命周期。|
|Prometheus|Prometheus 为云原生应用程序提供**实时监控、警报**包括强大的查询和可视化能力，并与许多流行的开源数据导入、导出工具集成。|
|Containerd|- Containerd 是由 Docker 开发并基于 Docker Engine 运行时的行业标准**容器运行时**组件。<br/>- 作为容器生态系统的选择，Containerd 通过提供运行时，可以将 Docker 和 OCI 容器镜像作为新平台或产品的一部分进行管理。|
|Envoy|- Envoy 是最初在 Lyft 创建的 **Service Mesh**（服务网格），现在用于Google、Apple、Netflix等公司内部。 <br/>- Envoy 是用 C++ 编写的，旨在最大限度地减少内存和 CPU 占用空间，同时提供诸如负载均衡、网络深度可观察性、微服务环境中的跟踪和数据库活动等功能。|
|Fluentd|- Fluentd 是一个统一的**日志记录工具**，可收集来自任何数据源（包括数据库、应用程序服务器、最终用户设备）的数据，并与众多警报、分析和存储工具配合使用。<br/>- Fluentd 通过提供一个统一的层来帮助用户更好地了解他们的环境中发生的事情，以便收集、过滤日志数据并将其路由到许多流行的源和目的地。|
|GRPC|gRPC 是一个高性能、开源和通用的 **RPC 框架**,语言中立，支持多种语言。|
|CNI|- CNI 就是这样一个标准，它旨在为容器平台提供**网络的标准**化。<br/>- 不同的容器平台能够通过相同的接口调用不同的网络组件。|
|Helm|- Helm 是 **Kubernetes 的包管理器**。<br/>- 包管理器类似于我们在Centos中使用的yum一样，能快速查找、下载和安装软件包。|
|Etcd|- 一个高可用的分布式键值**(key-value)数据库**。<br/>- etcd内部采用raft协议作为一致性算法，etcd基于Go语言实现。一般用的最多的就是作为一个注册中心来使用|
|Open Tracing|OpenTracing：为不同的平台，供应中立的API，使开发人员可以轻松地应用**分布式跟踪**。|
|Jaeger|Jaeger 是由 Uber 开发的**分布式追踪系统，**用于监控其大型微服务环境。<br/>- Jaeger 被设计为具有高度可扩展性和可用性，它具有现代 UI，旨在与云原生系统（如 OpenTracing、Kubernetes 和 Prometheus）集成。|

## 5 服务网格（service mesh）原理和价值

### 5.1 什么是服务网格（service mesh）

istio官网 也对什么是service mesh给出了[定义](https://istio.io/latest/about/service-mesh/)

> - A service mesh is an infrastructure layer that gives applications capabilities like zero-trust security, observability, and advanced traffic management, without code changes.
> - 服务网格是一个基础设施层，它为应用程序提供诸如零信任安全、可观察性和高级流量管理等功能，而无需更改代码。

> - Istio ensures that cloud native and distributed systems are resilient, helping modern enterprises maintain their workloads across diverse platforms while staying connected and protected. It enables security and governance controls including mTLS encryption, policy management and access control, powers network features like canary deployments, A/B testing, load balancing, failure recovery, and adds observability of traffic across your estate.
> - Istio确保云原生和分布式系统具有弹性，帮助现代企业在保持连接和保护的同时，跨不同平台维护其工作负载。它支持安全和治理控制，包括mTLS加密、策略管理和访问控制，支持金丝雀部署、A/B测试、负载平衡、故障恢复等网络功能，并增加整个资产的流量可观察性。

总结来看：服务网格：指的是微服务网络应用之间的交互，随着规模和复杂性增加，服务跟服务调用错综复杂

如下图所示

![07-service-mesh](/assets/images/Cloud-Native/01-basic/07-service-mesh.png)

如果每一个格子都是一个sidecar数据单元，然后sidecar进行彼此通信，从架构层面看起来跟网格很像，整体组成一个服务网格 service mech

服务网格将 “业务服务”和“基础设施”解耦，将一个微服务进程一分为二：

![07-service-mesh-detail](/assets/images/Cloud-Native/01-basic/07-service-mesh-detail.png)

### 5.2 Service Mesh的价值

服务网格将 “业务服务”和“基础设施”解耦，

Service Mesh上的sidecar支撑了所有的上层应用，业务开发者无须关心底层构成，可以用Java，也可以用Go等语言完成自己的业务开发。

Istio的理论概念是Service Mesh（服务网络），我们不必纠结于概念，实际也是微服务的一种落地形式。有点类似上面的SideCar模式。

Service Mesh 的主要思想是职责解耦、责任清晰，即不像SpringCloud一样将服务治理交给研发来做，也不集成到k8s中产生职责混乱。

Istio是通过为服务配 Agent代理来提供服务发现、负截均衡、限流、链路跟踪、鉴权等微服务治理手段。

Istio开始就是与k8s结合设计的，Istio结合k8s可以牛逼的落地微服务架构。

istio 超越 spring cloud等传统开发框架之处, 就在于不仅仅带来了远超这些框架所能提供的功能, 而且也不需要应用程序为此做大量的改动，开发人员也不必为上面的功能实现进行大量的知识储备。

> 总结：很明显Istio不仅拥有“数据平面（Data Plane）”，而且还拥有“控制平面（Control Plane），也就是拥有了数据 接管与集中控制能力。

### 5.3 其他服务网格

- Linkerd

Linkerd在面世之后，迅速获得用户的关注，并在多个用户的生产环境上成功部署、运行。2017年，Linkerd加入CNCF，随后宣布完成对千亿次生产环境请求的处理，紧接着发布了1.0版本，并且具备一定数量的商业用户，一时间风光无限，一直持续到Istio横空出世。

在早期的时候又要部署服务，又要部署sidecar，对于运维人员来说比较困难的，

所以没有得到很好的发展，其实主要的 问题是Linkerd只是实现了数据层面的问题，但没有对其进行很好的管理。

或者说， Linkerd 没有实现一套好的 **控制面** 组件。

- 蚂蚁金服 sofa Mesh
- 腾讯 Tencent Service Mesh
- 华为 CSE Mesher

> 总结：基本上都借鉴了Sidecar、Envoy和Istio等设计思想

## 6 云原生应用和传统应用的区别

云原生是一个很宽泛的概念，想要开发一个支持云原生的应用并不难，

可能就是简单的实现可基于容器部署、使用Kubernetes进行编排与调度，集成CI/CD工具以及Prometheus监控工具等。


|维度|传统应用|云原生应用|
|-|-|-|
|应用架构|单体式/紧密耦合|微服务/松耦合|
|应用构成|业务代码+中间件+数据库+运维功能|业务组件+平台|
|交付周期|长|短且持续|
|团队协作|部门墙导致团队彼此孤立|团队借助DevOps更容易达成协作|
|基础设施|以服务器为中心，不可移植|以容器平台为中心，可移植|
|需求开发模式|瀑布式开发|DevOps、持续集成、敏捷开发|
|开发语言|C/C++、企业级Java编写|Go、Node.js等新兴语言编写|
|服务耦合|单体服务耦合严重|微服务各自独立，高内聚，低耦合|
|依赖关系|单体服务耦合严重|微服务化，高内聚，低合|
|扩展性|运维手工扩容|动态扩缩容|
|可用性|故障后可能影响业务|无状态，面向失败编程|
|可靠性|本地资源，故障率高|云资源，可靠性高|
|可预测性|不可预测。通常构建时间更长，<br/>若大批量发布，只能逐渐扩展，<br/>并且会发生更多的单点故障|可预测。云原生应用符合旨在通过可预测行为，<br/>最大限度提高弹性的框架或“合同'|
|运维方式|人肉部署、手工运维|自动化部署，支撑频繁变更，持续交付，蓝绿部署|
|运维模式|定期巡检+告警通知+人工操作|策略声明+失败自动恢复/回滚|
|运维能力|手动运维|自动化运维能力|
|操作系统|依赖操作系统|操作系统抽象化|
|部署方式|对机器、网络资源强绑定|容器化，对网络和存储都没有这种限制|
|资源调度|资源冗余较多，缺乏扩展能力|资源调度有弹性|
|扩容模式|提前采购几余资源+手动扩展|自动编排，根据指标和策略来弹性伸缩|
|恢复能力|恢复缓慢|快速恢复|

云原生在一个更好的基础平台与设施上提供了更多的应用。

因为做了容器化就不需要指定操作系统，K8S 的资源调度更有弹性，之前需要通过代码来协调实现伸缩策略，比较麻烦，借助DevOps 会容易达成协作，因为它整个流程都是自动的，能够敏捷开发。

还有微服务是都是各自独立的，具有高内聚、低耦合的原则，具有自动化运维、快速恢复的特点，自愈能力强。当集群宕掉了，它会自动拉起。

> 总结： 云原生与传统应用有比较明显的区别，云原生更倡导敏捷、自动化、容错，而传统应用则大多还处于原生的瀑布开发模型和人工运维阶段。

## 7 云原生涉及的核心项目

![08-CloudNative-Core-Projects](/assets/images/Cloud-Native/01-basic/08-CloudNative-Core-Projects.png)


