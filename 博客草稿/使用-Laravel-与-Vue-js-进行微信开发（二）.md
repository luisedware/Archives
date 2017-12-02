---
title: 使用 Laravel 与 Vue.js 进行微信开发（二）
date: 2017-01-16 20:42:22
category: PHP
tags: Laravel
---

## 目标

本文的目标是实现微信用户授权，当用户进入 Web App 的时候，能够获取用户的信息。


## 配置回调地址

微信公众平台 -> 开发者工具 -> 公众平台测试帐号 -> 测试号管理 -> 体验接口权限表 -> 功能服务 -> 网页帐号 -> 修改 -> 填写回调域名，格式如：www.baidu.com

## 修改配置文件
可以修改文件 config/wechat.php ，也可以在 .env 文件进行配置

wechat.php 文件配置
```
/*
 * OAuth 配置
 *
 * only_wechat_browser: 只在微信浏览器跳转
 * scopes：公众平台（snsapi_userinfo / snsapi_base），开放平台：snsapi_login
 * callback：OAuth授权完成后的回调页地址(如果使用中间件，则随便填写。。。)
 */
 'oauth' => [
     'only_wechat_browser' => false,
     'scopes'   => array_map('trim', explode(',', env('WECHAT_OAUTH_SCOPES', 'snsapi_userinfo'))),
     'callback' => env('WECHAT_OAUTH_CALLBACK', '/examples/oauth_callback.php'),
 ],
```

.env 文件配置
```
WECHAT_APPID=****
WECHAT_SECRET=***
WECHAT_TOKEN=****
WECHAT_AES_KEY=

WECHAT_OAUTH_SCOPES=snsapi_userinfo
WECHAT_OAUTH_CALLBACK=/wechat/oauth-callback
```

## 新增路由

修改文件 app/Http/Routes/wechat.php，代码如下：
```php
<?php
/* 处理微信发送消息 */
Route::any('/request-handler', 'WeChatServerController@requestHandler');

/* 处理微信授权回调 */
Route::any('/oauth-callback', 'WeChatServerController@oauthCallback');

/* 微信应用首页 */
Route::get('/', [
    'as' => 'wechat.index',
    'uses' => 'WeChatApplicationController@index',
    'middleware' => [],
]);
```

## 关闭路由 CSRF
修改文件 app/Http/Middleware/VerifyCSsrfToken.php，代码如下：
```php
protected $except = [
	'/wechat/request-handler',
	'/wechat/oauth-callback'
];
```


## 获取授权

### 1.授权页面

新增文件 app/Http/Controllers/WeChat/WeChatApplicationController.php，代码如下：
```php
<?php

namespace App\Http\Controllers\WeChat;

use Illuminate\Http\Request;
use App\Http\Controllers\Controller;
use EasyWeChat\Foundation\Application as WeChat;

class WeChatApplicationController extends Controller
{
    public function index(Request $request, WeChat $wechat)
    {
    	// 如果 Session 中没有用户信息，则跳转至微信授权页面
        if (empty($request->session()->has('weChatUserInfo'))) {
            $oauth = $wechat->oauth;

            return $oauth->redirect();
        }
        $user = $request->session()->get('weChatUserInfo');
        dump($user);
    }
}
```

### 2.回调操作

修改文件 app/Http/Controllers/WeChat/WeChatServerController.php，代码如下：
```php
<?php
/**
 * 微信用户授权回调
 * @param Request $request
 * @param WeChat $weChat
 * @return \Illuminate\Http\RedirectResponse
 */
public function oauthCallback(Request $request, WeChat $weChat)
{
    $oauth = $weChat->oauth;
    $user = $oauth->user();
    $request->session()->put('weChatUserInfo', $user->toArray());

    return redirect()->route('wechat.index');
}
```

### 3.访问首页

下载微信的 Web 开发者工具，在工具中访问授权首页，比如访问:你的域名/wechat,可以跳转至授权页，如下图：
![](http://o93kt6djh.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-01-16%2022.24.46.png)
![](http://o93kt6djh.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-01-16%2022.25.06.png)
