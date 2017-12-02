---
title: Linux Shell 基础之运算符
date: 2016-10-04 10:53:55
tags: Linux
category: Linux
---


## Shell 运算符

- declare 声明变量
	- `declare [+/-][选项] 变量名`
		- \- 给变量设定类型属性
			- \-a 将变量声明为数组型
			- \-i 将变量声明为整数型
			- \-x 将变量声明为环境变量
			- \-r 将变量声明为只读变量
			- \-p 显示指定变量的被声明的类型
		- \+ 取消变量的类型属性
	- `declare -x test=123` 声明环境变量
- declare -p 查询变量的属性
	- `declare -p` 查询所有变量的属性
	- `declare -p 变量名` 查询指定变量的属性
- expr 或 let 数值运算工具

## Shell 运算符 例子

### Shell declare 例子
```shell
# 给变量 aa 和 bb 赋值
aa=11
bb=22

# 声明变量 cc 的类型是整数型，它的值是 aa 和 bb 的和
declare -i cc=$aa+$bb
```

### 声明数组变量例子
```shell
# 定义数组
movie[0]=zp
movie[1]=tp
declare -a movie[2]=live

# 查看数组
echo ${movie}
echo ${movie[2]}
echo ${movie[*]}
```


## Shell 环境变量配置文件

环境变量配置文件中主要是定义对系统操作环境生效的系统默认环境变量，如 PATH 等

- source 命令
	- `source 配置文件`
	- `. 配置文件`
- 常用的环境变量配置文件
	- /etc/profile
		- USER 变量
		- LOGNAME 变量
		- MAIL 变量
		- PATH 变量
		- HOSTNAME 变量
		- HISTSIZE 变量
		- umask 查看系统默认权限
			- 文件最高权限为 666
			- 目录最高权限为 777
			- 权限不能使用数字进行换算，而必须使用字母
			- umask 定义的权限，是系统默认权限中准备丢弃的权限
		- 调用 /etc/profile.d/*sh 文件
	- /etc/profile.d/*sh
	- ~/.bash_profile
	- ~/.bashrc
	- /etc/bashrc
- 其他配置文件
	- `~/.bash_logout` 注销时生效的环境变量配置文件
- Shell 登录信息
	- `cat /etc/issue` 本地终端欢迎信息
	- `cat /etc/issue.net` 远程终端欢迎信息
		- 转义符在 `/etc/issue.net` 文件中不能使用
		- 是否显示此欢迎信息，由 SSH 的配置文件 `/etc/ssh/sshd_config` 决定，加入 "Banner /etc/issue.net" 行才能显示（需要重启 SSH 服务）
	- `/etc/motd` 登陆后欢迎信息，不管是本地登录，还是远程登录，都可以显示此欢迎信息
