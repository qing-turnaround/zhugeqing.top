---
date: 2021-12-10
description: "docker 安装常见应用"
image: "images/docker.svg"
title: "使用Docker 安装应用"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- docker
series:
- 
---

## 安装Consul
```shell
docker run --name consul -d -p 8500:8500 -p 8300:8300 -p 8301:8301 -p 8302:8302 -p 8600:8600/udp consul consul agent -dev -client=0.0.0.0 
```
## 安装Elasticsearch

1. 创建相关映射目录
> mkdir -p /docker/elasticsearch/data
> chmod 777 -R /docker/elasticsearch

2. 安装es
```Shell
#!/bin/bash
docker run --name elasticsearch -p 9200:9200 -p 9300:9300 \
-e "discovery.type=single-node" \
-e ES_JAVA_OPTS="-Xms128m -Xmx256m" \
-v es-conf:/usr/share/elasticsearch/config \
-v es-data:/usr/share/elasticsearch/data \
-v es-plugins:/usr/share/elasticsearch/plugins \
-d elasticsearch:7.10.1
```

## 安装Kibana
`docker run -d --name kibana -p 5601:5601 -e "ELASTICSEARCH_HOSTS=http://192.168.200.129:9200" kibana:7.10.1`


## 安装Rocketmq 
```Shell 
#!/bin/bash
function rocketmq_mkdir() {
    mkdir -p /docker/rocketmq/logs /docker/rocketmq/stores /docker/rocketmq/conf/brokerconf 
}


function rocketmq_vim() {

cat > /docker/rocketmq/conf/brokerconf/broker.conf << EOF
brokerClusterName=DefaultCluster
brokerName=broker-a
brokerId=0
# 修改为你宿主机的 IP
brokerIP1=192.168.1.102
defaultTopicQueueNums=4
autoCreateTopicEnable=true
autoCreateSubscriptionGroup=true
listenPort=10911
deleteWhen=04
fileReservedTime=120
mapedFileSizeCommitLog=1073741824
mapedFileSizeConsumeQueue=300000
diskMaxUsedSpaceRatio=88
maxMessageSize=65536
brokerRole=ASYNC_MASTER
flushDiskType=ASYNC_FLUSH
EOF

cat > /docker/rocketmq/docker-compose.yml << EOF
version: '3.5'
services:
  rmqnamesrv:
    image: foxiswho/rocketmq:server
    container_name: rmqnamesrv
    ports:
      - 9876:9876
    volumes:
      - /docker/rocketmq/data/logs:/opt/logs
      - /docker/rocketmq/store:/opt/store
    networks:
        rmq:
          aliases:
            - rmqnamesrv

  rmqbroker:
    image: foxiswho/rocketmq:broker
    container_name: rmqbroker
    ports:
      - 10909:10909
      - 10911:10911
    volumes:
      - /docker/rocketmq/data/logs:/opt/logs
      - /docker/rocketmq/store:/opt/store
      - /docker/rocketmq/conf/brokerconf/broker.conf:/etc/rocketmq/broker.conf
    environment:
        NAMESRV_ADDR: "rmqnamesrv:9876"
        JAVA_OPTS: " -Duser.home=/opt"
        JAVA_OPT_EXT: "-server -Xms128m -Xmx128m -Xmn128m"
    command: mqbroker -c /etc/rocketmq/broker.conf
    depends_on:
      - rmqnamesrv
    networks:
      rmq:
        aliases:
          - rmqbroker

  rmqconsole:
    image: styletang/rocketmq-console-ng
    container_name: rmqconsole
    ports:
      - 8080:8080
    environment:
        JAVA_OPTS: "-Drocketmq.namesrv.addr=rmqnamesrv:9876 -Dcom.rocketmq.sendMessageWithVIPChannel=false"
    depends_on:
      - rmqnamesrv
    networks:
      rmq:
        aliases:
          - rmqconsole

networks:
  rmq:
    name: rmq
    driver: bridge

EOF

}

function rocketmq_start() {
    rocketmq_mkdir
    rocketmq_vim
    cd /docker/rocketmq
    docker-compose up
}

rocketmq_start
```

## 安装Jaeger
```Shell
docker run -d --name jaeger -p 6831:6831/udp -p 16686:16686 jaegertracing/all-in-one
```

## 安装Etcd 
```Shell
docker run -d --name etcd \
  -p 2379:2379 \
  -p 2380:2380 \
  --env ALLOW_NONE_AUTHENTICATION=yes \
  --env ETCD_ADVERTISE_CLIENT_URLS=http://0.0.0.0:2379 \
  bitnami/etcd:latest
```