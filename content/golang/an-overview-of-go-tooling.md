---
title: "Go 工具概述"
date: 2019-04-29T16:47:55+08:00
draft: false
tags: ["tag1", "tag2", "tag3"]
categories: ["cat"]
---

> 原文链接：[An Overview of Go's Tooling](https://www.alexedwards.net/blog/an-overview-of-go-tooling)

---


偶尔我会被问到“你为什么喜欢使用Go？”而且我经常提到的一件事就是作为go命令的一部分与语言一起存在的有思想的工具。我每天都使用一些工具 - 比如go fmt和go build- 以及其他类似go tool pprof我只用来帮助解决特定问题的工具。但在所有情况下，我都很欣赏他们使我的项目管理和维护更容易的事实。

在这篇文章中，我希望提供一些关于我认为最有用的工具的背景和背景，更重要的是，解释它们如何适应典型项目的工作流程。如果你是Go的新手，我希望它会给你一个良好的开端。

或者如果你已经和Go一起工作了一段时间，那些东西不适用于你，希望你仍然会发现一个你以前不知道的命令或 flag :)