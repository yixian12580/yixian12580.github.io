---
title: Linux挂载硬盘错误
categories:
  - 技术
  - 分区
tags: 挂载
abbrlink: e3336f7b
date: 2022-09-27 16:06:09
---

Linux挂载U盘时，报错mount: unknown filesystem type 'ntfs' 错误。这是由于Linux上无法识别NTFS格式的分区的原因。

<!--more-->

解决办法：安装ntfs-3g

```
wget https://tuxera.com/opensource/ntfs-3g_ntfsprogs-2017.3.23.tgz
tar -zxvf  ntfs-3g_ntfsprogs-2017.3.23.tgz
cd ntfs-3g_ntfsprogs-2017.3.23.tgz
./configure
make && make install
```


执行挂载命令，挂载U盘：

```
mount -t ntfs-3g /dev/sdb1 /data
```

至此，就解决了Linux系统无法识别ntfs格式的问题。
