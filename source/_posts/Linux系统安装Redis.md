---
title: Linux系统安装Redis
date: 2022-09-27 16:28:08
categories: 
- 技术
- 数据库
tags: Redis
---

REmote DIctionary Server(Redis) 是一个由 Salvatore Sanfilippo 写的 key-value 存储系统，是跨平台的非关系型数据库。

Redis 是一个开源的使用 ANSI C 语言编写、遵守 BSD 协议、支持网络、可基于内存、分布式、可选持久性的键值对(Key-Value)存储数据库，并提供多种语言的 API。

Redis 通常被称为数据结构服务器，因为值（value）可以是字符串(String)、哈希(Hash)、列表(list)、集合(sets)和有序集合(sorted sets)等类型。

<!--more-->

1.获取redis资源

```
wget http://download.redis.io/releases/redis-4.0.8.tar.gz
```

2.解压

```
tar xzvf redis-4.0.8.tar.gz
```

3.编译与安装

```
cd redis-4.0.8
make
cd src
make install PREFIX=/usr/local/redis
```

4.移动配置文件到安装目录下

```
cd ../
mkdir /usr/local/redis/etc
mv redis.conf /usr/local/redis/etc
```

 5.配置redis为后台启动

```
vi /usr/local/redis/etc/redis.conf 
//将daemonize no 改成daemonize yes
```

6.将redis加入到开机启动

```
vi /etc/rc.local
//在里面添加内容：/usr/local/redis/bin/redis-server /usr/local/redis/etc/redis.conf
```

7.开启redis

```
/usr/local/redis/bin/redis-server /usr/local/redis/etc/redis.conf 
```

8.将redis-cli,redis-server拷贝到bin下，让redis-cli指令可以在任意目录下直接使用

```
cp /usr/local/redis/bin/redis-server /usr/local/bin/
cp /usr/local/redis/bin/redis-cli /usr/local/bin/
```

9.设置redis密码

　　a.运行命令：

```
redis-cli
```

　　b.查看现有的redis密码命令(可选操作，可以没有)

```
config get requirepass 
```

如果没有设置过密码的话运行结果会如下

```
requirepass 
''
```

c.设置redis密码

运行命令：

```
config set requirepass ****
(****为你要设置的密码)，设置成功的话会返回‘OK’字样
```

d.测试连接

```
 redis-cli -h 127.0.0.1 -p 6379 -a ****
（****为你设置的密码）
 输入 redis-cli 进入命令模式，使用 auth '\*\*\*\*\*' 
（****为你设置的密码）登陆
```

redis-清理数据：

1）首先通过密码登陆redis

>```
>redis-cli
>auth 密码
>```

2）执行清理前查看(若不需要清理全部则清理指定key即可)

>```
>keys * //查看所有key值
>```

3）清理redis

>```
>del key //①删除指定key
>Flushdb //②删除当前数据库中的所有Key
>flushall //③删除所有数据库中的key
>```

10.让外网能够访问redis

a.配置防火墙: 

```
firewall-cmd --zone=public --add-port=6379/tcp --permanent（开放6379端口）
systemctl restart firewalld（重启防火墙以使配置即时生效）
```

查看系统所有开放的端口：

```
firewall-cmd --zone=public --list-ports
```

b.此时 虽然防火墙开放了6379端口，但是外网还是无法访问的，因为redis监听的是127.0.0.1：6379，并不监听外网的请求。

（一）把文件夹目录里的redis.conf配置文件里的bind 127.0.0.1前面加#注释掉

（二）命令：redis-cli连接到redis后，输入命令 config get  daemonize和config get  protected-mode 是不是都为no，如果不是，就用config set 配置名 属性 改为no。

常用命令

```
redis-server /usr/local/redis/etc/redis.conf //启动redis
pkill redis  //停止redis
```

卸载redis：

```
rm -rf /usr/local/redis //删除安装目录
rm -rf /usr/bin/redis-* //删除所有redis相关命令脚本
```

启动redis:

```
redis-server &
```

检测后台进程是否存在

```
ps -ef |grep redis
```

检测6379端口是否在监听

```
netstat -lntp | grep 6379
```


有时候会报异常

原因: Redis已经启动

解决: 关掉Redis,重启即可

```
redis-cli shutdown
redis-server
```


然后你就能看到Redis愉快的运行了.

使用redis-cli客户端检测连接是否正常

```
redis-cli
127.0.0.1:6379> keys *
(empty list or set)
127.0.0.1:6379> set key "hello world"
OK
127.0.0.1:6379> get key
"hello world"
```

停止redis:

使用客户端

```
redis-cli shutdown
```


因为Redis可以妥善处理SIGTERM信号，所以直接kill -9也是可以的

```
kill -9 PID
```

