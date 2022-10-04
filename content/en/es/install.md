---
date: 2022-09-17
description: "安装ELK"
image: "images/es.png"
title: "安装ELK"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- elasticsearch
series:
- 
---

## 使用docker-compose安装ELK

1. 编写docker-compose.yaml
```yaml:docker-compose.yaml
# 声明版本
version: "3"
services:
  elasticsearch:
    image: zhugeqing/elasticsearch:7.9.3 # 可以自行修改版本，不过ELK组件的版本都要一致
    ports:
      - "9200:9200"
      - "9300:9300"
    volumes:
      - ./elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearchs/config/elasticsearch.yml
    environment:
      ES_JAVA_OPTS: "-Xmx256m -Xms256m"
      ELASTIC_PASSWORD: zhugeqing
      discovery.type: single-node
      network.publish_host: _eth0_
  logstash:
    image: zhugeqing/logstash:7.9.3
    ports:
      - "5044:5044"
      - "5000:5000"
      - "9600:9600"
    volumes:
      - ./logstash/config/logstash.yml:/usr/share/logstash/config/logstash.yml
      - ./logstash/pipeline/logstash.conf:/usr/share/logstash/pipeline/logstash.conf
      - ./logstash/movies.csv:/usr/share/logstash/movies.csv
    environment:
      LS_JAVA_OPTS: "-Xmx256m -Xms256m"
  kibana:
    image: zhugeqing/kibana:7.9.3
    ports:
      - "5601:5601"
    volumes:
      - ./kibana/config/kibana.yml:/usr/share/kibana/config/kibana.yml
```

2. 在创建docker-compose.yaml所在的目录下，创建三个子目录：elasticsearch，kibana，logstash。

3. 在elasticsearch目录下创建elasticsearch.yaml
```yaml:elasticsearch.yaml
cluster.name: "elasticsearch"
network.host: 0.0.0.0

xpack.license.self_generated.type: trail
xpack.security.enabled: true
xpack.monitoring.collector.enabled: true
```

4. 在kibana目录下创建kibana.yaml
```yaml:kibana.yaml
---
server.name: kibana
server.host: 0.0.0.0
elasticsearch.hosts: ["http://elasticsearch:9200"]
monitoring.ui.container.elasticsearch.enabled: true

elasticsearch.username: elastic
elasticsearch.password: zhugeqing
i18n.locale: zh-CN # 设置中文插件
```

5. 在logstash目录下创建config、pipeline目录；并在config下创建logstash.yaml，在pipeline下创建logstash.config
```yaml:logstash.yaml
---
http.host: "0.0.0.0"
xpack.monitoring.elasticsearch.hosts: ["http://elasticsearch:9200"]

xpack.monitoring.enabled: true
xpack.monitoring.elasticsearch.username: elastic
xpack.monitoring.elasticsearch.password: zhugeqing
```

6. 先下载需要导入到logstash中的测试数据
> https://grouplens.org/datasets/movielens

* wget https://files.grouplens.org/datasets/movielens/ml-10m.zip 

* 进行解压，修改解压目录明为movietest(放在./logstash下)，然后编写logstash.conf文件。

```shell
input {
	beats {
		port => 5044
	}
	tcp {
		port => 5000
	}  
  file {
    path => "/usr/share/logstash/movies.csv"
    start_position => "beginning"
    sincedb_path => "/dev/null"
  }
}

output {
	elasticsearch {
		hosts => "elasticsearch:9200"
		user => "elastic"
		password => "zhugeqing"
		index => "%{[@metadata][-zhugeqing]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
	}
}
```

7.在docker-compose.yaml下，执行docker-compose up -d。

### tips
* 最终宿主机的目录树形结构如下
```shell
.
|-- docker-compose.yml
|-- elasticsearch
|   `-- config
|       `-- elasticsearch.yml
|-- kibana
|   `-- config
|       `-- kibana.yml
`-- logstash
    |-- config
    |   `-- logstash.yml
    |-- movietest
    |   |-- allbut.pl
    |   |-- movies.dat
    |   |-- ratings.dat
    |   |-- README.html
    |   |-- split_ratings.sh
    |   `-- tags.dat
    `-- pipeline
        `-- logstash.conf
```

### 安装分词器
* 进入es容器的/usr/share/elasticsearch目录，比如安装analysis-icu

* 命令行安装：可以执行`bin/elasticsearch-plugin install analysis-icu`
* url安装：可以执行`bin/elasticsearch-plugin install [url]`
> url填所需要哦分词器的地址，需要匹配版本
* 离线安装，将下载的分词器解压缩 到/usr/share/elasticsearch/plugins下