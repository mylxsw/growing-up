# CentOS 7 部署MongoDB复制集模式

[TOC]

## 架构

![Xnip2019-02-28_17-14-28](https://ssl.aicode.cc/Xnip2019-02-28_17-14-28.jpg)


三台Server:

- **A** 192.168.100.10 `Primary/Secondary`
- **B** 192.168.100.11 `Primary/Secondary`
- **C** 192.168.100.12 `Arbiter`

## 安装MongoDB

创建复制集访问控制密钥

	openssl rand -base64 756 > mongodb.key
	chmod 400 mongodb.key

以下操作三台服务器都需要做一遍

### 添加yum源

在 **/etc/yum.repo.d**目录下创建 **mongodb.repo** 文件

	[mongodb-org-4.0]
	name=MongoDB Repository
	baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.0/x86_64/
	gpgcheck=1
	enabled=1
	gpgkey=https://www.mongodb.org/static/pgp/server-4.0.asc

### 安装Mongod

	yum install -y mongodb-org

## 配置MongoDB

创建存储目录

	mkdir /data/mongodb
	chown mongod:mongod /data/mongodb

复制集访问控制文件keyfile，将生成的 **mongodb.key** 文件复制到 **/data/security/mongodb.key**。

	mkdir /data/security
	cp mongodb.key /data/security/mongodb.key # 需要根据实际mongodb.key所在的目录，将其复制到security目录
	chown mongod:mongod /data/security
	chmod 400 /data/security/mongodb.key

修改mongodb配置文件 **/etc/mongod.conf**，复制集的名称我们使用`aicode`

	systemLog:
	  destination: file
	  logAppend: true
	  path: /var/log/mongodb/mongod.log

	storage:
	  dbPath: /data/mongodb
	  journal:
		enabled: true
	
	processManagement:
	  fork: true
	  pidFilePath: /var/run/mongodb/mongod.pid

	net:
	  port: 27017
	  bindIp: 0.0.0.0

	security:
	  authorization: enabled
	  keyFile: /data/security/mongodb.key

	replication:
	  replSetName: aicode

启动服务

	systemctl enable mongod
	systemctl start mongod

## 创建复制集

在A服务器上面，连接到mongodb

### 创建管理帐号

	# 创建管理员帐号
	use admin
	db.createUser({user: "admin", pwd: "password", roles: ["root"]})

### 创建复制集

	use admin
	db.auth('admin', 'password')
	
	# 初始化复制集
	rs.initiate({_id: "aicode", members: [{_id: 1, host: "192.168.100.10:27017"}]})
	# 添加节点
	rs.add("192.168.100.11:27017")
	# 添加Arbiter节点
	rs.addArb("192.168.100.12:27017)

### 查看状态

	rs.status()

## 连接到复制集


	mongo --host aicode/192.168.100.10:27017,192.168.100.11:27017
