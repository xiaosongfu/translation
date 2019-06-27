---
title: "AWS Lambda 宣布支持 Go"
date: 2019-03-16T13:47:55+08:00
draft: false
tags: ["Golang", "Lambda"]
categories: ["Golang"]
---

> 原文链接：[Announcing Go Support for AWS Lambda](https://aws.amazon.com/cn/blogs/compute/announcing-go-support-for-aws-lambda/)

今天，我们很高兴宣布 AWS Lambde 支持 Go 语言了。

作为一个在Go开发中公平分享的人（最近的项目包括AWS SAM Local和GoFormation），这是我一直期待的版本。我将借此机会通过创建Go无服务器应用程序并将其部署到Lambda来引导您了解其工作原理。

## 先决条件

本文假设您已经在开发机器上安装并配置了Go，以及对Go开发概念的基本了解。有关更多详细信息，请参阅  https://golang.org/doc/install。

## 使用 Go 创建 serverless 示例应用

Lambda函数可以由各种事件源触发：

* 异步事件（例如放在Amazon S3存储桶中的对象）
* 流事件（例如，Amazon Kinesis流上的新数据记录）
* 同步事件（通过手动调用或 Amazon API Gateway 发起的 HTTPS 请求）

例如，您将创建一个使用API​​ Gateway事件源的应用程序来创建一个简单的Hello World RESTful API。可以在GitHub上找到此示例应用程序的完整源代码：https://github.com/aws-samples/lambda-go-samples。


