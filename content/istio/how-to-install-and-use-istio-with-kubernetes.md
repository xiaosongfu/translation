https://www.digitalocean.com/community/tutorials/how-to-install-and-use-istio-with-kubernetes

---

## 介绍

服务网格是一个基础结构层，允许您管理应用程序的微服务之间的通信。随着越来越多的开发人员使用微服务，服务网格已经发展到通过在分布式设置中整合通用管理和管理任务，使工作更轻松，更有效。

使用像Istio这样的服务网络可以简化服务发现，路由和流量配置，加密和身份验证/授权以及监控和遥测等任务。特别是，Istio旨在在不对现有服务代码进行重大更改的情况下工作。例如，在使用Kubernetes时，可以通过构建与现有应用程序资源一起使用的特定于Istio的对象，为服务器中运行的应用程序添加服务网格功能。

在本教程中，您将使用Kubernetes的Helm包管理器安装Istio。然后，您将使用Istio 通过创建网关和虚拟服务资源将演示Node.js应用程序公开给外部流量。最后，您将访问Grafana遥测插件以可视化您的应用程序流量数据。

