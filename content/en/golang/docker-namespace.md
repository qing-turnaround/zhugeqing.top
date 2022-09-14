---
date: 2022-06-11
description: "docker Namespace"
image: "/images/Go.jpg"
title: "docker Namespace"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- Go
series:
- 
---

## Namespace

* 对于docker而言，本身只是通过Linux的Namespace技术来将容器作为一个进程 与其他进程进行隔离。

* Namespace 技术实际上修改了应用进程看待整个计算机“视图”，即它的“视线”被操作系统做了限制，只能“看到”某些指定的内容。

* 使用UTS Namespace就可以隔离nodename和domainname；使用IPC Namespace就可以隔离进程间通信，使用USer Namespace就可以隔离用于和用户组的ID，使用Network Namespace就可以隔离网络资源，使用PID Namespace可以隔离进程ID，使用Mount Namespace可以隔离挂载点

* 下面使用go语言来使用这个隔离的系统调用（切换成Linux模式进行开发，不然有些变量没有提示）


### UTS
```go:main.go
package main

import (
	"log"
	"os"
	"os/exec"
	"syscall"
)

// UTS Namespace主要用于隔离nodeName 和 domainName
func uts_proc() {
	cmd := exec.Command("sh")
	cmd.SysProcAttr = &syscall.SysProcAttr{
		Cloneflags: syscall.CLONE_NEWUTS,
	}

	cmd.Stdin = os.Stdin
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr

	if err := cmd.Run(); err != nil {
		log.Fatal(err)
	}
}
```

### IPC
```go:main.go
package main

import (
	"log"
	"os"
	"os/exec"
	"syscall"
)

// IPC Namespace 用于隔离进程间通信
func ipc_proc() {
	cmd := exec.Command("sh")
	cmd.SysProcAttr = &syscall.SysProcAttr{
		Cloneflags: syscall.CLONE_NEWUTS | syscall.CLONE_NEWIPC,
	}

	cmd.Stdin = os.Stdin
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr

	if err := cmd.Run(); err != nil {
		log.Fatal(err)
	}
}
```

### PID

```go:main.go
package main

import (
	"log"
	"os"
	"os/exec"
	"syscall"
)

// PID Namespace 用于隔离 进程ID
func pid_proc() {
	cmd := exec.Command("sh")
	cmd.SysProcAttr = &syscall.SysProcAttr{
		Cloneflags: syscall.CLONE_NEWUTS | syscall.CLONE_NEWIPC | syscall.CLONE_NEWPID,
	}

	cmd.Stdin = os.Stdin
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr

	if err := cmd.Run(); err != nil {
		log.Fatal(err)
	}
}
```

### Mount
```go:main.go
package main

import (
	"log"
	"os"
	"os/exec"
	"syscall"
)

// Mount Namespace 用于隔离 挂载点
func mount_proc() {
	cmd := exec.Command("sh")
	cmd.SysProcAttr = &syscall.SysProcAttr{
		Cloneflags: syscall.CLONE_NEWUTS | syscall.CLONE_NEWIPC | syscall.CLONE_NEWPID |
			syscall.CLONE_NEWNS,
	}

	cmd.Stdin = os.Stdin
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr

	if err := cmd.Run(); err != nil {
		log.Fatal(err)
	}
}
```


### User
```go:main.go
package main

import (
	"log"
	"os"
	"os/exec"
	"syscall"
)

// user Namespace 用于隔离 用户 和 用户组（运行之后使用id查看）
func user_proc() {
	cmd := exec.Command("sh")
	cmd.SysProcAttr = &syscall.SysProcAttr{
		Cloneflags: syscall.CLONE_NEWUTS | syscall.CLONE_NEWIPC | syscall.CLONE_NEWPID |
			syscall.CLONE_NEWNS,
	}

	cmd.Stdin = os.Stdin
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr
	cmd.SysProcAttr.Credential = &syscall.Credential{
		Uid:         uint32(1),
		Gid:         uint32(1),
	}

	if err := cmd.Run(); err != nil {
		log.Fatal(err)
	}
}
```

### Network
```go:main.go
package main

import (
	"log"
	"os"
	"os/exec"
	"syscall"
)

// Network Namespace 用于隔离 网络资源
func network_proc() {
	cmd := exec.Command("sh")
	cmd.SysProcAttr = &syscall.SysProcAttr{
		Cloneflags: syscall.CLONE_NEWUTS | syscall.CLONE_NEWIPC | syscall.CLONE_NEWPID |
			syscall.CLONE_NEWNS | syscall.CLONE_NEWNET,
	}

	cmd.Stdin = os.Stdin
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr
	cmd.SysProcAttr.Credential = &syscall.Credential{
		Uid:         uint32(1),
		Gid:         uint32(1),
	}

	if err := cmd.Run(); err != nil {
		log.Fatal(err)
	}
}
```

