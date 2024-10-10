---
layout: post
title: 【Cloud Native】04 K8S 基础概念
categories: [CloudNative, K8S]
tags: []
---

## 0 文档

reference:

- [k8s学习圣经](https://www.cnblogs.com/crazymakercircle/p/17304541.html)

## 1 yaml理解

三个命令玩转所有的yamI写法
- kubectl explain pod.spec.xxx
- kubectl get xxx -oyaml
- kubectl create deploy xxx --dry-run-client -oyaml


### metadata

#### name

- Kubernetes RESTAPI中，可以通过 namespace + name 唯 性地确定一个RESTFUL对象
- 例如: /api/v1/namespaces/[namespace]/pods/[name]

#### UID:

- Kubernetes 中的 UID 是全局唯一的标识符
- UID 唯一确定一个 Kubernetes 对象, 这个对象不是 RESTFUL 对象，是Kubernetes 创建的资源对象
- Kubernetes 集群中，每创建一个对象，都有一个唯 的 UID.
- Kubernetes 所有的对象都是通过UID 唯一性确定
- UID 是由 Kubernetes 系统生成的，唯一标识某个 Kubernetes 对象的字符串。
- 用于区分多次创建的同名 RESTFUL 对象(如前所述，按照名字删除对象后，重新再创建同名对象时两次创建的对象 name 相同，但是 UID 不同。)

#### namespace

- Kubernetes通过名称空间 (namespace)在同一个物理集群上支持多个虚拟集群。
- Kubernetes 安装成功后，默认有初始化了三个名称空间:
  - default: 默认名称空间，如果 Kubernetes 对象中不定义metadata.namespace 字段,该对象将放在此名称空间下.
  - kube-system: Kubernetes系统创建的对象放在此名称空间下。
  - kube-public: 
    - 此名称空间自动在安装集群时自动创建，并且所有用户都是可以读取的(即使是那些未登录的用户)。
    - 主要是为集群预留的，例如，某些情况下，某些Kubernetes对象应该被所有集群用户看到。
- 命名规范：
  - 名称空间的名字必须与 DNS 兼容:
    - 不能带小数点
    - 不能带下划线
    - 使用数字、小写字母和减号 - 组成的字符串
- 作用：
  - 对象隔离：
    - 为不同团队的用户 (或项目)提供虚拟的集群空间，也可以用来区分开发环境/测试环境、准上线环境生产环境。
  - 为 名称 提供了作用域
    - 名称空间内部的同类型对象不能重名，但是跨名称空间可以有同名同类型对象。
    - 名称空间不可以嵌套。任何一个Kubernetes对象只能在一个名称空间中。
  - 团队隔离
    - 名称空间可以用来在不同的团队(用户)之间划分集群的资源，在Kubernetes 将来的版本中，同名称空间下的对象将默认使用相同的访问控制策略
    - 注意: 当Kubernetes对象之间的差异不大时，无需使用名称空间来区分，例如，同一个软件的不同版本，只需要使用 labels 来区分即可
    - 可能的使用情况：
      - 集群管理员通过一个Kubernetes集群支持多个用户组
      - 集群管理员将集群中某个名称空间的权限分配给用户组中的受信任的成员
      - 集群管理员可以限定某一个用户组可以消耗的资源数量，以避免其他用户组受到影响
      - 集群用户可以使用自己的Kubernetes对象，而不会与集群中的其他用户组相互干扰

- 如何访问不同名称空间的对象：
  - 原则是: **名称空间资源隔离，网络不隔离**
    - 不能访问其他 名称空间下的配置，如果把其他空间的配置直接拿来用，是不可以的。
    - 但是，可以访问其他空间的网络地址，网络地址没有屏蔽.

- 设置名称偏好
  - 可以通过set-context 命令改变当前 kubectl 上下文的名称空间，后续所有命令都默认在此名称空间下执行
    - kubectl config set-context --current --namespace=<您的名称空间>
  - 验证结果: 
    - kubectl config view --minify  grep namespace:

- 名称空间与DNS：
  - 当您创建一个 Service 时，Kubernetes 为其创建一个对应的 DNS 条目。
  - 该 DNS 记录的格式为 service.namespace.svc.cluster.ocal，也就是说，如果在容器中只使用，其DNS将解析到同名称空间下的 Service。
  - 这个特点在多环境的情况下非常有用，例如将开发环境、测试环境、生产环境部署在不同的名称空间下，应用程序只需要使用 即可进行服务发现，无需为不同的环境修改配置。
  - 如果您想跨名称空间访问服务，则必须使用完整的域名(fully qualified domain name，FQDN)

- 并非所有对象都在命名空间中
  - 大多数 kubernetes 资源 (例如 Pod、Service、副本控制器等)都位于某些命名空间中
    - 大部分的 Kubernetes 对象 (例如，Pod、Service、Deployment、StatefulSet等)都必须在名称空间里。
  - 但是命名空间资源本身并不在命名空间中。
  - 而且底层资源，例如 nodes 和持久化卷不属于任何命名空间。
    - 但是某些更低层级的对象，是不在任何名称空间中的，例如 nodes、persistentVolumes、storageClass 等
  - 查看哪些 Kubernetes 对象在名称空间里: `kubectl api-resources --namespaced=true`
  - 不在名称空间里: `kubectl api-resources --namespaced=false`


### spec （对象规约）

几乎每个 Kubernetes 对象包含两个嵌套的对象字段spec、status，它们负责管理对象的配置，两个字段的中文含义:

- 说明：
  - spec 描述你希望对象所具有的特征:期望状态(Desired State) 。
  - spec 是静态的，是存在于 YAML文件中的。
  - 当您在 Kubernetes 中创建一个对象时，您必须提供对象规约 (Spec)
  - 集群中所有的资源，都在那里? k8s只依赖一个存储就是etcd.

#### initContainers(pod yaml中定义)

[详情参考这里](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/init-containers/#init-containers-in-use)


### status （对象状态）

- 说明：
  - status 描述了对象在系统中的当前状态(Current State)，它是由 Kubernetes 系统和组件设置并更新的。
  - 在任何时刻，Kubernetes 控制平面都一直积极地管理着对象的实际状态，通过对应的控制器，不断地使实际状态趋向于您期望的目标状态
  - 如果这些实例中有的失败了(一种状态变更)，Kubernetes 系统通过执行修正操作 来响应规约和状态间的不一致 -- 在这里意味着它会启动一个新的实例来替换。
  - 在大多数情况下，用户不需要对此进行更改


## 2 服务流转

refer doc:
- 服务流转：https://blog.csdn.net/u010389826/article/details/122341564
- 四种service：https://zhuanlan.zhihu.com/p/157565821

### 2.1 Service

#### 2.1.1 服务简述

为什么会有service 之 pod 的 ip 漂移问题：
- k8s具有强大的副本控制能力，能保证任意副本（pod）挂掉时，自动从其他节点启动一个新的pod，还可以扩缩容等
- 那么，随着pod的销毁和创建，pod ip 肯定会动态变化
- 这个pod可能在任何时刻出现在任何节点上，也可能在任何时刻挂在任何节点上。

- service通过使用**标签选择器**来指定哪些pod属于同一组。
- 新的服务会分配一个集群内部IP，只能在集群内部访问，服务的主要目标是使集群内其他pod可以访问当前这种pod.
- 对内: 副本的选择、负载均衡。SVC通过 Label Selector 标签选择的方式，匹配一组Pod，对外访问服务。
- 对外: 流量的代理、转发。每一个 SVC可以理解成为一个微服务 gateway。

![10-service理解](/assets/images/Cloud-Native/02-k8s-macro-architecture/10-service理解.png)

service 能够提供负载均衡的能力，但是在使用上有以下限制:
- 只提供4层负载均衡能力 (只有 RR 轮询算法)，而没有7层功能（比如不支持cookie）
- 如果需要更多的转发规则来转发请求，4层上的负载均衡是不支持的.


#### 2.1.2 会话亲和性 sessionAffinity

- 说明：
  - 同一个客户端向一组pod的服务发起请求，服务每次将请求转发给不同的pod
  - 如果希望特定的客户端产生的所有请求都指向同一个pod，可以设置服务的sessionAfinity属性为ClientIP (而不是None，None是默认值)
  - 这种方式会将来自同一个clientIP的所有请求转发至同一个pod.
- k8s支持的会话亲和性：
  - k8s仅支持None和ClinetIP两种会话亲和性配置，不支持基于cookie的会话亲和性选项
  - 因为k8s的service不是在http层面工作，服务处理TCP和UDP包，cookie是HTTP协议的内容，所以service并不知道cookie。

#### 2.1.3 服务发现

客户端pod如何知道服务的IP和端口？有两种方式

- 通过环境变量：
  - 当创建一个新pod时，k8s会初始化一系列环境变量给容器
  - 其中包含当前存在的所有服务ip和端口，在任意一个pod容器中执行env命令，可以看到这些

- 通过DNS发现服务
  - k8s的kube-system命名空间下有一个自带的dns服务组件kube-dns，
  - 集群中的所有pod都使用它作为dns，在每个容器的/etc/resolv.conf中有配置，
  - 所有pod中的进程dns查询都会被这个dns服务器响应，这个服务器知道集群中运行的所有服务。
  - 客户端在知道服务名的前提下可以通过全限定域名（FQDN）来访问服务，格式为 `{servicename}.{namespace}.svc.cluster.local`
  - 例如：
    - 一个backend服务在default命名空间下，其域名为backend.default.svc,cluster.local，如果是在同一个命名空间下，可以省略命名空间及后面的部分

#### 2.1.4 就绪探针(readinessProbe)

我们已经知道pod的标签和服务选择器一样时，pod就会作为服务的后端

但是存在一种情况，pod刚启动时还没有加载好配置，或者需要执行预热防止第一个请求时间过长，这时不想让该pod接收请求。

方案是使用就绪探测器，k8s定期向pod执行探测，发现探测结果成功了才将服务转发过去。

就绪探针有三种类型

- Exec探针 ：在容器内执行进程，根据进程退出码决定
- HTTP GET探针： 向容器发送http get请求，根据返回码决定
- TCP socket探针 ：向容器指定端口打开TCP连接，如果连接成功，判定为就绪

#### 2.1.5 无头服务 (headless)

- 创建服务时默认会生成一个集群IP，客户端无需知道后面有几个pod
- 但是如果需要具体每一个pod 的IP呢，只需要指定clusterIp为None即可，即headless
- headless类型的区别就是不会为服务生成虚拟IP

- 常规的服务通过DNS服务器获得servicename.namespace.svc.cluster.local的是集群IP,
- 而headless服务获取该域名的实际IP是所有后台pod的IP列表。

> - **从客户端角度来看headless与常规服务并无不同**
> - 但是对于headless服务，由于DNS返回了pod的IP，客户端直接连接到pod，而不是通过服务代理。
> - **headless仍然提供跨pod负载均衡，但是是通过DNS轮询机制不是通过服务代理**。

- Headless Service还有一个重要用处（也是使用StatefulSet时需要Headless Service的真正原因），
- 它会为代理的每一个StatefulSet创建出来的Endpoint也就是Pod添加DNS域名解析，这样Pod之间就可以相互访问。

#### 2.1.6 Type: ClusterIP

- cluster 生成一个**虚拟IP**，为**一组pod**提供统一的接入点。当service存在时，它的IP地址和端口不会改变。
- 客户端通过service的ip和端口建立连接，由service将连接路由到该服务的任意一个pod上，
- 通过这种方式，客户端不需要知道每个pod的具体ip，pod可以随时移除或创建，同时实现pod间的负载均衡。

#### 2.1.7 Type: NodePort

![11-nodePort理解](/assets/images/Cloud-Native/02-k8s-macro-architecture/11-nodePort理解.png)

NodePort 服务特征如如下:
- (1)每个端口只能是一种服务
- (2)端口范围只能是 30000-32767 (可调整)
- (3)不在 YAML 配置文件中指定则会分配一个默认端口

nodePort的原理在于：
- 在node上开了一个端口，将向该端口的流量导入到kube-proxy，然后由 kube-proxy进一步到给对应的pod


- 创建Servcie时指定type为Nodeport，会在所有集群节点上开一个相同的端口
- 创建Nodeport时的同时会创建ClusterIP
- **指定端口不是强制的，如果忽略，集群会自动分配一个**

- 在集群内部可以通过clusterip访问
  - 到达任何一个节点的连接将会被重定向到一个随机的pod，该pod不一定是位于接收节点的Pod.
- 集群外部通过节点IP:nodeport 访问。

#### 2.1.8 Type: LoadBalancer

- LoadBlancer Service是 kubernetes 深度结合云平台的一个组件;
- 当使用 LoadBlancer Service暴露服务时，实际上是通过向底层云平台申请创建一个负载均衡器来向外暴露服务;
- 目前 LoadBlancerService 支持的云平台已经相对完善，比如国外的 GCE、DigitalOcean，国内的阿里云，私有云Openstack 等等，

> 由于 LoadBlancer Service 深度结合了云平台，所以只能在一些云平台上来使用

- LoadBalancer和NodePort很相似，目的都是向外部暴露一个端口，区别在于:
  - LoadBalancer会在集群的外部再来做一个负载均衡设备，而这个设备需要外部环境的支持，
  - 外部服务发送到这个设备上的请求,会被设备负载之后转发到集群中。

![12-loadBalance理解](/assets/images/Cloud-Native/02-k8s-macro-architecture/12-loadBalance理解.png)

- LoadBalancer类型的service 是可以实现集群外部访问服务的另外一种解决方案。
- 不过并不是所有的k8s集群都会支持，大多是在公有云托管集群中会支持该类型。
- 如果环境不支持，则不会调负责均衡器，该服务就像一个Nodeport服务一样了。
- 负载均衡器是异步创建的，关于被提供的负载均衡器的信息将会通过Service的status.loadBalancer字段被发布出去。

#### 2.1.9 Type: ExternalName

见下方：2.2.1 Service 也可以指向 k8s 外的服务

#### 2.1.10 SVC 的 DNS 解析

service: 
- A记录：非Headless Service: my-svc.my-namespace.svc.cluster.local，解析到 ClusterIP
- A记录：Headless Service: my-svc.my-namespace.svc.cluster.local，解析到一组 Pod IP
pod:
- A记录：pod-ip-address.my-namespace.pod.cluster.local

#### 2.1.11 SVC 流量分发的底层原理

- api-server 监听 kube-proxy 来进行服务、端口的发现
- kube-proxy 监控所哟 pod 节点信息、标签、IP、port等信息，并把它们写入到 iptables 规则链中。
- client 访问 svc，其实访问的是 iptables 规则，再由 iptables 规则导向后端 pod 节点

- service 在很多情况下只是一个概念，真正起作用的其实是 kube-proxy 服务进程
- 当创建service 的时候，会通过 apiserver 向 etcd 写入创建的service的信息，而kube-proxy会基于监听机制发现service的变化，然后将最新的service信息转换为iptables规则，从而实现流量分发。



### 2.2 Endpoint

- 服务service和pod并不是直接相连的，它们之间有个Endpoint资源。
- Endpoint资源就是一组pod的ip地址和端口的列表。
- 和其他k8s资源一样，可以通过kubectl get endpoints 查看信息。

- 在Service的spec中定义的pod选择器，不是由服务直接使用，而是用于构建IP和端口列表，然后存储在Endpoint资源中。
- 如果创建服务时不指定选择器，将不会创建Endpoints（没有选择器不知道服务该包含哪些pod），这样可以分别手动配置它们。

#### 2.2.1 Service 也可以指向 k8s 外的服务

- Endpoint对象需要与服务同样的名称，并包含目标IP地址和端口列表，这样服务就可以和有选择器一样正常使用了。
- 将IP配置为外部服务，就可以连接外部服务了


创建外部服务的别名：
- 创建service时指定type字段为ExternalName,例如有一个外部服务域名为api.somecompany.com
- 服务创建后可以通过external-service.defaul.svc.cluster.local 连接外部服务
- ExternaleName服务仅在DNS级别实施，为服务创建了简单的CNAME DNS记录，因此，连接到服务的客户端将直接连接到外部服务，完全绕过服务代理。
- 出于这个原因，这些类型的服务不会有集群IP.
- 注意：CNAME 记录指向完全限定的域名而不是数字IP地址。

### 2.3 Ingress

- 上文介绍的LoadBalancer需要自己的负载均衡器，以及独有的公网IP地址，而Ingress只需要一个公网IP就能为许多服务提供访问。
- 当客户端向Ingress发送http请求时，Ingress会根据请求的**主机名和路径**决定要发送的服务。
- Ingress在**应用层**工作，可以提供基于cookie的会话亲和性功能。
- Ingress必需要Ingress控制器才能运行，不同的k8s环境使用不同的控制器实现，首先要确保环境启用了Ingress控制器。
- 注意：
  - 使用Ingress，需要客户端访问域名能指向Ingress Controller所在host
  - 配置DNS服务器将kubia.example.com指向得到的IP
  - 或者在客户端 /etc/hosts 文件中配置。
- Ingress，可以指向Service.

### 2.4 如何排除服务故障

- 首先， 确保从集群内连接到服务的集群IP,而不是从外部
- 不要通过ping服务IP 来判断服务是否可访问（请记住，服务的集群IP是虚拟IP, 是无法ping通的）。
- 如果已经定义了就绪探针， 请确保 它返回成功；否则该pod不会成为服务的一部分 。
- 要确认某个容器是服务的一部分， 请使用kubectl get endpoints来检查相应的端点对象。
- 如果尝试通过FQDN或其中一部分来访问服务（例如,myservice.mynamespace.svc.cluster.local或 myservice.mynamespace), 但
- 并不起作用， 请查看是否可以使用其集群IP而不是FQDN来访问服务 。
- 检查是否连接到服务公开的端口，而不是目标端口 。
- 尝试直接连接到podIP以确认pod正在接收正确端口上的 连接。
- 如果甚至无法通过pod的IP 访问应用， 请确保应用不是仅绑定 到本地主机。

### 2.5 四种port：NodePort, Port, TargetPort, ContainerPort

NodePort: 
- nodePort提供了集群外部客户端访问service的一种方式，nodePort提供了集群外部客户端访问service的端口，
- 即nodeIP:nodePort提供了外部流量访问k8s集群中service的入口。

Port:
- port是暴露在cluster ip上的端口，port提供了集群内部客户端访问service的入口，即clusterlP:port。

TargetPort:
- targetPort是pod上的端口，

ContainerPort:
- containerPort是在pod控制器中定义的、pod中的容器需要暴露的端口。
- 如spring boot的8080，mysql的3306等。

总结：
- NodePort是kubernetes对外公布的一个和（某一个）service关联的端口，当外部流量到达这个nodePort时，service会将它导入port定义的端口上。
- 然后，再从port定义的端口将流量导入到targetPort定义的端口。targetPort是容器对外暴露的端口，你可以理解为targetPort:containerPort。

文档：
- https://medium.com/@deepeshtripathi/all-about-kubernetes-port-types-nodeport-targetport-port-containerport-e9f447330b19


### 2.6 基于客户端地址的会话保持模式的svc负载分发策略

对Service的访问被分发到了后端的Pod上去，目前kubernetes提供了两种负载分发策略:
- 如果不定义，默认使用kube-proxy的策略，比如轮询等。
- 基于客户端地址的会话保持模式，即来自同一个客户端发起的所有请求都会转发到固定的一个Pod上
  - 这对于传统基于Session的认证项目来说很友好，此模式可以在spec中添加sessionAffinity:ClientIP选项。


### 2.6 简单总结

Ingress: 指向service
Service: 指向Endpoint
- type:
  - ClusterIP
  - NodePort
  - LoadBalancer
  - ExternalName
Endpoint: 指向pods
Pods
- Probes:
  - readinessProbe
  - livenessProbe

## 3 RBAC

角色：
- Role 只能用来给某个特定 namespace 中的资源作鉴权。
- ClusterRole 定义可用于授予用户对某一个或多个 namespace，集群级资源 和 非资源类的 API（如 /healthz）做鉴权

授权：
- RoleBinding：把 Role 或 ClusterRole 中定义的各种权限映射到 User，Service Account 或者 Group，从而让这些用户继承角色在 namespace 中的权限。
- ClusterRoleBinding： 让用户继承 ClusterRole 在整个集群中的权限。

角色：
- 默认角色：
  - API Server 会创建一组默认的 ClusterRole 和 ClusterRoleBinding 对象。 
  - 这些默认对象中有许多包含 system: 前缀，表明这些资源由 Kubernetes基础组件”拥有”。 
    - 对这些资源的修改可能导致集群不可用。
    - 所有默认的 ClusterRole 和 ClusterRoleBinding 对象都会有一个Label: kubernetes.io/bootstrapping=rbac-defaults。
- 面向用户的角色：
  - 通过命令 kubectl get clusterrole 查看到并不是所有都是以 system:前缀，它们是面向用户的角色。
  - 这些角色包含超级用户角色（cluster-admin），即旨在利用 ClusterRoleBinding（cluster-status）在集群范围内授权的角色， 
    - cluster-admin：超级用户权限，允许对任何资源执行任何操作。 
  - 以及那些使用 RoleBinding（admin、edit和view）在特定命名空间中授权的角色。
    - admin：管理员权限，利用 RoleBinding 在某一命名空间内部授予。
    - edit：允许对某一个命名空间内大部分对象的读写访问，但不允许查看或者修改角色或者角色绑定。
    - view：允许对某一个命名空间内大部分对象的只读访问。 不允许查看角色或者角色绑定。 由于可扩散性等原因，不允许查看 secret 资源。
    - 核心组件角色、其它组件角色 和 控制器（Controller）角色 这里不在一一列出。

用户：
- K8S中有两种用户：
  - 服务账号(ServiceAccount)
    - ServiceAccount是由K8S管理的
  - 普通意义上的用户(User)
    - 而User通常是在外部管理，K8S不存储用户列表
    - 也就是说，添加/编辑/删除用户都是在外部进行，无需与K8S API交互
- doc:
  - https://zhuanlan.zhihu.com/p/43237959

用户组：
- 在Kubernetes集群中，证书内容中的通用名称（common name）CN 被当作请求的用户名，(Organization) O 被当作用户组
- doc:
  - https://blog.csdn.net/dustzhu/article/details/108853549

关于用户、用户组：
- k8s不管理用户（User）、用户组
- k8s通过访问的证书的信息 CN 来确认 用户，通过 O 来确认用户组。
- 证书的 CN, O 是集群管理员在创建证书时指定的。
- 所以：
  - 要创建用户、用户组：需要创建证书。
  - 要更改用户名、用户组名：需要重新创建证书。
  - 要更改用户所属的组：需要重新创建证书；
  - 要查看组有那些用户：不在k8s中看，而在csr列表中查看。

```bash
# 查看角色
kubectl get roles -owide -A
kubectl get clusterroles -owide -A

# 查看角色绑定
kubectl get rolebindings -owide -A
kubectl get clusterrolebindings -owide -A

# 查看用户
https://gitee.com/ycvayne/sre/blob/master/k8s/rbac/get-users.sh

# 查看用户组
https://gitee.com/ycvayne/sre/blob/master/k8s/rbac/get-groups.sh

# 查看用户组中的用户
https://gitee.com/ycvayne/sre/blob/master/k8s/rbac/get-group-users.sh

# 查看用户的角色
https://gitee.com/ycvayne/sre/blob/master/k8s/rbac/get-user-roles.sh

# 查看用户组的角色
https://gitee.com/ycvayne/sre/blob/master/k8s/rbac/get-group-roles.sh
```

## 4. 标签、标签选择器、注解、字段选择器

### 4.1 标签

- 标签(Label)是附加在Kubernetes对象上的一组键值对
- 作用：
  - 按照对用户有意义的方式来标识Kubernetes对象，同时，又不对Kubernetes的核心逻辑产生影响。
  - 标签可以用来组织和选择一组Kubernetes对象
  - 例子：
    - 使用标签，用户可以按照自己期望的形式组织 Kubernetes 对象之间的结构，而无需对 Kubernetes 有任何修改。
    - 应用程序的部署或者批处理程序的部署通常都是多维度的(例如，多个高可用分区、多个程序版本、多个微服务分层)。
    - 管理这些对象时，很多时候要针对某一个维度的条件做整体操作，
    - 例如，将某个版本的程序整体删除，这种情况下，如果用户能够事先规划好标签的使用，再通过标签进行选择，就会非常地便捷。
- 使用：
  - 您可以在创建Kubernetes对象时为其添加标签，也可以在创建以后再为其添加标签。
  - 每个Kubernetes对象可以有多个标签，同一个对象的标签的 Key 必须唯一
  - 那些非标识性的信息应该记录在注解 (annotation)
- 语法：
  - 标签的 key：
    - 可以有两个部分: **可选的前缀**和**必须的标签名**，通过 / 分隔
    - 前缀：
      - 如果省略标签前缀，则标签的 key 将被认为是专属于用户的。
      - Kubernetes的系统组件 (例如，kube.scheduler、kube-controller-manager、kube-apiserver、kubectl 或其他第三方组件)向用户的Kubernetes 对象添加标签时，必须指定一个前缀。 
      - kubernetes.io/和 k8s.io/ 这两个前缀是Kubernetes 核心组件预留的。
  - 标签的 value：
    - 可以为空

### 4.2 标签选择器

- 作用：
  - 通常来讲，会有多个Kubernetes对象包含相同的标签。通过使用标签选择器 (label selector)，用户客户端可以选择一组对象。
  - 标签选择器 (label selector)是 Kubernetes 中最主要的分类和筛选手段。

- 形式：
  - Kubernetes apiserver支持两种形式的标签选择器，equality-based 基于等式的和set-based 基于集合的。
  - 基于等式的选择方式
    - 三种操作符： =、==、!=。
      - 前两个操作符含义是一样的，都代表相等，后一个操作符代表不相等
  - 基于集合的选择方式
    - 支持的操作符有三种:in、notin、exists.

- 使用：
  - 标签选择器可以包含多个条件，并使用逗号分隔，此时只有满足所有条件的 Kubernetes 对象才会被选中

- 举例: 标签编写规则
```bash
# 选择所有的包含 environment 标签且值为 'production' 或 'qa' 的对象
environment in (production，qa)

# 选择所有的 tier 标签不为 'frontend' 和 'backend' 的对象，或不含 'tier’ 标签的对象
tier notin (frontend，backend)

# 选择所有包含 'partition' 标签的对象
partition

# 选择所有不包含 'partition' 标签的对象
!partition

# 选择包含 'partition'  标签(不检查标签值) 且 environment 不是 qa 的对象
partition, environment notin (qa)
```

- 举例：使用标签进行删选

matchLabels:
- matchLabels 是一个 {key,value}组成的 map.
- map 中的一个 {key,value} 条目相当于matchExpressions 中的一个元素，
- 其 key 为 map 的 key,operator 为 In， values 数组则只包含 value 一个元素。

matchExpression:
- matchExpression 等价于基于集合的选择方式，
- 支持的 operator 有 In、NotIn、Exists 和 DoesNotExist。
- 当 operator 为 In 或 NotIn 时,values 数组不能为空。

> 所有的选择条件都以 AND 的形式合并计算，即所有的条件都满足才可以算是匹配

```bash
kubectl get pods -l 'environment in (production), tier in (frontend)'

# Job、Deployment、ReplicaSet 和 DaemonSet 同时支持基于等式的选择方式和基于集合的选择方式。
selector:
  matchLabe1s:
    component: redis
  matchExpressions:
    - {key: tier, operator: In，values: [cache]}
    - {key: environment，operator: NotIn，values: [dev]}
```

### 4.3 注解

- 注解 (annotation)可以用来向 Kubernetes 对象的 metadata.annotations 字段添加任意的信息.
- Kubernetes 的客户端或者自动化工具可以存取这些信息以实现其自定义的逻辑。

### 4.4 字段选择器

- 字段选择器 ( Field selectors )允许您根据一个或多个资源字段的值 筛选 Kubernetes 资源
- 下面是一些使用字段选择器查询的例子:

```bash
metadata.name=my-service
metadata.namespace!=default
status.phase=Pending

kubect1 get pods --fie1d-selector status.phase=Running
```

## 5 k8s的网络

文档：
- [Kubernetes网络模型](https://kuboard.cn/learning/k8s-intermediate/service/network.html)


network namespace:
- 通常，我们认为虚拟机中的网络通信是直接使用以太网设备进行的，实际情况比这个示意图更加复杂一些。
- Linux系统中，每一个进程都在一个 network namespace (opens new window)中进行通信，
  - network namespace 提供了一个逻辑上的网络堆栈（包含自己的路由、防火墙规则、网络设备）。
  - 换句话说，network namespace 为其中的所有进程提供了一个全新的网络堆栈。
- 当创建 network namespace 时，同时将在 /var/run/netns 下创建一个挂载点（mount point）用于存储该 namespace 的信息。
- 执行 ls /var/run/netns 命令，或执行 ip 命令，可以查看所有的 network namespace.
- 默认情况下，Linux 将所有的进程都分配到 root network namespace，以使得进程可以访问外部网络.

k8s network namespace:
- 在 Kubernetes 中，Pod 是一组 docker 容器的集合，这一组 docker 容器将共享一个 network namespace。
- Kubernetes 为每一个 Pod 都创建了一个 network namespace。

k8s pod通信：
- 在 Kubernetes 中，每一个 Pod 都有一个真实的 IP 地址，并且每一个 Pod 都可以使用此 IP 地址与 其他 Pod 通信。
- 两个 Pod 在同一个节点上
  - pod 的ip
    - 从 Pod 的视角来看，Pod 是在其自身所在的 network namespace 与同节点上另外一个 network namespace 进程通信。
    - 在Linux上，不同的 network namespace 可以通过 Virtual Ethernet Device (opens new window)或 veth pair (两块跨多个名称空间的虚拟网卡)进行通信。
    - 每一组 veth pair 类似于一条网线，连接两端，并可以使流量通过。节点上有多少个 Pod，就会设置多少组 veth pair。
      ![pod-veth-pairs](https://kuboard.cn/assets/img/pod-veth-pairs.242c359f.png)
  - root network namespace的 network bridge
    - 为了让 Pod 可以互相通过 root network namespace 通信，我们将使用 network bridge（网桥）。
    - Linux Ethernet bridge 是一个虚拟的 Layer 2 网络设备，可用来连接两个或多个网段（network segment）。
      ![pods-connected-by-bridge](https://kuboard.cn/assets/img/pods-connected-by-bridge.8f775095.png)
  - 在 network namespace 将每一个 Pod 隔离到各自的网络堆栈的情况下，
    - 虚拟以太网设备（virtual Ethernet device）将每一个 namespace 连接到 root namespace，
    - 网桥将 namespace 又连接到一起，
  - 此时，Pod 可以向同一节点上的另一个 Pod 发送网络报文了。

- Pod 在不同节点上
  - Kubernetes 网络模型要求 Pod 的 IP 在整个网络中都可访问，但是并不指定如何实现这一点。实际上，这是所使用网络插件相关的，但是，仍然有一些模式已经被确立了。
  - 通常，集群中每个节点都被分配了一个 CIDR 网段，指定了该节点上的 Pod 可用的 IP 地址段。
  - 一旦发送到该 CIDR 网段的流量到达节点，就由节点负责将流量继续转发给对应的 Pod。


### k8s 中有几个网络

Pod 网络（Cluster CIDR）
- 例如：10.244.0.0/16
- 这个网段用于分配给集群中各个 Pod 的 IP 地址。CNI 插件会使用这个范围为每个节点分配子网。

服务网络（Service Cluster IP Range）
- 例如：10.96.0.0/16
- 这个网段用于为集群内的服务分配虚拟 IP 地址（Cluster IP），这些 IP 用于服务发现和负载均衡。

/etc/cni/net.d/10-containerd-net.conflist
- /etc/cni/net.d/10-containerd-net.conflist 是必须配置的，因为它定义了 CNI 插件如何为节点上的 Pod 分配网络资源。
- 如果没有这个配置，Kubernetes 将无法正确地为 Pod 设置网络连接，导致 Pod 无法通信。
- 使用 Flannel 时，你不需要手动设置每个节点的 /etc/cni/net.d/10-containerd-net.conflist 文件。Flannel 会自动管理网络配置。
- 自动配置：
  - Flannel 的 ConfigMap，自动配置了10.244.0.0/16：
    ```yaml
    {
      "Network": "10.244.0.0/16",
      "EnableNFTables": false,
      "Backend": {
        "Type": "vxlan"
      }
    }
    ```
- 手动配置：
  - [参考这里](https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/migrating-from-dockershim/troubleshooting-cni-plugin-related-errors/)


## OCI, CRI, CNI, CSI, CNM, CRD

首先我们来看一张图

![09-worker架构图](/assets/images/Cloud-Native/02-k8s-macro-architecture/09-worker架构图.png)


|缩写| 描述|
|:----|:----|
|CNI|- CNI 就是这样一个标准，它旨在为容器平台提供**网络的标准**化。<br/>- 不同的容器平台能够通过相同的接口调用不同的网络组件。|


OCI (Open Container Initiative)：
- 容器开放接口规范，
- 由多家公司共同组成于2015年6月成立的项目 (Docker,Google,CoreOS等公司)，并由Linux基金会运行管理，
- 旨在围绕容器格式和运行时制定一个开放的工业化标准. 
- 目前主要有两个标准文档: 
  - 容器运行时标准 (runtime spec)和
  - 容器镜像标准(image spec)

CRI (Container Runtime Interface) :
- 容器运行时接口，提供计算资源。
- kubernetes1.5版本之后，kubernetes项目推出了自己的运行时接口api-CRI(container runtimeinterface)。
- CRI由SIG-Node小组维护(Kubernete社区的一个小组)
  - kubernetes的社区是以SIG (Special Interest Group特别兴趣小组)和工作组的形式组织，每个工作组都会定期召开视频会议。
- 常见CRI组件:
  - cri-o: 同时兼容OC和CRI的容器运行时;
  - cri-containerd: 基于Containerd的kubernetes实现
  - rkt: 由于CoreOS主推的用来跟docker抗衡的容器运行时:
  - frakti: 基于hypervisor的CRI;
  - docker: kuberentes最初就开始支持的容器运行时，目前还没完全从kubelet中解耦，docker公司同时推广了OCI标准;
  - clear-containers: 由Intel推出的同时兼容OCI和CRI的容器运行时;
  - kata-containers: 符合OCI规范同时兼容CRI;
  - pouchContainer: 阿里巴巴集团开源的高效、轻量级企业级容器引擎技术，拥有隔离性强、可移植性高、资源占用少等特性。
- 更多文档：
  - [Docker，containerd，CRI，CRI-O，OCI，runc 分不清？看这一篇就够了](https://cloud.tencent.com/developer/article/1988350)
  - [Containerd 的前世今生和保姆级入门教程](https://www.cnblogs.com/ryanyangcs/p/14168400.html)

CNI (Container Network Interface)
- 容器网络接口，提供网络资源。
- 是和 CoreOS 主导制定的容器网络标准，它本身并不是实现或者代码，可以理解成一个协议。
- CNI旨在为容器平台提供网络的标准化。
- 容器平台可以从CNI获取到满足网络互通条件的网络参数(如IP地址、网关、路由、DNS等)。
- CNI标准规范是在rkt网络提议的基础上发展起来的，综合考虑了灵活性、扩展性、ip 分配、多网卡等因素。
- 不同的容器平台(比如kubernetes、Mesos和 RKT) 能够通过相同的接口调用不同的网络组件
- 这个协议连接了两个组件: 容器管理系统和网络插件，具体的事情都是插件实现，包括: 
  - 创建容器网络空间 (network namespace)、
  - 把网络接口(interface)放到对应的网络空间、
  - 给网络接口分配IP等。
- 常见CNI组件:
  - Flannel: 目前最普遍的实现，同时支持overlayer (UDP，VxLAN)和underlayer(host-gw)的网络后端实现;
  - Calico: 基于BGP的underlayer方案，功能丰富，对底层网络有一定要求;
  - Cilium: 基于eBPF和XDP的高性能Overlayer方案
  - Kube-Route: 采用BGP提供网络直连，集成基于LVS的负载均衡能力;
  - WeaveNet: 采用UDP封装实现的L2 0verlayer方案，支持用户态(慢，可加密)和内核态(快，无加密)两种实现;


CNM (Container Network Model) :
- 容器网络模型，提供网络资源。
- 由Docker公司提出.
- CNI CNM 的不同：
  - CNI(CoreOS公司)支持与第三方IPAM的集成，可以用于任何容器runtime。
    - 由于CNI简单的设计，许多人认为编写CNI插件会比编写CNM插件来得简单。
    - 模块化设计，模块化，增加了用户的选择。也促进了第三方网络插件以创新来提供更高级的网络功能
  - CNM(Docker公司)从设计上就仅仅支持Docker。
    - CNM 模式下的网络驱动不能访问容器的网络命名空间。
    - 这样做的好处是libnetwork可以为冲突解决提供仲裁。
    - 一个例子是: 
      - 两个独立的网络驱动提供同样的静态路由配置，但是却指向不同的下一跳IP地址。
    - 与此不同，CNI允许驱动访问容器的网络命名空间。CNI正在研究在类似情况下如何提供仲裁。


CSI (Container Storage Interface) :
- 容器存储接口，提供存储资源。
- 由 kubernetes、Mesos、Docker 等社区成员联合制定的一个行业标准接口规范，旨在将任意存储系统暴露给容器化应用程序。
- kubernetes 1.9 版为alpha阶段-->kubernetes 1.10版为beta阶段->kubernetes 1.13 GA.
- kubernetes中 Volume 的生命周期与Pod的生命周期相同，跟容器的生命周期不想关，当容器终止或重启时，Volume 中的数据不会丢失。
- 这点跟Docker中的Volume不同。
- kubernetes支持两种资源的供应模式:
  - 静态模式(Static)和动态模式(Dynamic)。


CRD (Custom Resource Definition):
- CRD到底有什么用：
  - Kubernetes API 默认提供的众多的功能性资源类型可用于容器编排以解决多数场景中的编排需求。
  - 但是，有些场景也需要借助于额外的或更高级别的编排抽象。例如：
     - 引入集群外部的一些服务并以资源对象的形式进行管理，
     - 或者把 Kubernetes 的多个标准资源对象合并为一个单一的更高级别的资源抽象
     - 等等。
  - 而这类的 API 扩展抽象通常也应该兼容 Kubernetes 系统的基本特性，如
     - 支持 kubectl 管理工具、CRUD 及 watch 机制、
     - 标签、etcd 存储、认证、授权、RBAC 及审计，等等，
     - 从而使得用户可将精力集中于构建业务逻辑本身。
  - 目前，扩展 Kubernetes API 的常用方式有三种：
    - 使用 CRD（CustomResourceDefinitions）自定义资源类型。
    - 开发自定义的 API Server 并聚合至主 API Server。
    - 定制扩展 Kubernetes 源码。
- k8s允许用户自定义资源
- 定义CRM对象，会创建一个具有您指定的名称和架构的新定义资源.
- Kubernetes API 提供并处理您的自定义资源的存储.

- 用户资源定义，拓展能力。
- Kubernetes 1.7+.

- 文档：
  - [CRD 就像 Kubernetes 中的一张表！](https://zhuanlan.zhihu.com/p/260797410)
  - [Kubernetes——自定义资源类型（CRD）](https://www.cnblogs.com/zuoyang/p/16454293.html)
