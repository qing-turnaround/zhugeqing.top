---
date: 2022-04-02
description: "k8 持久化存储 学习"
image: "images/kuber.jpg"
title: "k8 持久化存储 学习"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- k8s
series:
- 
---

## volume
> https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/

## 实战挂载NFS（Network File System）卷

1. 在 master 和 worker node 上安装 nfs服务
`yum install -y nfs-utils`

2. 修改 master 配置
`echo "/nfsdata  *(rw,sync,no_root_squash)" > /etc/exports`

3. 配置 master nfs 自启动
`systemctl enable --now rpcbind`
`systemctl enable --now nfs`

4. 创建`nfs.yaml` 测试nfs挂载
> 需要先在 master上创建对应的 /nfsdata 
```yaml:nfs.yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  labels:
    name: myapp
spec:
  containers:
  - name: myapp
    image: nginx
    volumeMounts:
      - mountPath: /usr/share/nginx/html
        name: nfs
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
    ports:
      - containerPort: 80
  volumes:
    - name: nfs
      nfs:
        server: master
        path: /nfsdata
```
5. `kubectl apply -f nfs.yaml` 创建pod

6. `kubectl exec -it myapp sh` 进入容器内部

7. `echo "zhugeqing" /usr/share/nginx/html/a.txt` 创建文件

8. 回到master，进入/nfsdata目录查看文件是否存在

## 实战pv（Persistent Volume） 和 pvc（PersistentVolumeClaims）
> https://kubernetes.io/zh-cn/docs/concepts/storage/persistent-volumes/

1. 创建 pv.yaml
```yaml:pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-demo
  labels:
    name: pv-demo
spec:
  capacity: 
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  nfs:
    path: /nfsdata
    server: master
```

2. `kubectl apply -f pv.yaml` 创建pv

3. 创建 pvc.yaml
```yaml:pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-demo
  labels:
    name: pvc-demo
spec:
  accessModes:
    - "ReadWriteOnce"
  resources:
    requests:
      storage: 2Gi
```

4. `kubectl apply -f pvc.yaml` 创建pvc

5. `kubectl get pv`，`kubectl get pvc`查看

## 实战storage Class
> https://kubernetes.io/zh-cn/docs/concepts/storage/storage-classes/

### 创建 Persistent Volume Provisioner
```yaml:nfs-pvp.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nfs-client-provisioner
  labels:
    app: nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: default  #与RBAC文件中的namespace保持一致
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nfs-client-provisioner
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: nfs-client-provisioner
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccountName: nfs-client-provisioner
      containers:
        - name: nfs-client-provisioner
          image: registry.cn-beijing.aliyuncs.com/qingfeng666/nfs-client-provisioner:v3.1.0
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              value: qgg-nfs-storage  #provisioner名称,请确保该名称与 nfs-StorageClass.yaml文件中的provisioner名称保持一致
            - name: NFS_SERVER
              value: master   #NFS Server IP地址
            - name: NFS_PATH
              value: /nfsdata   #NFS挂载卷
      volumes:
        - name: nfs-client-root
          nfs:
            server: master  #NFS Server IP地址
            path: /nfsdata    #NFS 挂载卷
```

### 创建用户和角色
```yaml:rbac.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: default        #根据实际环境设定namespace,下面类同
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-client-provisioner-runner
rules:
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["create", "update", "patch"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: run-nfs-client-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    # replace with namespace where provisioner is deployed
    namespace: default
roleRef:
  kind: ClusterRole
  name: nfs-client-provisioner-runner
  apiGroup: rbac.authorization.k8s.io
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
    # replace with namespace where provisioner is deployed
  namespace: default
rules:
  - apiGroups: [""]
    resources: ["endpoints"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    # replace with namespace where provisioner is deployed
    namespace: default
roleRef:
  kind: Role
  name: leader-locking-nfs-client-provisioner
  apiGroup: rbac.authorization.k8s.io
```

### 创建 storage class
```yaml:storageClass.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: managed-nfs-storage
provisioner: qgg-nfs-storage #这里的名称要和provisioner配置文件中的环境变量PROVISIONER_NAME保持一致
parameters:
  archiveOnDelete: "false"
```

### 创建 pvc
```yaml:pvc2.yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: test-claim
  annotations:
    volume.beta.kubernetes.io/storage-class: "managed-nfs-storage"   #与nfs-StorageClass.yaml metadata.name保持一致
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Mi
```

### 创建 一个 Pod测试
```yaml:pod.yaml
kind: Pod
apiVersion: v1
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
      - "touch /mnt/SUCCESS && exit 0 || exit 1"   #创建一个SUCCESS文件后退出
    volumeMounts:
      - name: nfs-pvc
        mountPath: "/mnt"
  restartPolicy: "Never"
  volumes:
    - name: nfs-pvc
      persistentVolumeClaim:
        claimName: test-claim  #与PVC名称保持一致
```