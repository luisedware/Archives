---
title: 使用 Laravel 与 Vue.js 进行微信开发（三）
date: 2017-01-16 23:04:33
category: PHP
tags: Laravel
---

## 目标

本文的目标是实现微信公众号菜单的创建与删除


## 创建路由

```php
/* 微信菜单创建 */
Route::get('/create-menus', [
    'as' => 'wechat.create.menus',
    'uses' => 'WeChatServerController@createMenus',
]);

/* 微信菜单创建 */
Route::get('/destroy-menus', [
    'as' => 'wechat.destroy.menus',
    'uses' => 'WeChatServerController@destroyMenus',
]);
```

## 修改控制器

修改控制器 app/Http/Controllers/WeChat/WeChatServerController.php，新增代码如下：
```
/**
 * 创建新的微信菜单
 * @param WeChat $weChat
 * @return string
 */
public function createMenus(WeChat $weChat)
{
    $menu = $weChat->menu;
    $buttons = [
        [
            "type" => "view",
            "name" => "进入好答",
            "url" => route('wechat.index'),
        ],
        [
            "type" => "click",
            "name" => "听我唱歌",
            "key" => "K0001_LET_ME_SING_MY_SONG",
        ],
    ];

    $json = $menu->add($buttons);
    $result = json_decode($json, true);

    if ($result['errcode'] == 0 && $result['errmsg'] == 'ok') {
        echo "创建菜单成功";
    } else {
        echo "创建菜单失败";
    }
}

/**
 * 删除所有微信菜单
 * @param WeChat $weChat
 * @return string
 */
public function destroyMenus(WeChat $weChat)
{
    $result = $weChat->menu->destroy();

    if ($result['errcode'] == 0 && $result['errmsg'] == 'ok') {
        echo "删除菜单成功";
    } else {
        echo "删除菜单失败";
    }
}
```
