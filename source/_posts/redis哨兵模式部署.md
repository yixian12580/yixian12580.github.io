---
title: redis哨兵模式部署
categories:
  - 技术
  - Redis
tags:
  - Redis
  - 哨兵模式
abbrlink: b68cbed0
date: 2023-10-21 11:57:38
---

上一篇文章讲解了Redis哨兵模式的原理，今天来实战一下，部署Redis哨兵高可用。

<!--more-->

### 下载

```bash
wget http://download.redis.io/releases/redis-5.0.14.tar.gz
```

### 解压文件

```bash
tar -zxvf redis-5.0.14.tar.gz -C /opt/
```

### 安装

```bash
cd redis-5.0.14/
make
mkdir -p /opt/redis
make PREFIX=/opt/redis install
```

### 拷贝配置文件

```bash
cd /opt/redis
mkdir /opt/redis/data
mkdir /opt/redis/logs
mkdir /opt/redis/conf

cp ../redis-5.0.14/*.conf ./conf/
```

### 三台机器配置redis.conf

修改以下项目

```bash
# 第一台机器（master）
bind 192.168.233.50
port 6379
pidfile /opt/redis/data/redis.pid
logfile /opt/redis/logs/redis.log
dir /opt/redis/data/
daemonize yes  #后台启动
protected-mode no
# replicaof 192.168.233.50 6379 # master不要配这个
replica-read-only yes # slave只读，不能写操作
requirepass redis # 给redis设置一个密码，连接时需要指定
masterauth redis # 和requirepass密码一样


# 第二台机器（slave）
bind 192.168.233.51
port 6379
pidfile /opt/redis/data/redis.pid
logfile /opt/redis/logs/redis.log
dir /opt/redis/data/
daemonize yes  #后台启动
protected-mode no
replicaof 192.168.233.50 6379 # 指定master
replica-read-only yes # slave只读，不能写操作
requirepass redis # 给redis设置一个密码，连接时需要指定
masterauth redis # 和requirepass密码一样

# 第三台机器（slave）
bind 192.168.233.52
port 6379
pidfile /opt/redis/data/redis.pid
logfile /opt/redis/logs/redis.log
dir /opt/redis/data/
daemonize yes  #后台启动
protected-mode no
replicaof 192.168.233.50 6379 # 指定master
replica-read-only yes # slave只读，不能写操作
requirepass redis # 给redis设置一个密码，连接时需要指定
masterauth redis # 和requirepass密码一样
```

### 三台机器配置sentinel.conf

```bash
bind 192.168.233.50
port 26379 #Sentinel使用端口
daemonize yes #守护线程启动(即后台启动)
# 保护模式关闭，这样其他服务起就可以访问此台redis
protected-mode no
pidfile /opt/redis/data/redis-sentinel.pid
logfile /opt/redis/logs/redis-sentinel.log
dir /opt/redis/data # 工作目录


#重要的来了
#sentinel monitor <master-name> <ip> <redis-port> <quorum>
#告诉sentinel去监听地址为ip:port的一个master,这里的master-name可以自定义,quorum是一个数字,指明当有多少个sentinel认为一个master失效时,master才算真正失效.需要注意的是master-ip 
sentinel monitor master1 192.168.233.50 6379 2

sentinel auth-pass master1 redis # master中redis的密码

#sentinel down-after-milliseconds <master-name> <milliseconds> 
#这个配置项指定需要多少时间无响应,一个master才会被这个sentinel主观地认为是不可用的.单位是毫秒,默认为30秒
sentinel down-after-milliseconds master1 30000

#sentinel parallel-syncs <master-name> <numslaves> 
#这个配置项指定了在发生failover主备切换时最多可以有多少个slave同时对新的master进行同步,这个数字越小,完成failover所需的时间就越长,但是如果这个数字越大,就意味着越 多的slave因为replication而不可用.可以通过将这个值设为1(默认就是1)来保证每次只有一个slave处于不能处理命令请求的状态
sentinel parallel-syncs master1 1

#sentinel failover-timeout <master-name> <milliseconds>
# failover过期时间，当failover开始后，在此时间内仍然没有触发任何failover操作，当前sentinel 将会认为此次failover失败,默认为3分钟,单位为毫秒
sentinel failover-timeout master1 180000

#是否拒绝从新配置通知脚本,默认拒绝(yes).
# sentinel deny-scripts-reconfig yes
```

### 启动reids和redis-sentinel

```bash
cd /opt/redis
bin/redis-server ../conf/redis.conf
bin/redis-sentinel ../conf/sentinel.conf

ps -ef|grep redis
root       5155      1  0 4月18 ?       00:02:20 bin/redis-server 192.168.233.52:6379
root      19703      1  0 10:52 ?        00:00:00 bin/redis-sentinel 192.168.233.50:26379 [sentinel]
root      19708  18564  0 10:52 pts/0    00:00:00 grep --color=auto redis

redis]# cat logs/redis-sentinel.log 
20202:X 19 Apr 2023 11:32:17.990 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
20202:X 19 Apr 2023 11:32:17.990 # Redis version=5.0.14, bits=64, commit=00000000, modified=0, pid=20202, just started
20202:X 19 Apr 2023 11:32:17.990 # Configuration loaded
20203:X 19 Apr 2023 11:32:17.994 * Increased maximum number of open files to 10032 (it was originally set to 1024).
20203:X 19 Apr 2023 11:32:17.996 * Running mode=sentinel, port=26379.
20203:X 19 Apr 2023 11:32:17.996 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
20203:X 19 Apr 2023 11:32:17.997 # Sentinel ID is b380ad31e39697f72ffaac428a82d3d6ac21062d
20203:X 19 Apr 2023 11:32:17.997 # +monitor master master1 192.168.233.50 6379 quorum 2
20203:X 19 Apr 2023 11:32:48.058 # +sdown master master1 192.168.233.50 6379
20203:X 19 Apr 2023 11:32:48.058 # +sdown slave 192.168.233.52:6379 192.168.233.52 6379 @ master1 192.168.233.50 6379
20203:X 19 Apr 2023 11:32:48.058 # +sdown slave 192.168.233.51:6379 192.168.233.51 6379 @ master1 192.168.233.50 6379
```

### 登录redis客户端查看主从关系

```bash
/opt/redis/bin/redis-cli -a redis -h hadoop01 -p 6379
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
hadoop01:6379> 
hadoop01:6379> 
hadoop01:6379>  info replication
# Replication
role:master
connected_slaves:2
slave0:ip=192.168.233.52,port=6379,state=online,offset=14,lag=0
slave1:ip=192.168.233.51,port=6379,state=online,offset=14,lag=0
master_replid:19cd44d6abe9c6a872863e713c25869e4a590998
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:14
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:14
```

### 测试master切换

```bash
# 1.正常启动后，hadoop01为master
/opt/redis/bin/redis-cli -a redis -h hadoop01 -p 6379
hadoop01:6379> info replication
# Replication
role:master
connected_slaves:2
slave0:ip=192.168.233.51,port=6379,state=online,offset=6654076,lag=0
slave1:ip=192.168.233.52,port=6379,state=online,offset=6653769,lag=0
master_replid:94a4846bcad96026bb82b9eca0f32d58be74a1cf
master_replid2:ea15cbfd6f165275295a66b24ecf6aacc49423ac
master_repl_offset:6654218
second_repl_offset:6620735
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:5605643
repl_backlog_histlen:1048576

# 2.master掉线后，hadoop03变为master
/opt/redis/bin/redis-cli -a redis -h hadoop03 -p 6379
hadoop03:6379> info replication
# Replication
role:master
connected_slaves:1
slave0:ip=192.168.233.51,port=6379,state=online,offset=6732533,lag=0
master_replid:81d768dfcdca5c707f3938b544705bd7d18e867f
master_replid2:94a4846bcad96026bb82b9eca0f32d58be74a1cf
master_repl_offset:6732675
second_repl_offset:6727797
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:6653770
repl_backlog_histlen:78906

# 2.查看sentinel日志可以看到master切换过程
tail -f logs/redis-sentinel.log 
20944:X 20 Apr 2023 17:09:08.644 # +sdown master master1 192.168.233.50 6379
20944:X 20 Apr 2023 17:09:08.672 # +new-epoch 4
20944:X 20 Apr 2023 17:09:08.673 # +vote-for-leader 5896ea07aa70daf08c9d84c040bcaafecf10b041 4
20944:X 20 Apr 2023 17:09:08.698 # +odown master master1 192.168.233.50 6379 #quorum 3/2
20944:X 20 Apr 2023 17:09:08.698 # Next failover delay: I will not start a failover before Thu Apr 20 17:15:09 2023
20944:X 20 Apr 2023 17:09:09.478 # +config-update-from sentinel 5896ea07aa70daf08c9d84c040bcaafecf10b041 192.168.233.50 26379 @ master1 192.168.233.50 6379
20944:X 20 Apr 2023 17:09:09.478 # +switch-master master1 192.168.233.50 6379 192.168.233.52 6379
20944:X 20 Apr 2023 17:09:09.478 * +slave slave 192.168.233.51:6379 192.168.233.51 6379 @ master1 192.168.233.52 6379
20944:X 20 Apr 2023 17:09:09.478 * +slave slave 192.168.233.50:6379 192.168.233.50 6379 @ master1 192.168.233.52 6379
20944:X 20 Apr 2023 17:09:39.544 # +sdown slave 192.168.233.50:6379 192.168.233.50 6379 @ master1 192.168.233.52 6379


# 3.再次启动hadoop01
/opt/redis/bin/redis-cli -a redis -h hadoop03 -p 6379
hadoop03:6379> info replication
# Replication
role:master
connected_slaves:2
slave0:ip=192.168.233.51,port=6379,state=online,offset=6773718,lag=0
slave1:ip=192.168.233.50,port=6379,state=online,offset=6773718,lag=0
master_replid:81d768dfcdca5c707f3938b544705bd7d18e867f
master_replid2:94a4846bcad96026bb82b9eca0f32d58be74a1cf
master_repl_offset:6773860
second_repl_offset:6727797
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:6653770
repl_backlog_histlen:120091
```
