---
title: 将本地项目推送到Github
date: 2017-06-01 23:21:44
tags:
    - Github
categories:
    - Github
---

#### 初始化本地项目

```
$ cd project
$ git config --global user.name "Jochen-M"
$ git config --global user.email "Jochen_M@163.com"
$ git init
$ git add .
$ git commit -m "init"
```

#### 提交到Github

首先在Github上新建仓库：
![Alt text](/uploads/git-demo.png)

成功后可以按照红框内的提示连接本地版本库：
![Alt text](/uploads/git-link.png)

```
$ git remote add origin https://github.com/Jochen-M/git-demo.git
$ git push -u origin master
```
