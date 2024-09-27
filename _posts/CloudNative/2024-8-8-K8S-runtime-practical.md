---
layout: post
title: 【Cloud Native】03 K8S 运行时实操
categories: [CloudNative, K8S学习圣经]
tags: [安装Kubernetes]
---

## 0 文档

reference:

- [k8s学习圣经](https://www.cnblogs.com/crazymakercircle/p/17304541.html)

## 1 部署 K8S

k8s有多种部署方式，目前主流的方式有3种：minikube、kubeadm、二进制包。

- 1）minikube：一个用于快速搭建kubernetes集群的工具
  - minikube 是本地 Kubernetes，
  - minikube基于go语言开发， 是一个易于在本地运行 Kubernetes 的工具，可在你的笔记本电脑上的虚拟机内轻松创建单机版 Kubernetes 集群。
  - 便于尝试 Kubernetes 或使用 Kubernetes 日常开发。
  - 可以在单机环境下快速搭建可用的k8s集群，非常适合测试和本地开发。
  - 现有的大部分在线k8s实验环境也是基于minikube.
  - 在生产环境 Kubernetes 集群主要有两种方式: kubeadm, 二进制包。
- 2）kubeadm：一个用于快速搭建单节点kubernetes的工具
  - Kubeadm 是一个 K8s 部署工具，提供 kubeadm init 和 kubeadm join，用于快速部署 Kubernetes 集群。
  - Kubeadm 降低部署门槛，但屏蔽了很多细节，遇到问题很难排查。
  - [官方地址](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm/)
- 3）二进制包：从官网下载每个组件的二进制包，依次去安装，此方式对于理解kubernetes组件更加有效
  - 从 github 下载发行版的二进制包，手动部署每个组件，组成 Kubernetes 集群。
  - 如果想更容易可控，推荐使用二进制包部署 Kubernetes 集群，虽然手动部署麻烦点，期间可以学习很多工作原理，也利于后期维护。

### 1.1 minikube

#### 1.1.1 env

- kernel: 6.5.0-27-generic
- os: ubuntu 22.04

#### 1.1.2 install cmds

[minikube 安装步骤]({% link _posts/CloudNative/2024-8-13-k8s-install-by-minikube.md %})

other install doc:

- [云原生,kubernetes,minikube的部署安装完全手册（修订版）](https://developer.aliyun.com/article/1399680)


#### 1.1.3 other minikube cmds

```bash
# 暂停minikube集群
minikube pause
# 取消暂停minikube集群
minikube unpause
# 停止minikube集群
minikube stop
# 设置minikube集群资源
minikube config set memory 9001
# 查看安装在minikube集群的服务
minikube addons list
# 删除minikebe集群
minikube delete --all
# 重建minikube集群
minikube delete
minikube start --driver=docker // 这里指定了用docker，不指定也会自动检测
```

minikube 启动后，用 kubectl 相关命令即可。可参考[这里](https://blog.csdn.net/ohalo/article/details/126245902)。

更多minikube命令参考：
- [该网页搜索: minikube常用命令](https://www.cnblogs.com/crazymakercircle/p/17304541.html)

### 1.2 kubeadm

请注意：

- 若集群只有1台机器，则需要允许单节点运行 Pods（我没试过）：
  - 在默认情况下，Kubernetes 不允许在控制平面节点（即 master 节点）上调度 Pods。要允许这台单节点机器上同时运行 master 和 node 角色，需要移除节点上的污点：

```
kubectl taint nodes --all node-role.kubernetes.io/control-plane- node-role.kubernetes.io/master-
```

部署参考文档:
- [史上最全干货！Kubernetes 原理+实战总结（全文6万字，90张图，100个知识点）](https://developer.aliyun.com/article/1366693)
- [Ubuntu 22.04 部署基于IPVS的高可用Kubernetes v1.27.6集群](https://warnier-zhang.github.io/How-to-deploy-Kubernetes-with-IPVS-on-Ubuntu/)

#### 1.2.1 env

- kernel: 6.5.0-27-generic
- os: ubuntu 22.04

- master: 192.168.31.143
- node: 192.168.31.144
- node: 192.168.31.145


#### 1.2.2 install cmds

[kubeadm 安装步骤]({% link _posts/CloudNative/2024-8-14-k8s-install-by-kubeadm.md %})

> refer doc:
> - [国内无法拉取Docker镜像了？这些方法拯救你的Docker](https://cloud.tencent.com/developer/article/2434428)
> - [k8s-container unix:///run/crio/crio.sock unix:///var/run/cri-dockerd.sock](https://www.cnblogs.com/LILEIYAO/p/17169234.html)

一些容器镜像站:
- https://cloud.tencent.com/developer/article/2428972
- https://hub.si.icu/


### 1.3 二进制包


#### 1.3.1 env

- kernel: 6.5.0-27-generic
- os: ubuntu 22.04

- master: 192.168.31.143
- node: 192.168.31.144
- node: 192.168.31.145


#### 1.3.2 install cmds

[二进制包 安装步骤]({% link _posts/CloudNative/2024-8-15-k8s-install-by-binary.md %})


### 1.4 Helm

#### 1.4.1 Helm 是什么

- java 使用 maven;
- 前端使用 npm;
- python 使用 pip; 
- 运维使用yum 或 apt.

分工不同，诉求却相同，都希望有一种资源管理工具，可以方便查找、下载、安装、使用和分发程序包。

helm 也一样，它是 k8s 的资源包管理工具。它使我们操作的对象不再是单个资源，而是一个实体.

- 比如我们需要一个负载均衡的web 服务，如果不使用 helm，我们需要写 deployment，service和ingress 才可以让集群外部的客户使用。
- 但是如果使用 helm，直接使用一个install 命令便可以实现相同的功能。

Helm是Deis 开发的一个用于 Kubernetes 应用的包管理T具，主要用来管理 Charts。有点类似于Ubuntu 中的 APT或 CentOs 中的 YUM。

#### 1.4.2 Helm Chart 是什么

Helm Chart 是用来封装 Kubernetes 原生应用程序的一系列YAML 文件。可以在你部署应用的时候自定义应用程序的一些 Metadata，以便于应用程序的分发。

- 对于应用发布者而言，可以通过 Helm 打包应用、管理应用依赖关系、管理应用版本并发布应用到软件仓库。
- 对于使用者而言，使用 Helm 后不用需要编写复杂的应用部署文件，可以以简单的方式在 Kubernetes上查找、安装、升级、回滚、卸载应用程序。

#### 1.4.3 组件

Helm
- Helm是一个命令行下的客户端工具。主要用于 Kubernetes 应用程序 Chart的创建、打包、发布以及创建和管理本地和远程的 Chart 仓库。

Chart
- Helm 的软件包，采用TAR格式。类似于APT的DEB 包或者YUM 的RPM包，其包含了一组定义 Kubernetes 资源相关的 YAML 文件。

Repoistory
- Helm 的软件仓库，Repository 本质上是一个Web 服务器，该服务器保存了一系列的 Chart软件包以供用户下载，并且提供了一个该 Repository的 Chart包的清单文件以供查询。
- Helm 可以同时管理多个不同的 Repository。

Release
- 使用 helm install 命令在 Kubernetes 集群中部署的 Chart 称为 Release。

Tiller (Helm3已弃用Tiller)
- Tiller 是 Helm 的服务端，部署在 Kubernetes 集群中。
- Tiller 用于接收 Helm 的请求，并根据Chart生成Kubernetes 的部署文件 (Helm 称为 Release )，然后提交给 Kubernetes 创建应用。

#### 1.4.4 Helm工作流程简述

- 开发者首先创建并编辑chart的配置；
- 接着打包并发布至Helm的仓库（Repository）；
- 当管理员使用helm命令安装时，相关的依赖会从仓库下载；
- 接着helm会根据下载的配置部署资源至k8s；
- Tiller 还提供了 Release 的升级、删除、回滚等一系列功能。

#### 1.4.5 Helm安装

在 [二进制安装中，有Helm的安装步骤](https://yc913344706.github.io/posts/k8s-install-by-binary/#106-helm)

#### 1.4.6 Helm使用

- https://developer.aliyun.com/article/1207395

#### 1.4.7 搭建自有 chart 仓库

- [搭建chart仓库](https://www.zhaowenyu.com/helm-doc/hub/create-chart-hub.html)
- [常见公共 Chart 仓库](https://developer.aliyun.com/article/1319780)


### 1.5 数据持久化

- etcd: 是Kubernetes提供默认的存储系统，保存所有集群数据，使用时需要为etcd数据提供备份计划。



## 100 some interesting knowledge

### 100.1 software version meaning

| name | description |
| - | - |
| alpha | α是希腊字母的第一个，表示**最早的版本**，内部测试版，一般不向外部发布，<br/> bug会比较多，功能也不全，一般只有测试人员使用。 |
| beta | β是希腊字母的第二个，公开测试版，比alpha版本晚些，主要会有“粉丝用户”测试使用，<br/> 该版本仍然存在很多bug，但比alpha版本稳定一些。这个阶段版本还**会不断增加新功能**。<br/> 分为Beta1、Beta2等，直到逐渐稳定下来进入RC版本。 |
| RC | RC是 Release Candidate 的缩写，即候选发布版本，**基本不再加入新的功能，主要修复bug**。<br/> 是最终发布成正式版的前一个版本，将bug修改完就可以发布成正式版了。 |
| GA | General Availability，正式发布的版本，**官方开始推荐广泛使用**，<br/> 国外有的用GA来表示release版本。 |
| RELEASE | 正式发布版，官方推荐使用的版本，有的用GA来表示 |
| Final | Final最终版，也是正式发布版的一种表示方法。 |
| Trial          | 试用版，通常都有时间限制，有些试用版软件还在功能上做了一定的限制。<br/> 可注册或购买成为正式版 |
| Unregistered   | 未注册版，通常没有时间限制，在功能上相对于正式版做了一定的限制。<br/> 可注册或购买成为正式版。 |
| Demo           | 演示版，仅仅集成了正式版中的几个功能，不能升级成正式版。 |
| Lite           | 精简版。 |
| Full version   | 完整版，属于正式版。 |
| Enhance        | 增强版或者加强版，属于正式版 |
| Free           | 自由版 |
| Upgrade        | 升级版 |
| Retail         | 零售版 |
| Cardware       | 属共享软件的一种，只要给作者回复一封电邮或明信片即可。<br/> 有的作者并由此提供注册码等，目前这种形式已不多见。 |
| Plus           | 属增强版，不过这种大部分是在程序界面及多媒体功能上增强。 |
| Preview        | 预览版 |
| Corporation /<br/> Enterprise | 企业版 |
| Standard       | 标准版 |
| Mini           | 迷你版也叫精简版只有最基本的功能 |
| Premium        | 贵价版 |
| Professional(Pro) | 专业版 |
| Express        | 特别版 |
| Regged         | 已注册版 |
| Build          | 内部标号 |
| Delux /<br/> Deluxe          | 豪华版 (deluxe: 豪华的，华丽的) |
| Turbo           | 涡轮版，通常表示性能优化或增强的版本。 |
| Basic           | 基础版，通常包含最基本的功能。 |
| Ultimate        | 终极版，通常包含所有功能和增强。 |
| Essentials      | 精华版，包含核心功能。 |
| Personal        | 个人版，针对个人用户的版本。 |
| Home            | 家庭版，面向家庭用户，通常功能比专业版少。 |
| Developer       | 开发者版，针对开发人员的特别版本，通常包含开发工具和调试功能。 |
| Student         | 学生版，面向教育和学生群体的版本。 |
| Server          | 服务器版，面向服务器使用，通常包含服务器特定功能。 |
| Cloud           | 云端版，面向云计算环境的版本。 |
| OEM             | 原始设备制造商版，通常预装在硬件设备上。 |
| Evaluation      | 评估版，供用户试用评估，一般有时间限制。 |
| Limited         | 限制版，功能或使用权限上有一定限制。 |
| Advanced        | 高级版，功能比标准版多。 |
| Preview         | 预览版，通常是公开测试版的前一个版本，供用户抢先体验。 |
| Legacy          | 旧版，表示不再维护的老版本。 |
| Portable        | 便携版，不需要安装即可运行。 |
| Nightly         | 每夜版，编译每天最新的代码，供测试使用。 |
| Experimental    | 实验版，用于测试新功能和新技术。 |

refer doc:
- [博客摘录「 软件版本GA、RC、Beta、RELEASE、Final、alpha等含义」2024年4月4日](https://blog.csdn.net/njy8850/article/details/137381653)


### 100.2 docker containerd

#### 100.2.1 背景介绍

- 很久以前，**Docker 强势崛起**，以“镜像”这个大招席卷全球，对其他容器技术进行致命的降维打击，使其毫无招架之力，就连 Google 也不例外。
- Google 为了不被拍死在沙滩上，被迫拉下脸面（当然，跪舔是不可能的），希望 Docker 公司和自己联合推进一个**开源的容器运行时**作为 Docker 的核心依赖，不然就走着瞧。Docker公司拒绝了。
- 紧接着，Google 联合 Red Hat、IBM 等几位巨佬，
  - 连哄带骗忽悠 Docker 公司将 **libcontainer** 捐给中立的社区（OCI，Open Container Intiative），并**改名为 runc**
  - 这还不够，为了彻底扭转 Docker 一家独大的局面，几位大佬又合伙成立了一个基金会叫 **CNCF**。
    - CNCF 的目标很明确，既然在当前的维度上干不过 Docker，干脆往上爬，**升级到大规模容器编排的维度**，以此来击败 Docker。
- Docker 公司当然不甘示弱，搬出了 **Swarm** 和 Kubernetes 进行 PK，最后的结局大家都知道了，Swarm 战败。
- 然后 Docker 公司耍了个小聪明，将自己的核心依赖 **Containerd** 捐给了 CNCF，以此来标榜 Docker 是一个 PaaS 平台。
- 巨佬们心想，想当初想和你合作搞个中立的核心运行时，你死要面子活受罪，就是不同意，好家伙，现在自己搞了一个，还捐出来了，这是什么操作？也罢，这倒省事了，我就直接拿 Containerd 来做文章吧。
  - 首先呢，为了表示 Kubernetes 的中立性，当然要搞个**标准化的容器运行时接口**，只要适配了这个接口的容器运行时，都可以和我一起玩耍哦，第一个支持这个接口的当然就是 Containerd 啦。至于这个接口的名字，大家应该都知道了，它叫 **CRI**（Container Runntime Interface）。
  - 这样还不行，为了蛊惑 Docker 公司，Kubernetes 暂时先委屈自己，专门在自己的组件中集成了一个 **shim**（你可以理解为垫片），用来将 CRI 的调用翻译成 Docker 的 API，让 Docker 也能和自己愉快地玩耍，温水煮青蛙，养肥了再杀。。。
  - 就这样，Kubernetes 一边假装和 Docker 愉快玩耍，一边背地里不断优化 Containerd 的健壮性以及和 CRI 对接的丝滑性。现在 Containerd 的翅膀已经完全硬了，是时候卸下我的伪装，和 Docker say bye bye 了。后面的事情大家也都知道了~~
- Docker 这门技术成功了，Docker 这个公司却失败了。
- 时至今日，Containerd 已经变成一个工业级的容器运行时了，连口号都有了：超简单！超健壮！可移植性超强！
- 当然，为了让 Docker 以为自己不会抢饭碗，Containerd 声称自己的设计目的主要是为了嵌入到一个更大的系统中（暗指 Kubernetes），而不是直接由开发人员或终端用户使用。
- 事实上呢，Containerd 现在基本上啥都能干了，开发人员或者终端用户可以在宿主机中管理完整的容器生命周期，包括容器镜像的传输和存储、容器的执行和管理、存储和网络等。大家可以考虑学起来了。

另：

- 2022 年 4 月 dockershim 将会从 Kubernetes 1.24 中完全移除。今后 Kubernetes 将取消对 Docker 的直接支持，而倾向于只使用实现其容器运行时接口的容器运行时，这可能意味着使用 containerd 或 CRI-O。

这是使用 bucketbench 对 Docker、crio 和 Containerd 的性能测试结果，包括启动、停止和删除容器，以比较它们所耗的时间：

![01-compare-docker-containerd](/assets/images/Cloud-Native/03-K8S-runtime-practical/01-compare-docker-containerd.png)

可以看到 Containerd 在各个方面都表现良好，总体性能还是优越于 Docker 和 crio 的。

#### 100.2.2 docker, containerd, CRI, CRI-O, OCI, runc

- Docker，Kubernetes 等工具来运行一个容器时会调用容器运行时（CRI）。比如： containerd，CRI-O
- 通过 CRI 来完成：容器的创建、运行、销毁等实际工作
  - Docker 使用的是 containerd 作为其运行时；
  - Kubernetes 支持 containerd，CRI-O 等多种容器运行时
- 这些容器运行时都遵循了 OCI 规范，并通过 runc 来实现：与操作系统内核交互，来完成容器的创建和运行。

**Docker**

- Docker 启动了整个容器的革命，它创造了一个很好用的工具来处理容器。也叫 Docker，这里最主要的要明白：
  - Docker 并不是这个唯一的容器竞争者
  - 容器也不再与 Docker 这个名字紧密联系在一起
- 目前的**容器工具**中，Docker 只是其中之一，其他著名的容器工具还包括：Podman，LXC，containerd，Buildah 等。
- 因此，如果你认为容器只是关于 Docker 的，那是片面的不对的。

**Docker组成**

- 当你用 Docker 运行一个容器时实际上是通过 Docker 守护程序、containerd 和 runc 来运行它。
  - docker-cli：这是一个命令行工具，它是用来完成 docker pull, build, run, exec 等命令进行交互。
  - containerd：这是一个管理和运行容器的守护进程。它推送和拉动镜像，管理存储和网络，并监督容器的运行。
  - runc：
    - 容器运行时通过 runc 来实现：与操作系统内核交互，来完成容器的创建和运行。
    - 这是低级别的容器运行时间（实际创建和运行容器的东西）。
    - 它包括 libcontainer，一个用于创建容器的基于 Go 的本地实现。

**Docker 镜像**

- 许多人所说的 Docker 镜像，实际上是以 Open Container Initiative（OCI）格式打包的镜像。
- 因此，如果你从 Docker Hub 或其他注册中心拉出一个镜像，你应该能够用 docker 命令使用它，
- 或在 Kubernetes 集群上使用，或用 podman 工具以及任何其他支持 OCI 镜像格式规范的工具。

**Dockershim**

- 用来将 CRI 的调用翻译成 Docker 的 API，让 Docker 也能和自己愉快地玩耍，温水煮青蛙，养肥了再杀。。。
- 2022 年 4 月 dockershim 将会从 Kubernetes 1.24 中完全移除。今后 Kubernetes 将取消对 Docker 的直接支持，而倾向于只使用实现其容器运行时接口的容器运行时，这可能意味着使用 containerd 或 CRI-O。

**Container Runtime Interface (CRI)**

- CRI（容器运行时接口）是 Kubernetes 用来控制创建和管理容器的不同运行时的 API，它使 Kubernetes 更容易使用不同的容器运行时。
- 它是一个插件接口，这意味着任何符合该标准实现的容器运行时都可以被 Kubernetes 所使用。
- Kubernetes 项目不必手动添加对每个运行时的支持，CRI API 描述了 Kubernetes 如何与每个运行时进行交互，由运行时决定如何实际管理容器，因此只要它遵守 CRI 的 API 即可。

**containerd**

- containerd 是一个来自 Docker 的高级容器运行时，并实现了 CRI 规范。
- 它是从 Docker 项目中分离出来，之后 containerd 被捐赠给云原生计算基金会（CNCF）为容器社区提供创建新容器解决方案的基础。

**CRI-O**

- CRI-O 是另一个实现了容器运行时接口（CRI）的高级别容器运行时，可以使用 OCI（开放容器倡议）兼容的运行时，它是 containerd 的一个替代品。
- CRI-O 诞生于 RedHat、IBM、英特尔、SUSE、Hyper 等公司。
- 当容器运行时（Container Runtime）的标准被提出以后，Red Hat 的一些人开始想他们可以构建一个更简单的运行时，而且这个运行时仅仅为 Kubernetes 所用。
- 这样就有了 skunkworks项目，最后定名为 CRI-O， 它实现了一个最小的 CRI 接口。
- 在 2017 Kubecon Austin 的一个演讲中， Walsh 解释说， ”CRI-O 被设计为比其他的方案都要小，遵从 Unix 只做一件事并把它做好的设计哲学，实现组件重用“。

**Open Container Initiative (OCI)**

- OCI 开放容器倡议，是一个由科技公司组成的团体，其目的是：围绕容器镜像和运行时，创建开放的行业标准。他们维护容器镜像格式的规范，以及容器应该如何运行。
- OCI 背后的想法是，你可以选择符合规范的不同运行时，这些运行时都有不同的底层实现。

**runc**

- runc 是轻量级的通用运行时容器，它遵守 OCI 规范，是实现 OCI 接口的最低级别的组件，它与内核交互创建并运行容器。
- runc 为容器提供了所有的低级功能，与现有的低级 Linux 功能交互，如命名空间和控制组，它使用这些功能来创建和运行容器进程。
- runc 的几个替代品：
  - crun: 一个用 C 语言编写的容器运行时（相比之下，runc 是用Go编写的。）
  - kata-runtime: 来自 Katacontainers 项目的 kata-runtime，它将 OCI 规范实现为单独的轻量级虚拟机（硬件虚拟化）。
  - gVisor: Google 的 gVisor，它创建了拥有自己内核的容器。它在其运行时中实现了 OCI，称为 runsc。
- runc 是一个在 Linux 上运行容器的工具，所以这意味着它可以在 Linux 上、裸机上或虚拟机内运行。

> refer doc:
> - [Containerd 的前世今生和保姆级入门教程](https://www.cnblogs.com/ryanyangcs/p/14168400.html)
> - [Docker，containerd，CRI，CRI-O，OCI，runc 分不清？看这一篇就够了](https://cloud.tencent.com/developer/article/1988350)


#### 100.2.3 常见命令

| 功能                    | docker           | ctr（containerd）            | crictl（kubernetes）   |
|-------------------------|------------------|------------------------------|-----------------------|
| 查看运行的容器          | `docker ps`      | `ctr task ls` / `ctr container ls` | `crictl ps`           |
| 查看镜像                | `docker images`  | `ctr image ls`               | `crictl images`       |
| 查看容器日志            | `docker logs`    | 无                           | `crictl logs`         |
| 查看容器数据信息        | `docker inspect` | `ctr container info`         | `crictl inspect`      |
| 查看容器资源            | `docker stats`   | 无                           | `crictl stats`        |
| 启动/关闭已有的容器     | `docker start/stop` | `ctr task start/kill`     | `crictl start/stop`   |
| 运行一个新的容器        | `docker run`     | `ctr run`                    | 无（最小单元为 pod） |
| 修改镜像标签            | `docker tag`     | `ctr image tag`              | 无                    |
| 创建一个新的容器        | `docker create`  | `ctr container create`       | `crictl create`       |
| 导入镜像                | `docker load`    | `ctr image import`           | 无                    |
| 导出镜像                | `docker save`    | `ctr image export`           | 无                    |
| 删除容器                | `docker rm`      | `ctr container rm`           | `crictl rm`           |
| 删除镜像                | `docker rmi`     | `ctr image rm`               | `crictl rmi`          |
| 拉取镜像                | `docker pull`    | `ctr image pull`             | `crictl pull`         |
| 推送镜像                | `docker push`    | `ctr image push`             | 无                    |
| 在容器内部执行命令      | `docker exec`    | 无                           | `crictl exec`         |

