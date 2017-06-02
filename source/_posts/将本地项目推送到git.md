---
title: 将本地项目推送到git
date: 2017-06-01 23:21:44
tags:
    - Github
categories:
    - Github
---

```
echo "# Project Name" >> README.md  // 新建一个README.md的文件，并将项目名写入此文件
git init // 新建一个本地仓库
git add README.md // 将README.md文件加入到仓库中
git commit -m "your commit" // 将文件commit到本地仓库
git remote add origin https://github.com/Jochen-M/MovieGoGo.git // 添加远程仓库，origin只是一个远程仓库的别名，可以随意取
git push -u origin master // 将本地仓库push远程仓库，并将origin设为默认远程仓库
```