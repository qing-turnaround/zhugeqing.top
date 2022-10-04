---
date: 2022-09-16
description: "实战 k8s PV 和 PVC"
image: "images/kuber.jpg"
title: "实战k8s PV 和 PVC"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- k8s
series:
- 
---

## 应对问题
* Volume 和 Pod的生命周期是绑定的，当Pod删除后，Volume里面的数据也有可能被一同删除。kubernetes 提供了PV（Persistent Volume）用于持久化存储数据。但是光有PV还不能直接使用，PV需要与PVC绑定起来，PVC就像我们面向对象中的接口，它声明的一些方法（也就是关于PV的规范），而PV则是具体实现的对象。
> https://kubernetes.io/zh-cn/docs/concepts/storage/persistent-volumes/
> https://kubernetes.io/zh-cn/docs/reference/kubernetes-api/config-and-storage-resources/persistent-volume-v1/

## 实战

1. 创建PVC
```yaml:pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-test
spec:
  accessModes:
    - "ReadWriteOnce"
  resources:
    requests:
      storage: 400M
```

2. 创建pv
```yaml:pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-test
spec:
  capacity:
    storage: 500M # 容量
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce # 访问权限
  persistentVolumeReclaimPolicy: Retain # 回收策略
  nodeAffinity: # 节点调度
    required:
      nodeSelectorTerms:
        - matchExpressions:
            - key: kubernetes.io/hostname
              operator: In
              values:
                - "node1"
  local:
    path: /k8s/pvtest/ # 绑定目录（存储插件选择local）
```

3. 创建Pod
```yaml:pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
    - name: test-pod
      image: nginx
      command:
        - "/bin/sh"
      args:
        - "-c"
        - "touch /mnt/SUCCESS && exit 0 || exit 1" #创建一个SUCCESS文件后退出
      volumeMounts:
        - name: pv-pvc-test
          mountPath: "/mnt"
  restartPolicy: "Never"
  volumes:
    - name: pv-pvc-test
      persistentVolumeClaim:
      claimName: pvc-test #与PVC名称保持一致
```


4. 依次使用kubelet apply yaml文件。

