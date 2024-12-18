---
layout: post
title: 【Cloud Native】03-3 K8S 安装 -- 二进制包安装
categories: [CloudNative, K8S]
tags: []
---

## 0 说明

若无特殊说明，均使用各机器上的root用户

> refer doc:
> - [保姆级二进制安装高可用k8s集群文档(1.23.8)](https://developer.aliyun.com/article/1273696)
> - [k8s二进制安装部署(详细)](https://blog.csdn.net/qq_44078641/article/details/120049473)

考虑到有些朋友电脑配置较低,一次性开四台虚拟机电脑跑不动, 所以搭建这套k8s高可用集群分两部分实施:
- 先部署一套单Master架构(三台),  
- 再扩容为多Master架构(4台或6台),  顺便再熟悉下Master扩容流程。

## 1. 【所有机器】基础配置

[同：minikube 中该步骤](https://yc913344706.github.io/posts/k8s-install-by-minikube)

## 2. 【所有机器】集群主机名配置

```bash
# set hostname
# ---------
hostnamectl set-hostname k8s-master-143
#hostnamectl set-hostname k8s-master-144
#hostnamectl set-hostname k8s-master-145

# set hosts
# ---------
cat >> /etc/hosts << EOF
192.168.31.143 k8s-master-143
192.168.31.144 k8s-master-144
192.168.31.145 k8s-master-145
EOF
```

## 3. 【所有机器】配置时钟同步

[同：kubeadm 中该步骤](https://yc913344706.github.io/posts/k8s-install-by-kubeadm)

## 4. 【所有机器】防火墙，iptables, swap

[同：kubeadm 中该步骤](https://yc913344706.github.io/posts/k8s-install-by-kubeadm)

## 5. 【所有机器】安装CRI: containerd

[同：kubeadm 中该步骤](https://yc913344706.github.io/posts/k8s-install-by-kubeadm)

## 6. 【所有机器】配置 ipvs, connectrack

[同：kubeadm 中该步骤](https://yc913344706.github.io/posts/k8s-install-by-kubeadm)

## 7. 安装etcd

Etcd 是一个分布式键值存储系统， Kubernetes使用Etcd进行数据存储，所以先准备一个Etcd数据库。

为解决Etcd单点故障，应采用集群方式部署，这里使用3台组建集群，可容忍1台机器故障，你也 可以使用5台组建集群，可容忍2台机器故障。

当集群较大时建议换成SSD加快etcd的访问速度.

> etcd 复用集群现有机器，部署在 192.168.31.143-192.168.31.145。
> 生产环境建议分开部署.

### 7.1 【master-143】etc安装：准备证书

找任意一台服务器操作，这里用 k8s-master-143 节点。

```bash
# install cfssl
# ---------
wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64 && \
    wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64 && \
    wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64

chmod +x cfssl_linux-amd64 cfssljson_linux-amd64 cfssl-certinfo_linux-amd64 

mv cfssl_linux-amd64 /usr/local/bin/cfssl && \
    mv cfssljson_linux-amd64 /usr/local/bin/cfssljson && \
    mv cfssl-certinfo_linux-amd64 /usr/bin/cfssl-certinfo

# prepare CA dir
# ---------
mkdir -p ~/TLS/{etcd,k8s} 
cd ~/TLS/etcd

# 准备 CA 配置文件
# ---------
# 证书过期时间自己可以改，这里默认是10年.
cat > ca-config.json << EOF 
{
    "signing":{
        "default":{
            "expiry":"87600h"
        },
        "profiles":{
            "www":{
                "expiry":"87600h",
                "usages":[
                    "signing",
                    "key encipherment",
                    "server auth",
                    "client auth"
                ]
            }
        }
    }
}
EOF

# 创建CA根证书申请文件
# ---------
cat > ca-csr.json << EOF 
{
    "CN":"etcd CA",
    "key":{
        "algo":"rsa",
        "size":2048
    },
    "names":[
        {
            "C":"CN",
            "L":"Beijing",
            "ST":"Beijing"
        }
    ]
}
EOF

# 生成CA根证书
# ---------
cfssl gencert -initca ca-csr.json | cfssljson -bare ca -
ls *pem
#ca-key.pem  ca.pem

# 创建SSL证书申请文件
# ---------
# 下面文件hosts字段中IP为所有etcd节点的集群内部通信IP，一个都不能少！为了方便后期扩容，可以多写几个预留的IP。
cat > server-csr.json << EOF
{
    "CN":"etcd",
    "hosts":[
        "192.168.31.143",
        "192.168.31.144",
        "192.168.31.145",
        "192.168.31.147",
        "192.168.31.148"
    ],
    "key":{
        "algo":"rsa",
        "size":2048
    },
    "names":[
        {
            "C":"CN",
            "L":"BeiJing",
            "ST":"BeiJing"
        }
    ]
}
EOF

# 生成SSL证书
# ---------
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=www server-csr.json | cfssljson -bare server
ls *server*pem
#server-key.pem  server.pem
```

### 7.2 【master-143】etc安装：安装主节点

#### 7.2.1 kubernetes 版本选择

[如何选择，参考 kubeadm 中，7.1 kubernetes 版本选择](https://yc913344706.github.io/posts/k8s-install-by-kubeadm)

本次选择: v1.29.7

> 选择时间：2024-08-16
> 版本发布时间：2024-07-16

#### 7.2.2 etcd 版本选择

etcd版本选择：

- [查看最新kubernetes版本对应的etcd版本](https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/configure-upgrade-etcd/)。
- [查看 v1.29 kubernetes版本对应的etcd版本](https://v1-29.docs.kubernetes.io/zh-cn/docs/tasks/administer-cluster/configure-upgrade-etcd/)。

> 可知：在生产环境中运行的 etcd 最低推荐版本为 3.4.22+ 和 3.5.6+。

本次选择：3.5.14

> 选择时间：2024-08-16
> 版本发布时间：2024-05-30

#### 7.2.3 命令

```bash
# 下载etcd
# ---------
cd ~ && wget https://github.com/etcd-io/etcd/releases/download/v3.5.14/etcd-v3.5.14-linux-amd64.tar.gz

# 创建工作目录并解压二进制包
# ---------
mkdir /opt/etcd/{bin,cfg,ssl} -p 
tar zxvf etcd-v3.5.14-linux-amd64.tar.gz 
mv etcd-v3.5.14-linux-amd64/{etcd,etcdctl} /opt/etcd/bin/

# 配置etcd
# ---------
cat > /opt/etcd/cfg/etcd.conf << EOF
#[Member]
ETCD_NAME="etcd-1"
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="https://192.168.31.143:2380"
ETCD_LISTEN_CLIENT_URLS="https://192.168.31.143:2379"

#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://192.168.31.143:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://192.168.31.143:2379"
ETCD_INITIAL_CLUSTER="etcd-1=https://192.168.31.143:2380,etcd-2=https://192.168.31.144:2380,etcd-3=https://192.168.31.145:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"
EOF
# ETCD_NAME：节点名称，集群中唯一
# ETCD_DATA_DIR：数据目录
# ETCD_LISTEN_PEER_URLS：集群通信监听地址
# ETCD_LISTEN_CLIENT_URLS：客户端访问监听地址
# ETCD_INITIAL_ADVERTISE_PEER_URLS：集群通告地址
# ETCD_ADVERTISE_CLIENT_URLS：客户端通告地址
# ETCD_INITIAL_CLUSTER：集群节点地址
# ETCD_INITIAL_CLUSTER_TOKEN：集群Token
# ETCD_INITIAL_CLUSTER_STATE：加入集群的当前状态，new是新集群，existing表示加入已有集群

# systemd管理etcd
# ---------
cat > /usr/lib/systemd/system/etcd.service << EOF
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
Type=notify
EnvironmentFile=/opt/etcd/cfg/etcd.conf
ExecStart=/opt/etcd/bin/etcd --cert-file=/opt/etcd/ssl/server.pem --key-file=/opt/etcd/ssl/server-key.pem --peer-cert-file=/opt/etcd/ssl/server.pem --peer-key-file=/opt/etcd/ssl/server-key.pem --trusted-ca-file=/opt/etcd/ssl/ca.pem --peer-trusted-ca-file=/opt/etcd/ssl/ca.pem
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF

# 拷贝刚才生成的证书
# ---------
cp ~/TLS/etcd/ca*pem ~/TLS/etcd/server*pem /opt/etcd/ssl/

# 启动并设置开机启动
# ---------
systemctl daemon-reload
systemctl enable --now etcd.service # 立即启动并开机自启动
# 注意：此时由于是先启动的主节点，其余节点尚未启动，所以status是failed，暂时不用管，等其余2台启动后，则会恢复正常
systemctl status etcd

# 将配置和证书copy到其他etcd节点
# ---------
scp -r /opt/etcd/ root@192.168.31.144:/opt/ && \
    scp /usr/lib/systemd/system/etcd.service root@192.168.31.144:/usr/lib/systemd/system/ && \
    scp -r /opt/etcd/ root@192.168.31.145:/opt/ && \
    scp /usr/lib/systemd/system/etcd.service root@192.168.31.145:/usr/lib/systemd/system/
```

### 7.3 【2个node节点】etc安装：安装从节点

- 安装 etcd：安装 etcd: 2个从节点

```bash
# 其他节点etcd配置需要修改的地方
# ---------
vim /opt/etcd/cfg/etcd.conf
# ETCD_NAME="etcd-1" # 修改此处，节点2改为etcd-2，节点3改为etcd-3
# ETCD_LISTEN_PEER_URLS="https://192.168.31.143:2380" # 修改此处为当前服务器IP
# ETCD_LISTEN_CLIENT_URLS="https://192.168.31.143:2379" # 修改此处为当前服务器IP
# ETCD_INITIAL_ADVERTISE_PEER_URLS="https://192.168.31.143:2380" # 修改此处为当前服务
# ETCD_ADVERTISE_CLIENT_URLS="https://192.168.31.143:2379" # 修改此处为当前服务器IP

# 启动并设置开机启动
# ---------
systemctl daemon-reload
systemctl enable --now etcd.service # 立即启动并开机自启动
systemctl status etcd
```

### 7.4 【master-143】etc安装：验证

```bash
# endpoint health
ETCDCTL_API=3 /opt/etcd/bin/etcdctl --cacert=/opt/etcd/ssl/ca.pem --cert=/opt/etcd/ssl/server.pem --key=/opt/etcd/ssl/server-key.pem --endpoints="https://192.168.31.143:2379,https://192.168.31.144:2379,https://192.168.31.145:2379" endpoint health --write-out=table

# member list
ETCDCTL_API=3 /opt/etcd/bin/etcdctl --cacert=/opt/etcd/ssl/ca.pem --cert=/opt/etcd/ssl/server.pem --key=/opt/etcd/ssl/server-key.pem --endpoints="https://192.168.31.143:2379,https://192.168.31.144:2379,https://192.168.31.145:2379" member list --write-out=table
```

## 8 【master-143】部署master

### 8.1 生成 kube-apiserver 证书

```bash
cd ~/TLS/k8s

# 准备 CA 配置文件
# ---------
# - 证书过期时间自己可以改，这里默认是10年.
cat > /root/TLS/k8s/ca-config.json << EOF
{
    "signing":{
        "default":{
            "expiry":"87600h"
        },
        "profiles":{
            "kubernetes":{
                "expiry":"87600h",
                "usages":[
                    "signing",
                    "key encipherment",
                    "server auth",
                    "client auth"
                ]
            }
        }
    }
}
EOF

# 创建CA根证书申请文件
# ---------
cat >/root/TLS/k8s/ca-csr.json << EOF
{
    "CN":"kubernetes",
    "key":{
        "algo":"rsa",
        "size":2048
    },
    "names":[
        {
            "C":"CN",
            "L":"Beijing",
            "ST":"Beijing",
            "O":"k8s",
            "OU":"System"
        }
    ]
}
EOF

# 生成CA根证书
# ---------
cfssl gencert -initca ca-csr.json | cfssljson -bare ca -
ls *pem
# ca-key.pem  ca.pem

# 创建证书请求文件
# ---------
cat > kube-apiserver-csr.json << EOF
{
  "CN": "system:kube-apiserver",
  "hosts":[ 
    "10.96.0.1",
    "127.0.0.1",
    "kubernetes",
    "kubernetes.default",
    "kubernetes.default.svc",
    "kubernetes.default.svc.cluster",
    "kubernetes.default.svc.cluster.local",
    "192.168.31.143",
    "192.168.31.144",
    "192.168.31.145",
    "192.168.31.147",
    "192.168.31.148"
  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "L": "BeiJing", 
      "ST": "BeiJing",
      "O": "system:masters",
      "OU": "System"
    }
  ]
}
EOF

# 生成证书
# ---------
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-apiserver-csr.json | cfssljson -bare kube-apiserver
ls kube-apiserver*pem
# kube-apiserver-key.pem  kube-apiserver.pem

mkdir -p /opt/kubernetes/{bin,cfg,ssl,logs}
cp ~/TLS/k8s/ca*pem ~/TLS/k8s/kube-apiserver*pem /opt/kubernetes/ssl
```

### 8.2 解压二进制包

```bash
# 下载二进制包
# https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.29.md#v1297

cd ~
wget https://dl.k8s.io/v1.29.7/kubernetes-server-linux-amd64.tar.gz
tar zxvf kubernetes-server-linux-amd64.tar.gz
cd kubernetes/server/bin
cp kube-apiserver kube-scheduler kube-controller-manager /opt/kubernetes/bin
cp kubectl /usr/bin/
```

### 8.3 部署 kube-apiserver

doc:
= 配置文件详情: https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/

```bash
# 创建配置文件
# 注意：
# - --insecure-port: 默认禁用已弃用的。从 v1.20 开始，该端口将被永久禁用，并在以后的版本中删除该标志。
# - --enable-swagger-ui: kubernetes v1.24版本以后弃用了 --enable-swagger-ui，所以在kube-apiserver配置文件中删除即可。
cat > /opt/kubernetes/cfg/kube-apiserver.conf << EOF
KUBE_APISERVER_OPTS="--enable-admission-plugins=NamespaceLifecycle,NodeRestriction,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota \\
  --anonymous-auth=false \\
  --bind-address=192.168.31.143 \\
  --secure-port=6443 \\
  --advertise-address=192.168.31.143 \\
  --authorization-mode=Node,RBAC \\
  --runtime-config=api/all=true \\
  --enable-bootstrap-token-auth \\
  --service-cluster-ip-range=10.96.0.0/16 \\
  --token-auth-file=/opt/kubernetes/cfg/token.csv \\
  --service-node-port-range=30000-32767 \\
  --tls-cert-file=/opt/kubernetes/ssl/kube-apiserver.pem  \\
  --tls-private-key-file=/opt/kubernetes/ssl/kube-apiserver-key.pem \\
  --client-ca-file=/opt/kubernetes/ssl/ca.pem \\
  --kubelet-client-certificate=/opt/kubernetes/ssl/kube-apiserver.pem \\
  --kubelet-client-key=/opt/kubernetes/ssl/kube-apiserver-key.pem \\
  --service-account-key-file=/opt/kubernetes/ssl/ca-key.pem \\
  --service-account-signing-key-file=/opt/kubernetes/ssl/ca-key.pem  \\
  --service-account-issuer=api \\
  --etcd-cafile=/opt/etcd/ssl/ca.pem \\
  --etcd-certfile=/opt/etcd/ssl/server.pem \\
  --etcd-keyfile=/opt/etcd/ssl/server-key.pem \\
  --etcd-servers=https://192.168.31.143:2379,https://192.168.31.144:2379,https://192.168.31.145:2379 \\
  --allow-privileged=true \\
  --apiserver-count=3 \\
  --audit-log-maxage=30 \\
  --audit-log-maxbackup=3 \\
  --audit-log-maxsize=100 \\
  --audit-log-path=/var/log/kube-apiserver-audit.log \\
  --event-ttl=1h \\
  --v=4"
EOF

# 生成token
head -c 16 /dev/urandom | od -An -t x | tr -d ' '

# 创建token文件（使用生成的token）
cat > /opt/kubernetes/cfg/token.csv << EOF
XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX,kubelet-bootstrap,10001,"system:node-bootstrapper"
EOF

# systemd管理kube-apiserver
cat > /etc/systemd/system/kube-apiserver.service << EOF
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes
After=etcd.service
Wants=etcd.service
[Service]
EnvironmentFile=-/opt/kubernetes/cfg/kube-apiserver.conf
ExecStart=/opt/kubernetes/bin/kube-apiserver \$KUBE_APISERVER_OPTS
Restart=on-failure
RestartSec=5
Type=notify
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target
EOF

# 开机自启动
systemctl daemon-reload
systemctl enable kube-apiserver --now
systemctl status kube-apiserver
```

#### 8.3.1 关键技术

TLS Bootstrap:

- Master apiserver启用TLS认证后， Node节点kubelet和kube-proxy要与kube- apiserver进行通信，必须使用CA签发的有效证书才可以。
- 当Node节点很多时，这种客户端证书颁发需 要大量工作，同样也会增加集群扩展复杂度。为了简化流程， Kubernetes引入了TLS bootstraping机制 来自动颁发客户端证书。
- kubelet会以一个低权限用户自动向apiserver申请证书， kubelet的证书由 apiserver动态签署。所以强烈建议在Node上使用这种方式，目前主要用于kubelet， kube-proxy还是 由我们统一颁发一个证书。


- [官网 关于 TLS Bootstrap](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/kubelet-tls-bootstrapping/)
- [关于 TLS Bootstrap](https://jimmysong.io/kubernetes-handbook/guide/tls-bootstrapping.html)
- [k8s TLS bootstrap解析-k8s TLS bootstrap流程分析](https://blog.csdn.net/kyle18826138721/article/details/123949710)

### 8.4 部署 kube-controller-manager


#### 8.4.1 生成 kube-controller-manager 证书

```bash
cd ~/TLS/k8s

# 创建证书请求文件
# ---------
cat > kube-controller-manager-csr.json << EOF
{
  "CN": "system:kube-controller-manager",
  "hosts":[],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "L": "BeiJing", 
      "ST": "BeiJing",
      "O": "system:masters",
      "OU": "System"
    }
  ]
}
EOF

# 生成证书
# ---------
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager
ls kube-controller-manager*pem
# kube-controller-manager-key.pem  kube-controller-manager.pem

cp ~/TLS/k8s/kube-controller-manager*pem /opt/kubernetes/ssl
```

#### 8.4.2 创建配置文件

doc:
- 配置文件详情: https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/

```bash
# - kube-controller-manager 的弃用--experimental-cluster-signing-duration标志现已删除。调整您的机器以使用--cluster-signing-duration自 v1.19 以来可用的标志。
cat > /opt/kubernetes/cfg/kube-controller-manager.conf << EOF
KUBE_CONTROLLER_MANAGER_OPTS="--bind-address=127.0.0.1 \\
  --kubeconfig=/opt/kubernetes/cfg/kube-controller-manager.kubeconfig \\
  --service-cluster-ip-range=10.96.0.0/16 \\
  --cluster-name=kubernetes \\
  --cluster-signing-cert-file=/opt/kubernetes/ssl/ca.pem \\
  --cluster-signing-key-file=/opt/kubernetes/ssl/ca-key.pem \\
  --allocate-node-cidrs=true \\
  --cluster-cidr=10.244.0.0/16 \\
  --cluster-signing-duration=87600h \\
  --root-ca-file=/opt/kubernetes/ssl/ca.pem \\
  --service-account-private-key-file=/opt/kubernetes/ssl/ca-key.pem \\
  --leader-elect=true \\
  --feature-gates=RotateKubeletServerCertificate=true \\
  --controllers=*,bootstrapsigner,tokencleaner \\
  --tls-cert-file=/opt/kubernetes/ssl/kube-controller-manager.pem \\
  --tls-private-key-file=/opt/kubernetes/ssl/kube-controller-manager-key.pem \\
  --use-service-account-credentials=true \\
  --v=2"
EOF

KUBE_CONFIG="/opt/kubernetes/cfg/kube-controller-manager.kubeconfig"
KUBE_APISERVER="https://192.168.31.143:6443"

kubectl config set-cluster kubernetes \
  --certificate-authority=/opt/kubernetes/ssl/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER} \
  --kubeconfig=${KUBE_CONFIG}
  
kubectl config set-credentials kube-controller-manager \
  --client-certificate=/opt/kubernetes/ssl/kube-controller-manager.pem \
  --client-key=/opt/kubernetes/ssl/kube-controller-manager-key.pem \
  --embed-certs=true \
  --kubeconfig=${KUBE_CONFIG}
  
kubectl config set-context default \
  --cluster=kubernetes \
  --user=kube-controller-manager \
  --kubeconfig=${KUBE_CONFIG}
  
kubectl config use-context default --kubeconfig=${KUBE_CONFIG}
```

#### 8.4.3 部署 kube-controller-manager

```bash
# systemd 管理kube-controler-manager
cat > /usr/lib/systemd/system/kube-controller-manager.service << EOF
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes
[Service]
EnvironmentFile=-/opt/kubernetes/cfg/kube-controller-manager.conf
ExecStart=/opt/kubernetes/bin/kube-controller-manager \$KUBE_CONTROLLER_MANAGER_OPTS
Restart=on-failure
RestartSec=5
[Install]
WantedBy=multi-user.target
EOF

# 启动并设置开机启动
systemctl daemon-reload
systemctl enable kube-controller-manager --now
systemctl status kube-controller-manager
```

### 8.4 部署 kube-scheduler


#### 8.4.1 生成 kube-scheduler 证书

```bash
cd ~/TLS/k8s

# 创建证书请求文件
# ---------
cat > kube-scheduler-csr.json << EOF
{
  "CN": "system:kube-scheduler",
  "hosts":[],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "L": "BeiJing", 
      "ST": "BeiJing",
      "O": "system:masters",
      "OU": "System"
    }
  ]
}
EOF

# 生成证书
# ---------
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-scheduler-csr.json | cfssljson -bare kube-scheduler
ls kube-scheduler*pem
# kube-scheduler-key.pem  kube-scheduler.pem

cp ~/TLS/k8s/kube-scheduler*pem /opt/kubernetes/ssl
```

#### 8.4.2 创建配置文件

```bash
# 配置文件详情: https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/
cat > /opt/kubernetes/cfg/kube-scheduler.conf << EOF
KUBE_SCHEDULER_OPTS="--bind-address=127.0.0.1 \\
--kubeconfig=/opt/kubernetes/cfg/kube-scheduler.kubeconfig \\
--leader-elect=true \\
--v=2"
EOF

KUBE_CONFIG="/opt/kubernetes/cfg/kube-scheduler.kubeconfig"
KUBE_APISERVER="https://192.168.31.143:6443"

kubectl config set-cluster kubernetes \
  --certificate-authority=/opt/kubernetes/ssl/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER} \
  --kubeconfig=${KUBE_CONFIG}
  
kubectl config set-credentials kube-scheduler \
  --client-certificate=/opt/kubernetes/ssl/kube-scheduler.pem \
  --client-key=/opt/kubernetes/ssl/kube-scheduler-key.pem \
  --embed-certs=true \
  --kubeconfig=${KUBE_CONFIG}
  
kubectl config set-context default \
  --cluster=kubernetes \
  --user=kube-scheduler \
  --kubeconfig=${KUBE_CONFIG}
  
kubectl config use-context default --kubeconfig=${KUBE_CONFIG}
```

#### 8.4.3 部署 kube-scheduler

```bash
# systemd管理kube-scheduler
cat > /usr/lib/systemd/system/kube-scheduler.service << EOF
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes
[Service]
EnvironmentFile=-/opt/kubernetes/cfg/kube-scheduler.conf
ExecStart=/opt/kubernetes/bin/kube-scheduler \$KUBE_SCHEDULER_OPTS
Restart=on-failure
RestartSec=5
[Install]
WantedBy=multi-user.target
EOF

# 启动并设置开机启动
systemctl daemon-reload
systemctl enable kube-scheduler --now
systemctl status kube-scheduler
```

### 8.5 查看集群状态

#### 8.5.1 生成 kubectl 连接集群的证书

```bash
cd ~/TLS/k8s

cat > admin-csr.json <<EOF
{
  "CN": "admin",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "L": "BeiJing",
      "ST": "BeiJing",
      "O": "system:masters",
      "OU": "System"
    }
  ]
}
EOF

cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes admin-csr.json | cfssljson -bare admin
ls admin*pem
# admin-key.pem  admin.pem

cp ~/TLS/k8s/admin*pem /opt/kubernetes/ssl
```

#### 8.5.2 生成kubeconfig文件

```bash
mkdir /root/.kube

KUBE_CONFIG="/root/.kube/config"
KUBE_APISERVER="https://192.168.31.143:6443"

kubectl config set-cluster kubernetes \
  --certificate-authority=/opt/kubernetes/ssl/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER} \
  --kubeconfig=${KUBE_CONFIG}
  
kubectl config set-credentials cluster-admin \
  --client-certificate=/opt/kubernetes/ssl/admin.pem \
  --client-key=/opt/kubernetes/ssl/admin-key.pem \
  --embed-certs=true \
  --kubeconfig=${KUBE_CONFIG}
  
kubectl config set-context default \
  --cluster=kubernetes \
  --user=cluster-admin \
  --kubeconfig=${KUBE_CONFIG}
  
kubectl config use-context default --kubeconfig=${KUBE_CONFIG}
```

#### 8.5.3 查看集群状态

```bash
kubectl get cs

kubectl get --raw='/readyz?verbose'

# 授权kubelet-bootstrap用户允许请求证书
kubectl create clusterrolebinding kubelet-bootstrap \
--clusterrole=system:node-bootstrapper \
--user=kubelet-bootstrap
```

## 9 【master-143】部署 Worker

既当 Master 节点,也当 Work Node 节点

### 9.1 拷贝文件

```bash
# 拷贝kubelet, kube-proxy
cd ~/kubernetes/server/bin/
cp kubelet kube-proxy /opt/kubernetes/bin/
```

### 9.2 安装kubelet

#### 9.2.1 创建配置文件

文档：
- https://kubernetes.io/zh-cn/docs/reference/command-line-tools-reference/kubelet/

```bash
# 创建配置文件
# --hostname-override ：显示名称,集群唯一(不可重复)。

cat > /opt/kubernetes/cfg/kubelet.conf << EOF
KUBELET_OPTS="--v=2 \\
--hostname-override=k8s-master-143 \\
--kubeconfig=/opt/kubernetes/cfg/kubelet.kubeconfig \\
--bootstrap-kubeconfig=/opt/kubernetes/cfg/bootstrap.kubeconfig \\
--config=/opt/kubernetes/cfg/kubelet-config.yml \\
--cert-dir=/opt/kubernetes/ssl \\
--pod-infra-container-image=registry.xxx.com/registry.k8s.io/pause:3.9" \\
--serverTLSBootstrap
EOF

# --network-plugin ：启用CNI。
# --kubeconfig ： 空路径,会自动生成,后面用于连接apiserver。
# --bootstrap-kubeconfig ：首次启动向apiserver申请证书。
# --config ：配置文件参数。
# --cert-dir ：kubelet证书目录。
# --pod-infra-container-image ：管理Pod网络容器的镜像 init container

# 创建配置参数文件
cat > /opt/kubernetes/cfg/kubelet-config.yml << EOF
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
address: 0.0.0.0
port: 10250
readOnlyPort: 10255
cgroupDriver: systemd
clusterDNS:
- 10.96.0.2
clusterDomain: cluster.local 
failSwapOn: false
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 2m0s
    enabled: true
  x509:
    clientCAFile: /opt/kubernetes/ssl/ca.pem 
authorization:
  mode: Webhook
  webhook:
    cacheAuthorizedTTL: 5m0s
    cacheUnauthorizedTTL: 30s
evictionHard:
  imagefs.available: 15%
  memory.available: 100Mi
  nodefs.available: 10%
  nodefs.inodesFree: 5%
maxOpenFiles: 1000000
maxPods: 110
serverTLSBootstrap: true
EOF

# 生成kubelet初次加入集群引导kubeconfig文件
KUBE_CONFIG="/opt/kubernetes/cfg/bootstrap.kubeconfig"
KUBE_APISERVER="https://192.168.31.143:6443" # apiserver IP:PORT
cat /opt/kubernetes/cfg/token.csv
TOKEN="XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX" # 与token.csv里保持一致  /opt/kubernetes/cfg/token.csv 

# 生成 kubelet bootstrap kubeconfig 配置文件
kubectl config set-cluster kubernetes \
  --certificate-authority=/opt/kubernetes/ssl/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER} \
  --kubeconfig=${KUBE_CONFIG}
  
kubectl config set-credentials "kubelet-bootstrap" \
  --token=${TOKEN} \
  --kubeconfig=${KUBE_CONFIG}

kubectl config set-context default \
  --cluster=kubernetes \
  --user="kubelet-bootstrap" \
  --kubeconfig=${KUBE_CONFIG}

kubectl config use-context default --kubeconfig=${KUBE_CONFIG}
```

#### 9.2.2 部署kubelet

```bash
# systemd管理kubelet
cat > /usr/lib/systemd/system/kubelet.service << EOF
[Unit]
Description=Kubernetes Kubelet
After=docker.service

[Service]
EnvironmentFile=/opt/kubernetes/cfg/kubelet.conf
ExecStart=/opt/kubernetes/bin/kubelet \$KUBELET_OPTS
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF

# 启动并设置开机启动
systemctl daemon-reload
systemctl enable kubelet --now
systemctl status kubelet
```

#### 9.2.3 【master-143】master 节点允许证书请求

```bash
#查看kubelet证书请求
kubectl get csr
# name: node-csr-KbHieprZUMOvTFMHGQ1RNTZEhsSlT5X6wsh2lzfUry4
# condition: Pending

#允许kubelet节点申请
kubectl certificate approve  node-csr-XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
# approved

#查看申请
kubectl get csr
# condition: Approved,Issued

#查看节点
kubectl get nodes
# k8s-master-143   NotReady   <none>   2m11s   v1.20.10
# 由于网络插件还没有部署,节点会没有准备就绪NotReady
```

### 9.3 安装kube-proxy


#### 9.3.1 创建 kube-proxy 证书

```bash
# 切换工作目录
cd ~/TLS/k8s

# 创建证书请求文件
cat > kube-proxy-csr.json << EOF
{
  "CN": "system:kube-proxy",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "L": "BeiJing",
      "ST": "BeiJing",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
EOF

# 生成证书
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-proxy-csr.json | cfssljson -bare kube-proxy
ls kube-proxy*pem
# kube-proxy-key.pem  kube-proxy.pem

cp ~/TLS/k8s/kube-proxy*pem /opt/kubernetes/ssl
```

#### 9.3.2 创建配置文件

doc:
- https://kubernetes.io/zh-cn/docs/reference/command-line-tools-reference/kube-proxy/

```bash
# 创建配置文件
cat > /opt/kubernetes/cfg/kube-proxy.conf << EOF
KUBE_PROXY_OPTS="--v=2 \\
--config=/opt/kubernetes/cfg/kube-proxy-config.yml"
EOF

# 创建配置参数文件
cat > /opt/kubernetes/cfg/kube-proxy-config.yml << EOF
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
bindAddress: 0.0.0.0
metricsBindAddress: 0.0.0.0:10249
clientConnection:
  kubeconfig: /opt/kubernetes/cfg/kube-proxy.kubeconfig
hostnameOverride: k8s-master-143
clusterCIDR: 10.244.0.0/16
EOF

# 生成 kube-proxy kubeconfig 配置文件
KUBE_CONFIG="/opt/kubernetes/cfg/kube-proxy.kubeconfig"
KUBE_APISERVER="https://192.168.31.143:6443"

kubectl config set-cluster kubernetes \
  --certificate-authority=/opt/kubernetes/ssl/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER} \
  --kubeconfig=${KUBE_CONFIG}

kubectl config set-credentials kube-proxy \
  --client-certificate=/opt/kubernetes/ssl/kube-proxy.pem \
  --client-key=/opt/kubernetes/ssl/kube-proxy-key.pem \
  --embed-certs=true \
  --kubeconfig=${KUBE_CONFIG}

kubectl config set-context default \
  --cluster=kubernetes \
  --user=kube-proxy \
  --kubeconfig=${KUBE_CONFIG}

kubectl config use-context default --kubeconfig=${KUBE_CONFIG}
```

#### 9.3.3 部署kube-proxy

```bash
cat > /usr/lib/systemd/system/kube-proxy.service << EOF
[Unit]
Description=Kubernetes Proxy
After=network.target

[Service]
EnvironmentFile=/opt/kubernetes/cfg/kube-proxy.conf
ExecStart=/opt/kubernetes/bin/kube-proxy \$KUBE_PROXY_OPTS
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF

# 启动并设置开机启动
systemctl daemon-reload
systemctl enable kube-proxy --now
systemctl status kube-proxy
```

## 10 剩余基础组件

### 10.1 【master-143】安装CNI: flannel

选择flannel，但注意：也有其他可选CNI插件。

[同：kubeadm 中该步骤](https://yc913344706.github.io/posts/k8s-install-by-kubeadm)

### 10.2 【所有master】部署CNI-plugin

参考文档：
- [青蛙小白](https://blog.frognew.com/2021/04/relearning-container-03.html)
- https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/migrating-from-dockershim/troubleshooting-cni-plugin-related-errors/


```bash
wget https://github.com/containernetworking/plugins/releases/download/v1.5.1/cni-plugins-linux-amd64-v1.5.1.tgz

mkdir -p /opt/cni/bin /etc/cni/net.d

tar -zxvf cni-plugins-linux-amd64-v1.5.1.tgz -C /opt/cni/bin/

# 重建 flannel pod
```

### 10.3 【master-143】授权apiserver访问kubelet

应用场景：如kubectl logs

```bash
cd ~/YAMLS/k8s/rbac

cat > apiserver-to-kubelet-rbac.yaml << EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:kube-apiserver-to-kubelet
rules:
  - apiGroups:
      - ""
    resources:
      - nodes/proxy
      - nodes/stats
      - nodes/log
      - nodes/spec
      - nodes/metrics
      - pods/log
    verbs:
      - "*"
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: system:kube-apiserver
  namespace: ""
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-apiserver-to-kubelet
subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: kubernetes
EOF

kubectl apply -f apiserver-to-kubelet-rbac.yaml
```

### 10.4 【master-143】部署Dashboard

我用Lens，暂不需要

### 10.5 【master-143】crictl

refer:
- https://github.com/kubernetes-sigs/cri-tools

```bash
VERSION="v1.29.0"
wget https://github.com/kubernetes-sigs/cri-tools/releases/download/$VERSION/crictl-$VERSION-linux-amd64.tar.gz
sudo tar zxvf crictl-$VERSION-linux-amd64.tar.gz -C /usr/local/bin
rm -f crictl-$VERSION-linux-amd64.tar.gz

crictl config runtime-endpoint unix:///var/run/containerd/containerd.sock
```

### 10.6 【master-143】Helm

版本选择：
- https://helm.sh/zh/docs/topics/version_skew/

本次选择：
- 由于 k8s 版本为 1.29.7，所以 Helm 选择 3.14.4 (20240811发布)

部署参考文档：
- helm3: https://www.zhaowenyu.com/helm-doc/install/helm3-install.html
- helm2（已弃用）: https://www.cnblogs.com/zhanglianghhh/p/14165995.html

> 注：Helm2 依赖 Tiller，而 Helm3 已经移除了 Tiller 的依赖，只需安装一个 helm 客户端即可。

```bash
cd ~/
wget https://get.helm.sh/helm-v3.14.4-linux-amd64.tar.gz
cp -a linux-amd64/helm /usr/bin/helm
helm version

# 查看 kubernetes 集群中已经通过 Chart 安装的 Release
helm ls

# helm3 管理集群默认依赖 ~/.kube/config 文件，如果非默认路径需要通过 --kubeconfig 指定路径。
# helm --kubeconfig /tmp/test11-kubeconfig.conf ls
```

### 10.7 【master-143】CoreDNS

coredns作用：
- 在集群内提供Service和Pod的域名解析服务。

coredns版本选择：
- https://kubernetes.io/docs/tasks/administer-cluster/coredns/
- key words: `Your Kubernetes server must be at or later than version v1.9.`


部署参考文档：
- https://cloud.tencent.com/developer/article/2383057

问题解决文档：
- https://bbs.huaweicloud.com/blogs/370311

```bash
cd ~/YAMLS/k8s

# 拷贝该Yaml内容：https://github.com/coredns/helm/blob/master/charts/coredns/values.yaml
# 保存为coredns.yml

# 修改:
# - 镜像地址
# - coredns 的 service 的 cluster ip: 10.96.0.2  # 与 kubelet 的配置文件一致

# 启动
helm --namespace=kube-system install coredns coredns/coredns -f ~/YAMLS/k8s/coredns.yaml

# 卸载
helm --namespace=kube-system uninstall coredns
```

### 10.8 【master-143】配置自动补全

```bash
apt install -y bash-completion

echo 'source <(kubectl completion bash)' >> ~/.bashrc
kubectl completion bash > /etc/bash_completion.d/kubectl
source /usr/share/bash-completion/bash_completion
```

### 10.9 安装 metrics-server

refer doc:
- [kubernetes集群二进制部署metrics-server](https://blog.csdn.net/jthello123/article/details/105468136)
- [k8s学习笔记-二进制高可用metrics-server安装](https://blog.csdn.net/weixin_30586257/article/details/95895556)
- [二进制部署Kubernetes安装metrics-server遇到的问题](https://blog.csdn.net/heian_99/article/details/115137827)
- [【K8S 七】Metrics Server部署中的问题](https://blog.csdn.net/avatar_2009/article/details/126016679?spm=1001.2014.3001.5502)

启用APIAggregator
- API Aggregation 允许在不修改 Kubernetes 核心代码的同时扩展 KubernetesAPI，
- 即: 将第三方服务注册到 Kubernetes API中，这样就可以通过 Kubernetes API来访问第三方服务了，Metrics Server API就是将自己的服务注册到 Kubernetes API中。

metric-server 版本选择：
- 根据[官方文档](https://github.com/kubernetes-sigs/metrics-server)，我们的集群版本，安装0.7.x即可
- 当前最新为0.7.2，安装它

#### 10.9.1 【master-143】生成证书

```bash
cd ~/TLS/k8s
cat > metrics-server-csr.json << EOF
{
  "CN": "system:metrics-server",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BeiJing",
      "O": "k8s",
      "OU": "system"
    }
  ]
}
EOF

# 生成证书
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes metrics-server-csr.json | cfssljson -bare metrics-server

# 将生成的证书拷贝至集群所有master节点
ALL_MASTER_NODES=(
  192.168.31.143
  192.168.31.144
  192.168.31.145
)

for i in "${ALL_MASTER_NODES[@]}"; do
  scp -r metrics-server*.pem root@$i:/opt/kubernetes/ssl/
done

# rbac授权
# -----------
cd ~/YAMLS/k8s/rbac
cat > auth-metrics-server.yaml << EOF
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: system:auth-metrics-server-reader
  labels:
    rbac.authorization.k8s.io/aggregate-to-view: "true"
    rbac.authorization.k8s.io/aggregate-to-edit: "true"
    rbac.authorization.k8s.io/aggregate-to-admin: "true"
rules:
- apiGroups: ["metrics.k8s.io"]
  resources: ["pods", "nodes"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: metrics-server:system:auth-metrics-server
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:auth-metrics-server-reader
subjects:
- kind: User
  name: system:metrics-server
  namespace: kube-system
EOF
kubectl apply -f auth-metrics-server.yaml
```

#### 10.9.2 【所有master节点】更改配置

```bash
# 修改配置文件：kube-apiserver
# 检查 API Server 是否开启了 Aggregator Routing：
# - 具体来说，就是查看API Server 是否具有--enable-aggregator-routing=true 选项
vim /opt/kubernetes/cfg/kube-apiserver.conf
--requestheader-client-ca-file=/opt/kubernetes/ssl/ca.pem \
--requestheader-allowed-names=aggregator \
--requestheader-extra-headers-prefix=X-Remote-Extra- \
--requestheader-group-headers=X-Remote-Group \
--requestheader-username-headers=X-Remote-User \
--proxy-client-cert-file=/opt/kubernetes/ssl/metrics-server.pem \
--proxy-client-key-file=/opt/kubernetes/ssl/metrics-server-key.pem \
--enable-aggregator-routing=true \

# 修改配置文件：kube-controller-manager
vim /opt/kubernetes/cfg/kube-controller-manager.conf
--horizontal-pod-autoscaler-use-rest-clients=true \

# 重启服务
systemctl daemon-reload
systemctl restart kube-controller-manager kube-apiserver
systemctl status kube-controller-manager kube-apiserver
```

#### 10.9.3 【master-143】部署 metrics-server

```bash
# 下载 yaml
mkdir ~/metric-server
cd ~/metric-server
wget https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.7.2/components.yaml

# 修改: image，imagePullPolicy
vim components.yaml

# 启动
kubectl apply -f components.yaml
```

#### 10.9.4 问题记录

问题：Failed to scrape node" err="Get \"https://xxx/metrics/resource\": x509: cannot validate certificate for xxx because it doesn't contain any IP SANs" node="xxx"

- https://www.cnblogs.com/ki11-9/articles/16659943.html
- https://www.tomking.xyz/posts/kubernetes-deploy-metrics-server-with-tls/

> 至此一个单Master的k8s节点就已经完成了

## 11 集群扩容：增加master节点

### 11.1 master节点高可用说明

- Kubernetes作为容器集群系统，通过健康检查+重启策略实现了Pod故障自我修复能力，
- 通过调度算法实现将Pod分布式部署，并保持预期副本数，
- 根据Node失效状态自动在其他Node拉起Pod，实现了应用层的高可用性。

针对Kubernetes集群，高可用性还应包含以下两个层面的考虑：
- Etcd数据库的高可用性
  - 而Etcd我们已经采用3个节点组建集群实现高可用
- Kubernetes Master组件的高可用性。
  - Master节点扮演着总控中心的角色，通过不断与工作节点上的Kubelet和kube-proxy进行通信来维护整个集群的健康工作状态。
  - 如果Master节点故障，将无法使用kubectl工具或者API做任何集群管理。
  - Master节点主要有三个服务kube-apiserver、kube-controller-manager和kube-scheduler，
    - 其中kube-controller-manager和kube-scheduler组件自身通过选择机制已经实现了高可用，
    - 所以Master高可用主要针对kube-apiserver组件，而该组件是以HTTP API提供服务
    - 因此对他高可用与Web服务器类似，增加负载均衡器对其负载均衡即可，并且可水平扩容。

本节将对Master节点高可用进行说明和实施。

### 11.2 【master-143】拷贝文件到新Master节点


```bash
KUBE_NEW_MASTERS=(
    192.168.31.144
    192.168.31.145
)

for i in "${KUBE_NEW_MASTERS[@]}"
do
  ssh-copy-id root@$i
  ssh root@$i "mkdir -p /opt/kubernetes/{bin,cfg,ssl,logs} /opt/etcd/ssl"

  scp -r /opt/kubernetes root@$i:/opt
  scp -r /opt/etcd/ssl root@$i:/opt/etcd
  scp /etc/systemd/system/kube-apiserver.service root@$i:/etc/systemd/system/
  scp /usr/lib/systemd/system/kube* root@$i:/usr/lib/systemd/system
  scp /usr/bin/kubectl  root@$i:/usr/bin
  scp -r ~/.kube root@$i:~

  ssh root@$i "rm -f /opt/kubernetes/cfg/kubelet.kubeconfig"
  ssh root@$i "rm -f /opt/kubernetes/ssl/kubelet*"
done
```

### 11.3 【master-02】 修改配置文件

```bash
vim /opt/kubernetes/cfg/kube-apiserver.conf 
# %s/31.143/31.145/gc
# --bind-address=192.168.31.144 \
# --advertise-address=192.168.31.144 \

vim /opt/kubernetes/cfg/kube-controller-manager.kubeconfig
# server: https://192.168.31.144:6443

vim /opt/kubernetes/cfg/kube-scheduler.kubeconfig
# server: https://192.168.31.144:6443

vim /opt/kubernetes/cfg/kubelet.conf
# --hostname-override=k8s-master2

vim /opt/kubernetes/cfg/kube-proxy-config.yml
# hostnameOverride: k8s-master2

vim ~/.kube/config
# server: https://192.168.31.144:6443
```

### 11.4 【master-02】 启动服务

```bash
systemctl daemon-reload
systemctl enable kube-apiserver kube-controller-manager kube-scheduler kubelet kube-proxy --now
systemctl status kube-apiserver kube-controller-manager kube-scheduler kubelet kube-proxy
```

### 11.5 【master-143】 同意证书请求

```bash
# 查看证书请求
kubectl get csr
#node-csr-EQoVFfTbo6DcvcWfaRzBbMst4BXmdyds99DEYk2oDDE   33m   kubernetes.io/kube-apiserver-client-kubelet   kubelet-bootstrap   Pending

# 同意请求
kubectl certificate approve node-csr-XXXXXXXXXXXXXXXXXXXXXXXXX
# certificatesigningrequest.certificates.k8s.io/node-csr-EQoVFfTbo6DcvcWfaRzBbMst4BXmdyds99DEYk2oDDE approved

# 查看Node
kubectl get nodes
# NAME          STATUS   ROLES    AGE     VERSION
# k8s-master1   Ready    <none>   6d23h   v1.20.10
# k8s-master2   Ready    <none>   9m11s   v1.20.10
# k8s-node1     Ready    <none>   5d      v1.20.10
# k8s-node2     Ready    <none>   5d      v1.20.10
```

### 11.6 【master-02】 验证

```bash
kubectl get cs

kubectl get --raw='/readyz?verbose'
```

### 11.7 【所有Master节点】安装 Nginx + Keepalived

refer doc:
- https://www.cnblogs.com/gkhost/p/17814956.html
- https://blog.csdn.net/weixin_44352521/article/details/126678485

```bash
# # 安装 nginx + keepalived
# apt install nginx keepalived -y
# 
# # 查看是否已经安装 --with-stream模块
# nginx -V
# # 如果没有，参考文档进行安装： https://cloud.tencent.com/developer/article/1521251

# 源码安装 nginx

apt update
apt install -y libgeoip1 libnginx-mod-http-geoip geoip-database \
  gzip gcc make wget \
  libmaxminddb0 libmaxminddb-dev mmdb-bin build-essential libpcre3 \
  libpcre3-dev zlib1g zlib1g-dev openssl libssl-dev libxml2-dev \
  libxslt-dev libgd-dev
cd /tmp/
apt remove nginx-* libnginx-*
rm -f /etc/systemd/system/nginx.service /usr/lib/systemd/system/nginx.service
which nginx
# 应该没有值

cd ~
wget https://nginx.org/download/nginx-1.26.2.tar.gz
tar -xf nginx-1.26.2.tar.gz 
cd nginx-1.26.2/
./configure \
  --prefix=/usr/local/nginx-1.26.2 \
  --pid-path=/var/run/nginx-1.26.2/nginx.pid \
  --lock-path=/var/lock/nginx-1.26.2.lock \
  --error-log-path=/var/log/nginx-1.26.2/error.log \
  --http-log-path=/var/log/nginx-1.26.2/access.log \
  --with-http_gzip_static_module \
  --http-client-body-temp-path=/var/temp/nginx-1.26.2/client \
  --http-proxy-temp-path=/var/temp/nginx-1.26.2/proxy \
  --http-fastcgi-temp-path=/var/temp/nginx-1.26.2/fastcgi \
  --http-uwsgi-temp-path=/var/temp/nginx-1.26.2/uwsgi \
  --http-scgi-temp-path=/var/temp/nginx-1.26.2/scgi \
  --with-stream
make && make install
ln -sf /usr/local/nginx-1.26.2/sbin/nginx /usr/sbin/nginx
mkdir -p /var/temp/nginx-1.26.2/client /var/run/nginx-1.26.2/

# 修改配置文件: nginx
vim /usr/local/nginx-1.26.2/conf/nginx.conf
# events {
#     ...
# }
#
# # 四层负载均衡，为两台Master apiserver组件提供负载均衡
# stream {
# 
#     log_format  main  '$remote_addr $upstream_addr - [$time_local] $status $upstream_bytes_sent';
# 
#     access_log  /var/log/nginx/k8s-access.log  main;
# 
#     upstream k8s-apiserver {
#        server 192.168.31.143:6443;   # Master1 APISERVER IP:PORT
#        server 192.168.31.144:6443;   # Master2 APISERVER IP:PORT
#        server 192.168.31.145:6443;   # Master2 APISERVER IP:PORT
#     }
#     
#     server {
#        listen 16443; # 由于nginx与master节点复用，这个监听端口不能是6443，否则会冲突
#        proxy_pass k8s-apiserver;
#     }
# }
#
# http {
#     ...
# }

cat > /usr/lib/systemd/system/nginx.service << EOF
[Unit]
Description=nginx
After=network.target

[Service]
Type=forking
ExecStart=/usr/sbin/nginx
ExecReload=/usr/sbin/nginx -s reload
ExecStop=/usr/sbin/nginx -s quit
PrivateTmp=true

[Install]
WantedBy=multi-user.target
EOF

# apt 安装 keepalived
apt install keepalived -y

# 修改配置文件: keepalived
cat > /etc/keepalived/keepalived.conf << EOF
global_defs { 
   router_id LVS_K8S   # 全局逻辑组表示，三节点统一标识
} 
vrrp_script check_nginx {
    script "/etc/keepalived/check_nginx.sh"
}

vrrp_instance VI_1 { 
    state MASTER         # change: 只有2种值：MASTER、BACKUP
    interface enp1s18    # change: 修改为实际网卡名
    virtual_router_id 51 # 按照业务端口，定义多个vrrp组，同一个id在一个组内，例如：主备节点统一101，为一组；
    priority 100         # change: 优先级越高越优先，Master节点优先级大于Backup节点值； 如：master: 100, backup: 90
    advert_int 1         # 指定VRRP 心跳包通告间隔时间，默认1秒 
    authentication {     # 授权类型和密钥字段
        auth_type PASS           # 认证方式：PASS或ha；
        auth_pass 1qaz@WSX3edc   # 认证密钥：三节点密钥字段一样；
    }  
    # 虚拟IP
    virtual_ipaddress {     # IPV4虚拟IP，三节点虚拟IP在同一个网段；
        192.168.31.147/24
    }
    track_script {
        check_nginx
    } 
}
EOF
vim /etc/keepalived/keepalived.conf

# 准备状态检查脚本
cat > /etc/keepalived/check_nginx.sh  << "EOF"
#!/bin/bash
count=$(ss -antp |grep 16443 |egrep -cv "grep|$$")

if [ "$count" -eq 0 ];then
    exit 1
else
    exit 0
fi
EOF

chmod +x /etc/keepalived/check_nginx.sh

# 启动服务
systemctl daemon-reload
systemctl enable nginx keepalived --now
systemctl status nginx keepalived
systemctl restart nginx keepalived

# 【keepalived master 节点】查看keepalived工作状态
ip -4 a


# 测试 keepalived工作状态01
# - 关闭主节点Nginx，测试VIP是否漂移到备节点服务器。
#   - 在Nginx Master执行 
#     - systemctl stop nginx && sleep 3 && systemctl start nginx
#   - 在Nginx Backup，ip addr命令查看已成功绑定VIP。
#     - ip -4 a |grep 146

# 测试 keepalived工作状态02
# - 找K8s集群中任意一个节点，使用curl查看K8s版本测试，使用VIP访问：
curl -k https://192.168.31.147:16443/version

# 测试 keepalived工作状态03
# - 通过查看Nginx日志也可以看到转发apiserver IP
tail -f /var/log/nginx/k8s-access.log 
```

### 11.8 【所有 worker】修改所有的Work Node连接LB VIP

```bash
sed -i 's#192.168.31.143:6443#192.168.31.147:16443#' /opt/kubernetes/cfg/*
systemctl restart kubelet kube-proxy

# 检查节点状态
kubectl get nodes
```


可以参考：8.5.2 生成kubeconfig文件
重新生成~/.kube/config文件

至此一个双Master节点k8s集群已经部署完毕

## 12 集群扩容：增加（计划内）worker节点

计划内：在创建下面两个证书时，已经包含该节点。

- etcd证书：
  - 7. 安装etcd
    - 7.1 【master-143】etc安装：准备证书
- k8s证书：
  - 8 【master-143】部署master
    - 8.1 生成 kube-apiserver 证书

### 12.1 【master-143】复制文件到新节点

```bash
ALL_EXIST_NODES=(
  192.168.31.143
  192.168.31.144
  192.168.31.145
)

NEW_NODES=(
  192.168.31.148
)

for i in "${NEW_NODES[@]}"; do
  scp -r /opt/kubernetes root@$i:/opt/
  scp -r /usr/lib/systemd/system/{kubelet,kube-proxy}.service root@$i:/usr/lib/systemd/system
  scp -r /opt/kubernetes/ssl/ca.pem root@$i:/opt/kubernetes/ssl/
  ssh root@$i "rm -f /opt/kubernetes/cfg/kubelet.kubeconfig"
  ssh root@$i "rm -f /opt/kubernetes/ssl/kubelet*"

  for j in "${ALL_EXIST_NODES[@]}"; do
    ssh root@$j "echo $i k8s-node-${i##*.} >> /etc/hosts"
  done

done
```

### 12.2 【所有 worker】基础配置

#### 12.2.1 【所有机器】基础配置

[同：minikube 中该步骤](https://yc913344706.github.io/posts/k8s-install-by-minikube)

#### 12.2.2 【所有机器】集群主机名配置

```bash
# set hostname
# ---------
hostnamectl set-hostname k8s-node-148

# set hosts
# ---------
cat >> /etc/hosts << EOF
192.168.31.143 k8s-master-143
192.168.31.144 k8s-master-144
192.168.31.145 k8s-master-145
EOF
```

#### 12.2.3 【所有机器】配置时钟同步

[同：kubeadm 中该步骤](https://yc913344706.github.io/posts/k8s-install-by-kubeadm)

#### 12.2.4 【所有机器】防火墙，iptables, swap

[同：kubeadm 中该步骤](https://yc913344706.github.io/posts/k8s-install-by-kubeadm)

#### 12.2.5 【所有机器】安装CRI: containerd

[同：kubeadm 中该步骤](https://yc913344706.github.io/posts/k8s-install-by-kubeadm)

#### 12.2.6 【所有机器】配置 ipvs, connectrack

[同：kubeadm 中该步骤](https://yc913344706.github.io/posts/k8s-install-by-kubeadm)


### 12.3 【所有 worker】修改配置文件

```bash
vim /opt/kubernetes/cfg/kubelet.conf
#--hostname-override=k8s-node-148

vim /opt/kubernetes/cfg/kube-proxy-config.yml
#hostnameOverride: k8s-node-148

systemctl daemon-reload
systemctl enable kubelet kube-proxy --now
systemctl status kubelet kube-proxy
```

### 12.3 【master-143】接受证书请求

```bash
#查看证书请求
kubectl get csr
#同意
kubectl certificate approve node-csr-XXXXXXXXXXXX

# 查看node
kubectl get node
```

## 13 集群扩容：增加（计划外）worker节点

目前看，与 `12 集群扩容：增加（计划内）worker节点` 相同步骤，即可node ready。

以后看，若有新的问题，再行记录。


## 100 其他k8s命令

### 100.1 从集群中软删除node

软删除：只是不调度，并不真正删除node

```bash
kubectl drain k8s-node-148 --delete-local-data --ignore-daemonsets --force
kubectl cordon k8s-node-148
```
