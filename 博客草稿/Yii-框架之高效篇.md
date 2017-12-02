---
title: Yii 框架之高效篇
date: 2016-10-25 20:42:56
tags: Yii
category: PHP
---

## Yii 的延迟加载

### 类的延迟加载
sql_autoload_register

Yii 如何使用 sql_autoload_register

自动加载文件路径：vendor/yiisoft/yii2/Yii.php
入口文件路径：web/index.php

### 类的映射表机制

yii2快速的机制：使用类的映射表，把常用的类放入映射表中，加快加载速度

映射表文件：vendor/yiisoft/yii2/classes.php
类的映射变量：Yii::$classMap
vendor/yiisoft/yii2/BaseYii.php

可以通过使用Yii::$classMap，对延迟加载机制进行优化，是典型的空间换时间的做法，所以不建议往classMap中放入太多不常用的内容，避免内存占用过多

只建议将常用的高频率class放进‘类的映射表’，不常用的就不要放，否则映射表大了，不但降低查询效率，还将占用不必要的内存。

### 组件的延迟加载

- index.php
	- app
		- application class
		- components
			- vendor/yiisoft/yii2/web/Application.php
			- vendor/yiisoft/yii2/base/Application.php
		- controller

## 数据缓存的使用

### 数据缓存的基础使用

如何配置缓存服务器

如何将数据写入缓存

如何将缓存数据读出

如何修改缓存的数据

如何删除缓存的数据

如何清空缓存中数据

如何设置缓存有效期

### 数据缓存的依赖关系详解

可以用来解决缓存实时更新的问题


文件依赖
yii框架缓存的DB依赖,跟其他缓存依赖一样,如果依赖中的sql取出的数据发生了改变,相关联的缓存就会消失


表达式依赖

1、文件依赖(FileDependency)：一旦文件改变，缓存将失效
$dependency=new \yii\caching\FileDependency(['filename'=>'hw.txt']);
$cache->add('file_key','hello word!',3000,$dependency);
2、表达式依赖(ExpressionDependency)：一旦表达式改变，缓存将失效，即参数发生了变化
$dependency=new \yii\caching\ExpressionDependency(['expression'=>'\YII::$app->request->get["name"]']);
3、DB依赖(DbDependency)：一旦数据改变，缓存将失效
$dependency=new \yii\caching\ExpressionDependency(
['sql'=>'select count(*) from user']);

4.链式依赖
5.组依赖

Yii 框架是如何监听文件改变、表达式改变和数据改变？

## 片段缓存的使用

//片段缓存介绍(主要负责把前端界面的一些区域[不会经常变动的区域：如京东商品分类]缓存起来[缓存到内存或文件中]，下次访问时直接从缓存里把数据拿出来，而不用再从数据库抓取信息，提高了程序的执行效率)。


```
<!-- 使用视图组件($this)里的beginCache('缓存数据的名字')方法把cache_div给缓存起来;这个方法开启会检查当前有没有缓存，如果没有返回false -->
<!-- 如何证明cache_div的代码片段被缓存了呢？ 在两个div里都添加上相同的内容。如果cache_div被缓存起来将会使用缓存里的内容，而不会使用修改后的内容。提高了程序的运行效率。 -->
<?php if($this->beginCache('cache_div')){?>
<div id='cache_div'>
	<div>这里待会会被缓存(这里是缓存过后添加的)</div>
</div>
<?php
	$this->endCache();//结束缓存
}
?>

<div id='no_cache_div'>
	<div>这里不会被缓存(这里是缓存过后添加的)</div>
</div>
```

片段缓存的设置

片段缓存的时间
$this->beginCache('cache_div',['duration'=>$duration]);
$this->endCache();

片段缓存的依赖
$this->beginCache('cache_div',['dependency'=>$dependency]);
$this->endCache();

片段缓存的开关
$this->beginCache('cache_div',['enabled'=>$enabled']);
$this->endCache();

片段缓存的嵌套使用

谨慎使用：外层的失效时间应该短于内层，外层的依赖条件应该低于内层，以确保最小的片段，返回的是最新的数据。

片段嵌套缓存  外面的缓存时间大于里面的  那么里面的数据即使超过了缓存时间后有修改也不会被执行

## 页面缓存的使用

```
public function actionIndex()
{

}

//页面访问index()方法之前会先访问behavior()方法行为。
public function behavior()
{
    return [
        [
            'class'      => 'yii\filters\PageCache',//页面缓存类
            'duration'   => 1000,//缓存时间
            'only'       => ['index', 'test'],//仅仅缓存index和test方法的数据]
            'dependency' => ['class' => 'yii\caching\FileDependency', 'fileName' => 'hw.txt']//文件缓存依赖
        ],
    ];
}
```



## HTTP 缓存的使用

如何校验服务器与浏览器的数据一致

用矩形L代表浏览器，矩形F代表服务器。
1.当用户到F浏览网站或一个页面时，就会用L给F发送一个请求，然后F就会处理这个请求，处理完之后会把处理的结果作为一个响应返回给L，然后L从这个响应里拿数据，再显示出来，这样用户就可以看到他想看到的页面了。
2.但这时会遇到一个小问题，如果用户从F拿过来的数据不是他想要的数据(比如12306订票页面想订的票没有了)，这时用户猛刷新L，那么L就会给F发送很多的请求，实际上F的数据是没有发生变化的，F收到这么多请求后都会照常的处理，但处理的结果没有发生变化，所以处理的结果跟第一次返回出去的数据是一样的。所以这样对F来说，很浪费资源、时间。
3.F想了一个办法，当F第一次把数据(小圆表示)发送给L时，它让L把数据缓存到L里面，也就是把它缓存起来，缓存起来以后如果下一次L发送了请求到了F这边之后，F会对比F这边的数据和L那边的数据是不是一样的(F是如何校验的呢？)，如果是一样的那就没必要再把这个数据发送回给L了，就让L使用缓存里的数据就行了。
4.F如何校验F和L的数据是一样的，F就想了一个办法。当F第一次发送数据给L时，会在数据中塞入一个last-modified字段(记录了F这边数据的修改时间)，那么L也有了这个字段。那么当L下一次发送请求时，会把这个修改时间也带上，也就是把lm(last-modified字段)发送回给F，F会校验lm和F里的那个数据的修改时间是否是一样的，如果是一样的那么这个数据肯定没有被修改过，那就没必要再发送回给L了，直接告诉L去使用缓存里的数据就行了。
5.另一种校验方式是基于last-modified的小缺陷设计出来的(开发人员把F的数据@改为#，然后又改回@，那么数据@虽然没变化，但它的修改时间改变了。那么L发送过来的lm和数据@的修改时间是对不上的，然后F又把@发送给L，但实际上@没有发生变化，这样也会造成一些问题)。所以F想了一个办法，F第一次发送@数据包时，会根据@里的内容生成一个标志etag，通过etag可以大概了解数据包@里的内容。F会把etag和last-modified一起发送给L，那么L下一次访问F时，会传lm和etag。那么F先验证lm(两边数据修改时间是否一样)，一样则使用L的缓存；如果不一样，F则验证etag是否一样，如果一样证明@没有被修改，则让L使用缓存


服务器响应304，not modifired

服务器怎么告诉浏览器缓存起来？

```
public function behaviors(){
	return [
		[
			'class'=>'yii\filters\HttpCache',
			'lastModified'=>function(){
				return 1432817564;
			},
			'etagSeed'=>function(){
				return 'etagseed2f;'
			}
		]
	];
}
```
//使用 HttpCache 之后再请求头部header会多了一个Cache-Control:last-modified 来作为标志



在服务器修改数据之后如何让浏览器更新缓存

httpCache的依赖:
lastModified: 时间标识
etagSeed：内容标识
当二者同时存时，需要两个均发生变化才会发送200并返回新数据


## gii 工具简介

- Model Generator（数据模型的代码生成器）
- Form Generator（表单的代码生成器）
- Module Generator（模块的代码生成器）
- CRUD Generator（增删改查的代码生成器）
- Controller Generator（控制器的代码生成器）
- Extension Generator（扩展的代码生成器）
