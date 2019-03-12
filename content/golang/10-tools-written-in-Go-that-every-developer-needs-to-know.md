---
title: "每个开发人员都需要知道的用 Go 编写的10个工具"
date: 2019-01-30T14:06:11+08:00
draft: false
tags: ["Golang"]
categories: ["Golang"]
---

> 原文链接：[10 tools written in Go that every developer needs to know](https://gustavohenrique.net/en/2019/01/10-tools-written-in-go-that-every-dev-needs-to-know/)

我将向您展示10种非常棒的工具，这些工具可以帮助我完成我的工作。因为它们是用Go编写的，所以不需要进行复杂的设置，您只需要根据您的操作系统（Linux，MacOS或Windows）运行二进制文件版本。

## 1. Caddy

Caddy 是一个轻量级的 HTTP/2 Web 服务器，具有自动 HTTPS（使用 Let's Encrypt），易于配置和高性能。它还可以在桌面计算机中用于提供当前文件夹中的静态文件。几年后我用 Caddy 取代了 Nginx，我对这个选择非常满意。我喜欢将他用作 Web 应用程序和 SPA 容器（VueJS和ReactJS）的代理。

```
cd $HOME/Downloads
caddy browse
```

## 2. Ngrok

Ngrok 允许您出于多种目的发布公共 URL。常见的用法是暴露用于演示的本地 Web 服务器，但开发人员还用于构建 webhook 集成，测试聊天机器人以及在没有公共 IP 地址的计算机上访问 SSH 服务器。Ngrok 还有一个 Web 界面，显示有关收到的请求的数据。它易于使用，只需运行它即可将协议和端口作为参数公开：

```
ngrok http 3000
```

## 3. Ctop

Ctop是容器指标的类似Top的界面，提供了对多个容器的实时指标的简明扼要概述。

```
ctop

# running in container
docker run --rm -ti -v /var/run/docker.sock:/var/run/docker.sock quay.io/vektorlab/ctop:latest
```

## 4. GoTTY

GoTTY 是一个简单的命令行工具，可将您的 CLI 工具转换为 Web 应用程序。它非常适合Linux 或 MacOS 机器没有安装 SSH 服务器并且普通用户希望提供远程访问的情况。

```
gotty -w bash  # the default port is 8080
```

## 5. Mattermost Server

Mattermost是一个安全的消息传递工作场所，可部署到IT控制下的云或本地基础架构。开源服务器提供Web客户端，但您可以在产品网站上下载桌面和移动客户端。当你的公司或朋友开始谈论Slack vs Discord vs Rocket Chat时，你可以谈论Mattermost延长（或不延长）讨论时间。

```
docker run -d -p 8065:8065 mattermost/mattermost-preview
```

## 6. Minio

Minio 是一款高性能分布式对象存储服务器，专为大规模私有云基础架构而设计。它类似于Amazon S3 和 Google Cloud Storage。
我从来没有使用它，但我认为这是一个有趣的项目。

```
export MINIO_ACCESS_KEY="AKIAIOSFODNN7EXAMPLE"
export MINIO_SECRET_KEY="wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
./minio server $HOME

# running in container
docker run -p 9000:9000 -v $HOME:/data -e "MINIO_ACCESS_KEY=AKIAIOSFODNN7EXAMPLE" -e "MINIO_SECRET_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY" minio/minio server /data
```

## 7. Piknik

Piknik无缝且安全地在任意主机之间传输URL，代码片段，文档，几乎任何东西。不需要SSH，主机可以在不同网络的NAT网关后面。它有一个VS Code编辑器插件，可以在短时间内共享代码片段。我一直在使用很多东西在同一个无线网络中的Mac和我的Linux工作站之间共享内容。

用法是：

```
# To fill the "clipboard"
piknik -copy < /home/gustavo/Downloads/picture.jpg

# Magically retrieve that content from any other host
piknik -paste > /Users/gustavo/Downloads/picture.jpg
```
