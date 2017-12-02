---
title: 「Tips」.htaccess 配置 HTTP Header
date: 2017-03-15 17:18:21
tags: PHP
category: PHP
---

使用 Yii 2 做 RESTFul API 项目时，发现在前端发起一个请求时，会出现以下错误：
```console
Request header field access-token is not allowed by Access-Control-Allow-Headers in preflight response.
```

排查问题进行了很久，最后才发现在 Yii 2 的 `web` 目录下，有个文件 `.htaccess`，是可以配置 `Apache` 服务器接受 HTTP 请求的头部信息的，在特定选项加入 `access-token` 即刻不再报错，因为本次项目使用 `HTTP Header` 传递 `access-token` 的，没有加入 `access-token` 就意味 `Apache` 服务器不接受头部带有 `access-token` 的 HTTP 请求 。文件配置代码如下：
```apache
Header always set Access-Control-Allow-Origin "*"
Header always set Access-Control-Allow-Methods "POST, GET, OPTIONS, DELETE, PUT"
Header always set Access-Control-Max-Age "1000"
Header always set Access-Control-Allow-Headers "x-requested-with, Content-Type, origin, authorization, accept, client-security-token, access-token"

RewriteEngine on

RewriteCond %{REQUEST_METHOD} OPTIONS
RewriteRule ^(.*)$ $1 [R=200,L]

RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . index.php
```
