---
date: 2022-09-16
description: "实战 k8s Pod的PriorityClass 和抢占"
image: "images/kuber.jpg"
title: "k8s PriorityClass"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- k8s
series:
- 
---

## 应对问题

* Kubernetes集群资源十分紧张，而此时又需要部署一些比较重要的业务，如何去进行“抢占”集群资源，让关键业务在集群跑起来？
> https://kubernetes.io/zh-cn/docs/concepts/scheduling-eviction/pod-priority-preemption/#how-to-use-priority-and-preemption

## PriorityClass

* PriorityClass 是一个无命名空间对象，主要有三个字段
> `value`：可以设置为任何小于或等于 10 亿的 32 位整数值，value越大，代表使用这个PriorityClass的Pod优先级越高，越不容易被抢占资源；
 `globalDefault`：globalDefault 字段表示这个 PriorityClass 的值应该用于没有 priorityClassName 的 Pod， 
 系统中只能存在一个 globalDefault 设置为 true 的 PriorityClass，如果不存在设置了 globalDefault 的 PriorityClass， 则没有 priorityClassName 的 Pod 的优先级为零。
 `description`： 是一个任意字符串。 它用来告诉集群用户何时应该使用此 PriorityClass。


## 抢占测试说明

### 集群环境

* k8s v1.24
* 两台Work Node，每台CPU数都为两核，Memory 不限(足够测试)。

### 测试流程

* 创建两个PriorityClass，一个为High，一个为low，然后创建两个PriorityClass为low的Pod，且CPU均需要占用1.3核CPU。然后再创建一个需要占用1.3核心的CPU的Pod,且PriorityClass为high

* 当PriorityClass被创建后，会分别占用两台Node的资源，且无法均再创建一个1.3核心CPU的Pod（1.3 + 1.3 > 2），当创建PriorityClass为High的Pod后，就会强制抢占PriorityClass其中的一个


## 实战

### 创建PriorityClass
> high PriorityClass的 value 高于 low PriorityClass的 value

1. 创建high PriorityClass
```yaml:high-priorityclass.yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1001
globalDefault: false
description: "此优先级类应仅用于 XYZ 服务 Pod。"
```

2. 创建low PriorityClass
```yaml:low-priorityclass.yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: low-priority
value: 1000
globalDefault: false
description: "此优先级类应仅用于 ABC 服务 Pod。"
```

### 创建low-nginx Pod

1. 创建nginx-low-1 Pod
```yaml:nginx-low-1.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-low-1
spec:
  containers:
    - name: nginx
      image: nginx
      imagePullPolicy: IfNotPresent
      resources:
        limits:
          memory: "300Mi"
          cpu: "1300m"
  priorityClassName: low-priority
```

2. 创建nginx-low-2 Pod
```yaml:nginx-low-2.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-low-2
spec:
  containers:
    - name: nginx
      image: nginx
      imagePullPolicy: IfNotPresent
      resources:
        limits:
          memory: "300Mi"
          cpu: "1300m"
  priorityClassName: low-priority
```

3. kubectl apply -f nginx-low-1.yaml nginx-low-2.yaml

### 创建high-ngin Pod
1. 创建nginx-high Pod
```yaml:nginx-high.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-high
spec:
  containers:
    - name: nginx
      image: nginx
      imagePullPolicy: IfNotPresent
      resources:
        limits:
          memory: "100Mi"
          cpu: "1300m"
  priorityClassName: high-priority
```

2. 可以另开一个终端，执行`kubectl get pod -w`来观察Pod的变化

3. 执行`kubectl apply -f nginx-high.yaml`

4. 可以观察有一个low-nginx Pod被抢占


## End（实战非抢占Priority Class）
* 想要设置优先级高的Pod，并且不想让它去抢占别的Pod，可以在它使用的PriorityClass的preemptionPolicy字段设置为Never

1. 创建非抢占优先级的PriorityClass
```yaml:nopreemption.yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority-nonpreempting
value: 1002
preemptionPolicy: Never
globalDefault: false
description: "This priority class will not cause other pods to be preempted."
```

2. 创建使用它的Pod
```yaml:nginx-nopreemption.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-nopreemption
spec:
  containers:
    - name: nginx
      image: nginx
      imagePullPolicy: IfNotPresent
      resources:
        limits:
          memory: "100Mi"
          cpu: "1300m"
  priorityClassName: high-priority-nonpreempting
```

3. 依次使用命令创建对象（在之前抢占示例的基础上），发现就算这个PriorityClass的value比较大（优先级大），也无法抢占别的Pod