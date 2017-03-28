---
title: 关于 PHP 函数 array_filter 的一些想法
date: 2016-11-21 16:00:39
tags: PHP
category: PHP
---

## array_filter

(PHP 4 >= 4.0.6, PHP 5, PHP 7)
array_filter — 用回调函数过滤数组中的单元

## 说明

`array array_filter ( array $array [, callable $callback [, int $flag = 0 ]] )`

依次将 array 数组中的每个值传递到 callback 函数。如果 callback 函数返回 TRUE，则 input 数组的当前值会被包含在返回的结果数组中。数组的键名保留不变。

## 参数

- array
	- 要循环的数组
- callback
	- 使用的回调函数
	- 如果没有提供 callback 函数， 将删除 input 中所有等值为 FALSE 的条目。更多信息见转换为布尔值。
- flag
	- 决定callback接收的参数形式:
		- ARRAY_FILTER_USE_KEY - callback接受键名作为的唯一参数
		- ARRAY_FILTER_USE_BOTH - callback同时接受键名和键值

## 返回值

返回过滤后的数组。


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

$data = [
    'int'=>0,
    'null'=>null,
    'bool'=>false,
    'string'=>'hello',
    'number'=>506510463,
    'int-string'=>'0',
    'null-string'=>'',
];
var_dump(array_filter($data));

var_dump(array_filter($data, function ($value) { if (!empty($value)) {
     return $value;
 }}));

var_dump(array_filter($data, 'strlen'));

var_dump(array_map('intval', $data));

var_dump(array_map(function ($value) { if (!empty($value)) {
    return $value;
}}, $data));

var_dump(array_filter(array_map(function ($value) { if ($value['status']===1) {
     return $value['brand'].$value['model'];
 }}, $graphics_card)));
```
