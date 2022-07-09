---
date: 2022-03-25
description: "k8s Service学习"
image: "images/kuber.jpg"
title: "k8s Service"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- k8s
series:
- 
---

## Service 快速使用
> 通过创建Service，可以为一组具有相同功能的Pod提供一个统一的入口地址，并且将请求负载分发到后端的各个Pod应用上
> https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/


1. 创建service.yaml
```yaml:nginx-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
spec:
  selector:
    run: my-nginx
  ports:
  - port: 80
    targetPort: 80
```
2. 创建pod.yaml
```yaml:nginx-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx
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

3. 执行命令 `kubectl create -f nginx-service.yaml`，`kubectl create -f nginx-deployment.yaml`

4. 执行 `kubectl describe svc my-nginx`，`ubectl get pods -l run=my-nginx -o yaml | grep podIP` 查看相关信息

## Service 服务发现
> https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#discovering-services
> https://kubernetes.io/zh-cn/docs/concepts/services-networking/dns-pod-service/#pod-sethostnameasfqdn-field
* Service 服务发现一共有`环境变量` 和 `DNS` 两种方式
* `环境变量`：需要注意 相关联的 Service 需要在 Pod之前创建，不然环境变量无法自动注入
* `DNS`：可以为Service 和 POd 定义 DNS记录

### 快速创建Pod的 DNS记录

1. 编写`dns.yaml`文件（注意hostname 和 subdomain字段）
```yaml:dns.yaml
apiVersion: v1
kind: Service
metadata:
  name: busybox
spec:
  selector:
    app: busybox
  ports:
  - port: 80
    targetPort: 80
---
apiVersion: v1
kind: Pod
metadata:
  name: busybox-1
  labels:
    app: busybox
spec:
  hostname: busybox-1
  subdomain: default-subdomain
  containers:
  - name: busybox
    image: busybox:1.28
    command: ["sh","-c","sleep 1h"]
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
    ports:
      - containerPort: 80
---
apiVersion: v1
kind: Pod
metadata:
  name: busybox-2
  labels:
    app: busybox
spec:
  hostname: busybox-2
  subdomain: default-subdomain
  containers:
  - name: myapp
    image: busybox:1.28
    command: ["sh","-c","sleep 1h"]
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
    ports:
      - containerPort: 80
```

2. 执行`kubectl create -f dns.yaml`

3. 执行`kubectl exec -it busybox-1 sh` 或者 `kubectl exec -it busybox-2 sh` 进入Pod 容器内部

4. 执行`hostname -f` 查看容器当前完整DNS记录

## 从集群外部访问 Service
> https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#type-nodeport

