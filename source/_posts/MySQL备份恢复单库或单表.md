---
title: MySQL备份恢复单库或单表
categories:
  - 技术
  - 数据库
tags: MySQL
abbrlink: 7fcc32a7
date: 2024-12-30 11:29:15
---

MySQL 逻辑备份工具最常用的就是 mysqldump 了，一般我们都是备份整个实例或部分业务库恢复场景可能就比较多了，比如恢复某个库或某个表等，那么如何从全备中恢复单库或单表，这其中又有哪些隐藏的坑呢？

<!--more-->

#### 如何恢复单库或单表

每个数据库实例中都不止一个库，一般备份都是备份整个实例，但恢复需求又是多种多样的，比如说我想只恢复某个库或某张表，这个时候应该怎么操作呢？

如果你的实例数据量不大，可以在另外一个环境恢复出整个实例，然后再单独备份出所需库或表用来恢复。不过这种方法不够灵活，并且只适用数据量比较少的情况。

其实从全备中恢复单库还是比较方便的，有个 `--one-database` 参数可以指定单库恢复，下面来具体演示下：

```sql
# 查看及备份所有库
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sbtest             |
| sys                |
| testdb             |
| testdb2            |
+--------------------+

mysqldump -uroot -pxxxx  -R -E --single-transaction --all-databases > all_db.sql

# 删除testdb库 并进行单库恢复
mysql> drop database testdb;
Query OK, 36 rows affected (2.06 sec)

# 恢复前 testdb库不存在的话要手动新建
mysql -uroot -pxxxx --one-database testdb < all_db.sql
```

除了上述方法外，恢复单库或单表还可以采用手动筛选的方法。这个时候 Linux 下大名鼎鼎的 `sed` 和 `grep` 命令就派上用场了，我们可以利用这两个命令从全备中筛选出单库或单表的语句，筛选方法如下：

```sql
# 从全备中恢复单库
sed -n '/^-- Current Database: `testdb`/,/^-- Current Database: `/p' all_db.sql > testdb.sql

# 筛选出单表语句
cat all_db.sql | sed -e '/./{H;$!d;}' -e 'x;/CREATE TABLE `test_tb`/!d;q' > /tmp/test_tb_info.sql 
cat all_db.sql | grep --ignore-case  'insert into `test_tb`' > /tmp/test_tb_data.sql
```

#### 小心有坑

对于上述手动筛选来恢复单库或单表的方法，看起来简单方便，其实隐藏着一个小坑，下面我们来具体演示下：

```sql
# 备份整个实例
mysqldump -uroot -pxxxx  -R -E --single-transaction --all-databases > all_db.sql

# 手动备份下test_tb 然后删除test_tb
mysql> create table test_tb_bak like test_tb;
Query OK, 0 rows affected (0.03 sec)

mysql> insert into test_tb_bak select * from test_tb;
Query OK, 4 rows affected (0.02 sec)
Records: 4  Duplicates: 0  Warnings: 0

mysql> drop table test_tb;
Query OK, 0 rows affected (0.02 sec)

# 从全备中筛选test_db建表及插数据语句
cat all_db.sql | sed -e '/./{H;$!d;}' -e 'x;/CREATE TABLE `test_tb`/!d;q' > test_tb_info.sql 
cat all_db.sql | grep --ignore-case  'insert into `test_tb`' > test_tb_data.sql

# 查看得到的语句 貌似没问题
cat test_tb_info.sql

DROP TABLE IF EXISTS `test_tb`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `test_tb` (
  `inc_id` int(11) NOT NULL AUTO_INCREMENT COMMENT '自增主键',
  `col1` int(11) NOT NULL,
  `col2` varchar(20) DEFAULT NULL,
  `col_dt` datetime DEFAULT NULL,
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`inc_id`)
) ENGINE=InnoDB AUTO_INCREMENT=5 DEFAULT CHARSET=utf8 COMMENT='测试表';
/*!40101 SET character_set_client = @saved_cs_client */;

cat test_tb_data.sql

INSERT INTO `test_tb` VALUES (1,1001,'dsfs','2020-08-04 12:12:36','2020-09-17 06:19:27','2020-09-17 06:19:27'),
(2,1002,'vfsfs','2020-09-04 12:12:36','2020-09-17 06:19:27','2020-09-17 06:19:27'),
(3,1003,'adsfsf',NULL,'2020-09-17 06:19:27','2020-09-17 06:19:27'),
(4,1004,'walfd','2020-09-17 14:19:27','2020-09-17 06:19:27','2020-09-18 07:52:13');

# 执行恢复单表操作
mysql -uroot -pxxxx testdb < test_tb_info.sql
mysql -uroot -pxxxx testdb < test_tb_data.sql

# 查看恢复数据 并和备份表比对
mysql> select * from test_tb;
+--------+------+--------+---------------------+---------------------+---------------------+
| inc_id | col1 | col2   | col_dt              | create_time         | update_time         |
+--------+------+--------+---------------------+---------------------+---------------------+
|      1 | 1001 | dsfs   | 2020-08-04 12:12:36 | 2020-09-17 06:19:27 | 2020-09-17 06:19:27 |
|      2 | 1002 | vfsfs  | 2020-09-04 12:12:36 | 2020-09-17 06:19:27 | 2020-09-17 06:19:27 |
|      3 | 1003 | adsfsf | NULL                | 2020-09-17 06:19:27 | 2020-09-17 06:19:27 |
|      4 | 1004 | walfd  | 2020-09-17 14:19:27 | 2020-09-17 06:19:27 | 2020-09-18 07:52:13 |
+--------+------+--------+---------------------+---------------------+---------------------+
4 rows in set (0.00 sec)

mysql> select * from test_tb_bak;
+--------+------+--------+---------------------+---------------------+---------------------+
| inc_id | col1 | col2   | col_dt              | create_time         | update_time         |
+--------+------+--------+---------------------+---------------------+---------------------+
|      1 | 1001 | dsfs   | 2020-08-04 12:12:36 | 2020-09-17 14:19:27 | 2020-09-17 14:19:27 |
|      2 | 1002 | vfsfs  | 2020-09-04 12:12:36 | 2020-09-17 14:19:27 | 2020-09-17 14:19:27 |
|      3 | 1003 | adsfsf | NULL                | 2020-09-17 14:19:27 | 2020-09-17 14:19:27 |
|      4 | 1004 | walfd  | 2020-09-17 14:19:27 | 2020-09-17 14:19:27 | 2020-09-18 15:52:13 |
+--------+------+--------+---------------------+---------------------+---------------------+
4 rows in set (0.00 sec)
```

如果你仔细观察的话，会发现恢复出来的数据有问题，貌似时间不太对，你再仔细看看，是不是有的时间差了8小时！详细探究下来，我们发现 timestamp 类型字段的时间数据恢复有问题，准确来讲备份文件中记录的是0时区，而我们系统一般采用东八区，所以才会出现误差8小时的问题。

那么你会问了，为什么全部恢复不会出问题呢？问的好，我们看下备份文件就知道了。

```sql
# 备份文件开头
-- MySQL dump 10.13  Distrib 5.7.23, for Linux (x86_64)
--
-- Host: localhost    Database:
-- ------------------------------------------------------
-- Server version       5.7.23-log

/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!40101 SET NAMES utf8 */;
/*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
/*!40103 SET TIME_ZONE='+00:00' */;  
注意上面两行
/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;


# 备份文件结尾
/*!40103 SET TIME_ZONE=@OLD_TIME_ZONE */;
/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
/*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;

-- Dump completed on 2020-09-18 15:56:40
```

仔细看备份文件，你会发现 mysqldump 备份出来的文件中，首先会将会话时区改为0，结尾处再改回原时区。这就代表着，备份文件中记录的时间戳数据都是以0时区为基础的。如果直接执行筛选出的SQL，就会造成0时区的时间戳插入的东八区的系统中，显然会造成时间相差8小时的问题。

看到这里，不知道你是否看懂了呢，可能有过备份恢复经验的同学好理解些。解决上述问题的方法也很简单，那就是在执行SQL文件前，更改当前会话时区为0，再次来演示下：

```sql
# 清空test_db表数据
mysql> truncate table test_tb;
Query OK, 0 rows affected (0.02 sec)

# 文件开头增加时区声明
vim test_tb_data.sql
set session TIME_ZONE='+00:00';
INSERT INTO `test_tb` VALUES (1,1001,'dsfs','2020-08-04 12:12:36','2020-09-17 06:19:27','2020-09-17 06:19:27'),
(2,1002,'vfsfs','2020-09-04 12:12:36','2020-09-17 06:19:27','2020-09-17 06:19:27'),
(3,1003,'adsfsf',NULL,'2020-09-17 06:19:27','2020-09-17 06:19:27'),
(4,1004,'walfd','2020-09-17 14:19:27','2020-09-17 06:19:27','2020-09-18 07:52:13');

# 执行恢复并比对 发现数据正确
mysql> select * from test_tb;
+--------+------+--------+---------------------+---------------------+---------------------+
| inc_id | col1 | col2   | col_dt              | create_time         | update_time         |
+--------+------+--------+---------------------+---------------------+---------------------+
|      1 | 1001 | dsfs   | 2020-08-04 12:12:36 | 2020-09-17 14:19:27 | 2020-09-17 14:19:27 |
|      2 | 1002 | vfsfs  | 2020-09-04 12:12:36 | 2020-09-17 14:19:27 | 2020-09-17 14:19:27 |
|      3 | 1003 | adsfsf | NULL                | 2020-09-17 14:19:27 | 2020-09-17 14:19:27 |
|      4 | 1004 | walfd  | 2020-09-17 14:19:27 | 2020-09-17 14:19:27 | 2020-09-18 15:52:13 |
+--------+------+--------+---------------------+---------------------+---------------------+
4 rows in set (0.00 sec)

mysql> select * from test_tb_bak;
+--------+------+--------+---------------------+---------------------+---------------------+
| inc_id | col1 | col2   | col_dt              | create_time         | update_time         |
+--------+------+--------+---------------------+---------------------+---------------------+
|      1 | 1001 | dsfs   | 2020-08-04 12:12:36 | 2020-09-17 14:19:27 | 2020-09-17 14:19:27 |
|      2 | 1002 | vfsfs  | 2020-09-04 12:12:36 | 2020-09-17 14:19:27 | 2020-09-17 14:19:27 |
|      3 | 1003 | adsfsf | NULL                | 2020-09-17 14:19:27 | 2020-09-17 14:19:27 |
|      4 | 1004 | walfd  | 2020-09-17 14:19:27 | 2020-09-17 14:19:27 | 2020-09-18 15:52:13 |
+--------+------+--------+---------------------+---------------------+---------------------+
4 rows in set (0.00 sec)
```

**总结：**

我们在网络中很容易搜索出恢复单库或单表的方法，大多都有提到上述利用 sed 、grep 命令来手动筛选的方法。但大部分文章都未提及可能出现的问题，如果你的表字段有timestamp 类型，用这种方法要格外注意。无论面对哪种恢复需求，我们都要格外小心，不要造成越恢复越糟糕的情况，最好有个空实例演练下，然后再进行恢复。