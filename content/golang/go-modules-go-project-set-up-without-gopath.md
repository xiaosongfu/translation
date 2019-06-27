---
title: "Go Module: 项目配置不再需要 GOPATH"
date: 2019-04-24T09:59:09+08:00
draft: false
tags: ["Golang", "Go Module", "GOPATH"]
categories: ["Golang"]
---

> 原文链接：[Go Modules : Project set up without GOPATH](https://blog.francium.tech/go-modules-go-project-set-up-without-gopath-1ae601a4e868)

---

Go setup 不是这里要讨论的新话题。但是在 Go 1.12发布之后，我认为必须重新定义步骤，因为项目设置变得比以前简单明了。

在搜索步骤时，除了少数 Go Blog 之外，大部分内容仍然是分享旧方式，即

* 设置GOPATH 环境变量
* 在GOPATH中创建项目以解析自定义包路径。
* 使用Go dep或其他外部工具（如 golide）管理依赖
* 依赖包已安装在同一工作区中。

通过本文，我们将看到项目设置及其发布如何在Go 1.12版本中进行。

在1.11版本中，Go 引入了名为 [Go Modules](https://github.com/golang/go/wiki/Modules) 的内置包管理，这是对Go生态系统巨大变革的开始。它是GOPATH 的替代品，集成了版本控制和软件包分发支持。

来自 [Go Blog](https://blog.golang.org/modules2019)

> 在我们使用GOPATH的八年中，创建了大量的工具，假设Go源代码存储在GOPATH中。
迁移到 Go Module 将是Go生态系统中从 Go 1 开始影响最深远的变化。将整个生态系统-代码，用户，工具等-从 GOPATH 转换为 Module 将需要在许多不同领域开展工作。

由于1.12版本的 Go Module 默认启用，GOPATH将在1.13版本中弃用。

对于那些开始使用Go 1.12的人来说，安装和设置将如下所示。

## 安装Go
### 在Mac上

```
brew install go
```

###在Ubuntu上

```
curl -O https://dl.google.com/go/go1.12.4.linux-amd64.tar.gz
tar -xvf go1.12.4.linux-amd64.tar.gz
sudo mv go /usr/local
```

从Go 1.8开始的将GOPATH设置为环境变量不是必需的。如果我们没有设置，Go 将使用默认的 GOPATH `$HOME/go`

## 设置 Go 项目

让我们在GOPATH之外的首选位置为go项目创建文件夹

注意： Go模块在GOPATH中被禁用。所以我们不能在GOPATH位置内部创建项目$HOME/go
mkdir firstgo_app
cd firstgo_app

### 启动模块

Go模块以模块或项目名称启动。

go mod init firstgo_app
这将创建模块配置文件go.mod，其中包含模块名称和版本。




