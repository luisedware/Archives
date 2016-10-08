---
title: 使用 Laravel 编写自定义命令
date: 2016-08-25 21:48:36
tags: Laravel
category: PHP
---

本文讲述了如何使用 Laravel 开发命令行

## 环境

- PHP 7
- Laravel 5.1
- OS X El Capitan 10.11.4

## 简介

Artisan 是 Laravel 的命令行接口的名称，它提供了许多实用的命令来帮助你开发 Laravel 应用，它由强大的 Symfony Console 组件所驱动。

## 文档

 [Artisan](http://laravel-china.org/docs/5.1/artisan)


## 目的

本文目的旨在实现三个简单的命令，分别是优化项目、清除缓存和发送邮件


## 实现步骤

### 新建命令
```
php artisan make:console LaravelOptimize --command=laravel:optimize
php artisan make:console LaravelClean --command=laravel:clean
php artisan make:console SendEmails --command=emails:send
```

### 注册命令
打开文件 `app/Console/Kernel.php`，修改代码如下：
```
protected $commands = [
    \App\Console\Commands\Inspire::class,
    \App\Console\Commands\SendEmails::class,
    \App\Console\Commands\LaravelClean::class,
    \App\Console\Commands\LaravelOptimize::class,
];
```

### 编写命令执行代码

文件 `SendEmails.php` 代码如下：
```
<?php

namespace App\Console\Commands;

use App\Facades\UserRepository;
use Illuminate\Console\Command;
use Mail;

class SendEmails extends Command
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'emails:send';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = 'use SendCloud to send emails';

    /**
     * Create a new command instance.
     *
     * @return void
     */
    public function __construct()
    {
        parent::__construct();
    }

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $this->line("即将执行邮件发送命令");

        if($this->confirm('确定执行?')){
            $users = UserRepository::all();

            $bar = $this->output->createProgressBar(count($users));

            foreach ($users as $user) {
                $icon = "http://o93kt6djh.bkt.clouddn.com/Laravel-SendEmaillaravel-200x50.png";
                $image = "http://o93kt6djh.bkt.clouddn.com/Laravel-SendEmaillaravel-600x300.jpg";

                Mail::send('emails.test-image',
                    [
                        'name'  => $user->name,
                        'icon'  => $icon,
                        'image' => $image,
                    ],
                    function ($email) use ($user) {
                        $email->to($user->email)->subject('图文邮件标题');
                    });

                $bar->advance();
            }

            $bar->finish();

            $this->info("发送邮件完毕");
        } else {
            $this->info("取消执行邮件发送命令");
        }
    }
}
```

文件 `LaravelOptimize.php` 代码如下：
```
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Support\Facades\Artisan;

class LaravelOptimize extends Command
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'laravel:optimize';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = 'optimize laravel project';

    /**
     * Create a new command instance.
     *
     * @return void
     */
    public function __construct()
    {
        parent::__construct();
    }

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $this->line("即将执行优化缓存命令");
        if($this->confirm('确定执行?')){
            $this->line("开始执行缓存命令");

            Artisan::call("config:cache");
            $this->info('Configuration cache cleared!');
            $this->info('Configuration cached successfully!');

            Artisan::call("route:cache");
            $this->info('Route cache cleared!');
            $this->info('Routes cached successfully!');

            Artisan::call("optimize");
            $this->info('Generating optimized class loader');

            if(function_exists('exec')){
                exec("composer dump-autoload");
            }

            Artisan::call("ide-helper:generate");
            $this->info('A new helper file was written to _ide_helper.php');

            $this->info("优化缓存成功");
        } else {
            $this->info("取消执行优化缓存命令");
        }
    }
}
```

文件 `LaravelClean.php` 代码如下：
```
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Support\Facades\Artisan;

class LaravelClean extends Command
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'laravel:clean';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = 'clean project all cache';

    /**
     * Create a new command instance.
     *
     * @return void
     */
    public function __construct()
    {
        parent::__construct();
    }

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $this->line("即将执行清除缓存命令");
        if($this->confirm('确定执行?')){
            $this->line("开始执行缓存命令");

            Artisan::call("clear-compiled");

            Artisan::call("auth:clear-resets");
            $this->info('Expired reset tokens cleared!');

            Artisan::call("cache:clear");
            $this->info('Application cache cleared!');

            Artisan::call("config:clear");
            $this->info('Configuration cache cleared!');

            Artisan::call("debugbar:clear");
            $this->info('Debugbar Storage cleared!');

            Artisan::call("route:clear");
            $this->info('Route cache cleared!');

            Artisan::call("view:clear");
            $this->info('Compiled views cleared!');

            $this->info("清除缓存成功");
        } else {
            $this->info("取消执行清除缓存命令");
        }
    }
}
```

## 执行效果

在终端分别执行以下代码：
```
php artisan laravel:optimize
php artisan laravel:clean
php artisan emails:send
```

执行效果分别如下显示：

#### php artisan laravel:optimize
![执行缓存清楚](http://o93kt6djh.bkt.clouddn.com/laravel-optimize.png)

#### php artisan laravel:clean
![执行缓存清楚](http://o93kt6djh.bkt.clouddn.com/laravel-clean.png)

#### php artisan emails:send
![执行邮件发送](http://o93kt6djh.bkt.clouddn.com/email-send.png)


