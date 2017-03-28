---
title: PHP Codebook(九)
date: 2017-01-24 11:15:01
tags: PHP
category: PHP
---

## 9.1 处理表单输入

#### 问题

希望使用一个 HTML 页面提交表单，然后在同一个页面中处理这个表单中输入的数据

#### 实现

使用 $_SERVER['REQUEST_METHOD'] 变量来确定请求是用 get 还是 post 方法提交的。

## 9.2 验证表单输入：必填域

#### 问题

希望确保必须为一个表单元素提供一个值。例如，希望保证一个文本框不为空

#### 实现

```php
<?php
if (!(filter_has_var(INPUT_GET, 'flavor')
    && (strlen(filter_input(INPUT_GET, 'flavor'))))
) {
    print "You must enter your favorite ice cream flavor.\n";
} else {
    echo $_GET['flavor'];
}

if ((filter_has_var(INPUT_GET, 'color'))
    && (strlen(filter_input(INPUT_GET, 'color', FILTER_SANITIZE_STRING)) <= 5)
) {
    print "Color must be more than 5 characters.";
}

if (!(filter_has_var(INPUT_GET, 'choices'))
    && filter_input(INPUT_GET, 'choices', FILTER_DEFAULT, FILTER_REQUIRE_ARRAY)
) {
    print "You must select some choices.\n";
}

?>
```

## 9.3 验证表单输入：数字

#### 问题

希望确保在一个表单输入框中输入了一个数。

#### 实现
```php
<?php
$age = filter_input(INPUT_GET, 'age', FILTER_VALIDATE_INT);
if ($age === false) {
    print "Submitted age is invalid.";
}

$price = filter_input(INPUT_GET, 'price', FILTER_VALIDATE_FLOAT);
if ($price === false) {
    print "Submitted price is invalid.";
}
```

## 9.4 验证表单输入：email 地址

#### 问题

希望知道用户提供的一个 email 地址是否合法

#### 实现
```php
<?php
$email = filter_input(INPUT_GET, 'email', FILTER_VALIDATE_EMAIL);
if ($email === false) {
    print "Submitted email is invalid.";
}
```

## 9.16 处理变量名中包含点号的远程变量

#### 问题

希望处理一个变量，变量名中有一个点号，不过提交表单时，无法在 $_GET 或者 $_POST 中找到这个变量。


#### 实现

将变量名中的点号替换为一个下划线。
