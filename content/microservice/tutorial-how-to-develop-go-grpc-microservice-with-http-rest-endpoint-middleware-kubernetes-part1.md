---
title: "[教程,第1部分]如何使用 HTTP/REST 端点，中间件，Kubernetes deployment 等开发 Go gRPC 微服务"
date: 2019-04-19T10:54:48+08:00
draft: false
tags: ["microservice"]
categories: ["Microservice"]
---

> 原文链接：[[Tutorial, Part 1] How to develop Go gRPC microservice with HTTP/REST endpoint, middleware, Kubernetes deployment, etc.](https://medium.com/@amsokol.com/tutorial-how-to-develop-go-grpc-microservice-with-http-rest-endpoint-middleware-kubernetes-daebb36a97e9)


中文：https://mp.weixin.qq.com/s/MSd-ZSLU50mouVprUqaC5A

---

![]()

有很多文章介绍了如何使用一些优秀的 Web 框架和/或路由(web frameworks and/or routers) 创建 Go REST 微服务。当我为公司寻找最佳方法时，我阅读了大部分内容。突然间，我发现了另一种非常有趣的方法来开发 HTTP/REST 微服务。它是来自 Google 的 protobuf/gRPC 框架。我相信大家都知道。有人正在使用 gRPC。但我相信没有那么多人有使用 protobuf gRPC 开发 HTTP/REST 微服务的经验。我发现只有一篇实际的 Medium 文章：[Why choose between gRPC and REST?](https://medium.com/@thatcher/why-choose-between-grpc-and-rest-bc0d351f2f84)。

我不打算重复这篇伟大的文章。我想提供一步一步的教程，如何使用gRPC和HTTP/REST端点开发简单的CRUD“待办事项列表”微服务。我演示了如何编写测试并将中间件（请求ID和日志记录/跟踪）添加到微服务中。我提供了如何在最后构建和部署这个微服务到Kubernetes的示例。

## 内容列表

文章由4部分组成：

* 这第一部分是关于如何创建GRPC CRUD服务和客户端
* 第2部分是关于如何将HTTP/REST端点添加到gRPC服务
* 第3部分是关于如何向gRPC服务和HTTP/REST端点添加中间件（例如，日志记录/跟踪）
* 第4部分将专门介绍如何添加Kubernetes部署配置并且进行运行状态检查以及如何构建项目并将其部署到 Google Cloud

## 先决条件

* 本文不是Go语言的培训材料。我假设你已经有了一些经验。
* 您必须在开始之前安装并配置Go v1.11。我们将使用Go模块功能。
* 您必须具备如何安装/配置任何SQL数据库以将其用作本教程的持久存储的经验

## API first

这对我意味着什么？

* API定义必须是语言，协议，传输中立
* API定义和API实现必须松散耦合
* API版本控制
* 我需要排除手动工作以同步API定义，API实现和API文档。我需要API实现存根/骨架和API文档自动从API定义生成。

我在教程中强调了这一点。

## “待办事项列表”微服务

“待办事项列表”微服务允许管理“待办事项”项目。待办事项包含以下字段：

* ue integer identifier)
* Title (text)
* Description (text)
* Reminder (timestamp)

ToDo服务还包含经典的CRUD方法Create，Read，Update，Delete和ReadAll方法。

## 第1部分：创建gRPC CRUD服务

#### 第1步：创建API定义

第1部分是关于如何开发gRPC CRUD服务和客户端来测试它。

> 第1部分的源代码可在此处获得：[](https://github.com/amsokol/go-grpc-http-rest-microservice-tutorial/tree/part1)

在开始之前，我们需要创建Go项目结构。

这里有一个很棒的Go项目模板：[](https://github.com/golang-standards/project-layout)

请像我一样使用它！

> 我使用的是Windows 10 x64环境。不过我认为将CMD命令转换为MacOS / Linux BASH并不是问题。

首先创建并输入root项目文件夹go-grpc-http-rest-microservice-tutorial（在GOPATH之外找到它以使用Go模块）。比初始化Go项目：

```
mkdir go-grpc-http-rest-microservice-tutorial

cd go-grpc-http-rest-microservice-tutorial

go mod init github.com/<you>/go-grpc-http-rest-microservice-tutorial
```

为API定义创建文件夹结构：

```
mkdir -p api\proto\v1
```

其中v1是API版本。

> API版本控制：我的最佳做法是在不同的文件夹中找到主要版本的API。

接下来在api \ proto \ v1文件夹中创建todo-service.proto文件，并添加ToDoService定义，只有一个方法 Create 作为开头：

```
syntax = "proto3";
package v1;

import "google/protobuf/timestamp.proto";

// Taks we have to do
message ToDo {
    // Unique integer identifier of the todo task
    int64 id = 1;
    // Title of the task
    string title = 2;
    // Detail description of the todo task
    string description = 3;
    // Date and time to remind the todo task
    google.protobuf.Timestamp reminder = 4;
}

// Request data to create new todo task
message CreateRequest{
    // API versioning: it is my best practice to specify version explicitly
    string api = 1;

    // Task entity to add
    ToDo toDo = 2;
}

// Response that contains data for created todo task
message CreateResponse{
    // API versioning: it is my best practice to specify version explicitly
    string api = 1;

    // ID of created task
    int64 id = 2;
}

// Service to manage list of todo tasks
service ToDoService {
    // Create new todo task
    rpc Create(CreateRequest) returns (CreateResponse);
}
```

> 你可以在这里获得proto语言规范：[](https://developers.google.com/protocol-buffers/docs/proto3)

> 正如您所看到的，我们的API定义绝对是语言，协议，传输中立。这是protobuf的关键特性之一。

要编译Proto文件，我们需要安装必要的工具并添加包。

* 在这里下载Proto编译器二进制文件：[](https://github.com/protocolbuffers/protobuf/releases)
* 将包解压缩到PC上的任何文件夹，并将“ bin ” 添加到PATH环境变量中
* 在“ go-grpc-http-rest-microservice-tutorial ”中创建“ third_party ”文件夹
* 将所有内容从Proto编译器“ include ”文件夹复制到“ third_party ”文件夹：

![]()

* 

```
go get -u github.com/golang/protobuf/protoc-gen-go
```


#### 第2步：


#### 第3步：

#### 第4步：


#### 第5步：

#### 第6步：