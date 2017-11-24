---
title: ImageMagick
date: 2017-11-24 17:08:53
tags:
    - Ubuntu
    - Nodejs
categories:
    - Ubuntu
---

ubuntu中可以用convert命令对图像的格式和大小等进行转换（convert命令可以用在脚本中，如Nodejs中的gm模块），但是convert不是ubuntu自带的命令，需要先安装ImageMagick，之后才可以使用。

#### 安装命令
```
sudo apt-get install imagemagick
```

#### 测试是否安装成功
首先查看版本，命令：
```
convert -version
```

如果看到下面的信息，则说明安装成功：
```
Version: ImageMagick 6.7.7-10 2014-03-06 Q16 http://www.imagemagick.org
Copyright: Copyright (C) 1999-2012 ImageMagick Studio LLC
Features: OpenMP
```
