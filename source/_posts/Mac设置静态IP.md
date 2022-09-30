---
title: Mac设置静态IP
tags: 静态ip
categories:
  - 桌面
  - Mac
abbrlink: 9bc44327
date: 2022-09-23 10:07:12
---

问题表现：Mac笔记本设置静态IP地址显示：无效的服务器地址 BasicIPv6ValidationError

<!--more-->

解决方法：

打开终端：

```
networksetup -setv6off Ethernet（以太网名称）#关闭ipv6

networksetup -setmanual Ethernet 192.168.31.2 255.255.255.0 192.168.1.1 #设置IP地址，对应IP地址、子网掩码、网关
```

