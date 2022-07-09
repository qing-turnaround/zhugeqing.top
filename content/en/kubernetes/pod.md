---
date: 2022-03-25
description: "k8s Pod学习"
image: "images/kuber.jpg"
title: "k8s Pod"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- k8s
series:
- 
---

## 快速使用
* Pod 是可以在 Kubernetes 中创建和管理的、最小的可部署的计算单元。
* Pod 是一组（一个或多个） 容器； 这些容器共享存储、网络、以及怎样运行这些容器的声明。
* 如果一个Pod 由多个容器构成，那么这些容器将绑定在同一个环境中，可以理解为这些应用在同一个主机中运行。
> 创建一个 nginx Pod

### step 1: 便携nginx.yaml
```yaml:nginx.yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-nginx
  labels:
    name: my-nginx
spec:
  containers:
    - name: my-nginx
      image: nginx
      resources:
        limits:
          memory: "128Mi"
          cpu: "500m"
      ports:
        - containerPort: 80
```

### step 2: 通过yaml文件创建 Pod
* `kubectl create -f nginx.yaml` （创建pod）
* `kubectl get pod` （查看pod运行情况）
> -w 持续查看
* `kubectl describe pod my-nginx` （查看pod信息）
* `kubectl exec -it my-nginx sh` （进入容器内部）
* `kubectl delete pod my-nginx` （删除Pod）
* `kubectl delete -f nginx.yaml`（删除Pod）

## Pod 生命周期
> 

#### Pod Phase
* `Pending（挂起）：Pod 已被 Kubernetes 系统接受，但有一个或者多个容器尚未创建亦未运行。此阶段包括等待 Pod 被调度的时间和通过网络下载镜像的时间。`
* `Running（运行中）：Pod 已经绑定到了某个节点，Pod 中所有的容器都已被创建。至少有一个容器仍在运行，或者正处于启动或重启状态。`
* `Succeeded（成功）：Pod 中的所有容器都已成功终止，并且不会再重启。`
> 也就是调度任务成功执行完成
* `Failed（失败）	Pod 中的所有容器都已终止，并且至少有一个容器是因为失败终止。也就是说，容器以非 0 状态退出或者被系统终止。`
* `Unknown（未知）	因为某些原因无法取得 Pod 的状态。这种情况通常是因为与 Pod 所在主机通信失败。`

## 为容器的生命周期事件设置处理函数
> https://kubernetes.io/docs/tasks/configure-pod-container/attach-handler-lifecycle-event/

### 定义 postStart 和 preStop 处理函数 
* Kubernetes 在容器创建后立即发送 postStart 事件
* Kubernetes 在容器结束前立即发送 preStop 事件

```yaml:lifecycle-event.yaml
apiVersion: v1

kind: Pod

metadata:

  name: lifecycle-demo

spec:

  containers:

  - name: lifecycle-demo-container

    image: nginx

    lifecycle:

      postStart:

        exec:

          command: ["/bin/sh", "-c", "echo Hello from the postStart handler > /usr/share/message"]

      preStop:

        exec:

          command: ["/bin/sh","-c","nginx -s quit; while killall -0 nginx; do sleep 1; done"]
```

## 创建包含Init容器的Pod
> https://kubernetes.io/docs/concepts/workloads/pods/init-containers/
* 每个 Pod 中可以包含多个容器， 应用运行在这些容器里面，同时 Pod 也可以有一个或多个先于应用容器启动的 Init 容器。
* 总是运行到完成
* 如果Pod 的Init容器运行失败，k8s 会不断重启 该Pod（除非设置restartPolicy 为 Never）
* Init 容器支持应用容器的全部字段和特性，包括资源限制、数据卷和安全设置，不支持 lifecycle、livenessProbe、readinessProbe 和 startupProbe， 因为它们必须在 Pod 就绪之前运行完成。

### 创建
```yaml:InitPod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-myapp
  labels:
    app: init-myapp
spec:
  containers:
    - name: myapp-container
      image: busybox:1.28
      command: ["sh", "-c", "echo The app is running! && sleep 3600"]
      resources:
        limits:
          memory: "128Mi"
          cpu: "500m"
  initContainers:
    - name: init-container
      image: busybox:1.28
      command: ["sh", "-c", "echo The app is Inited! && sleep 10"]
```

* `kubectl create -f InitPod.yaml` （创建Pod）
* `kubectl logs init-myapp-c myapp-container`（查看 init-myapp Pod 中Init容器之后的第一个容器）
* `kubectl logs init-myapp -c init-container`  （查看init-myapp Pod 中 Init容器日志）
* `kubectl get -f initpod.yaml -w` 

## 用探针检查 Pod的健康性
> https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/pod-lifecycle/#container-probes
> https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/
* probe(探针) 是由 kubelet 对容器执行的定期诊断。 要执行诊断，kubelet 既可以在容器内执行代码，也可以发出一个网络请求
### 探针机制
* `exec`：在容器内执行指定命令。如果命令退出时返回码为 0 则认为诊断成功。
* `grpc`：使用 gRPC 执行一个远程过程调用。 目标应该实现 gRPC健康检查。 如果响应的状态是 "SERVING"，则认为诊断成功。
`httpGet`：对容器的 IP 地址上指定端口和路径执行 HTTP GET 请求。如果响应的状态码大于等于 200 且小于 400，则诊断被认为是成功的。
`tcpSocket`：对容器的 IP 地址上的指定端口执行 TCP 检查。如果端口打开，则诊断被认为是成功的。

### 探测结果
* `Success（成功）`：容器通过了诊断。
* `Failure（失败）`：容器未通过诊断。
* `Unknown（未知）`：诊断失败，因此不

### 尝试
```yaml:http-liveness.yaml
apiVersion: v1

kind: Pod

metadata:

  labels:

    test: liveness

  name: liveness-http

spec:

  containers:

  - name: liveness

    image: mirrorgooglecontainers/liveness

    args:

    - /server

    livenessProbe:

      httpGet:

        path: /healthz

        port: 8080

        httpHeaders:

        - name: Custom-Header

          value: Awesome

      initialDelaySeconds: 3

      periodSeconds: 3
```

* 在这个配置文件中，periodSeconds 字段指定了 kubelet 每隔 3 秒执行一次存活探测。 initialDelaySeconds 字段告诉 kubelet 在执行第一次探测前应该等待 3 秒。 kubelet 会向容器内运行的服务（服务在监听 8080 端口）发送一个 HTTP GET 请求来执行探测。 如果服务器上 /healthz 路径下的处理程序返回成功代码，则 kubelet 认为容器是健康存活的。 如果处理程序返回失败代码，则 kubelet 会杀死这个容器并将其重启。


## 为容器设置启动时要执行的命令和参数
> https://kubernetes.io/zh-cn/docs/tasks/inject-data-application/define-command-argument-container/

## 为容器定义相互依赖的环境变量 
> https://kubernetes.io/zh-cn/docs/tasks/inject-data-application/define-interdependent-environment-variables/
* 使用的变量需要提前定义，不然无法使用  

## 为容器和 Pods 分配 CPU 资源
> https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/assign-cpu-resource/
* 启动容器时一定要主要对cpu资源进行限制
* Pod 调度是基于`资源请求值`来进行的。 仅在某节点具有足够的 CPU 资源来满足 Pod CPU 请求时，Pod 将会在对应节点上运行，比如当resource-request-cpu超过所有节点时，尽管args-cpus设置的比较低也是无法调度Pod的


## 用节点亲和性把 Pods 分配到对应节点上
> https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity/

1. `kubectl get nodes --show-labels`，列出集群中的节点及其标签
2. `kubectl label node node2 disktype=ssd`选择一个节点（比如这里选择node2节点），给它添加一个标签
3. 创建yaml文件
```yaml:affinity.yaml
apiVersion: v1

kind: Pod

metadata:

  name: nginx

spec:

  affinity:

    nodeAffinity:

      requiredDuringSchedulingIgnoredDuringExecution:

        nodeSelectorTerms:

        - matchExpressions:

          - key: disktype

            operator: In

            values:

            - ssd            

  containers:

  - name: nginx

    image: nginx

    imagePullPolicy: IfNotPresent
```

4.创建Pod，发现运行在node2上（可以再修改lable 和 yaml文件尝试）

## 将configmap数据注入容器

1. 编写configmap.yaml（定义configmap 和 Pod）
```yaml:configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-db-config
data:
  db-url: localhost
---
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  labels:
    name: myapp
spec:
  containers:
  - name: myapp
    image: busybox
    command: ["sh","-c","env"]
    envFrom:
      - configMapRef:
          name: my-db-config
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
```

2. `kubectl create -f configmap.yaml` （创建资源）


## 使用非 root 用户 运行 Pod


### docker 用户权限
* 对于docker 来说，启动容器，默认是赋予 root 权限，但是这个root用户 权限比较少，比如无法修改hostname，需要赋予更高`特权模式权限`， 在运行时加入参数 `--privileged`


### 为 Pod 或容器配置安全上下文
>  https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/security-context/