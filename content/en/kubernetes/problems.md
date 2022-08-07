---
date: 2022-03-24
description: "解决一些在k8s中遇到的问题"
image: "images/kuber.jpg"
title: "k8s的一些问题"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- k8s
series:
- 
---

## 忘记master节点init后的join命令怎么办
* 在master节点上 执行 `kubeadm token create --print-join-command`，然后将打印出来的对应 命令在 work节点上运行

## 升级k8s
1. 用最新的补丁版本号替换 1.24.x-0 中的 x
`yum install -y kubeadm-1.24.x-0 --disableexcludes=kubernetes`

2. 用最新的补丁版本号替换 1.24.x-00 中的 x
`yum install -y kubelet-1.24.x-0 kubectl-1.24.x-0 --disableexcludes=kubernetes`

3. 查看可以安装的版本
`yum list --showduplicates kubeadm --disableexcludes=kubernetes`
`yum list --showduplicates kubelet --disableexcludes=kubernetes`
`yum list --showduplicates kubectl --disableexcludes=kubernetes`


## k8s 被限制映射的端口
1. `vim /etc/kubernetes/manifests/kube-apiserver.yaml`

2. 在 `spec.containers.command`下添加
`- --service-node-port-range=1-65535`