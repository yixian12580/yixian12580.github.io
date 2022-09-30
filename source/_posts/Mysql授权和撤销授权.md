---
title: Mysql授权和撤销授权
categories:
  - 技术
  - 数据库
tags: MySQL
abbrlink: ee8bdffb
date: 2022-09-27 21:39:24
---

对某个用户授予对某个表的某种操作的权限，并可限制具体的网段

<!--more-->

```
GRANT SELECT,INSERT ON book_review.* TO 'test'@'10.100.100.%' IDENTIFIED BY 'test000';
```

查询某个网段的用户信息，因为授权或者禁用是根据某个网段的主机进行操作:

```
SELECT User,Host from `mysql`.user where Host like '10.100.100.%'
```

再根据上面查到的信息，查看已经授权的项:

```
SHOW grants for test@10.100.100.%
```

接下来就可以根据上面查到的选项进行撤销授权：

```
revoke ALL ON `test_db`.* FROM 'test'@'10.100.100.%';
```

