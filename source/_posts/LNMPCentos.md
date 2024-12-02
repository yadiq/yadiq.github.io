---
title: Centos搭建LNMP环境
date: 2018-12-29 21:26:21
tags:
categories: Linux
---


![LNMP](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/12/29/167f952ff0becb26~tplv-t2oaga2asx-image.image)

## 1.防火墙

开启80端口、3306端口

## 2.安装nginx

```
    yum install nginx      #安装nginx，根据提示，输入Y安装即可成功安装	   
    systemctl start nginx  #启动
    systemctl enable nginx #设为开启启动
    rm -rf /usr/share/nginx/html/*  #删除ngin默认测试页、
```

## 3.安装MySQL

查找存储库下载链接 （yum版本，对应红帽家族）
https://dev.mysql.com/downloads/repo/yum/

```
		#添加存储库
    wget https://repo.mysql.com//mysql57-community-release-el7-11.noarch.rpm
    #安装下载的发行包
    yum install mysql57-community-release-el7-11.noarch.rpm

    ###启用和禁用 版本（可忽略）###
    yum repolist all | grep mysql
    yum-config-manager --disable mysql57-community
    yum-config-manager --enable mysql56-community
    #查看 启用版本
    yum repolist enabled | grep mysql
    #############################

    #安装mysql
    yum install mysql-community-server
    #启动
    systemctl start mysqld.service
    #检查状态
    systemctl status mysqld.service

    #查看初始密码
    grep "password" /var/log/mysqld.log
    mysql -uroot -p
    #修改密码（必须包含数字、字母大小写、特殊符号，且大于8位）
    ALTER USER 'root'@'localhost' IDENTIFIED BY 'NewPass1!';
    #删除MySQL的Repository
    yum -y remove mysql57-community-release-el7-11.noarch.rpm
    #开机启动
    systemctl enable mysqld.service
```

## 4.安装PHP

```
#安装php
    yum install php
    #安装PHP组件，使PHP支持 MySQL、PHP支持FastCGI模式
    yum install php-mysql php-gd libjpeg* php-imap php-ldap php-odbc php-pear php-xml php-xmlrpc php-mbstring php-mcrypt php-bcmath php-mhash libmcrypt libmcrypt-devel php-fpm
    #启动php-fpm
    systemctl start php-fpm
    #设置开机启动
    systemctl enable php-fpm
```

## 5.配置Nginx支持PHP

Nginx配置文件 

```
vim /etc/nginx/nginx.conf 
```

```
 http {
        log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                          '$status $body_bytes_sent "$http_referer" '
                          '"$http_user_agent" "$http_x_forwarded_for"';
    
        access_log  /var/log/nginx/access.log  main;
    
        sendfile            on;
        tcp_nopush          on;
        tcp_nodelay         on;
        keepalive_timeout   65;
        types_hash_max_size 2048;
    
        include             /etc/nginx/mime.types;
        default_type        application/octet-stream;
    
        # Load modular configuration files from the /etc/nginx/conf.d directory.
        # See http://nginx.org/en/docs/ngx_core_module.html#include
        # for more information.
        include /etc/nginx/conf.d/*.conf;
    
        server {
            listen       80 default_server;
            listen       [::]:80 default_server;
            server_name  _;
            root         /home/html/;
            index        index.php index.html index.htm;
    
            # Load configuration files for the default server block.
            include /etc/nginx/default.d/*.conf;
    
            location / {
            }
    
            #支持PHP的配置
            location ~ .*\.php(\/.*)*$ {
                fastcgi_pass   127.0.0.1:9000;
                fastcgi_index  index.php;
                fastcgi_param  SCRIPT_FILENAME   $document_root$fastcgi_script_name;
                include        fastcgi_params;
            }
    
            error_page 404 /404.html;
                location = /40x.html {
            }
    
            error_page 500 502 503 504 /50x.html;
                location = /50x.html {
            }
        }
    }
```


## 6.测试篇

```
#新建网站首页
cd /home/html;
vim index.php;
```

```
<?php
    #测试数据库连接
    $con = mysql_connect("localhost","root","NewPass1!");
    if (!$con)
      {
      die('Could not connect: ' . mysql_error());
      }else{
            echo "Mysql connected！";
    }
    #打印php信息
    phpinfo();
    ?>
```