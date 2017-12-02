---
title: MySQL 性能优化
date: 2016-10-13 20:32:23
tags: MySQL
category: MySQL
---


## 可以进行优化的层面

- 硬件
- 系统配置
- 数据库表结构
- SQL 语句和索引

## 进行优化前的数据准备

打开以下链接下载数据
```
http://downloads.mysql.com/docs/sakila-db.zip
```

打开终端，执行以下命令
```shell
# 登录 MySQL Cli 模式
mysql -uroot -p

# 创建数据库
SOURCE /Users/LuisEdware/Downloads/sakila-db/sakila-schema.sql.sql

# 填充数据到数据库
SOURCE /Users/LuisEdware/Downloads/sakila-db/sakila-data.sql

# 使用 sakila 数据库
USE sakila;
```


## SQL 语句和索引

### MySQL 慢查询日志

如何发现有问题的 SQL？答案是使用 MySQL 慢查询日志对有效率问题的 SQL 进行监控，执行命令如下：
```shell
# 查看是否开启慢查询日志
show variables like "slow_query_log";

# 查看是否设置了把没有索引的记录到慢查询日志
show variables like "log_queries_not_using_indexes";

# 查看是否设置慢查询的 SQL 执行时间
show variables like "long_query_time";

# 查看慢查询日志记录位置
show variables like "slow_query_log_file";

# 开启慢查询日志
set global slow_query_log=on

# 设置没有索引的记录到慢查询日志
set global log_queries_not_using_indexes=on

# 设置到慢查询日志的 SQL 执行时间
set global long_query_time=0

# 查看慢查询日志（在 Linux 终端下执行）
tail -50 /usr/local/var/mysql/luyiyuandeMacBook-Pro-slow.log;
```

慢查询日志所包含的内容

- SQL 的执行时间：**\# Time: 2016-10-13T10:01:45.914267Z**
- SQL 的执行主机：**\# User@Host: root[root] @ localhost []  Id:   949**
- SQL 的执行信息：**\# Query_time: 0.000227  Lock_time: 0.000099 Rows_sent: 2  Rows_examined: 2**
- SQL 的执行时间：**SET timestamp=1476352905;**
- SQL 的执行内容：**select * from store;**

慢查询日志分析工具

- mysqldumpslow
	- 安装：MySQL 数据库自带
	- 使用：`mysqldumpslow /usr/local/var/mysql/luyiyuandeMacBook-Pro-slow.log;`
	- 选项
	- 参数
- pt-query-digest
	- 安装：`brew install brew install percona-toolkit`
	- 使用：`pt-query-digest /usr/local/var/mysql/luyiyuandeMacBook-Pro-slow.log | more;`
	- 选项
	- 参数


如何通过慢查询日志发现有问题的 SQL？

- 查询次数多且每次查询占用时间长的 SQL
- IO 大的 SQL
- 未命中索引的 SQL

### EXPLAIN 分析 SQL 的执行计划

使用 EXPLAIN 分析 SQL 的执行计划的例子如下：
```sql
EXPLAIN SELECT * FROM staff;
```

使用 EXPLAIN 分析 SQL 的各列参数含义如下：

- id：SQL 语句执行顺序编号
- select_type：SQL 语句执行的类型，主要区别普通查询、联合查询和子查询之类的复杂查询
- table：SQL 语句执行所引用的数据表
- type：显示连接使用的类型
- possible_keys：指出 MySQL 能在该数据表中使用哪些索引有助于查询
- key：SQL 语句执行时所使用的索引
- key_len：SQL 语句执行时所使用的索引的长度。在不损失精确性的情况下，长度越短越好
- ref：显示索引的哪一列被使用了
- rows：MySQL 认为必须检查的用来返回请求数据的行数
- Extra：提供 MySQL 优化器一系列额外信息

### MAX() 和 COUNT() 的优化

#### MAX()

分析 SQL 语句：使用 MAX() 方法查询最后一笔交易的时间
```shell
EXPLAIN SELECT MAX(payment_date) FROM payment_date
```
执行结果如下：
```
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: payment
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 16086
     filtered: 100.00
        Extra: NULL
1 row in set, 1 warning (0.00 sec)
```

如果数据表的数据非常大，查询频率又非常高，那么服务器的 IO 消耗也会非常高，所以这条 SQL 语句需要优化。可以通过建立索引进行优化。执行代码如下：

```sql
CREATE INDEX idx_paydate ON payment(payment_date);
```

然后再分析 SQL 语句，执行结果如下：
```shell
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: NULL
   partitions: NULL
         type: NULL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: NULL
     filtered: NULL
        Extra: Select tables optimized away
1 row in set, 1 warning (0.01 sec)
```

经过优化之后，由于索引是按顺序排列的，MySQL 不需要查询表中的数据，而是通过查询索引最后的一个数据，就可以得知执行结果了。而且这个时候，不管表的数据量多大，查询 MAX() 所需要的时间都是基本固定的，这样就尽可能地减少了 IO 操作。

#### COUNT()

分析 SQL 语句：使用 COUNT() 函数在一条 SQL 中同时查出 2006 年和 2007 年电影的数量
```
SELECT
	count(release_year = '2006' OR NULL) AS '2006 年电影数量' ,
	count(release_year = '2007' OR NULL) AS '2007 年电影数量'
FROM
	film;
```

count(*) 包含空值，count(id) 不包含空值。上述语句就是优化 Count() 函数取值


### 子查询

分析 SQL 语句：查询 sandra 出演的所有影片
```sql
SELECT
	title ,
	release_year ,
	LENGTH
FROM
	film
WHERE
	film_id IN(
		SELECT
			film_id
		FROM
			film_actor
		WHERE
			actor_id IN(
				SELECT
					actor_id
				FROM
					actor
				WHERE
					first_name = 'sandra'
			)
	)
```

通常情况下，需要把子查询优化为 join 查询，但在优化时要注意关联键是否有一对多的关系，要注意重复数据。


### GROUP BY

group by 可能会出现临时表、文件排序等，影响效率。可以通过关联的子查询，来避免产生临时表和文件排序，可以节省 IO。

group by 查询优化前：
```sql
EXPLAIN SELECT
	actor.first_name ,
	actor.last_name ,
	Count(*)
FROM
	sakila.film_actor
INNER JOIN sakila.actor USING(actor_id)
GROUP BY
	film_actor.actor_id;
```
执行结果如下：
```
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: actor
   partitions: NULL
         type: ALL
possible_keys: PRIMARY
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 200
     filtered: 100.00
        Extra: Using temporary; Using filesort
*************************** 2. row ***************************
           id: 1
  select_type: SIMPLE
        table: film_actor
   partitions: NULL
         type: ref
possible_keys: PRIMARY,idx_fk_film_id
          key: PRIMARY
      key_len: 2
          ref: sakila.actor.actor_id
         rows: 27
     filtered: 100.00
        Extra: Using index
2 rows in set, 1 warning (0.01 sec)
```

group by 查询优化后：
```
EXPLAIN SELECT
	actor.first_name ,
	actor.last_name ,
	c.cnt
FROM
	sakila.actor
INNER JOIN(
	SELECT
		actor_id ,
		count(*) AS cnt
	FROM
		sakila.film_actor
	GROUP BY
		actor_id
) AS c USING(actor_id);
```
执行结果如下：
```
*************************** 1. row ***************************
           id: 1
  select_type: PRIMARY
        table: actor
   partitions: NULL
         type: ALL
possible_keys: PRIMARY
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 200
     filtered: 100.00
        Extra: NULL
*************************** 2. row ***************************
           id: 1
  select_type: PRIMARY
        table: <derived2>
   partitions: NULL
         type: ref
possible_keys: <auto_key0>
          key: <auto_key0>
      key_len: 2
          ref: sakila.actor.actor_id
         rows: 27
     filtered: 100.00
        Extra: NULL
*************************** 3. row ***************************
           id: 2
  select_type: DERIVED
        table: film_actor
   partitions: NULL
         type: index
possible_keys: PRIMARY,idx_fk_film_id
          key: PRIMARY
      key_len: 4
          ref: NULL
         rows: 5462
     filtered: 100.00
        Extra: Using index
3 rows in set, 1 warning (0.00 sec)
```

### LIMIT

LIMIT 常用于分页处理，时常会伴随 ORDER BY 从句使用，因此大多时候会使用 Filesorts ，这样会造成大量的 IO 问题

优化步骤1：使用有索引的列或主键进行 Order By 操作
优化步骤2：记录上次返回的主键，在下次查询时使用主键过滤（保证主键是自增且有索引）

### 索引优化

#### 1.为合适的列建立索引

- 在 where 从句，group by 从句，order by 从句，on 从句中出现的列
- 索引字段越小越好
- 离散度的列放到联合索引的前面

例如：
```
SELECT * FROM payment WHERE staff_id = 2 AND customer_id = 584;
```
上述 SQL 语句，是 `index(staff_id,customer_id)` 合理，还是 `index(customer_id,staff_id)` 合理。执行语句如下：
```
SELECT count(DISTINCT customer_id) , count(DISTINCT staff_id) FROM payment;

-- 结果是 599	2
```
由于 customer_id 的离散度更大，所以应该使用 `index(customer_id,staff_id)`

#### 2.找到重复和冗余的索引

之所以要找到重复和冗余的索引，是因为过多的索引不但影响写入，而且影响查询，索引越多，分析越慢。那么为何重复索引、冗余索引？概念如下：

重复索引是指相同的列以相同的顺序建立的同类型的索引，如下表中 primary key 和 ID 列上的索引就是重复索引，例子如下：
```sql
CREATE TABLE test(
	id INT NOT NULL PRIMARY KEY ,
	NAME VARCHAR(10) NOT NULL ,
	title VARCHAR(50) NOT NULL ,
	UNIQUE(id)
) ENGINE = INNODB;
```
`UNIQUE(ID)` 和 `PRIMARY KEY` 重复了。

冗余索引是指多个索引的前缀列相同，或是在联合索引中包含了主键的索引，例子如下：
```sql
CREATE TABLE test(
	id INT NOT NULL PRIMARY KEY ,
	NAME VARCHAR(10) NOT NULL ,
	title VARCHAR(50) NOT NULL ,
	KEY(NAME , id)
) ENGINE = INNODB;
```

查找重复及冗余索引的 SQL 语句如下：
```
USE information_schema;

SELECT
	a.TABLE_SCHEMA AS '数据名' ,
	a.table_name AS '表名' ,
	a.index_name AS '索引1' ,
	b.INDEX_NAME AS '索引2' ,
	a.COLUMN_NAME AS '重复列名'
FROM
	STATISTICS a
JOIN STATISTICS b ON a.TABLE_SCHEMA = b.TABLE_SCHEMA
AND a.TABLE_NAME = b.table_name
AND a.SEQ_IN_INDEX = b.SEQ_IN_INDEX
AND a.COLUMN_NAME = b.COLUMN_NAME
WHERE
	a.SEQ_IN_INDEX = 1
AND a.INDEX_NAME <> b.INDEX_NAME
```

也可以使用工具 `pt-duplicate-key-checker` 检查重复索引和冗余索引,使用例如：
```
pt-duplicate-key-checker -uroot -p '123456' -h 127.0.0.1 -d sakila
```
执行结果如下：
```sql
# ########################################################################
# Summary of indexes
# ########################################################################

# Size Duplicate Indexes   118425374
# Total Duplicate Indexes  24
# Total Indexes            1439
```

#### 3.删除不用的索引

目前 MySQL 中还没有记录索引的使用情况，但是在 PerconMySQL 和 MariaDB 中可以通过 INDEX_STATISTICS 表来查看哪些索引未使用，但在 MySQL 中目前只能通过慢查询日志配合共组 `pt-index-usage` 来进行索引使用情况的分析。

```
pt-index-usage -uroot -p '123456' /usr/local/var/mysql/luyiyuandeMacBook-Pro-slow.log;
```

## 数据库结构优化

### 选择合适的数据类型

- 使用可以存下你的数据的最小的数据类型
- 使用简单的数据类型。Integer 要比 varchar 类型在 MySQL 中处理更高效
- 尽可能使用 not null 定义字段
- 尽量少用 text 类型，非用不可时最好考虑分表


### 数据库表的范式化优化与反范式化优化

略

### 数据库表的垂直拆分

垂直拆分，就是把原来一个有很多列的表拆分成多个表，这解决了表的宽度问题。通常垂直拆分就可以按以下原则进行：

- 把不太常用的字段单独存放到一个表中
- 把大字段独立存放到一个表中
- 把经常一起使用的字段放到一起

### 数据库表的水平拆分

当单表的数据量过大，导致增删查改等操作过慢，这时候需要对表进行水平拆分。水平拆分的表，每一张表的结构都是完全一致的。

常用的水平拆分方法为：

1. 对 customer_id 进行 hash 运算，如果要拆分成 5 个表则使用 mod(customer_id,5) 取出 0-4 个值
1. 针对不同的 hashID 把数据存到不同的表中

挑战：

- 跨分区表进行数据查询
- 统计及后台报表操作

## 系统配置优化

### 数据库系统配置优化

数据库是基于操作系统的，目前大多数 MySQL 都是安装在Linux 系统之上，所以对于操作系统的一些参数配置也会影响到 MySQL 的性能，下面列举一些常用到的系统配置。

网络方面的配置，要修改文件 `/etc/sysctl.conf`
```
# 增加 tcp 支持的队列数
net.ipv4.tcp_max_syn_backlog = 65535

# 减少断开连接时，资源回收
net.ipv4.tcp_max_tw_buckets = 8000
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_fin_timeout = 10
```

打开文件数的限制，可以使用 ulimit -a 查看目录的各项限制，可以修改文件 `/etc/security/limits.conf` ，增加以下内容以修改打开文件数量的限制
```
soft nofile 65535
hard nofile 65535
```
除此之外最好在 MySQL 服务器上关闭 iptables，selinux 等防火墙软件。

### MySQL 配置文件

MySQL 可以通过启动时指定配置参数和使用配置文件两种方法进行配置，在一般情况下，配置文件位于 `/etc/my.cnf` 或是 `/etc/mysql/my.cnf`，MySQL 查询配置文件的顺序是可以通过以下方法过的

常用参数说明

- innodb_buffer_pool_size：用于配置 Innodb 的缓冲池
    - 如果数据库中只有 Innodb 表，则推荐配置量为总内存的 75%
    - Innodb_buffer_pool_size >= Total MB
```sql
SELECT
ENGINE ,
round(
  sum(data_length + index_length) / 1024 / 1024 ,
  1
) AS 'Total MB'
FROM
information_schema. TABLES
WHERE
table_schema NOT IN(
  "information_schema" ,
  "performance_schema"
)
GROUP BY
ENGINE;
```
- **innodb_buffer_pool_instances**：MySQL 5.5 中新增参数，可以控制缓冲池的个数，默认情况下只有一个缓冲池。
- **innodb_log_buffer_size**：Innodb 日志缓冲的大小，由于日志最长，每秒钟就会刷新，所以一般不用太大。
- **innodb_flush_log_at_trx_commit**：对 Innodb 的 IO 效率影响很大。
- **innodb_file_per_table**：控制 Innodb 每一个表都使用独立的表空间，默认为 OFF，也就是所有表都会建立在共享表空间中。
- **innodb_stats_on_metadata**：决定 MySQL 在什么情况下会刷新 innodb 表的统计信息。

### 第三方配置工具使用

**`percona：https://tools.percona.com/`**

## 服务器硬件优化

- 如何选择 CPU
  - MySQL 有一些工作只能使用到单核 CPU，选择高频
  - MySQL 对 CPU 核数的支持并不是越多越快，MySQL 5.5 版本不要超过 32 个核
- 硬盘 IO 优化
  - RAID 级别简介
    - RAID 0：也称为条带，就是把多个磁盘链接成一个硬盘使用，这个级别 IO 最好
    - RAID 1：也成为镜像，要求至少两个磁盘，每组磁盘存储的数据相同
    - RAID 1 + 0：就是 RAID 1 和 RAID 0的结合。同时具备两个级别的优缺点。一般建议数据库使用这个级别。
    - RAID 5：把多个（最少 3 个）硬盘合并成 1 个逻辑盘使用，数据读写时会建立奇偶校验信息，并且奇偶校验信息和相对应的数据分别存储在不同的磁盘上。当 RAID 5 的一个磁盘数据发生损坏后，利用剩下的数据和相应的奇偶校验信息去恢复被损坏的数据。
