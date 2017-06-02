---
title: Ubuntu + Docker 学习笔记
date: 2017-05-02 22:42:41
tags: 
  - Ubuntu
  - Docker
categories: 
  - Docker
---

## 什么是Docker
+ Docker is the world’s leading software container platform.
+ Docker公司开发，开源，托管在github
+ 跨平台，支持Windows、Macos、Linux

<!-- more -->

## Docker思想
+ 集装箱
+ 标准化，包括运输方式、存储方式、API接口标准化
+ 隔离（类似于虚拟机，不过更加轻量）


## Docker核心技术
##### 镜像（集装箱）--Build
![Alt text](/uploads/docker-image.png)
##### 仓库（超级码头）--Ship
hub.docker.com 官方
c.163.com 网易蜂巢
![Alt text](/uploads/docker-repository.png)
##### 容器（运行程序的地方）--Run
![Alt text](/uploads/docker.png)


## Linux(Ubuntu)安装Docker

##### 安装
```
sudo apt-get update
sudo apt-get install -y docker.io
// 若要安装最新版本：
curl -s https://get.docker.com | sh
```

##### 检查是否安装成功
```
service docker start
docker version
```
若出现如下结果，表示安装成功：
![](/uploads/docker-success.png)


##### 第一个Docker镜像：hello-world
```
docker pull hello-world    // 拉取镜像
docker images    // 查看本地镜像
docker run hello-world    // 运行
```

##### 运行nginx镜像
```
docker pull hub.c.163.com/library/nginx:latest    // 地址来自上述网易蜂巢仓库
docker run hub.c.163.com/library/nginx    // 前台挂起运行
docker run -d hub.c.163.com/library/nginx    // 后台运行
docker ps    // 查看运行中的容器
docker exec -it ID bash    // 进入容器shell

```

##### Docker网络
###### 网络类型 
Bridge： 容器映射到本地主机
Host： 容器与主机共用端口
None： 没有网络，Docker不与外界通讯

###### 端口映射:
```
docker run -d -p 8080:80 hub.c.163.com/library/nginx    // 本地8080与容器80映射    
docker run -d -P hub.c.163.com/library/nginx    // 所有监听端口与主机映射
```

##### 制作自己的镜像
jpress.war
Dockerfile
```
from hub.c.163.com/library/tomcat

MAINTAINER Jochen_M xxx@163.com

COPY jpress.war /usr/local/tomcat/webapps
```
docker build
```
docker build -t jpress:latest .    // 构建镜像
docker run -d -p 8888:8080 jpress    // 运行
```

## 常用命令

```
docker pull [OPTIONS] NAME[:TAG]    // 拉取镜像
docker images [OPTIONS] [REPOSITORY[:TAG]]    //查看本地镜像
docker build []-t NAME:TAG]    // 构建镜像
docker run [-p 8888:8080]/[-P] NAME    // 运行容器
docker stop NAME
docker restart NAME
docker exec -it ID bash    // 进入容器Shell
```

> Docker官网
> []https://www.docker.com/](https://www.docker.com/)
