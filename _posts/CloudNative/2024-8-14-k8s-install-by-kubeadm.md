---
layout: post
title: 【Cloud Native】03-2 K8S 安装 -- kubeadm安装
categories: [CloudNative, K8S]
tags: []
---

## 0 说明

若无特殊说明，均使用各机器上的root用户

## 1. 【所有机器】基础配置

[同：minikube 中该步骤](https://yc913344706.github.io/posts/k8s-install-by-minikube/#1-%E9%85%8D%E7%BD%AE%E5%9F%BA%E7%A1%80%E7%8E%AF%E5%A2%83-root-%E7%94%A8%E6%88%B7)

## 2. 【所有机器】集群主机名配置

```bash
# set hostname
# ---------
hostnamectl set-hostname k8s-master-143
#hostnamectl set-hostname k8s-node-144
#hostnamectl set-hostname k8s-node-145

# set hosts
# ---------
cat >> /etc/hosts << EOF
192.168.31.143 k8s-master-143
192.168.31.144 k8s-node-144
192.168.31.145 k8s-node-145
EOF
```

## 3. 【所有机器】配置时钟同步

```bash
# set chrony
# ---------
apt install -y chrony
systemctl status chrony && date
```

## 4. 【所有机器】防火墙，iptables, swap

```bash

# set firewalld and iptables
# ---------
systemctl status firewalld iptables
iptables -nvL

# disable swap
# ---------
free -h
swapoff -a
vim /etc/fstab
# 注释掉swap
mount -a && free -h
```

## 5. 【所有机器】安装CRI: containerd

```bash
# install CRI: containerd
# ---------
apt install -y containerd
mkdir /etc/containerd && containerd config default > /etc/containerd/config.toml
vim /etc/containerd/config.toml
# sandbox_image = "registry.XXX.com/registry.k8s.io/pause:3.9"
#
# [plugins."io.containerd.grpc.v1.cri".registry]
#   [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
#        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
#          endpoint = ["https://hub.si.icu"]
#
# [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
#             SystemdCgroup = true

systemctl restart containerd.service && sleep 3 &&systemctl status containerd.service
systemctl enable containerd.service
```

## 6. 【所有机器】配置 ipvs, connectrack

```bash
# install ipvs
# ---------
apt install -y ipset ipvsadm conntrack
vim /etc/modules-load.d/ipvs.conf
# ip_vs
# ip_vs_rr
# ip_vs_wrr
# ip_vs_sh
# nf_conntrack
# br_netfilter
systemctl restart systemd-modules-load.service
lsmod | grep -e ip_vs -e nf_conntrack -e br_netfilter


# set sysctl
# ---------
cat > /etc/sysctl.d/k8s.conf << EOF 
net.bridge.bridge-nf-call-ip6tables = 1 
net.bridge.bridge-nf-call-iptables = 1 
net.ipv4.ip_forward = 1

net.bridge.bridge-nf-call-ip6tables = 1 
net.bridge.bridge-nf-call-iptables = 1 
net.ipv4.ip_forward = 1

fs.may_detach_mounts = 1
vm.overcommit_memory=1
net.ipv4.conf.all.route_localnet = 1

vm.panic_on_oom=0
fs.inotify.max_user_watches=89100
fs.file-max=52706963
fs.nr_open=52706963
net.netfilter.nf_conntrack_max=2310720

net.ipv4.tcp_keepalive_time = 600
net.ipv4.tcp_keepalive_probes = 3
net.ipv4.tcp_keepalive_intvl =15
net.ipv4.tcp_max_tw_buckets = 36000
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_max_orphans = 327680
net.ipv4.tcp_orphan_retries = 3
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 16768
net.ipv4.ip_conntrack_max = 65536
net.ipv4.tcp_timestamps = 0
net.core.somaxconn = 16768
EOF
sysctl -a |grep -e net.ipv4.ip_forward -e bridge-nf-call-ip
sysctl --system  #生效

reboot

lsmod | grep -e ip_vs -e nf_conntrack
sysctl -a |grep -e net.ipv4.ip_forward -e bridge-nf-call-ip
sysctl -a |grep -e net.ipv4.ip_forward -e bridge-nf-call-ip
```

## 7. 【所有机器】安装 k8s 组件

### 7.1 kubernetes 版本选择

Kubernetes 项目维护最近三个次要版本（如：1.30、1.29、 1.28）的发布分支。

- 版本发布: https://kubernetes.io/zh-cn/releases
- 所有版本支持时间: https://kubernetes.io/zh-cn/releases/patch-releases/#support-period

选择策略：
- 正式环境：最近3个支持版本的第2个；
- 开发、测试环境：可以考虑持续更新；

本次选择：1.29.6

> 选择时间：2024-08-12
> 版本发布时间：2024-06-11

但是，由于apt安装，采用 https://mirrors.aliyun.com/kubernetes/apt 源，最新只到 1.28.2，所以本次选择 1.28.2.

> refer doc:
>
> - [Ubuntu 下安装 kubectl, kubelet, kubeadm](https://www.cnblogs.com/FengZeng666/p/15502138.html)
> - [k8s.gcr.io、registry.k8s.io镜像下载失败解决方案](https://blog.csdn.net/weixin_42173770/article/details/126177889)
> - [kubernetes安装（国内网络+阿里云）](https://blog.csdn.net/telundusiji/article/details/114033799)

### 7.2 命令

```bash
# install kubernetes 组件
# ---------
apt update && apt install -y apt-transport-https
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add -

cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF

apt update
# 查看kubectl可用版本
apt-cache madison kubectl
apt install -y kubectl=1.28.2-00 kubeadm=1.28.2-00 kubelet=1.28.2-00

systemctl status kubelet

cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
# 确定配置文件为: /etc/default/kubelet
vim /etc/default/kubelet
#KUBELET_CGROUP_ARGS="--cgroup-driver=systemd"
#KUBE_PROXY_MODE="ipvs"
journalctl -fu kubelet

vim /lib/systemd/system/kubelet.service
#  6 After=containerd.service,network-online.target
#  7 Requires=containerd.service,network-online.target
#  8
#  9 [Service]
# 10 ExecStart=/usr/bin/kubelet \
# 11     --container-runtime=remote --runtime-request-timeout=15m --container-runtime-endpoint=unix:///run/containerd/containerd.sock
systemctl daemon-reload && systemctl restart kubelet.service
systemctl status kubelet
# 此步若有错误：failed to load Kubelet config file /var/lib/kubelet/config.yaml
# - 不用管，因为还没有init

# 配置 k8s CRI
# ---------
crictl config runtime-endpoint unix:///var/run/containerd/containerd.sock
```

## 8. 【所有机器】准备集群镜像

```bash
# 准备集群镜像
# ---------
kubeadm config images list
images=(
registry.k8s.io/kube-apiserver:v1.28.2
registry.k8s.io/kube-controller-manager:v1.28.2
registry.k8s.io/kube-scheduler:v1.28.2
registry.k8s.io/kube-proxy:v1.28.2
registry.k8s.io/pause:3.9
registry.k8s.io/etcd:3.5.9-0
registry.k8s.io/coredns/coredns:v1.10.1

flannel/flannel:v0.25.5
flannel/flannel-cni-plugin:v1.5.1-flannel1
)
# 请注意，使用 ctr，则需要下载到 k8s.io 这个 NS 中，因为 crictl 是 k8s 使用的
for imageName in ${images[@]};do ctr -n k8s.io image pull registry.xxx.com/${imageName};  ctr -n k8s.io image tag registry.xxx.com/${imageName} $imageName; ctr -n k8s.io image rm registry.xxx.com/${imageName}; done
```

## 8. 【master-143】初始化集群

### 8.1 kubeadm init 参数

> refer doc:
> - [官网参数说明](https://kubernetes.io/zh-cn/docs/reference/setup-tools/kubeadm/kubeadm-init)


常见参数：

| 参数 | 说明 |
| --- | --- |
| --kubernetes-version | 为控制平面选择一个特定的 Kubernetes 版本。**默认**值："stable-1" |
| --pod-network-cidr | 指明 Pod 网络可以使用的 IP 地址段。<br/> 如果设置了这个参数，控制平面将会为每一个节点自动分配 CIDR。 |
| --service-cidr | 为服务的虚拟 IP 地址另外指定 IP 地址段。**默认**值："10.96.0.0/12" |
| --apiserver-advertise-address | API 服务器所公布的其正在监听的 IP 地址。<br/> 如果未设置，则使用**默认**网络接口。 |
| -v | 指定 kubeadm 的日志级别。<br/> **默认**值：0. [更多](https://docs.openshift.com/container-platform/4.15/rest_api/editing-kubelet-log-level-verbosity.html#log-verbosity-descriptions_editing-kubelet-log-level-verbosity) |

### 8.2 kubeadm init 命令

- k8s-master-143, 执行以下命令

```bash

# 初始化集群
# ---------

kubeadm reset
kubeadm init --v 4   --kubernetes-version v1.28.2   --pod-network-cidr=10.244.0.0/16   --apiserver-advertise-address=192.168.31.143
# 记录 join 命令
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# 查看Node
# ---------
kubectl get node

```

## 9. 【node节点】加入集群

- k8s-node-144，k8s-node-145，均执行以下命令

```bash
# join 集群
# ---------
kubeadm join 192.168.31.143:6443 --token XXXXXXXXXXXXXX \
    --discovery-token-ca-cert-hash sha256:XXXXXXXXXXXXX

# 查看Node
# ---------
kubectl get node
```

## 10. 【master-143】安装CNI: flannel

flannel 版本选择：
- https://github.com/flannel-io/flannel/tree/master?tab=readme-ov-file#deploying-flannel-manually
- key words: `For Kubernetes v1.17+`

- k8s-master-143, 执行以下命令

```bash
# 安装CNI: flannel
# ---------

# flannel
# web: download: https://github.com/flannel-io/flannel/tree/master/Documentation/kube-flannel.yml
mkdir -p ~/YAMLS/k8s
cd ~/YAMLS/k8s
wget https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml

grep image kube-flannel.yml && crictl images
kubectl apply -f ./kube-flannel.yml
kubectl get pod -n kube-system | grep flannel

# 查看Node
# ---------
kubectl get node
```

## 11. 【客户端】使用 Lens 界面化管理

- [lens简单说明](https://blog.csdn.net/jiangyou0k/article/details/105784868)

```bash
# 添加Lens
# ---------
kubectl get node
cat $HOME/.kube/config
```