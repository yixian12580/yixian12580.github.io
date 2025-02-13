---
title: MySQL删除所有表
categories:
  - 技术
  - 数据库
tags: MySQL
abbrlink: 5a754aee
date: 2024-12-16 10:43:55
---

当我们需要在MySQL数据库中删除所有已经存在的表时，我们可以通过执行多条drop语句的方式逐个删除，但是这样过于繁琐，没有效率，而且容易出错。

<!--more-->

以下介绍一次性删除数据库中的所有表的方法。

### 方案一：使用mysqldump命令

`mysqldump`是一个备份工具，也可以用来删除数据库中的所有表。在命令行输入如下命令即可：

```mysql
mysqldump -u [用户名] -p --add-drop-table --no-data [数据库名] | grep ^DROP | mysql -u [用户名] -p [数据库名]
```

其中`[用户名]`指的是你在MySQL中的用户名，`[数据库名]`指的是需要删除表格的数据库名称。它执行的步骤是先使用`mysqldump`备份数据库，然后从备份文件中查找`DROP`语句并执行。

下面是一个示例：

```mysql
$ mysqldump -u root -p --add-drop-table --no-data mydb | grep ^DROP | mysql -u root -p mydb
```

执行时需要输入你的MySQL账户密码。此方法适用于需要备份数据库的情况。



### 方案二：利用MySQL的元数据

MySQL的元数据中记录着所有的表名称，我们可以通过查询并删除所有表的方式来实现一次性删除。在命令行输入如下语句即可：

```mysql
SELECT concat('DROP TABLE IF EXISTS `', table_schema, '`.`', table_name, '`;')
FROM information_schema.tables
WHERE table_schema = '[数据库名]';
```

其中`[数据库名]`指的是需要删除表格的数据库名称。它执行的步骤是使用`SELECT`语句查询`information_schema`表中的`tables`表，构建并输出每个表格的`DROP TABLE`语句，然后执行这些语句。

下面是一个示例：

```mysql
mysql> SELECT concat('DROP TABLE IF EXISTS `', table_schema, '`.`', table_name, '`;')
    -> FROM information_schema.tables
    -> WHERE table_schema = 'mydb';
+--------------------------------------------------------------+
| concat('DROP TABLE IF EXISTS `', table_schema, '`.`', table_name, '`;') |
+--------------------------------------------------------------+
| DROP TABLE IF EXISTS `mydb`.`table1`;                       |
| DROP TABLE IF EXISTS `mydb`.`table2`;                       |
| DROP TABLE IF EXISTS `mydb`.`table3`;                       |
+--------------------------------------------------------------+
3 rows in set (0.00 sec)
```

执行查询返回的结果即可删除所有表格。此方法适用于非备份的情况。

```
DROP TABLE IF EXISTS `mydb`.`table1`; 
DROP TABLE IF EXISTS `mydb`.`table2`; 
DROP TABLE IF EXISTS `mydb`.`table3`;  
```



### 总结

本文介绍了两种可以一次性删除MySQL数据库中所有表格的方法。使用`mysqldump`命令可以备份并删除数据库，而利用MySQL的元数据可以查询并删除所有表格。



题外话：

查看数据库大小sql：

```
SELECT table_schema "Database Name", sum( data_length + index_length ) / 1024 / 1024 / 1024"Database Size in GB" FROM information_schema.TABLES GROUP BY table_schema;
```



