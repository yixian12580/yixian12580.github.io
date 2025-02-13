---
title: Centos机器磁盘扩容
categories:
  - 技术
  - 分区
tags: 磁盘
abbrlink: adf12f0a
date: 2024-11-11 18:35:47
---

内网机器磁盘可用空间不足，发现磁盘分区不合理，且有的分区磁盘未挂载使用

<!--more-->

![](Centos机器磁盘扩容/image-20250123145400157.png)



 把分区初始化为物理卷，以便被 LVM 使用：

```
pvcreate /dev/nvme0n1p3
```

![](Centos机器磁盘扩容/image-20250122183914741.png)

查看卷组信息：

![](Centos机器磁盘扩容/image-20250122183929423.png)

扩展卷组：

```
vgextend centos /dev/nvme0n1p3
```

![](Centos机器磁盘扩容/image-20250122183942757.png)



再查看卷组信息，发现有空闲的空间了：

![](Centos机器磁盘扩容/image-20250122183954453.png)



扩容空间：把全部空间扩容给 /dev/mapper/centos-root

```
lvextend -l +``100``%FREE /dev/mapper/centos-root
```

![](Centos机器磁盘扩容/image-20250122184013999.png)



看一下你的分区文件系统格式，运行cat /etc/fstab：

![](Centos机器磁盘扩容/image-20250122184027705.png)



扩展/dev/mapper/centos-root文件系统：

如果是ext文件系统：`resize2fs /dev/mapper/centos-root`
如果是XFS文件系统：`xfs_growfs /dev/mapper/centos-root`

![](Centos机器磁盘扩容/image-20250122184042978.png)



查看文件系统是否扩容成功：

扩容前：

![](Centos机器磁盘扩容/image-20250122184053342.png)

扩容后：

![](Centos机器磁盘扩容/image-20250122184102228.png)