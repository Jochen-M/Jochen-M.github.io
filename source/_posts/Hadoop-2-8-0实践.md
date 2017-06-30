---
title: Hadoop-2.8.0实践
date: 2017-06-28 19:29:29
tags:
    - Hadoop
categories:
    - 大数据
---

### 安装
#### 准备
+ 系统： Linux最佳，Windows亦可（本教程基于Linux Ubuntu 16.04 LTS）
+ 软件：
    * Java 1.7及以上
    * ssh 和 sshd
+ 安装ssh:
```
$ sudo apt-get install ssh
$ sudo apt-get install rsync
```

#### 下载 Hadoop 2.8.0
下载地址：http://mirror.bit.edu.cn/apache/hadoop/common/hadoop-2.8.0/
选择下载 hadoop-2.8.0.tar.gz，并解压。

#### 为 Hadoop 配置 Java 路径
编辑 etc/hadoop/hadoop-env.sh：
```
export JAVA_HOME=/path/to/java/root/dir
```

#### 启动 Hadoop
运行命令： 
```
bin/hadoop
```
若打印出帮助信息，则表示安装成功。

### 标准模式（单机模式）操作
默认情况下，Hadoop被配置为以非分布式模式运行，作为一个单一的Java进程。这对于调试非常有用。
下面的示例复制未打包的conf目录作为输入，然后找到并显示给定正则表达式的每一个匹配项。输出被写入到给定的输出目录。
```
$ mkdir input
$ cp etc/hadoop/*.xml input
$ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.8.0.jar grep input output 'dfs[a-z.]+'
$ cat output/*
```

### 伪分布模式操作
Hadoop还可以在一个伪分布模式下运行，每个Hadoop守护进程在一个单独的Java进程中运行。
#### 配置
etc/hadoop/core-site.xml:
```
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```

etc/hadoop/hdfs-site.xml:
```
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```

#### ssh免密码连接本地主机
检查是否可以使用ssh到本地主机，而无需使用密码:
```
$ ssh localhost
```

如果不能在没有密码的情况下ssh到localhost，请执行以下命令:
```
$ ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
$ chmod 0600 ~/.ssh/authorized_keys
```

#### 执行
##### 格式文件系统:
```
$ bin/hdfs namenode -format
```

##### 启动NameNode守护进程和DataNode守护进程:
```
$ sbin/start-dfs.sh
```

##### 浏览NameNode的web界面;默认情况下:
http://localhost:50070/

##### 创建执行MapReduce作业所需的HDFS目录:
```
$ bin/hdfs dfs -mkdir /user
$ bin/hdfs dfs -mkdir /user/<username>
```

##### 将输入文件复制到分布式文件系统中:
```
$ bin/hdfs dfs -put etc/hadoop input
```

##### 运行示例:
```
$ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.8.0.jar grep input output 'dfs[a-z.]+'
```

##### 检查输出文件
将输出文件从分布式文件系统复制到本地文件系统，并检查它们:
```
$ bin/hdfs dfs -get output output
$ cat output/*
```

或者 查看分布式文件系统上的输出文件:
```
$ bin/hdfs dfs -cat output/*
```

##### 当完成这些，可以停止守护进程：
```
$ sbin/stop-dfs.sh
```

#### 单节点YARN
可以通过设置一些参数和运行ResourceManager守护进程及NodeManager守护进程，在伪分布模式下运行MapReduce作业。
（以下操作假设以上说明的前四步均已执行！）

##### 配置
etc/hadoop/mapred-site.xml:
```
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

etc/hadoop/yarn-site.xml:
```
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
```

##### 启动ResourceManager守护进程和NodeManager守护进程
```
$ sbin/start-yarn.sh
```

##### 浏览ResourceManager web界面;默认情况下：
http://localhost:8088/

##### 运行MapReduce作业

##### 完成后，停止守护进程：
```
$ sbin/stop-yarn.sh
```

### HDFS - Hadoop 分布式文件系统

### MapReduce - Map & Reduce

### YARN - Hadoop 资源管理器
+ YARN的基本思想是将资源管理和作业调度/监控的功能拆分到不同的守护进程。这种思想需要有一个全局的资源管理器（RM）和（每个应用程序都要有的）应用程序管理器（AM）。
+ 资源管理器（RM）和节点管理器（NodeManager）形成了数据计算框架。资源管理器（RM）是在系统中所有应用程序间仲裁资源的最终权威。节点管理器（NodeManager）是每台机器的框架代理，负责容器的管理，监控他们的资源使用情况(cpu、内存、磁盘、网络)，并向资源管理器（RM）/调度器报告该情况。
+ 每个应用程序的应用程序管理器（AM）实际上是一个特定的框架的库，它的任务是与资源管理器（RM）协商资源，并与节点管理器（NodeManager）一起工作来执行和监视任务。
+ 资源管理器（RM）有两个主要组件:调度程序和应用程序管理器（AM）。
    * 调度程序负责将资源分配给各种运行的应用程序。调度程序是纯粹的调度器，因为它不执行应用程序的状态监视或跟踪。另外，它也不能保证重新启动失败的任务，无论是由于应用程序失败还是硬件故障。
    * 应用程序管理器（AM）负责接收提交的工作，协商执行应用程序的第一个容器，并并提供在失败时重新启动应用程序管理器(AM)容器的服务。每个应用程序管理器(AM)负责从调度程序中协商适当的资源容器，跟踪它们的状态并监视进程。
+ YARN 还支持资源预定的概念，保留资源以确保重要工作的可预见性执行。预订系统会对资源进行跟踪，对预订进行控制，并动态地指导底层的调度程序，以确保预订是满的。

![Alt text](/uploads/hadoop_yarn.png)

> 推荐：
> Apache Hadoop: http://hadoop.apache.org/
> Hadoop Commands Guide: http://hadoop.apache.org/docs/r2.8.0/hadoop-project-dist/hadoop-common/CommandsManual.html
> Apache Hadoop Main 3.0.0-alpha3 API: http://hadoop.apache.org/docs/current/api/
> 我的另一篇博文： [《Hadoop大数据平台架构与实践》](https://jochen-m.github.io/2017/05/19/Hadoop%E5%A4%A7%E6%95%B0%E6%8D%AE%E5%B9%B3%E5%8F%B0%E6%9E%B6%E6%9E%84%E4%B8%8E%E5%AE%9E%E8%B7%B5/)