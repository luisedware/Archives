---
title: PHP Web Application 运行流程、概念术语、机制原理和代码实践
date: 2016-06-10 20:55:55
tags: PHP
category: PHP
---

本文的目的在于了解 PHP 处理 HTTP 请求的过程

## 运行流程

1. 客户端做出请求操作（输入网址、点击链接、提交表单）。
2. 向客户端设定的 DNS 服务器请求 IP 地址。
3. 客户端根据 DNS 服务器返回 IP 地址采用三次握手与服务端建立 TCP 连接。
4. TCP 连接成功后，客户端向服务端发送 HTTP 请求。
5. 服务端的 Web Server 会判断 HTTP 请求的资源类型，进行内容分发处理；如果请求的资源为 PHP 文件，服务端软件会启动对应的 CGI 程序进行处理，并返回处理结果。
6. 服务端将 Web Server 的处理结果响应给客户端
7. 客户端接收服务端的响应，并渲染处理结果，呈现出来并断开 TCP 连接。


## 概念术语

1.**客户端**

客户端，是指与服务端相对应，为客户提供本地服务的程序。一般安装在普通的用户机上，需要与服务端互相配合运行。Web Application 的客户端一般是浏览器。

2.**服务端**

服务端是为客户端服务的，服务的内容诸如向客户端提供资源，保存客户端数据。

3.**三次握手**

三次握手，又名询问握手协议，是一个用来验证用户或网络提供者的协议。

4.**CGI**

CGI 全称是 Common Gateway Interface，CGI 是外部应用程序与 Web Server 之间的接口标准，是在 CGI 程序和 Web 服务器之间传递信息的规程。CGI 规范允许 Web Server 执行外部程序，并将它们的输出发送给Web浏览器。

5.**DNS**

DNS 全称是 Domain Name System，译为域名系统。它可以将域名和 IP 地址互相映射，能够让用户使用域名就可以访问互联网服务。

6.**HTTP**

HTTP 全称是 HyperText Transfer Protocol，译为超文本传输协议，是一种网络协议。

7.**Web Server**

通常指 Apache、Nginx 等服务器软件


##机制原理

首先从 HTTP 协议开始谈起，HTTP 协议工作流程是客户端发送一个请求给服务端，服务端在接收到这个请求后将产生一个响应返回给客户端。那么 HTTP 请求和 HTTP 响应是什么？

### HTTP 请求
当客户端向服务端发出请求时，它向服务端传递一个数据块，即请求信息，HTTP 请求信息由三部分组成：

- **请求行**
- **请求报头**
- **请求正文**

### 请求行

请求行以一个方法符号开头，以空格分开，后面跟着请求的 URI 和协议的版本，格式如下：
```c
Request Method - URI HTTP-Version CRLF
```
上述格式中各参数说明如下：

- **Method**：请求方法，请求方法有多种
	- PUT
	- GET
	- POST
	- HEAD
	- TRACE
	- DELETE
	- OPTIONS
	- CONNECT
- **Request-URI**：一个统一资源标识符
- **HTTP-Version**：请求的 HTTP 协议版本。
- **CRLF**：回车和换行

### 请求报头

请求报头属于 HTTP 消息报头之一，请求报头允许客户端向服务端传递请求的附加信息以及客户端自身的信息。
每个报头域组成形式如下：
```
名字 + : + 空格 + 值
```
比较重要的几个报头如下：

- **Host**：头域指定请求资源的 Internet 主机和端口号，必须表示请求 URL 的原始服务器或者网关的位置。
- **Accept**：告诉服务器可以接受的文件格式。
- **Cookie**：Cookie 份两种，一种是客户端向服务器发送的，使用 Cookie 报头，用来标记一些信息；另一种是服务器发送给浏览器的，报头为 Set-Cookie。
- **Referer**：头域允许客户端指定请求 URI 的源资源地址，这可以允许服务器生成回退链表，可用来登录、优化缓存等。
- **User-Agent**：UA 包含发出请求的用户信息。通常 User-Agent 包含浏览者的信息，主要是浏览器的名称版本和所用的操作系统。
- **Connection**：表示是否需要持久连接。
- **Cache-Control**：指定请求和响应遵循的缓存机制
- **Content-Range**：响应的资源范围。可以在每次请求中标记请求的资源范围，在连接断开重连时，客户端只请求重连时，客户端只请求该资源未下载的部分，而不是重新请求整个资源。
- **Content-Length**：内容长度
- **Accept-Encoding**：指定所能接受的编码方式。通常服务器会对页面进行 GZIP 压缩后再输出以减少流量。一般浏览器均支持对这种压缩后的数据进行处理。

### 请求正文

请求头和请求正文之间的一个空行表示请求头已经结束了。请求正文中可以包含提交的查询字符串信息。GET 方式没有请求正文。
```
username=admin&password=123456
```

### HTTP 请求示例
```
GET /menu HTTP/1.1
Host: localhost:5000
Connection: keep-alive
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/51.0.2704.84 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Encoding: gzip, deflate, sdch
Accept-Language: zh-CN,zh;q=0.8,en;q=0.6
Cookie: cartalyst_sentry=luisedware#anneason
```


### HTTP 响应
服务端在接收和解释请求消息后，服务端会返回一个 HTTP 响应消息。HTTP 响应也由三个部分组成，分别是：

- **响应行**
- **响应报头**
- **响应正文**

### 响应行

响应行格式如下：
```
HTTP - Version Status  - Code Reason - Phrase CRLF
```

上述格式中各参数说明如下：

- **HTTP - Version**：服务端 HTTP 协议的版本。
- **Status-Code**：服务端发回的响应状态代码。
	- 状态代码由三位数字组成，第一个数字定义了响应的类别，有五种可能取值：
		- 1xx：提示信息 - 请求已接收，继续处理。
		- 2xx：成功 - 请求已被成功接收、理解、接受。
		- 3xx：重定向 - 要完成请求必须进行更进一步的操作。
		- 4xx：客户端错误 - 请求有语法错误或请求无法实现。
		- 5xx：服务端错误 - 服务器未能实现合法的请求。
	- 常见状态代码、状态描述和说明如下：
		- 200 OK： 客户端请求成功
		- 400 Bad Request：客户端请求有语法错误，不能被服务器所理解
		- 401 Unauthorize：请求未经授权，这个状态代码必须和 WWW-Authenticate 报头域一起使用
		- 403 Forbidden：服务器收到请求，但是拒绝提供服务。
		- 404 Not Found：请求资源不存在，例如输入了错误的 URL
		- 500 Internal Server Error：服务器发生了不可预期的错误。
		- 503 Server Unavailable：服务器当前不能处理客户端的请求，一段时间后可能恢复正常
- **Reason-Phrase**：状态代码的文本描述。

### 响应报头

响应报头属于 HTTP 消息报头之一，响应报头允许服务器传递不能放在响应行中的附加响应信息，以及关于服务器的信息和对 Request-URI 所标识的资源进行下一步的访问的信息（如 Location）

### 响应正文
响应正文就是服务端返回的资源内容，响应报头和正文之间也必须使用空行分隔。

### HTTP 响应示例
```
HTTP/1.1 200 OK
host: localhost:8000
connection: close
x-powered-by: PHP/7.0.5
cache-control: no-cache
date: Fri, 10 Jun 2016 09:29:07 GMT
content-type: text/html; charset=UTF-8
set-cookie: XSRF-TOKEN=eyJpdiI6IlwvKzBIXC9JNzN2YUtpNGNaNndyTG40dz09IiwidmFsdWUiOiJtZlBDOEFQSmNUdzkyMXNQV1NSN2hPVjVFK1FHXC8xVU9nVGMwWFdcL3Q4MWlzc1A0dnhvRlBQckw1YVNpV0hQSTdFODBOQ2FFanF6YVg1TlRiQUhleEtnPT0iLCJtYWMiOiI5ZGRkOWEyYzk3OWY2YzZhOTA5MmFiOTA5ZmFmZmRiNTYxMzA5MjQ4NDdjYjcyNzIzNThjMzAxNmRjNDkzN2UxIn0%3D; expires=Fri, 10-Jun-2016 11:29:07 GMT; Max-Age=7200; path=/
set-cookie: laravel_session=eyJpdiI6ImdZWUZNOGFKTzV6YWpcL1lPRzRDTHdnPT0iLCJ2YWx1ZSI6IjFxRGRUOG5jRGVSdWJLXC9CMTl6MVJFdVlmaVpER04xb0piMWp3Q3JcL3dMZmR6b3UrU3lpb3FzQTR5QlpoTWxlOE92cmVCbW5iZGZwUUZ1a0d3ZjQrVUE9PSIsIm1hYyI6ImQ5N2M2MjVmYjcxNmE5YzgyY2IyZWJhODhiZTA0NGUxNTdlYmZhYjBkOGEzYzdiNTBiYmJjODE3MWJiOTA5NTIifQ%3D%3D; expires=Fri, 10-Jun-2016 11:29:07 GMT; Max-Age=7200; path=/; HttpOnly
Transfer-Encoding: chunked

<html>
<head>
<title>HTTP响应示例<title>
</head>
<body>
Hello World!
</body>
</html>
```

## Apache + PHP/Nginx + FPM + PHP 的工作原理
当服务端接收 HTTP 请求后，服务端的服务器软件，如Apache、Nginx则会根据配置处理 HTTP 请求进行分发处理。如果客户端请求的是 index.html 文件，那么 Web Server 会根据配置文件去对应目录下找到这个文件，然后让服务端发送给客户端，这样分发的是静态资源。如果客户端请求的是 index.php 文件，根据配置文件，Web Server 知道这个不是静态资源，需要去找 PHP 解析器来处理，那么 Web Server 会把这个请求简单处理，然后交给 PHP 解析器。

当 Web Server 收到 index.php 这个请求后，会启动对应的 CGI 程序，CGI 程序就是 PHP 的解析器了。PHP 解析器会解析 php.ini 文件，初始化执行环境，然后处理请求，再以 CGI 规定的格式返回处理结果，Web Server 再把结果返回给客户端。实现 PHP 解析器的方法有三种：

- **Module**：PHP Module加载方法。
- **PHP-CGI**：是 PHP 对 Web Server 提供的 CGI 协议的接口程序。
- **PHP-FPM**：是 PHP 对 Web Server 提供的 FastCGI 协议的接口程序，额外提供了相对智能的任务管理。

以上三种实现方法都是本质都是通过 PHP 的 SAPI 层调用 PHP 执行的。



### Module
在了解 CGI 之前，我们先了解一下 Apache 处理 PHP 的一种常见方法：PHP Module 加载方式。在 Apache 的配置文件 httpd.conf，我们可以查找到以下语句：

```html
LoadModule php7_module  /usr/local/opt/php70/libexec/apache2/libphp7.so

<FilesMatch .php$>
    SetHandler application/x-httpd-php
</FilesMatch>

<IfModule dir_module>
    DirectoryIndex index.php index.html
</IfModule>
```

这种方法，本质上是使用 Apache 的 `LoadModule` 来加载 `php7_module`，也就是把 PHP 作为 Apache 一个子模块来运行。当客户端请求 PHP 文件时，`Apache` 就会调用 `php7_module` 来解析 PHP 代码。

这种模式将 PHP 模块安装到 Apache 中，每一次 Apache 接收请求时，都会产生一条进程，这条进程就完整的包括 PHP 的运算计算，数据读取等各种操作。

由于每次请求都需要产生一条进程来连接 SAPI 来完成请求，一旦并发数过高，服务端就会承受不住。而且把 php7_module 加载进 Apache 中，出现问题时很难定位是 PHP 的问题 还是 Apache 的问题。


### PHP-CGI
PHP-CGI 是使用 PHP 实现 CGI 接口的程序。当 Web Server 接收到 PHP 文件请求时，会分发给 CGI 程序处理，CGI 程序处理完毕后，会返回结果给 Web Server，Web Server 再返回给客户端。

PHP-CGI 的好处就是完全独立于 Web Server，只是作为中间层，提供接口给 Apache 和 PHP，它们通过 CGI 来完成数据传递。这样做的好处就减少了两者之间的关联，出现错误时能够较好地定位。

但是 PHP-CGI 采用的是 fork-and-execute 模式，就是每次请求都会有进程启动和退出的过程，在高并发下，Web Server 容易奔溃。

### PHP-FPM
在了解 PHP-FPM 之前，我们先来了解一下 FastCGI。

从根本上来说，FastCGI 是用来提高 CGI 程序性能的。类似于 CGI，FastCGI 也可以说是一种协议。

FastCGI 像是一个常驻（long-live）型的 CGI，它可以一直执行着，只要激活后，不会每次都要花费时间去 fork 一次。它还支持分布式的运算, 即 FastCGI 程序可以在网站服务器以外的主机上执行，并且接受来自其它网站服务器来的请求。

FastCGI 是语言无关的、可伸缩架构的 CGI 开放扩展，其主要行为是将 CGI 解释器进程保持在内存中，并因此获得较高的性能。众所周知，CGI 解释器的反复加载是 CGI 性能低下的主要原因，如果 CGI 解释器保持在内存中，并接受 FastCGI 进程管理器调度，则可以提供良好的性能、伸缩性、Fail- Over 特性等等。

FastCGI 接口方式采用C/S结构，可以将 Web Server 和脚本解析服务器分开，同时在脚本解析服务器上启动一个或者多个脚本解析守护进程。当HTTP服务器每次遇到动态程序时，可以将其直接交付给 FastCGI 进程来执行，然后将得到的结果返回给浏览器。这种方式可以让 Web Server 专一地处理静态请求，或者将动态脚本服务器的结果返回给客户端，这在很大程度上提高了整个应用系统的性能。

PHP-FPM 是对 FastCGI 协议的具体实现，它负责管理一个进程池，来处理 Web Server 的请求。它自身并不直接处理请求，它会交给 PHP-CGI 去进行处理。因为 PHP-CGI 只是一个 CGI 程序，它只会解析请求，返回结果，不会管理进程。PHP-FPM 是用于调度管理 PHP 解释器 PHPCGI 的管理程序。

### CGI、FastCGI、PHP-CGI、PHP-FPM 的区别

- **CGI**：是一种协议，是外部应用程序与 Web Server 之间的接口标准，是在 CGI 程序和 Web 服务器之间传递信息的规程。
- **FastCGI**：是一种协议，通过守护进程来管理 CGI 程序，将 CGI 程序常驻于内存中，不必每次处理请求重新初始化 php.ini 和其他数据，提高 CGI 程序的性能。其本身并不处理 PHP 文件，只是负责进程的调度。
- **PHP-CGI**：是使用 PHP 实现 CGI 协议的程序，用于解析和处理 PHP 脚本。
- **PHP-FPM**：是一个只用于 PHP 的进程管理器，提供更好的 PHP 进程管理方式，可以有效控制进程，平滑地加载 PHP 配置文件。

### Apache 常用的配置与模块

- **Options**（配置选项）
	- **Indexes**：目录浏览，是否允许在目录下没有 index.html,index.php 的时候显示目录。
	- **Multiviews**：文件匹配，服务器执行一个隐含的文件名模式匹配，并在其结果中选择。
	- **FollowSymLinks**：符号链接，是否允许通过符号链接跨越 DocumentRoot 访问其他目录。
- **AllowOverride**（是否允许根目录下的 `.htaccess` 文件起到 `URL rewrite` 的作用）
	- **All**
	- **None**
- **LoadModule**（加载模块）
- **Listen**（Apache 默认监听端口）
- **ServerRoot**：Apache 安装的基础目录
- **DocumentRoot**：Apache 缺省文件根目录
- **DirectoryIndex**：网站默认首页文件
- **LogLevel**：日志级别设置
- **ErrorLog**：错误日志存放路径
- **CustomLog**：访问日志存放路径
- **VirutalHost**（虚拟主机与配置参数）
	- **ServerName**：虚拟域名
	- **ServerAlias**：虚拟域名的别名
	- **ServerAdmin**：服务器管理员邮箱
	- **DocumentRoot**：项目代码根目录
	- **LogLevel**：日志级别设置
	- **ErrorLog**：错误日志存放路径
	- **CustomLog**：访问日志存放路径
- **MPM**（工作模式/多处理模块）
	- **Prefork**
		- **StartServers**：服务器启动时默认开启的进程数
		- **MinSpareServers**：最小的空闲进程数
		- **MaxSpareServers**：最大的空闲进程数
		- **ServerLimit**：在Apache的生命周期内，限制MaxClients的最大值
		- **MaxClients**：最大的并发请求数，最大值不能超过 ServerLimit 设置的值
		- **MaxRequestsPerChild**：一个进程可以处理的最多的请求数（进程复用），如请求超过该设置则杀死进程，0表示永不过期。
	- **Worker**
	- **Event**
- **Order, Deny, Allow**（认证，授权与访问控制）
- **HostnameLookups off**（避免 DNS 查询）
- **Timeout**（请求超时）
- **Include**（文件包含）
- **Proxy、mod_proxy**（代理）
- **Cache Guide**（缓存）

### Nginx 常用模块与配置指令上下文

#### Nginx 模块

Nginx 的模块从功能角度主要可以分为以下三类：

- **Handler** 模块：主要负责处理客户端请求并产生待响应内容，比如 `ngx_http_static_module`模块，负责客户端的静态页面请求处理并将对应的磁盘文件准备为响应内容输出。
- **Filter** 模块：主要负责对输出的内容进行处理，可以对输出进行修改，如 `ngx_http_not_modified_filter_module`,`ngx_http_header_filter_module` 模块。
- **Upstream** 模块：实现反向代理的功能，将真正的请求转发到后端服务器上，如 `ngx_http_proxy_module`、`ngx_http_fastcgi_module` 模块。


#### Nginx 配置指令

- **main**（Nginx 在运行时与具体业务功能无关的一些参数，比如工作进程数，运行的身份等）
	- **user**
	- **worker_processes**
	- **error_log**
	- **events**
	- **http**
	- **mall**
- **events**（定义 Nginx 事件工作模式与连接数上限等参数）
	- **use** [ kqueue | rtsig | epoll | /dev/poll | select | poll ];
	- **worker_connections**
- **http**（与提供 HTTP 服务相关的一些配置参数，如是否使用 keepalive、是否使用 gzip 进行压缩等）
	- **server**
- **server**（http服务上支持若干虚拟主机。每个虚拟主机一个对应的server配置项，配置项里面包含该虚拟主机相关的配置。在提供mail服务的代理时，也可以建立若干server.每个server通过监听的地址来区分）
	- **listen**
	- **server_name**
	- **access_log**
	- **location**
	- **protocol**
	- **proxy**
	- **smtp_auth**
	- **xclient**
- **location**（http服务中，某些特定的URL对应的一系列配置项。）
	- **root**
	- **index**
- **mail**（实现email相关的SMTP/IMAP/POP3代理时，共享的一些配置项（因为可能实现多个代理，工作在多个监听地址上）
	- **server**
	- **auth_http**
	- **imap_capabilities**

