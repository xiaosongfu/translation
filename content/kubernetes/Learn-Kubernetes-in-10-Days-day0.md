
> 原文链接：[Day-0: What is Kubernetes? What are its components](https://github.com/ajeetraina/kubernetes101/blob/master/architecture/README.md)

# 第0天：什么是 Kubernetes？它都由哪些组件组成

## 什么是Kubernetes？

Kubernetes（通常称为K8s）是容器技术的编排引擎，例如Docker和rkt，它们在过去几年中接管了DevOps场景。它已作为托管服务在Azure和Google Cloud上提供。

Kubernetes可以通过简单，自动化的部署，更新（滚动更新）以及几乎零停机时间管理我们的应用程序和服务来加速开发过程。它还提供自我修复。当进程在容器内崩溃时，Kubernetes可以检测并重新启动服务。Kubernetes最初由谷歌开发，自推出以来一直是开源的，并由大量贡献者组成的社区管理。

任何开发人员都可以使用基本的Docker知识打包应用程序并将其部署在Kubernetes上。

## K8s是由什么组成的？

#### Kubectl

* Kubernetes的CLI工具

![]()

#### Master Node

* 控制 nodes 的主机
* 所有管理任务的主要入口点
* 它处理工作 nodes 的编排

![]()

#### Worker Node

* 它是Kubernetes的一个工作 node（曾经被称为 minion）
* 本机执行请求的任务。每个 node 由 master node 控制
* 在 pod 内运行容器
* 这是Docker引擎运行的地方，负责下载 image 和启动 containers

![]()

#### Kubelet

* 主要的节点代理
* 确保容器的正常运行和健康

#### Kubernetes Pod：

* Pod 可以托管多个容器和存储卷
* Pod 是 Deployments 的实例
* 一个 Deployments 可以有多个 pod
* 使用 Horizo​​ntal Pod Autoscaling，可以根据CPU使用情况自动启动和暂停部署的Pod
* 同一 Pod 中的容器可以访问共享存储卷
* 每个 Pod 在群集中都有其唯一的 IP 地址
* Pod 将启动并运行，直到有人（或控制器）摧毁它们
* 保存在 Pod 中的任何数据在没有持久存储的情况下都将会消失



---

参考：

* https://github.com/ajeetraina/kubernetes101/

