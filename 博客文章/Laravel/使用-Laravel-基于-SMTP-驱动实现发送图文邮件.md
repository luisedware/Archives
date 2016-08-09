---
title: 使用 Laravel 基于 SMTP 驱动实现发送文字邮件
date: 2016-08-08 11:44:47
tags: Laravel
category: PHP
---

## 环境

- PHP 7
- Laravel 5.1
- OS X El Capitan 10.11.4


## 简介

Laravel 基于热门的 SwiftMailer 函数库提供了一个简洁的 API。Laravel 为 SMTP、Mailgun、Mandrill、Amazon SES、PHP 的 mail 函数及 sendmail 提供驱动，让你可以快速地从所选择的本地或云端服务开始发送邮件。（摘录 PHPhub 翻译文档）

## 配置

邮件配置文件是`config/mail.php`
```php
return [
	// 默认发送邮件驱动
	'driver' => env('MAIL_DRIVER', 'smtp'),

	// 发送邮件主机地址
    'host' => env('MAIL_HOST', 'smtp.mailgun.org'),

    // 发送邮件主机端口
    'port' => env('MAIL_PORT', 587),

    // 指定发送邮件的邮箱地址和用户名称
    'from' => ['address' => null, 'name' => null],

    // 指定发送邮件协议
    'encryption' => env('MAIL_ENCRYPTION', 'tls'),

    // 邮箱登录账号
    'username' => env('MAIL_USERNAME'),

    // 邮箱登录密码
    'password' => env('MAIL_PASSWORD'),

    // 当驱动为 sendmail 时，指定驱动的命令地址
    'sendmail' => '/usr/sbin/sendmail -bs',

	// false 发送邮件不记录日志，true 记录日志不发送邮件
    'pretend' => false,
];
```

## 编写程序

### env
本文采用 QQ 邮箱进行测试，首先修改邮箱配置
```
MAIL_DRIVER=smtp
MAIL_HOST=smtp.qq.com
MAIL_PORT=465
MAIL_USERNAME=（填写 QQ 邮箱账号）
MAIL_PASSWORD=（填写 QQ 邮箱密码）
MAIL_ENCRYPTION=ssl
```

### 路由
```
/* 邮件管理模块 */
Route::get('email/send', [
    'as'   => 'backend.email.send',
    'uses' => 'EmailController@send',
]);
```

### 控制器
新增控制器
```
php artisan make:controller Backend/EmailController --plain
```

控制器代码如下
```
<?php

namespace App\Http\Controllers\Backend;

use App\Facades\UserRepository;
use Illuminate\Http\Request;

use App\Http\Requests;
use App\Http\Controllers\Controller;

use Mail;

class EmailController extends Controller
{
    public function send(Request $request, $id)
    {
        $user = UserRepository::find($id);

        $result = Mail::send('emails.test', ['user' => $user], function ($email) use ($user) {
            $email->to('2794408425@qq.com')->subject('Hello World');
        });

        if($result){
            echo '发送邮件成功';
        } else {
            echo '发送邮件失败';
        }
    }
}

```

### 新增视图

新增视图`emails/test.blade.php`，代码如下：
```html
<html>
<head>
    <title></title>
</head>
<body>
	你好,{{$user->name}},这是一封测试邮件。
</body>
</html>
```

## 执行代码
在浏览器访问指定路由，然后去邮箱查看邮件是否发送成功。
![配置 PHP 的使用版本](http://o93kt6djh.bkt.clouddn.com/Laravel-SendEmail%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-08-09%2009.49.12.png)


