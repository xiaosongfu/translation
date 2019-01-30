---
title: "Firecracker–用于无服务器计算的轻量级虚拟化"
date: 2019-01-29T09:56:55+08:00
draft: false
tags: ["Firecracker", "aws", "Lambda"]
categories: ["Serverless"]
---

> 原文链接：[Firecracker – Lightweight Virtualization for Serverless Computing](https://aws.amazon.com/cn/blogs/aws/firecracker-lightweight-virtualization-for-serverless-computing/)


One of my favorite [Amazon Leadership Principles](https://www.amazon.jobs/en/principles) is Customer Obsession. When we launched [AWS Lambda](https://aws.amazon.com/lambda/), we focused on giving developers a secure [serverless](https://aws.amazon.com/serverless/) experience so that they could avoid managing infrastructure. In order to attain the desired level of isolation we used dedicated EC2 instances for each customer. This approach allowed us to meet our security goals but forced us to make some tradeoffs with respect to the way that we managed Lambda behind the scenes. Also, as is the case with any new AWS service, we did not know how customers would put Lambda to use or even what they would think of the entire serverless model. Our plan was to focus on delivering a great customer experience while making the backend ever-more efficient over time.


我最喜欢的 [亚马逊领导原则](https://www.amazon.jobs/en/principles) 之一是客户痴迷。当我们推出 [AWS Lambda](https://aws.amazon.com/lambda/) 时，我们专注于为开发人员提供安全的 [无服务器](https://aws.amazon.com/serverless/) 体验，以便他们可以避免管理基础架构。为了达到理想的隔离级别，我们为每个客户使用了专用的EC2实例。这种方法使我们能够实现我们的安全目标，但迫使我们在幕后管理Lambda的方式上做出一些权衡。此外，与任何新AWS服务的情况一样，我们不知道客户如何使用Lambda，甚至不知道他们对整个无服务器模型的看法。我们的计划是专注于提供卓越的客户体验，同时使后端随着时间的推移变得更加高效。



---

参考：

1. [宣布推出 Firecracker 开源技术：适用于无服务器计算的安全、快速的 microVM](https://aws.amazon.com/cn/blogs/china/firecracker-open-source-secure-fast-microvm-serverless/)