---
title: PHP Codebook(六)
date: 2017-01-24 08:34:59
tags: PHP
category: PHP
---

## 6.4 使用命名参数

#### 问题

希望按名为函数指定参数，而不是通过函数调用时的位置来指定。

#### 实现
```php
<?php
function image1($params)
{
    if (!isset($params['src'])) {
        $image['src'] = 'cow.png';
    }

    if (!isset($params['alt'])) {
        $image['alt'] = 'Milk Factory';
    }

    if (!isset($params['height'])) {
        $image['height'] = '100';
    }

    if (!isset($params['width'])) {
        $image['width'] = '50';
    }

    return '<img src="' . $image['src'] . '" alt="' . $image['alt'] . '" width="' . $image['width'] . '" height="' . $image['height'] . '">';
}

function image2($params)
{
    $defaults = [
        'src' => 'cow.png',
        'alt' => 'milk factory',
        'width' => 100,
        'height' => 50,
    ];

    $params = array_merge($defaults, $params);
    return '<img src="' . $image['src'] . '" alt="' . $image['alt'] . '" width="' . $image['width'] . '" height="' . $image['height'] . '">';
}
?>
```

## 6.6 创建参数个数可变的函数

#### 问题

希望定义一个参数个数可变的函数

#### 实现
```php
<?php

function mean($numbers)
{
    $sum = 0;

    $size = count($numbers);

    for ($i = 0; $i < $size; $i++) {
        $sum += $numbers[$i];
    }

    $average = $sum / $size;

    return $average;
}

echo mean([96, 93, 98, 98]) . "\n";

function mean1()
{
    $sum = 0;

    $size = func_num_args();

    for ($i = 0; $i < $size; $i++) {
        $sum += func_get_arg($i);
    }

    $average = $sum / $size;

    return $average;

}

echo mean1(96, 93, 98, 98) . "\n";

function mean2()
{
    $sum = 0;

    $size = func_num_args();

    foreach (func_get_args() as $arg) {
        $sum += $arg;
    }
    $average = $sum / $size;

    return $average;
}

echo mean2(96, 93, 98, 98) . "\n";
```

## 6.7 按引用返回值

#### 问题

希望按引用返回一个值，而不是按值返回。这样就无需为变量建立一个重复的副本。

#### 实现
```php
<?php

function &array_find_value($needle, &$haystack)
{
    foreach ($haystack as $key => $value) {
        if ($needle == $value) {
            return $haystack[$key];
        }
    }
}

$names = ['Ann Eason', 'Luis Edware', 'Ivan Tomic', 'RouniFul'];

$prince =& array_find_value('Ann Eason', $names);
$prince = "梁非凡";
print_r($names);
```

## 6.8 返回多个值

#### 问题

希望从函数返回多个值。

#### 实现
```php
<?php

function array_stats($values)
{
    $min = min($values);
    $max = max($values);
    $mean = array_sum($values) / count($values);

    return [$min, $max, $mean];
}

$values = range(1, 100);
list($min, $max, $mean) = array_stats($values);

echo sprintf("min is %d,max is %d,mean is %d", $min, $max, $mean);
```

## 6.10 返回失败

#### 问题

希望从函数中指示失败

#### 实现
```php
<?php

function lookup($name)
{
    if (empty($name)) {
        return false;
    }
}

$name = false;
if (false !== lookup($name)) {

}
```

## 6.11  调用可变函数


#### 问题

希望根据一个变量的值来调用不同的函数

#### 实现
```php
<?php

function get_file($fileName)
{
    return file_get_contents($fileName);
}

$function = 'get_file';
$fileName = 'graphic.png';

call_user_func($function, $fileName);
```
