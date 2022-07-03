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
> 创建一个 nginx Pod

### step 1: 便携nginx.yaml
```yaml:nginx.yaml
apiversion: v1
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