---
layout: post
title: 【Cloud Native】03-1 K8S 安装 -- minikube
categories: [CloudNative, K8S安装]
tags: [minikube]
---

## 说明

minikube 是单机部署的 k8s 集群。部署步骤如下。

## 1. 【root 用户】基础配置

```bash
# set history
# ---------
echo 'HISTTIMEFORMAT="%d/%m/%y %T "' >> ~/.bashrc

# set vimrc
# ---------
apt install -y git curl
git clone https://gitee.com/ycvayne/sre.git
ln -sfT sre/os/vim/.vimrc .vimrc
```

## 2. 【root 用户】挂载磁盘（按需）

```bash
# mount data
# ---------
lsblk
pvcreate /dev/vdb
vgcreate data /dev/vdb
lvcreate -l 100%FREE -n data data
mkfs.ext4 /dev/data/data
mkdir /data
vim /etc/fstab
# /dev/data/data /data ext4 defaults 0 0
mount -a
lsblk
```

## 3. 【root 用户】安装 docker

```bash
# install docker
# ---------
apt install docker docker.io
ls /etc/docker/
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": [
    "https://hub.si.icu",
    "https://hub.xiaosi.fun",
  ]
}
EOF
systemctl daemon-reload
systemctl restart docker
systemctl status docker
```

## 4. 【root 用户】安装 minikube

```bash
# install minikube
# ---------
mkdir /data/install
cd /data/install/
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb
sudo dpkg -i minikube_latest_amd64.deb

# create user: minikube
# ---------
useradd minikube
usermod -aG docker minikube
usermod -aG sudo minikube
passwd minikube
mkhomedir_helper minikube
usermod --shell /bin/bash minikube
```

## 5. 【minikube 用户】启动 minikube

```bash
su - minikube

# set http proxy, for download / docker pull
# ---------
export https_proxy=http://[PROXY_HOST]:[PROXY_PORT] http_proxy=http://[PROXY_HOST]:[PROXY_PORT] all_proxy=socks5://[PROXY_HOST]:[PROXY_PORT] no_proxy="localhost,127.0.0.1,::1,.local"

# start minikube
# ---------
minikube start
minikube addons enable metrics-server
minikube dashboard --url &

# set alias
# ---------
alias kubectl="minikube kubectl --"
vim .bashrc
. .bashrc

# reset minikube proxy port
# ---------
kubectl proxy --port=8000 --address='192.168.10.143' --accept-hosts='^.*' &
# then, we can access following dashboard url in web browser
# http://192.168.10.143:8000/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/#/ingress?namespace=default
```