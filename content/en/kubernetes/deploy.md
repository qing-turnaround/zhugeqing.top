---
date: 2022-05-01
description: "k8s 部署应用"
image: "images/kuber.jpg"
title: "使用k8s部署应用"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- k8s
series:
- 
---

## 部署Mysql

### 编写Mysql StatefulSet文件
```yaml:mysql.yaml
apiVersion: v1
kind: Service
metadata:
  name: onlineshopping-mysql
  labels:
    app: mysql
spec:
  ports:
  - port: 3306
    name: mysql
    nodePort: 30306
  type: NodePort
  selector:
    app: mysql
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql # has to match .spec.template.metadata.labels
  serviceName: "mysql"
  replicas: 1 # by default is 1
  template:
    metadata:
      labels:
        app: mysql # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: mysql
        image: mysql:5.7
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: host-path
          mountPath: /var/lib/mysql
        env:
          - name: MYSQL_ROOT_PASSWORD
            value: "root" # 密码
      volumes:
        - name: host-path
          hostPath:
              path: /k8s/mysql/ # 本地 目录
              type: DirectoryOrCreate
```

### 创建对应资源
* kubectl apply -f mysql.yaml


## 部署redis