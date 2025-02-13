---
title: k8s之Ingress资源详解
categories:
  - 技术
  - k8s
tags: Ingress
abbrlink: 56fdeaa
date: 2024-08-06 09:59:49
---

**Ingress** 是一种 Kubernetes 资源对象，用于对外暴露服务。它定义了不同主机名（域名）及 URL 和对应后端 Service 的绑定，根据不同的路径路由 HTTP 和 HTTPS 流量。**Ingress Controller** 是一个 Pod 服务，封装了一个 Web 前端负载均衡器，并根据 Ingress 的定义动态生成配置文件。

<!--more-->

以下是我工作中使用到的一些配置，记录一下：

```
用于认证：
    nginx.ingress.kubernetes.io/auth-realm: '''Authentication Required - admin'''     #设置认证提示信息
    nginx.ingress.kubernetes.io/auth-secret: kibana-auth    #指定存储认证信息的secret
    nginx.ingress.kubernetes.io/auth-type: basic    #设置认证类型为基本认证（basic）

用于访问控制：	
    nginx.ingress.kubernetes.io/whitelist-source-range: 121.33.0.0/16     #用于指定允许访问Ingress资源的IP地址范围
    
用于反向代理超时设置：
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "300"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "300"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "300"

用于文件大小限制：
    nginx.ingress.kubernetes.io/proxy-body-size: 50m   #设置Nginx代理能够接收的最大请求体大小

用于请求限制：
    nginx.ingress.kubernetes.io/limit-connections: '20'   #设置每个客户端IP允许的最大并发连接数
    nginx.ingress.kubernetes.io/limit-rps: '4'    #设置每秒允许的请求速率，'4'表示每秒最多允许4个请求
    nginx.ingress.kubernetes.io/limit-whitelist: '10.0.0.0/8,172.16.0.0/20'   #指定不受连接数和请求速率限制的白名单IP地址范围

用于证书设置：
    ingress.kubernetes.io/ssl-redirect: 'true'    #控制是否将HTTP流量重定向到HTTPS
  spec:
    ingressClassName: nginx
    tls:
      - hosts:
          - biz.chain.test.com
        secretName: biz-chain-tls-secret

用于缓存设置：
    nginx.ingress.kubernetes.io/configuration-snippet: |      #提供自定义的Nginx配置片段，这些配置片段将被直接插入到Nginx的配置中，从而允许对Ingress的行为进行细粒度的控制
      if ($request_uri = "/index.html") {
        more_set_headers "cache-Control: no-cache, no-store";
      }
      if ($request_uri = "/") {
        more_set_headers "cache-Control: no-cache, no-store";
      }
      if ($request_uri = "/login") {
        more_set_headers "cache-Control: no-cache, no-store";
      }
      if ($request_uri ~ "/login.*") {
        more_set_headers "Cache-Control: no-cache, no-store";
      }
注解包含了四个if语句，每个语句都检查请求的URI，并根据URI设置不同的Cache-Control头部。具体来说：
如果请求的URI是/index.html，则设置Cache-Control头部为no-cache, no-store。
如果请求的URI是根路径/，同样设置Cache-Control头部为no-cache, no-store。
如果请求的URI是/login，也设置Cache-Control头部为no-cache, no-store。
如果请求的URI以/login开头（使用正则表达式匹配），同样设置Cache-Control头部为no-cache, no-store。
Cache-Control头部用于控制浏览器和其他缓存机制如何缓存响应内容。no-cache指令表示在每次请求时都需要向服务器验证缓存内容是否仍然有效，而no-store指令则完全禁止缓存响应内容。
通过这些配置，您可以确保对于特定的请求URI，浏览器不会缓存响应内容，这对于需要频繁更新或敏感信息的页面特别有用。例如，登录页面通常不应该被缓存，以防止敏感信息（如登录令牌）被存储在用户的浏览器中。

其他：
    nginx.ingress.kubernetes.io/use-regex: 'true'      当设置为'true'时，表示Ingress规则中的路径将使用正则表达式进行匹配
    kubernetes.io/ingress.class: nginx    注解指定Ingress控制器类型为nginx
    kubesphere.io/creator: admin          注解用于标识创建者信息
```

