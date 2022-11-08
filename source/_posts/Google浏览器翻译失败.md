---
title: Google浏览器翻译失败
categories: 桌面
tags:
  - google
  - 翻译
abbrlink: 4415a2fd
date: 2022-10-17 16:43:10
---

谷歌浏览器默认使用的翻译网站域名为translate.google.com IP为203.208.46.200

<!--more-->

请求超时是很正常的，毕竟网站服务器是国外

谷歌设有国内的翻译网站translate.google.cn IP为 203.208.40.66
这个网站是能够正常ping同和请求访问的

解决方法：

打开C:\Windows\System32\drivers\etc目录下的hosts文件。

该文件需要管理员权限，或者是只读。无法更改内容。此时进行文件的属性权限修改让管理员拿到所有权就可以进行修改或者把文件复制到桌面，修改完后再覆盖回去。

在hosts文件中增加以下两行内容即可:

```
203.208.40.66 translate.google.com
203.208.40.66 translate.googleapis.com
```

使用`win + R`键运行`cmd` 窗口
输入指令

```
ipconfig /flushdns
```

至此，就可以使用翻译功能。
