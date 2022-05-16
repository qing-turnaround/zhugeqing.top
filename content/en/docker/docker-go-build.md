---
date: 2022-04-02
description: "使用docker正确的构建go程序镜像"
image: "images/docker.svg"
title: "docker 构建go镜像"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- docker
series:
- 
---

## 构建最小的go程序镜像
> 我一般通常会在云服务器上安装Go，然后build源程序，最后通过在busybo中进行运行 来构建go程序镜像

* 现在删除Go，用Go image来构建吧！

```dockerfile
FROM zhugeqing/golang:1.17  AS builder
WORKDIR /app
MAINTAINER 诸葛青

COPY . .
ARG TARGETPLATFORM="linux/amd64"
# 设置代理环境变量
ARG GOPROXY="https://goproxy.cn,direct"
# 编译，在image内运行
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o main main.go

FROM zhugeqing/alpine 
# 拷贝缓存区构建完成的文件（这样镜像还是非常小的）
COPY --from=builder /app/main ./main 
# 暴露端口，只是显示而已
EXPOSE 6666 
# 运行 可执行文件
CMD [ "/main" ]
```

* 缺点就是会产生缓存image，需要执行`docker system prune`进行清理 
