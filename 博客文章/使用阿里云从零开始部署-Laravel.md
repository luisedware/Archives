---
title: 使用阿里云从零开始部署 Laravel
date: 2016-08-31 20:03:03
tags: AliCloud
category: Tooling
---

## 阿里云 ECS 选购

- 地域
	- 地域：华南 1
	- 可用区：华南 1 可用区 A
- 网络
	- 网络类型：经典网络
- 实例
	- 实例系列：系列 I
	- 实力规格：1 核 1GB
- 带宽：
	- 公网带宽：按固定带宽
	- 带宽：1 Mbps
- 镜像：
	- 镜像类型：公共镜像
	- 公共镜像：CentOS 6.5 32位
- 存储
	- 系统盘：40 GB
- 收费模式
	- 包年包月
	- 每月 68 元

## 阿里云域名选购解析

- 国际域名.cc
	- 名称：cowcat.cc
	- 价格：16 元一年
- 实名认证：
	- 输入身份证号码
	- 提交身份证正面照片
- 域名解析：
	- 输入阿里云 ECS 公网 IP 地址



## CentOS 6.5 搭建 LNMP

### 搭建准备

查看 `CentOS` 版本
```
cat /etc/centos-release
```

安装 `epel`
```
yum install -y epel-release
```

安装 `vim`

```
yum install -y vim
```

安装 `htop`
```
yum install -y htop
```

安装 `git`
```
yum install -y git
```

### 安装 Nginx

修改 yum 源：
```shell
# 进入 `/etc/yum.repos.d` 目录
cd /etc/yum.repos.d/

# 创建一个 `nginx.repo` 文件
vim nginx.repo

# 写入内容如下
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=0
enabled=1
```

更新 yum 源
```
yum udpate
```

安装 Nginx
```
yum install nginx
```

查看 Nginx 版本
```
# nginx version: nginx/1.10.1
nginx -v
```
启动 Nginx
```
service nginx start
```

开机启动 Nginx
```
chkconfig nginx on
```

### 安装 MySQL

官网下载源码包
```
wget http://dev.mysql.com/get/mysql57-community-release-el6-7.noarch.rpm
```

安装 MySQL 的 yum 源
```
rpm -Uvh mysql57-community-release-el6-7.noarch.rpm
```

打开 mysql-community.repo 看关于mysql的内容，确定 mysql57 的 enable 是打开的
```
vim /etc/yum.repos.d/mysql-community.repo
```

安装 MySQL
```
yum install mysql-community-server
```

启动 MySQL
```
service mysqld start
```

启动 MySQL 后，查看自动生成的密码
```
grep "password" /var/log/mysqld.log
```

修改 MySQL初始化密码
```
mysql_secure_installation
```

MySQL 登录验证
```
mysql -uroot -p
```

MySQL 的配制文件默认在 `/etc/my.cnf`

开机启动 MySQL
```
chkconfig mysqld on
```

### 安装 PHP

删除之前的 PHP
```
yum remove php* php-common
```

更新 yum 源
```
wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-6.noarch.rpm
wget http://rpms.remirepo.net/enterprise/remi-release-6.rpm
rpm -Uvh remi-release-6.rpm epel-release-latest-6.noarch.rpm
```

安装 PHP 7
```
yum install php70w php70w-fpm php70w-cli php70w-mysql php70w-pdo php70w-mbstring php70w-mcrypt php70w-pear php70w-opcache php70w-bcmath php70w-xml php70w-pecl-redis
```

查看 PHP 7 的版本与扩展
```
php -v
php -m
```

简单修改一些配置
```
# 打开 PHP 配置文件
vim /etc/php.ini

# 修改 PHP 配置参数
date.timezone = Asia/Shanghai
upload_max_filesize = 20M
post_max_size = 20M
display_errors = Off
expose_php = Off
```

重启 PHP 7
```
service php-fpm restart
```

开机启动 PHP 7
```
chkconfig php-fpm on
```

### 安装 Composer


安装 Composer
```
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"

php -r "if (hash_file('SHA384', 'composer-setup.php') === 'e115a8dc7871f15d853148a7fbac7da27d6c0030b848d9b3dc09e2a0388afed865e6a3d6b3c0fad45c48e2b5fc1196ae') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"

php composer-setup.php --install-dir=/usr/bin --filename=composer

php -r "unlink('composer-setup.php');"
```

查看 Composer 版本
```
composer -v
```

全局设置国内镜像
```
composer config -g repo.packagist composer https://packagist.phpcomposer.com
```

### 安装 Node.js (4.5 LTS)

编译安装 Node.js
```
wget https://nodejs.org/dist/v4.5.0/node-v4.5.0.tar.gz
tar zxf node-v4.5.0.tar.gz
cd node-v4.5.0
./configure
make
make install
```

如果提示 gcc 版本过低，需要进行升级，执行命令如下
```
sudo rpm --import http://ftp.scientificlinux.org/linux/scientific/5x/x86_64/RPM-GPG-KEYs/RPM-GPG-KEY-cern
wget -O /etc/yum.repos.d/slc6-devtoolset.repo http://linuxsoft.cern.ch/cern/devtoolset/slc6-devtoolset.repo
sudo yum install devtoolset-2
scl enable devtoolset-2 bash
```

然后查看版本
```
gcc --version
g++ --version
gfortran --version
```

### 安装 Redis
```
rpm -Uvh http://download.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-6.rpm
yum --enablerepo=remi,remi-test install redis
```

开启 Redis
```
chkconfig --add redis
chkconfig --level 345 redis on
service redis start
```

### 配置 Nginx 与 PHP 7

Nginx 安装完毕之后，默认的网站根目录是在 `/usr/share/nginx/html/`

虚拟主机的配置在 `/etc/nginx/conf.d`，如果要配置新的域名在这里就可以了。

默认有一个 `default.conf` 的配置，可以在里面配置以下参数
```
server{
    listen 80;
    server_name www.cowcat.cc;

    root /www/CowCat/public;
    index index.php index.html index.htm;

    # 日志记录
    access_log  /var/log/nginx/cowcat.cc.access.log  main;
    error_log /var/log/nginx/cowcat.cc.error.log;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    # 错误页面
    error_page 404              /404.html;
    error_page 500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    #PHP 脚本请求全部转发到 FastCGI处理. 使用FastCGI默认配置.
    location ~ \.php$ {
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
}
```

##
