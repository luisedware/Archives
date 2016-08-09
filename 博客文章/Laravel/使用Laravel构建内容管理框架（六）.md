---
title: 使用 Laravel 构建内容管理框架（六）
date: 2016-04-04 09:33:28
tags: Laravel
category: PHP
---

完成Entrust的配置

# 安装zizaco/entrust Package

***

zizaco/entrust是Laravel下基于角色的权限管理方案。

1.修改文件`composer.json`
```
"require": {
    "php": ">=5.5.9",
    "laravel/framework": "5.1.*",
    "gregwar/captcha": "1.*",
    "zizaco/entrust": "dev-laravel-5"
},
```

2.在终端运行命令
```
composer install

// or

composer update
```

3.在文件`config/app.php`的`providers`数组和`aliase`数组分别添加
```
// providers
Zizaco\Entrust\EntrustServiceProvider::class

// aliase
'Entrust' => Zizaco\Entrust\EntrustFacade::class,
```


4.在文件`app/Http/Kernel.php`的`$routeMiddleware`添加
```
'role' => Zizaco\Entrust\Middleware\EntrustRole::class,
'permission' => Zizaco\Entrust\Middleware\EntrustPermission::class,
'ability' => Zizaco\Entrust\Middleware\EntrustAbility::class,
```

5.Entrust会根据文件`config/auth.php`的`model`来选择用户模型，如下：
```
'model' => App\Models\User::class
```

执行以下命令生成配置文件`config/entrust.php`
```
php artisan vendor:publish
```
修改配置文件`entrust.php`如下:
```
'role'=>'App\Models\Role'
'permission'=>'App\Models\Permission'
```

6.生成数据库迁移文件并执行
```
php artisan entrust:migration
php artisan migrate
```

7.生成自动加载
```
composer dump-autoload
```

# 管理模型

***

为了使用Entrust，我们需要新建两个模型，在终端运行以下命令：
```
// 角色模型
php artisan make:model Models/Role

// 权限模型
php artisan make:model Models/Permission
```

## Role.php
```
<?php
namespace App\Models;

use Zizaco\Entrust\EntrustRole;

class Role extends EntrustRole
{
    protected $guarded = [];
}
```

## Permission.php
```
<?php
namespace App\Models;

use Zizaco\Entrust\EntrustPermission;

class Permission extends EntrustPermission
{
    protected $guarded = [];
}
```

为了配合Entrust的使用，需要修改User.php的代码
```
<?php

namespace App\Models;

use Illuminate\Auth\Authenticatable;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Auth\Passwords\CanResetPassword;
use Illuminate\Foundation\Auth\Access\Authorizable;
use Illuminate\Contracts\Auth\Authenticatable as AuthenticatableContract;
use Illuminate\Contracts\Auth\Access\Authorizable as AuthorizableContract;
use Illuminate\Contracts\Auth\CanResetPassword as CanResetPasswordContract;
use Zizaco\Entrust\Traits\EntrustUserTrait;

class User extends Model implements AuthenticatableContract,
    AuthorizableContract,
    CanResetPasswordContract
{
    use Authenticatable, Authorizable, CanResetPassword, EntrustUserTrait{
        Authorizable::can insteadof  EntrustUserTrait;
        EntrustUserTrait::can as hasPermission;
    }

    /**
     * The database table used by the model.
     *
     * @var string
     */
    protected $table = 'users';

    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = ['name', 'email', 'password'];

    /**
     * The attributes excluded from the model's JSON form.
     *
     * @var array
     */
    protected $hidden = ['password', 'remember_token'];
}
```

完成上述所有操作之后，Entrust的配置就完成了。
