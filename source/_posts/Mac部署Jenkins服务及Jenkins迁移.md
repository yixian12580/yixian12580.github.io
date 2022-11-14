---
title: Mac部署Jenkins服务Jenkins迁移
categories:
  - 技术
  - Jenkins
tags:
  - Jenkins
  - Jenkins迁移
abbrlink: 89a4bfe3
date: 2022-11-14 15:40:37
---

由于老的Jenkins服务器配置跟不上时代的发展了，需要把服务迁移至新Macmini机器上。

<!--more-->

通过brew安装，首先安装brew：

```
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"
```

安装好brew后，用brew命令安装Jenkins：

```
brew install jenkins
```

安装完成后使用一下命令启动Jenkins：

```
/opt/homebrew/opt/jenkins/bin/jenkins --httpListenAddress=0.0.0.0 --httpPort=8080
```

可用以下命令查看端口是否在监听：

```
netstat -AaLlnW | grep 8080
```

![image-20221114154422678](Mac部署Jenkins服务Jenkins迁移/image-20221114154422678.png)

或者：

```
sudo lsof -nP |grep LISTEN
```

打开浏览器，IP+8080访问：

找到页面提示的路径中的文件，输入文件中的密码信息：

![image-20221114154454182](Mac部署Jenkins服务Jenkins迁移/image-20221114154454182.png)

勾选需要安装的插件，等待配置：

![image-20221114154511515](Mac部署Jenkins服务Jenkins迁移/image-20221114154511515.png)

来到创建用户的界面，设置管理员用户名和密码：

![image-20221114154540571](Mac部署Jenkins服务Jenkins迁移/image-20221114154540571.png)

按照提示继续，就能来到这个界面啦：

![image-20221114154556778](Mac部署Jenkins服务Jenkins迁移/image-20221114154556778.png)

至此，Jenkins服务器部署好了，需要把旧服务器数据迁移过来，把Jenkins整个目录拷贝过来就行：

备份.jenkins目录：

```
mv .jenkins .jenkins.bak
```

拷贝目录：

```
scp -r ./.jenkins/ stary@172.22.13.45:/Users/stary/
```

重启Jenkins即可：

![image-20221114154709775](Mac部署Jenkins服务Jenkins迁移/image-20221114154709775.png)
