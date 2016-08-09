---
title: 使用 PHPStorm 与 Xdebug 调试 Laravel (二)
date: 2016-06-27 10:36:54
tags: PHPStorm
category: Tooling
---

## 环境

- 系统版本：OSX 10.11.4
- PHP 版本：7.0.5
- Xdebug 版本：2.4.0
- Laravel 版本：5.1.31
- PHPStorm 版本：10.0.4


根据上篇文章的配置，在工作时会发现，我们需要经常调整 `PHP Web Application` 的 URL 进行 Debug。

举个例子，假如想要 `Debug` 菜单列表，我需要修改成 `/menu/`，如果想要 `Debug` 新增菜单页面，我需要修改成 `/menu/create`。
![](http://o93kt6djh.bkt.clouddn.com/2-phpstorm-debug-web-applicationPHP%20Web%20Application.png)
这样进行 `Debug` 的过程十分烦琐，所以需要更加友好的操作方式，以便加快工作效率。


## PHPStorm 配置

打开 PHPStorm，打开配置面板
`Preferences => Language & Frameworks -> PHP -> Debug`。

点击蓝色链接 `Use debugger bookmarklets to initiate debugging from your favorite browser`。
![](http://o93kt6djh.bkt.clouddn.com/2-phpstorm-debug-web-applicationPHP%20Debug%20Configuration.png)

点击页面左下角的蓝色按钮，生成 PHPStorm Debug 的专属书签。
![](http://o93kt6djh.bkt.clouddn.com/2-phpstorm-debug-web-applicationPHPStorm%20Debug%20Config.png)

然后将生成好的 `DEBUG` 书签`Start debugger`、`Stop debugger`、`Debug this page` 拖动保存到浏览器的书签栏中，方便随时进行 `Debug`。
![](http://o93kt6djh.bkt.clouddn.com/2-phpstorm-debug-web-applicationBookMark.png)

监听浏览器的 `Debug` 操作，`Run -> Start Listening for PHP Debug Connections`

![](http://o93kt6djh.bkt.clouddn.com/2-phpstorm-debug-web-applicationListening%20PHP%20Debug.png)

然后在浏览器输入想要进行 `Debug` 的页面，然后点击书签栏的 `Start debugger`，刷新页面，就能在 `PHPStorm` 里面看见 `Debug` 的控制台了。
![](http://o93kt6djh.bkt.clouddn.com/2-phpstorm-debug-web-applicationDebug%20Page2.png)
![](http://o93kt6djh.bkt.clouddn.com/2-phpstorm-debug-web-applicationDebug%20Page.png)

PHPStorm 的 Debug 方式不仅仅局限于 Laravel 框架，同样适用于 ThinkPHP 与其他框架，也适用于原生的 PHP 代码。
学会使用这种方式之后，一般很少使用 `echo`，`var_dump`，`dd()`，`dump()`等原生或框架辅助函数进行 `Debug` 了。


