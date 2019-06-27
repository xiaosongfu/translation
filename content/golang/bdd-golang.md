---
title: "Golang 中的 BDD"
date: 2019-03-19T16:59:09+08:00
draft: false
tags: ["Golang", "BDD", "Test"]
categories: ["Golang"]
---

> 原文链接：[BDD in Golang](https://alicegg.tech/2019/03/09/gobdd.html)

在我看来，行为驱动开发（BDD）是解决具有复杂业务逻辑的项目的最佳开发实践之一。BDD旨在通过为规范和开发创建通用（自然）语言来帮助团队的技术和非技术成员之间的通信。在尝试实现领域驱动方法时，它特别有用，因为它将帮助开发人员与客户共享对业务逻辑的共同理解。

作为Go开发人员，我很高兴地发现由于使用了 [godog](https://github.com/DATA-DOG/godog) 包，BDD测试很容易实现，我将尝试向您展示如何将它集成到您​​的测试中。