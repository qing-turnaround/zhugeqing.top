---
date: 2022-09-17
description: "实战 k8s CRD开发"
image: "images/kuber.jpg"
title: "k8s CRD"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- k8s
series:
- 
---

## 为什么需要CRD
> https://kubernetes.io/zh-cn/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/
* 虽然kubernetes为我们提供了非常多的Api对象，但是我们生产环境中总有一些特殊情况，而CRD就可以用于自定义api对象来应对这些“特殊情况”


## 声明CRD对象
> 实战的第一步：定义好需要的CRD对象
1. 创建CRD对象的yaml文件
```yaml:crd.yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  # 名字必需与下面的 spec 字段匹配，并且格式为 '<名称的复数形式>.<组名>'
  name: networks.testcrd.k8s.io
spec:
  # 组名称，用于 REST API: /apis/<组>/<版本>
  group: testcrd.k8s.io
  # 列举此 CustomResourceDefinition 所支持的版本
  versions:
    - name: v1
      # 每个版本都可以通过 served 标志来独立启用或禁止
      served: true
      # 其中一个且只有一个版本必需被标记为存储版本
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                cronSpec:
                  type: string
                image:
                  type: string
                replicas:
                  type: integer
  # 可以是 Namespaced 或 Cluster
  scope: Namespaced
  names:
    # 名称的复数形式，用于 URL：/apis/<组>/<版本>/<名称的复数形式>
    plural: network
    # 名称的单数形式，作为命令行使用时和显示时的别名
    singular: network
    # kind 通常是单数形式的驼峰命名（CamelCased）形式。你的资源清单会使用这一形式。
    kind: Network
    # shortNames 允许你在命令行使用较短的字符串来匹配资源
    shortNames:
    - ct
```


## 编写CRD 控制器

### 实现main函数逻辑
