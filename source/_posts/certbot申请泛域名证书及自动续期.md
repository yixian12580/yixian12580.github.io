---
title: certbot申请泛域名证书及自动续期
categories:
  - 技术
  - 域名证书
tags: 免费证书
abbrlink: 540ef920
date: 2024-06-25 16:12:07
---

阿里云申请的免费证书有效期变短了，还有数量限制，而且不能申请泛域名证书，可以通过cerbot申请免费的域名证书并且自动续期。

<!--more-->

### 原理



当我们使用 certbot 申请通配符证书时，需要手动添加 TXT 记录。每个 certbot 申请的证书有效期为 3 个月，虽然 certbot 提供了自动续期命令，但是当我们把自动续期命令配置为定时任务时，我们无法手动添加新的 TXT 记录用于 certbot 验证。

好在 certbot 提供了一个 hook，可以编写一个 Shell 脚本。在续期的时候让脚本调用 DNS 服务商的 API 接口动态添加 TXT 记录，验证完成后再删除此记录。

### 安装(CommandLine)



1. 安装 aliyun cli 工具

   ```
   wget https://aliyuncli.alicdn.com/aliyun-cli-linux-latest-amd64.tgz
   tar xzvf aliyun-cli-linux-latest-amd64.tgz
   sudo cp aliyun /usr/local/bin
   rm aliyun
   ```

   

   安装完成后需要配置[凭证信息](https://help.aliyun.com/document_detail/110341.html)

2. 安装 certbot-dns-aliyun 插件

   ```
   wget https://cdn.jsdelivr.net/gh/justjavac/certbot-dns-aliyun@main/alidns.sh
   sudo cp alidns.sh /usr/local/bin
   sudo chmod +x /usr/local/bin/alidns.sh
   sudo ln -s /usr/local/bin/alidns.sh /usr/local/bin/alidns
   rm alidns.sh
   ```

   

3. 申请证书

   测试是否能正确申请：

   ```
   certbot certonly -d *.example.com --manual --preferred-challenges dns --manual-auth-hook "alidns" --manual-cleanup-hook "alidns clean" --dry-run
   ```

   

   正式申请时去掉 `--dry-run` 参数：

   ```
   certbot certonly -d *.example.com --manual --preferred-challenges dns --manual-auth-hook "alidns" --manual-cleanup-hook "alidns clean"
   ```

   

4. 证书续期

   ```
   certbot renew --manual --preferred-challenges dns --manual-auth-hook "alidns" --manual-cleanup-hook "alidns clean" --dry-run
   ```

   

   如果以上命令没有错误，把 `--dry-run` 参数去掉。

5. 自动续期

   添加定时任务 crontab。

   ```
   crontab -e
   ```

   

   输入

   ```
   1 1 */1 * * root certbot renew --manual --preferred-challenges dns --manual-auth-hook "alidns" --manual-cleanup-hook "alidns clean" --deploy-hook "nginx -s reload"
   ```

   

   上面脚本中的 `--deploy-hook "nginx -s reload"` 表示在续期成功后自动重启 nginx。

### 安装（Docker）



编写Dockerfile以及entrypoint.sh,确保他们在同一文件夹下，目前Dockerfile中默认下载amd64版本，其他架构请修改对应的Aliyun CLI URL。

DockerFile：

```
FROM alpine:latest

# Install dependencies
RUN apk --no-cache add wget tar sudo certbot bash python3 py3-pip && \
    apk --no-cache add --virtual build-dependencies gcc musl-dev python3-dev libffi-dev openssl-dev make

# Install aliyun-cli
RUN wget https://aliyuncli.alicdn.com/aliyun-cli-linux-latest-amd64.tgz && \
    tar xzvf aliyun-cli-linux-latest-amd64.tgz && \
    mv aliyun /usr/local/bin && \
    rm aliyun-cli-linux-latest-amd64.tgz

# Copy and install certbot-dns-aliyun plugin
RUN wget https://cdn.jsdelivr.net/gh/justjavac/certbot-dns-aliyun@main/alidns.sh && \
    mv alidns.sh /usr/local/bin/alidns && \
    chmod +x /usr/local/bin/alidns

# Create virtual environment for Python packages
RUN python3 -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# Install Python dependencies in virtual environment
RUN pip install --upgrade pip && \
    pip install aliyun-python-sdk-core aliyun-python-sdk-alidns

# Copy entrypoint script
COPY entrypoint.sh /usr/local/bin/entrypoint.sh
RUN chmod +x /usr/local/bin/entrypoint.sh

# Set environment variables (to be provided during runtime)
ENV REGION=""
ENV ACCESS_KEY_ID=""
ENV ACCESS_KEY_SECRET=""
ENV DOMAIN=""
ENV EMAIL=""
ENV CRON_SCHEDULE="0 0 * * *"

# Setup cron job for certbot renew
RUN echo "$CRON_SCHEDULE /opt/venv/bin/certbot renew --manual --preferred-challenges dns --manual-auth-hook '/usr/local/bin/alidns' --manual-cleanup-hook '/usr/local/bin/alidns clean' --agree-tos --email $EMAIL --deploy-hook 'cp -r /etc/letsencrypt/live/$DOMAIN/* /etc/letsencrypt/certs'" > /etc/crontabs/root

# Create directory for certificates
RUN mkdir -p /etc/letsencrypt/certs

# Make sure cron is running
RUN touch /var/log/cron.log

ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
```

entrypoint.sh：

```
#!/bin/bash

# Activate the virtual environment
source /opt/venv/bin/activate

# Configure Aliyun CLI
aliyun configure set --profile akProfile --mode AK --region $REGION --access-key-id $ACCESS_KEY_ID --access-key-secret $ACCESS_KEY_SECRET

# Obtain the certificate
certbot certonly -d "$DOMAIN" --manual --preferred-challenges dns --manual-auth-hook "/usr/local/bin/alidns" --manual-cleanup-hook "/usr/local/bin/alidns clean" --agree-tos --email $EMAIL --non-interactive

# Start cron daemon
crond -f -l 2
```



创建Image

进入Dockerfile同目录:

```
docker build -t certbot-alicli .
```



使用代理（可选）:

```
docker build . \
 --build-arg "HTTP_PROXY=http://127.0.0.1:7890" \
 --build-arg "HTTPS_PROXY=http://127.0.0.1:7890" \
 -t certbot-alicli
```



运行容器

```
docker run \
-e REGION=YOUR_REGEION \
-e ACCESS_KEY_ID=YOUR_ACCESS_KEY \
-e ACCESS_KEY_SECRET=YOUR_ACCESS_SECRET \
-e DOMAIN=YOUR_DOMAIN \
-e EMAIL=YOUR_NOTIFICATION_EMAIL \   // 证书刷新通知邮箱
-e CRON_SCHEDULE="0 0 * * *" \   // 自定义证书刷新间隔
-v /path/letsencrypt:/etc/letsencrypt \ // 将容器内的证书路径完整映射到宿主机
-d certbot-alicli
```

