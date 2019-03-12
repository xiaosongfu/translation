---
title: "使用 Go Modules"
date: 2019-01-30T16:56:11+08:00
draft: false
tags: ["Golang", "Go Modules"]
categories: ["Golang"]
---

> 原文链接：[Working with Go Modules](https://blog.jetbrains.com/go/2019/01/22/working-with-go-modules/)

在这篇博文中，我们将探讨如何在创建新的 Go 项目或使用现有项目时使用 Go Modules。

首先，让我们通过从欢迎屏幕中选择新建项目来创建一个新项目。然后选择Go Modules（vgo）作为项目类型。然后，我们可以将项目的位置（现在可以在GOPATH之外设置）指定到系统中的任何目录。确保GOROOT指向一个1.11或更高版本的Go SDK。虽然可以使用安装了特殊vgo二进制文件的Go 1.10，但不建议这样做，因为vgo不会经常更新以与Go版本相提并论，并且可能在混合Go版本环境中产生无法预料的影响。

最后，我们可以使用 `Proxy` 字段指定是否要直接从Internet下载Go Modules（或软件包）（默认设置），或通过代理系统（如Athens或JFrog的Go Registry）下载，以防您想要存储在一个中心位置的依赖关系，团队中的每个人都可以在更好管理的环境中访问相同的版本。

创建项目后，我们可以看到它已经包含一个go.mod文件。我们可以编辑文件来更改模块名称以更好地满足我们的需求。例如，我们将在github.com/JetBrains/go-samples下发布，以便其他人可以在其代码中使用import语句 `import“github.com/JetBrains/go-samples”` 。

接下来，像以前一样创建Go文件，因此这里没有任何变化。我们可以使用Simple Application类型而不是Empty File来直接使用样板代码。

在编写一个小的HelloWorld函数之后，假设我们想要使用流行的 `github.com/pkg/errors` 包处理代码中的错误。要导入此依赖项，请编写import语句 `import“github.com/pkg/errors”` 并使用github.com/JetBrains/go-samples快速修复的同步包。这将运行所需的Go命令以下载并安装所需的依赖项版本。

在撰写本文时，github.com/pkg/errors的版本为0.8.1，因此我们将安装此版本，因为我们没有在go.mod文件中指定任何内容。如果我们想要使用版本0.8.0，我们可以手动编辑go.mod文件，这将触发刷新依赖项并使用预期版本。

如果您不想两次下载依赖项，如上所述，那么您可以先编辑go.mod文件并在其末尾添加以下行， `require github.com/pkg/errors v0.8.0`然后转到Go文件并添加import语句 `import“github.com/pkg/errors”` 并调用快速修复程序以下载依赖项。