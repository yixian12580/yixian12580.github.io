---
title: MySQL异常关机数据恢复
categories:
  - 技术
  - 数据库
tags: MySQL
abbrlink: 32c87208
date: 2022-09-27 21:47:12
---

MySQL因为停电异常关机，导致数据库无法启动成功，出现如下报错：

<!--more-->

```
2021-03-15T01:50:21.759534Z 0 [Note] /usr/sbin/mysqld (mysqld 5.7.20) starting as process 22291 …
2021-03-15T01:50:21.767474Z 0 [Note] InnoDB: PUNCH HOLE support available
2021-03-15T01:50:21.767508Z 0 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
2021-03-15T01:50:21.767521Z 0 [Note] InnoDB: Uses event mutexes
2021-03-15T01:50:21.767530Z 0 [Note] InnoDB: GCC builtin __sync_synchronize() is used for memory barrier
2021-03-15T01:50:21.767538Z 0 [Note] InnoDB: Compressed tables use zlib 1.2.3
2021-03-15T01:50:21.767549Z 0 [Note] InnoDB: Using Linux native AIO
2021-03-15T01:50:21.768478Z 0 [Note] InnoDB: Number of pools: 1
2021-03-15T01:50:21.768631Z 0 [Note] InnoDB: Using CPU crc32 instructions
2021-03-15T01:50:21.770450Z 0 [Note] InnoDB: Initializing buffer pool, total size = 2G, instances = 8, chunk size = 128M
2021-03-15T01:50:21.925994Z 0 [Note] InnoDB: Completed initialization of buffer pool
2021-03-15T01:50:21.949851Z 0 [Note] InnoDB: If the mysqld execution user is authorized, page cleaner thread priority can be changed. See the man page of setpriority().
2021-03-15T01:50:21.962020Z 0 [Note] InnoDB: Highest supported file format is Barracuda.
2021-03-15T01:50:21.977821Z 0 [Note] InnoDB: Log scan progressed past the checkpoint lsn 701872788860
2021-03-15T01:50:22.006445Z 0 [Note] InnoDB: Doing recovery: scanned up to log sequence number 701873297569
2021-03-15T01:50:22.008894Z 0 [Note] InnoDB: Database was not shutdown normally!
2021-03-15T01:50:22.008927Z 0 [Note] InnoDB: Starting crash recovery.
2021-03-15T01:50:22.027234Z 0 [ERROR] InnoDB: Trying to access page number 3841987584 in space 0, space name innodb_system, which is outside the tablespace bounds. Byte offset 0, len 16384, i/o type read. If you get this error at mysqld startup, please check that your my.cnf matches the ibdata files that you have in the MySQL server.
2021-03-15T01:50:22.027297Z 0 [ERROR] InnoDB: Server exits.
2021-03-15T01:56:15.606226Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use –explicit_defaults_for_timestamp server option (see documentation for more details).
2021-03-15T01:56:15.606344Z 0 [Warning] ‘NO_ZERO_DATE’, ‘NO_ZERO_IN_DATE’ and ‘ERROR_FOR_DIVISION_BY_ZERO’ sql modes should be used with strict mode. They will be merged with strict mode in a future release.
2021-03-15T01:56:15.606352Z 0 [Warning] ‘NO_AUTO_CREATE_USER’ sql mode was not set.
```

解决方法：

1. 修改/etc/my.cnf文件，添加innodb_force_recovery=6；
2. 重启MySQL服务，能够顺利启动MySQL服务；
3. 使用mysqldump将数据导出到他处；
4. 删除原来的数据文件/var/lib/mysql，并重新初始化数据库；
5. 将mysqldump导出的文件再导入回来。

注意：在执行上述操作之前，记得先备份一下MySQL的数据文件，以免出现innodb_force_recovery=6都无法启动MySQL的情况。

经验：虽然使用innodb_force_recovery可以进行innodb数据恢复，但是还是有可能丢失数据，所以如果是关键数据库，建议还是定期进行备份比较稳妥，备份脚本可查看[此文章](https://yixian12580.github.io/2022/094f3e6852.html#MySQL%E5%A4%87%E4%BB%BD)。

