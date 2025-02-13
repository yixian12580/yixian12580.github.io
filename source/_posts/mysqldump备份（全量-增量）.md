---
title: mysqldump备份（全量+增量）
categories:
  - 技术
  - 数据库
tags: MySQL
abbrlink: 378b45cf
date: 2024-12-28 10:58:41
---

在日常运维工作中，对mysql数据库的备份是万分重要的，以防在数据库表丢失或损坏情况出现，可以及时恢复数据。

<!--more-->

**线上数据库备份场景：**
每周日执行一次全量备份，然后每天下午1点执行MySQLdump增量备份.

下面对这种备份方案详细说明下：

### MySQLdump增量备份配置

执行增量备份的前提条件是MySQL打开binlog日志功能，在my.cnf中加入

```
log-bin=/opt/Data/MySQL-bin
```

“log-bin=”后的字符串为日志记载目录，一般建议放在不同于MySQL数据目录的磁盘上。

```
-----------------------------------------------------------------------------------
mysqldump >       导出数据
mysql <           导入数据  （或者使用source命令导入数据，导入前要先切换到对应库下）
 
注意一个细节：
若是mysqldump导出一个库的数据，导出文件为a.sql，然后mysql导入这个数据到新的空库下。
如果新库名和老库名不一致，那么需要将a.sql文件里的老库名改为新库名，
这样才能顺利使用mysql命令导入数据（如果使用source命令导入就不需要修改a.sql文件了）。
-----------------------------------------------------------------------------------
```

### MySQLdump增量备份

假定星期日下午1点执行全量备份，适用于MyISAM存储引擎。

```
mysqldump --lock-all-tables --flush-logs --master-data=2 -u root -p test > backup_sunday_1_PM.sql
```

**对于InnoDB将--lock-all-tables替换为--single-transaction**
--flush-logs为结束当前日志，生成新日志文件；
--master-data=2 选项将会在输出SQL中记录下完全备份后新日志文件的名称

用于日后恢复时参考，例如输出的备份SQL文件中含有：

```
CHANGE MASTER TO MASTER_LOG_FILE=’MySQL-bin.000002′, MASTER_LOG_POS=106;
```

### MySQLdump增量备份其他说明：

如果MySQLdump加上–delete-master-logs 则清除以前的日志，以释放空间。但是如果服务器配置为镜像的复制主服务器，用MySQLdump –delete-master-logs删掉MySQL二进制日志很危险，因为从服务器可能还没有完全处理该二进制日志的内容，在这种情况下，使用 PURGE MASTER LOGS更为安全。

每日定时使用 MySQLadmin flush-logs来创建新日志，并结束前一日志写入过程。并把前一日志备份，例如上例中开始保存数据目录下的日志文件 MySQL-bin.000002 , ...

#### 恢复完全备份

```
mysql -u root -p < backup_sunday_1_PM.sql
```



#### 恢复增量备份

```
mysqlbinlog MySQL-bin.000002 … | mysql -u root -p
```

注意此次恢复过程亦会写入日志文件，如果数据量很大，建议先关闭日志功能

参数解释：

```
--compatible=name
它告诉 MySQLdump，导出的数据将和哪种数据库或哪个旧版本的 MySQL 服务器相兼容。值可以为 ansi、MySQL323、MySQL40、postgresql、oracle、mssql、db2、maxdb、no_key_options、no_tables_options、no_field_options 等，要使用几个值，用逗号将它们隔开。当然了，它并不保证能完全兼容，而是尽量兼容。

--complete-insert，-c
导出的数据采用包含字段名的完整 INSERT 方式，也就是把所有的值都写在一行。这么做能提高插入效率，但是可能会受到 max_allowed_packet 参数的影响而导致插入失败。因此，需要谨慎使用该参数，至少我不推荐。

--default-character-set=charset
指定导出数据时采用何种字符集，如果数据表不是采用默认的 latin1 字符集的话，那么导出时必须指定该选项，否则再次导入数据后将产生乱码问题。

--disable-keys
告诉 MySQLdump 在 INSERT 语句的开头和结尾增加 /*!40000 ALTER TABLE table DISABLE KEYS */; 和 /*!40000 ALTER TABLE table ENABLE KEYS */; 语句，这能大大提高插入语句的速度，因为它是在插入完所有数据后才重建索引的。该选项只适合 MyISAM 表。

--extended-insert = true|false
默认情况下，MySQLdump 开启 --complete-insert 模式，因此不想用它的的话，就使用本选项，设定它的值为 false 即可。

--hex-blob
使用十六进制格式导出二进制字符串字段。如果有二进制数据就必须使用本选项。影响到的字段类型有 BINARY、VARBINARY、BLOB。

--lock-all-tables，-x
在开始导出之前，提交请求锁定所有数据库中的所有表，以保证数据的一致性。这是一个全局读锁，并且自动关闭 --single-transaction 和 --lock-tables 选项。

--lock-tables
它和 --lock-all-tables 类似，不过是锁定当前导出的数据表，而不是一下子锁定全部库下的表。本选项只适用于 MyISAM 表，如果是 Innodb 表可以用 --single-transaction 选项。

--no-create-info，-t
只导出数据，而不添加 CREATE TABLE 语句。

--no-data，-d
不导出任何数据，只导出数据库表结构。
mysqldump --no-data --databases mydatabase1 mydatabase2 mydatabase3 > test.dump
将只备份表结构。--databases指示主机上要备份的数据库。

--opt
这只是一个快捷选项，等同于同时添加 --add-drop-tables --add-locking --create-option --disable-keys --extended-insert --lock-tables --quick --set-charset 选项。本选项能让 MySQLdump 很快的导出数据，并且导出的数据能很快导回。该选项默认开启，但可以用 --skip-opt 禁用。注意，如果运行 MySQLdump 没有指定 --quick 或 --opt 选项，则会将整个结果集放在内存中。如果导出大数据库的话可能会出现问题。

--quick，-q
该选项在导出大表时很有用，它强制 MySQLdump 从服务器查询取得记录直接输出而不是取得所有记录后将它们缓存到内存中。

--routines，-R
导出存储过程以及自定义函数。

--single-transaction
该选项在导出数据之前提交一个 BEGIN SQL语句，BEGIN 不会阻塞任何应用程序且能保证导出时数据库的一致性状态。它只适用于事务表，例如 InnoDB 和 BDB。本选项和 --lock-tables 选项是互斥的，因为 LOCK TABLES 会使任何挂起的事务隐含提交。要想导出大表的话，应结合使用 --quick 选项。

--triggers
同时导出触发器。该选项默认启用，用 --skip-triggers 禁用它。
```

**跨主机备份**
使用下面的命令可以将host1上的sourceDb复制到host2的targetDb，前提是host2主机上已经创建targetDb数据库：

```
mysqldump --host=host1 --opt sourceDb| mysql --host=host2 -C targetDb
```

-C 指示主机间的数据传输使用数据压缩

**结合Linux的cron命令实现定时备份**
比如需要在每天凌晨1:30备份某个主机上的所有数据库并压缩dump文件为gz格式：

```
30 1 * * * mysqldump -u root -pPASSWORD --all-databases | gzip > /mnt/disk2/database_`date '+%m-%d-%Y'`.sql.gz
```

一个完整的Shell脚本备份MySQL数据库示例。

比如备份数据库opspc：

```
[root@loacalhost ~]# vim /root/backup.sh
#!bin/bash
echo "Begin backup mysql database"
mysqldump -u root -ppassword opspc > /home/backup/mysqlbackup-`date +%Y-%m-%d`.sql
echo "Your database backup successfully completed"

[root@loacalhost ~]# crontab -e
30 1 * * * /bin/bash -x /root/backup.sh > /dev/null 2>&1
```



**mysqldump全量备份+mysqlbinlog二进制日志增量备份**
1）从mysqldump备份文件恢复数据会丢失掉从备份点开始的更新数据，所以还需要结合mysqlbinlog二进制日志增量备份。
首先确保已开启binlog日志功能。在my.cnf中包含下面的配置以启用二进制日志：

```
[mysqld]
log-bin=mysql-bin
```

2）mysqldump命令必须带上--flush-logs选项以生成新的二进制日志文件：

```
mysqldump --single-transaction --flush-logs --master-data=2 > backup.sql
```

其中参数--master-data=[0|1|2]
0: 不记录
1：记录为CHANGE MASTER语句
2：记录为注释的CHANGE MASTER语句



mysqldump全量+增量备份方案的具体操作可参考下面两篇文档：
[数据库误删除后的数据恢复操作说明](http://www.cnblogs.com/kevingrace/p/5904800.html)
[解说mysql之binlog日志以及利用binlog日志恢复数据](http://www.cnblogs.com/kevingrace/p/5907254.html)

\-----------------------------------------------------------------------------------------------------------------------------------------------------------

### shell脚本

下面分享一下自己用过的mysqldump全量和增量备份脚本

#### 应用场景

1）增量备份在周一到周六凌晨3点，会复制mysql-bin.00000*到指定目录；
2）全量备份则使用mysqldump将所有的数据库导出，每周日凌晨3点执行，并会删除上周留下的mysq-bin.00000*，然后对mysql的备份操作会保留在bak.log文件中。

#### 脚本实现：

##### 全量备份脚本

（假设mysql登录密码为123456；注意脚本中的命令路径）：

```
vim /root/Mysql-FullyBak.sh
```

```
#!/bin/bash
# Program
# use mysqldump to Fully backup mysql data per week!
# History
# Path
BakDir=/home/mysql/backup
LogFile=/home/mysql/backup/bak.log
Date=`date +%Y%m%d`
Begin=`date +"%Y年%m月%d日 %H:%M:%S"`
cd $BakDir
DumpFile=$Date.sql
GZDumpFile=$Date.sql.tgz
/usr/local/mysql/bin/mysqldump -uroot -p123456 --quick --events --all-databases --flush-logs --delete-master-logs --single-transaction > $DumpFile
/bin/tar -zvcf $GZDumpFile $DumpFile
/bin/rm $DumpFile
Last=`date +"%Y年%m月%d日 %H:%M:%S"`
echo 开始:$Begin 结束:$Last $GZDumpFile succ >> $LogFile
cd $BakDir/daily
/bin/rm -f ./*
```

##### 增量备份脚本

（脚本中mysql的数据存放路径是/home/mysql/data，具体根据自己的实际情况进行调整）

```
vim /root/Mysql-DailyBak.sh
```

```
#!/bin/bash
# Program
# use cp to backup mysql data everyday!
# History
# Path
BakDir=/home/mysql/backup/daily           #增量备份时复制mysql-bin.00000*的目标目录，提前手动创建这个目录
BinDir=/home/mysql/data                  #mysql的数据目录
LogFile=/home/mysql/backup/bak.log
BinFile=/home/mysql/data/mysql-bin.index      #mysql的index文件路径，放在数据目录下的
/usr/local/mysql/bin/mysqladmin -uroot -p123456 flush-logs  #这个是用于产生新的mysql-bin.00000*文件
Counter=`wc -l $BinFile |awk '{print $1}'`
NextNum=0

#这个for循环用于比对$Counter,$NextNum这两个值来确定文件是不是存在或最新的
for file in `cat $BinFile`
do
  base=`basename $file`
  #basename用于截取mysql-bin.00000*文件名，去掉./mysql-bin.000005前面的./
  NextNum=`expr $NextNum + 1`
  if [ $NextNum -eq $Counter ]
  then
    echo $base skip! >> $LogFile
  else
    dest=$BakDir/$base
    if(test -e $dest)
    #test -e用于检测目标文件是否存在，存在就写exist!到$LogFile去
    then
      echo $base exist! >> $LogFile
    else
      cp $BinDir/$base $BakDir
      echo $base copying >> $LogFile
     fi
   fi
done
echo `date +"%Y年%m月%d日 %H:%M:%S"` $Next Bakup succ! >> $LogFile
```

##### 设置crontab任务

执行备份脚本，先执行的是增量备份脚本，然后执行的是全量备份脚本：

```
[root@localhost ~]# crontab -e
0 3 * * 0 /bin/bash -x /root/Mysql-FullyBak.sh >/dev/null 2>&1   #每个星期日凌晨3:00执行完全备份脚本
0 3 * * 1-6 /bin/bash -x /root/Mysql-DailyBak.sh >/dev/null 2>&1    #周一到周六凌晨3:00做增量备份
```



手动执行上面两个脚本，测试下备份效果

```
[root@localhost backup]# pwd
/home/mysql/backup
[root@localhost backup]# mkdir daily
[root@localhost backup]# ll
total 4
drwxr-xr-x. 2 root root 4096 Nov 29 11:29 daily
[root@localhost backup]# ll daily/
total 0
```

先执行增量备份脚本

```
[root@localhost backup]# sh /root/Mysql-DailyBak.sh
[root@localhost backup]# ll
total 8
-rw-r--r--. 1 root root 121 Nov 29 11:29 bak.log
drwxr-xr-x. 2 root root 4096 Nov 29 11:29 daily
[root@localhost backup]# ll daily/
total 8
-rw-r-----. 1 root root 152 Nov 29 11:29 mysql-binlog.000030
-rw-r-----. 1 root root 152 Nov 29 11:29 mysql-binlog.000031
[root@localhost backup]# cat bak.log
mysql-binlog.000030 copying
mysql-binlog.000031 copying
mysql-binlog.000032 skip!
2016年11月29日 11:29:32 Bakup succ!
```

然后执行全量备份脚本

```
[root@localhost backup]# sh /root/Mysql-FullyBak.sh
20161129.sql
[root@localhost backup]# ll
total 152
-rw-r--r--. 1 root root 145742 Nov 29 11:30 20161129.sql.tgz
-rw-r--r--. 1 root root 211 Nov 29 11:30 bak.log
drwxr-xr-x. 2 root root 4096 Nov 29 11:30 daily
[root@localhost backup]# ll daily/
total 0
[root@localhost backup]# cat bak.log
mysql-binlog.000030 copying
mysql-binlog.000031 copying
mysql-binlog.000032 skip!
2016年11月29日 11:29:32 Bakup succ!
开始:2016年11月29日 11:30:38 结束:2016年11月29日 11:30:38 20161129.sql.tgz succ
```

