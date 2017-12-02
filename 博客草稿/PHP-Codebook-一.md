---
title: PHP Codebook(一)
date: 2017-01-22 06:06:56
tags: PHP
category: PHP
---

## 1.1 访问子串

#### 问题
想知道一个字符串是否包含一个特定的子串。例如，想查看一个 email 地址是否包含一个 @

#### 实现
```php
<?php
$string = '506510463@qq.com';

if (($length = strpos($string, '@')) === false) {
    echo 'There was no @ in the e-mail address';
} else {
    echo 'This is a right e-mail address-' . $length;
}
```
strpos 找到子串时，会返回子串的位置。找不到子串时，会返回 false，为了区分 0 和 false，必须使用 恒等操作符或非恒等操作符来进行判断。

## 1.2 抽取子串

#### 问题

希望从字符串中的某个特定位置开始抽取这个字符串的一部分。例如，对于输入到一个表单的用户名，想要得到这个用户名的前八位字符。

#### 实现
```php
<?php
$start = 0;
$length = 8;
$string = "LuisEdware";

echo substr($string, $start, $length) . "\n";
// LuisEdwa

// 如果忽略 $length，substr() 会返回从 $start 到原字符串末尾的子串
$start = 5;
echo substr($string, $start) . "\n";
// dware

// 如果 $start 大于字符串的长度，substr() 将返回 false
$start = 20;
var_dump(substr($string, $start));
// false

// 如果 $start 加 $length 超过了字符串末尾，substr() 将返回从 $start 开始至字符串末尾的所有字符
echo substr($string, 8, 5) . "\n";
// re

// 如果 $start 为负数，substr() 会从字符串末尾倒数来确定子串从哪里开始
echo substr($string, -6) . "\n";
// Edware
echo substr($string, -6, 2) . "\n";
// Ed

// 如果 $length 为负数，substr() 会从字符串末尾倒数来确定子串到哪里结束
echo substr($string, 2, -2) . "\n";
// isEdwa
echo substr($string, -4, -2) . "\n";
// wa
```

## 1.3 替换子串

#### 问题

希望用另外一个不同的字符串来替换一个子串。例如，打印一个信用卡号之前，想要对除了后四位以外的部分模糊处理。或者当需要显示的文档过大，无法一次全部显示，可能希望显示一部分文本，另外还有一个链接指向其余的文本。

#### 实现
```php
<?php
$creditCard = "6217 0032 4000 0600 075";
echo substr_replace($creditCard, '****', 0, strlen($creditCard) - 4) . "\n";
// **** 075

echo substr_replace($creditCard, '**** **** **** ****', 0, strlen($creditCard) - 4) . "\n";
// **** **** **** **** 075

$text = "Chief Justice Roberts, President Carter, President Clinton, President Bush, President Obama, fellow Americans and people of the world, thank you.

We, the citizens of America, are now joined in a great national effort to rebuild our country and restore its promise for all of our people.";

echo substr_replace($text, "...", 25);
```

## 1.4 逐字节处理字符串

#### 问题

需要分别处理字符串中的各个字节

#### 实现
```php
<?php

$string = "This weekend,I am going shopping for pet chicken.";
$vowels = 0;

for ($i = 0, $j = strlen($string); $i < $j; $i++) {
    if (strstr('aeiouAEIOU', $string[$i])) {
        $vowels++;
        $string[$i] = ucwords($string[$i]);
    }
}

echo $vowels . "\n";
echo $string;
```

## 1.5 按单词或字节反转字符串

#### 问题

希望反转一个字符串中的单词或字节

#### 解决方案
```php
<?php

$string = "This is not a palindrome";

// 按字节反转
echo strrev($string) . "\n";
// emordnilap a ton si sihT

// 按单词反转
$string = "Hello World,My Name is Luis Edware.";

// 将字符串分解为单词
$words = explode(' ', $string);

// 反转单词数组
$words = array_reverse($words);

// 重新构建字符串
$string = implode(' ', $words);
echo $string;
// Edware. Luis is Name World,My Hello
```

## 1.6 生成随机字符串

#### 问题

希望生成一个随机字符串

#### 实现
```php
<?php
function str_rand($length, $characters = null)
{
    if ($characters === null) {
        $characters = '0123456789qwertyuiopasdfghjklzxcvbnm!@#$%^&*()-=_+[]{}:";,.<>/?';
    }

    if (!is_int($length) || $length < 0) {
        return false;
    }

    $charactersLength = strlen($characters);
    $string = '';

    for ($i = $length; $i > 0; $i--) {
        $string .= $characters[mt_rand(0, $charactersLength)];
    }

    return $string;
}

echo str_rand(32);
```

## 1.8 控制大小写

#### 问题

需要将字符串中的字母全大写或全小写，或者改变字母的大小写。例如，希望名字中的首字母大写，而其余字母都为小写。

#### 实现
```php
<?php
// 使用 ucfirst() 或 ucwords() 将一个单词或多个单词中的首字母大写
$string = "my name is Luis edware";
echo ucfirst($string) . "\n";
echo ucwords($string) . "\n";

// strtoupper()、strtolower() 将作用整个字符串
echo strtoupper($string) . "\n";
echo strtolower($string);
```

## 1.10 去除字符串首尾的空格

#### 问题

希望从字符串开头和末尾删除空白符。例如，在验证用户输入之前，可能希望先完成清理。

#### 实现
```php
<?php
/**
 * ltrim() 函数从字符串开头删除空白符
 * rtrim() 函数从字符串末尾删除空白符
 * trim() 函数则删除字符串开头和末尾的空白符
 */

$lString = "    LuisEdware";
$rString = "LuisEdware    ";
$string = "    LuisEdware    ";

echo ltrim($lString) . "\n";
echo rtrim($rString) . "\n";
echo trim($string) . "\n";

// trim() 函数还可以从字符串中删除用户指定的字符。
echo ltrim('506510463 2794408425 Luis Edware', '0..9') . "\n";
echo rtrim('LuisEdware 5065104632794408425', '0..9') . "\n";
```

## 1.13 生成固定宽度字段数据记录

#### 问题

需要格式化数据记录，使得每个字段占据指定数目的字符。

#### 实现
```php
<?php

$books = [
    ['trueName', 'nickName', 'birthday'],
    ['Elmer Gantry', 'Sinclair Lewis', 1927],
    ['The Scarlatti Inheritance', 'Robert Ludlum', 1971],
    ['The Parsifal Mosaic', 'William Styron', 1979],
];

foreach ($books as $book) {
    print pack('A30A20A10', $book[0], $book[1], $book[2]) . "\n";
}
/*
trueName                      nickName            birthday
Elmer Gantry                  Sinclair Lewis      1927
The Scarlatti Inheritance     Robert Ludlum       1971
The Parsifal Mosaic           William Styron      1979
 */
```

## 1.15 分解字符串

#### 问题

需要将一个字符串分解为片段。例如希望访问用户在一个 <textarea\> 表单域中输入的各行文本
```php
<?php
$string = 'My sentence is not very complicated';
$words = explode(' ', $string);
/*
Array
(
    [0] => My
    [1] => sentence
    [2] => is
    [3] => not
    [4] => very
    [5] => complicated
)
*/
```

## 1.16 使文本在指定行长度自动换行

#### 问题

需要实现字符串自动换行

#### 实现
```php
<?php
$text = "Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus. Aenean eu leo quam. Pellentesque ornare sem lacinia quam venenatis vestibulum. Sed posuere consectetur est at lobortis. Cras mattis consectetur purus sit amet fermentum.";

echo '<pre>' . $text . '</pre>' . "\n";
echo wordwrap($text, 50);
```





