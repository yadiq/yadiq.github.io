---
title: Ubuntu17.04搭建LNMP环境
date: 2018-12-29 21:26:21
tags:
categories: Linux
---


![LNMP](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/12/29/167f95cd175d128a~tplv-t2oaga2asx-image.image)

## 1.登录和更新
    
```
#以管理身份登录
    su
    #k系统更新
    apt-get update
    apt-get upgrade
```

## 2.安装nginx

```
sudo apt install nginx
    systemctl enable nginx
```

## 3.安装MySQL

```
sudo apt install mysql-server
    systemctl enable mysql
```

## 4.安装PHP
    
```
sudo apt install php
    sudo apt install php-fpm
```

## 5.配置Nginx支持PHP

```
server {
            listen 80 default_server;
            listen [::]:80 default_server;
    
            root /home/html;
    
            index index.php index.html index.htm index.nginx-debian.html;
    
            server_name localhost;
    
            location / {
                    try_files $uri $uri/ =404;
            }
    
            location ~ \.php$ {
                    include snippets/fastcgi-php.conf;
            #       # With php-fpm (or other unix sockets):
                    fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
            #       # With php-cgi (or other tcp sockets):
            #       fastcgi_pass 127.0.0.1:9000;
            }
    
            location ~ /\.ht {
                    deny all;
            }
    }
```

## 6.测试篇
注意新版本Mysql 中操作命令 "mysql"改为"mysqli"

```
#新建网站首页
cd /home/html;
vim index.php;
```

```
<?php
    echo "测试数据库连接";
    $con = mysqli_connect("localhost","root","NewPass1!");
    if (!$con) {
        die('Could not connect: ' . mysql_error());
    }else{
            echo "conneted";
    }
    #打印php信息
    phpinfo();
    ?>
```