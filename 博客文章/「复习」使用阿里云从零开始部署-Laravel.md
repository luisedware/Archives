---
title: 「复习」使用阿里云从零开始部署 Laravel
date: 2017-02-23 14:35:50
tags: AliCloud
category: Tooling
---

## 准备

更新 Yum
```
yum update
```

更新 Yum 源
```
rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
rpm -Uvh http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
```

安装 Git
```
yum install -y git
```

安装 Htop
```
yum install -y htop
```

安装 Vim
```
yum install -y vim
```

## Nginx

安装 Nginx 代码仓库
```
yum install -y epel-release
```

安装 Nginx
```
yum install -y nginx
```

启动 Nginx
```
systemctl start nginx
```

开机启动 Nginx
```
systemctl enable nginx
```

Nginx 配置文件

- /usr/share/nginx/html 默认的代码目录
- /etc/nginx/conf.d/default.conf 默认的配置文件

## MariaDB

安装 MariaDB
```
yum install -y mariadb-server mariadb
```

启动 MariaDB
```
systemctl start mariadb
```

初始化 MariaDB
```
mysql_secure_installation
```

开机启动 MariaDB
```
systemctl enable mariadb
```

## PHP

安装 PHP
```
yum install -y php71w php71w-mysql php71w-xml php71w-soap php71w-xmlrpc php71w-mbstring php71w-json php71w-gd php71w-mcrypt php71w-fpm php71w-pdo php71w-pecl-redis
```

启动 PHP-FPM
```
systemctl start php-fpm
```

开机启动
```
systemctl enable php-fpm
```

配置 PHP 关联 Nginx，编辑 Nginx 配置文件如下：
```nginx
server{
    listen       80;
    server_name  cowcat.cc;

    # note that these lines are originally from the "location /" block
    root   /usr/share/nginx/html;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }
    error_page 404 /404.html;
    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
        root /usr/share/nginx/html;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

在 `/usr/share/nginx/html` 新增文件 `index.php`，编辑代码如下：
```php
<?php
phpinfo();
```

输入域名，检查 PHP 是否与 Nginx 关联起来

## Redis

安装 Redis
```
wget http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-9.noarch.rpm
rpm -ivh epel-release-7-9.noarch.rpm
yum install -y redis
```

启动 Redis
```
systemctl start redis
```

开机启动 Redis
```
systemctl enable redis
```
## Beanstalkd

安装 Beanstalkd
```
wget http://cbs.centos.org/kojifiles/packages/beanstalkd/1.9/3.el7/x86_64/beanstalkd-1.9-3.el7.x86_64.rpm
wget http://cbs.centos.org/kojifiles/packages/beanstalkd/1.9/3.el7/x86_64/beanstalkd-debuginfo-1.9-3.el7.x86_64.rpm
rpm -ivh beanstalkd-1.9-3.el7.x86_64.rpm
rpm -ivh beanstalkd-debuginfo-1.9-3.el7.x86_64.rpm
yum install -y beanstalkd
```

启动 Beanstalkd
```
systemctl start beanstalkd
```

开机启动
```
systemctl enable beanstalkd
```

## Composer

安装 Composer
```
curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer
composer -v
```

全局设置国内镜像
```
composer config -g repo.packagist composer https://packagist.phpcomposer.com
```

## Node.js

安装 Node.js
```
yum install -y gcc-c++ make
yum install -y nodejs
```

安装 webpack、gulp、bower
```
npm install bower -g
npm install gulp -g
npm install webpack -g
```

## Gogs

设置子域名
```
hostnamectl set-hostname git.cowcat.cc
```

禁用 SELinux 相关选项
```
sed -i /etc/selinux/config -r -e 's/^SELINUX=.*/SELINUX=disabled/g'
systemctl reboot
```

创建 gogs 数据库
```
CREATE DATABASE IF NOT EXISTS gogs CHARACTER SET utf8 COLLATE utf8_general_ci;
use gogs;
SET default_storage_engine=INNODB;
```

安装 Go
```
yum install -y go
```

安装 Gogs
```
// 下载二进制包
wget http://7d9nal.com2.z0.glb.qiniucdn.com/0.10rc/linux_amd64.zip

// 解压
unzip linux_amd64.zip

cd gogs/
// 后台运行
nohup ./gogs web &
tail -f nohup.out
```

访问 URL 进行安装，安装 URL 是主机 IP 地址加上 3000 端口。

以守护进程的形式运行 Gogs：[https://github.com/gogits/gogs/blob/master/scripts/init/centos/gogs](https://github.com/gogits/gogs/blob/master/scripts/init/centos/gogs)


## Walle

进入 MySQL，新增数据库如下：
```
create database walle default character set utf8 COLLATE utf8_general_ci;
```

安装依赖
```
yum install ansible
```

代码检出
```
mkdir -p /data/www/                                     # 新建目录
git clone https://github.com/meolu/walle-web.git        # 代码检出
```

连接数据库
```
cd walle-web/
vim config/local.php

// 修改 PHP 数组
'db' => [
    'dsn'       => 'mysql:host=127.0.0.1;dbname=walle', # 新建数据库walle
    'username'  => 'username',                          # 连接的用户名
    'password'  => 'password',                          # 连接的密码
],
```

安装 vendor
```
cd /data/www/walle-web
composer install --prefer-dist --no-dev --optimize-autoloader -vvvv
```

初始化项目
```
cd /data/www/walle-web
./yii walle/setup
```

配置 Nginx
```
server {
    listen       80;
    server_name  walle.compony.com; # 改你的host
    root /the/dir/of/walle-web/web; # 根目录为web
    index index.php;

    # 建议放内网
    # allow 192.168.0.0/24;
    # deny all;

    location / {
        try_files $uri $uri/ /index.php$is_args$args;
    }

    location ~ \.php$ {
        try_files $uri = 404;
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
}
```

然后访问你配置好的域名或服务器地址，管理员默认账号密码是 `admin/admin`，开发者默认账号密码是 `demo/demo`。

有些错误 walle 捕捉不到，默认操作日志在 /tmp/walle/ 下，具体可在 config/local.php 里 log.dir 配置路径，tail 着日志，部署看日志。

如果发现 /tmp/walle 下没有日志，很有可能是因为 CentOs 7 yum 安装的 php-fpm 默认 /tmp 目录不可写：/usr/lib/systemd/system/php-fpm.service 中的 PrivateTmp=true 禁止了向tmp目录写日志，解决方法如下：
```
vi /usr/lib/systemd/system/php-fpm.service
PrivateTmp=false

systemctl daemon-reload
systemctl reload php-fpm
```

## Supervisor

安装 Supervisor
```

```

## Alias 常用进程命令

- Nginx
    - alias nginx.start='systemctl start nginx'
    - alias nginx.restart='systemctl restart nginx'
    - alias nginx.stop='systemctl stop nginx'
    - alias nginx.status='systemctl status nginx'
- Mysql
    - alias mysql.start='systemctl start mysql'
    - alias mysql.restart='systemctl restart mysql'
    - alias mysql.stop='systemctl stop mysql'
    - alias mysql.status='systemctl status mysql'
- PHP-FPM
    - alias php-fpm.start='systemctl start php-fpm'
    - alias php-fpm.restart='systemctl restart php-fpm'
    - alias php-fpm.stop='systemctl stop php-fpm'
    - alias php-fpm.status='systemctl status php-fpm'
- Redis
    - alias redis.start='systemctl start redis'
    - alias redis.restart='systemctl restart redis'
    - alias redis.stop='systemctl stop redis'
    - alias redis.status='systemctl status redis'
- Beanstalkd
    - alias beanstalkd.start='systemctl start beanstalkd'
    - alias beanstalkd.restart='systemctl restart beanstalkd'
    - alias beanstalkd.stop='systemctl stop beanstalkd'
    - alias beanstalkd.status='systemctl status beanstalkd'
- Gogs
    - alias gogs.start='/etc/rc.d/init.d/gogs start'
    - alias gogs.stop='/etc/rc.d/init.d/gogs stop'
    - alias gogs.restart='/etc/rc.d/init.d/gogs restart'
    - alias gogs.status='/etc/rc.d/init.d/gogs status'

