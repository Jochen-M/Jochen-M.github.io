---
title: MongoDB数据实时导入elasticsearch
date: 2017-12-01 10:43:57
tags:
    - MongoDB
    - elasticsearch
categories:
    - MongoDB
    - elasticsearch
---

项目中需要实现对数据的全文检索功能，数据主要存储在了 MongoDB 中。MongoDB 本身是自带文本检索功能的，但是不支持中文，而且当数据量增大时，MongoDB 的检索效率会大大降低。

由于最近在学习 Elasticsearch，而 Elasticsearch 的特性又十分适合全文检索，于是就选择了它。

那么如何在对 MongoDB 进行增删改查时，实时地将 MongoDB 的数据导入 Elasticsearch 中呢？

经过几番调研，发现 mongo-connecter 是实现这个功能的利器。这个是 MongoDB 官方的开发人员用 Python 写的一个工具，目前支持将 MongoDB 的数据同步到 Solr、ElasticSearch 中，并且支持用户自己扩展。

下面是具体的部署过程：

### 搭建 MongoDB 副本集

#### MongoDB 副本集的架构
副本集是 mongodb 提供的一种高可用解决方案。相对于原来的主从复制，副本集能自动感知 Primary 节点的下线，并提升其中一个 Secondary 节点作为 Primary 。整个过程对业务透明，同时也大大降低了运维的成本。

架构图如下：
![Alt text](/uploads/mongodb_replset.png)

#### MongoDB 副本集的角色

##### Primary
默认情况下，读写都是在 Primary 上操作的。

##### Secondary

+ 通过 oplog 来重放 Primary 上的所有操作，拥有 Primary 节点数据的完整拷贝。
+ 默认情况下，不可写，也不可读。
+ 根据不同的需求，Secondary 又可配置为如下形式：
  * Priority 0 Replica Set Members - 优先级为0的节点

    优先级为0的成员永远不会被选举为 primary。在mongoDB副本集中，允许给不同的节点设置不同的优先级。优先级的取值范围为0-1000，可设置为浮点数，默认为1。拥有最高优先级的成员会优先选举为primary。譬如，在副本集中添加了一个优先级为2的成员node3:27020，而其它成员的优先级为1，只要node3:27020拥有最新的数据，那么当前的primary就会自动降级，node3:27020将会被选举为新的primary节点，但如果node3:27020中的数据不够新，则当前primary节点保持不变，直到node3:27020的数据更新到最新。
  * Hidden Replica Set Members - 隐藏节点

    隐藏节点的优先级同样为0，同时对客户端不可见使用rs.status()和rs.config()可以看到隐藏节点，但是对于db.isMaster()不可见。客户端连接到副本集时，会调用db.isMaster()命令来查看可用成员信息。所以，隐藏节点不会受到客户端的读请求。隐藏节点常用于执行特定的任务，譬如报表，备份。
  * Delayed Replica Set Members - 延迟节点
    延迟节点会比primary节点延迟指定的时间（通过slaveDelay参数来指定）。延迟节点必须是隐藏节点。

##### Arbiter
仲裁节点，只是用来投票，且投票的权重只能为1，不复制数据，也不能提升为primary。仲裁节点常用于节点数量是偶数的副本集中。建议：通常将Arbiter部署在业务服务器上，切忌将其部署在Primary节点或Secondary节点服务器上。

**注： 一个副本集最多有50个成员节点，7个投票节点。**

#### MongoDB 副本集的搭建
##### 创建数据目录和日志目录
```
mkdir -p data/27017/ data/27018/ data/27019/
mkdir -p log/
```

##### 启动 mongod 实例
```
mongod --replSet myrs --dbpath data/27017/ --port 27017 --logpath log/27017.log --fork
mongod --replSet myrs --dbpath data/27018/ --port 27018 --logpath log/27018.log --fork
mongod --replSet myrs --dbpath data/27019/ --port 27019 --logpath log/27019.log --fork
```

##### 搭建副本集
连接副本集任意成员，如 27017 端口：
```
mongo --port 27017
```

初始化副本集:
```
> rs.initiate()
{
	"info2" : "no configuration specified. Using a default configuration for the set",
	"me" : "ubuntu:27017",
	"ok" : 1
}
```

添加节点:
```
myrs:SECONDARY> rs.add("ubuntu:27018")
{ "ok" : 1 }
```

添加仲裁节点:
```
myrs:PRIMARY> rs.addArb("ubuntu:27019")
{ "ok" : 1 }
```

通过 rs.conf() 查看副本集配置信息：
```
myrs:PRIMARY> rs.conf()
{
	"_id" : "myrs",
	"version" : 3,
	"protocolVersion" : NumberLong(1),
	"members" : [
		{
			"_id" : 0,
			"host" : "ubuntu:27017",
			"arbiterOnly" : false,
			"buildIndexes" : true,
			"hidden" : false,
			"priority" : 1,
			"tags" : {

			},
			"slaveDelay" : NumberLong(0),
			"votes" : 1
		},
		{
			"_id" : 1,
			"host" : "ubuntu:27018",
			"arbiterOnly" : false,
			"buildIndexes" : true,
			"hidden" : false,
			"priority" : 1,
			"tags" : {

			},
			"slaveDelay" : NumberLong(0),
			"votes" : 1
		},
		{
			"_id" : 2,
			"host" : "ubuntu:27019",
			"arbiterOnly" : true,
			"buildIndexes" : true,
			"hidden" : false,
			"priority" : 1,
			"tags" : {

			},
			"slaveDelay" : NumberLong(0),
			"votes" : 1
		}
	],
	"settings" : {
		"chainingAllowed" : true,
		"heartbeatIntervalMillis" : 2000,
		"heartbeatTimeoutSecs" : 10,
		"electionTimeoutMillis" : 10000,
		"catchUpTimeoutMillis" : 60000,
		"getLastErrorModes" : {

		},
		"getLastErrorDefaults" : {
			"w" : 1,
			"wtimeout" : 0
		},
		"replicaSetId" : ObjectId("5a20bb163d61965112afa99c")
	}
}
```

##### 验证副本集的可用性
在 primary 中创建一个数据库，并插入一个文档：
```
myrs:PRIMARY> use test
switched to db test
myrs:PRIMARY> db.blog.insert({"title": "hello world"});
WriteResult({ "nInserted" : 1 })
```

在 secondary 中进行验证：
```
myrs:SECONDARY> show dbs
2017-12-01T10:17:05.138+0800 E QUERY    [thread1] Error: listDatabases failed:{
	"ok" : 0,
	"errmsg" : "not master and slaveOk=false",
	"code" : 13435,
	"codeName" : "NotMasterNoSlaveOk"
} :
_getErrorWithCode@src/mongo/shell/utils.js:25:13
Mongo.prototype.getDBs@src/mongo/shell/mongo.js:62:1
shellHelper.show@src/mongo/shell/utils.js:769:19
shellHelper@src/mongo/shell/utils.js:659:15
@(shellhelp2):1:1
myrs:SECONDARY> rs.slaveOk()
myrs:SECONDARY> show dbs
admin  0.000GB
local  0.000GB
test   0.000GB
myrs:SECONDARY> use test
switched to db test
myrs:SECONDARY> db.blog.find()
{ "_id" : ObjectId("5a20bc0b39b7d2896cd3765a"), "title" : "hello world" }
```

因仲裁节点实际上并不存储任何数据，所以无法通过连接仲裁节点查看刚刚插入的文档
```
myrs:ARBITER> show dbs
2017-12-01T12:36:57.429+0800 E QUERY    [thread1] Error: listDatabases failed:{
	"ok" : 0,
	"errmsg" : "not master and slaveOk=false",
	"code" : 13435,
	"codeName" : "NotMasterNoSlaveOk"
} :
_getErrorWithCode@src/mongo/shell/utils.js:25:13
Mongo.prototype.getDBs@src/mongo/shell/mongo.js:62:1
shellHelper.show@src/mongo/shell/utils.js:769:19
shellHelper@src/mongo/shell/utils.js:659:15
@(shellhelp2):1:1
myrs:ARBITER> rs.slaveOk()
myrs:ARBITER> show dbs
local  0.000GB
```

### 启动 elasticsearch

### 安装并运行 mongo-connector
#### 安装 pip
```
sudo apt-get install python-pip
```

#### 安装 mongo-connector
```
pip install mongo-connector
```

#### 运行 mongo-connector
```
mongo-connector -m 127.0.0.1:27017 -t 127.0.0.1:9200 -d elastic_doc_manager
```

即可在 elasticsearch 中看到数据，同时可以动态（有一点延迟）看到 MongoDB 的增删改查。
