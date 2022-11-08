---
title: jira插件破解
categories:
  - 技术
  - jira
tags:
  - jira
  - 插件破解
abbrlink: 473314d
date: 2022-11-08 09:50:30
---

我使用的jira版本是7.13.0，与confluence的破解方式不一样。

<!--more-->

### 破解jira服务：

jira服务器授权更新：
将atlassian-extras-3.2.jar替换掉/usr/local/jira/atlassian/jira/atlassian-jira/WEB-INF/lib/目录下对应jar包，重启jira服务即可。

### jira插件授权更新：

使用的破解文件是：atlassian-universal-plugin-manager-plugin-2.22.9.jar  

JIRA V7.13安装后默认的atlassian-universal-plugin-manager-plugin版本就是2.22.9

方法很简单，直接将这个破解文件覆盖原来的文件，路径在：/usr/local/jira/atlassian-jira/WEB-INF/atlassian-bundled-plugins

然后重启jira服务。

插件破解原理：

atlassian-universal-plugin-manager-plugin插件是进行插件管理的，只需要破解了这个插件，剩下的所有插件都自动破解完成了

上面两个文件已配置好，直接到插件市场找你想要的插件

找到想要的插件后，免费试用，转到官网得到一个试用的license，把这个license直接黏贴到JIRA后台的插件管理中输入插件license的地方，会发现就破解了，插件的deadline就是你jira应用的deadline：2033年6月5日（前提是jira已经破解了）

注意，不要去升级插件：atlassian-universal-plugin-manager-plugin，就让版本一直是2.22.9，升级了插件就用不了了



**如果升级了，可以用如下方法：**

```
#cd /
#find . -name *manager-pl* |grep jira
```

返回如下：

```
find: ‘./proc/67008’: No such file or directory
find: ‘./proc/67009’: No such file or directory
find: ‘./proc/67010’: No such file or directory
find: ‘./proc/67011’: No such file or directory
./var/atlassian/application-data/jira/plugins/installed-plugins/plugin.7291572728482458613.atlassian-universal-plugin-manager-plugin-3.0.1.jar
./var/atlassian/application-data/jira/plugins/.osgi-plugins/transformed-plugins/atlassian-universal-plugin-manager-plugin-2.22.9_1543375156000.jar
./var/atlassian/application-data/jira/plugins/.osgi-plugins/transformed-plugins/plugin.7291572728482458613.atlassian-universal-plugin-manager-plugin-3.0.1_1545962501000.jar
./var/atlassian/application-data/jira/plugins/.osgi-plugins/transformed-plugins/atlassian-universal-plugin-manager-plugin-2.22.9_1545974690000.jar
./opt/atlassian/jira/atlassian-jira/WEB-INF/atlassian-bundled-plugins/atlassian-universal-plugin-manager-plugin-2.22.9.jar
./opt/atlassian/jira/temp/plugin.7291572728482458613.atlassian-universal-plugin-manager-plugin-3.0.1.jar
```

把这些jar文件都删除，再把文件：atlassian-universal-plugin-manager-plugin-2.22.9.jar 放/opt/atlassian/jira/atlassian-jira/WEB-INF/atlassian-bundled-plugins 目录下，重启jira.



**如果想修改Jira破解方式跟Confluence一样，需要先从数据库中得到 licences ，用破解包生成的 licences 替换。**

**生成 licenses**

　　进入后台管理面板页，在系统信息中获得服务器ID (管理——系统——系统信息——服务器 ID)
　　之后，参照wiki插件破解方式获取激活码(license)，保存备用。

**修改 licenses**

　　我们需要进入数据库，使用上文得到的激活码（license）来替换原有的 license。

**登陆数据库**

```
# 执行SELECT * FROM productlicense\G获得原来的激活信息（别忘了备份旧的license信息，方便回退）： 
MySQL [jiradb]> SELECT * FROM productlicense\G
*************************** 1. row ***************************
     ID: 10200
LICENSE: AAABZg0xxxx
*************************** 2. row ***************************
     ID: 10201
LICENSE: AAABcQ0ODAoxxxx
 
# 利用上面获取licences的方式获取licences后，更新记录
update productlicense set license ='xxxxx' WHERE id=10200;
update productlicense set license ='xxxxx' WHERE id=10201;
```

#### 修改启动命令

停止 jira 后，在 setenv.sh 文件末尾加入：

```
export JAVA_OPTS="-javaagent:/usr/local/confluence/ ${JAVA_OPTS}"
```

重新启动即可。
