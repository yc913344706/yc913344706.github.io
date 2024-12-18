---
layout: post
title: 【Cloud Native】07 K8S 资源控制
categories: [CloudNative, K8S]
tags: []
---

## 动态扩缩容的重要性

- 在微服务应用中，在高并发场景下，需要对 springboot、SpringCloud、nginx 应用进行动态的扩容缩容
  - 当微服务的性能不足以支撑庞大的访问量，可以动态扩容
  - 而当访问量减少时，过多的微服务实例会白白占用服务器资源，造成资源浪费时，可以动态缩容

- 所以动态的扩容缩容，是高并发应用的标配。
- 目前业界主流的方案，是基于K8S实现 动态的扩容缩容。

为了更好地解决Pod编排的问题：
- k8s在V1.2版本开始，引入了deployment资源对象

## ReplicaController, ReplicaSet

### ReplicaController

- Replication controller 简称 RC，RC是Kubernetes 系统中的核心概念之一，
- 简单来说，RC可以保证在任意时间运行 Pod 的副本数量，能够保证 Pod 总是可用的。
- 如果实际 Pod 数量比指定的多，那就干掉多余的，如果实际数量比指定的少就新启动一些 Pod，当 Pod 失败、被删除或者挂掉后，RC 都会去自动创建新的 Pod 来保证副本数量，
- 所以生产场景中，哪怕只有一个pod，也应该使用 RC 来管理我们的 Pod。
- 就是由于RC的副本控制机制，哪怕运行 Pod 的节点挂了，RC 检测到 Pod 失败了，就会去合适的节点重新启动一个 Pod 就行，不需要我们手动去新建一个 Pod 了。

### ReplicaSet

- Replication Set 简称 RS，随着Kubernetes 的高速发展，官方已经推荐我们使用 RS 和Deployment来代替RC了，
- 实际上 RS和RC 的功能基本一致，目前唯一的一个区别就是 RC 只支持基于等式的 selector (env=dev或 environment!=qa)，
- 但 RS 还支持基于集合的 selector (version in(v1.0,v2.0)) ，这对复杂的运维管理就非常方便了。

- kubectl 命令行工具中关于 RC 的大部分命令同样适用于我们的 RS 资源对象
- 不过我们也很少会去单独使用 RS，它主要被 Deployment 这个更加高层的资源对象使用，
- 除非用户需要自定义升级功能或根本不需要升级 Pod，在一般情况下，我们推荐使用 Deplovment而不直接使用Replica Set.

### 总结

- 随着Kubernetes 的高速发展，官方已经推荐我们使用 RS 和Deployment来代替RC了，
- 不过我们也很少会去单独使用 RS，它主要被 Deployment 这个更加高层的资源对象使用

## deployment

注意:
- deployment资源并不直接管理pod，而是通过管理replicaset来间接管理pod
- 简单的说: deployment管理replicaset，而 replicaset管理pod。

deployment的主要功能有下面几个:
- 支持replicaset的所有功能
- 支持发布的停止、继续
- 支持版本的滚动更新和版本回退
- 扩容和缩容

> 所以: 
> - 我们部署一个应用一般不直接写Pod，而是部署一个Deployment
> - 编写一个Deployment的yaml赋予Pod自愈和故障转移能力

Deployment 使用场景: 
- Deployment主要针对无状态服务，有状态服务使用 StatefulSet. 

[扩缩容命令见这里]({% link _posts/CloudNative/2024-9-19-k8s-cmds.md %})

### deployment 滚动更新

- 仅当 Deployment Pod 模板(即.spectemplate ) 发生改变时，触发 Deployment 滚动更新。
  - 例如：模板的标签或容器镜像被更新
- 其他更新不会触发滚动更新动作。
  - 例如：对 Deployment 执行扩缩容的操作

滚动更新原理: 
- 创建新的rs，准备就绪后，替换旧的rs
- 旧版本此时不会删除，因为 revisionHistoryLimit 指定了保留几个版本

滚动机制相关的命令
- 新创建: 当Deployment controller 观测到有新的 deployment被创建时，会直接创建出一个新的 ReplicaSet 来做这件事。
- 变更: 
  - 当更新了一个的已存在并正在进行中的 Deployment，
  - 每次更新 Deployment都会创建个新的 ReplicaSet并扩容它，同时回滚之前扩容的 ReplicaSet，将它添加到旧的 ReplicaSet 列表中，开始缩容。
- 回滚
- 暂停
- 恢复

[命令参考见这里：rollout]({% link _posts/CloudNative/2024-9-19-k8s-cmds.md %})

### deployment 进行灰度发布

MARK: wait to complete.

## DaemonSet

- DaemonSet用来确保每个node节点 (或者指定部分节点)运行一个Pod副本
- 这个副本随着node节点的加入而自动创建;若node节点被移除，Pod也将会被移除。
- 注意: 
  - 默认master除外，master节点默认不会把Pod调度过去; 
  - DaemonSet无需指定副本数量;因为默认给每个机器都部署一个

应用场景:
- 在每个节点上运行集群的存储守护进程，例如 glusterd、ceph
- 在每个 Node 上运行日志收集守护进程，例如fluentd、logstash
- 在每个 Node 上运行监控守护进程，例如 Prometheus Node

## pod调度

### nodeSelector

- label标签是 kubernetes 中一个非常重要的概念，用户可以非常灵活的利用 label来管理集群中的资源
- 比如最常见的 Service 对象通过label 去匹配 Pod 资源，而Pod的调度也可以根据节点的label来进行调度。

- 我们先给节点node打上标签
  - `kubect1 1abel nodes node-01 important=very`
- 可以用 --show-labels 参数可以查看上述标签是否生效
  - `kubect1 get nodes --show-1abe1s`
- 当节点被打上了相关标签后，在调度的时候就可以使用这些标签了，只需要在 Pod 的 spec 字段中添加nodeSelector 字段，里面是我们需要被调度的节点的label 标签，
- 比如，下面的 Pod 我们要强制调度到 node-01 这个节点上去，我们就可以使用 nodeSelector 来表示了指定调度到 important=very 的节点
```yaml
spec:
  nodeSelector:
    important=very
```

### nodeAffinity

nodeAffinity 根据软策略和硬策略分为2种：
- 硬策略（requiredDuringSchedulingIgnoredDuringExecution）
  - 表示POD 必须部署到满足条件的节点上，如果没有满足条件的节点，就不停重试。
  - 其中 IgnoredDuringExecution表示 PDO 部署完成之后，如果节点标签发生了变化，不再满足POD指定的条件，POD也会继续运行
- 软策略（preferredDuringSchedulingIgnoredDuringExecution）
  - 表示 POD 优先部署到满足条件的节点上，如果没有满足条件的节点，就忽略这些条件，按照正常逻辑部署

文档：
- [k8s入门系列---- Node Affinity](https://www.cnblogs.com/fenggq/p/15065496.html)

### podAffinity

- pod affinity是用来定义pod与pod间的亲和性
- 所谓pod与pod的亲和性是指，pod更愿意和那个或那些pod在一起；
- 与之相反的也有pod更不愿意和那个或那些pod在一起，这种我们叫做pod anti afinity，即pod与pod间的反亲和性;

- 所谓在一起是指和对应pod在同一个位置，
  - 这个位置可以是按主机名划分，也可以按照区域划分，这样来我们要定义pod和pod在一起或不在一起，定义位置就显得尤为重要，也是评判对应pod能够运行在哪里标准:
  - 这里指定“同一位置” 是通过 topologyKey 来定义的，
  - topologyKey 对应的值是 node 上的一个标签名称，
    - 如果使用 k8s.io/hostname，则表示拓扑域为 Node 范围，那么 k8s.io/hostname 对应的值不一样就是不同的拓扑域。
    - 如果使用 failure-domain.k8s.io/zone ，则表示拓扑域为一个区域。
    - 当然，topologyKey 也可以使用自定义标签。

podAffinity 提供两种条件选择方式
- 调度到某一条件的节点 podAffinity
- 不调度到某一条件的节点 podAntiAffinity


文档：
- [k8s之pod亲和性与反亲和性的topologyKey](https://blog.csdn.net/yujia_666/article/details/116532953)
- [k8s affinity topology key 理解](https://www.jianshu.com/p/d906e819245c)

### Toleration

理解：
- DaemonSet 还会给这个 Pod 自动加上另外一个与调度相关的字段，叫作 tolerations。
- 这个字段意味着这个 Pod，会”容忍”(Toleration) 某些 Node 的“污点”(Taint)
- 在正常情况下，被标记了 unschedulable"污点”的 Node，是不会有任何 Pod 被调度上去的(effect: NoSchedule)。
- 可是，DaemonSet 自动地给被管理的 Pod 加上了这个特殊的Toleration，就使得这些 Pod 可以忽略这个限制，继而保证每个节点上都会被调度一个 Pod。
- 当然，如果这个节点有故障的话，这个 Pod 可能会启动失败，而 DaemonSet则会始终尝试下去，直到 Pod 启动成功。

作用：
- 通过这样一个 Toleration，调度器在调度这个 Pod 的时候，就会忽略当前节点上的”污点”，从而成功地将网络插件的 Agent 组件调度到这台机器上启动起来。
- 这种机制，正是我们在部署 Kubernetes 集群的时候，能够先部署 Kubernetes 本身、再部署网络插件的根本原因：
  - 因为当时我们所创建的 flannel 的 YAML，实际上就是一个 DaemonSet.

## StatefulSet

Deployment应用我们一般称为无状态应用 (stateless); StatefulSet 称为有状态副本集。

无状态应用与有状态应用区别：
- 无状态应用是指：
  - 不依赖于持久化存储或特定服务器状态的应用程序。
  - 每个请求都是独立的，无需维护会话状态或共享数据。
  - 无状态应用可以轻松扩展，通过增加更多的实例来处理更高的负载。
  - 例如：Nginx, Apache
- 有状态应用是指：
  - 依赖于持久化存储或特定服务器状态的应用程序。
  - 它们通常需要维护会话状态、缓存数据或与外部系统交互。
  - 管理有状态应用需要确保数据的一致性、持久性和可靠性。
  - 例如：Mysql, Redis

k8s处理无状态应用与有状态应用区别：
- 无状态应用：
  - 无论哪个实例处理请求，用户都会得到相同的响应。
  - 在Kubernetes中，我们可以使用Deployment对象来管理这个无状态Web应用。
  - Deployment可以确保在需求增加时自动扩展实例数量，并在实例故障时自动替换。
- 有状态应用：
  - 假设我们在一个在线银行系统中使用MySQL数据库来存储用户账户信息和交易记录。
  - 这个数据库服务是有状态的，因为它依赖于持久化的数据存储，并需要保证数据的一致性。
  - 在Kubernetes中，可以使用StatefulSet对象来管理有状态应用。
  - StatefulSet确保每个Pod都有一个固定的标识符和持久化存储，即使Pod重新启动或迁移也能保持数据不丢失。

StatefulSet为什么能处理有状态应用：
- StatefulSet就像是一种特殊的Deployment，它使用Kubernetes里的两个标准功能：Headless Service 和 PVC，实现了对的拓扑状态和存储状态的维护。
- 维护应用拓扑状态：
  - StatefulSet通过Headless Service， 为它管控的每个Pod创建了一个固定保持不变的DNS域名，来作为Pod在集群内的网络标识。
  - 加上为Pod进行编号并严格按照编号顺序进行Pod调度，这些机制保证了StatefulSet对维护应用拓扑状态的支持。
- 维护存储状态：
  - 而借由StatefulSet定义文件中的volumeClaimTemplates声明Pod使用的PVC，
  - 它创建出来的PVC会以名称编号这些约定与它创建出来的Pod进行绑定，
  - 借由PVC独立于Pod的生命周期和两者之间的绑定机制的帮助，StatefulSet完成了应用存储状态的维护。

StatefulSet 和 Deployment 的不同点：
- pod管理策略(podManagementPolicy)
  - OrderedReady: pod安装顺序创建和销毁。且每个pod的创建和销毁都必须要求前一个pod已经完全创建或销毁完毕。这个是默认的行为。
  - Parallel: 允许同时创建和销毁多个pod，不必等待前一个pod创建或销毁完毕。仅仅对扩容缩容有效，pod update过程不受影响。
- updateStrategy:更新策略
  - OnDelete: 如果pod发生更新StatefulSet不会更新已经运行的pod，必须将这些pod手工删除。新创建出来的pod是已更新过的pod.
  - RollingUpdate: 默认使用这个配置。如果发生pod更新，StatefulSet会挨个删除并重新创建所有的pod。操作的顺序为从pod n-1到 pod0。
  - partition: 发生更新的时候，所有序号大于等于partition的pod会更新，所有序号小于partition的pod不会更新。
    - 即便是把序号小于partition的pod删除了，pod还是会按照之前的版本创建，不会更新。
- 必须配置 headless service

文档：
- [云原生系列：有状态应用和无状态应用](https://www.51cto.com/article/789578.html)
- [深入理解StatefulSet，用Kubernetes编排有状态应用](https://zhuanlan.zhihu.com/p/333244616)

## Job, CronJob

Job:
- Kubernetes中的Job负责批处理任务，即仅执行一次的任务.
- Job 对象将创建一个或多个 Pod，并确保指定数量的 Pod 可以成功执行到进程正常结束.

CronJob:
- CronJob 是基于Job 来工作的:
- 在给定时间点，只运行一次周期性的定时运行(数据库备份、发送邮件)
- 一个CronJob 对象类似于 crontab (cron table) 文件中的一行记录

## HPA(Horizontal Pod Autoscaling)水平自动伸缩, VPA

HPA使Pod水平自动缩放,不再需要手动扩容

后面，会介绍更加高级的特合cAdvisor+ Prometheus (推荐)实现 QPS 吞吐量自动化伸缩，那属于高级内容。
在学习高级内容之前，咱们先掌握基础的: 基于metrics-server，实现CPU 利用率的简单HPA机制。

HPA 与 VPA:
- 水平扩缩意味着对增加的负载的响应是部署更多的 Pod。
-  这与“垂直（Vertical）”扩缩不同，对于 Kubernetes， 垂直扩缩意味着将更多资源（例如：内存或 CPU）分配给已经为工作负载运行的 Pod。
-  HPA 是通过扩展 Pod 的数量实现的
-  VPA 是通过增加单个 Pod 的可用资源实现的
- 通常 HPA 可用于水平扩展较容易的情况，例如 Serverless、FaaS、无状态微服务等
- 而 VPA 适用于水平扩展较复杂的情况，例如消息顺序处理、文件读写、数据库操作等。

HPA使用的对象：
- Horizontal Pod Autoscaling 仅适用于 Deployment和ReplicaSet, HPA由 API server和 controller 共同实现。
- Horizontal Pod Autoscaling,kubernetes 能够根据监测到的自动的扩容 replication controller，deployment 和 replica set。

HPA依赖：
- HPA 依赖到性能指标，收集指标信息插件是metrics-server
  - metrics-server是一个集群范围内的资源数据集和工具，同样的，metrics-server也只是显示数据，并不提供数据存储服务，
  - 主要关注的是资源度量API的实现，比如CPU、文件描述符、内存、请求延时等指标，
  - metric-server收集数据给k8s集群内使用，如kubectl,hpa,scheduler等
  - [metric-server安装指导]({% link _posts/CloudNative/2024-8-15-k8s-install-by-binary.md %})

- 创建HPA:
  - 这里有个小前提，要伸缩的pod必须进行资源限制: **限制监控指标**
  - 创建hpa，默认默认创建的HPA名称和需要自动伸缩的对象名一致，可以通过--name来指定HPA名
  - [创建、查看、删除命令参考这里]({% link _posts/CloudNative/2024-9-19-k8s-cmds.md %})

文档：
- [Pod 水平自动扩缩](https://kubernetes.io/zh-cn/docs/tasks/run-application/horizontal-pod-autoscale/)
- [Kubernetes：HPA 详解-基于 CPU、内存和自定义指标自动扩缩容](https://blog.csdn.net/fly910905/article/details/105375822)

## Garbage Collection（垃圾回收）

- 定义：
  - 一般来说，垃圾回收（GC）就是从系统中删除未使用的对象，并释放分配给它们的计算资源。
  - GC 存在于所有的高级编程语言中，较低级的编程语言通过系统库实现 GC。
- 在 K8s 中，有两大类 GC：
  - 级联（Cascading）：在级联删除中，所有者被删除，那集群中的从属对象也会被删除。
    - 在级联删除中，又有两种模式：前台（foreground）和后台（background）。
    - 前台：
      - 这种删除策略中，所有者对象的删除将会持续到其所有从属对象都被删除为止。
    - 后台：
      - 这种删除策略会简单很多，它会立即删除所有者的对象，并由垃圾回收器在后台删除其从属对象。
      - 这种方式比前台级联删除快的多，因为不用等待时间来删除从属对象。
  - 孤儿（Orphan）：这种情况下，对所有者的进行删除只会将其从集群中删除，并使所有对象处于“孤儿”状态。
    - 在孤儿删除策略（orphan deletion strategy）中，会直接删除所有者对象，并将从属对象中的 ownerReference 元数据设置为默认值。
    - 之后垃圾回收器会确定孤儿对象并将其删除
    - 如果对象的 OwnerReferences 元数据中没有任何所有者对象，那么垃圾回收器会删除该对象。
- [删除资源命令参考这里]({% link _posts/CloudNative/2024-9-19-k8s-cmds.md %})

文档：
- [Kubernetes 中的垃圾回收](https://zhuanlan.zhihu.com/p/160816303)



