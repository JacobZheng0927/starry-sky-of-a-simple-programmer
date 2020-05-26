# K8S 从入门到放弃

[toc]

## 1.简介

Kubernetes是一个开源的，用于管理云平台中多个主机上的容器化的应用，Kubernetes的目标是让部署容器化的应用简单并且高效（powerful）,Kubernetes提供了应用部署，规划，更新，维护的一种机制。

## 2.架构

![architecture](https://markdown-image-upload.oss-cn-beijing.aliyuncs.com/img/architecture.png)

- **Kubernetes 主控组件（Master）** 

  包含三个进程，都运行在集群中的某个节上，主控组件通常这个节点被称为 master 节点。这些进程包括：[kube-apiserver](https://kubernetes.io/docs/admin/kube-apiserver/)、[kube-controller-manager](https://kubernetes.io/docs/admin/kube-controller-manager/) 和 [kube-scheduler](https://kubernetes.io/docs/admin/kube-scheduler/)。

  Kubernetes master 节点负责维护集群的目标状态。当你要与 Kubernetes 通信时，使用如 `kubectl` 的命令行工具，就可以直接与 Kubernetes master 节点进行通信。

  > “master” 是指管理集群状态的一组进程的集合。通常这些进程都跑在集群中一个单独的节点上，并且这个节点被称为 master 节点。master 节点也可以扩展副本数，来获取更好的可用性及冗余。

- Kubernetes Node 节点

  集群中的 node 节点（虚拟机、物理机等等）都是用来运行你的应用和云工作流的机器。Kubernetes master 节点控制所有 node 节点；你很少需要和 node 节点进行直接通信。每个node都运行两个进程：

  - **[kubelet](https://kubernetes.io/docs/admin/kubelet/)**，和 master 节点进行通信。

  - **[kube-proxy](https://kubernetes.io/docs/admin/kube-proxy/)**，一种网络代理，将 Kubernetes 的网络服务代理到每个节点上。

## 3.Kubernetes 对象

基本的 Kubernetes 对象包括：

- Pod

  > `Pod` 是最小可部署的 Kubernetes 对象模型。

  Pod 是 Kubernetes 应用程序的基本执行单元，即它是 Kubernetes 对象模型中创建或部署的最小和最简单的单元。Pod 表示在 集群 上运行的进程。

  Pod 封装了应用程序容器（或者在某些情况下封装多个容器）、存储资源、唯一网络 IP 以及控制容器应该如何运行的选项。 Pod 表示部署单元：Kubernetes 中应用程序的单个实例，它可能由单个 容器 或少量紧密耦合并共享资源的容器组成。

  Docker 是 Kubernetes Pod 中最常用的容器运行时，但 Pod 也能支持其他的容器运行时。

  Kubernetes 集群中的 Pod 可被用于以下两个主要用途：

  - **运行单个容器的 Pod**。“每个 Pod 一个容器"模型是最常见的 Kubernetes 用例；在这种情况下，可以将 Pod 看作单个容器的包装器，并且 Kubernetes 直接管理 Pod，而不是容器。
  - **运行多个协同工作的容器的 Pod**。 Pod 可能封装由多个紧密耦合且需要共享资源的共处容器组成的应用程序。 这些位于同一位置的容器可能形成单个内聚的服务单元 —— 一个容器将文件从共享卷提供给公众，而另一个单独的“挂斗”（sidecar）容器则刷新或更新这些文件。 Pod 将这些容器和存储资源打包为一个可管理的实体。

  Pod 为其组成容器提供了两种共享资源：*网络* 和 *存储*。

  - 网络

  每个 Pod 分配一个唯一的 IP 地址。 Pod 中的每个容器共享网络命名空间，包括 IP 地址和网络端口。 *Pod 内的容器* 可以使用 `localhost` 互相通信。 当 Pod 中的容器与 *Pod 之外* 的实体通信时，它们必须协调如何使用共享的网络资源（例如端口）。

  - 存储

  一个 Pod 可以指定一组共享存储卷。 Pod 中的所有容器都可以访问共享卷，允许这些容器共享数据。 卷还允许 Pod 中的持久数据保留下来，以防其中的容器需要重新启动。

- Service

  Kubernetes `Service` 定义了这样一种抽象：逻辑上的一组 `Pod`，一种可以访问它们的策略 —— 通常称为微服务。 这一组 `Pod` 能够被 `Service` 访问到，通常是通过`selector`实现的。

  举个例子，考虑一个图片处理 backend，它运行了3个副本。这些副本是可互换的 —— frontend 不需要关心它们调用了哪个 backend 副本。 然而组成这一组 backend 程序的 `Pod` 实际上可能会发生变化，frontend 客户端不应该也没必要知道，而且也不需要跟踪这一组 backend 的状态。 `Service` 定义的抽象能够解耦这种关联。

- Volume

  Kubernetes 卷具有明确的生命周期——与包裹它的 Pod 相同。 因此，卷比 Pod 中运行的任何容器的存活期都长，在容器重新启动时数据也会得到保留。 当然，当一个 Pod 不再存在时，卷也将不再存在。也许更重要的是，Kubernetes 可以支持许多类型的卷，Pod 也能同时使用任意数量的卷。

  卷的核心是包含一些数据的目录，Pod 中的容器可以访问该目录。 特定的卷类型可以决定这个目录如何形成的，并能决定它支持何种介质，以及目录中存放什么内容。

- Namespace

  命名空间适用于存在很多跨多个团队或项目的用户的场景。对于只有几到几十个用户的集群，根本不需要创建或考虑命名空间。当需要名称空间提供的功能时，请开始使用它们。

  命名空间为名称提供了一个范围。资源的名称需要在命名空间内是唯一的，但不能跨命名空间。命名空间不能相互嵌套，每个 Kubernetes 资源只能在一个命名空间中。

Kubernetes 也包含大量的被称作 Controller 的高级抽象。控制器基于基本对象构建并提供额外的功能和方便使用的特性。具体包括：

- Deployment
- DaemonSet
- StatefulSet
- ReplicaSet
- Job

## 4.单机k8s安装

1.

## 9.常用命令



## 10.名词解释

- 容器运行时

  Kubernetes 使用容器运行时来实现在 pod 中运行容器。

  例如：Docker、CRI-O、containerd

- 