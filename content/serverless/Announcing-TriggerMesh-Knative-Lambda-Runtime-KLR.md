---
title: "宣布 TriggerMesh Knative Lambda Runtime (KLR)"
date: 2019-01-10T20:36:55+08:00
draft: false
tags: ["TriggerMesh", "Knative", "Lambda"]
categories: ["Serverless"]
---

> 原文链接：[Announcing TriggerMesh Knative Lambda Runtime (KLR)](https://triggermesh.com/2019/01/09/announcing-triggermesh-knative-lambda-runtime-klr/)

今天我们宣布了 TriggerMesh Knative Lambda Runtime（TriggerMesh KLR）开源项目。该项目旨在为 Knative 原生群集和 Knative 启用的无服务器云基础架构提供完整的 Amazon Lambda 功能可移植性，而无需重写这些无服务器函数。Knative 于去年宣布，是由 Google Cloud 主导的基于 Kubernetes 的平台，用于构建，部署和管理现代无服务器工作负载的。

Knative Lambda Runtimes（缩写为 KLR，发音为 clear）是 Knative [构建模板](https://github.com/knative/build-templates)，可用于在使用Knative 安装的 Kubernetes 集群中运行 AWS Lambda 函数。

AWS Lambda 函数运行的执行环境是 AWS Lambda 云环境的克隆，这要归功于自定义 [AWS运行时界面](https://github.com/triggermesh/aws-custom-runtime) 和 [LambCI](https://github.com/lambci/docker-lambda) 项目的一些灵感。

使用这些模板，您可以在 Knative 加持的 Kubernetes 集群中运行 AWS Lambda 函数。

正如我们 [之前所指出的那样](https://triggermesh.com/2018/11/14/serverless-adoption-by-the-numbers/)，无服务器计算正在滚雪球似地迅速增大，并且这种增长正在多个云中发生。我们相信，启用云原生应用程序的关键是在不同的云基础架构中提供真正的可移植性和通信。使用 TriggerMesh KLR，无服务器用户可以在他们的 Knative 和 AWS Lambda 之间来回移动他们的函数。AWS [Lambda Custom Runtime API](https://docs.aws.amazon.com/lambda/latest/dg/runtimes-custom.html) 结合 [Knative Build](https://github.com/knative/build) 系统使 KLR 成为可能。

KLR 朝着 TriggerMesh 的开放云功能迈出的又一步。无服务器用户需要能够将他们的函数移植到他们选择的云中。KLR 将提供一致的执行环境。在将来的功能中，通过使用 Knative eventing，触发器也可以将事件路由到适当的函数。

---

> 参考链接：

* 1. https://github.com/triggermesh/knative-lambda-runtime
* 2. https://github.com/knative/build
* 3. https://github.com/knative/build-templates