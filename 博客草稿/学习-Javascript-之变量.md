---
title: 学习 Javascript 之变量
date: 2017-03-23 10:16:35
tags: Javascript
category: Javascript
---

## 变量的声明

```js
/* 声明变量 */
var x = 3.14;
var y;
var z = x + y;

var a = 1, b = 2, c = true;
var foo = "Hello" + "World";
var bar = 2 > 1 ? 2 != 1 : 1 > 2 ;

/* 声明常量 */
const PI = 3.1415;

/* ES6 - 声明变量 */
let a = 10;
let [a, b, c] = [1, 2, 3];
```

## 变量的声明作用域

- var
	- 在全局作用域内有效
- let
	- 只在声明所在的块级作用域内有效
- const
	- 只在声明所在的块级作用域内有效

## 变量的声明陷阱

- 变量提升
	- 变量无法在 let 命令声明之前使用，否则报错
	- 变量可以在 var 命令声明之前使用，值为 undefined
- 暂时性死区
	- 在代码块内，使用 let 命令声明变量之前，该变量都是不可用的
- 重复声明
	- var 命令允许在相同作用域内，重复声明一个变量
	- let 命令不允许在相同作用域内，重复声明一个变量

## 解构赋值
```js
/* 数组的解构赋值 */
let [foo, [[bar], baz]] = [1, [[2], 3]];

/* 对象的解构赋值 */
let { foo, bar } = { foo: "aaa", bar: "bbb" };

/* 字符串的解构赋值 */
const [a, b, c, d, e] = 'hello';

/* 函数的解构赋值 */
function add([x, y]){
  return x + y;
}
add([1, 2]); // 3
```

变量的解构赋值用途如下：

- 交换变量的值
- 从函数返回多个值
- 函数参数的定义
- 提取 JSON 数据
- 函数参数的默认值
- 遍历 Map 结构
- 输入模块的指定方法
