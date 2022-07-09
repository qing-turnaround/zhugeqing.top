---
date: 2022-03-28
description: "k8s 控制器 学习"
image: "images/kuber.jpg"
title: "k8s controller"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- k8s
series:
- 
---

## 初步了解
* 在Kubernetes平台上，很少会直接创建一个Pod，在大多数情况下会通过RC（Replication Controller）、Deployment、DaemonSet、Job等控制器完成对一组Pod副本的创建、调度及全生命周期的自动控制任务。


## ReplicaSet
> https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/replicaset/
* ReplicaSet 的目的是维护一组在任何时候都处于运行状态的 Pod 副本的稳定集合。 因此，它通常用来保证给定数量的、完全相同的 Pod 的可用性。（我们不应该直接使用底层的ReplicaSet来控制Pod副本，而应该使用管理ReplicaSet的Deployment对象来控制副本，这是来自官方的建议。）

### 快速使用

1. 创建 replicaSet.yaml
```yaml:replicaSet.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx
          ports:
            - containerPort: 80
```

2. 执行`kubectl apply -f replicaSet.yaml`


## Deployment
> https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/

1. 创建 deployment.yaml
```yaml:deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 80
```

2. 执行`kubectl apply -f deployment.yaml`

3. 可以修改deployment.yaml，再重新执行apply命令，达到扩展副本的效果

## StatefulSet
> https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/statefulset/
* 删除或者收缩 StatefulSet 并不会删除它关联的存储卷。 这样做是为了保证数据安全，它通常比自动清除 StatefulSet 所有相关的资源更有价值。
* 当删除 StatefulSets 时，StatefulSet 不提供任何终止 Pod 的保证。 为了实现 StatefulSet 中的 Pod 可以有序地且体面地终止，可以在删除之前将 StatefulSet 缩放为 0。


## DaemonSet
> https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/daemonset/


## Job
> https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/job/