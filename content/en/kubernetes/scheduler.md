---
date: 2022-09-17
description: "实战 k8s Pod调度的高级特性"
image: "images/kuber.jpg"
title: "k8s 调度器"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- k8s
series:
- 
---

## kube-scheduler 调度流程 
* 过滤：根据规则，将生成一个可以让Pod 调度的节点列表
* 打分：根据评分规则，将Pod调度到得分最高的节点
> https://kubernetes.io/zh-cn/docs/concepts/scheduling-eviction/kube-scheduler/#scheduling

## 调度器的高级特性
* NodeName 和 NodeSelector，可以让Pod直接调度到设置的节点上
* NodeAffinity：决定Pod部署在哪些主机上
* PodAffinity：决定Pod和哪些已经在运行中的Pod部署在同一拓扑域
* PodAntiAffinity：决定Pod‘不和’哪些已经在运行中的Pod部署在同一拓扑域


## 实战Nodename
1. 编写yaml文件
```yaml:nodename.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nodename-test
spec:
  containers:
    - name: nginx
      image: nginx
      imagePullPolicy: IfNotPresent
  nodeName: node1
```

2. 执行kubectl apply，然后可以修改nodeName，查看Pod是否每一次都调度到指定的节点

3. Nodename注意点：Nodename调度的优先级高于NodeSelector以及节点亲和性 和 非亲和性的规则
> https://kubernetes.io/zh-cn/docs/concepts/scheduling-eviction/assign-pod-node/#nodename


## 实战NodeSelector
> https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/assign-pods-nodes/

1. 给想要调度的节点打上label
* `kubectl label node node1 nodeselect=node1`
> key为nodeselect，value为node1
> kubectl get nodes node1 --show-labels 用于查看节点对应的label

2. 编写yaml文件
```yaml:nodeselector.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nodeselector-test
spec:
  containers:
    - name: nginx
      image: nginx
      imagePullPolicy: IfNotPresent
  nodeSelector:
    nodeselect: node1
```

3. 执行kubectl apply，然后可以给其他node打上label，修改nodeSelector，查看Pod是否每一次都调度到对应label的节点

## 实战NodeAffinity
> https://kubernetes.io/zh-cn/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity
* 在有了`nodeName`和`nodeSelector`之后，我们可以将Pod调度到指定的节点或包含某种标签的节点上。而`NodeAffinity`的功能更加强大，它不仅能 按照‘硬性要求’只将Pod调度到含有特殊标签的节点上，它还能按照‘软性规则’将尝试将Pod调度到某中标签的节点上，如果没有这样的节点，调度器仍然可以调度该Pod

* `nodeAffinity`可以这种两种规则，`requiredDuringSchedulingIgnoredDuringExecution`： 调度器只有在规则被满足的时候才能执行调度。此功能类似于 nodeSelector， 但其语法表达能力更强。
`preferredDuringSchedulingIgnoredDuringExecution`： 调度器会尝试寻找满足对应规则的节点。如果找不到匹配的节点，调度器仍然会调度该 Pod。
> IgnoredDuringExecution 意味着如果节点标签在 Kubernetes 调度 Pod 时发生了变更，Pod 仍将继续运行。

1. 编写yaml文件并提前创建好标签
> 在node1上有标签zone=zoneA, node2上有标签zone=zoneB
```yaml:nodeAffinity.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nodeaffinity
spec:
  containers:
    - name: nginx
      image: nginx
      imagePullPolicy: IfNotPresent
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: kubernetes.io/hostname
                operator: In # 匹配规则为 key必须要在node1或node2中
                values:
                  - "node1"
                  - "node2"
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 1 # 权重为1
          preference:
            matchExpressions: # 匹配规则为zone 为zoneA
              - key: zone
                operator: In
                values:
                  - "zoneA"
        - weight: 2 # 权重为2
          preference:
            matchExpressions: # 匹配规则为zone 为zoneA
              - key: zone
                operator: In
                values:
                  - "zoneB"
```

2. 执行kubectl apply，查看Pod 被调度到的节点为node2


### 注意事项
* requiredDuringSchedulingIgnoredDuringExecution 是硬性要求，是必须要满足的条件，如果指定了多个与 nodeAffinity 类型关联的 nodeSelectorTerms， 只要其中一个 nodeSelectorTerms 满足的话，Pod 就可以被调度到节点上。

* 如果指定了多个与同一 nodeSelectorTerms 关联的 `matchExpressions`， 就必须要当 matchExpressions 都满足时 Pod 才可以被调度到节点上。

* 同时指定了 nodeSelector 和 nodeAffinity，两者 必须都要满足， 才能将 Pod 调度到候选节点上。

* 如果在NodeAffinity中同时设置了硬性要求 和 软性要求，就必须当硬性要求满足之一才能进行软性要求的策略

* preferredDuringSchedulingIgnoredDuringExecution 亲和性类型可以设置`weight`，将所有满足匹配的weight值相加，总分越高，优先级也越高

## 实战Pod 间亲和性和反亲和性

* 对于Nodeaffinity是基于节点的label来实现亲和性，而这里使用到的PodAffinity和PodAntiAffinity是基于Pod的label来实现亲和性和反亲和性

* PodAffinity 和 PodAntiAffinity 同样拥有requiredDuringSchedulingIgnoredDuringExecution
preferredDuringSchedulingIgnoredDuringExecution两个类型

## 实战
1. 创建对应的环境
* 在node1节点上拥有一个name为Pod1，lable含有name=pod1的Pod，且node1设置一个label：topologyX=node1（表示它属于topologyX这个拓扑域）
* 在node2节点上拥有一个name为Pod2，lable含有name=pod2的Pod，且node1设置一个label：topologyX=node2，topologyY=node2（表示它属于topologyX和topologyY这两个拓扑域）
1. 设置label
```shell
kubectl label nodes node1 topologyX=node1
kubectl label nodes node2 topologyX=node2
kubectl label nodes node2 topologyY=node2
```

2. 编写Pod1和Pod2 yaml文件
```yaml:pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  labels:
    name: pod1
spec:
  containers:
    - name: nginx
      image: nginx
      imagePullPolicy: IfNotPresent
  nodeName: node1
---
apiVersion: v1
kind: Pod
metadata:
  name: pod2
  labels:
    name: pod2
spec:
  containers:
    - name: nginx
      image: nginx
      imagePullPolicy: IfNotPresent
  nodeName: node2
```

3. 编写测试podAffinity的Pod文件
```yaml:podAffinity.yaml
apiVersion: v1
kind: Pod
metadata:
  name: podaffinity
spec:
  containers:
    - name: nginx
      image: nginx
      imagePullPolicy: IfNotPresent
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        - labelSelector:
            matchExpressions:
              - key: name
                operator: In # 匹配规则为 key必须要在pod1或pod2中
                values:
                  - "pod1"
                  - "pod2"
          topologyKey: "topologyX"
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 100
          podAffinityTerm:
            labelSelector:
              matchExpressions:
                - key: name
                  operator: In
                  values:
                    - "pod1"
                    - "pod2"
            topologyKey: "topologyY"
```

4. 依次执行，查看 podaffinity这个pod的调度结果，

5. 分析结果，最终 podaffinity被调度到node1上，首先是先匹配podAffinity的'硬性要求'，他们都能匹配matchExpressions，而且都属于topologyX。
然后就是匹配 podAntiAffinity的软性要求，由于Pod2不仅匹配了matchExpressions，而且也属于topologyY这一个拓扑域，所以Pod不能调度到node2上，最终选择node1

### 注意点
* podAffinity 规范当前Pod`应该`调度到 ‘已经运行了某些规定Pod’上的node上，而podAntiAffinity就是规范当前Pod`不应该`调度到 ‘已经运行了某些规定Pod’上的node上


## 实战污点和容忍度
> https://kubernetes.io/zh-cn/docs/concepts/scheduling-eviction/taint-and-toleration/
* 对于Pod的亲和性和反亲和性，可以分别使用PodAffinity 和 PodAntiAffinity，而对于Node的亲和性，也可以使用nodeSelector或者NodeAffinity，那么Node的反亲和性是怎么来做呢？答案是：`污点`（taint）

* 当一个Node上有污点（taint），就表示它需要排斥某一类Pod，而如果Pod想调度到这个Node，就必须要设置一个容忍度，如果容忍度的规则 与 污点匹配的话，就可以调度到这Node。（就好比我们生活中，总有一些讨厌去的地方，但是我们都在尽量地容忍，比如有的不想去教室上课，但为了某种目的，我们最终还是要容忍着去上课）


### 简单实战

1. 给Node加上污点
```shell
# 给node1打上一个 key1：value1键值对 且效果为NodeSchedule的污点
kubectl taint nodes node1 key1=value1:NoSchedule 
# 上同
kubectl taint nodes node2 key2=value2:NoSchedule
```

2. 编写yaml文件
```yaml:taint.yaml
apiVersion: v1
kind: Pod
metadata:
  name: taint-pod
spec:
  containers:
    - name: nginx
      image: nginx
      imagePullPolicy: IfNotPresent
  tolerations:
    - key: key1
      operator: Equal
      value: value1
      effect: NoSchedule
```

3. 最终结果：Pod运行在了Pod1上，因为Pod无法容忍node2节点（没有针对node2设置tolerations）

### 注意事项
* 对于tolerations的operator可以使用两种，如果 operator 是 Exists （此时容忍度不能指定 value），key就需要存在；如果operator 是 Equal ，则它们的 value 应该相等

* 对于污点effect可以这种三种，`NoSchedule`：如果不能容忍污点，就不能调度到该node；`perferNoSchedule`：如果不能容忍污点，“尽量”不要调度到这个node，`NoExcute`：如果不能容忍污点，就无法在该node上运行，即使污点是在运行之后加的，也会将之前的pod进行驱逐。

* operator 是 Exists，并且没有设置Key，就代表这个容忍度能容忍任何污点

* 如果effect为空，则代表可以匹配任何的effect