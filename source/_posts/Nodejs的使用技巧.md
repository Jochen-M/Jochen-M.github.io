---
title: Nodejs的使用技巧
date: 2017-05-16 00:50:26
tags:
    - Nodejs
categories:
    - Nodejs
---
> 一、node.js升级到最新版本
> 二、使用nvm管理node版本
> 三、几个npm的常用命令
> 四、使用淘宝 NPM 镜像

<!-- more -->

#### 一、node.js升级到最新版本
node有一个模块叫n，是专门用来管理node.js的版本的。
首先安装n模块：
```
npm install -g n
```
第二步：
升级node.js到最新稳定版
```
n stable
```

#### 二、使用nvm管理node版本
linux 安装 nvm
```
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.31.1/install.sh | bash
```
或者
```
wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.31.1/install.sh | bash
```
然后
```
source ~/.bashrc
```
安装node
```
nvm install stable #安装最新稳定版
```

*额外说明*
```
nvm install stable #安装最新稳定版 node
nvm install 4.2.2 #安装 4.2.2 版本
nvm install 0.12.7 #安装 0.12.7 版本
nvm alias default 0.12.7 #设置默认 node 版本为 0.12.7
```

*nvm切换版本*
如果你的默认 node 版本（通过 nvm alias 命令设置的）与项目所需的版本不同，则可在项目根目录或其任意父级目录中创建 .nvmrc 文件，在文件中指定使用的 node 版本号，例如：
```
cd <项目根目录>  #进入项目根目录
echo 4 > .nvmrc #添加 .nvmrc 文件
nvm use #无需指定版本号，会自动使用 .nvmrc 文件中配置的版本
node -v #查看 node 是否切换为对应版本
```

#### 三、几个npm的常用命令
```
npm -v          #显示版本，检查npm 是否正确安装。
npm install express   #安装express模块
npm install -g express  #全局安装express模块
npm list         #列出已安装模块
npm show express     #显示模块详情
npm update        #升级当前目录下的项目的所有模块
npm update express    #升级当前目录下的项目的指定模块
npm update -g express  #升级全局安装的express模块
npm uninstall express  #删除指定的模块
```

#### 四、使用淘宝 NPM 镜像
国内直接使用 npm 的官方镜像是非常慢的，这里推荐使用淘宝 NPM 镜像。
淘宝 NPM 镜像是一个完整 npmjs.org 镜像，你可以用此代替官方版本(只读)，同步频率目前为 10分钟一次以保证尽量与官方服务同步。
你可以使用淘宝定制的 cnpm (gzip 压缩支持) 命令行工具代替默认的 npm:
```
npm install -g cnpm --registry=https://registry.npm.taobao.org
```
这样就可以使用 cnpm 命令来安装模块了：
```
cnpm install [name]
```
