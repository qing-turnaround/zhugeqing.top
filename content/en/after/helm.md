---
date: 2022-09-20
description: "实战 Helm的使用"
image: "images/kuber.jpg"
title: "Helm的使用"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- k8s
series:
- 
---


## 为什么需要Helm
* 当我需要在kubernetes中部署一个应用时，往往需要多种对象，包括但不仅限于 ConfigMap，Secret，Pod，PriorityClass等。这样我们就需要编写多个yaml文件（或者是写入同一个yaml，但需要分隔），这会比较麻烦，而Heml将一个应用程序在kubernetes所需要的对象 都打包成一个 `chart`，我如果需要部署一个应用在kubernetes，就可以直接安装一个`chart`包，或者自己来进行编写。
> https://helm.sh/docs/

## 概念
* `Chart`：Chart 代表着 Helm 包。相当于 Yum RPM 在Kubernetes 中的等价物。
* `Repository`： 是用来存放和共享 charts 的地方
* `release`：运行在 Kubernetes 集群中的 chart 的实例（同一个chart可以安装多次）

## 编写一个简单的Chart
> https://helm.sh/docs/chart_template_guide/getting_started/
1. 我们可以直接使用`heml create mychart`来创建一个默认的chart（创建了一个目录），里面包含了一些初始化的默认文件

2. 在 ./mychart/templates 目录下创建一个 `configmap.yaml`

```yaml:configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mychart-configmap
data:
  today: "2022-09-20"
```

3. 执行`helm install mychart ./mychart` 安装这个chart
> 使用kubectl get cm，查看这个configmap是否创建成功

