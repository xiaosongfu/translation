---
title: "使用 Go Modules"
date: 2019-01-30T16:56:11+08:00
draft: false
tags: ["Golang", "Go Modules"]
categories: ["Golang"]
---

> 原文链接：[Working with Go Modules](https://blog.jetbrains.com/go/2019/01/22/working-with-go-modules/)

In this blog post, we’ll explore how to work with Go Modules when creating new Go projects or working with existing ones.

在这篇博文中，我们将探讨如何在创建新的 Go 项目或使用现有项目时使用 Go Modules。

First, let’s create a new project by selecting New Project from the Welcome Screen. Then choose Go Modules (vgo) as the project type. We can then specify the location of the project, which can now be set outside of the GOPATH, to any directory in your system.
Ensure that GOROOT points to a Go SDK which is 1.11 or newer. While Go 1.10 with the special vgo binary installed can be used, this is not recommended as vgo is not updated as frequently to be on par with the Go releases and might have an unforeseen impact in a mixed Go versions environment.

首先，让我们通过从欢迎屏幕中选择新建项目来创建一个新项目。然后选择Go Modules（vgo）作为项目类型。然后，我们可以将项目的位置（现在可以在GOPATH之外设置）指定到系统中的任何目录。确保GOROOT指向一个1.11或更高版本的Go SDK。虽然可以使用安装了特殊vgo二进制文件的Go 1.10，但不建议这样做，因为vgo不会经常更新以与Go版本相提并论，并且可能在混合Go版本环境中产生无法预料的影响。

Finally, we can use the `Proxy` field to specify if we want to download the Go Modules (or packages) directly from the Internet (default setting), or via a proxy system such as Athens or JFrog’s Go Registry in case you want to store the dependencies in a central place for everyone in the team to have access to the same versions in a better-managed environment.

最后，我们可以使用 `Proxy` 字段指定是否要直接从Internet下载Go Modules（或软件包）（默认设置），或通过代理系统（如Athens或JFrog的Go Registry）下载，以防您想要存储在一个中心位置的依赖关系，团队中的每个人都可以在更好管理的环境中访问相同的版本。

After the project is created, we can see that it already contains a go.mod file. We can edit the file to change the module name to better suit our needs. For example, we’ll publish this under github.com/JetBrains/go-samples so others can use the import statement `import "github.com/JetBrains/go-samples"` in their code.

创建项目后，我们可以看到它已经包含一个go.mod文件。我们可以编辑文件来更改模块名称以更好地满足我们的需求。例如，我们将在github.com/JetBrains/go-samples下发布，以便其他人可以在其代码中使用import语句 `import“github.com/JetBrains/go-samples”` 。

接下来，像以前一样创建Go文件，因此这里没有任何变化。我们可以使用Simple Application类型而不是Empty File来直接使用样板代码。

After writing a small HelloWorld function, let’s say we want to handle errors in our code using the popular `github.com/pkg/errors` package. To import this dependency, write the import statement `import "github.com/pkg/errors"` and use the Sync packages of github.com/JetBrains/go-samples quick-fix on it. This will run the required Go commands to download and install the desired version of our dependency.

在编写一个小的HelloWorld函数之后，假设我们想要使用流行的 `github.com/pkg/errors` 包处理代码中的错误。要导入此依赖项，请编写import语句 `import“github.com/pkg/errors”` 并使用github.com/JetBrains/go-samples快速修复的同步包。这将运行所需的Go命令以下载并安装所需的依赖项版本。

At the time of writing, github.com/pkg/errors is at version 0.8.1, so this version will be installed as we did not specify anything in our go.mod file. If we want to use version 0.8.0 instead, we can edit the go.mod file manually, which will trigger a refresh of the dependencies and use the expected version.

在撰写本文时，github.com/pkg/errors的版本为0.8.1，因此我们将安装此版本，因为我们没有在go.mod文件中指定任何内容。如果我们想要使用版本0.8.0，我们可以手动编辑go.mod文件，这将触发刷新依赖项并使用预期版本。

If you don’t want to download the dependencies twice, as described above, then you can first edit the go.mod file and add the following line at the end of it `require github.com/pkg/errors v0.8.0` then go to the Go file and add the import statement `import "github.com/pkg/errors"` and invoke the quick-fix to download the dependency.

如果您不想两次下载依赖项，如上所述，那么您可以先编辑go.mod文件并在其末尾添加以下行， `require github.com/pkg/errors v0.8.0`然后转到Go文件并添加import语句 `import“github.com/pkg/errors”` 并调用快速修复程序以下载依赖项。