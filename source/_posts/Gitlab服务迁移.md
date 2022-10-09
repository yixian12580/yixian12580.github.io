---
title: Gitlab服务迁移
categories:
  - 技术
  - Gitlab
tags:
  - Linux
  - Gitlab
abbrlink: 45efe774
date: 2022-10-09 17:16:44
---

由于Gitlab自身的兼容性问题，高版本的Gitlab无法恢复低版本备份的数据，所以新旧服务器Gitlab版本必须一致。

<!--more-->

查看GitLab版本：

```
cat /opt/gitlab/embedded/service/gitlab-rails/VERSION
```

新机器安装Gitlab服务，在此不再赘述，可参考此[文章](https://yixian12580.github.io/2022/099becdeed.html)。

备份旧服务器数据：

```
gitlab-rake gitlab:backup:create
```

默认将会在 /var/opt/gitlab/backups/ 目录下生成备份文件。

使用scp命令从旧服务器复制文件到新服务器：

```
scp /nfsdrive/backup/gitlab/1665250409_2022_10_09_12.10.14_gitlab_backup.tar root@172.31.0.123:/var/opt/gitlab/backups/
```

新服务器上将备份文件权限修改为777，避免出现权限不够的问题：

```
cd /var/opt/gitlab/backups
chmod 777 1665250409_2022_10_09_12.10.14_gitlab_backup.tar
```

新服务器上停止数据连接服务：

```
gitlab-ctl stop unicorn
gitlab-ctl stop sidekiq
```

新服务器上恢复备份文件至Gitlab：

```
gitlab-rake gitlab:backup:restore BACKUP=备份文件编号
```

例如：备份文件名为1665250409_2022_10_09_12.10.14_gitlab_backup.tar，则编号为1665250409_2022_10_09_12.10.14。
在提示中敲入“yes”继续。

完成后会看到提示：

![image-20221009171931511](Gitlab服务迁移/image-20221009171931511.png)

/etc/gitlab/gitlab-secrets.json和/etc/gitlab/gitlab.rb文件包含敏感数据，需要手动恢复这些文件。
如果不覆盖这两个文件会造成CI/CD配置页报500错误：

![image-20221009172008159](Gitlab服务迁移/image-20221009172008159.png)

需要旧服务器上把这两个文件拷贝至新服务器上覆盖：

```
scp /etc/gitlab/gitlab-secrets.json root@172.31.0.123:/etc/gitlab/gitlab-secrets.json
scp /etc/gitlab/gitlab.rb root@172.31.0.123:/etc/gitlab/gitlab.rb
```

重新配置并重启：

```
gitlab-ctl reconfigure && gitlab-ctl restart
```

检测恢复数据情况：

```
gitlab-rake gitlab:check SANITIZE=true
```

PS：如果网站配置了证书，需把证书放到对应路径：

![image-20221009172208268](Gitlab服务迁移/image-20221009172208268.png)

至此，Gitlab服务迁移完成。
