---
date: 2022-03-23
description: "安装中..."
image: "images/kuber.jpg"
title: "k8s的安装"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- k8s
series:
- 
---

{{< notice info >}}
Linux版本为centos 7.x，建议全部看完再进行操作
{{< /notice >}}

## 准备工作
> 有两种安装需求，第一种就是主机都是内网IP能ping通的（比如自己电脑的两个虚拟机）。第二种用公网才能Ping通的（比如两台云服务器），下面会对两者进行区别。
1. 先开启云服务器控制台的的端口TCP入规则
> 端口：6443,2379,2380,80,2381


## 在同一个网络的机器
> 把脚本放入对应机器执行即可（记得修改基本里面的一些参数，比如IP地址）
> 这里用虚拟机 中ip地址做示例，master IP为 192.168.200.128，node1为 192.168.200.129
### Master节点
```Shell:master.sh
#!/bin/bash
set -e

# 安装日志
install_log=/var/log/install_k8s.log
tm=$(date +'%Y%m%d %T')

# 日志颜色
COLOR_G="\x1b[0;32m"  # green
RESET="\x1b[0m"

function info(){
    echo -e "${COLOR_G}[$tm] [Info] ${1}${RESET}"
}

function run_cmd(){
  sh -c "$1 | $(tee -a "$install_log")"
}

function run_function(){
  $1 | tee -a "$install_log"
}

function install_docker(){
  info "1.使用脚本自动安装docker..."
  curl -sSL https://get.daocloud.io/docker | sh

  info "2.启动 Docker CE..."
  systemctl enable docker
  systemctl start docker

  info "3.添加镜像加速器..."
  if [ ! -f "/etc/docker/daemon.json" ];then
    touch /etc/docker/daemon.json
  fi
  cat <<EOF > /etc/docker/daemon.json
{
  "registry-mirrors": ["https://registry.cn-hangzhou.aliyuncs.com"],
  "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF

  info "4.重新启动服务..."
  systemctl daemon-reload
  systemctl restart docker

  info "5.测试 Docker 是否安装正确..."
  docker run hello-world

  info "6.检测..."
  docker info
}

function install_k8s() {
    info "初始化k8s部署环境..."
    init_env

    info "添加k8s安装源..."
    add_aliyun_repo

    info "安装kubelet kubeadmin kubectl..."
    install_kubelet_kubeadmin_kubectl

    info "安装kubernetes master..."
    yum -y install net-tools
    if [[ ! "$(ps aux | grep 'kubernetes' | grep -v 'grep')" ]];then
      kubeadmin_init
    else
      info "kubernetes master已经安装..."
    fi

    info "安装网络插件flannel..."
    install_flannel

    info "去污点..."
    kubectl taint nodes --all node-role.kubernetes.io/master-
}

# 初始化部署环境
function init_env() {
  info "关闭防火墙"
  systemctl stop firewalld
  systemctl disable firewalld

  info "关闭selinux"
  sed -i 's/^SELINUX=enforcing$/SELINUX=disabled/g' /etc/selinux/config
  source /etc/selinux/config

  info "关闭swap（k8s禁止虚拟内存以提高性能）"
  swapoff -a
  sed -i '/swap/s/^\(.*\)$/#\1/g' /etc/fstab

  info "设置网桥参数"
  cat <<-EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
  sysctl --system  #生效
  sysctl -w net.ipv4.ip_forward=1
}

# 添加aliyun安装源
function add_aliyun_repo() {
  cat > /etc/yum.repos.d/kubernetes.repo <<- EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
}

function install_kubelet_kubeadmin_kubectl() {
  yum install kubelet-1.19.4 kubeadm-1.19.4 kubectl-1.19.4 -y
  systemctl enable kubelet.service

  info "确认kubelet kubeadmin kubectl是否安装成功"
  yum list installed | grep kubelet
  yum list installed | grep kubeadm
  yum list installed | grep kubectl
  kubelet --version
}

function kubeadmin_init() {
  sleep 1
# 修改
ip=192.168.200.128
hostnamectl set-hostname master
cat >> /etc/hosts << EOF 
${ip} master
192.168.200.129 node1
EOF
  kubeadm init --apiserver-advertise-address="${ip}" --image-repository registry.aliyuncs.com/google_containers --kubernetes-version v1.19.4 --service-cidr=10.96.0.0/12 --pod-network-cidr=10.244.0.0/16
  mkdir -p "$HOME"/.kube
  cp -i /etc/kubernetes/admin.conf "$HOME"/.kube/config
  chown "$(id -u)":"$(id -g)" "$HOME"/.kube/config


}

function install_flannel() {
  yum -y install wget
  wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
  kubectl apply -f kube-flannel.yml
}



# 安装docker
read -p "是否安装docker？默认为：no. Enter [yes/no]：" is_docker
if [[ "$is_docker" == 'yes' ]];then
  run_function "install_docker"
fi

# 安装k8s
read -p "是否安装k8s？默认为：no. Enter [yes/no]：" is_k8s
if [[ "$is_k8s" == 'yes' ]];then
  run_function "install_k8s"
fi

```


### Work Node
```Shell
#!/bin/bash
set -e

# 安装日志
install_log=/var/log/install_k8s.log
tm=$(date +'%Y%m%d %T')

# 日志颜色
COLOR_G="\x1b[0;32m"  # green
RESET="\x1b[0m"

function info(){
    echo -e "${COLOR_G}[$tm] [Info] ${1}${RESET}"
}

function run_cmd(){
  sh -c "$1 | $(tee -a "$install_log")"
}

function run_function(){
  $1 | tee -a "$install_log"
}

function install_docker(){
  info "1.使用脚本自动安装docker..."
  curl -sSL https://get.daocloud.io/docker | sh

  info "2.启动 Docker CE..."
  systemctl enable docker
  systemctl start docker

  info "3.添加镜像加速器..."
  if [ ! -f "/etc/docker/daemon.json" ];then
    touch /etc/docker/daemon.json
  fi
  cat <<EOF > /etc/docker/daemon.json
{
  "registry-mirrors": ["https://registry.cn-hangzhou.aliyuncs.com"],
  "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF

  info "4.重新启动服务..."
  systemctl daemon-reload
  systemctl restart docker

  info "5.测试 Docker 是否安装正确..."
  docker run hello-world

  info "6.检测..."
  docker info
}

function install_k8s() {
    info "初始化k8s部署环境..."
    init_env

    info "添加k8s安装源..."
    add_aliyun_repo

    info "安装kubelet kubeadmin kubectl..."
    install_kubelet_kubeadmin_kubectl

    info "安装kubernetes master..."
    yum -y install net-tools
    if [[ ! "$(ps aux | grep 'kubernetes' | grep -v 'grep')" ]];then
      kubeadmin_init
    else
      info "kubernetes master已经安装..."
    fi

}

# 初始化部署环境
function init_env() {
  info "关闭防火墙"
  systemctl stop firewalld
  systemctl disable firewalld

  info "关闭selinux"
  sed -i 's/^SELINUX=enforcing$/SELINUX=disabled/g' /etc/selinux/config
  source /etc/selinux/config

  info "关闭swap（k8s禁止虚拟内存以提高性能）"
  swapoff -a
  sed -i '/swap/s/^\(.*\)$/#\1/g' /etc/fstab

  info "设置网桥参数"
  cat <<-EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
  sysctl --system  #生效
  sysctl -w net.ipv4.ip_forward=1
}

# 添加aliyun安装源
function add_aliyun_repo() {
  cat > /etc/yum.repos.d/kubernetes.repo <<- EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
}

function install_kubelet_kubeadmin_kubectl() {
  yum install kubelet-1.19.4 kubeadm-1.19.4 kubectl-1.19.4 -y
  systemctl enable kubelet.service

  info "确认kubelet kubeadmin kubectl是否安装成功"
  yum list installed | grep kubelet
  yum list installed | grep kubeadm
  yum list installed | grep kubectl
  kubelet --version
}

function kubeadmin_init() {
  sleep 1
# 修改
hostnamectl set-hostname node1
cat >> /etc/hosts << EOF 
192.168.200.128 master
192.168.200.129 node1
EOF
## 修改
mkdir -p /etc/cni/net.d/
read -p "请将 master 节点的 admin.conf 拷贝到 node1（20秒后自动跳过，在master上执行 scp /etc/kubernetes/admin.conf root@node1:/etc/kubernetes/）:" -t 20 a
read -p "请将master节点下面 /etc/cni/net.d/下面的所有文件拷贝到node节点上（20秒后自动跳过，在master上执行scp /etc/cni/net.d/* root@node1:/etc/cni/net.d/）" -t 20 a

  mkdir -p "$HOME"/.kube
  cp -i /etc/kubernetes/admin.conf "$HOME"/.kube/config
  chown "$(id -u)":"$(id -g)" "$HOME"/.kube/config
}


# 安装docker
read -p "是否安装docker？默认为：no. Enter [yes/no]：" is_docker
if [[ "$is_docker" == 'yes' ]];then
  run_function "install_docker"
fi

# 安装k8s
read -p "是否安装k8s？默认为：no. Enter [yes/no]：" is_k8s
if [[ "$is_k8s" == 'yes' ]];then
  run_function "install_k8s"
fi
```


## 不同网络的机器
> 把上面脚本放入对应机器，先不要执行！
### 先写好配置文件
* 先编辑好一个要用的etcd需要配置的文件 `vim ectd.yaml`（只有第二种需要操作）
> 将下面的`公网IP`替换成自己云服务器的公网IP（需要四处）
``` yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/etcd.advertise-client-urls: https://公网IP:2379
  creationTimestamp: null
  labels:
    component: etcd
    tier: control-plane
  name: etcd
  namespace: kube-system
spec:
  containers:
  - command:
    - etcd
    - --advertise-client-urls=https://公网IP:2379
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt
    - --client-cert-auth=true
    - --data-dir=/var/lib/etcd
    - --initial-advertise-peer-urls=https://公网IP:2380
    - --initial-cluster=master-k8s=https://公网IP:2380
    - --key-file=/etc/kubernetes/pki/etcd/server.key
    - --listen-metrics-urls=http://127.0.0.1:2381
    - --listen-client-urls=https://0.0.0.0:2379
    - --listen-peer-urls=https://0.0.0.0:2380
    - --name=master-k8s
    - --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
    - --peer-client-cert-auth=true
    - --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
    - --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    - --snapshot-count=10000
    - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    image: registry.aliyuncs.com/google_containers/etcd:3.4.3-0
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /health
        port: 2381
        scheme: HTTP
      initialDelaySeconds: 15
      timeoutSeconds: 15
    name: etcd
    resources: {}
    volumeMounts:
    - mountPath: /var/lib/etcd
      name: etcd-data
    - mountPath: /etc/kubernetes/pki/etcd
      name: etcd-certs
  hostNetwork: true
  priorityClassName: system-cluster-critical
  volumes:
  - hostPath:
      path: /etc/kubernetes/pki/etcd
      type: DirectoryOrCreate
    name: etcd-certs
  - hostPath:
      path: /var/lib/etcd
      type: DirectoryOrCreate
    name: etcd-data
status: {}
``` 

### master开始执行
* `chmod 777 master.sh` 赋予执行权限
* `sh k8s.sh` 执行脚本
* 当再开一个ssh终端连接master云服务器，当执行shell脚本出现`[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s`时候，在新开的终终端执行`cat etcd.yaml > /etc/kubernetes/manifests/etcd.yaml`


