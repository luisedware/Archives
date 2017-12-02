---
title: 使用 Yii2 构建 RESTFul API
date: 2017-02-28 16:49:29
category: PHP
tags: Yii2
---


## 安装 Yii2 高级模板

```
composer global require "fxp/composer-asset-plugin:^1.2.0"
composer create-project --prefer-dist yiisoft/yii2-app-advanced Api
```

初始化项目
```shell
cd Api
sudo php init
```

打开文件 `common/config/main-local.php`，配置数据库信息
```php
<?php
return [
    'components' => [
        'db' => [
            'class' => 'yii\db\Connection',
            'dsn' => 'mysql:host=localhost;dbname=api',
            'username' => 'root',
            'password' => '123456',
            'charset' => 'utf8',
        ],
        'mailer' => [
            'class' => 'yii\swiftmailer\Mailer',
            'viewPath' => '@common/mail',
            // send all mails to a file by default. You have to set
            // 'useFileTransport' to false and configure a transport
            // for the mailer to send real emails.
            'useFileTransport' => true,
        ],
    ],
];
```

执行迁移文件，生成数据表
```shell
php yii migrate
```

新建模板 `api`,执行操作如下：

- 复制目录 `frontend` 为 `api`
- 复制目录 `environments/dev/frontend` 为 `environments/dev/api`
- 复制目录 `environments/prod/frontend` 为 `environments/prod/api`
- 修改目录 `api` 里面文件的命名空间 `namespace`
- 打开文件 `common/config/bootstrap.php` 并加入代码 `Yii::setAlias('api', dirname(dirname(__DIR__)) . '/api');`

清理模板 `api`，删除文件如下：

- api/assets
- api/views
- api/controllers
- api/web/assets
- api/web/css


配置 Apache 虚拟主机
```Apache
<VirtualHost *:80>
   ServerName api.frontend.app
   DocumentRoot "/Users/LuisEdware/Code/Api/frontend/web/"

   <Directory "/Users/LuisEdware/Code/Api/frontend/web/">
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
   ServerName api.backend.app
   DocumentRoot "/Users/LuisEdware/Code/Api/backend/web/"

   <Directory "/Users/LuisEdware/Code/Api/backend/web/">
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
   ServerName api.app
   DocumentRoot "/Users/LuisEdware/Code/Api/api/web/"

   <Directory "/Users/LuisEdware/Code/Api/api/web/">
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

## 搭建基本的 RESTFul 架构

#### 创建控制器
在目录 `api/controllers` 下创建控制器 `UserController`，代码如下：
```php
<?php

namespace api\modules\v1\controllers;

use yii\rest\ActiveController;

class OrderController extends ActiveController
{
    // 声明控制器所采用的数据模型
    public $modelClass = 'api\modules\v1\models\Order';

    // 序列化输出
    public $serializer = [
        'class' => 'yii\rest\Serializer',
        'collectionEnvelope' => 'items',
    ];
}
```

#### 配置路由规则
打开文件 `api/config/main-local.php`，在数组 `$config['components']` 中新增代码如下：
```php
<?php
'components' => [
        // 接受 JSON 格式的数据
        'request' => [
            'cookieValidationKey' => 'sTGCwRd3spEa33BW2HMjuuwZnsJO6RMM',
            'parsers' => [
                'application/json' => 'yii\web\JsonParser',
            ],
        ],
        // 配置路由规则
        'urlManager' => [
            'enablePrettyUrl' => true,
            'enableStrictParsing' => true,
            'showScriptName' => false,
            'rules' => [
                [
                    'class' => 'yii\rest\UrlRule',
                    'controller' => 'user',
                    'pluralize' => false,
                ],
            ],
        ],
    ],
```
#### 创建数据模型

在目录 `api\models` 新增数据模型，代码如下：
```php
<?php

namespace api\models;

use yii\db\ActiveRecord;

class User extends ActiveRecord
{
    public static function tableName()
    {
        return "user";
    }

    public function fields()
    {
        // 删除敏感字段
        $fields = parent::fields();
        unset($fields['auth_key'], $fields['password_hash'], $fields['password_reset_token']);
        return $fields;
    }
}
```

#### 测试接口地址

使用 `HTTP GET` 的方式去访问 `http://api.app/user`,得到 JSON 数据如下：
```json
{
  "items": [
    {
      "id": 1,
      "username": "okirlin",
      "email": "brady.renner@rutherford.com",
      "role": 10,
      "status": 10,
      "created_at": 1391885313,
      "updated_at": 1391885313
    },
    {
      "id": 2,
      "username": "troy.becker",
      "email": "nicolas.dianna@hotmail.com",
      "role": 10,
      "status": 0,
      "created_at": 1391885313,
      "updated_at": 1391885313
    },
    {
      "id": 3,
      "username": "miles",
      "email": "506510463@qq.com",
      "role": 10,
      "status": 0,
      "created_at": 1231231233,
      "updated_at": 1231231233
    }
  ],
  "_links": {
    "self": {
      "href": "http://api.app/user?page=1"
    }
  },
  "_meta": {
    "totalCount": 8,
    "pageCount": 1,
    "currentPage": 1,
    "perPage": 20
  }
}
```
## 版本控制

#### 实现方式

有两种方案可以实现 API 版本控制：

- 在 URL 中带上请求的版本号，比如：`http://example.com/v1/users`
- 把版本号放在HTTP请求头中，通常通过 Accept 头，像这样：`Accept: application/json; version=v1`

Yii2 两种方案都支持，官方推荐，使用模块来实现版本化，简化代码维护和管理，在 URL 上带上一个大版本号（比如： v1, v2），在每个大版本中，使用 `Accept HTTP` 请求头来判定小版本号并编写针对各小版本的条件语句。

#### 创建模块
新建目录 `api/modules/v1`，目录下新建文件 `Module.php`，代码如下：
```php
<?php

namespace api\modules\v1;

/**
 * v1 module definition class
 */
class Module extends \yii\base\Module
{
    /**
     * @inheritdoc
     */
    public $controllerNamespace = 'api\modules\v1\controllers';

    /**
     * @inheritdoc
     */
    public function init()
    {
        parent::init();
    }
}
```

#### 创建控制器
新建目录 `api/modules/v1/controllers`，在该目录下新增控制器 `OrderController.php`，代码如下：
```php
<?php

namespace api\modules\v1\controllers;

use yii\rest\ActiveController;

class OrderController extends ActiveController
{
    // 声明控制器所采用的数据模型
    public $modelClass = 'api\modules\v1\models\Order';

    // 序列化输出
    public $serializer = [
        'class' => 'yii\rest\Serializer',
        'collectionEnvelope' => 'items',
    ];
}
```

#### 创建模型
新建目录 `api/modules/v1/models`，在该目录下新增数据模型 `Order.php`，代码如下：
```php
<?php

namespace api\modules\v1\models;

use yii\db\ActiveRecord;

class Order extends ActiveRecord
{
    public static function tableName()
    {
        return "z_order";
    }

    // 返回指定的字段
    public function fields()
    {
        return [
            'id',
            // 重命名字段
            'orderNumber' => 'orderno'
        ];
    }
}
```

#### 配置路由与模块
打开文件 `api/config/main-local.php`，在数组 `$config[]` 新增模块版本，在数组 `$config['components']['urlManager']['rules']` 新增路由规则，代码如下：
```php
<?php
$config = [
    'modules' => [
        'v1' => [
            'class' => 'api\modules\v1\Module',
        ],
    ],
    'components' => [
        'request' => [
            // !!! insert a secret key in the following (if it is empty) - this is required by cookie validation
            'cookieValidationKey' => 'sTGCwRd3spEa33BW2HMjuuwZnsJO6RMM',
            'parsers' => [
                'application/json' => 'yii\web\JsonParser',
            ],
        ],
        'urlManager' => [
            'enablePrettyUrl' => true,
            'enableStrictParsing' => true,
            'showScriptName' => false,
            'rules' => [
                [
                    'class' => 'yii\rest\UrlRule',
                    'controller' => 'user',
                    'pluralize' => false,
                ],
                [
                    'class' => 'yii\rest\UrlRule',
                    'controller' => 'v1/order',
                    'pluralize' => false,
                ],
            ],
        ],
    ],
];
```
#### 测试接口地址

使用 `HTTP GET` 的方式去访问 `http://api.app/v1/order`,得到 JSON 数据如下：
```json
{
  "orders": [
    {
      "id": 57,
      "orderNumber": "201310140001"
    },
    {
      "id": 68,
      "orderNumber": "201310150001"
    },
    {
      "id": 69,
      "orderNumber": "201310150002"
    },
    {
      "id": 70,
      "orderNumber": "201310150003"
    },
    {
      "id": 71,
      "orderNumber": "201310170001"
    },
    {
      "id": 72,
      "orderNumber": "201310170002"
    },
    {
      "id": 73,
      "orderNumber": "201310170003"
    },
    {
      "id": 74,
      "orderNumber": "201310170004"
    },
  ]
}
```

## API 认证

Yii 2 有三种认证用户的验证方式

- `HttpBasicAuth::className()`：是通过弹窗填用户名密码，默认只需要在用户名栏填入 `Access-Token` 返回接口数据。
- `HttpBearerAuth::className()`：是通过请求 `Headers` 带上 `Authorization: Bearer Access-Token` 参数返回接口数据。
- `QueryParamAuth::className()`：是通过 `URL` 带上 `?access-token=Access-Token` 参数返回接口数据。


采用 `QueryParamAuth` 方式进行验证，因为 `RESTFul API` 是无状态的，不能通过 `session` 或 `cookie` 来保持状态，在模块文件 `Module.php` 中修改代码如下：
```php
<?php

namespace api\modules\v1;

use Yii;

/**
 * v1 module definition class
 */
class Module extends \yii\base\Module
{
    /**
     * @inheritdoc
     */
    public $controllerNamespace = 'api\modules\v1\controllers';

    /**
     * @inheritdoc
     */
    public function init()
    {
        parent::init();
        // 禁止使用 Session
        Yii::$app->user->enableSession = false;
    }

    // v1 模块的请求都需要认证
    public function behaviors()
    {
        $behaviors = parent::behaviors();
        $behaviors['authenticator'] = [
            'class' => QueryParamAuth::className(),
        ];
        return $behaviors;
    }
}

```

使用 `HTTP GET` 的方式去访问 `http://api.app/v1/order?access-token=HelloWorld`,得到 JSON 数据如下：
```json
{
  "orders": [
    {
      "id": 57,
      "orderNumber": "201310140001"
    },
    {
      "id": 68,
      "orderNumber": "201310150001"
    },
    {
      "id": 69,
      "orderNumber": "201310150002"
    },
    {
      "id": 70,
      "orderNumber": "201310150003"
    },
  ]
}
```

## 使用 Swagger 生成 API 可调试文档

#### 配置 Swagger-UI
执行以下命令，将 `Swagger-UI` 克隆到 `api/web/v1/` 文件夹下：
```shell
cd api/web
mkdir v1
cd v1
git clone https://github.com/swagger-api/swagger-ui.git
```

执行以下命令，在目录 `api/web/v1` 下生成 `Swagger` 文档配置文件：
```
mkdir -p swagger-document/
vim swagger-document/swagger.json
```

打开文件 `api/web/v1/swagger-ui/dist/index.html`，修改生成文档的配置文件地址：
```js
if (url && url.length > 1) {
  url = decodeURIComponent(url[1]);
} else {
  url = "http://api.app/v1/swagger-document/swagger.json";
}
```

#### Swagger-PHP 的基本语法

参考文档

- [Swagger-Annotations](http://zircote.com/swagger-php/1.x/index.html)
- [Swagger-Explained](http://bfanger.nl/swagger-explained/)

红色为必写的字段，蓝色为封装的字段。

- <font color="blue">@SWG\Swagger</font>
    - swagger
    - <font color=red>info</font>: <font color="blue">@SWG\Info</font>
    - host - 接口域名
    - basePath - 接口前缀路径
    - schemes: ["http", "https", "ws", "wss"]
    - consumes - 请求 Mime Types
    - produces - 响应 Mime Types
    - paths: <font color="blue">@SWG\Path</font>
- <font color="blue">@SWG\Info</font>
    - title - 文档标题
    - description - 文档描述
    - termsOfService - 所属团队
    - contact: <font color="blue">@SWG\Contact</font> - 联系方式
- <font color="blue">@SWG\Contact</font>
    - url - 联系链接
    - name - 联系名称
    - email - 联系邮箱
- <font color="blue">@SWG\License</font>
    - <font color=red>name</font> - 开源许可证
    - url - 许可证地址
- <font color="blue">@SWG\Path</font>
    - get: <font color="blue">@SWG\Get</font> - HTTP Get 请求
    - put: <font color="blue">@SWG\Put</font> - HTTP Put 请求
    - post: <font color="blue">@SWG\Post</font> - HTTP Post 请求
    - delete: <font color="blue">@SWG\Delete</font> - HTTP Delete 请求
    - options: <font color="blue">@SWG\Options</font> - HTTP Options 请求
- <font color="blue">@SWG\GET</font>
    - tags - 请求分类
    - summary - 请求简介
    - description - 请求描述
    - operationId - 请求编号，要求唯一
    - consumes - 请求 Mime Types
    - produces - 响应 Mime Types
    - parameters: <font color="blue">@SWG\Parameter</font> - 请求参数
    - <font color=red>responses</font>: <font color="blue">@SWG\Response</font> - 请求响应
    - schemes: ["http","https","ws","wss"] - 请求协议
    - deprecated - 是否弃用
- <font color="blue">@SWG\Parameter</font>
    - <font color=red>name</font> - 请求参数名称
    - <font color=red>in</font>: ["query","header","path","formData","body"] - 请求参数存放方式
    - description - 请求参数描述
    - required - 是否要求
    - <font color=red>schema</font> - 当 `in` 为 `body` 时可以使用，用于描述参数
    - <font color=red>type</font>: ["string", "number", "integer", "boolean", "array", "file"] - 请求参数类型
    - format: ["int32", "int64", "float", "double", "byte", "date", "date-time"] - 请求参数格式
    - allowEmptyValue - 是否允许空值
    - items - 当 `type` 为 `array`,`items` 为 `required`，描述参数数组
    - collectionFormat
    - default - 请求参数默认值
- <font color="blue">@SWG\Response</font>
  - <font color=red>default</font>
    - response object
        - <font color=red>description</font> - 响应描述
        - schema: <font color="blue">@SWG\Schema</font>
        - headers: <font color="blue">@SWG\Header</font> - 响应头部
        - example: <font color="blue">@SWG\Example</font> - 响应数据例子
    - reference object
        - <font color=red>$ref</font>
  - HTTP Status Code
    - response object
          - <font color=red>description</font>
        - schema: <font color="blue">@SWG\Schema</font>
        - headers: <font color="blue">@SWG\Header</font>
        - example: <font color="blue">@SWG\Example</font>
    - reference object
        - <font color=red>$ref</font>
- <font color="blue">@SWG\Definition</font>
  - definition
  - required
  - <font color="blue">@SWG\Property</font>
- <font color="blue">@SWG\Property</font>
  - property - 模型成员属性
  - type: ["string", "number", "integer", "boolean", "array", "file"] - 模型参数类型
  - format: ["int32", "int64", "float", "double", "byte", "date", "date-time"] - 模型参数格式
- <font color="blue">@SWG\Header</font>
  - header - 头部名称
  - type - 头部数值类型
  - description - 头部简介

#### 配置 Swagger-PHP

使用 `Composer` 安装 `Swagger-PHP`，在项目根目录中执行命令如下：
```
composer require zircote/swagger-php
```

在 `api\controllers` 新增文档控制器 `DocumentController.php`，因为要生成不同版本的文档，控制器必须放在公用的目录，代码如下：
```php
<?php

namespace api\controllers;

use Yii;
use Swagger;
use yii\base\Exception;
use yii\web\Controller;

class DocumentController extends Controller
{
    // 根据版本号生成不同版本的 API 文档
    public function actionBuild($version)
    {
        $apiDir = Yii::getAlias('@api');
        $swagger = Swagger\scan($apiDir);
        $jsonFile = $apiDir . "/web/{$version}/swagger-document/swagger.json";
        if (!file_exists($jsonFile)) {
            throw new Exception("Swagger.json 文件不存在");
        }

        $isWrite = file_put_contents($jsonFile, $swagger);
        if ($isWrite == true) {
            $this->redirect("/{$version}/swagger-ui/dist/index.html");
        } else {
            throw new Exception("Swagger.json 文件写入失败");
        }
    }
}
```

新增路由如下：
```php
<?php
[
  'class' => 'yii\web\UrlRule',
  'pattern' => 'document/build/<version:\w+>',
  'route' => 'document/build',
],
```

模块文件 `app/modules/Module.php` 新增注释如下：
```php
/**
 * v1 module definition class
 *
 * @SWG\Swagger(
 *   swagger="2.0",
 *   @SWG\Info(
 *      title="安乐窝商品库 API(标题)",
 *      description="安乐窝商品库 API(描述)",
 *      termsOfService="安乐窝开发团队",
 *      version="1.0.0(版本号)",
 *      @SWG\Contact(
 *          email="2794408425@qq.com(邮件)",
 *          name="安乐窝(联系称呼)",
 *          url="http://api.app/v1/swagger-ui/dist/index.html"
 *      ),
 *      @SWG\License(
 *          name="MIT",
 *          url="http://github.com/gruntjs/grunt/blob/master/LICENSE-MIT"
 *      ),
 *  ),
 *  host="api.app/",
 *  basePath="v1/",
 *  schemes={"http"},
 *  produces={"application/json"},
 *  consumes={"application/json"},
 *      @SWG\Definition(
 *          definition="ErrorModel",
 *          required={"code", "message"},
 *          @SWG\Property(
 *              property="code",
 *              type="integer",
 *              format="int32"
 *          ),
 *          @SWG\Property(
 *              property="message",
 *              type="string"
 *          )
 *      ),
 * )
 */
```

Yii 2 的 RESTFul APT 会自动为控制器提供六个动作如：`index`、`view`、`create`、`update`、`delete`、`options`，给控制器 `app/v1/controlelr/OrderController.php` 新增以下注释，为其生成接口文档。
```php
/**
 * @SWG\Get(
 *     tags={"order"},
 *     path="/order",
 *     summary="列出所有的订单",
 *     operationId="index",
 *     description="列出所有的订单",
 *     @SWG\Parameter(
 *         name="access-token",
 *         in="query",
 *         description="Access Token for RESTFul API",
 *         required=true,
 *         type="string",
 *         format="string"
 *     ),
 *     @SWG\Response(
 *         response=200,
 *         description="Successful Operation",
 *     ),
 *     @SWG\Response(
 *         response="404",
 *         description="Not Found Operation",
 *         @SWG\Schema(
 *             ref="#/definitions/Error"
 *         )
 *     )
 * )
 */
```

打开控制器 `app/modules/v1/controller/UserController.php`，继承 actions() 方法，修改 `user/create` 的场景，并添加 `user/create` 文档生成注释代码，如下：
```php
<?php

namespace api\modules\v1\controllers;

use yii\rest\ActiveController;

class UserController extends ActiveController
{
    // 声明控制器所采用的数据模型
    public $modelClass = 'api\models\User';

    // 序列化输出
    public $serializer = [
        'class' => 'yii\rest\Serializer',
        'collectionEnvelope' => 'items',
    ];

    // 修改 user/create 的 rules 场景
    public function actions()
    {
        $actions = parent::actions();
        $actions['create']['scenario'] = 'createScenario';

        return $actions;
    }
}
/**
 * @SWG\Post(
 *   path="/user",
 *   tags={"user"},
 *   summary="创建一名用户",
 *   description="This can only be done by the logged in user.",
 *   operationId="create",
 *   @SWG\Parameter(
 *     in="path",
 *     name="username",
 *     default="LuisEdware",
 *     description="Created user object",
 *     required=true,
 *     type="string",
 *     format="string"
 *   ),
 *   @SWG\Parameter(
 *     in="path",
 *     name="auth_key",
 *     default="iwTNae9t34OmnK6l4vT4IeaTk-YWI2Rv",
 *     description="Created user object",
 *     required=true,
 *     type="string",
 *     format="string"
 *   ),
 *   @SWG\Parameter(
 *     in="path",
 *     name="password_hash",
 *     default="$2y$13$CXT0Rkle1EMJ/c1l5bylL.EylfmQ39O5JlHJVFpNn618OUS1HwaIi",
 *     description="Created user object",
 *     required=true,
 *     type="string",
 *     format="string"
 *   ),
 *  @SWG\Parameter(
 *     in="path",
 *     name="password_reset_token",
 *     default="t5GU9NwpuGYSfb7FEZMAxqtuz2PkEvv_1487320567",
 *     description="Created user object",
 *     required=true,
 *     type="string",
 *     format="string"
 *   ),
 *  @SWG\Parameter(
 *     in="path",
 *     name="email",
 *     default="2794408425@qq.com",
 *     description="Created user object",
 *     required=true,
 *     type="string",
 *     format="string"
 *   ),
 *  @SWG\Parameter(
 *     in="path",
 *     name="role",
 *     default="1",
 *     description="Created user object",
 *     required=true,
 *     type="integer",
 *     format="integer"
 *   ),
 *  @SWG\Parameter(
 *     in="path",
 *     name="status",
 *     default="1",
 *     description="Created user object",
 *     required=true,
 *     type="integer",
 *     format="integer"
 *   ),
 *  @SWG\Parameter(
 *     in="path",
 *     name="created_at",
 *     default="1391885313",
 *     description="Created user object",
 *     required=true,
 *     type="integer",
 *     format="integer"
 *   ),
 *   @SWG\Parameter(
 *     in="path",
 *     name="updated_at",
 *     default="1391885313",
 *     description="Created user object",
 *     required=true,
 *     type="integer",
 *     format="integer"
 *   ),
 *   @SWG\Parameter(
 *     in="path",
 *     name="access_token",
 *     default="HelloWorld",
 *     description="Created user object",
 *     required=true,
 *     type="string",
 *     format="string"
 *   ),
 *   @SWG\Parameter(
 *     in="query",
 *     name="access-token",
 *     default="HelloWorld",
 *     description="Created user object",
 *     required=true,
 *     type="string",
 *     format="string"
 *   ),
 *   @SWG\Response(response="default", description="successful operation")
 * )
 */
```

打开文件 `app/modules/v1/models/User.php`，新增代码如下：
```php
<?php
public function rules()
{
    return [
       [
           [
               'username',
               'auth_key',
               'password_hash',
               'password_reset_token',
               'email',
               'role',
               'status',
               'created_at',
               'updated_at',
               'access_token',
           ],
           'required',
           'on' => ['createScenario'],
       ]
    ];
}
```

在 `ap1/modules/v1` 下新增控制器 `MenuController.php`，新增模型 `Menu.php`，配置好路由，控制器和模型代码如下：
```php
<?php

namespace api\modules\v1\controllers;

use yii\rest\ActiveController;

class MenuController extends ActiveController
{
    // 声明控制器所采用的数据模型
    public $modelClass = 'api\modules\v1\models\Menu';

    // 序列化输出
    public $serializer = [
        'class' => 'yii\rest\Serializer',
        'collectionEnvelope' => 'items',
    ];

    public function actions()
    {
        $actions = parent::actions();
        $actions['create']['scenario'] = 'createScenario';

        return $actions;
    }
}
/**
 * @SWG\Post(
 *     path="/menu",
 *     tags={"menu"},
 *     operationId="createMenu",
 *     description="创建菜单",
 *     consumes={"application/json"},
 *     produces={"application/json"},
 *     @SWG\Parameter(
 *         name="body",
 *         in="body",
 *         description="menu to add to the store",
 *         required=true,
 *         @SWG\Schema(ref="#/definitions/menu"),
 *     ),
 *     @SWG\Parameter(
 *         in="query",
 *         name="access-token",
 *         default="HelloWorld",
 *         description="Created user object",
 *         required=true,
 *         type="string",
 *         format="string"
 *     ),
 *     @SWG\Response(
 *         response=200,
 *         description="pet response",
 *         @SWG\Schema(ref="#/definitions/menu")
 *     ),
 *     @SWG\Response(
 *         response="default",
 *         description="unexpected error",
 *         @SWG\Schema(ref="#/definitions/ErrorModel")
 *     )
 * )
 */

```
模型
```php
<?php

namespace api\modules\v1\models;

use yii\db\ActiveRecord;

class Menu extends ActiveRecord
{
    public static function tableName()
    {
        return "menu";
    }

    // 返回指定的字段
    public function fields()
    {
        return [
            'id',
            'name',
            'route',
        ];
    }

    public function rules()
    {
        return [
            [
                [
                    'name',
                    'parent',
                    'route',
                    'order',
                    'data',
                ],
                'required',
                'on' => ['createScenario'],
            ],
        ];
    }
}
```

在浏览器中打开 URL: `http://api.app/document/build/v1` 生成文档。

## API 授权

现在已经解决了 RESTFUL API 的脚手架、版本化、API 验证和 API 文档生成的问题，接下来我们来完成 API 授权功能。

首先我们需要建立授权数据，而所有授权数据相关的任务如下：

- 定义角色和权限；
- 建立角色和权限的关系；
- 定义规则；
- 将规则与角色和权限作关联；
- 指派角色给用户。

我们先配置所需的组件，我们使用 Yii2 自带 RBAC 组件进行权限控制，打开配置文件 `api/config/main.php`，在数组 `components` 新增代码如下：
```js
'authManager' => [
  'class' => 'yii\rbac\DbManager',
],
```

然后在根目录执行命令，在数据库创建好相关的数据表，执行命令如下：
```shell
php yii migrate --migrationPath=@yii/rbac/migrations
```

在项目根目录 `console/controllers` 文件夹新建控制器 `RbacController.php`，代码如下：
```php
<?php

namespace console\controllers;

use Yii;
use yii\console\Controller;

class RbacController extends Controller
{
    public function actionInit()
    {
        $authManager = Yii::$app->authManager;

        // 添加 "createMenu" 权限
        $createMenu = $authManager->createPermission('menu/create');
        $createMenu->description = 'Create a Menu';
        $authManager->add($createPost);

        // 添加 "updateMenu" 权限
        $updateMenu = $authManager->createPermission('menu/update');
        $updateMenu->description = 'Update Menu';
        $authManager->add($updateMenu);

        // 添加 "author" 角色并赋予 "menu/create" 权限
        $author = $authManager->createRole('author');
        $authManager->add($author);
        $authManager->addChild($author, $createPost);

        // 添加 "admin" 角色并赋予 "menu/update" 和 "author" 权限
        $admin = $authManager->createRole('admin');
        $authManager->add($admin);
        $authManager->addChild($admin, $updatePost);
        $authManager->addChild($admin, $author);

        // 为用户指派角色。其中 1 和 2 是由 IdentityInterface::getId() 返回的id （译者注：user表的id）
        $authManager->assign($admin, 1);
        $authManager->assign($author, 2);
    }
}
```

然后在根目录中执行命令 `php yii rbac/init`，填充 RBAC 的测试数据，并在文件 `v1/Module.php` 中更改代码如下:
```php
<?php
// v1 模块的请求都需要认证
public function behaviors()
{
    $behaviors = parent::behaviors();

    // API 认证
    $behaviors['authenticator'] = [
        'class' => QueryParamAuth::className(),
    ];

    // API 授权
    Yii::$app->user->setIdentity(User::findIdentityByAccessToken(Yii::$app->request->get('access-token')));
    $operation = Yii::$app->controller->id . '/' . Yii::$app->controller->action->id;

    if (!Yii::$app->user->can($operation)) {
        echo '没有' . $operation . '的操作权限';
    }
}
```

## 编写测试

如果是 RESTFul API 的架构，我会选择编写 API 测试和单元测试。首先先从 API 测试开始。首先在 `api` 中创建 API 测试配置文件,执行命令如下：
```
codecept generate:suite api
```

进入文件夹`api/tests`，打开文件 `api.suite.yml`，编写配置如下：
```
class_name: ApiTester
modules:
    enabled: [PhpBrowser, REST]
    config:
        PhpBrowser:
            url: http://api.app
        REST:
            url: http://api.app
            depends: PhpBrowser
```

在文件夹 `api` 生成 API 测试文件，执行命令如下：
```
codecept generate:cept api/v1 SearchMenu
codecept generate:cept api/v1 CreateMenu
```

打开文件 `api/tests/api/v1/SearchMenuCept.php`，编写代码如下：
```php
<?php
use api\tests\ApiTester;
use Codeception\Util\HttpCode;

$I = new ApiTester($scenario);
$I->wantTo('search menus');
$I->haveHttpHeader('Content-Type', 'application/json');
$I->sendGet('/v1/menu', ['access-token' => "HelloWorld"]);
$I->seeResponseCodeIs(HttpCode::OK);
$I->seeResponseIsJson();
$I->seeResponseMatchesJsonType([
    'items' => 'array',
    '_links' => 'array',
    '_meta' => [
        'totalCount' => 'integer',
        'pageCount' => 'integer',
        'currentPage' => 'integer',
        'perPage' => 'integer',
    ],
]);
```

打开文件 `api/tests/api/v1/CreateMenuCept.php`，编写代码如下：
```php
<?php
use api\tests\ApiTester;

$I = new ApiTester($scenario);
$I->wantTo('create menu');
$I->haveHttpHeader('Content-Type', 'application/json');
$I->sendPOST('/v1/menu?access-token=HelloWorld', [
    'name' => '歪理兔',
    'parent' => '1',
    'route' => '/go/to/very-to',
    'order' => 0,
    'data' => 'hello',
]);
$I->seeResponseCodeIs(201);
$I->seeResponseIsJson();
$I->seeResponseMatchesJsonType([
    'id' => 'integer',
    'name' => 'string',
    'route' => 'string',
]);
```

接着构建测试环境，并运行所编写的 API 测试，执行命令如下：
```
codecept build
codecept run
```

## 错误处理

#### Yii 2 的 REST 框架的 HTTP 状态代码如下：

- 200: OK。一切正常。
- 201: 响应 POST 请求时成功创建一个资源。Location header 包含的URL指向新创建的资源。
- 204: 该请求被成功处理，响应不包含正文内容 (类似 DELETE 请求)。
- 304: 资源没有被修改。可以使用缓存的版本。
- 400: 错误的请求。可能通过用户方面的多种原因引起的，例如在请求体内有无效的JSON 数据，无效的操作参数，等等。
- 401: 验证失败。
- 403: 已经经过身份验证的用户不允许访问指定的 API 末端。
- 404: 所请求的资源不存在。
- 405: 不被允许的方法。 请检查 Allow header 允许的HTTP方法。
- 415: 不支持的媒体类型。 所请求的内容类型或版本号是无效的。
- 422: 数据验证失败 (例如，响应一个 POST 请求)。 请检查响应体内详细的错误消息。
- 429: 请求过多。 由于限速请求被拒绝。
- 500: 内部服务器错误。 这可能是由于内部程序错误引起的。

#### 对应的 class 使用如下:

- BadRequestHttpException - 400
- UnauthorizedHttpException - 401
- ForbiddenHttpException - 403
- NotFoundHttpException - 404
- MethodNotAllowedHttpException - 405
- UnsupportedMediaTypeHttpException - 415
- UnprocessableEntityHttpException - 422
- TooManyRequestsHttpException - 429
- ServerErrorHttpException - 500


