---
title: 利用 apache2 在多个服务器上部署多个项目实践
date: 2017-11-25 12:31:06
tags:
    - Apache
    - Ubuntu
    - Safe
categories:
    - Ubuntu
---

近日在开发某电商平台时，应项目需求，要在两个远程服务器上利用 apache2 部署多个项目（前后端分离）：

+ Server (Nodejs/Koa2)
+ site1.com (Angular2)
+ site2.com、site3.com(site2.com:8080) (Angular2)
+ Android APP、Apple APP (ionic3)

现将部署过程及遇到的问题记录下来。

### 服务器配置说明
项目一共用到了两台服务器，具体配置信息如下：

**VPS 1 (域名: site1.com)**
+ System: Ubuntu 16.04
+ Mem: 8G
+ Disk: 200G
+ Bindwidth: 10Mbps

**VPS 2 (域名: site2.com)**
+ System: Ubuntu 16.04
+ Mem: 2G
+ Disk: 50G
+ Bindwidth: 2.5Mbps

### 架构图

为了清晰明了，特绘制架构图如下：
![Alt text](/uploads/apache2_multi_sites.png)

根据上图可知，这里将Server、site1.com等部署在了性能更优的服务器(VPS 1)，而将site2.com、site3.com部署在性能较低的服务器(VPS 2)。之所以这样分配，原因有三：
1. Server需要接收并处理所有前端（web + app）的请求，还要进行数据的存储（MongoDB）和搜索（ElasticSearch），对内存、带宽、硬盘容量等的要求都比较高；
2. 做web版微信支付等开发时，要求配置独立的域名，不能带端口（微信支付具体要求很复杂，主要是为了安全性）；
3. 由于配套网站较多（这里是 3 个），需要多个域名或多个端口。采用一个服务器开放多个端口的方式，节省成本，但降低了网站的识别度，也不满足上述第 2 条要求；而每个网站都配备一个域名和服务器，意味着更高的成本，对于小型项目来说不是经济的做法，而且不利于管理。

因此，本项目采取了架构图所示的折中的方式。

### apache2 的安装与配置

#### 安装
ubuntu 上自带了 apache2 软件包，只需要执行命令安装：
```
sudo apt-get install apache2
```

#### VPS 1 中 Apache2 的配置

在 /etc/apache2/sites-available 中新增配置文件 site1.conf，忽略掉所有注释后，配置如下，重点注意这里的重定向配置:

```
<VirtualHost *:80>
  ServerAdmin webmaster@localhost
  DocumentRoot /var/www/site1_com  # 项目目录

  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined
  <Directory "/var/www/site1_com">  # 项目目录
    AllowOverride All
  </Directory>

  # 重定向
  # 将所有对该apache服务器上 site1.com(:80) 的 http 请求，都重定向到 https://site1.com/REQUEST_URI
  # RewriteEngine on
  # RewriteCond %{SERVER_NAME} = site1.com
  # RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]
  RewriteEngine On
  RewriteCond %{HTTPS} off
  RewriteRule (.*) https://%{HTTP_HOST}%{REQUEST_URI}
</VirtualHost>
```

该项目可以直接通过公网调用 server 的接口：site1.com:3000，及其不安全。

为了安全性，需要对网站进行 ssl 加密。这里先保留以上不安全的配置文件和项目目录，另外再新增一个 ssl 加密配置文件和项目目录 site_com_ssl。

加密的具体做法可以参考[《Apache2 SSL加密》](https://jochen-m.github.io/2017/11/24/Apache2-SSL%E5%8A%A0%E5%AF%86/)。

加密完成后，/etc/apache2/sites-available/ 目录下会多出一个配置文件 site1-le-ssl.conf，内容大体如下：
```
<IfModule mod_ssl.c>
<VirtualHost *:443>
  ServerAdmin webmaster@localhost
  DocumentRoot /var/www/site1_com_ssl

  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined
  <Directory "/var/www/site1_com_ssl">
    AllowOverride All
  </Directory>

  ServerName site1.com

  # 以下为 Certbot 自动配置的信息，引入了 ssl 证书和密钥等。
  Include /etc/letsencrypt/options-ssl-apache.conf
  SSLCertificateFile /etc/letsencrypt/live/site1.com/fullchain.pem
  SSLCertificateKeyFile /etc/letsencrypt/live/site1.com/privkey.pem
</VirtualHost>

<VirtualHost *:3001>
  # 重点 - 反向代理
  # 将外部对 https://site1.com:3001 的访问请求，反向代理到服务器的 http://localhost:3000 （即 Node 监听的端口）。
  # 通过反向代理，外部只能访问 https://site1.com:3001 的安全接口，而将 http://localhost:3000 “隐藏”起来，可以防止服务器遭受外部恶意攻击。
  # 相应地，site1_com_ssl 访问服务器时的请求地址也要修改为 https://site1.com:3001。
  ProxyPreserveHost On
  ProxyPass / http://localhost:3000/
  ProxyPassReverse / http://localhost:3000/

  ServerName site1.com

  Include /etc/letsencrypt/options-ssl-apache.conf
  SSLCertificateFile /etc/letsencrypt/live/site1.com/fullchain.pem
  SSLCertificateKeyFile /etc/letsencrypt/live/site1.com/privkey.pem
</VirtualHost>
</IfModule>
```

**因此在正式生产环境下，应将 :80端口 重定向到 :443 端口，并关闭防火墙的 3000 端口，打开 3001 端口。**

#### VPS 2 中 Apache2 的配置

该服务器的配置和部署相对简单，和 VPS 1 的配置大同小异。

需要注意的是，apache2 还要监听 :8080 的 ssl 端口，因此 /etc/apache2/ports.conf 的配置如下：

```
Listen 80
<IfModule ssl_module>
	Listen 443
	Listen 8080
</IfModule>

<IfModule mod_gnutls.c>
	Listen 443
	Listen 8080
</IfModule>
```

**不要忘记打开防火墙的该端口**
```
sudo ufw allow 8080
```

另外，site2.com 、site3.com 以及 Android/Apple 请求数据的地址应为 https://site1.com:3001 。

至此，在两个服务器上部署多个项目基本完成，更多细节不再赘述。
