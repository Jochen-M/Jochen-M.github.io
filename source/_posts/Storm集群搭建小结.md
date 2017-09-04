---
title: Storm集群搭建小结
date: 2017-07-13 13:23:11
tags:
    - Storm
    - Java
categories:
    - Big Data
---

### 环境
+ 五台Ubuntu 16.04；
+ 关闭防火墙，配置hosts文件；
+ 安装java并配置环境变量；
+ 下载zookeeper-3.4.10.tar.gz、apache-storm-0.9.1.tar.gz;

### 安装zookeeper
解压zookeeper-3.4.10.tar.gz到/home/hadoop/目录
#### 建立zookeeper的data目录
```
mkdir /home/hadoop/zookeeper-3.4..10/data
```
#### 编辑配置文件
home/hadoop/zookeeper-3.4.10/conf/zoo.cfg:
```
tickTime=2000
dataDir=/home/hadoop/zookeeper-3.4.10/data/
clientPort=2181
initLimit=5
syncLimit=2
server.1=master:2888:3888
server.2=node1:2888:3888
server.3=node2:2888:3888
server.4=node3:2888:3888
server.5=node4:2888:3888
```
#### 建立zookeeper集群的myid文件
```
echo  “1” > home/hadoop/zookeeper-3.4.10/data/myid
```

### 安装Storm
解压apache-storm-0.9.1.tar.gz到/home/hadoop目录下
#### 创建本地数据目录
```
mkdir -p /home/hadoop/apache-storm-0.9.1/data
```
#### 配置conf/storm.yaml文件：
```
storm.zookeeper.servers:
    - 'master'
    - 'node1'
    - 'node2'
    - 'node3'
    - 'node4'
  nimbus.host: 'master'
  supervisor.slots.ports:
    - 6700
    - 6701
    - 6702
    - 6703
  storm.local.dir: '/home/hadoop/apache-storm-0.9.1-incubating/data'
```

### 启动集群
#### 启动zookeeper：
在zookeeper根目录下执行：
```
bin/zkServer.sh start
```
查看启动状态:
```
bin/zkServer.sh status
```
若启动成功，则一台为leader，另外四台为follower。
#### 启动nimbus：
在nimbus主机的strom根目录下执行：
```
# 启动主节点
nohup bin/storm nimbus > /dev/null 2>&1 &
# 启动web-ui
nohup bin/storm ui > /dev/null 2>&1 &
# 启动logviewer功能
nohup bin/storm logviewer > /dev/null 2>&1 &
```
#### 启动supervisor：
在supervisor的storm根目录下执行：
```
nohup bin/storm supervisor > /dev/null 2>&1 &
nohup bin/storm logviewer > /dev/null 2>&1 &
```
#### 检查
打开浏览器，输入<code>master：8080</code>，即可看到集群信息。

### 部署程序
使用maven + IDEA 开发wordcount程序
#### 添加maven依赖：
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>storm</groupId>
    <artifactId>StormWordCount</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>org.apache.storm</groupId>
            <artifactId>storm-core</artifactId>
            <version>0.9.3</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.17</version>
        </dependency>
    </dependencies>
</project>
```
#### 编写程序

#### 打包部署
进入项目根目录执行：
```
mvn clean install -DskipTests=true
mvn package
```
两条命令成功执行，将会生成jar包。

进入storm根目录，执行以下命令提交任务到集群：
```
bin/storm jar {WordCount}/target/*.jar TopologyMain
```
#### 通过Storm UI即可查看任务提交和运行情况
![Alt text](/uploads/storm-cluster-summary.png)
![Alt text](/uploads/storm-topology.png)

#### 关闭topology：
在Storm UI中点击相应任务，在新页面中点击kill即可。
