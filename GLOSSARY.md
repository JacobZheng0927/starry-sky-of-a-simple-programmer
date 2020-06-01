## 原语

是执行过程中不可被打断的基本操作，你可以理解为一段代码，这段代码在执行过程中不能被打断(在多道程序设计里进程间相互切换，还可能有中断发生，经常会被打断)。像原子一样具有不可分割的特性, 所以叫原语, 像原子一样的语句

## znode
Zookeeper的一个命名空间，也是zk的一个节点。由数据寄存器组成

## CSI

Container Storage Interface（容器存储接口）

## CRI

Container Runtime Interface（容器运行时接口）。定义了**容器**和**镜像**的服务的接口，因为容器运行时与镜像的生命周期是彼此隔离的，因此需要定义两个服务。

![CRI架构](https://markdown-image-upload.oss-cn-beijing.aliyuncs.com/img/CRI%E6%9E%B6%E6%9E%84.png)

## CNI

Container Network Interface（容器网络接口）由一组用于配置Linux容器的网络接口的规范和库组成，同时还包含了一些插件。CNI仅关心容器创建时的网络分配，和当容器被删除时释放网络资源。

## CI/CD

Continuous Integration & Continuous Deployment 持续集成&持续部署

## FaaS

Functions as a Service

## 容器运行时

能够基于在线获取的镜像来创建和运行容器的程序。例如：Docker、CRI-O、containerd

## 栈帧

方法运行时的基础数据结构