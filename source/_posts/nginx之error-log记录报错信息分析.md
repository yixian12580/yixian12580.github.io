---
title: nginx之error.log记录报错信息分析
categories:
  - Linux
  - Nginx
tags:
  - Nginx
abbrlink: 701bd567
date: 2024-02-29 13:24:26
---

我们经常遇到各种各样的nginx错误日志，平时根据一些nginx错误日志就可以分析出原因了，不过不是很系统，这里记录一下关于nginx的error.log的详细说明，方便以后查看了解。

<!--more-->

| 错误信息                                                     | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| “upstream prematurely（过早的） closed connection”           | 请求uri的时候出现的异常，是由于upstream还未返回应答给用户时用户断掉连接造成的，对系统没有影响，可以忽略 |
| “recv() failed (104: Connection reset by peer)”              | （1）服务器的并发连接数超过了其承载量，服务器会将其中一些连接Down掉； （2）客户关掉了浏览器，而服务器还在给客户端发送数据； （3）浏览器端按了Stop |
| “(111: Connection refused) while connecting to upstream”     | 用户在连接时，若遇到后端upstream挂掉或者不通，会收到该错误   |
| “(111: Connection refused) while reading response header from upstream” | 用户在连接成功后读取数据时，若遇到后端upstream挂掉或者不通，会收到该错误 |
| “(111: Connection refused) while sending request to upstream” | Nginx和upstream连接成功后发送数据时，若遇到后端upstream挂掉或者不通，会收到该错误 |
| “(110: Connection timed out) while connecting to upstream”   | nginx连接后面的upstream时超时                                |
| “(110: Connection timed out) while reading upstream”         | nginx读取来自upstream的响应时超时                            |
| “(110: Connection timed out) while reading response header from upstream” | nginx读取来自upstream的响应头时超时                          |
| “(110: Connection timed out) while reading upstream”         | nginx读取来自upstream的响应时超时                            |
| “(104: Connection reset by peer) while connecting to upstream” | upstream发送了RST，将连接重置                                |
| “upstream sent invalid header while reading response header from upstream” | upstream发送的响应头无效                                     |
| “upstream sent no valid HTTP/1.0 header while reading response header from upstream” | upstream发送的响应头无效                                     |
| “client intended to send too large body”                     | 用于设置允许接受的客户端请求内容的最大值，默认值是1M，client发送的body超过了设置值 |
| “reopening logs”                                             | 用户发送kill -USR1命令                                       |
| “gracefully shutting down”,                                  | 用户发送kill -WINCH命令                                      |
| “no servers are inside upstream”                             | upstream下未配置server                                       |
| “no live upstreams while connecting to upstream”             | upstream下的server全都挂了                                   |
| “SSL_do_handshake() failed”                                  | SSL握手失败                                                  |
| “SSL_write() failed (SSL:) while sending to client”          |                                                              |
| “(13: Permission denied) while reading upstream”             |                                                              |
| “(98: Address already in use) while connecting to upstream”  |                                                              |
| “(99: Cannot assign requested address) while connecting to upstream” |                                                              |
| “ngx_slab_alloc() failed: no memory in SSL session shared cache” | ssl_session_cache大小不够等原因造成                          |
| “could not add new SSL session to the session cache while SSL handshaking” | ssl_session_cache大小不够等原因造成                          |
| “send() failed (111: Connection refused)”                    |                                                              |

