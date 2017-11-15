# 中小团队日志集中式处理方案-ELK架构

[TOC]

## 环境安装

为了方便演示，我使用 **vagrant** 创建了两个 **centos/7** 的虚拟机进行演示。

- 虚拟机 **192.168.100.10** 安装整个**ELK**服务，这里只安装单机版，集群扩展以后再做介绍
- 虚拟机 **192.168.100.11** 作为演示用的应用服务器，安装**Filebeat**进行日志收集

### 应用服务

> 这一步也可以忽略，在自己的电脑上安装 **filebeat** 收集本地日志发送到ELK服务器也可以。

首先创建一台用于演示用的应用服务器虚拟机，这里我们使用一个**Laravel**项目进行演示

    mkdir app-server && cd app-server
    vagrant init laravel/homestead

创建后，修改**Vagrantfile**

    Vagrant.configure("2") do |config|
      config.vm.box = "laravel/homestead"
      config.vm.network "private_network", ip: "192.168.100.11"
    end

启动虚拟机

    # 启动虚拟机
    vagrant up
    # 启动后，登录到虚拟机
    vagrant ssh



### ELK 服务

首先创建一台用于安装ELK服务的虚拟机

    mkdir elk && cd elk
    vagrant init centos/7

创建后，修改**Vagrantfile**

    Vagrant.configure("2") do |config|
      config.vm.box = "centos/7" 
      config.vm.network "private_network", ip: "192.168.100.10"
     
      config.vm.provider "virtualbox" do |vb|
        vb.memory = "2048"
      end
    end

启动虚拟机

    # 启动虚拟机
    vagrant up
    # 启动后，登录到虚拟机
    vagrant ssh

在虚拟机中，创建**elk**的yum仓库配置，将下面的内容保存为`/etc/yum.repos.d/elk-6.repo`

    [logstash-6.x]
    name=Elastic repository for 6.x packages
    baseurl=https://artifacts.elastic.co/packages/6.x/yum
    gpgcheck=1
    gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
    enabled=1
    autorefresh=1
    type=rpm-md

    [elasticsearch-6.x]
    name=Elasticsearch repository for 6.x packages
    baseurl=https://artifacts.elastic.co/packages/6.x/yum
    gpgcheck=1
    gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
    enabled=1
    autorefresh=1
    type=rpm-md
    
    [kibana-6.x]
    name=Kibana repository for 6.x packages
    baseurl=https://artifacts.elastic.co/packages/6.x/yum
    gpgcheck=1
    gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
    enabled=1
    autorefresh=1
    type=rpm-md
    
接下来就可以安装 **Logstash**，**ElasticSearch**，**Kibana** 三个软件了，注意的是，因为**Logstash**和**ElasticSearch**依赖于Java8环境，因此还要先安装JDK8。

    # 安装open JDK
    sudo yum install java-1.8.0-openjdk -y
    # 安装ELK
    sudo yum install logstash elasticsearch kibana -y

安装完成后，三个软件分别安装在了

- /usr/share/logstash
- /usr/share/elasticsearch
- /usr/share/kibana


## Logstash

Logstash启动关闭命令

    initctl start logstash
    initctl stop logstash

**/etc/logstash/conf.d/biz-logs.conf**

<script src="https://gist.github.com/mylxsw/48f2981274a259495ea0a8d027bcd239.js"></script>

需要注意的几个配置（**/etc/logstash/logstash.yml**）
    
    # 配置Logstash数据存储目录，Logstash插件的数据将会存储到这个目录
    path.data: /var/lib/logstash
    # 配置Logstash日志文件目录
    path.logs: /data/logs/logstash

## ElasticSearch

ElasticSearch启动关闭命令

    service elasticsearch start
    service elasticsearch stop

需要注意的几个配置项

    # 集群的名字
    cluster.name: yunsom-log-elastic
    # Elasticsearch数据存储目录，这个目录比较重要，也会随着数据量的增多而变大，建议修改
    path.data: /data/elasticsearch
    # Elasticsearch日志存储目录
    path.logs: /data/logs/elasticsearch
    # Elasticsearch请求端口
    http.port: 9200
    # ElasticSearch绑定的HTTP地址，如果绑定为127.0.0.1则无法通过其它外部主机进行访问
    # 正式环境中建议使用内网地址，比如192.168.0.0等
    network.host: 0.0.0.0

## Kibana

Kibana启动关闭命令

    service kibana start
    service kibana stop

需要注意的几个配置项

    # Kibana监听的地址
    server.host: "0.0.0.0"
    # Kibana监听的端口
    server.port: 35601
    # ElasticSearch服务地址
    # 注意ElasticSearch绑定的地址是否可以访问
    elasticsearch.url: "http://localhost:9200"


## 扩展

### 安装配置X-Pack

### ElasticSearch索引定期清理

创建脚本 **/data/scripts/clear-elasticsearch-index.sh**

    #!/bin/bash
    #定时清除elk索引，15天
    DATE=`date -d "15 days ago" +%Y.%m.%d`
    
    curl -X DELETE "http://127.0.0.1:9200/logstash-$DATE"
    curl -X DELETE "http://127.0.0.1:9200/nginx-access-$DATE"
    curl -X DELETE "http://127.0.0.1:9200/php-slow-$DATE"
    curl -X DELETE "http://127.0.0.1:9200/mysql-slow-$DATE"

创建完成后，增加可执行权限

    chmod +x /data/scripts/clear-elasticsearch-index.sh

之后，配置定时任务，每天执行一次

    # 定时每天1点清理过期的elastic日志索引
    0 1 * * * /bin/bash /data/scripts/clear-elasticsearch-index.sh

## 常见问题

1. 启动Vagrant的时候，报错**yum install -y kernel-devel-`uname -r` gcc binutils make perl bzip2** 执行失败

    忽略，直接使用`vagrant ssh`登录，之后修改dns地址即可
    
        # 修改dns服务器
        sudo sh -c "echo 'nameserver 8.8.8.8' > /etc/resolv.conf"
        # 防止重启后dns备还原
        sudo chattr +i /etc/resolv.conf

