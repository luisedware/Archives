---
title: LuisEdware 版 PHP 编程规范（遵循 PSR-1/PSR-2）
date: 2017-02-11 16:23:06
tags: PHP
category: PHP
---

## 八荣八耻

- 以「<b>动手实践</b>」为荣, 以「<b>只看不练</b>」为耻;
- 以「<b>打印日志</b>」为荣, 以「<b>单步跟踪</b>」为耻;
- 以「<b>空格缩进</b>」为荣, 以「<b>制表缩进</b>」为耻;
- 以「<b>单元测试</b>」为荣, 以「<b>人工测试</b>」为耻;
- 以「<b>模块复用</b>」为荣, 以「<b>复制粘贴</b>」为耻;
- 以「<b>多态应用</b>」为荣, 以「<b>分支判断</b>」为耻;
- 以「<b>优雅高效</b>」为荣, 以「<b>冗余拖沓</b>」为耻;
- 以「<b>总结分享</b>」为荣, 以「<b>跪求其解</b>」为耻。

## 代码风格

编程开发时，需要遵循 PSR 规范如下：

- PSR 2（编码风格规范）：[https://laravel-china.org/topics/2079](https://laravel-china.org/topics/2079)
- PSR 4（自动加载规范）：[https://laravel-china.org/topics/2081](https://laravel-china.org/topics/2081)

## 目录与文件

- 目录使用小写加下划线；
- 类库、函数文件统一以 `.php` 为后缀；
- 类的文件名均以命名空间定义，并且命名空间的路径和类库文件所在路径一致；
- 类文件采用驼峰法命名（首字母大写），其它文件采用小写+下划线命名；
- 类名和类文件名保持一致，统一采用首字母大写的驼峰法命名；

## 函数、成员属性、成员方法

- 函数的命名使用小写字母和下划线的方式
- 成员属性的命名使用首字母小写的驼峰法
- 成员方法的命名使用首字母小写的驼峰法

## 数据表和字段

- 数据表采用小写加下划线方式命名
- 字段采用首字母小写的驼峰法命名


## 常量和配置

- 常量以大写字母和下划线命名
- 配置以小写字母和下划线命名


## 示例代码

函数声明示例如下：
```php
<?php
if( ! function_exists('array_random')){
    /**
     * 随机返回数组中的值
     *
     * @param  $array
     *
     * @return mixed
     */
    function array_random($array)
    {
        return $array[array_rand($array)];
    }
}
```

类文件声明示例如下：
```php
<?php
/**
 * 符合 RSR-1,RSR-2 的编程实例
 *
 * @Date      2017/02/17 18:00
 * @Author    七月十五九月初七
 * @License   广东安乐窝网络科技有限公司
 * @Copyright Copyright (c) 2017 AnLeWo Ltd
 */
namespace Standard; // 顶部命名空间

use Yii;//  空一行 use 引入类

/**
 * 类描述
 * 类名必须大写开头驼峰.
 */
abstract class ExampleClass // {}必须换行
{
    /**
     * 常量描述
     *
     * @var string
     */
    const THIS_IS_A_CONST = ''; // 常量全部大写下划线分割

    /**
     * 属性描述
     *
     * @var string
     */
    public $nameTest = ''; // 属性名称建议开头小写驼峰,成员属性必须添加public（不能省略）， private, protected修饰符

    /**
     * 属性描述
     *
     * @var string
     */
    private $_privateNameTest = ''; // 类私有成员属性，【个人建议】下划线小写开头驼峰

    /**
     * 构造函数
     *
     * 构造函数描述
     *
     * @param  string $value 形参名称/描述
     */
    public function __construct($value = '')// 成员方法必须添加public（不能省略）， private, protected修饰符
    {// {}必须换行
        $this->nameTest = new TestClass();
        // 链式操作
        $this->nameTest->functionOne()
            ->functionTwo()
            ->functionThree();
        // 一段代码逻辑执行完毕 换行
        // code...
    }

    /**
     * 成员方法名称
     *
     * 成员方法描述
     *
     * @param  string $value 形参名称/描述
     *
     * @return 返回值类型 返回值描述
     */
    public static function staticFunction($value = '')// static位于修饰符之后
    {
        // code...
    }

    /**
     * 成员方法名称
     *
     * 成员方法描述
     *
     * @param  string $value 形参名称/描述
     *
     * @return 返回值类型 返回值描述
     * 返回值类型：string，array，object，mixed（多种，不确定的），void（无返回值）
     */
    public function testFunction($value = '')// 成员方法必须小写开头驼峰
    {
        // code...
    }

    /**
     * 成员方法名称
     *
     * 成员方法描述
     *
     * @param  string $value 形参名称/描述
     *
     * @return 返回值类型 返回值描述
     */
    abstract public function abstractFunction($value = '');

    /**
     * 成员方法名称.
     * 成员方法描述
     *
     * @param  string $value 形参名称/描述
     *
     * @return 返回值类型 返回值描述
     */
    final public function finalFunction($value = '')// final位于修饰符之前
    {
        // code...
    } // abstract位于修饰符之前

    /**
     * 成员方法名称
     *
     * 成员方法描述
     *
     * @param  string $paramOne   形参名称/描述
     * @param  string $paramTwo   形参名称/描述
     * @param  string $paramThree 形参名称/描述
     * @param  string $paramFour  形参名称/描述
     * @param  string $paramFive  形参名称/描述
     * @param  string $paramSix   形参名称/描述
     *
     * @return 返回值类型 返回值描述
     */
    public function tooLangFunction(
        $paramOne = '', // 变量命名可小写开头驼峰或者下划线命名
        $paramTwo = '',
        $paramThree = '',
        $paramFour = '',
        $paramFive = '',
        $paramSix = ''
    )// 参数过多换行
    {
        if ($paramOne === $paramTwo) {// 控制结构=>后加空格,同{一行，（右边和)左边不加空格
            // code...
        }

        switch ($paramThree) {
            case 'three':
                // code...
                break;
            default:
                // code...
                break;
        }

        do {
            // code...
        } while ($paramFour <= 10);

        while ($paramFive <= 10) {
            // code...
        }

        for ($i = 0; $i < $paramSix; $i++) {
            // code...
        }
    }

    /**
     * 成员方法名称
     *
     * 成员方法描述
     *
     * @param  string $value 形参名称/描述
     *
     * @return 返回值类型 返回值描述
     */
    private function _privateTestFunction($value = '')// 私有成员方法【个人建议】下划线小写开头驼峰
    {
        // code...
    }
}
```
