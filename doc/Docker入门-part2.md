# 【翻译】Docker入门，第二部分：容器

[TOC]

## 准备

- [安装Docker 1.13以上版本][install-docker-113]
- 阅读[第一部分][tutorial-1]
- 快速测试一下你的环境是否可OK

        docker run hello-world

> 提示：为了确保本教程的顺利进行，请使用最新版的Docker服务和客户端。例如，遗留的应用比如Docker Toolbox可能会给你不符合预期的本地IP地址，老版本的Docker可能不支持示例中的所有特性。

## 简介

是时候使用Docker的方式来构建应用了。我们将会从下往上的介绍，这里我们从最底部开始，也就是容器，在这之上是服务，定义了容器在生产环境中的行为，我们将会在[第三部分][tutorial-3]介绍。最后，在最顶层定义了所有服务之间的交互，将在[第五部分][tutorial-5]介绍。

- Stack
- Services
- **Container**

## 新建开发环境

在过去，如果你要开始写一个Python应用的话，你的第一步是在你的电脑上安装Python运行时。但是，为了让你的应用能够运行，导致了你的电脑环境必须做一些调整。

当使用Docker时，你可以使用Python运行时镜像，不需要安装。然后，在这个Python镜像之上构建的的代码，确保你的应用，依赖和运行时都放在一起。

这个可移植的镜像叫做`Dockerfile`。

## 使用`Dockerfile`文件定义一个容器

`Dockerfile`定义了你的容器中的环境是怎样的。容器中对资源的访问，网络接口和磁盘驱动等都就进行了虚拟化，它们与你的系统是隔离的，因此你必须映射到外部世界的端口，指定从外部世界复制哪些文件到这个环境中。在做了这些之后，无论你的应用在哪里运行，它都会按照Dockerfile中的定义，表现出完全一致的行为。

### Dockerfile

创建一个空目录，使用`cd`命令进入到新创建的目录中，创建一个名为`Dockerfile`的文件，复制，然后粘贴下面内容到这个文件中保存。其中的注释说明了每条语句的作用。

    # 使用官方的Python运行时作为父镜像
    FROM python:2.7-slim
    
    # 设置工作目录为/app
    WORKDIR /app
    
    # 复制当前目录中的内容到容器的/app目录中
    ADD . /app
    
    # 安装在requirements.txt指定的包
    RUN pip install --trusted-host pypi.python.org -r requirements.txt
    
    # 确保端口80对容器外部的世界可用
    EXPOSE 80
    
    # 定义环境变量
    ENV NAME World
    
    # 当容器启动的时候，运行app.py
    CMD ["python", "app.py"]

> 你在一个代理服务器之后吗？
> 
> 代理服务器可能会堵塞你的web应用的连接。如果你出于一个代理服务器之后，在你的Dockerfile中添加下面几行，使用`ENV`命令指定代理服务器的地址和端口
> 
>     # 设置代理服务器，替换host:port为你的代理服务器配置
>     ENV http_proxy host:port
>     ENV https_proxy host:port

这个Dockerfile引用了一些我们还没有创建的文件`app.py`和`requirements.txt`，接下来我们来创建它们。

## 创建应用

创建`requirements.txt`和`app.py`两个文件，把它们放到和`Dockerfile`同一个目录。这样我们的应用就完成了，你可以看到，这是非常简单的，当上面的`Dockerfile`构建为一个镜像的时候，因为`Dockerfile`中使用了`ADD`命令，因此`app.py`和`requirements.txt`也将会在其中。同时，由于使用了`EXPOSE`命令，我们也可以通过HTTP访问`app.py程序的输出。

**requirements.txt**

    Flask
    Redis

**app.py**

    from flask import Flask
    from redis import Redis, RedisError
    import os
    import socket
    
    # Connect to Redis
    redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)
    
    app = Flask(__name__)
    
    @app.route("/")
    def hello():
        try:
            visits = redis.incr("counter")
        except RedisError:
            visits = "<i>cannot connect to Redis, counter disabled</i>"
    
        html = "<h3>Hello {name}!</h3>" \
               "<b>Hostname:</b> {hostname}<br/>" \
               "<b>Visits:</b> {visits}"
        return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=visits)
    
    if __name__ == "__main__":
        app.run(host='0.0.0.0', port=80)
        
现在，我们看到`pip install -r requirements.txt`命令会为Python安装Flask和Redis库，然后应用会打印环境变量`NAME`和`socket.gethostname()`的输出。最后，因为Redis并没有运行（我们只是安装了Python库，并没有安装Redis），我们预期的输出将会是输出错误信息。

> 注意：在容器中获取host的时候，我们获取到的是容器ID，与运行的可执行文件的进程ID类似。

这就是全部！你不需要再你的系统中安装Python或者是`requirements.txt`中的依赖。看起来你并没有建立Python和Flask的运行环境，但是实际上你已经建立了。

## 构建应用

我们已经准备好构建应用了。确保你依然在这个新的目录中

    $ ls
    Dockerfile    app.py    requirements.txt

现在，运行构建命令。这个命令将会创建一个Docker镜像，我们使用`-t`选项为它起了一个友好的名字。

    docker build -t friendlyhello .

你构建的镜像在哪里呢？它在你的电脑的本地Docker镜像仓库中：

    $ docker images
    
    REPOSITORY            TAG                 IMAGE ID
    friendlyhello         latest              326387cea398

> 提示：你可以使用命令`docker images`或者新的`docker image ls`命令列出镜像。它们会给你相同的输出。

## 运行应用


运行应用，使用`-p`选项映射你本地的4000端口到容器的80端口

    docker run -p 4000:80 friendlyhello

你将会看到Python运行在`http://0.0.0.0:80`的消息。但是这个消息是来自于容器内部的。它并不知道你映射了80端口为宿主机的4000端口，正确的URL应该是`http://localhost:4000`。

在web浏览器中访问这个URL地址，你将会在web页面中看到输出信息，包含"Hello World"，容器的ID和Redis错误消息。

![](media/15128874414723/15128908659941.png)

> 注意：如果你在Windows 7下使用Docker Toolbox，请使用Docker机器的IP地址代替`localhost`。例如，`http://192.168.99.100:4000/`。查看IP地址，使用命令`docker-machine ip`。

你也可以在shell中使用`curl`命令查看同样的输出

    $ curl http://localhost:4000
    
    <h3>Hello World!</h3><b>Hostname:</b> 8fc990912a14<br/><b>Visits:</b> <i>cannot connect to Redis, counter disabled</i>

这里的端口重新映射`4000:80`是为了举例说明与在Dockerfile中使用`EXPOSE`命令的区别，以及使用`docker run -p`的结果。在后面的步骤中，我们将仅仅映射宿主机的80端口到容器的80端口，你可以直接使用`http://localhost`。

使用`CTRL+C`退出控制台。

> 在Windows下停止容器
> 
> 在Windows系统中，使用`CTRL+C`并不会停止容器。因此，首先键入`CTRL+C`，然后键入`docker container ls`列出当前运行的容器，然后使用`docker container stop <Container NAME or ID>`命令停止容器。否则，当你重新运行容器的时候回收到错误的响应。

现在让我们在后台运行我们的应用，使用detached模式

    docker run -d -p 4000:80 friendlyhello
    
你会得到一个你的应用的很长的容器ID，然后回到控制台。你的容器现在正以后台模式运行。可以使用`doceker container ls`命令查看容器ID的缩写。

    $ docker container ls
    CONTAINER ID        IMAGE               COMMAND             CREATED
    1fa4ab2cf395        friendlyhello       "python app.py"     28 seconds ago

你会看到`CONTAINER ID`与访问`http://localhost:4000`获取都的值是一致的。

现在使用`docker container stop`命令终止进程，使用`CONTAINER ID`，比如

    docker container stop 1fa4ab2cf395

## 共享你的镜像

为了举例说明我们刚创建的应用的可移植性，让我们上传我们构建的镜像然后在其它地方运行它。在这之后，你将会学到当你希望在生产环境中部署容器的时候，如何将其推送到Registry。

Registry是一个仓库的集合，仓库是镜像的集合（还有点类似Github的仓库，除了它的代码是已经编译之后的）。在regisgry中的账号可以创建很多歌仓库。默认情况下，`docker` CLI使用了Docker的公共Registry。

> 注意：在这里我们之所以使用Docker的公共registry，是因为它是免费的，而且已经预先配置好的。有很多公开的仓库可以选择，你甚至可以使用[Docker Trusted Registry][docker-trusted-registry]建立自己的私有仓库。

### 使用你的Docker ID登录

如果你没有Docker账户，首先到 [cloud.docker.com][cloud-docker] 注册一个，然后在你的电脑上登录到Docker的公共registry

    $ docker login
    
### 为镜像打标签

本地镜像与registry上帝额仓库的关联名称为`username/repository:tag`。标签是可选的，但是推荐使用，因为他是registry用来维护docker镜像的版本。给你的仓库和标签一个在上下文中有意义的名字，比如`get-started:part2`，这样就将这个镜像放置到`get-started`仓库，并且打了一个`part2`的标签。

现在，让我们为镜像打一个标签。运行`docker tag image`命令，跟着你的用户名，仓库名和标签名，这样你的镜像将会上传到你希望的位置。该命令的语法如下

    docker tag image username/repository:tag

例如

    docker tag friendlyhello john/get-started:part2

运行 [docker images][docker-images] 查看你新打标签的镜像（你也可以使用`docker image ls`命令）。

    $ docker images
    REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
    friendlyhello            latest              d9e555c53008        3 minutes ago       195MB
    john/get-started         part2               d9e555c53008        3 minutes ago       195MB
    python                   2.7-slim            1c7128a655f6        5 days ago          183MB
    ...

### 发布镜像

上传已经打过标签额镜像到仓库中：

    docker push username/repository:tag
    
一旦完成，这个上传的镜像将会对所有人公开。如果你登录到[Docker Hub][docker-hub]，你将会看到你的新镜像以及它的拉取命令。

### 从远程仓库拉去并运行镜像

从现在开始，你可以使用`docker run`命令，在任何机器上运行你的应用

    docker run -p 4000:80 username/repository:tag

如果本地没有该镜像的话，Docker将会先从远程仓库中拉取它。

    $ docker run -p 4000:80 john/get-started:part2
    Unable to find image 'john/get-started:part2' locally
    part2: Pulling from john/get-started
    10a267c67f42: Already exists
    f68a39a6a5e4: Already exists
    9beaffc0cf19: Already exists
    3c1fe835fb6b: Already exists
    4c9f1fa8fcb8: Already exists
    ee7d8f576a14: Already exists
    fbccdcced46e: Already exists
    Digest: sha256:0601c866aab2adcc6498200efd0f754037e909e5fd42069adeff72d1e2439068
    Status: Downloaded newer image for john/get-started:part2
     * Running on http://0.0.0.0:80/ (Press CTRL+C to quit)

> 注意：如果你没有指定`:tag`部分的话，docker将会假设使用`:latest`标签。

无论在哪里运行`docker run`，它都会拉取你的镜像。宿主机不需要安装任何除了Docker之外的组件。

## 第二部分结论

这就是本节中的所有内容。在下一部分中，我们将会学习如何通过用服务的方式运行我们的容器以扩展我们的应用。

## 总结和备忘单[可选]

下面是一个[控制台视频][terminal-recording]，记录了本教程的内容

[![asciicast](https://asciinema.org/a/107090.png)](https://asciinema.org/a/107090)

下面是本文中用到的一些基本的Docker命令

    # 使用当前目录中的Docekrfile创建镜像
    docker build -t friendlyname .
    # 运行"friendlyname"镜像，映射端口4000到80
    docker run -p 4000:80 friendlyname
    # 与上面一样，但是以后台模式运行
    docker run -d -p 4000:80 friendlyname
    # 列出所有正在运行的容器
    docker container ls
    # 列出所有容器，包含未运行的
    docker container ls -a
    # 平滑的停止指定容器
    docker container stop <hash>
    # 强制关闭指定容器
    docker container kill <hash>
    # 从当前系统中移除指定容器
    docker container rm <hash>
    # 移除所有容器
    docker container rm $(docker container ls -a -q)
    # 列出当前系统中的所有镜像
    docker image ls -a
    # 从当前系统中删除指定的镜像
    docker image rm <image id>
    # 从当前系统中删除所有镜像
    docker image rm $(docker image ls -a -q) 
    # 使用Docker通行证登录CLI会话
    docker login 
    # 为镜像打标签，以上传到Registry
    docker tag <image> username/repository:tag 
    # 上传已经打过标签的镜像到Registry 
    docker push username/repository:tag
    # 从Registry中运行一个镜像
    docker run username/repository:tag
    
[install-docker-113]:https://docs.docker.com/engine/installation/
[tutorial-1]:https://docs.docker.com/get-started/
[tutorial-3]:https://docs.docker.com/get-started/part3/
[tutorial-5]:https://docs.docker.com/get-started/part5/
[docker-trusted-registry]:https://docs.docker.com/datacenter/dtr/2.2/guides/
[cloud-docker]:https://cloud.docker.com/
[docker-images]:https://docs.docker.com/engine/reference/commandline/images/
[terminal-recording]:https://asciinema.org/a/blkah0l4ds33tbe06y4vkme6g
[docker-hub]:https://hub.docker.com/

原文：[Get Started, Part 2: Containers](https://docs.docker.com/get-started/part2/)
