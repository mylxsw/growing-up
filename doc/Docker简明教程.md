# Docker简明教程

本文还 **未完成**，敬请期待...

[TOC]

## 基础命令 - 镜像管理

### 获取镜像

使用`docker pull`命令从网络上下载镜像。

    docker pull NAME[:TAG]
    
例如

    $ docker pull ubuntu
    Using default tag: latest
    latest: Pulling from library/ubuntu
    
    43db9dbdcb30: Downloading 1.494 MB/49.33 MB
    2dc64e8f8d4f: Download complete
    670a583e1b50: Download complete
    43db9dbdcb30: Pull complete
    2dc64e8f8d4f: Pull complete
    670a583e1b50: Pull complete
    183b0bfcd10e: Pull complete
    Digest: sha256:c6674c44c6439673bf56536c1a15916639c47ea04c3d6296c5df938add67b54b
    Status: Downloaded newer image for ubuntu:latest
    
不指定Tag的时候默认使用`:latest`，因此，上述命令实际上是`docker pull ubuntu:latest`。

### 查看镜像信息

使用`docker images`可以列出本地主机上已有的镜像列表。

    $ docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    php                 latest              fe1a2c2228f4        2 days ago          364 MB
    mongo               latest              87bde25ffc68        2 days ago          326.7 MB
    ubuntu              latest              42118e3df429        9 days ago          124.8 MB
    redis               latest              4465e4bcad80        6 weeks ago         185.7 MB
    nginx               latest              0d409d33b27e        8 weeks ago         182.8 MB

还可以使用`docker inspect`命令查看单个镜像的详细信息

    $ docker inspect ubuntu
    [
        {
            "Id": "sha256:42118e3df429f09ca581a9deb3df274601930e428e452f7e4e9f1833c56a100a",
            "RepoTags": [
                "ubuntu:latest"
            ],
            "RepoDigests": [
                "ubuntu@sha256:c6674c44c6439673bf56536c1a15916639c47ea04c3d6296c5df938add67b54b"
            ],
              },
              ...
            "RootFS": {
                "Type": "layers",
                "Layers": [
                    "sha256:ea9f151abb7e06353e73172dad421235611d4f6d0560ec95db26e0dc240642c1",
                    "sha256:0185b3091e8ee299850b096aeb9693d7132f50622d20ea18f88b6a73e9a3309c",
                    "sha256:98305c1a8f5e5666d42b578043e3266f19e22512daa8c6b44c480b177f0bf006",
                    "sha256:9a39129ae0ac2fccf7814b8e29dde5002734c1699d4e9176061d66f5b1afc95c"
                ]
            }
        }
    ]

查看单项信息

    $ docker inspect -f {{".Config.Hostname"}} ubuntu
    827f45722fd6

### 搜索镜像

使用`docker search`命令搜索远程仓库中共享的镜像。

    docker search TERM
    
例如搜索名称为mysql的镜像

    $ docker search mysql
    NAME                DESCRIPTION                 STARS  OFFICIAL  AUTOMATED
    mysql               MySQL is a widely used...   2763   [OK]
    mysql/mysql-server  Optimized MySQL Server...   178    [OK]

### 删除镜像

使用`docker rmi`命令删除镜像。

    docker rmi IMAGE [IMAGE...]

其中IMAGE可以是镜像标签或者ID。

例如

    docker rmi ubuntu
    docker rmi php:7.0.1
    
### 创建镜像

创建镜像有三种方法：

- 基于已有镜像创建
- 基于本地模板导入
- 基于Dockerfile创建

#### 使用已有镜像创建

该方法主要使用`docker commit`命令。

    docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
    
包含以下主要选项

* -a --author=""，作者信息
* -m --message=""，提交信息
* -p --pause=true，提价时暂停容器运行

例如

    $ docker run -i -t ubuntu:latest /bin/bash
    root@5a86b68c4e6a:/# ls
    bin   dev  home  lib64  mnt  proc  run   srv  tmp  var
    boot  etc  lib   media  opt  root  sbin  sys  usr
    root@5a86b68c4e6a:~# exit
    exit

    $ docker commit -m "create a new images" -a "mylxsw" 5a86b68c4e6a test-cont
    sha256:68f1237c24a744b05a934f1317ead38fc68061ade7981eaae158a2ba8da02a9b
    
    $ docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    test-cont           latest              68f1237c24a7        3 seconds ago       124.8 MB
    php                 latest              fe1a2c2228f4        2 days ago          364 MB
    mongo               latest              87bde25ffc68        2 days ago          326.7 MB
    ubuntu              latest              42118e3df429        9 days ago          124.8 MB
    redis               latest              4465e4bcad80        6 weeks ago         185.7 MB
    nginx               latest              0d409d33b27e        8 weeks ago         182.8 MB

#### 基于本地模板导入

#### 基于Dockerfile创建

### 保存镜像文件

使用`docker save`命令保存镜像文件为本地文件。

    docker save -o ubuntu_latest.tar ubuntu:latest

### 载入镜像

使用`docker load`从本地文件再导入镜像压缩包到本地镜像仓库。

    docker load --input ubuntu_latest.tar
    692b4b3b88ff: Loading layer  2.56 kB/2.56 kB
    Loaded image: ubuntu:latest

### 上传镜像

上传镜像使用`docker push`命令。

    docker push NAME[:TAG]
    
默认上传镜像到DockerHub官方仓库。


## 基础命令 - 容器

### 创建容器

使用`docker create`命令创建一个容器，使用该命令创建的容器处于停止状态，需要使用`docker start`命令启动容器。

    docker create -it ubuntu:latest
    
例如：

    $ docker create -it ubuntu
    ddb96bff9de60765a5c10ef91c684e206866a095ec1dae2dbc66924b65d26602
    $ docker ps -a
    CONTAINER ID    IMAGE     COMMAND      CREATED         STATUS   PORTS    NAMES
    ddb96bff9de6    ubuntu   "/bin/bash"  10 seconds ago   Created           grave_shaw

也可以直接使用`docker run`命令创建并启动一个新的容器，等价于执行命令`docker create`和`docker start`。

    $ docker run ubuntu /bin/echo 'Hello world'
    Hello world

下面的命令让docker启动一个bash终端，允许用于与其进行交互

    $ docker run -i -t ubuntu /bin/bash
    root@d808be915a22:/#

> `-t`选项让docker分配一个伪终端并绑定到容器的标准输入上，`-i`则让容器的标准输入保持打开。

大多数情况下，我们希望容器以后台守护进程的形式运行，可以使用`-d`选项。

    $ docker run -d ubuntu /bin/sh -c "while true; do echo hello world; sleep 1;done"
    1927a78fd6e6ca32dbf6a8efe86d83162dd974e6302d930a1766b44142f33804
    $ docker ps
    CONTAINER ID   IMAGE    COMMAND                  CREATED       STATUS    PORTS    NAMES
    1927a78fd6e6   ubuntu  "/bin/sh -c 'while tr"   3 seconds ago  Up 2 seconds prickly_mcclintock
    $ docker logs prickly_mcclintock
    hello world
    hello world
    hello world


### 终止容器

使用`docker stop`命令终止运行中的容器。

    $ docker stop prickly_mcclintock
    prickly_mcclintock

> 容器终止后可以使用`docker start`命令再次启动，也可以对运行的容器执行`docker restart`使其重启。

### 进入容器

使用`-d`选项启动容器后，容器会进入后台运行，用户无法查看容器中的信息。有时候需要进入容器进行操作，可以使用`docker attach`命令以及`docker exec`命令，`nsenter`等工具。

#### attach命令

`docker attach`命令是Docker自带的命令，使用的时候并不太方便，当多个窗口attach到同一个容器，所有窗口都会同步显示。某一个窗口堵塞，其它创建窗口就无法继续进行操作了。

    docker attach prickly_mcclintock
    
> 使用`docker attach`之后，如果使用`Ctrl+C`退出，则容器也会退出运行


#### exec命令

Docker提供了一个更加方便的工具exec，使用它可以直接在容器内运行命令。

    $ docker exec -it 9b3d /bin/bash
    root@9b3d40ebc289:/# ls
    bin   dev  home  lib64  mnt  proc  run   srv  tmp  var
    boot  etc  lib   media  opt  root  sbin  sys  usr

### 删除容器

使用`docker rm`命令删除处于终止状态的容器。

    docker rm [OPTIONS] CONTAINER [CONTAINER...]
    
* -f --force=false 强制终止并删除一个运行中的容器
* -l --link=false 删除容器的连接，但是保留容器
* -v --volumes=false 删除容器挂载的数据卷

### 导出容器

使用导入容器命令可以实现将一个已经创建的容器导出到一个文件，一般可以用于容器的迁移。

    docker export CONTAINER

例如

    docker export 9b3d40 > container-migrate.tar
    
可以将导出的文件传输到其它机器上再进行导入。

### 导入容器

使用`docker import`命令导入容器作为镜像。

    $ cat container-migrate.tar| docker import - test/ubuntu
    sha256:7cae85635deaacdca3120196d9d068d6fc9980b73b2c904b80354a4ece3ceed5
    $ docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    test/ubuntu         latest              7cae85635dea        4 seconds ago       109.9 MB

## 基础命令 - 数据管理

容器数据管理主要有两种方式：

- 数据卷
- 数据卷容器

### 数据卷

数据卷是一个可以供容器使用的特殊目录，它绕过了文件系统，提供了以下特性

- 在容器之间共享和重用
- 修改立马生效
- 对数据卷的更新不会影响镜像
- 卷会一直存在，直到没有容器使用

在运行容器的时候，使用`-v`选项创建数据卷，可以多次使用，创建多个数据卷。

    $ docker run -i -t --name test-vol -v /Users/mylxsw/Downloads:/opt/aicode ubuntu /bin/bash
    root@7ab155e22ec7:/# ls /opt/aicode/
    PHP2016@DevLink  container-migrate.tar  removeDocker.sh  test-cont.tar  ubuntu-test.tar

上述命令将本地的/Users/mylxsw/Downloads目录映射到了容器的/opt/aicode目录。

> 可以指定`:ro`，设置映射目录为只读: `-v /Users/mylxsw/Downloads:/opt/aicode:ro`，同时，`-v`也支持挂载单个文件到容器。

### 数据卷容器

如果用户需要在容器之间共享一些持续更新的数据，最简单的方法是使用数据卷容器。数据卷容器实际上就是一个普通的容器，专美提供数据卷供其他容器使用。

    $ docker run -it -v /backup --name backup ubuntu
    root@be8de791d367:/#

上述命令创建了一个用来作为数据卷的容器，接下来创建几个server容器，用于向该数据卷写入数据，写入数据后，多个容器之间是互通的。

    $ docker run -it --volumes-from backup --name server1 ubuntu

使用`--volumes-from`指定要数据卷容器。

备份数据卷容器中的内容，可以参考以下命令

    docker run --volumes-from backup -v $(pwd):/backup --name worker ubuntu tar cvf /backup/backup.tar /backup
    
恢复则使用下面的命令

    docker run -v /backup --name backup2 ubuntu /bin/bash
    docker run --volumes-from backup2 -v $(pwd):/backup ubuntu tar xvf /backup/backup.tar

## 基础命令 - 网络配置

当Docker启动的时候，它会在宿主主机上创建一个名为`docker0`的虚拟网络接口。

![](media/14700581752688/14913989768427.jpg)

### 查看网络配置

使用下面的命令查看容器的网络详细信息

    docker inspect --format '{{.NetworkSettings}}' 容器名称

![](media/14700581752688/14913993674689.jpg)

如果只需要查看容器的IP，则可以使用下面这样

    # docker inspect --format '{{.NetworkSettings.IPAddress}}' elated_brown
    172.17.0.2

### 容器网络模式

Docker提供了两种网络驱动：**bridge**和**overlay**，默认使用**bridge**驱动。每个docker引擎都会自动包含三个默认的网络：**none**，**bridge**，**host**。

    $ docker network ls
    NETWORK ID          NAME                DRIVER              SCOPE
    de0b57542c6f        bridge              bridge              local
    35c03710cdbf        host                host                local
    519f801ae97a        none                null                local

其中的**bridge**网络就是前面说的**docker0**网络，它是docker安装后自动创建的。**none**网络下的容器不配置网络环境，这时容器不具备访问网络的能力。**host**网络共享主机的网络环境，容器不会再拥有独立的网络空间，与宿主机共享、网卡、主机名、网络配置等。

> **overlay**驱动用于swarm集群模式下，为不同的swarm节点提供网络服务。

使用`docker network create`可以创建一个网络：

    $ docker network create --driver bridge my-bridge-network
    44f32da2819820ffbae9cc18a9c64ec5000f48805b2b81e16bcc02d08fda6aef

    $ docker network ls
    NETWORK ID          NAME                DRIVER              SCOPE
    de0b57542c6f        bridge              bridge              local
    35c03710cdbf        host                host                local
    44f32da28198        my-bridge-network   bridge              local
    519f801ae97a        none                null                local

使用`docker network inspect`可以查看网络的详细信息

    $ docker network inspect my-bridge-network
    [
        {
            "Name": "my-bridge-network",
            "Id": "44f32da2819820ffbae9cc18a9c64ec5000f48805b2b81e16bcc02d08fda6aef",
            "Created": "2017-04-06T12:07:42.949362054Z",
            "Scope": "local",
            "Driver": "bridge",
            "EnableIPv6": false,
            "IPAM": {
                "Driver": "default",
                "Options": {},
                "Config": [
                    {
                        "Subnet": "172.18.0.0/16",
                        "Gateway": "172.18.0.1"
                    }
                ]
            },
            "Internal": false,
            "Attachable": false,
            "Containers": {},
            "Options": {},
            "Labels": {}
        }
    ]

在启动容器的时候，使用`--net`选项指定容器使用哪个网络

    docker run -d --net=my-bridge-network --name db training/postgres

不同网络的容器之间是无法直接通信的，如果需要通信，使用`docker network connect`连接到同一个网桥即可

    docker network connect my-bridge-network web



Docker提供了四种网络模式可供使用，在执行`docker run`命令的时候，可以使用`--net`选项指定采用的模式：

- **--net=bridge**：默认的网络模式，使用网桥来设置容器的网络，默认会连接到`docker0`网桥上
- **--net=host**：共享主机的网络环境，容器不会再拥有独立的网络空间，与宿主机共享、网卡、主机名、网络配置等
- **--net=container:NAME or ID**：容器会共享已经创建好的容器的网络环境，两个容器中的进程可以通过本地回环地址进行访问
- **--net=none**：不配置网络环境，这时容器不具备访问网络的能力

### 暴露网络端口

Docker目前提供了**映射容器端口到宿主主机**和**容器互联**的机制为容器提供网络服务。

#### 端口映射

容器中运行了网络服务，我们可以通过`-P`或者`-p`参数指定端口映射。

* **-P** Docker会随机映射一个49000-49900之间的端口到容器内部的开放端口。
* **-p** 可以指定要映射的端口，格式为`ip:hostPort:containerPort`，可以多次使用`-p`指定多个映射的端口。

例如：

    docker run -d -p 127.0.0.1:5000:5000 training/webapp python app.py
    docker run -d -p 5000:5000 training/webapp python app.py
    docker run -d -p 5000:5000/udp training/webapp python app.py
    docker run -d -P training/webapp python app.py

> 使用`docker port [容器名称] 容器内端口` 查看端口映射绑定的地址。
    
    docker port nostalgic_morse 5000

#### 容器互联通信

容器的链接（linking）系统是除了端口映射外的另一种容器应用之间交互的方式，它会在源和接收容器之间创建一个隧道，接收容器可以看到源容器指定的信息。

容器之间互联通过`--link`参数指定，格式为`--link name:alias`，其中name为要链接到的容器的名称，`alias`为这个连接的别名。

    docker run -d --name mysql-demo -e MYSQL_ROOT_PASSWORD=root mysql
    docker run --rm --name web --link mysql-demo:db ubuntu env
    
![-w567](https://oayrssjpa.qnssl.com/2016-08-13-14710787536308.jpg)

    
使用`docker ps`可以看到容器的连接。

Docker会在两个互联的容器之间创建一个安全的隧道，而且不用映射端口到宿主主机。Docker中通过两种方式为容器公开连接信息：

* **环境变量** 环境变量的方式采用连接别名的大写前缀开头，比如前面的例子中，所有以`DB_`开头的环境变量。
* **更新`/ect/hosts`文件** Docker也会添加host信息到父容器的`/etc/hosts`文件

查看`/etc/hosts`文件：
    
    docker run --rm --name web --link mysql-demo:db -i -t ubuntu /bin/bash

![-w615](https://oayrssjpa.qnssl.com/2016-08-13-14710792151456.jpg)


## Dockerfile

Dockerfile是一个文本格式的配置文件，用户可以使用Dockerfile快速创建自定义的镜像。

### 基本结构

一般来说，Dockerfile分为四部分：

* 基础镜像信息
* 维护者信息
* 镜像操作指令
* 容器启动时执行的指令

### 指令

指令一般格式为`INSTRUCTION arguments`。

#### FROM

格式为`FROM <image>`。第一条指令必须为`FROM`指令，指定了基础镜像。

    FROM ubuntu:latest

#### MAINTAINER

格式为`MAINTAINER <name>`指定维护者信息。

    MAINTAINER mylxsw mylxsw@aicode.cc
    
#### RUN

格式为`RUN <command>`或者`RUN ["executable", "param1", "param2"...]`。每条指令将在当前镜像的基础上执行，并提交为新的镜像。

格式`RUN <command>`时将在shell终端中执行命令，也就是`/bin/sh -c`中执行，而`RUN ["executable", "param1", "param2"...]`则使用`exec`执行。

#### CMD

该命令提供容器启动时执行的命令，每个Dockerfile中只能与一条CMD命令，如果指定了多条，则只有最后一条会被执行。如果用户启动容器的时候指定了运行的命令，则会覆盖CMD指令。

格式支持三种：

* `CMD ["executable", "param1", "param2"] ` 使用exec执行
* `CMD command param1 param2`  使用`/bin/sh -c`执行
* `CMD ["param1", "param2"]` 提供给ENTRYPOINT的默认参数

#### EXPOSE

格式为`EXPOSE <port> [<port>...]`，该指令用于告诉Docker容器要暴露的端口号，供互联系统使用。
    
    EXPOSE 22 80 8443
    
上述指令暴露了22, 80, 8443端口供互联的系统使用，使用的时候可以指定`-P`或者`-p`参数进行端口映射。

#### ENV

格式为`ENV <key> <value>`。指定一个环境变量，会被后续的RUN指令使用，并且在容器运行时保持。

比如：

    ENV PG_MAJOR 9.3
    ENV PG_VERSION 9.35

#### ADD

格式为`ADD <src> <dest>`。该命令复制指定的`<src>`到`<dest>`，其中`<src`可以是Dockerfile所在目录的一个相对路径（文件或目录），也可以是网络上的资源路径或者是tar包。

> 如果`<src>`是tar包的话，会在dest位置**自动解压**为目录。

#### COPY

格式为`COPY <scr> <dest>`，复制本地主机的`<src>`到容器的`<dest>`，目标路径不存在则自动创建。使用本地目录为源目录时，推荐使用COPY。

> 注意，`ADD`命令和`COPY`命令基本上是一样的，只不过是`ADD`命令可以复制网络资源，同时会对压缩包进行自动解压，而`COPY`则是单纯的复制本地文件（目录）。

#### ENTRYPOINT

配置容器启动后执行的命令，并且不会被`docker run`提供的参数覆盖。每个Dockerfile中只能有一个ENTRYPOINT，当指定多个的时候，只有最后一个生效。

格式有两种：

* ENTRYPOINT ["executable", "param1", "param2"]
* ENTRYPOINT command param1 param2

#### VOLUME

格式为`VOLUME ["/data"]`，创建一个可以从本地主机或其它容器挂载的挂载点，一般用来存放数据库和需要保持的数据等。

#### USER

格式为`USER daemon`，用于指定运行容器时的用户名或者UID，后续的RUN命令也会使用指定的用户。

#### WORKDIR

格式为`WORKDIR /path/to/workdir`，用于为后续的RUN，CMD，ENTRYPOINT指令配置工作目录。

可以多次使用，如果后续指定的路径是相对路径，则会基于前面的路径。

    WORKDIR /a
    WORKDIR b
    RUN pwd
    
则最后得到的路径是`/a/b`。

#### ONBUILD

指定基于该镜像创建新的镜像时自动执行的命令。格式为`ONBUILD [INSTRUCTION]`。

### 创建镜像

编写完Dockerfile之后，就可以通过`docker build`命令构建一个镜像了。

> 可以通过`.dockerignore`指定忽略的文件和目录，类似于git中的`.gitignore`文件。

比如

    docker build -t build_repo/first_image /tmp/docker_builder
    

---

参考：

- [Docker技术入门与实战 杨保卫 戴王剑 曹亚伦编著](https://book.douban.com/subject/26284823/)

