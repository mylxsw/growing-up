# 【翻译】Docker入门，第一部分：定位和安装

[TOC]

欢迎！非常高兴你想学习如何使用Docker。

在这个六部分的教程中，你将会学到：

1. 在本文中学习到Docker的定位和安装方法
2. 创建和运行你的第一个app
3. 将你的app转变成一个可扩展的服务
4. 跨多台机器部署你的服务
5. 添加一个持久化数据的访问者计数器
6. 在生产环境中部署你的swarm

这个应用本身是非常简单的，因此你不需要在这些代码是做什么上面分散太多精力。毕竟Docker的价值是如何构建，部署和运行应用；它并不知道你的应用是做什么的。

## 准备

我们将会定义一些概念，在开始之前，最好你已经理解了[Docker是什么][what-docker-is]和[为什么要使用Docker][why-you-would-use-docker]。

我们假设你已经熟悉下面这些概念：

- IP地址和端口
- 虚拟机
- 编辑配置文件
- 基本熟悉代码构建和依赖的原理
- 机器资源使用状态术语，比如CPU百分比，内存使用等

## 容器的简单解释

**镜像**（image） 是一个包含你的软件运行所需要的所有组件的轻量级的，独立的、可执行的包，它包含代码，运行时，类库和环境变量、配置文件。

**容器**（container）是镜像的运行实例（当实际执行的时候，镜像在内存中的样子）。默认情况下，它的运行时与宿主环境完全隔离的，只可以访问配置的宿主文件和端口。

容器中的应用是在宿主机的内核上运行的，它有着比只能通过hypervisor访问宿主机资源的虚拟机有着更好的性能。容器可以通过本地访问，每一个容器都运行在独立的进程中，相比其它可执行文件占用的内存并不会很多。

## 容器vs虚拟机

下面的图中比较了容器和虚拟机

### 虚拟机结构图

![](https://oayrssjpa.qnssl.com/15128858464202.png)

虚拟机uyunxing在宿主操作系统上，注意的是每个盒子中包含一个OS层。这是资源密集的，导致了磁盘镜像和应用的状态与操作系统的配置，安装的依赖，系统安全补丁等纠缠在一起，非常容易丢失，很难去复制。

### 容器结构图

![](https://oayrssjpa.qnssl.com/15128861348558.png)

容器可以互相共享单个内核，在容器镜像中唯一需要的信息是你的可执行文件和它的依赖，这些东西永远都不需要安装在宿主系统中。这些进程就像本地进程一样运行，就像在Linux中使用`ps`命令查看活动的进程那样，你可以使用命令`docker ps`等命令独立的管理它们。最后，因为他们包含了它们所有的依赖，因此它不会与宿主机的配置纠缠在一起；一个容器话的应用可以“在任何地方运行”。

## 安装

在开始之前，确保你的系统中已经安装了最新版的Docker。

[安装Docker][install-docker]

> 注意：需要1.13更高版本

你应该可以使用`docker run hello-world`命令查看它的响应：

> 注意：如果你不希望使用sudo来执行这个命令，你可能需要添加你的当前用户到`docker`用户组中。[了解更多](docker-run-read-more)
> 如果在安装的时候遇到网络问题，`docker run hello-world`命令可能会执行失败。当你处于代理服务器后面，你推测到是因为它堵塞了连接，请参考本教程的[下一部分][tutorial-2]。

    $ docker run hello-world
    
    Hello from Docker!
    This message shows that your installation appears to be working correctly.
    
    To generate this message, Docker took the following steps:
    ...(snipped)...

现在需要确定一下你使用的是否是1.13以上版本，运行`docker --version`命令

    $ docker --version
    Docker version 17.05.0-ce-rc1, build 2878a85

如果你看到了类似上面的信息，说明你已经准备好开始我们的旅程了。

## 总结

扩展的单位成为了独立的个体，对于可移植的应用来说有非常重要的意义。它意味着CI/CD可以为为分布式应用的任意部分推送更新，系统的依赖不在是问题，资源的密度也增加了。扩展行为编排的关键是创建新的可执行文件，而不是新的虚拟机。

我们后面将会学习这些，但是首先让我们先学会走路吧。

[what-docker-is]:https://www.docker.com/what-docker
[why-you-would-use-docker]:https://www.docker.com/use-cases
[install-docker]:https://docs.docker.com/engine/installation/
[docker-run-read-more]:https://docs.docker.com/engine/installation/linux/linux-postinstall/
[tutorial-2]:https://docs.docker.com/get-started/part2/

原文：[Get Started, Part 1: Orientation and setup](https://docs.docker.com/get-started/)
