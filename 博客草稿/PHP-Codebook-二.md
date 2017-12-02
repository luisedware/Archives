---
title: PHP Codebook(二)
date: 2017-01-22 11:38:37
tags: PHP
category: PHP
---

## 2.1 检查变量是否包含一个合法数字

#### 问题

希望确保变量包含一个数（即使变量的类型是字符串），或者希望检查变量不仅是一个数，而且要特别指定其类型是一个数字类型。

#### 实现

```php
<?php
    foreach ([5,'5','05',12.3,'16.7','five',oxDECAFBAD,'10e2000'] as $maybeNumber) {
        $isItNumeric = is_numeric($maybeNumber);
        $actualType = gettype($maybeNumber);
        $string = "Is the $actualType $maybeNumber numeric?";

        if(is_numeric($maybeNumber)){
            $answer = "yes";
        }else{
            $answer = "no";
        }

        echo pack("A40A3",$string,$answer)."\n";
    }
?>
/*
Is the integer 5 numeric?               yes
Is the string 5 numeric?                yes
Is the string 05 numeric?               yes
Is the double 12.3 numeric?             yes
Is the string 16.7 numeric?             yes
Is the string five numeric?             no
Is the string oxDECAFBAD numeric?       no
Is the string 10e2000 numeric?          yes
*/
```

## 2.2 比较浮点数


#### 问题

希望检查两个浮点数是否相等

#### 实现
```php
<?php
$delta = 0.00001;
$a = 1.00000001;
$b = 1.00000000;

if(abs($a - $b) < $delta){
    print "$a and $b are equal enough";
}
```

## 2.3 浮点数舍入

#### 问题

希望舍入一个浮点数，可能取整为一个整数，或者舍入为指定的小数位数

#### 实现

```php
<?php

// 四舍五入 round()
echo round(2.4) . "\n";
echo round(2.5) . "\n";

// 向上取整
echo ceil(2.4) . "\n";

// 向下取整
echo floor(2.9) . "\n";

// 一个数正好落在两个整数中间的位置，PHP 会向远离 0 的方向取整
echo round(2.5) . "\n";
echo round(-2.5) . "\n";
echo round(2.4) . "\n";
echo round(-2.4) . "\n";

echo ceil(2.5) . "\n";
echo ceil(-2.5) . "\n";
echo ceil(2.4) . "\n";
echo ceil(-2.4) . "\n";

echo floor(2.5) . "\n";
echo floor(-2.5) . "\n";
echo floor(2.4) . "\n";
echo floor(-2.4) . "\n";
```

## 2.4 处理一系列整数

#### 问题

希望对一个整数范围应用一段代码

#### 实现
```php
// 使用 range() 函数实现
<?php
foreach (range(0, 100, 5) as $number) {
    echo $number . "\n";
    printf("%d squared is %d\n", $number, $number * $number);
    printf("%d cubed is %d\n", $number, $number * $number * $number);
}
```

## 2.5 在指定范围内生成随机数

#### 问题

希望在指定的数字范围内生成一个随机数

#### 实现
```php
<?php
$lower = 0;
$upper = 1024;
$randNumber = mt_rand($lower, $upper);

echo $randNumber;
```

## 2.6 生成可预测的随机数

#### 问题

希望生成可预测的随机数，从而可以保证可重复的行为

#### 实现
```php
<?php

function pick_color()
{
    $colors = ['red', 'orange', 'yellow', 'blue', 'green', 'indigo', 'violet'];
    $i = mt_rand(0, count($colors) - 1);

    return $colors[$i];
}

mt_srand(34534);
$first = pick_color();
$second = pick_color();

print "$first is red and $second is yellow";
```

## 2.10 格式化数字

#### 问题

希望输出一个数，要包括千分位分隔符和小数点。

#### 实现
```php
<?php
$number = 1234.56;
// 千位分隔符
echo number_format($number) . "\n";

// 第二个参数指定使用了小数位数
echo number_format($number, 2) . "\n";

// 第三个参数指定小数点字符，第四个参数指定千位分隔符
echo number_format($number, 2, "@", '#') . "\n";
```

## 2.16 转换进制

#### 问题

需要将一个数从一个进制转换为另一个进制

#### 实现
```php
<?php
// 十六进制
$hex = 'a1';

// 从十六进制转换为十进制
$decimal = base_convert($hex, 16, 10) . "\n";
echo $decimal;

$decimal = 1024;
echo base_convert($decimal, 10, 2) . "\n";
echo base_convert($decimal, 10, 8) . "\n";
echo base_convert($decimal, 10, 16);
```
