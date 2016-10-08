---
title: MySQL 5.7 版本新特性
date: 2016-09-12 21:12:40
tags: MySQL
category: MySQL
---

## 第 1 章 MySQL 服务功能增强


### 1-1 初始化方式变更

MySQL 5.7 之前
`scripts/mysql_install_db \
	--datadir=/data/sql_data \
	--user=mysql --basedir=/home/mysql`

MySQL 5.7 之后
`bin/mysql --initialize --user = mysql \
	--basedir=/home/mysql \
	--datadir=/home/mysql/data`

### 1-3 旧版本支持为表增加计算列演练

MySQL 5.7 之前，需要增加一个插入触发器和更新触发器来实现计算列的功能，或者是一个视图来实现计算列的功能

MySQL 5.7 之后，在 `CREATE TABLE` 及 `ALTER TABLE` 语句中支持增加计算列的方式
```sql
col_name data_type [GENERATED ALWAYS] AS (expression)
[VIRTUAL | STORED] [UNIQUE [KEY]] [COMMENT comment]
[[NOT] NULL] [[PRIMARY] KEY]
```

### 1-4 MySQL5.7支持为表增加计算列实际演练

```sql
CREATE TABLE t(id int AUTO_INCREMENT not null,c1 int,c2 int,c3 int as (c1+c2),PRIMARY KEY (id));

INSERT INTO t(c1,c2) VALUES(1,2);

SELECT * FROM t;

UPDATE t set c1 = 5 WHERE id = 1;

SELECT * FROM t;
```

### 1-5 引入JSON列类型及相关函数

MySQL 5.7 之前

只能在 varchar 或是 text 等字符类型的列中存储 JSON 类型的字符串，并通过程序解析使用 JSON 字符串。

MySQL 5.7 之后

增加了 JSON 列类型及以 JSON 开头的相关处理函数，如 JSON_TYPE(),JSON_OBJECT()，JSON_MERGE() 等。
```
SELECT JSON_ARRAY(1,2,3,now());
SELECT JSON_OBJECT('keyA',1,'keyB',2);
```

## 第 2 章 Replication 相关增强

### 2-1 支持多源复制

略

### 2-2 基于库或是逻辑锁的多线程复制

MySQL 5.7 之前

从 MySQL 5.6 开始支持多线程复制，只不过是对于每一个库一个复制线程。

MySQL 5.7 之后

MySQL 5.7 后对多线程复制功能进行了加强，增加了 `slave_paraller_type` 参数可以控制并发同步是基于 `database` 还是 `logical_clock`

### 2-3 在线变更复制方式

MySQL 5.7 之前

要把基于日志点的复制方式变为基于 gtid 的复制方式或是把基于 gtid 的复制方式变成基于日志点的复制方式必须要重启 master 的服务器

MySQL 5.7 之后

允许在线变列复制的方式，而不用重启 master 服务器


## 第 3 章 InnoDB 引擎增强

### 3-1 支持缓冲池大小在线变更

MySQL 5.7 之前

要变更 `innodb_buffer_pool` 大小必须更改 `my.cnf` 文件后重启数据库服务器

MySQL 5.7 之后

`innodb_buffer_pool_size` 参数变为动态参数，可以在线调整 `innodb` 缓存池的大小

### 3-2 增加 `innodb_buffer_pool` 导入导出功能

MySQL 5.7 之后，新增以下参数控制 `innodb_buffer_pool` 的导入导出
```
innodb_buffer_pool_dump_pct
innodb_buffer_pool_dump_now
innodb_buffer_pool_dump_at_shutdown
innodb_buffer_pool_load_at_startup
innodb_buffer_pool_load_now
```

### 3-3 支持为 `innodb` 表建立表空间

MySQL 5.7 之前

具有系统表空间及可以为每个表建立一个独立的表空间


MySQL 5.7 之后

支持 `CREATE TABLESPACE` 语法为一个表或多个表建立共用的表空间


## 第 4 章 安全及管理方面增强

### 4-1 不再支持old_password认证



### 4-2 增加账号默认过期时间

`show variables like 'default_password_lifetime';`

### 4-3 加强了对账号的管理功能

### 4-4 增加了sys管理数据库

