---
date: 2022-09-17
description: "实战 k8s RBAC鉴权和StorageClass"
image: "images/kuber.jpg"
title: "k8s RBAC鉴权和StorageClass"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- k8s
series:
- 
---


## RBAC是什么
* RBAC(Role-Based Access control，基于角色的访问控制) 是一种基于角色（用户）来调节控制对资源的访问的方法。

* RBAC中一种拥有四种顶级资源对象：Role，ClusterRole，RoleBinding，ClusterRoleBinding，与其他的APi资源一样，我们可以使用kubectl 或者 API调用来操作这些资源对象

* `role`代表了`一组权限的集合`，这些定义的权限，都是许可的形式，并且范围是在一个命名空间中。如果想要定义一个`集群级别的权限集合` 就需要使用`ClusterRole`

* `roleBinding` 表示 将 `role` 绑定到一个目标上，这个目标可以是User、Group、ServiceAccount。这样就可以给这些目标进行授权，`roleBinding`代表命名空间范围内的授权，`ClusterRoleBinding`为集群范围内授权

## 实战RBAC

### Role
```yaml:role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: role-test
rules:
  - apiGroups: [""] # API 组
    resources: ["pods"] # 资源类型
    verbs: ["get", "watch", "list"] # 能够对资源进行的操作
```

### CLusterRole
```yaml:clusterRole.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  # "namespace" 被忽略，因为 ClusterRoles 不受名字空间限制
  name: clusterrole-test
rules:
- apiGroups: [""]
  # 在 HTTP 层面，用来访问 Secret 资源的名称为 "secrets"
  resources: ["secrets"]
  verbs: ["get", "watch", "list"]
```

### RoleBinding
```yaml:roleBinding.yaml

```
