---
title: 使用 Laravel 基于 SMTP 驱动实现发送邮件
date: 2016-08-08 21:44:47
tags: Laravel
category: PHP
---

本文主要讲述了如何快速使用 Laravel 构建发送邮件的功能

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

## 编写发送文字邮件程序

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
/* 发送文字邮件 */
Route::get('email/send/{id}', [
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

        Mail::send('emails.test', ['user' => $user], function ($email) use ($user) {
            $email->to('2794408425@qq.com')->subject('Hello World');
        });
    }
}

```

### 视图

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

### 执行代码
在浏览器访问指定路由，然后去邮箱查看邮件是否发送成功。
![发送文字邮件](http://o93kt6djh.bkt.clouddn.com/Laravel-SendEmail%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-08-09%2009.49.12.png)


## 编写发送图文邮件程序

### 路由
```php
Route::get('email/sendPicture/{id}', [
    'as'   => 'backend.email.sendPicture',
    'uses' => 'EmailController@sendPicture',
]);
```

### 控制器
```
public function sendPicture(Request $request, $id)
{
    $user = UserRepository::find($id);
    $icon = "http://o93kt6djh.bkt.clouddn.com/Laravel-SendEmaillaravel-200x50.png";
    $image = "http://o93kt6djh.bkt.clouddn.com/Laravel-SendEmaillaravel-600x300.jpg";

    Mail::send('emails.image', ['name' => $user->name, 'icon' => $icon, 'image' => $image], function ($email) {
        $someOne = '2794408425@qq.com';
        $email->to($someOne)->subject('图文邮件标题');
    });

    echo "已发送邮件,请注意查收";
}
```

### 视图
新增视图`emails/image.blade.php`，代码如下：
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
    <meta name="viewport" content="width=device-width"/>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
    <title>CowCat Email Test</title>
    <style type="text/css">
        * {
            margin: 0;
            padding: 0;
        }

        * {
            font-family: "Helvetica Neue", "Helvetica", Helvetica, Arial, sans-serif;
        }

        img {
            max-width: 100%;
        }

        .collapse {
            margin: 0;
            padding: 0;
        }

        body {
            -webkit-font-smoothing: antialiased;
            -webkit-text-size-adjust: none;
            width: 100% !important;
            height: 100%;
        }

        /* -------------------------------------
                ELEMENTS
        ------------------------------------- */
        a {
            color: #2BA6CB;
        }

        .btn {
            text-decoration: none;
            color: #FFF;
            background-color: #666;
            padding: 10px 16px;
            font-weight: bold;
            margin-right: 10px;
            text-align: center;
            cursor: pointer;
            display: inline-block;
        }

        p.callout {
            padding: 15px;
            background-color: #ECF8FF;
            margin-bottom: 15px;
        }

        .callout a {
            font-weight: bold;
            color: #2BA6CB;
        }

        table.social {
            /*  padding:15px; */
            background-color: #ebebeb;

        }

        .social .soc-btn {
            padding: 3px 7px;
            font-size: 12px;
            margin-bottom: 10px;
            text-decoration: none;
            color: #FFF;
            font-weight: bold;
            display: block;
            text-align: center;
        }

        a.fb {
            background-color: #3B5998 !important;
        }

        a.tw {
            background-color: #1daced !important;
        }

        a.gp {
            background-color: #DB4A39 !important;
        }

        a.ms {
            background-color: #000 !important;
        }

        .sidebar .soc-btn {
            display: block;
            width: 100%;
        }

        /* -------------------------------------
                HEADER
        ------------------------------------- */
        table.head-wrap {
            width: 100%;
        }

        .header.container table td.logo {
            padding: 15px;
        }

        .header.container table td.label {
            padding: 15px;
            padding-left: 0px;
        }

        /* -------------------------------------
                BODY
        ------------------------------------- */
        table.body-wrap {
            width: 100%;
        }

        /* -------------------------------------
                FOOTER
        ------------------------------------- */
        table.footer-wrap {
            width: 100%;
            clear: both !important;
        }

        .footer-wrap .container td.content p {
            border-top: 1px solid rgb(215, 215, 215);
            padding-top: 15px;
        }

        .footer-wrap .container td.content p {
            font-size: 10px;
            font-weight: bold;

        }

        /* -------------------------------------
                TYPOGRAPHY
        ------------------------------------- */
        h1, h2, h3, h4, h5, h6 {
            font-family: "HelveticaNeue-Light", "Helvetica Neue Light", "Helvetica Neue", Helvetica, Arial, "Lucida Grande", sans-serif;
            line-height: 1.1;
            margin-bottom: 15px;
            color: #000;
        }

        h1 small, h2 small, h3 small, h4 small, h5 small, h6 small {
            font-size: 60%;
            color: #6f6f6f;
            line-height: 0;
            text-transform: none;
        }

        h1 {
            font-weight: 200;
            font-size: 44px;
        }

        h2 {
            font-weight: 200;
            font-size: 37px;
        }

        h3 {
            font-weight: 500;
            font-size: 27px;
        }

        h4 {
            font-weight: 500;
            font-size: 23px;
        }

        h5 {
            font-weight: 900;
            font-size: 17px;
        }

        h6 {
            font-weight: 900;
            font-size: 14px;
            text-transform: uppercase;
            color: #444;
        }

        .collapse {
            margin: 0 !important;
        }

        p, ul {
            margin-bottom: 10px;
            font-weight: normal;
            font-size: 14px;
            line-height: 1.6;
        }

        p.lead {
            font-size: 17px;
        }

        p.last {
            margin-bottom: 0px;
        }

        ul li {
            margin-left: 5px;
            list-style-position: inside;
        }

        /* -------------------------------------
                SIDEBAR
        ------------------------------------- */
        ul.sidebar {
            background: #ebebeb;
            display: block;
            list-style-type: none;
        }

        ul.sidebar li {
            display: block;
            margin: 0;
        }

        ul.sidebar li a {
            text-decoration: none;
            color: #666;
            padding: 10px 16px;
            /*  font-weight:bold; */
            margin-right: 10px;
            /*  text-align:center; */
            cursor: pointer;
            border-bottom: 1px solid #777777;
            border-top: 1px solid #FFFFFF;
            display: block;
            margin: 0;
        }

        ul.sidebar li a.last {
            border-bottom-width: 0px;
        }

        ul.sidebar li a h1, ul.sidebar li a h2, ul.sidebar li a h3, ul.sidebar li a h4, ul.sidebar li a h5, ul.sidebar li a h6, ul.sidebar li a p {
            margin-bottom: 0 !important;
        }

        /* ---------------------------------------------------
                RESPONSIVENESS
                Nuke it from orbit. It's the only way to be sure.
        ------------------------------------------------------ */

        /* Set a max-width, and make it display as block so it will automatically stretch to that width, but will also shrink down on a phone or something */
        .container {
            display: block !important;
            max-width: 600px !important;
            margin: 0 auto !important; /* makes it centered */
            clear: both !important;
        }

        /* This should also be a block element, so that it will fill 100% of the .container */
        .content {
            padding: 15px;
            max-width: 600px;
            margin: 0 auto;
            display: block;
        }

        /* Let's make sure tables in the content area are 100% wide */
        .content table {
            width: 100%;
        }

        /* Odds and ends */
        .column {
            width: 300px;
            float: left;
        }

        .column tr td {
            padding: 15px;
        }

        .column-wrap {
            padding: 0 !important;
            margin: 0 auto;
            max-width: 600px !important;
        }

        .column table {
            width: 100%;
        }

        .social .column {
            width: 280px;
            min-width: 279px;
            float: left;
        }

        /* Be sure to place a .clear element after each set of columns, just to be safe */
        .clear {
            display: block;
            clear: both;
        }

        /* -------------------------------------------
                PHONE
                For clients that support media queries.
                Nothing fancy.
        -------------------------------------------- */
        @media only screen and (max-width: 600px) {

            a[class="btn"] {
                display: block !important;
                margin-bottom: 10px !important;
                background-image: none !important;
                margin-right: 0 !important;
            }

            div[class="column"] {
                width: auto !important;
                float: none !important;
            }

            table.social div[class="column"] {
                width: auto !important;
            }

        }
    </style>
</head>

<body bgcolor="#FFFFFF">

<!-- HEADER -->
<table class="head-wrap" bgcolor="#999999">
    <tr>
        <td></td>
        <td class="header container">

            <div class="content">
                <table bgcolor="#999999">
                    <tr>
                        <td><img src="{{$icon}}"/></td>
                        <td align="right"><h6 class="collapse">Test</h6></td>
                    </tr>
                </table>
            </div>

        </td>
        <td></td>
    </tr>
</table><!-- /HEADER -->


<!-- BODY -->
<table class="body-wrap">
    <tr>
        <td></td>
        <td class="container" bgcolor="#FFFFFF">

            <div class="content">
                <table>
                    <tr>
                        <td>

                            <h3>Welcome, {{$name}}</h3>
                            <p class="lead">This is a Test Email of the CowCat</p>

                            <!-- A Real Hero (and a real human being) -->
                            <p><img src="{{$image}}"/></p><!-- /hero -->

                            <!-- Callout Panel -->
                            <p class="callout">
                                Laravel is the best PHP Framework
                                <a href="http://laravel-china.org/docs/5.1">Learn it Now! &raquo;</a>
                            </p><!-- /Callout Panel -->

                            <h3>Hello
                                <small>World</small>
                            </h3>
                            <p>Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor
                                incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud
                                exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure
                                dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur.
                                Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt
                                mollit anim id est laborum.</p>
                            <a class="btn">Click Me!</a>

                            <br/>
                            <br/>

                            <!-- social & contact -->
                            <table class="social" width="100%">
                                <tr>
                                    <td>

                                        <!--- column 1 -->
                                        <table align="left" class="column">
                                            <tr>
                                                <td>

                                                    <h5 class="">Connect with Us:</h5>
                                                    <p class=""><a href="#" class="soc-btn fb">Facebook</a>
                                                        <a href="#" class="soc-btn tw">Twitter</a>
                                                        <a href="#" class="soc-btn gp">Google+</a></p>


                                                </td>
                                            </tr>
                                        </table><!-- /column 1 -->

                                        <!--- column 2 -->
                                        <table align="left" class="column">
                                            <tr>
                                                <td>

                                                    <h5 class="">Contact Info:</h5>
                                                    <p>Phone: <strong>408.341.0600</strong><br/>
                                                        Email:
                                                        <strong><a href="emailto:hseldon@trantor.com">hseldon@trantor
                                                                .com</a></strong></p>

                                                </td>
                                            </tr>
                                        </table><!-- /column 2 -->

                                        <span class="clear"></span>

                                    </td>
                                </tr>
                            </table><!-- /social & contact -->


                        </td>
                    </tr>
                </table>
            </div>

        </td>
        <td></td>
    </tr>
</table><!-- /BODY -->

<!-- FOOTER -->
<table class="footer-wrap">
    <tr>
        <td></td>
        <td class="container">

            <!-- content -->
            <div class="content">
                <table>
                    <tr>
                        <td align="center">
                            <p>
                                <a href="#">Terms</a> |
                                <a href="#">Privacy</a> |
                                <a href="#">
                                    <unsubscribe>Unsubscribe</unsubscribe>
                                </a>
                            </p>
                        </td>
                    </tr>
                </table>
            </div><!-- /content -->

        </td>
        <td></td>
    </tr>
</table><!-- /FOOTER -->

</body>
</html>
```

### 执行代码
在浏览器访问指定路由，然后去邮箱查看邮件是否发送成功。
![发送图文邮件](http://o93kt6djh.bkt.clouddn.com/Laravel-SendEmail%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-08-09%2016.46.51.png)

## 编写发送附件邮件程序

## 路由

```
Route::get('email/sendFile/{id}', [
    'as'   => 'backend.email.sendFile',
    'uses' => 'EmailController@sendFile',
]);
```

## 控制器
```
public function sendFile(Request $request, $id)
{
    $user = UserRepository::find($id);
    $icon = "http://o93kt6djh.bkt.clouddn.com/Laravel-SendEmaillaravel-200x50.png";

    Mail::send('emails.test-file', ['name' => $user->name, 'icon' => $icon], function ($message) {
        $someOne = "2794408425@qq.com";
        $file = storage_path("app/PHP7介绍和应用实践.pdf");
        $message->to($someOne)->subject("附件邮件标题");
        $message->attach($file, ['as' => 'PHP7 Introduction and Application Action.pdf', 'mime' => 'application/pdf']);
    });


    echo '已发送邮件,请注意查收';
}
```

## 视图
新增视图`emails/test-file.blade.php`，代码如下：
```html
<html>
    <head>
        <title></title>
    </head>
    <body>
        <p>
            你好,{{$name}},这是一封测试邮件。
            下面将会显示一张原始数据图片
        </p>
        <p>
            <img src="{{$message->embed($icon)}}" alt="原始数据图片">
        </p>
    </body>
</html>
```

## 执行代码
在浏览器访问指定路由，然后去邮箱查看邮件是否发送成功。
![发送附件邮件](http://o93kt6djh.bkt.clouddn.com/Laravel-SendEmail%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-08-10%2013.40.54.png)
