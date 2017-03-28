---
title: 复习使用 MySQL（三）
date: 2016-09-04 21:08:24
tags: MySQL
category: MySQL
---

## 环境

- MySQL 5.7.13

## MySQL 优化：硬件

- CPU：使用多核 CPU，能够充分发挥新版 MySQL 多核下的效果，建议 4 核以上
- 内存：如果数据量比较大，建议使用不要低于 20 G 内存的服务器
- 硬盘：SSD 硬盘，提高 TPS，选择适合的 RAID 方案
- 网卡：保证足够的吞吐量，建议千兆网卡

## MySQL 优化：软件

- OS 类型
    - Linux
    - FreeBSD
- Linux 关键配置
	- 文件打开描述符：/etc/security/limits.conf 中 nofile、/proc/sys/fs/nr_open
	- 可分配文件句柄数：/etc/sysctl.conf 中 fs.file-max、/proc/sys/fs/file-max
	- 进程数：/etc/security/limits.conf 中的 nproc
	- 线程数：/proc/sys/kernel/thread-max
	- 其他 TCP 和网络相关选项：/etc/sysctl.conf
	- 这些都可以通过ulimit 或 直接调整 /proc 中变量进行临时修改
	- 建议关闭 SWAP 分区（内存要足够大才行）；如果数据太大，为了防止夯死主机，可以设置 2G 左右的 SWAP 分区


## MySQL 优化：基础配置

基础配置 & MyISAM 配置
```c
max_connections=3000
max_user_connections=2800
max_connect_errors=10
max_allowed_packet=64M
max_heap_table_size=512M
tmp_table_size=512M
max_length_for_sort_data=16k
wait_timeout=172800
interactive_timeout=172800

net_buffer_length=8K
read_buffer_size=4M
read_rnd_buffer_size=1M
sort_buffer_size=256K
join_buffer_size=2M
table_open_cache=512
thread_cache_size=512
query_cache_type=1
query_cache_size=256M
```

InnoDB 配置
```c
innodb_file_per_table=1
innodb_open_files=7168
innodb_use_sys_malloc=1
innodb_additional_mem_pool_size=64M
innodb_buffer_pool_instances=4
innodb_buffer_pool_size=20G
innodb_data_home_dir=/home/work/data
innodb_data_file_path=ibdata1:1024M:autoextend
innodb_autoextend_increment=128
innodb_thread_concurrency=0
innodb_flush_log_at_trx_commit=1
innodb_fast_shutdown=1
innodb_force_recovery=0

innodb_log_buffer_size=16M
innodb_log_file_size=128M
innodb_log_files_in_group=4
innodb_log_group_home_dir=/home/work/data
innodb_max_dirty_pages_pct=60
innodb_purge_threads=0
innodb_lock_wait_timeout=50
innodb_rollback_on_timeout=1
innodb_commit_concurrency=0
innodb_concurrency_tickets=1024
innodb_autoinc_lock_mode=2
innodb_change_buffering=all
```

## MySQL 优化：表设计原则

- 互联网业务的特点：数据量大、读写操作多、业务模式相对简单
- 单表字段不宜过多，尽量不按照第三范式去设计表结构，
- 尽量减少表关联，适当的冗余保证不联表查询
- 表名和字段尽量采用小写字母+下划线的命名规则，尽量使用英文，例如：user_score
- 每个表一定要有主键，一般建议自增ID；单表索引不宜过多，一般5~6个索引
- 字段尽量使用高效类型，比如数字或时间，IP地址、手机号等都可以使用数字类型存储，非不得已采用字符

## MySQL 优化：查询语句

- 不要写太复杂的SQL语句，尽量简单，能够让业务去做的，就不要让数据库区操作
- 尽量避免：子查询、group by、distinct 等操作（子查询可以用left join替代）
- where、order by 等关键字段一定要建立索引，where里多个条件经常使用的可以建立联合索引（注意建立索引的数据必须是尽量分布多并且具备单调性，重复率低）
- 把数据当做神一样来供着，能够缓存的尽量缓存（redis、memcached等），能够多次查询的就不要关联查询
- 所有的变更操作（update、delete）必须有where条件！
- 不适合使用MySQL服务的数据功能操作，尽量采用别的第三方服务支持，比如全文检索功能



## MySQL 优化：SQL 优化实例

- Like 优化
- Limit 优化
- InnoDB 中 count 优化
- 定期使用 Explain 查看执行计划



## PHP 与 MySQL 交互

- 扩展层
- 代码层
	- 为防止mysql太慢夯住PHP，建议设置SQL超时，或者设置execute_time等超时设置
	- $mysqli->options(MYSQL_OPT_READ_TIMEOUT, 3);
	- $mysqli->options(MYSQL_OPT_WRITE_TIMEOUT, 1);
	- 务必在代码里记录相关SQL语句执行性能和时间等信息，方便后续优化

- 超时设置
	- PHP代码连接后端的超时设置
	- php.ini中execute_time设置
	- php-fpm中request_terminate_timeout，request_slowlog_timeout
	- nginx upstream相关超时设置


## PHP 与 MySQL 安全

- SQL 注入
- MySQL 服务端
- 备份机制
