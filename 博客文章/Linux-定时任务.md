---
title: Linux 定时任务
date: 2016-09-26 17:25:18
tags: Linux
category: Linux
---


## Linux 定时任务

- at 一次性定时任务
- crontab 循环定时任务
- crontab 设置
- anacron 配置

## at 一次性定时任务

```shell
# at 服务是否安装
chkconfig --list | grep atd

# at 服务的启动
service atd restart
```

at 的访问控制

- 如果系统中有 /etc/at.allow 文件，那么只有写入 /etc/at.allow （白名单）文件中的用户可以使用 at 命令（/etc/at.deny 文件会被忽略）
- 如果系统中没有 /etc/at.allow 文件，只有 /etc/at.deny 文件，那么写入 /etc/at.deny （黑名单）文件中的用户不能使用 at 命令。对 root 不起作用
- 如果系统中这两个文件都不存在，那么只有 root 用户可以使用 at 命令


at 命令

- at
	- `at [选项] 时间`
		- 选项：
			- \-m 当 at 工作完成后，无论命令是否有输出，都用 email 通知执行 at 命令的用户
			- \-c 工作号：显示该 at 工作的实际内容
		- 时间：
			- HH:MM 02:30
			- HH:MM YYYY-MM-DD 02:30 2013-07-25
			- HH:MM[am|pm] [month] [date] 02:30 July 25
			- HH:MM[am|pm] + [minutes|hours|days|weeks] now + 5 minutes
		- 例子：
			- `at now + 2 minutes` && `/root/hello.sh >> /root/hello.log`
			- `at 02:00 2013-07-26` && `/bin/sync` && `/sbin/shutdown -r now`
		- 其他：
			- `atq` 查询当前服务器上的 at 工作
			- `atrm [工作号]` 删除指定的 at 任务


## anncron

anacron 是用来保证在系统关机的时候错过的定时任务，可以在系统开机之后再执行

anacron 检测周期

- anacron 会使用一天，七天，一个月作为检测周期
- 在系统的 /var/spool/anacron/ 目录中存在 cron.{daily，weekly，monthly} 文件，用于记录上次执行的 cron 的时间
- 和当前时间做比较，如果两个时间的差值超过了 anacron 的指定时间差值，证明有 cron 任务被漏执行

anacron 配置文件

- vim /etc/anacrontab
	- `RANDOM_DELAY=45` 最大随机延迟
	- `START_HOURS_RANGE=3-22` anacron 的执行时间范围是 3:00 - 22:00
	- `1	5	cron.daily 		nice run-parts /etc/cron.daily` 天数 强制延迟（分钟） 工作名称 实际执行的命令

anacron 执行过程（以 cron.daily 为例）

- 首先读取 /var/spool/anacron/cron.daily 中的上一次 anacron 执行的时间
- 和当前时间比较，如果两个时间的差值超过 1 天，就执行 cron.daily 工作
- 执行这个工作只能在 03:00-22:00 之间
- 执行工作时强制延迟时间为 5 分钟，再随机延迟 0-45 分钟时间
- 使用 nice 命令指定默认优先级，使用 run-parts 脚本执行 /etc/cron.daily 目录中的所有可执行文件

## Crontab 简介

业务场景

- 每分钟需要执行一个程序检查系统运行状态
- 每天凌晨需要对过去一天的业务数据进行统计
- 每个星期需要把日志文件备份
- 每个月需要把数据库进行备份

概念理解

**Crontab 是一个用于设置周期性被执行的任务的工具。**

## Crontab 实践

- 相关工具
	- Mac OS 自带 SSH
- 安装并检查 Crontab 服务
	- 检查 cron 服务
		- 检查 Crontab 工具是否安装：`crontab -l`
		- 检查 Crontab 可执行的命令：`service crond`
		- 检查 Crontab 服务是否启动：`service crond status`
		- 删除当前用户所有的定时任务： `crontab -r`
		- 打开定时任务列表，进行编辑：`crontab -e`
			- 实际上在修改目录 `/var/spool/cron/root` 下用户对应的文件
			- `crontab 文件名` 会把（crontab -e）文件里的内容都覆盖，使用时需要注意
		- 启动 Crontab 服务：`service crond start`
	- 安装 Crontab 服务（依次执行）
		- `yum install vixie-cron`
		- `yum install crontabs`
	- `tail` 命令
		- 作用：按照要求将指定的文件的最后部分输出到标准设备，如果文件有更新，tail 会主动刷新，确保你看到最新的文件内容
		- 选项
			- \-f 循环读取
			- \-q 不显示处理信息
			- \-v 显示详细的处理信息
			- \-c [数目] 显示的字节数
			- \-n [行数] 显示行数
			- \-s 与-f合用,表示在每次反复的间隔休眠 s 秒
			- \-q 从不输出给出文件名的首部
- Crontab 的基本组成
	- 配置文件
	- 系统服务 Crond
	- 配置工具 Crontab
- Crontab 的配置文件格式
	- `* * * * * Command`
		- 第一个星号：分钟 0~59
		- 第二个星号：小时 0~23
		- 第三个星号：日期 1~31
		- 第四个星号：月份 1~12
		- 第五个星号：星期 0~7（0、7 都是星期天）
	- 用法：
		- 每晚的 21:30 重启 apache
			- **`30 21 * * * service httpd restart`**
		- 每月 1 到 10 日的 4:45 重启 apache
			- **`45 4 1-10 * * service httpd restart`**
		- 每月 1、10、22 日的 4:45 重启 apache
			- **`45 4 1,10,22 * * service httpd restart`**
		- 每隔两分钟重启 apache 服务器
			- **`*/2 * * * * service httpd restart`**
		- 晚上 11 点到早上 7 点之间，每隔一小时重启 apache
			- **`0 23-7/1 * * * service httpd restart`**
		- 每天 18:00 至 23:00 之间每隔 30 分钟重启 apache
			- **`0,30 18-23 * * * service httpd restart`**
			- **`0-59/30 18-23 * * * service httpd restart`**
	- 总结：
		- \* 表示任何时候都匹配
		- 可以用 "A,B,C" 表示 A 或者 B 或者 C 时执行命令
		- 可以用 "A-B" 表示 A 到 B 之间时执行命令
		- 可以用 "*/A" 表示每 A 时间执行一次命令
- Crontab 配置文件
	- 全局配置文件
		- `/etc/crontab` 查看系统级别的 Crontab
	- 用户配置文件
		- `/var/spool/cron/用户名称` 查看用户级别的 Crontab
- Crontab 的日志
	- 监听 Crontab 执行日志：`tail -f /var/log/cron`，可以看到用户级别和系统级别的 Crontab
	- 查看每天的 Crontab 日志：`ls /var/log/cron*`
- Crontab 常见错误
	- 环境变量：涉及到文件路径时写全局路径。注意声明 `#!/bin/bash` 确保脚本中环境变量能够被识别
	- 第三和第五个域之间执行的是 "或" 操作
		- 四月的第一个星期日早晨 1 时 59 分运行 a.sh：`59 1 1-7 4 * test 'date + \%w -eq 0 && /root/a.sh'`
	- 两个小时运行一次
		- 错误例子：`* 0,2,4,6,8,10,12,14,16,18,20,22 * * * date`
		- 正确例子：`0 */2 * * * date`
- Crontab 补充
	- 半分钟执行命令
		- `date && sleep 0.5s && date`
	- Crontab 最小时间是 1 分钟，控制 1 分钟执行多次，本应该是同时执行，但第二条被推迟了30s执行，效果就是1分钟执行了2次
		 - `*/1 * * * * date >> /root/test/half.log`
		 - `*/1 * * * * sleep 30s; date >> /root/test/half.log`

