---
title: 'openVPN客户端安装失败 '
categories:
  - 技术
  - openVPN
tags:
  - Windows
  - openVPN
abbrlink: d0bf260f
date: 2022-10-11 15:07:37
---

安装openVPN时进度条卡在“tap install (tap0901) (may require confirmation)”，最后弹出错误“an error ocurred installing the TAP device driver”

<!--more-->

查看日志提示：

```
There are no TAP-Windows adapters on this system. You should be able to create a TAP-Windows adapter by going to Start -> All Programs -> TAP-Windows -> Utilities -> Add a new TAP-Windows virtual ethernet adapter.
```

![image-20221011151537341](openVPN客户端安装失败/image-20221011151537341.png)

此问题是由于TAP网络适配器没有安装成功所导致的，单独安装TAP适配器也是报错。

解决办法：

1.重置网络（可能会导致断网可提前下载ccleaner）

2.运行ccleaner，扫描注册表并选择修复所有问题。

![image-20221011151649393](openVPN客户端安装失败/image-20221011151649393.png)

3.重新安装openVPN。
