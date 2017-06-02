---
title: Nodejs + Express + MongoDB快速搭建网站
date: 2017-05-26 22:17:56
tags:
    - Nodejs
    - Express
    - MongoDB
categories:
    - Nodejs
---

#### 基本架构
+ 后台：
  + Node.js
  + Express框架(通过npm安装)
  + Moment.js——处理时间格式(通过npm安装)
  + mongoose——mongoDB数据库插件(通过npm安装)
+ 数据库：
  + mongoDB
+ 前端：
  + jade模板引擎(通过npm安装)
  + Bower(通过npm安装)
  + jQuery(通过Bower安装)
  + Bootstrap(通过Bower安装)

<!-- more -->

#### 注意事项及技巧
+ 使用bower安装依赖的库和文件时，应事先在项目根目录建立.bowerrc文件，并在文件中添加配置信息，表示库和文件的安装目录：
  ```
  {
    "directory": "public/libs"
  }
  ```
+ 使用<code>bower init</code>和<code>cnpm init</code>生成配置文件。

#### 项目具体源代码：
Github: [https://github.com/Jochen-M/MovieGoGo.git](https://github.com/Jochen-M/MovieGoGo.git)

> 项目来源：慕课网Scott老师的教学视频《node+mongodb 建站攻略》
> 本文仅作为学习使用！