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

### 编写configmap.yaml，存放一些redis相关配置
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: redis
  labels:
    app: redis
data:
  redis-user.conf: |
    # 容器内部端口
    port 6379
    # daemonize yes 

    logfile "$port.log" 
    dbfilename "dump-redis-$port.rdb"

    # 访问redis-server密码
    requirepass 123456


    # 设置redis最大内存
    maxmemory 100MB
    # 设置redis内存淘汰策略
    maxmemory-policy volatile-lru

    # 开启AOF
    appendonly yes
    # 总是追加到AOF文件
    appendfsync always
    # 超过100MB就进行重写
    auto-aof-rewrite-min-size 100mb
    # 超过增长率就进行重写
    auto-aof-rewrite-percentage 40
    # 开启AOF-RDB 混合持久化
    aof-use-rdb-preamble yes

    # 配置慢查询日志（不超过100微秒就不会被记录）
    slowlog-log-slower-than 100
    # 最多记录100条，先进先出
    slowlog-max-len 100
--
```

### 便携部署redis 的StatefulSet 文件
```yaml
apiVersion: v1
kind: Service
metadata:
  name: onlineshopping-redis
  labels:
    app: redis
spec:
  ports:
    - port: 6379
      name: redis-user
      nodePort: 6379
  type: NodePort
  selector:
    app: redis
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis-user
spec:
  selector:
    matchLabels:
      app: redis # has to match .spec.template.metadata.labels
  serviceName: "redis"
  replicas: 1 # by default is 1
  template:
    metadata:
      labels:
        app: redis # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
        - name: redis
          image: zhugeqing/redis
          command: ["sh", "-c", "redis-server", "/etc/redis/redis.conf"]
          ports:
            - containerPort: 6379
              name: redis
          resources:
            limits:
              memory: 300Mi
          volumeMounts:
            - name: conf
              mountPath: /etc/redis/
            - name: data
              mountPath: /data
            - name: time # 同步时间
              mountPath: /etc/localtime
      volumes:
        - name: conf
          configMap:
            name: redis
            items:
              - key: redis-user.conf
                path: redis.conf
        - name: data
          hostPath:
            path: /k8s/redis-user/
            type: DirectoryOrCreate
        - name: time
          hostPath: 
            path: /etc/localtime
```
