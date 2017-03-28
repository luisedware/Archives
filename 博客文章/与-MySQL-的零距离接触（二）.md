---
title: 与 MySQL 的零距离接触（二）
date: 2016-09-09 20:14:34
tags: MySQL
category: MySQL
---


## 第 6 章 运算符和函数

- ### 6-1 回顾和概述
	- 运算符
	- 函数
		- 字符函数
		- 数值运算符和函数
		- 比较运算符和函数
		- 日期时间函数
		- 信息函数
		- 聚合函数
		- 加密函数
- ### 6-2 字符函数
	- CONCAT() - 字符连接
	- CONCAT_WS() - 使用指定的分隔符进行字符连接
	- FORMAT() - 数字格式化
	- LOWER() - 转换成小写字母
	- UPPER() - 转换成大写字母
	- LEFT() - 获取左侧字符
	- RIGHT() - 获取右侧字符
	- LENGTH() - 获取字符串长度
	- LTRIM() - 删除前导空格
	- RTRIM() - 删除后续空壳
	- TRIM() - 删除前导和后续空格
	- SUBSTRING() - 字符串截取
	- [NOT] LIKE - 模式匹配
		- %（百分号）：代表任意个字符
		- _（下划线）：代表任意一个字符
	- REPLACE() - 字符串替换
- ### 6-3 数值运算符和函数
	- CEIL() - 进一取整
	- DIV - 整数除法
	- FLOOR() - 舍一取整
	- MOD - 取余数
	- POWER() - 幂运算
	- ROUND() - 四舍五入
	- TRUNCATE() - 数字截取
- ### 6-4 比较运算符和函数
	- [NOT] BETWEEN...AND... - [不]在范围之内
	- [NOT] IN() - [不]在列出值范围内
	- IS [NOT] NULL - [不]为空
- ### 6-5 日期时间函数
	- NOW() - 当前日期和时间
	- CURDATE() - 当前日期
	- CURTIME() - 当前时间
	- DATE_ADD() - 日期变化
	- DATEDIFF() - 日期差值
	- DATE_FORMAT() - 日期格式化
- ### 6-6 信息函数
	- CONNECTION_ID() - 连接 ID
	- DATEBASE() - 当前数据库
	- LAST_INSERT_ID() - 最后插入记录的 ID
	- VERSION() - 最后插入记录的 ID 号
	- USER() - 当前用户
- ### 6-7 聚合函数
	- AVG() - 平均值
	- COUNT() - 计数
	- MAX() - 最大值
	- MIN() - 最小值
	- SUM() - 求和
- ### 6-8 加密函数
	- MD5() - 信息摘要算法
	- PASSWORD() - 密码

## 第 7 章 自定义函数

- ### 7-1 回顾和概述
	- 略
- ### 7-2 自定义函数简介
	- 自定义函数：用户自定义函数（user-defined function,UDF）是一种对 MySQL 扩展的途径，其用法与内置函数相同。
	- 自定义函数的两个必要条件：
		- 参数
		- 返回值
		- 函数可以接受任意类型的参数，也可以返回任意类型的值
		- 函数的参数与返回值没有任何必然、内在的联系
	- 创建自定义函数
		- `CREATE FUNCTION function_name RETURNS {STRING|INTEGER|REAL|DECIMAL} routine_body`
		- routine_body - 函数体
			- 函数体由合法的 SQL 语句构成
			- 函数体可以是简单的 SELECT 或 INSERT 语句
			- 函数体如果为复合结构则使用 BEGIN...END 语句
			- 复合结构可以包含声明，循环，控制结构
- ### 7-3 创建不带参数的自定义函数
	- `create function f1() returns varchar(30) return date_format(now(),'%Y年%m月%d %H:%i:%s');`
	- 执行 `SELECT f1();`，输出 `2016年09月09 17:30:08`
- ### 7-4 创建带有参数的自定义函数
	- `create function f2(num1 smallint unsigned,num2 smallint unsigned) returns float(10,2) unsigned return (num1+num2)/2;`
	- 执行 `SELECT f2(10,30);`，输出 `20.00`
- ### 7-5 创建具有复合结构函数体的自定义函数
	- 重定义 SQL 语句结束分隔符 - `DELIMITER string`
	- `create function adduser(username varchar(20)) returns int unsigned begin insert test(username) values(username); return last_insert_id(); end //`

## 第 8 章 MySQL 存储过程

- 8-1 课程回顾
	- 略
- 8-2 存储过程简介
	- SQL 命令执行过程
	 	- SQL 命令 > MySQL 引擎 > 分析语法 > 可识别命令 > 执行 > 返回执行结果
	 - 存储过程的简介
	 	- 存储过程是 SQL 语句和控制语句的预编译集合，以一个名称存储并作为一个单元处理
	 - 存储过程的有点
	 	- 增强 SQL 语句的功能和灵活性
	 	- 实现较快的执行速度
	 	- 减少网络流量
- 8-3 存储过程语法结构解析
	- 创建存储过程的语法结构
		- `CREATE`
		- `[DEFINER = {user | CURRENT_USER}]`
		- `PROCEDURE sp_name([proc_parameter[,...]])`
		- `[characteristic ...] routine_body`
		- `proc_parameter:`
		- `[IN | OUT | INOUT] param_name type `
	- 创建存储过程的参数含义
		- IN，表示该参数的值必须在调用存储过程时指定
		- OUT，表示该参数的值可以被存储过程改变，并且可以返回
		- INOUT，表示该参数在调用时指定，并且可以被改变和返回
	- 创建存储过程的过程体
		- 过程体由合法的 SQL 语句构成
		- 过程体可以是任意 SQL 语句
		- 过程体如果为复合结构则使用 BEGIN...END 语句
		- 复合结构可以包含声明，循环，控制结构
- 8-4 创建不带参数的存储过程
	- SQL 语句 - `CREATE PROCEDURE sp1() SELECT VERSION();`
	- 调用 - `CALL sp1();` 或 `CALL sp1`
- 8-5 创建带有IN类型参数的存储过程
	- SQL 语句 - `CREATE PROCEDURE removeUserById(IN user_id INT UNSIGNED) BEGIN DELETE FROM users WHERE id = user_id; END //`
	- 调用 - `CALL removeUserById(22);`
- 8-6 创建带有IN和OUT类型参数的存储过程
	- SQL 语句 - `CREATE PROCEDURE removeUserAndReturnUserNums(IN user_id INT UNSIGNED,OUT user_nums INT UNSIGNED) BEGIN DELETE FROM users WHERE id = user_id; SELECT cout(id) FROM users INTO user_nums; END //`
	- 调用 - `CALL removeUserAndReturnUserNums(27,@user_nums); SELECT @user_nums`
	- `@name` 声明用户变量，只能是当前用户使用
- 8-7 创建带有多个OUT类型参数的存储过程
	- SQL 语句 - `CREATE PROCEDURE removeUserByAgeAndReturnInfos(IN user_age SMALLINT UNSIGNED,OUT deleteUsers SMALLINT UNSIGNED,OUT user_counts SMALLINT UNSIGNED) BEGIN DELETE FROM users WHERE age = user_age; SELECT ROW_COUNT() INTO deleteUsers; SELECT COUNT(id) FROM users INTO user_counts; END //`
	- 调用 - `CALL removeUserByAgeAndReturnInfos(20,@a,@b)`
- 8-8 存储过程与自定义函数的区别
	- 存储过程实现的功能要复杂一些；而函数的针对性更强
	- 存储过程可以返回多个值；函数只能有一个返回值
	- 存储过程一般都是独立执行；而函数可以作为其他 SQL 语句的组成部分来出现。

## 第 9 章 MySQL 存储引擎

- 9-1 课程回顾
	- 略
- 9-2 存储引擎简介
	- 简介：MySQL 可以将数据以不同的技术存储在文件（内存）中，这种技术就成为存储引擎。每一种存储引擎使用不同的存储机制、索引技巧、锁定水平，最终提供广泛且不同的功能。
	- 类型：
		- MyISAM
		- InnoDB
		- Memory
		- CSV
		- Archive
- 9-3 相关知识点之并发处理
	- 并发控制：当多个连接对记录进行修改时保证数据的一致性和完整性
	- 锁
		- 共享锁（读锁）：在同一时间段内，多个用户可以读取同一个资源，读取过程中数据不会发生任何变化。
		- 排它锁（写锁）：在任何时候只能有一个用户写入资源，当进行写锁时会阻塞其他的读锁或者写锁操作。
	- 锁颗粒
		- 表锁，是一种开销最小的锁策略
		- 行锁，是一种开销最大的锁策略
- 9-4 相关知识点之事务处理
	- 事务
		- 事务用于保证数据库的完整性
- 9-5 相关知识点之外键和索引
	- 外键
		- 外键是保证数据一致性的策略
	- 索引
		- 是对数据表中一列或多列的值进行排序的一种结构。
- 9-6 各个存储引擎特点

特性/引擎|MyISAM | Aria | InnoDB | XtraDB | Memory | Archive|
---|---|---|---|---|---|---
数据存储|传统顺序数据储存 |传统顺序数据储存|表空间存储方式|表空间存储方式|||
事务支持|不支持 | 支持|支持|支持|不支持|不支持|
外键|不支持 | 不支持|支持|支持|不支持|不支持|
全文检索|支持 | 支持|5.6 之后支持|5.6 之后支持|||
锁级别|表级锁 | 表级锁|行级锁|行级锁|表级锁|行级锁|
Count 速度|快 | 快|慢|慢|||
适合业务|读多写少、单表数据量小于 1 KW |读多写少、单表数据量小于 1 KW|读写均衡、数据量不限|读写均衡、数据量不限|||
可用版本|MySQL、MariaDB、Percona | MariaDB|MySQL、MariaDB、Percona|MariaDB、Percona|||
其他说明|传统顺序索引数据库，适合读多写少小数据量业务 | MyISAM 增强版，性能更好|适合高压力高性能的业务模型|InnoDB 增强版|||

- 9-7 设置存储引擎
	- 修改搜索引擎的方法，主要有两种：
		- 通过修改 MySQL 配置文件实现：`default-storage-engine = engine`
		- 通过创建数据表命令实现
			- `CREATE TABLE table_name(...,...) ENGINE = engine;`
		- 通过修改数据表命令实现
			- `ALTER TABLE table_name ENGINE [=] engine_name;`

## 第 10 章 MySQL 图形化管理工具

- 10-1 课程介绍
	- PHPMyAdmin
	- Navicat
	- MySQL Workbench
- 10-2 管理工具之 phpMyAdmin
- 10-3 管理工具之 Navicat for MySQL
- 10-4 管理工具之 MySQL Workbench


