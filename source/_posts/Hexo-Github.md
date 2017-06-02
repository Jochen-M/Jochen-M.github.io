---
title: Hexo + Github 搭建个人博客
date: 2017-01-25 13:46:13
tags: 
  - Github
  - Hexo
  - NexT
categories: 
  - Hexo
---

### 用Hexo + Github 搭建个人博客的具体步骤如下：

#### 注册Github账号

##### 1.点击[Github](http://www.github.com)注册账号，已有账号的同学可以跳过。

##### 2.创建仓库

<!-- more -->

登录账号后，在Github页面的右上方选择New repository，创建一个仓库，仓库名字必须为``yourname.github.io``,例如：

![Alt text](/uploads/create-new-repository.png)

(由于本人已经创建了该仓库，所以出现错误提示！)

##### 3.生成添加密钥：

在终端输入：
```
ssh-keygen -t rsa -C "Github注册邮箱地址"
```
待秘钥生成完毕，会得到两个文件id_rsa和id_rsa.pub，打开id_rsa.pub,复制里面的所有内容，然后进入[https://github.com/settings/ssh](https://github.com/settings/ssh):

![Alt text](/uploads/settings-ssh.png)

将复制的内容粘贴到Key的输入框，随意起一个Title，点击``Add SSH key``按钮即可。

#### 安装配置Hexo

#### ps：有能力的同学可以自行查看[官方文档](https://hexo.io/zh-cn/)进行安装配置

##### 1.在终端切换到放置本地Hexo的目录

*强烈建议*：*不要*选择需要管理员权限才能创建文件(夹)的目录。

##### 2.下载安装Hexo

```
$ npm install -g hexo-cli
```

##### 3.初始化Hexo

```
// 建立一个Hexo文件夹，<folder>为文件夹名字，名字任意
$ hexo init <folder>
// 进入Hexo文件夹
$ cd <folder>
// node.js的命令，根据博客既定的dependencies配置安装所有的依赖包
$ npm install
```

##### 4.配置博客

* 修改网站相关信息

```
title: Jochen-M
subtitle: 
    description: 每一个不曾起舞的日子都是对生命的辜负！
    author: Jochen-M
    language: zh-Hans
    timezone: Asia/Shanghai
```

*注意*：每一项的填写，其:后面都要保留一个空格，下同。

* 配置统一资源定位符（个人域名）

```
url: http://jochen-m.github.io
```

* 配置部署

```
deploy:
    type: git
    repo: http://github.com/Jochen-M/Jochen-M.github.io.git
    branch: master
```

其中repo项是Github上创建好的仓库的地址。

#### 至此，Hexo已经安装配置完成，我们就可以试着发表自己的文章了：

##### 新建一篇文章：

```
// 新建一篇文章
$ hexo new "文章标题"
```

新建的markdown文件会保存在Hexo目录下的source/_post文件夹下，我们可以用markdown编辑器打开它进行编辑。

![Alt text](/uploads/sublime-markdown.png)

（本人用的markdown编辑器是Sublime Text 3，只需安装markdown相关插件。）

##### 保存后，在本地发布：

```
// 生成静态文件
$ hexo generate
// 在本地发布
$ hexo server
```

成功后，可以在终端看到如下提示：

![Alt text](/uploads/hexo-server.png)

##### 查看效果，在浏览器输入：

```
http://localhost:4000
```

即可在浏览器看到我们的博客和文章。

##### 但此时，博客仅仅部署在本地，我们需要将博客部署到网上。在终端输入：

```
$ hexo deploy
```

这时，我们的博客就已经部署到网上了。在浏览器输入我们的域名，如：[jochen-m.github.io](jochen-m.github.io)。

> #### 推荐：
> Hexo官方文档：[https://hexo.io/zh-cn/](https://hexo.io/zh-cn/)
> NexT主题文档（本博客所用主题）：[http://theme-next.iissnan.com/](http://theme-next.iissnan.com/)
> Markdown语法说明（中文版）：[http://www.appinn.com/markdown/](http://www.appinn.com/markdown/)
>
>
> #### 本文参考：
> 简书作者inerdstack的博文：[《20分钟教你使用hexo搭建github博客》](http://www.jianshu.com/p/e99ed60390a8)
