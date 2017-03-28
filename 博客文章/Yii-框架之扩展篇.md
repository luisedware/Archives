---
title: Yii 框架之扩展篇
date: 2016-10-29 16:13:23
tags: Yii
category: PHP
---


## Yii 扩展分类

- 模块化
- 事件机制
- MixIN （混合、多重继承）
- 依赖注入


## 模块化技术

模块化：通过对业务详细拆分，分化出不同的小模块，
可以通过思维导图进行梳理
系统加载模块时通过配置文件进行控制，若模块暂不可用可于配置中标明以通知系统模块暂不可用

使用 Gii 的 Module Generator

模块化实现：<br>
1、父级模块化（1级）<br>
   通过gii的modules生成对应的子模块（2级），<br>
      然后修改config.php中的web.php的配置信息，添加以下信息(例子)：<br>
      'modules'=>['article'=>'app/modules/article/Article']<br>
   访问方式，例如：localhost/basic/index.php?r=article/default/index
2、2级模块化（1级），类推<br>
   通过gii的modules生成对应的子模块（3级），<br>
      然后修改该模块下的.php配置信息，添加以下信息(例子)：<br>
         在actionInit中添加如下信息：<br>
          $this->modules = ['category'=>['class'=>'app/modules/article/modules/test/Test']];
          访问方式，例如：localhost/basic/index.php?r=article/test/default/index

## 事件机制

<?php
namespace app\controllers;
use yii\web\Controller;
use vendor\animal\Cat;
use vendor\animal\Mourse;
use vendor\animal\Dog;
use \yii\base\Event;
class AnimalController extends Controller
{
    public function actionIndex(){
        $cat = new Cat();
        $cat2 = new Cat();
        $mourse = new Mourse();
        $dog = new Dog();
/*
        //案例一
        //实现猫叫，老鼠跑绑定事件,dog关注
        $cat->on('miao',[$mourse,'run']);  //on()方法来自于Componment,on()方法实现绑定事件
        //猫叫的时候给老鼠传递一些参数信息

        $cat->on('miao',[$dog,'look']);//dog关注cat的miao
#-*/
/*
        //案例二
        //实现猫叫，老鼠跑绑定事件,dog关注后,取消关注
        $cat->on('miao',[$mourse,'run']);  //on()方法来自于Componment,on()方法实现绑定事件
        //猫叫的时候给老鼠传递一些参数信息

        $cat->on('miao',[$dog,'look']);//dog关注cat的miao
        $cat->off('miao',[$dog,'look']);//dog取消关注cat的miao
#---*/


## MixIn

Behavior

## 依赖注入


