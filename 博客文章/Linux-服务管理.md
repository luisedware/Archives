---
title: Linux 服务管理
date: 2016-09-23 20:26:20
tags: Linux
category: Linux
---

## Linux 服务管理

### 简介与分类

- #### 系统的运行级别
	- 查看运行级别命令：`runlevel`
	- 修改运行级别命令：`init 运行级别`
	- 系统开机进入级别：`vim /etc/inittab` ~ `id:3initdefault:`

运行级别|含义
---|---
0 | 关机
1 | 单用户模式，主要用于系统修复
2 | 不完全的命令行模式，不含 NFS 服务
3 | 完全的命令行模式，就是标准字符界面
4 | 系统保留
5 | 图形模式
6 | 重启动

- #### 服务的分类
	- Linux 服务
		- PRM 包默认安装的服务
			- 独立的服务
				- 默认配置文件位置
					- /etc/init.d/ 启动脚本位置
					- /etc/sysconfig/ 初始化环境配置文件位置
					- /etc/ 配置文件位置
					- /etc/xinetd.conf xinetd 配置文件
					- /etc/xinetd.d/ 基于 xineted 服务的启动脚本
					- /var/lib/ 服务产生的数据放在这里
					- /var/log/ 日志
			- 基于 xinetd 服务
		- 源码包安装的服务
			- 源码包安装位置，一般是 `/usr/local` 下
		- PRM 包安装和源码包安装的区别：两者服务安装的位置不同
- #### 服务与端口
	- 概念：如果把 IP 地址比作一间房子，端口就是出入这间房子的门。真正的房子只有几个门，但是一个 IP 地址的端口可以有 65536 个
	- 服务与端口的对应：`cat /etc/services`
	- 查询系统中开启的服务：`netstat [选项]`
		- \-t 列出 TCP 数据
		- \-u 列出 UDP 数据
		- \-l 列出正在监听的网络服务（不包含已经连接的网络服务）
		- \-n 用端口号来显示服务，而不是用服务名
		- \-p 列出该服务的进程 ID
	- chkconfig 和 netstat 的区别
		- `chkconfig` 查看自启动命令
		- `netstat` 查看启动命令
- #### 服务启动与自启动
	- 独立服务的自启动
		- `ntsysv` 命令图形界面管理自启动
		- `/etc/rc.d/rc.local` 文件修改管理自启动
		- `chkconfig --list` 查看服务自启动状态，可以看到所有 RPM 包安装的服务
		- `chkconfig [--level 运行级别] [独立服务名] [on|off]` 修改服务自启动状态
	- 独立服务的启动
		- `/etc/init.d/独立服务名 start、stop、status、restart`
		- `service 独立服务名 start、stop、status、restart`
	- 源码包安装服务的启动
		- 使用绝对路径，调用启动脚本来启动。不同的源码包的启动脚本不同。可以查看源码包的安装说明，查看启动脚本的方法。
		- 用法：`/usr/local/apache2/bin/apachectl start`
	- 源码包服务的自启动
		- `vi /etc/rc.d/rc.local` 加入 `/usr/local/apache2/bin/apachectl start`
	- 源码包的 Apache 服务能被 Service 命令管理启动
		- `ln -s /usr/local/apache2/bin/apachectl /etc/init.d/apache`
	- 源码包的 Apache 服务能被 chkconfig 与 ntsysv 命令管理自启动
		- `vi /etc/init.d/apache` 加入 `# chkconfig:35 86 76` 和 `description:source package apache`
		- `chkconfig --add apache` 把源码包 apache 加入 chkconfig 命令

