---
title: PHP Codebook(四)
date: 2017-01-22 14:56:46
tags: PHP
category: PHP
---

## 4.1 指定并非从元素 0 开始的数组

#### 问题

希望一步为一个数组赋多个元素，不过不希望第一个元素的索引为0


#### 实现
```php
<?php

$presidents = [1 => 'Washington', 'Adams', 'Lincoln', 'Jefferson'];
var_dump($presidents);

$fruits[1] = 'apple';
$fruits[] = 'banana';
var_dump($fruits);
```

## 4.3 数组初始化为一个整数范围

#### 问题

希望将一系列连续的整数赋至一个数组

#### 实现
```php
<?php
$cards = range(1,52);
```

## 4.4 迭代处理数组
```php
<?php
$array = range(0, 12);

foreach ($array as $item) {
    echo $item . "\n";
}

foreach ($array as $key => $value) {
    echo $value . "\n";
}

for ($key = 0, $size = count($array); $key < $size; $key++) {
    echo $array[$key] . "\n";
}

// 将内部指针重置为数组起始位置
reset($array);
while (list($key, $value) = each($array)) {
    echo $value . "\n";
}

array_map(function($value) {
    return $value;
}, $array);
```

## 4.5 从数组删除元素

#### 问题

希望从一个数组删除一个或多个数组

#### 实现
```php
<?php
$array = range(0, 12);

// 删除元素
unset($array[0]);

// 删除多个连续的元素,array_splice 会自动对数组重新建立索引，避免留出空位。
array_splice($array, 1, 9);
var_dump($array);
```


## 4.6 改变数组大小

#### 问题

希望改变一个数组的大小，使它大于或小于目前的大小

#### 实现
```php
<?php

// array_pad() 第一个
$array = ['apple', 'banana', 'cocount'];
$array = array_pad($array, 4, 'dates');
print_r($array);

$array = array_pad($array, -10, 'zucchini');
print_r($array);
```

## 4.7 将数组追加到另一个数组

#### 问题

希望把两个数组合并为一个数组

#### 实现
```php
<?php
// 合并只使用数值键的数组时，数组将重新编号，以保证值不会丢失。
$array1 = ['hello', 'world'];
$array2 = ['luis', 'edware', 'hello', 'world'];
$array3 = array_merge($array1, $array2);
print_r($array3);

// + 操作符也可以用来合并数组
print_r($array1 + $array2);

// 倘若两个数组有重复的键，第二数组会覆盖这些重复键的值
$array1 = ['A' => 'apple', 'B' => 'banana', 'O' => 'orange'];
$array2 = ['A' => 'Luis', 'C' => 'Imooc'];
print_r(array_merge($array1, $array2));
```

## 4.8 将数组转换为字符串

#### 问题

希望将一个数组转换为一个格式化的字符串

#### 实现
```php
<?php
$array = range(0, 9);
$string1 = join(',', $array);
$string2 = implode(',', $array);
print_r($string1 . "\n");
print_r($string2 . "\n");
```

## 4.10 检查一个键是否存在数组中

#### 问题

想知道一个数组是否包含某个键


#### 实现
```php
<?php

$array = ['a' => 'apple', 'b' => 'banana', 'c' => 'cao ni ma'];
if (array_key_exists('a', $array)) {
    echo $array['a'] . "\n";
}

if (isset($array['a'])) {
    echo $array['c'] . "\n";
}
```

## 4.11 检查一个元素是否在数组中

#### 问题

希望知道一个数组是否包含某个值

#### 实现
```php
<?php

$array = ['a' => 'apple', 'b' => 'banana', 'c' => 'cao ni ma'];

if (in_array('apple', $array)) {
    echo 'Own it';
} else {
    echo 'Need it';
}

$array = ['Emma', 'Pride and Prejudice', 'Northhanger Abbey'];
$array = array_flip($array);
if (array_key_exists('Emma', $array)) {
    echo 'Own it';
} else {
    echo 'Need it';
}
```

## 4.12 查找一个值在数组中的位置

#### 问题

希望知道一个值是否在数组中。如果这个值确实在数组中，希望知道它的键。

#### 实现
```php
<?php

// 如果一个值在数组中多次出现，array_serach() 只能保证返回其中一个实例，而不一定是第一个实例。
$position = ['a'=>'apple','banana','orange','b'=>'not bad'];
if(($result = array_search('123', $position)) !== false){
    echo $result;
}else{
    echo 'false';
}
```

## 4.13 查找通过某个测试的元素

#### 问题

希望找出数组中满足某些需求的元素

#### 实现
```php
<?php
    $array = range(0, 20,2);

    $result = array_filter($array,function($value){
        if($value%2 === 0){
            return true;
        }else {
            return false;
        }
    });
?>
```

## 4.14 查找数组中最大值或最小值元素

#### 问题

希望找出数组中有最大值或最小值的元素。


#### 实现
```php
<?php
$array = range(1, 100);

echo max($array) . "\n";
echo min($array) . "\n";
```

## 4.15 反转数组

#### 问题

希望反转数组中元素的顺序

#### 实现
```php
<?php

$array = range(1, 100);
$array = array_reverse($array);
print_r($array);

$fruits = ['a' => 'Apple', 'b' => 'Banana', 'o' => 'orange'];
$fruits = array_reverse($fruits);
print_r($fruits);
```

## 4.18 多个数据的排序

#### 问题

希望对多个数组或多位数组排序

#### 实现
```php
<?php

$colors = ['Red', 'White', 'Blue'];
$cities = ['Boston', 'New York', 'Chicago'];
$stuff = [
    'colors' => $colors,
    'cities' => $cities,
];

array_multisort($colors, $cities);
print_r($colors);
print_r($cities);


array_multisort($stuff['colors'], $stuff['cities']);
print_r($stuff);


$numbers = [0, 1, 2, 3];
$letters = ['a', 'b', 'c', 'd'];

array_multisort($numbers, SORT_NUMERIC, SORT_DESC, $letters, SORT_STRING, SORT_DESC);
print_r($numbers);
print_r($letters);
```

## 4.20 随机调整数组

#### 问题

希望按一种随机的顺序重排数组中的元素

#### 实现
```php
<?php

$name = ['Ann', 'Eason', 'Luis', 'Edware'];
print_r($name);

shuffle($name);
print_r($name);
```

## 4.21 删除数组中重复的元素

#### 问题

希望删除数组中重复的元素

#### 实现
```php
<?php

$array = ['Ann', 'Eason', 'Luis', 'Edware', 'Ann', 'Eason', 'Luis', 'Edware'];
print_r($array);

$unique = array_unique($array);
print_r($unique);

$unique2 = [];
foreach ($array as $item) {
    if (!in_array($item, $unique2)) {
        $unique2[] = $item;
    }
}
print_r($unique2);
```
## 4.22 对数组中的各个元素应用一个函数

#### 问题

希望对数组中的各个元素应用一个函数或方法，这就允许一次转换所有输入数据

#### 实现
```php
<?php

$names = [
    'firstName' => 'Ann‘Eason',
    'lastName' => 'Luis’Edware',
];


array_walk($names, function(&$value, $key) {
    $value = htmlentities($value, ENT_QUOTES);
});

foreach ($names as $name) {
    echo $name . "\n";
}

echo '****************************************' . "\n";

// 对于嵌套数据，可以使用 array_walk_recursive()
$names = [
    'firstName' => ['Ann"', 'Luis"'],
    'lastName' => ['"Eason', '"Edware'],
];

array_walk_recursive($names, function(&$value, $key) {
    $value = htmlentities($value, ENT_QUOTES);
});

foreach ($names as $name) {
    foreach ($name as $item) {
        echo $item . "\n";
    }
}
```

## 4.23 查找两个数组的并集、交集或差集

#### 问题

有两个数组，希望找出它们的并集、交集、差集和对称差集


#### 实现
```php
<?php

$a = range('a', 'n');
$b = range('h', 't');
// 要计算并集
$union = array_unique(array_merge($a, $b));
print_r($union);

// 要计算交集
$intersection = array_intersect($a, $b);
print_r($intersection);

// 简单差集
$difference1 = array_diff($a, $b);
$difference2 = array_diff($b, $a);
print_r($difference1);
print_r($difference2);

// 对称差集
$difference = array_merge(array_diff($a, $b), array_diff($b, $a));
print_r($difference);
```

## 4.24 高效迭代处理大型数据集

#### 问题

希望迭代处理一个元素列表，不过整个列表会占用大量内存，或者生成整个列表的速度非常慢。

#### 实现


## 4.25 使用数组语法访问对象

#### 问题

有一个对象，不过希望它作为一个数组来读写数据。这样不仅可以得到面向对象设计的好处，还可以利用我们熟悉的数组接口

#### 实现
```php
<?php

class FakeArray implements ArrayAccess
{
    private $elements;

    public function __construct()
    {
        $this->elements = [];
    }

    public function offsetExists($offset)
    {
        return isset($this->elements[$offset]);
    }

    public function offsetGet($offset)
    {
        return $this->elements[$offset];
    }

    public function offsetSet($offset, $value)
    {
        return $this->elements[$offset] = $value;
    }

    public function offsetUnset($offset)
    {
        unset($this->elements[$offset]);
    }
}

$array = new FakeArray;
$array['animal'] = "wabbit";

if (isset($array['animal']) && $array['animal'] == 'wabbit') {
    unset($array['animal']);
}

if (!isset($array['animal'])) {
    print "Well,What did you expect in an";
}
```

