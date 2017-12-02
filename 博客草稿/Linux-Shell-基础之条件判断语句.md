---
title: Linux Shell 基础之条件判断语句
date: 2016-10-04 10:56:05
tags: Linux
category: Linux
---


## Shell 条件判断式语句

### 按文件类型判断

测试选项|作用
---|---
\-b 文件 | 判断该文件是否存在，并且是否为块设备文件
\-c 文件 | 判断该文件是否存在，并且是否为字符设备文件
\-d 文件 | 判断该文件是否存在，并且是否为目录文件
\-e 文件 | 判断该文件是否存在（存在为真）
\-f 文件 | 判断该文件是否存在，并且是否为普通文件
\-L 文件 | 判断该文件是否存在，并且是否为符号链接文件
\-p 文件 | 判断该文件是否存在，并且是否为管道文件
\-s 文件 | 判断该文件是否存在，并且是否为非空
\-S 文件 | 判断该文件是否存在，并且是否为套接字文件

两种判断格式

- `test -e /root/install.log`
- `[ -e /root/install.log ]`
- `[ -e /root/install.log ] && echo yes || echo no`

### 按文件权限判断

测试选项|作用
---|---
\-r 文件 | 判断该文件是否存在，并且是否该文件拥有读权限
\-w 文件 | 判断该文件是否存在，并且是否该文件拥有写权限
\-x 文件 | 判断该文件是否存在，并且是否该文件拥有执行权限
\-u 文件 | 判断该文件是否存在，并且是否该文件拥有 SUID 权限
\-g 文件 | 判断该文件是否存在，并且是否该文件拥有 SGID 权限
\-k 文件 | 判断该文件是否存在，并且是否该文件拥有 SBit 权限

### 两个文件之间的比较

测试选项|作用
---|---
文件1 -nt 文件2 | 判断文件 1 的修改时间是否比文件 2 的新
文件1 -ot 文件2 | 判断文件 1 的修改时间是否比文件 2 的旧
文件1 -ef 文件2 | 判断文件 1 是否和文件 2 的 Inode 号一致，可以理解为两个文件是否为同一个文件，这个判断用于硬链接判断是很好的方法

例子
```shell
# 创建一个硬链接
ln /root/student.txt/tmp/stu.txt

# 测试
[/root/student.txt -ef /tmp/stu.txt] && echo "yes" || echo "no"
```

### 两个整数之间的比较

测试选项|作用
---|---
整数1 -eq 整数2 | 判断整数 1 是否和整数 2 相等
整数1 -ne 整数2 | 判断整数 1 是否和整数 2 不相等
整数1 -gt 整数2 | 判断整数 1 是否大于整数 2
整数1 -lt 整数2 | 判断整数 1 是否小于整数 2
整数1 -ge 整数2 | 判断整数 1 是否大于等于整数 2
整数1 -le 整数2 | 判断整数 1 是否小于等于整数 2

例子

```shell
[ 23 -ge 22 ] && echo "yes" || echo "no"
[ 23 -le 22 ] && echo "yes" || echo "no"
```

### 字符串的判断

测试选项|作用
---|---
\-z | 判断字符串是否为空
\-n | 判断字符串是否为非空
字符串1==字符串2 | 判断字符串 1 是否和字符串 2 相等
字符串1!=字符串2 | 判断字符串 1 是否和字符串 2 不相等


例子
```shell
# 给 name 变量赋值
name=fengj

# 判断 name 变量是否为空
[ -z "$name" ] && echo "yes" || echo "no"

# 给变量 aa 和变量 bb 赋值
aa=11
bb=22

# 判断两个变量的值是否相等
[ "$aa" == "$bb" ] && echo yes || echo no
```

### 多重条件判断

测试选项|作用
---|---
判断1 -a 判断2 | 逻辑与，判断 1 和判断 2 都成立，最终的结果才为真
判断1 -o 判断2 | 逻辑或，判断 1 和判断 2 有一个成立，最终的结果就为真
! 判断 | 逻辑非，使原始的判断式取反

例子
```shell
# 给变量 aa 赋值
aa=11

# 判断变量 aa 是否有值，同时判断变量 aa 是否大于 23
[ -n "$aa" -a "$aa" -gt 23 ] && echo "yes" || echo "no"
```

## Shell 单分支if语句

### 概述

如何 "背" 程序

1. 抄写目标程序并能正确运行
1. 为目标程序补全注释
1. 删掉注释，为代码重新加注释
1. 删除代码，看注释写代码
1. 删除注释和代码，从头开始写

### 单分支 if 语句

语法支持
```Shell
if [ 条件判断式 ];then
	程序
fi

if [ 条件判断式 ]
	then
	程序
fi
```

例子1：判断登录的用户是否为 root
```shell
#!/bin/bash

test=$(env|grep "USER" |cut -d "=" -f2)

if [ "$test" == root ]
	then
	echo "Current user is root."
fi
```

### 单分支 if 语句例子：判断分区使用率

```shell
#!/bin/bash

# 统计根分区使用率，把根分区使用率作为变量值赋予变量 rate
rate=$(df -h | grep "/dev/disk1" | awk '{print $5}' | cut -d "%" -f 1)

if [ $rate -ge 70 ]
then
	echo "Warning! /dev/disk1 is full!!"
fi
```

## Shell 双分支if语句

语法支持
```shell
if [ 条件判断式 ]
	then
		条件成立时，执行的程序
	else
		条件不成立时，执行的另一个程序
fi
```

### 双分支 if 语句例子：判断输入的是否是一个目录
```shell
#!/bin/bash

read -t 30 -p "Please input a dir:" dir

if [ -d "$dir" ]
then
    echo "is a Dir"
else
    echo "is not a Dir"
fi
```

### 双分支 if 语句例子：判断Apache服务是否启动
```shell
#!/bin/bash

# 列出所有进程，选取 httpd 进程，反选 grep
test=$(ps aux | grep httpd | grep -v grep)

if [ -n "$test" ]
	then
		echo "$(date) httpd is ok!" >> /tmp/autostart-acc.log
	else
		sudo apachectl restart  &> /dev/null
		echo "$(date) restart httpd !!" >> /tmp/autostart-err.log
fi
```

## Shell 多分支 if 语句

语言支持：
```shell
if [ 条件判断式 1 ]
	then
		条件成立 1 时，执行的程序
elif [ 条件判断式 2 ]
	then
		条件成立 2 时，执行的程序
else
		条件不成立时，执行的另一个程序
fi
```

### 多分支 if 语句例子：判断用户输入的是什么文件
```shell
#!/bin/bash
# 判断用户输出的是什么文件

# 接受键盘的输入，并赋予变量file
read -p "Please input a filename:" file

# 判断变量是否为空
if [ -z "$file" ]
then
	echo "Error,Please input a filename"
	exit 1
# 判断 file 的值是否存在
elif [ ! -e "$file" ]
then
	echo "Your input is not a file!"
	exit 2
# 判断 file 的值是否为普通文件
elif [ -f "$file" ]
then
	echo "$file is a regulare file!"
# 判断 file 的值是否为目录文件
then
	echo "$file is a directory!"
else
	echo "$file is an other file!"
```

## Shell 多分支case语句

语法支持
```shell
case $变量名 in
	"值1")
		如果变量的值等于值 1，则执行程序1
	;;
	"值2")
		如果变量的值等于值 2，则执行程序2
	;;
	*)
		如果变量的值都不是以上的值，则执行此程序
	;;
```
