---
title: Apache2 SSL加密
date: 2017-11-24 17:17:27
tags:
    - Ubuntu
    - Apache
    - Safe
categories:
    - Ubuntu
---

### 关于HTTPS和HTTP

#### 超文本传输协议HTTP
超文本传输协议HTTP协议被用于在Web浏览器和网站服务器之间传递信息。HTTP协议以明文方式发送内容，不提供任何方式的数据加密，如果攻击者截取了Web浏览器和网站服务器之间的传输报文，就可以直接读懂其中的信息，因此HTTP协议不适合传输一些敏感信息，比如信用卡号、密码等。

#### 安全套接字层超文本传输协议HTTPS
为了解决HTTP协议的这一缺陷，需要使用另一种协议：安全套接字层超文本传输协议HTTPS。为了数据传输的安全，HTTPS在HTTP的基础上加入了SSL协议，SSL依靠证书来验证服务器的身份，并为浏览器和服务器之间的通信加密。

#### HTTPS和HTTP的区别主要为以下四点：
+ https协议需要到ca申请证书，一般免费证书很少，需要交费。
+ http是超文本传输协议，信息是明文传输，https则是具有安全性的ssl加密传输协议。
+ http和https使用的是完全不同的连接方式，用的端口也不一样，前者是80，后者是443。（默认情况下）
+ http的连接很简单，是无状态的；HTTPS协议是由SSL+HTTP协议构建的可进行加密传输、身份认证的网络协议，比http协议安全。

#### TLS(传输层安全)及其前身SSL(安全套接字层)，是为了在受保护的、加密的信道中通信而创建的安全协议。

### 如何对Apache中的项目进行加密处理呢?

一种方式是用openssl手动生成证书和密钥，这种方式的缺点是：浏览器不认这类证书，通过浏览器访问时会有安全性提示，影响体验，所以这里不再赘述。具体方法可以参考这篇文章[《How To Create a SSL Certificate on Apache for Ubuntu 14.04》](https://www.digitalocean.com/community/tutorials/how-to-create-a-ssl-certificate-on-apache-for-ubuntu-14-04)。

这里主要记录另一种方法：通过ubuntu的Certbot工具来生成证书和秘钥，对apache进行加密。

具体步骤及说明如下：

#### 安装Certbot
在Ubuntu系统中，Certbot团队维护了一个PPA软件源。将其添加到仓库列表中，并安装Certbot。
```
$ sudo apt-get update
$ sudo apt-get install software-properties-common
$ sudo add-apt-repository ppa:certbot/certbot
$ sudo apt-get update
$ sudo apt-get install python-certbot-apache
```

#### 开始加密
Certbot有一个很稳定的测试版Apache插件，它会自动获取并安装certs。
```
$ sudo certbot --apache
```
运行该命令，你将会得到一个证书，并且Certbot会自动修改apache的配置文件使加密生效。

#### 自动更新
Certbot包附带一个cron定时任务，该任务会在证书到期之前自动更新证书。因为加密证书有效期为90天，所以最好三个月定时更新。你可以使用如下命令来测试证书的自动更新：
```
$ sudo certbot renew --dry-run
```

你可以通过cron或者systemd任务来执行：
```
certbot renew
```
