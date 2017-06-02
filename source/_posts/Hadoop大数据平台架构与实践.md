---
title: Hadoop大数据平台架构与实践
date: 2017-05-19 22:06:37
tags:
    - Hadoop
categories:
    - 大数据
---

### What Is Apache Hadoop?
> The Apache™ Hadoop® project develops open-source software for reliable, scalable, distributed computing.
The Apache Hadoop software library is a framework that allows for the distributed processing of large data sets across clusters of computers using simple programming models. It is designed to scale up from single servers to thousands of machines, each offering local computation and storage. Rather than rely on hardware to deliver high-availability, the library itself is designed to detect and handle failures at the application layer, so delivering a highly-available service on top of a cluster of computers, each of which may be prone to failures.

<!-- more -->

> #### The project includes these modules:
+ *Hadoop Common*: The common utilities that support the other Hadoop modules.
+ *Hadoop Distributed File System (HDFS™)*: A distributed file system that provides high-throughput access to application data.
+ *Hadoop YARN*: A framework for job scheduling and cluster resource management.
+ *Hadoop MapReduce*: A YARN-based system for parallel processing of large data sets.

> #### And other Hadoop-related projects at Apache.


### Hadoop安装（以hadoop-1.2.1为例）
#### 准备条件
* Linux操作系统
* 安装JDK以及配置相关环境变量
* 下载Hadoop安装包，如：hadoop-1.2.1.tar.gz（官网下载地址：[http://hadoop.apache.org/releases.html](http://hadoop.apache.org/releases.html)）

#### 安装
将hadoop-1.2.1.tar.gz解压到指定目录，如：/opt/hadoop-1.2.1/

#### 配置hadoop环境变量
在/etc/profile中配置如下信息：
```
export JAVA_HOME=/opt/jdk1.8.0_131
export JRE_HOME=/opt/jdk1.8.0_131/jre
export HADOOP_HOME=/opt/hadoop-1.2.1
export CLASSPATH=.:$CLASSPATH:$JAVA_HOME/lib:$JRE_HOME/Lib
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$HADOOP_HOME/bin:$PATH
```

#### 修改四个配置文件
这四个配置文件均在/opt/hadoop-1.2.1/conf/目录下。

* (a)修改hadoop-env.sh,设置JAVA_HOME:
```
# The java implementation to use.  Required.
export JAVA_HOME=/opt/jdk1.8.0_131
```
* (b)修改core-site.xml,设置hadoop.tmp.dir,dfs.name.dir,fs.default.name:
```
<configuration>
  <property>
    <name>hadoop.tmp.dir</name>     <!-- hadoop临时工作目录 -->
    <value>/home/jochen/hadoop</value>
  </property>

  <property>
    <name>dfs.name.dir</name>       <!-- hadoop源数据目录 -->
    <value>/home/jochen/hadoop/name</value>
  </property>

  <property>
    <name>fs.default.name</name>    <!-- 文件系统namenode => 地址：端口号 -->
    <value>hdfs://localhost:9000</value>
  </property>
</configuration>
```
* (c)修改mapred-site.xml,设置mapred.job.tracker:
```
<configuration>
  <property>
    <name>mapred.job.tracker</name>
    <value>localhost:9001</value>
  </property>
</configuration>
```
* (d)修改hdfs-site.xml,设置dfs.data.dir:
```
<configuration>
  <property>
    <name>dfs.data.dir</name>       <!-- dfs文件块存放目录 -->
    <value>/home/jochen/hadoop/data</value>
  </property>
</configuration>
```

#### 格式化
执行命令：
```
$ hadoop namenode -format
```
正确执行的结果如下所示：
```
Warning: $HADOOP_HOME is deprecated.

17/05/19 23:46:05 INFO namenode.NameNode: STARTUP_MSG: 
/************************************************************
STARTUP_MSG: Starting NameNode
STARTUP_MSG:   host = ubuntu/127.0.0.1
STARTUP_MSG:   args = [-format]
STARTUP_MSG:   version = 1.2.1
STARTUP_MSG:   build = https://svn.apache.org/repos/asf/hadoop/common/branches/branch-1.2 -r 1503152; compiled by 'mattf' on Mon Jul 22 15:23:09 PDT 2013
STARTUP_MSG:   java = 1.8.0_131
************************************************************/
17/05/19 23:46:05 INFO util.GSet: Computing capacity for map BlocksMap
17/05/19 23:46:05 INFO util.GSet: VM type       = 64-bit
17/05/19 23:46:05 INFO util.GSet: 2.0% max memory = 932184064
17/05/19 23:46:05 INFO util.GSet: capacity      = 2^21 = 2097152 entries
17/05/19 23:46:05 INFO util.GSet: recommended=2097152, actual=2097152
17/05/19 23:46:05 INFO namenode.FSNamesystem: fsOwner=jochen
17/05/19 23:46:05 INFO namenode.FSNamesystem: supergroup=supergroup
17/05/19 23:46:05 INFO namenode.FSNamesystem: isPermissionEnabled=true
17/05/19 23:46:05 INFO namenode.FSNamesystem: dfs.block.invalidate.limit=100
17/05/19 23:46:05 INFO namenode.FSNamesystem: isAccessTokenEnabled=false accessKeyUpdateInterval=0 min(s), accessTokenLifetime=0 min(s)
17/05/19 23:46:05 INFO namenode.FSEditLog: dfs.namenode.edits.toleration.length = 0
17/05/19 23:46:05 INFO namenode.NameNode: Caching file names occuring more than 10 times 
17/05/19 23:46:05 INFO common.Storage: Image file /home/jochen/hadoop/dfs/name/current/fsimage of size 112 bytes saved in 0 seconds.
17/05/19 23:46:06 INFO namenode.FSEditLog: closing edit log: position=4, editlog=/home/jochen/hadoop/dfs/name/current/edits
17/05/19 23:46:06 INFO namenode.FSEditLog: close success: truncate to 4, editlog=/home/jochen/hadoop/dfs/name/current/edits
17/05/19 23:46:06 INFO common.Storage: Storage directory /home/jochen/hadoop/dfs/name has been successfully formatted.
17/05/19 23:46:06 INFO namenode.NameNode: SHUTDOWN_MSG: 
/************************************************************
SHUTDOWN_MSG: Shutting down NameNode at ubuntu/127.0.0.1
************************************************************/
```

#### 启动
```
$ cd /opt/hadoop-1.2.1/bin
$ ./start-all.sh
```

#### 查看当前运行的java进程
在Terminal输入命令，出现如下结果表示hadoop安装成功：
```
$ jps
12785 JobTracker
1161 Jps
23626 TaskTracker
23275 DataNode
21659 NameNode
23436 SecondaryNameNode
```