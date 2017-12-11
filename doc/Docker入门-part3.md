# Docker入门，第三部分：服务

[TOC]

## 准备

- [安装Docker 1.13以上版本][install-docker-113]
- 获取[Docker Compose][docker-compose]。在 [Docker for Mac][docker-for-mac] 和 [Docker for Windows][docker-for-windows]上已经预先安装了，因此你可以忽略。在Linux系统中，你需要 [直接安装][compose-install-directly]，在没有Hyper-V的Windows 10以前的系统中，请使用 [Docker Toolbox][docker-toolbox]
- 阅读[第一部分][tutorial-1]
- 学习[第二部分][tutorial-2]如何创建容器
- 确保前面创建的`friendlyhello`镜像已经发布到Registry，我们将会使用这个共享的镜像
- 确保你的镜像能够部署为容器运行。运行`docker run -p 80:80 username/repo:tag`，替换其中的`username`，`repo`，`tag`为你自己的值，然后访问`http://localhost`

## 简介

在第三部分中，我们将会启用负载均衡并且扩展我们的应用。为了实现这个目标，我们需要进入到一个分布式应用层级的上一级：服务。

- Stack
- **Services**
- Container （[第二部分][tutorial-2]）

## 关于服务

在分布式应用中，我们将应用的不同组成部分叫做“服务”。例如，一个视频分享网站，可能包含一个用于在数据库中存储应用数据的服务，一个在用户上传视频后用于在后台做视频转码的服务，一个用于前端展示的服务等。

服务其实就是“生产环境中的容器”。一个服务只运行一个镜像，它与镜像的运行方式一致（使用什么端口，多少容器的副本一起运行以满足服务的容量需求等）。通过改变运行某部分服务的容器实例数目实现对服务的扩展，给该服务更多的计算资源。

幸运的是，使用Docker平台，你可以非常容易的定义、运行和扩展服务 - 只需要写一个`docker-compose.yml`文件。

## 你的第一个`docker-compose.yml`文件

`docker-compose.yml`文件是一个YAML格式的文件，它定义了Docker容器在生产环境的行为。

### docker-compose.yml

在你希望的地方创建文件`docker-compose.yml`，首先要确保在[第二部分][tutorial-2]闯进的镜像已经推送到了Registry。然后更新这个`.yml`文件，替换其中的`username/repo:tag`部分。

    version: "3"
    services:
      web:
        # 替换 username/repo:tag 
        image: username/repo:tag
        deploy:
          replicas: 5
          resources:
            limits:
              cpus: "0.1"
              memory: 50M
          restart_policy:
            condition: on-failure
        ports:
          - "80:80"
        networks:
          - webnet
    networks:
      webnet:

这个`docker-compose.yml`文件告诉Docker去做下面几件事

- 从Regsitry拉取我们在[第二部分][tutorial-2]上传的镜像
- 创建5个实例运行镜像，名称为`web`，限制每个实例最多占用10%的CPU和50M内存
- 如果其中某个容器宕掉了则自动重启
- 映射宿主机的80端口到`web`的80端口
- 让`web`中的容器通过名为`webnet`的负载均衡网络共享其80端口
- 使用默认配置定义`webnet`网络

## 运行负载均衡后的应用

在我们使用`docker stack deploy`命令之前，首先运行

    docker swarm init

> 注意： 我们将会在[第四部分][tutorial-4]介绍这个命令的含义。如果你没有运行`docker swarm init`命令，你将得到一个“this node is not a swarm manager.”的错误。

现在让我们运行它。你必须给你的应用一个名字，这里我们设置为`getstartedlab`：

    docker stack deploy -c docker-compose.yml getstartedlab
    
我们的这个服务栈中现在运行了5个我们不熟的镜像的容器实例。

获取我们应用的服务的ID：

    docker service ls

你将会看到`web`服务的输出，前面是应用的名称。如果你给它的名字与本示例中是一样的，那么这个名字将会是`getstartedlab_web`。服务的ID也被列了出来，后面跟着副本数，镜像名称和暴露的端口。

在服务中运行的单个容器被称为**任务(task)**。任务都有一个唯一自增的数值ID，最大为你在`docker-compose.yml`中指定的`replicas`值。列出服务中的任务：

    docker service ps getstartedlab_web

如果你只是列出了系统中所有的容器，那么任务也会被列出，但是不会按照服务进行过滤。

    docker container ls -q

现在你可以运行`curl -4 http://localhost`几次，或者在浏览器中刷新几次看看结果

![](https://oayrssjpa.qnssl.com/15130067465181.png)

无论使用哪种方式，你都会看到容器的ID是在变化的，这就证明了负载均衡已经生效，对于每个请求，有以轮询的方式从5个任务中选择出一个来响应请求。容器的ID与之前执行`docker container ls -q`命令的输出相匹配。


> 运行Windows 10？
> 
> 在Windows 10的PowerShell中已经内置了`curl`命令，如果没有的话你可以使用[Git Bash][git-bash]或者是下载[wget for Windows][wget-for-windows]，使用上都是相似的。


> 响应时间很慢？
> 
> 根据环境的网络配置不同，可能会耗费30秒的时间从容器中响应HTTP请求。这并不是Docker或者swarm的性能问题，而是因为Redis依赖没有安装，连接超时所致，我们将会在后面的教程中介绍。现在，访问统计也是因为同样的原因无法正常执行；我们还没有为服务添加持久化数据。

## 扩展应用

可以通过修改`docker-compose.yml`中的`replicas`值来扩展应用，保存变更，然后重新执行`docker stack deploy`命令：

    docker stack deploy -c docker-compose.yml getstartedlab
    
Docker将会执行实时更新，不需要关掉这个栈或者是kill掉任何容器。

现在，重新运行`docker container ls -q`命令查看从新配置的容器实例。如果你增加了副本数，则更多的任务（容器）会被开启。

### 关闭应用和Swarm

- 使用`docker stack rm`命令关闭应用

        docker stack rm getstartedlab

- 关闭swarm

        docker swarm leave --force
    
这与在Docker中开始和扩展应用一样简单。你节省了很多去学习如何在生产环境中运行容器的时间。在下一节中，你将会学到如何在Docker集群中运行这个而应用作为一个真正的swarm。

### 总结和备忘单[可选]

下面是一个[控制台视频][terminal-recording]，记录了本教程的内容

[![asciicast](https://asciinema.org/a/113831.png)](https://asciinema.org/a/113831)

下面是本文中用到的一些基本的Docker命令

    # 列出应用的stacks
    docker stack ls
    # 运行指定的compose文件
    docker stack deploy -c <composefile> <appname>
    # 列出与当前应用关联的运行中的服务
    docker service ls
    # 列出与当前应用相关的任务
    docker service ps <service>
    # 审查任务或者容器
    docker inspect <task or container> 
    # 列出容器ID
    docker container ls -q 
    # 关闭应用
    docker stack rm <appname> 
    # 关闭单个swarm节点
    docker swarm leave --force
    
[install-docker-113]:https://docs.docker.com/engine/installation/
[tutorial-1]:https://docs.docker.com/get-started/
[tutorial-2]:https://docs.docker.com/get-started/part2/
[tutorial-4]:https://docs.docker.com/get-started/part4/
[docker-compose]:https://docs.docker.com/compose/overview/
[docker-for-mac]:https://docs.docker.com/docker-for-mac/
[docker-for-windows]:https://docs.docker.com/docker-for-windows/
[compose-install-directly]:https://github.com/docker/compose/releases
[docker-toolbox]:https://docs.docker.com/toolbox/overview/
[git-bash]:https://git-for-windows.github.io/
[wget-for-windows]:http://gnuwin32.sourceforge.net/packages/wget.htm
[terminal-recording]:https://asciinema.org/a/b5gai4rnflh7r0kie01fx6lip

原文：[Get Started, Part 3: Services](https://docs.docker.com/get-started/part3/#recap-and-cheat-sheet-optional)

