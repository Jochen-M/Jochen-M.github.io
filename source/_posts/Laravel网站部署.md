---
title: Laravel网站部署
date: 2017-05-06 23:27:17
tags: 
  - Laravel
  - Ubuntu
  - Nginx
  - MySQL
  - PHP
categories: 
  - PHP
---

## 在Ubuntu中搭建Linux+Nginx+Mysql+PHP环境

### 首先，通过下面的命令来删除Apache
```
sudo service apache2 stop
update-rc.d -f apache2 remove
sudo apt-get remove apache2
```

<!-- more -->

删除完之后，更新一下包列表
```
sudo apt-get update
```

### 1.安装Nginx
```
sudo apt-get install nginx
```
安装完Nginx，执行
```
sudo service nginx start
```
再在浏览器地址栏输入公网IP，就可以看到welcome to Nginx的界面了

### 2.安装Mysql
```
sudo apt-get install mysql-server mysql-client
```
过程中会提示你设置Mysql的密码，就跟平时的密码设置一样，一
次输入，一次确认。密码确认完毕后基本等一会就安装好了。尝试
```
mysql -u root -p
```
如果登录成功，那Mysql就正确安装了。

### 3.安装PHP
```
sudo apt-get install php5-fpm php5-cli php5-mcrypt
```
只有通过php5-fpm，PHP在Nginx下才能正常运行，遂，安装之。
至于php5-mcrypt，有些PHP框架会依赖于这个，比如Laravel就是，所以也把它装上了。

### 4.配置PHP
```
sudo vim /etc/php5/fpm/php.ini
```
打开PHP配置文件，找到cgi.fix_pathinfo选项，去掉它前面的注释分号;，然后将它的值设置为0,如下:
cgi.fix_pathinfo=0

### 5.启用php5-mcrypt:
```
sudo php5enmod mcrypt
```

### 6.重启php5-fpm:
```
sudo service php5-fpm restart
```
在搭建完LEMP环境之后，首先要明确两个重要目录:
> Nginx的默认root文件夹
> /usr/share/nginx/html
> 
> Nginx的服务器配置文件所在目录
> /etc/nginx/sites-available/

## 部署Laravel
### 1.创建网站的根目录
```
sudo mkdir -p /var/www
```

### 2.配置nginx服务器
```
sudo vim /etc/nginx/sites-available/default
```
打开nginx的配置文件之后，找到server这一块，大概是长这个样子的
```
server {
        listen 80 default_server;
        listen [::]:80 default_server ipv6only=on;

        root /usr/share/nginx/html;
        index index.html index.htm;

        server_name localhost;

        location / {
                try_files $uri $uri/ =404;
        }
}
```
其中root，index ，server_name和location这几行需要稍微修改一下:

root修改

root /var/www/laravel/public;
这里就是将nginx服务器的根目录指向Laravel的public文件夹下，后续的Laravel项目的代码我们会放在我们之前创建的/var/www/laravel目录下

index修改

index index.php index.html index.htm;
这里需要注意的是，将index.php排在最前面

server_name修改

server_name server_domain_or_IP;
将server_domain_or_IP修改为你的公网IP

location修改

location / {
        try_files $uri $uri/ /index.php?$query_string;
}
修改完是这样的：
```
server {
    listen 80 default_server;
    listen [::]:80 default_server ipv6only=on;

    root /var/www/laravel/public;
    index index.php index.html index.htm;

    server_name server_domain_or_IP;

    location / {
            try_files $uri $uri/ /index.php?$query_string;
    }
}
```
最后我们还需要配置一下Nginx，让其执行PHP文件。同样是在这个文件里，在location下方添加下面的配置：
```
server {
    listen 80 default_server;
    listen [::]:80 default_server ipv6only=on;

    root /var/www/laravel/public;
    index index.php index.html index.htm;

    server_name server_domain_or_IP;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        try_files $uri /index.php =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php5-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```
注意，这一块是自己加上去的：
```
 location ~ \.php$ {
        try_files $uri /index.php =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php5-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
```
配置完之后重启Nginx，使上面的配置项生效。
```
sudo service nginx restart
```
### 3.创建Laravel项目
在配置完nginx后，怎么获取Laravel的项目代码呢？有以下几种方法：

(1).直接composer安装

直接通过composer来安装
```
cd ~
curl -sS https://getcomposer.org/installer | php
```
上面命令会安装composer

composer全局使用：
```
sudo mv composer.phar /usr/local/bin/composer
```
然后在/var/www目录下直接执行
```
sudo composer create-project laravel/laravel laravel
```
因为我们之前创建/var/www目录，你可以直接cd /var/www然后执行上面的命令。然后坐等安装完成。

(2).直接上传代码

使用下面命令上传
```
scp -r laravel root@your_IP:
```
然后将laravel移动到/var/www目录下

(3).使用Git和Coding平台

一旦本地代码都推送到Coding，然后在/var/www目录下直接使用
```
git clone your-project-git-link
```
your-project-git-link替换为你Coding上的laravel项目地址

### 4.最后的最后
不管哪种方式安装的代码，/var/www/都是属于root用户的，而访问网站的用户则需要正确的权限和访问限制，我们可以通过下面的命令来实现。
```
sudo chown -R :www-data /var/www/laravel
```
根据Laravel的官方文档，/var/www/laravel/storage 目录需要给网站的用户写权限
```
sudo chmod -R 775 /var/www/laravel/storage
```

### 5.在浏览器输入：

http://server_domain_or_IP

*本文仅做学习使用*
*来源：* https://laravist.com/article/19
