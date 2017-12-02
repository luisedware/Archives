---
title: 关于 PHP 函数 array_map 的一些想法
date: 2016-11-21 14:42:50
tags: PHP
category: PHP
---


## array_map

array_map - 为数组的每个元素应用回调函数

## 说明

`array array_map(callable $callback, array $array1[, array $...])`

array_map()：返回数组，是为 array1 每个元素应用 callback函数之后的数组。 callback 函数形参的数量和传给 array_map() 数组数量，两者必须一样。

## 参数

- callback
	- 回调函数，应用到每个数组里的元素
- array1
	- 数组，遍历运行 callback 函数
- ...
	- 数据列表，每个都遍历运行 callback 函数

## 范例

### array_map 实现 foreach 的效果
```php
<?php

class GraphicsCard
{
    public $brand;
    public $model;

    public function __construct($brand, $model)
    {
        $this->brand = $brand;
        $this->model = $model;
    }
}

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


$graphics_card2 = [];
foreach ($graphics_card as $value) {
    $graphics_card2[] = new GraphicsCard($value['brand'], $value['model']);
}

// Demo 1
$demoA1 = $graphics_card;
$demoA2 = $graphics_card2;
foreach ($demoA1 as $key => $value) {
    $demoA1[$key]['model'] = '1080';
    $demoA1[$key]['brand'] = '按摩店';
}
foreach ($demoA2 as $key => $item) {
    $demoA2[$key]->brand = '按摩店';
    $demoA2[$key]->model = '1080';
}
var_dump($demoA1);
var_dump($demoA2);

// Demo 2
$demoB1 = $graphics_card;
$demoB2 = $graphics_card2;
$demoB1 = array_map(function ($value) {
    $value['brand'] = '按摩店';
    $value['model'] = '1080';
    return $value;
}, $demoB1);
array_map(function ($value) {
    $value->brand = '按摩店';
    $value->model = '1080';
}, $demoB2);
var_dump($demoB1);
var_dump($demoB2);

// Demo 3
$demoC1 = $graphics_card;
$demoC2 = $graphics_card2;
array_map(function ($key, $value) use (&$demoC1) {
    $demoC1[$key]['brand'] = '按摩店';
    $demoC1[$key]['model'] = '1080';
}, array_keys($demoC1), $demoC1);
array_map(function ($key, $value) use ($demoC2) {
    $demoC2[$key]->brand = '按摩店';
    $demoC2[$key]->model = '1080';
}, array_keys($demoC2), $demoC2);
var_dump($demoC1);
var_dump($demoC2);

// Demo 4
$brands = [];
$models = [];
foreach ($graphics_card as $item) {
    $brands[] = $item['brand'];
}

$models = array_map(function ($value) {
    return $value['model'];
}, $graphics_card);

var_dump($brands);
var_dump($models);
// 也可以使用 array_column 来实现 Demo 4 的效果
var_dump(array_column($graphics_card, 'brand'));
var_dump(array_column($graphics_card, 'model'));

// Demo5
$items = [
    'item1'=>'Hello',
    'item2'=>'World',
    'item3'=>'Luis',
    'item4'=>'Edware',
    'item5'=>'Ann',
    'item6'=>'Eason',
];

foreach($items as $key => $value){
	if($key === 'item1'){
		$items[$key] = 'World';
	}elseif($key === 'item3'){
		$items[$key] = 'Edware';
	}
}
var_dump($items);

$items_to_modify = ['item1'=>'Hello','item3'=>'Luis'];
array_map(function($key,$value) use(&$items) {
    $items[$key] = $value;
},array_keys($items_to_modify),$items_to_modify);
var_dump($items);
```

## 使用 array_map 实现 array_column 的效果
```php
<?php

function array_pluck($key, $input) {
    if (is_array($key) || !is_array($input)) return array();
    $array = array_map(function($value) use($key){
       if(array_key_exists($key, $value)) return $value[$key];
    },$input);
    return $array;
}

var_dump(array_pluck('brand',$graphics_card));
var_dump(array_column($graphics_card,'brand'));
```
