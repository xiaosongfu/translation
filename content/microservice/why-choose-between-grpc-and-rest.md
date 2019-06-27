---
title: "为什么选择gRPC和REST？允许您的服务在单个端口上公开 REST 和 gRPC API 的选项"
date: 2019-04-19T11:14:48+08:00
draft: false
tags: ["microservice"]
categories: ["Microservice"]
---

> 原文链接：[Why choose between gRPC and REST?](https://medium.com/@thatcher/why-choose-between-grpc-and-rest-bc0d351f2f84)

---

## 1. 摘要

在本文中，我将探讨使用 Go 编写服务和在单个端口上暴露 gRPC 和 REST API。我解释了如何使用流行的 [grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway) 从 gRPC 定义自动生成 REST-full JSON API，然后展示如何使用 HTTP 多路复用器使两个API使用相同的端口。然后我得出结论，这种方法存在严重的局限性，并使用 CMUX（TCP多路复用器）显示替代方案。这样可以实现更清洁，更好的架构。

## 介绍

我发现自己正在开发一个名为 Moulin 的开源项目，这是一个用 Go 编写并由 Redis 支持的批处理的可靠队列。由于存在不同的用例，我希望为用户提供快速 gRPC 选项和更传统的 REST API 之间的选择。我还希望以后能够添加 Web 界面。

从 gRPC docs 页面我读到有一个名为 GRPC Gateway 的“热门项目” 。它允许您注释您的 Protobuf 定义，然后它将自动生成 HTML 处理程序的代码，负责从 JSON 到 Protobuf 的转换并返回，并将其转发到上游 gRPC 服务器。

我决定如果两个 API 可以共享相同的端口（即80或443），那将会很棒。在部署中，用户可以映射单个主机名，然后就“可以工作”。原因是 gRPC 使用 http2，这使得它全部都是 HTTP。

来自 CoreOS 的 Brendon Philips 发表了一篇题为“ 使用HTTP/2，Protobufs 和 Swagger 开发 REST ”的文章，他解释了这种方法。我最初决定写一篇关于同一主题的文章，因为我觉得用这种方法创建和共享TLS证书的主题存在问题并且没有很好的文档记录。后来我发现了一个更好的解决方案（根据我的需要）。

## 这个怎么运作：

![]()

gRPC和HTTP客户端都连接到应用程序上的同一端口，由go的http请求多路复用器提供服务。

多路复用器检查http版本是否为http2，以及内容类型是否为“application / grpc”。

如果是，我们只是将请求传递给gRPC处理程序。如果不是，我们将HTTP消息传递给gRPC gateway，然后 gRPC gateway 将其变为 gRPC消息，创建新连接，然后将其转发到gRPC服务器。当然，有一点开销，但gRPC很快，这节省了很多代码（...）。

在这里您可以找到完整的源代码：[](https://github.com/dhrp/grpc-rest-go-example)

以下是实际歧视的功能：

```
// grpcHandlerFunc returns an http.Handler that delegates to grpcServer on incoming gRPC
// connections or otherHandler otherwise. Copied from cockroachdb.
func grpcHandlerFunc(grpcServer *grpc.Server, otherHandler http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		if r.ProtoMajor == 2 && strings.Contains(r.Header.Get("Content-Type"), "application/grpc") {
			grpcServer.ServeHTTP(w, r)
		} else {
			otherHandler.ServeHTTP(w, r)
		}
	})
}
```

设置并不容易，但它确实有效。

## 注意事项

问题是这个解决方案带有几个警告......

起初我以为我可以为gRPC服务器禁用TLS，因为虽然gRPC鼓励TLS，但它确实有一个`WithInsecure()`函数。问题的核心是我们使用的是http多路复用器，所以必须使用`grpc.ServeHTTP()`而不是`grpc.Serve()`。这使用了go的http2库，它们确实 需要TLS。所以我们必须在内部处理TLS。这令人沮丧。在上图中，它是grpc-gateway和grpc处理程序之间的连接。您可以在此处阅读有关此问题的更多信息：https：//github.com/grpc/grpc-go/issues/555

好的，那么.. REST代理和gRPC服务器之间的（内部）连接必须通过TLS，但是gRPC + REST端口也需要可以远程访问。真正的痛苦在于，证书需要不是针对localhost发布，而是针对一些实际的主机名或IP发布。无论何时部署，都需要重新创建证书，并且需要在内部配置代理以使用该新地址或主机名，否则将拒绝连接。

这很麻烦，根据您的网络设置，也可能导致网络流量回路通过负载均衡器。如果您正在开发内部API或托管（专有）解决方案，并期望获得很多不必重新创建REST端点的好处，那么它可能仍然值得。但请注意，共享开发工作可能会更加困难。然后，实际上可以更好地保持经典设置从不同主机提供grpc网关（代理）。

在尝试使用 gRPC gateway 时，我还发现了其他限制，主要是由于 REST 没有干净地映射到 gRPC，结果是 REST API 有点难看。例如，您不能使用查询参数或表单上传文件。

## 备择方案

因此，虽然我们已经展示（并给出了代码）以使其工作，但我们也认识到它并不漂亮。那么替代方案是什么？

仅存在内部TLS连接的问题，因为我们在收到HTTP请求后正在设置新的gRPC连接。

如果我们不使用gRPC网关，我们仍然可以使用HTTP多路复用器将流量发送到gRPC处理程序或常规HTTP处理程序。

## 使用（仅）HTTP多路复用器

![]()

如图所示，我们现在仍然有一个HTTP请求多路复用器侦听的开放端口，但是有两组处理程序。

HTTP多路复用器检查它的请求类型，并将其转发给适当的处理程序。

然后我们可以编写简单的HTTP处理函数，它们只需调用与gRPC调用相同的业务逻辑。

可以在此处找到有关如何在代码中执行此操作的一个小示例。

由于使用了http2，外部http多路复用器端口仍然需要TLS，但这不是一个真正的问题。但如果你愿意，你也可以解决这个问题。

## 使用TCP多路复用器（cmux）

也可以在TCP级复用连接。有一个名为cmux的小型go库，它允许在单个端口上混合和过滤不同的协议，而不仅仅是在不同的路由或HTTP版本上进行过滤。

它的工作原理是从传入请求中读取前几个字节，以确定它是哪个协议。如果是HTTP，则将其转发给HTTP处理程序，如果是gRPC，则将其转发给gRPC处理程序。但它不仅限于HTTP和gRPC。它也可以做SSH，或FTP，或任何你的心愿。例如，同一端口上的HTTP和HTTPS。这使它非常灵活。

在另一个分支中，我已经移植了我首先使用这种方法展示的示例代码，它就像一个魅力。

[](https://github.com/dhrp/grpc-rest-go-example/tree/nogateway)

## 结论

在花时间弄清楚这些不同选项的细节后，我相信 GRPC-Gateway 不是我需求的绝佳选择。虽然这是一个很酷的项目和想法，但内部TLS的设置是必需的并且令人讨厌，并且在这种配置中它可能导致意外的网络问题和开销。grpc-gateway还有一些其他限制因素会影响REST API的设计。

没有它，我们的应用程序将需要两次实现事件处理程序。一次用于gRPC，一次用于HTTP调用。根据您的应用程序结构和您拥有的端点数量，开销受到限制。在我的情况下，节省的费用不值得。无论您选择HTTP还是TCP多路复用器都不太重要，但TCP多路复用选项为您提供了最大的灵活性，是我们现在要做的。

谢谢阅读！

## 致谢/链接
* GRPC Gateway: . This is really what inspired me to explore this option.
* A tale of two ports; CockroachDB was (among the) first to push for the use-case of mixing gRPC and REST on one port in Golang.
* Take a REST with HTTP/2, Protobufs, and Swagger; Who showed how to make it work on one port by Brandon Philips (CoreOS)
* Golang: Run multiple services on one port; Article by Ashwin Ramesh of Dgraph showing how to use cmux.
* CMUX, TCP multiplexer for Golang.
* AlesSI for pointing to Cmux on the go-grpc issue.
* And thanks to Mark Gituma for giving me feedback on the article!
