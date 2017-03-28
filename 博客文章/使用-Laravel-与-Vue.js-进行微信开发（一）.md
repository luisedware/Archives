---
title: 使用 Laravel 与 Vue.js 进行微信开发（一）
date: 2017-01-15 09:11:57
category: PHP
tags: Laravel
---

## 目标

本文的目标是实现与微信服务器的通信。

## 环境

- MacOS
- Laravel 5.1
- Vue.js 2.1
- EasyWeChat

## 安装 EasyWechat

```
composer require "overtrue/laravel-wechat:~3.0"
```

## 配置

- 注册 ServiceProvider:
	- Overtrue\LaravelWechat\ServiceProvider::class,
- 创建配置文件
	- php artisan vendor:publish
- 修改 config/wechat.php
	- app_id：微信测试号 appID
	- secret：微信测试号 appsecret
	- token：自定义，要与微信测试号填写的 Token 保持一致
	- aes_key：加密信息用的，本文采用明文的方式，暂不填写

## 创建微信分组路由

修改文件 app/Http/routes.php，代码如下：
```php
Route::group([
    'prefix' => 'wechat',
    'namespace' => 'WeChat',
    'middleware' => [],
], function() {
    require_once __DIR__ . '/Routes/wechat.php';
});
```

新增文件 app/Http/Routes/wechat.php，代码如下：
```php
Route::any('/request-handler', 'WeChatServerController@requestHandler');
```

## 关闭路由 CSRF
修改文件 app/Http/Middleware/VerifyCSsrfToken.php，代码如下：
```php
protected $except = [
	'/wechat/request-handler'
];
```

## 创建控制器

新增文件 app/Http/Controllers/WeChat/WeChatServerController.php，代码如下：
```php
<?php

namespace App\Http\Controllers\WeChat;

use Illuminate\Http\Request;
use App\Http\Controllers\Controller;
use EasyWeChat\Foundation\Application as WeChat;

class WeChatServerController extends Controller
{
    /**
     * 处理微信的请求消息
     * @return string
     */
    public function requestHandler(WeChat $weChat)
    {
        $weChat->server->setMessageHandler(function($message) {
            return "欢迎关注 {$message}!";
        });

        return $weChat->server->serve();
    }
}
```

## 验证

- 配置微信公众平台测试号
	- 接口配置信息
		- URL：域名/wechat
		- Token：自定义，要与 wechat.php 文件的 token 一致
	- JS 接口安全域名
		- 填写域名，例如 baidu.com ，不需要写 www 和 http[s]://
    - 测试号二维码
        - 添加测试用户

URL 和 Token 通过验证之后，关注微信公众平台测试号，并发送测试消息，成功的话，如下图：
![](http://o93kt6djh.bkt.clouddn.com/yanzhengwechatmessage.jpeg)

