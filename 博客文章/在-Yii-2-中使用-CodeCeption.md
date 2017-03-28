---
title: 在 Yii 2 中使用 CodeCeption
date: 2017-02-14 14:43:55
tags: Yii
category: PHP
---

## 安装 Yii2

使用 Composer 安装 Yii 2 高级模板项目
```shell
sudo composer global require "fxp/composer-asset-plugin:^1.2.0"
sudo composer create-project --prefer-dist yiisoft/yii2-app-advanced yii-application
cd yii-application
sudo composer install
```

配置 Apache 服务器
```apache
<VirtualHost *:80>
   ServerName frontend.dev
   DocumentRoot "/path/to/yii-application/frontend/web/"

   <Directory "/path/to/yii-application/frontend/web/">
       # use mod_rewrite for pretty URL support
       RewriteEngine on
       # If a directory or a file exists, use the request directly
       RewriteCond %{REQUEST_FILENAME} !-f
       RewriteCond %{REQUEST_FILENAME} !-d
       # Otherwise forward the request to index.php
       RewriteRule . index.php

       # use index.php as index file
       DirectoryIndex index.php

       # ...other settings...
       # Apache 2.4
       Require all granted

       ## Apache 2.2
       # Order allow,deny
       # Allow from all
   </Directory>
</VirtualHost>

<VirtualHost *:80>
   ServerName backend.dev
   DocumentRoot "/path/to/yii-application/backend/web/"

   <Directory "/path/to/yii-application/backend/web/">
       # use mod_rewrite for pretty URL support
       RewriteEngine on
       # If a directory or a file exists, use the request directly
       RewriteCond %{REQUEST_FILENAME} !-f
       RewriteCond %{REQUEST_FILENAME} !-d
       # Otherwise forward the request to index.php
       RewriteRule . index.php

       # use index.php as index file
       DirectoryIndex index.php

       # ...other settings...
       # Apache 2.4
       Require all granted

       ## Apache 2.2
       # Order allow,deny
       # Allow from all
   </Directory>
</VirtualHost>
```

## 安装 Codeception

在项目根目录执行以下命令安装 codeception 扩展
```shell
sudo composer global require "codeception/codeception=*"
sudo composer global require "codeception/specify=*"
sudo composer global require "codeception/verify=*"
sudo composer require --dev "yiisoft/yii2-faker=*"
```

将 codeception 命令配置进环境变量中
```shell
vim .zshrc

// 在文件 .zshrc 中添加以下代码，且保存退出
alias codecept="/Users/LuisEdware/.composer/vendor/codeception/codeception/codecept"

source .zshrc
```

在项目中使用 codeception
```
cd frontend/
sudo chmod -R 777 tests
codecept build
codecept run
```

## 单元测试（Unit Test）

单元测试是针对程序模块来进行正确性检验的测试工作，可以通过单元测试来确认深藏在代码里面的某些功能仍然可用, 单元测试能消除程序单元的不可靠性。

Codeception 的单元测试功能是基于 PHPUnit 之上的, 你可以照样写 PHPUnit 的测试代码, Codeception 一样能运行。

#### 创建单元测试

使用 Codeception 的生成器来执行以下命令生成单元测试
```
cd frontend
codecept generate:test unit Example
```

上述命令会在 tests/unit 目录创建一个 ExampleTest 文件，代码如下：
```php
<?php
namespace frontend\tests;

class ExampleTest extends \Codeception\Test\Unit
{
    /**
     * @var \frontend\tests\UnitTester
     */
    protected $tester;

    protected function _before()
    {
    }

    protected function _after()
    {
    }

    public function testPushAndPop()
    {
        $stack = [];
        $this->assertEquals(0, count($stack));

        array_push($stack, 'foo');
        $this->assertEquals('foo', $stack[count($stack)-1]);
        $this->assertEquals(1, count($stack));

        $this->assertEquals('foo', array_pop($stack));
        $this->assertEquals(0, count($stack));
    }
}
```

###### 测试依赖

通过 `@depends` 让测试方法之间进行关联依赖

```php
<?php
namespace frontend\tests;

class ExampleTest extends \Codeception\Test\Unit
{
    public function testEmpty()
    {
        $stack = [];
        $this->assertEmpty($stack);

        return $stack;
    }

    /**
     * @depends testEmpty
     */
    public function testPush(array $stack)
    {
        array_push($stack, 'foo');
        $this->assertEquals('foo', $stack[count($stack)-1]);
        $this->assertNotEmpty($stack);

        return $stack;
    }

    /**
     * @depends testPush
     */
    public function testPop(array $stack)
    {
        $this->assertEquals('foo', array_pop($stack));
        $this->assertEmpty($stack);
    }
}
```
###### 数据提供商

通过 `@dataProvider` 让测试方法迭代运行数据提供者方法的数据

```php
<?php
namespace frontend\tests;

class DataTest extends \Codeception\Test\Unit
{
    /**
     * @var \frontend\tests\UnitTester
     */
    protected $tester;

    protected function _before()
    {
    }

    protected function _after()
    {
    }

    /**
     * @dataProvider additionProvider
     */
    public function testAdd($a, $b, $expected)
    {
        echo $a;
        echo $b;
        echo $expected;
        $this->assertEquals($expected, $a + $b);
    }

    public function additionProvider()
    {
        return [
            [0, 0, 0],
            [0, 1, 1],
            [1, 0, 1],
            [1, 1, 2]
        ];
    }
}
```

###### 抛出异常

通过 `@expectedException` 或方法 `$this->expectException()` 在测试实例中抛出异常

```php
<?php
namespace frontend\tests;

class ExceptionTest extends \Codeception\Test\Unit
{
    public function testException()
    {
        $this->expectException(InvalidArgumentException::class);
    }

    /**
     * @expectedException InvalidArgumentException
     */
    public function testExceptionTow()
    {

    }
}
```

###### 断点输出

通过方法 `$this->expectOutputString()` 生成一个预期的输出。当实际输出和预期输出不一致时，则测试不通过。

```php
<?php
use PHPUnit\Framework\TestCase;

class OutputTest extends TestCase
{
    public function testExpectFooActualFoo()
    {
        $this->expectOutputString('foo');
        print 'foo';
    }

    public function testExpectBarActualBaz()
    {
        $this->expectOutputString('bar');
        print 'baz';
    }
}
```

Codeception 的更多 Unit Tests 信息请参考链接：

- [https://github.com/Codeception/Codeception/blob/2.0/docs/06-UnitTests.md](https://github.com/Codeception/Codeception/blob/2.0/docs/06-UnitTests.md)
- [https://github.com/Codeception/Codeception/blob/2.0/docs/modules/Yii2.md](https://github.com/Codeception/Codeception/blob/2.0/docs/modules/Yii2.md)

## 创建接口测试（RESTFUL API TEST）

创建 RESTFUL API 测试配置文件，新增文件 `api.suite.yml` 和文件夹 `api`
```shell
codecept generate:suite api
```

修改配置文件 `api.suite.yml` 如下：
```
class_name: ApiTester
modules:
    enabled:
        - REST:
            url: www.frontend.dev/index.php?r=
            depends: Yii2
        - \ApiBundle\Helper\Api
    config:
        - Yii2
```

创建 RESTFUL API 测试文件
```
codecept generate:cept api CreateUser
```

修改生成文件代码如下：
```php
<?php
use frontend\tests\ApiTester;

// 发送 HTTP GET 请求
$I = new ApiTester($scenario);
$I->wantTo('I want to test RESTFUL API');
$I->sendGET('/api/v1/users/create');
$I->seeResponseCodeIs(\Codeception\Util\HttpCode::OK);
$I->seeResponseIsJson();
$I->seeResponseContains('{"result":"ok"}');

// 发送 HTTP POST 请求
$I->sendPOST('/users', ['name' => 'davert', 'email' => 'davert@codeception.com']);

// 验证 JSON 响应数据类型
$I->seeResponseMatchesJsonType([
    'id' => 'integer',
    'name' => 'string',
    'email' => 'string:email',
    'homepage' => 'string:url|null',
    'created_at' => 'string:date',
    'is_active' => 'boolean'
]);
```

常用的 REST API 如下：

- wantTo()：配置测试标题
- sendGET()：发送 HTTP GET 请求
- sendPUT()：发送 HTTP PUT 请求
- sendPOST()：发送 HTTP POST 请求
- sendDELETE()：发送 HTTP DELETE 请求
- haveHttpHeader()：设置 HTTP 请求头
- seeHttpHeader()：检查 HTTP 请求头
- seeResponseCodeIs()：检查响应状态码
- seeResponseContains()：检查响应内容
- seeResponseContainsJson()：检查响应数据类型是否为 JSON 和核对响应内容
- seeResponseIsJson()：检查响应是否返回 JSON 数据
- seeResponseJsonMatchesJsonPath()：使用 JSON 数据去检查响应内容
- seeResponseMatchesJsonType()：检查响应数据的数据类型

Codeception 的更多 RESTFUL API 信息请参考链接：

- [https://github.com/Codeception/Codeception/blob/2.0/docs/modules/REST.md](https://github.com/Codeception/Codeception/blob/2.0/docs/modules/REST.md)
- [https://github.com/Codeception/Codeception/blob/2.0/docs/10-WebServices.md](https://github.com/Codeception/Codeception/blob/2.0/docs/10-WebServices.md)

## 数据库测试（DB TEST）

常用的 DB TEST API 如下：

- seeInDatabase()
- dontSeeInDatabase()
- haveInDatabase()
- grabFromDatabase()

示例代码如下：
```php
<?php
// 检查 User 表是否存在 'username' 为 'Davert', 'email' 为 'davert@mail.com' 的数据，存在则报错
$I->dontSeeInDatabase('user', ['username' => 'Davert', 'email' => 'davert@mail.com']);

// 检查 User 表是否存在 'username' 为 'okirlin', 'email' 为 'brady.renner@rutherford.com' 的数据，不存在则报错
$I->seeInDatabase('user', ['username' => 'okirlin', 'email' => 'brady.renner@rutherford.com']);

// 检查 User 表是否能够新增以下数据，无法新增则报错
$I->haveInDatabase('user',
  [
    'username' => 'miles',
    'email' => 'miles@davis.com',
    'auth_key' => 'EdKfXrx88weFMV0vIxuTMWKgfK2tS321',
    'password_hash' => '$2y$13$g5nv41Px7VBqhS3hVsVN2.MKfgT3jFdkXEsMC4rQJLfaMa7VaJqL2',
    'password_reset_token' => '4BSNyiZNAuxjs5Mty990c47sVrgllIi_1487317115',
    'email' => '506510463@qq.com',
    'role' => 10,
    'status' => 0,
    'created_at' => 1391885313,
    'updated_at' => 1391885313,
]);

// 按照正序获取一条 username = miles 的数据的 email 字段值
$mail = $I->grabFromDatabase('user', 'email', array('username' => 'troy.becker'));
var_dump($mail);exit;
```

Codeception 的更多 DB TEST API 信息请参考链接：

- DB：[https://github.com/Codeception/Codeception/blob/2.0/docs/modules/Db.md](https://github.com/Codeception/Codeception/blob/2.0/docs/modules/Db.md)
- DBH：[https://github.com/Codeception/Codeception/blob/2.0/docs/modules/Dbh.md](https://github.com/Codeception/Codeception/blob/2.0/docs/modules/Dbh.md)
- Doctrine1：[https://github.com/Codeception/Codeception/blob/2.0/docs/modules/Doctrine1.md](https://github.com/Codeception/Codeception/blob/2.0/docs/modules/Doctrine1.md)
- Doctrine2：[https://github.com/Codeception/Codeception/blob/2.0/docs/modules/Doctrine2.md](https://github.com/Codeception/Codeception/blob/2.0/docs/modules/Doctrine2.md)
- Working with Data：[https://github.com/Codeception/Codeception/blob/2.0/docs/09-Data.md](https://github.com/Codeception/Codeception/blob/2.0/docs/09-Data.md)
