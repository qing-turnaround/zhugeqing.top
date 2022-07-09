---
date: 2022-03-25
description: "安装一些k8s的插件"
image: "images/kuber.jpg"
title: "安装k8s 插件"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- k8s
series:
- 
---

## 安装 Dashboard
> 均操作在 master上
* wget https://zhugeqing.top/kubernetes/kubernetes-dashboard.yaml
* kubectl create -f kubernetes-dashboard.yaml 
* 访问`https://ip:port` ，默认yaml文件里面的端口为31111
* 输入`kubectl -n kube-system describe $(kubectl -n kube-system get secret -n kube-system -o name | grep namespace) | grep token` ，得到token后输入在登陆界面

## k8s 命令补全（centos）
```sh:bash-completion.sh
#!/bin/bash
yum -y install bash-completion
source /usr/share/bash-completion/bash_completion
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc
```