---
title: 查看 Eloquent 模型关联时执行的 SQL 语句
date: 2016-09-01 20:34:56
tags: Laravel
category: PHP
---

本文目的是查看 Eloquent 在执行模型关联时运行的 SQL 语句


## 环境

- PHP 7
- Laravel 5.1


## 简介

数据表之间经常会互相进行关联。例如，一篇博客文章可能会有多条评论，或是一张订单可能对应一个下单客户。Eloquent 让管理和处理这些关联变得很容易，同时也支持多种类型的关联：

- 一对一
- 一对多
- 多对多
- 远层一对多
- 多态关联
- 多态多对多关联

接下来，我会根据公司项目的数据库结构，进行举例说明

## 准备

新增控制器
```shell
php artisan make:controller EloquentController
```

新增路由
```php
Route::get('/', function () {
    return view('welcome');
});

// 一对一
Route::get('eloquent/one-to-one', [
    'as'   => 'eloquent.one.to.one',
    'uses' => 'EloquentController@oneToOne',
]);

// 一对多
Route::get('eloquent/one-to-many', [
    'as'   => 'eloquent.one.to.many',
    'uses' => 'EloquentController@oneToMany',
]);

// 多对多
Route::get('eloquent/many-to-many', [
    'as'   => 'eloquent.many.to.many',
    'uses' => 'EloquentController@manyToMany',
]);

// 远程一对多
Route::get('eloquent/has-many-through', [
    'as'   => 'eloquent.has.many.through',
    'uses' => 'EloquentController@hasManyThrough',
]);

// 多态关联
Route::get('eloquent/polymorphic-relations', [
    'as'   => 'eloquent.polymorphic.relations',
    'uses' => 'EloquentController@polymorphicRelations',
]);

// 多对多多态管理
Route::get('eloquent/many-to-many-polymorphic-relations', [
    'as'   => 'eloquent.many.to.many.polymorphic.relations',
    'uses' => 'EloquentController@manyToManyPolymorphicRelations',
]);
```

## 一对一

场景是如每一名加盟商都对应着一份加盟商配置

新增模型
```
php artisan make:model Models/League
php artisan make:model Models/LeagueConfig
```

在 `League` 模型添加代码如下
```php
// League.php 一名加盟商有一份配置
public function config()
{
    return $this->hasOne('App\Models\LeagueConfig');
}
```

在 `LeagueConfig` 模型添加代码如下
```
// LeagueConfig.php 一份配置对应一名加盟商
public function league()
{
    return $this->hasOne('App\Models\League');
}
```

控制器新增代码如下
```php
public function oneToOne(Request $request)
{
    DB::enableQueryLog();
    $data = League::find(8)->config;
    foreach (DB::getQueryLog() as $sql) {
        dump($sql['query']);
    }
    dump($data);
}
```

查看 SQL
```sql
SELECT
	*
FROM
	`z_league`
WHERE
	`z_league`.`id` = 8
LIMIT 1;


SELECT
	*
FROM
	`z_leagueconfig`
WHERE
	`z_leagueconfig`.`league_id` = 8
AND `z_leagueconfig`.`league_id` IS NOT NULL
LIMIT 1;
```

## 一对多

场景是仓库一种 SKU 的水果能被多名加盟商进货

新增模型
```
php artisan make:model Models/Product
php artisan make:model Models/ProductLeague
```

`Product` 模型中添加代码
```php
 public function league()
{
    return $this->hasMany('App\Models\ProductLeague', 'pid');
}
```

控制器新增代码如下：
```
 public function oneToMany(Request $request)
{
    DB::enableQueryLog();
    $data = Product::find(3062)->league;
    foreach (DB::getQueryLog() as $sql) {
        dump($sql['query']);
    }

    dump($data);
}
```

查看 SQL
```sql
SELECT
	*
FROM
	`z_product`
WHERE
	`z_product`.`id` = 3062
LIMIT 1;

SELECT
	*
FROM
	`z_productleague`
WHERE
	`z_productleague`.`pid` = 3062
AND `z_productleague`.`pid` IS NOT NULL;
```

## 多对多

场景是一名用户使用多张不同种类的优惠券，一种优惠券被不同的用户使用

新增模型
```shell
php artisan make:model Models/User
php artisan make:model Models/Voucher
php artisan make:model Models/VoucherRecord
```

`User` 模型添加代码如下
```php
public function voucher()
{
    return $this->belongsToMany('App\Models\Voucher', 'z_voucher_record', 'uid', 'vid');
}
```

`Voucher` 模型添加代码如下
```php
public function user()
{
    return $this->belongsToMany('App\Models\User', 'z_voucher_record', 'vid', 'uid');
}
```

控制器新增代码如下
```
public function manyToMany(Request $request)
{
    DB::enableQueryLog();
    $vouchers = User::find(412115)->voucher;
    foreach (DB::getQueryLog() as $sql) {
        dump($sql['query']);
    }
    dump($vouchers);

    DB::enableQueryLog();
    $user = Voucher::find(395)->user;
    foreach (DB::getQueryLog() as $sql) {
        dump($sql['query']);
    }
    dump($user);
}
```

查看 SQL
```sql
# 查询 ID 为 42115 的用户使用过的优惠券
SELECT
	*
FROM
	`z_user`
WHERE
	`z_user`.`id` = 412115;

LIMIT 1 SELECT
	`z_voucher`.*, `z_voucher_record`.`uid` AS `pivot_uid` ,
	`z_voucher_record`.`vid` AS `pivot_vid`
FROM
	`z_voucher`
INNER JOIN `z_voucher_record` ON `z_voucher`.`id` = `z_voucher_record`.`vid`
WHERE
	`z_voucher_record`.`uid` = 412115;

# 查询 ID 为 395 的优惠券被哪些用户使用过

SELECT
	*
FROM
	`z_voucher`
WHERE
	`z_voucher`.`id` = 395;

LIMIT 1 SELECT
	`z_user`.*, `z_voucher_record`.`vid` AS `pivot_vid` ,
	`z_voucher_record`.`uid` AS `pivot_uid`
FROM
	`z_user`
INNER JOIN `z_voucher_record` ON `z_user`.`id` = `z_voucher_record`.`uid`
WHERE
	`z_voucher_record`.`vid` = 395;
```

## 远程一对多

新增模型
```
php artisan make:model Models/Order
```

`Order` 模型新增代码如下
```
 public function order()
{
    return $this->hasManyThrough('App\Models\Order', 'App\Models\User', 'league_id', 'uid');
}
```

控制器新增代码如下
```
public function hasManyThrough(Request $request)
{
    DB::enableQueryLog();
    $data = League::find(20)->order;
    foreach (DB::getQueryLog() as $sql) {
        dump($sql['query']);
    }

    dump($data);
}
```

查看 SQL
```sql
# 查询 ID 为 20 的加盟商旗下用户的商品订单
SELECT
	*
FROM
	`z_league`
WHERE
	`z_league`.`id` = 20
LIMIT 1;

SELECT
	`z_order`.*, `z_user`.`league_id`
FROM
	`z_order`
INNER JOIN `z_user` ON `z_user`.`id` = `z_order`.`uid`
WHERE
	`z_user`.`league_id` = 20;
```

## 多态关联

新增模型如下
```shell
php artisan make:model Models/Staff
php artisan make:model Models/photos
php artisan make:model Models/Products
```

控制器新增代码如下
```php
public function polymorphicRelations(Request $request)
{
    DB::enableQueryLog();
    $staff = Staff::find(8)->photos;
    foreach (DB::getQueryLog() as $sql) {
        dump($sql['query']);
    }

    dump($staff);

    DB::enableQueryLog();
    $products = Products::find(1)->photos;
    foreach (DB::getQueryLog() as $sql) {
        dump($sql['query']);
    }

    dump($products);
}
```

查看 SQL
```sql
# 查看 ID 为 8 的职员的所有照片
SELECT
	*
FROM
	`eloquent_staff`
WHERE
	`eloquent_staff`.`id` = 8
LIMIT 1;

SELECT
	*
FROM
	`eloquent_photos`
WHERE
	`eloquent_photos`.`imageable_id` = 8
AND `eloquent_photos`.`imageable_id` IS NOT NULL
AND `eloquent_photos`.`imageable_type` = "App\Models\Staff";

# 查看 ID 为 1 的产品的所有图片
SELECT
	*
FROM
	`eloquent_products`
WHERE
	`eloquent_products`.`id` = 1
LIMIT 1;

SELECT
	*
FROM
	`eloquent_photos`
WHERE
	`eloquent_photos`.`imageable_id` = 1
AND `eloquent_photos`.`imageable_id` IS NOT NULL
AND `eloquent_photos`.`imageable_type` = "App\Models\Products";
```

## 多态多对多关联

新增模型如下
```php
php artisan make:model Models/Tag
php artisan make:model Models/Post
php artisan make:model Models/Video
```

控制器新增代码如下
```php
public function manyToManyPolymorphicRelations(Request $request)
{
    DB::enableQueryLog();
    $post_tags = Post::find(8)->tags;
    foreach (DB::getQueryLog() as $sql) {
        dump($sql['query']);
    }

    dump($post_tags);

    DB::enableQueryLog();
    $video_tags = Video::find(8)->tags;
    foreach (DB::getQueryLog() as $sql) {
        dump($sql['query']);
    }

    dump($video_tags);

    DB::enableQueryLog();
    $posts = Tag::find(4)->posts;
    foreach (DB::getQueryLog() as $sql) {
        dump($sql['query']);
    }

    dump($posts);

    DB::enableQueryLog();
    $videos = Tag::find(4)->videos;
    foreach (DB::getQueryLog() as $sql) {
        dump($sql['query']);
    }

    dump($videos);
}
```

查看 SQL
```sql
# 查询 ID 为 8 的文章所使用的标签
SELECT
	*
FROM
	`eloquent_posts`
WHERE
	`eloquent_posts`.`id` = 8
LIMIT 1;

SELECT
	`eloquent_tags`.*, `eloquent_taggables`.`taggable_id` AS `pivot_taggable_id` ,
	`eloquent_taggables`.`tag_id` AS `pivot_tag_id`
FROM
	`eloquent_tags`
INNER JOIN `eloquent_taggables` ON `eloquent_tags`.`id` = `eloquent_taggables`.`tag_id`
WHERE
	`eloquent_taggables`.`taggable_id` = 8
AND `eloquent_taggables`.`taggable_type` = "App\Models\Post";

# 查询 ID 为 8 的视频所使用的标签
SELECT
	*
FROM
	`eloquent_videos`
WHERE
	`eloquent_videos`.`id` = 8
LIMIT 1;

SELECT
	`eloquent_tags`.*, `eloquent_taggables`.`taggable_id` AS `pivot_taggable_id` ,
	`eloquent_taggables`.`tag_id` AS `pivot_tag_id`
FROM
	`eloquent_tags`
INNER JOIN `eloquent_taggables` ON `eloquent_tags`.`id` = `eloquent_taggables`.`tag_id`
WHERE
	`eloquent_taggables`.`taggable_id` = 8
AND `eloquent_taggables`.`taggable_type` = "App\Models\Video";

# 查询 ID 为 4 的标签下的文章
SELECT
	*
FROM
	`eloquent_tags`
WHERE
	`eloquent_tags`.`id` = 4
LIMIT 1;

SELECT
	`eloquent_posts`.*, `eloquent_taggables`.`tag_id` AS `pivot_tag_id` ,
	`eloquent_taggables`.`taggable_id` AS `pivot_taggable_id`
FROM
	`eloquent_posts`
INNER JOIN `eloquent_taggables` ON `eloquent_posts`.`id` = `eloquent_taggables`.`taggable_id`
WHERE
	`eloquent_taggables`.`tag_id` = 4
AND `eloquent_taggables`.`taggable_type` = "App\Modes\Post";

# 查询 ID 为 4 的标签下的视屏

SELECT
	*
FROM
	`eloquent_tags`
WHERE
	`eloquent_tags`.`id` = 4
LIMIT 1;

SELECT
	`eloquent_videos`.*, `eloquent_taggables`.`tag_id` AS `pivot_tag_id` ,
	`eloquent_taggables`.`taggable_id` AS `pivot_taggable_id`
FROM
	`eloquent_videos`
INNER JOIN `eloquent_taggables` ON `eloquent_videos`.`id` = `eloquent_taggables`.`taggable_id`
WHERE
	`eloquent_taggables`.`tag_id` = 4
AND `eloquent_taggables`.`taggable_type` = "App\Models\Video";
```
