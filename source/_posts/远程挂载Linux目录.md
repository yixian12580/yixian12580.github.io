---
title: 远程挂载Linux目录
categories: 技术
tags:
  - Linux
  - 挂载
abbrlink: a4e0a084
date: 2022-11-29 17:43:14
---

Linux现有比较成熟的解决方案有两种，一种是NFS远程挂载，另一种是Samba共享目录。下面就这两方式的配置进行说明。

<!--more-->

环境：机器A：192.168.1.1 共享目录 /data/share 机器B：192.168.1.2 关联目录 /data/store

大致逻辑是这样：将A机器的目录/data/share设置为共享目录，机器B通过mount的方式和A机器的共享文件夹进行连接。

## 一、NFS远程挂载

### 1、概念

NFS：即网络文件系统（Network File System）分布式文件系统协议

### 2、操作步骤

**[机器A]**

- **安装NFS**

```shell
#由于NFS是依赖于RPC协议来进行的协议传输，所以，此时需同时安装，NFS 和 RPC 两个应用程序
#安装NFS和RPC（安装nfs-utils，rpcbind） 
yum -y install nfs-utils rpcbind
```

- **设置共享目录**

NFS的配置文件在/etc/exports，内容默认为空。配置格式为

目录位置 客户机地址(权限选项)

```text
vim /etc/exports
/data/share 192.168.1.2(rw,sync,no_root_squash)
 
#客户机地址 可以是 ：  主机名、IP地址、网段地址、或者"*、?"通配符；
#权限选项：rw表示允许读写(ro为只读)
#　　　　　sync表示同步写
#　　　　  no_root_squash表示当前客户机以root身份访问时，赋予本地root权限(默认是root_squash,将作为nfsnobody用户降权对待)  （NFS 服务器共享目录用户的属性，如果用户是 root，那么对于这个共享目录来说就具有 root 的权限。）
 
#给多个地址授权
/data/share 192.168.1.2(rw,sync,no_root_squash)  192.168.1.3(rw,sync,no_root_squash)
#给某个网段内所有IP授权
/data/share 192.168.1.*(rw,sync,no_root_squash)
```

- **启动NFS服务**

配置完上述的目录文件配置后，则启动NFS服务；先启动 RPC服务，再启动 NFS 服务

```text
#启动rpc服务
systemctl start rpcbind
#启动nfs服务
systemctl start nfs
#查看rpc服务状态
systemctl status rpcbind
#查看nfs服务状态
systemctl status nfs
#查看对应进程信息
ps -ef | grep rpcbind 
ps -ef | grep nfs
```

- **查看当前机器已经发布的NFS共享目录**

```text
showmount -e 192.168.1.1
显示
Export list for 192.168.1.1:
/data/share 192.168.1.2
```

此时共享机器A的配置已经完成，可直接在机器B进行目录的挂载操作

**[机器B]**

- **安装RPC服务**

目录的挂载于共享是基于RPC协议进行的，所以B服务器作为挂载方，也应同时具备RPC的应用功能，所以也应同时安装对应的 rpcbind 服务插件。（安装rpcbind时，最好也可以直接把 nfs-utils 同步安装下，后续再次作为共享方时，则也会方便很多）

```text
yum -y install rpcbind nfs-utils
```

- **挂载**

```text
使用mount命令，此处表示将IP为：192.168.1.1所共享的/data/share目录，挂载到当前服务的 /data/store 目录下
 
mount -t nfs 192.168.1.1:/data/share /data/store
```

- **开机自动挂载**

```text
vim /etc/fstab

192.168.1.1:/data/share    /data/store    nfs    defaults,_netdev 0 0
```

- **开机自动启动**

```text
systemctl enable rpcbind.service
systemctl enable nfs-server.service
```

- **查看当前机器挂载点**

```text
df -h
```

## **二、Samba共享目录**

### **1、概念**

和NFS不同的是，NFS解决的是UNIX/Linux之间的资源共享，而Samba除了Linux之间，也可用于Windows和Linux之间的资源共享，所以配置相对于NFS来说麻烦一些。

### 2、操作步骤

【服务端】

Linux作为服务端安装Samba

- 安装samba程序包

```text
#安装samba的主程序包、共享程序包、Linux作为客户端访问windows的客户端包
yum -y intall samba samba-common samba-client 

systemctl status firewalld #查看防火墙状态
setenforce 0 # 关闭SELinux 1启用 0 告警不启用
getenforce # 查看Selinux状态 disabled 关闭 enforcing 打开
```

- 配置共享目录

```text
vim /etc/samba/smb.conf

[sambdir]
    public = yes
    path = /data/samba
    writable = yes
```

- 启动Samba服务

```text
systemctl restart smb
systemctl enable smb 
systemctl restart nmb
systemctl enable nmb
```

【客户端】

- 安装Samba客户端包

```text
yum -y intall samba samba-common samba-client
```

- 目录挂载

```text
mount -t cifs -o "rw,dir_mode=0644,file_mode=0644,username=username,password=yourpassword" //192.168.1.100/yourshare_folder_name /usr/local/your_server_folder
```

