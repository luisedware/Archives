---
title: 关于 PHP 函数 array_walk 的一些想法
date: 2016-11-22 11:21:28
tags: PHP
category: PHP
---

## array_walk

(PHP 4, PHP 5, PHP 7)
array_walk — 使用用户自定义函数对数组中的每个元素做回调处理

## 说明

`bool array_walk ( array &$array , callable $callback [, mixed $userdata = NULL ] )`

将用户自定义函数 funcname 应用到 array 数组中的每个单元。

array_walk() 不会受到 array 内部数组指针的影响。array_walk() 会遍历整个数组而不管指针的位置。

## 参数

- array
	- 输入的数组。
- callback
	- 典型情况下 callback 接受两个参数。array 参数的值作为第一个，键名作为第二个。
- Note:
	- 如果 callback 需要直接作用于数组中的值，则给 callback 的第一个参数指定为引用。这样任何对这些单元的改变也将会改变原始数组本身。
- Note:
	- 参数数量超过预期，传入内置函数 (例如 strtolower())， 将抛出警告，所以不适合当做 funcname。
	只有 array 的值才可以被改变，用户不应在回调函数中改变该数组本身的结构。例如增加/删除单元，unset 单元等等。如果 array_walk() 作用的数组改变了，则此函数的的行为未经定义，且不可预期。
- userdata
	- 如果提供了可选参数 userdata，将被作为第三个参数传递给 callback funcname。

## 返回值

成功时返回 TRUE， 或者在失败时返回 FALSE。


## 范例
```php
<?php
$graphics_card = [
    [
        'brand'=>'华硕',
        'model'=>'GTX980(Ti)',
        'status'=>1,
    ],
    [
        'brand'=>'技嘉',
        'model'=>'GTX970',
        'status'=>0,
    ],
    [
        'brand'=>'微星',
        'model'=>'GTX960',
        'status'=>1,
    ],
    [
        'brand'=>'七彩虹',
        'model'=>'GTX950',
        'status'=>0,
    ],
];

array_walk($graphics_card, function(&$item){
    $item['brand'] = '按摩店';
    $item['model'] = '1080';
});
var_dump($graphics_card);

array_map(function($key) use (&$graphics_card) {
    $graphics_card[$key]['brand'] = '英伟达';
    $graphics_card[$key]['model'] = '泰坦';
}, array_keys($graphics_card));
var_dump($graphics_card);

```
