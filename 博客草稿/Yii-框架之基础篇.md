---
title: Yii 框架之基础篇
date: 2016-10-25 21:59:58
tags: Yii
category: PHP
---

## 安装
检查当前 PHP 环境是否满足 Yii 最基本需求：
```
php requirements.php
```

通过 Composer 进行安装
```
// 安装基础应用模板
sudo composer create-project --prefer-dist yiisoft/yii2-app-basic yii

// 安装高级应用模板
sudo composer create-project --prefer-dist yiisoft/yii2-app-advanced advanced
```

如果安装的过程中提示需要 Github Token，则进入以下网址生成 Token
```
https://github.com/settings/tokens
```

## Packages
```shell
// AdminLte 后台响应模板
composer require dmstr/yii2-adminlte-asset "2.*"

// Yii2 RBAC 的一套管理工具，实现了漂亮的界面和完整的权限管理功能
composer require mdmsoft/yii2-admin "2.x-dev"
```

## 控制器

### 操作

控制器由操作组成，它是执行终端用户请求的最基础的单元，一个控制器可有一个或多个操作。

操作分为独立操作和内联操作

内联操作容易创建，在无需重用的情况下优先使用； 独立操作相反，主要用于多个控制器重用， 或重构为扩展

### 路由

终端用户通过所谓的路由寻找到操作，路由是包含以下部分的字符串：

- 模块 ID
- 控制器 ID
- 操作 ID

路由两种主要的格式
```php
# 默认目录下的路由
ControllerID/ActionID

# 模块目录下的路由
ModuleID/ControllerID/ActionID
```

### 部署

在应用配置文件 `web.php` 配置默认控制器，当请求没有指定路由，该属性值作为路由使用:
```php
<?php

[
	'defaultRoute' => 'main',
]
```

在应用配置文件 `web.php` 强制控制器 ID 和类名对应，通常用在使用第三方不能掌控类名的控制器上
```php
<?php

[
    'controllerMap' => [
        // 用类名申明 "account" 控制器
        'account' => 'app\controllers\UserController',

        // 用配置数组申明 "article" 控制器
        'article' => [
            'class' => 'app\controllers\PostController',
            'enableCsrfValidation' => false,
        ],
    ],
]
```

### 操作结果
```
// 返回响应对象，进行页面跳转
$this->redirect('http://example.com');
```

### 操作参数
```
namespace app\controllers;

use yii\web\Controller;

class PostController extends Controller
{
    public function actionView(int $id, $version = null)
    {
        // ...
    }
}
```

操作参数会被不同的参数填入，如下所示：

- http://hostname/index.php?r=post/view&id=123: $id 会填入'123'， $version 仍为 null 空因为没有version请求参数;
- http://hostname/index.php?r=post/view&id=123&version=2: $id 和 $version 分别填入 '123' 和 '2'`；
- http://hostname/index.php?r=post/view: 会抛出yii\web\BadRequestHttpException 异常 因为请求没有提供参数给必须赋值参数$id；
- http://hostname/index.php?r=post/view&id[]=123: 会抛出yii\web\BadRequestHttpException 异常 因为$id 参数收到数字值 ['123']而不是字符串.

### 默认操作

每个控制器都有一个由 yii\base\Controller::defaultAction 属性指定的默认操作，当路由只包含控制器ID， 会使用所请求的控制器的默认操作。
```php
<?php
namespace app\controllers;

use yii\web\Controller;

class SiteController extends Controller
{
    public $defaultAction = 'home';

    public function actionHome()
    {
        return $this->render('home');
    }
}
```

## 模型

### 属性

模型通过属性来代表业务数据，每个属性像是模型的公有可访问属性，yii\base\Model::attributes() 指定模型所拥有的属性。
```php
<?php

$model = new \app\models\ContactForm;

// "name" 是ContactForm模型的属性
$model->name = 'example';
echo $model->name;
```

模型实现了 ArrayAccess 和 ArrayIterator 接口，可以像数组单元项一样访问属性
```php
<?php

$model = new \app\models\ContactForm;

// 像访问数组单元项一样访问属性
$model['name'] = 'example';
echo $model['name'];

// 迭代器遍历模型
foreach ($model as $name => $value) {
    echo "$name: $value\n";
}
```

### 属性标签

如果你不想用自动生成的标签， 可以覆盖 yii\base\Model::attributeLabels() 方法明确指定属性标签，例如：
```php
<?php
namespace app\models;

use yii\base\Model;

class ContactForm extends Model
{
    public $name;
    public $email;
    public $subject;
    public $body;

    public function attributeLabels()
    {
        return [
            'name' => 'Your name',
            'email' => 'Your email address',
            'subject' => 'Subject',
            'body' => 'Content',
        ];
    }
}
```

### 场景
模型可能在多个场景下使用，例如 User 模块可能会在收集用户登录输入，也可能会在用户注册时使用。在不同的场景下，模型可能会使用不同的业务规则和逻辑，例如 email 属性在注册时强制要求有，但在登陆时不需要。模型设置场景如下：
```php
<?php

// 场景作为属性来设置
$model = new User;
$model->scenario = 'login';

// 场景通过构造初始化配置来设置
$model = new User(['scenario' => 'login']);
```

场景设置活动属性和验证规则
```php
<?php
namespace app\models;

use yii\db\ActiveRecord;

class User extends ActiveRecord
{
    const SCENARIO_LOGIN = 'login';
    const SCENARIO_REGISTER = 'register';

    public function scenarios()
    {
        $scenarios = parent::scenarios();
        $scenarios[self::SCENARIO_LOGIN] = ['username', 'password'];
        $scenarios[self::SCENARIO_REGISTER] = ['username', 'email', 'password'];
        return $scenarios;
    }

	public function rules()
	{
	    return [
	        // 在"register" 场景下 username, email 和 password 必须有值
	        [['username', 'email', 'password'], 'required', 'on' => 'register'],

	        // 在 "login" 场景下 username 和 password 必须有值
	        [['username', 'password'], 'required', 'on' => 'login'],
	    ];
	}
}
```

### 块赋值

块赋值只用一行代码将用户所有输入填充到一个模型，非常方便，它直接将输入数据对应填充到 yii\base\Model::attributes 属性。以下两段代码效果是相同的，都是将终端用户输入的表单数据赋值到 ContactForm 模型的属性，明显地前一段块赋值的代码比后一段代码简洁且不易出错。
```php
<?php

# 块赋值
$model = new \app\models\ContactForm;
$model->attributes = \Yii::$app->request->post('ContactForm');

# 普通赋值

$model = new \app\models\ContactForm;
$data = \Yii::$app->request->post('ContactForm', []);
$model->name = isset($data['name']) ? $data['name'] : null;
$model->email = isset($data['email']) ? $data['email'] : null;
$model->subject = isset($data['subject']) ? $data['subject'] : null;
$model->body = isset($data['body']) ? $data['body'] : null;
```

### 对象转换数组

```php
<?php

$post = \app\models\Post::findOne(100);
$array = $post->attributes;
```

### 字段
```php
<?php

$array = $model->toArray([], ['prettyName', 'fullAddress']);

// 明确列出每个字段，特别用于你想确保数据表或模型
// 属性改变不会导致你的字段改变(保证后端的API兼容)。
public function fields()
{
    return [
        // 字段名和属性名相同
        'id',

        // 字段名为 "email"，对应属性名为 "email_address"
        'email' => 'email_address',

        // 字段名为 "name", 值通过PHP代码返回
        'name' => function () {
            return $this->first_name . ' ' . $this->last_name;
        },
    ];
}

// 过滤掉一些字段，特别用于你想
// 继承父类实现并不想用一些敏感字段
public function fields()
{
    $fields = parent::fields();

    // 去掉一些包含敏感信息的字段
    unset($fields['auth_key'], $fields['password_hash'], $fields['password_reset_token']);

    return $fields;
}
```

## 视图

### 控制器渲染视图的方法

```php
<?php

// 渲染一个视图名并使用一个布局返回到渲染结果
yii\base\Controller::render();

// 渲染一个视图名并且不使用布局
yii\base\Controller::renderPartial();

// 渲染一个视图名并且不使用布局，并注入所有注册的JS/CSS脚本和文件，通常使用在响应AJAX网页请求的情况下。
yii\base\Controller::renderAjax();

// 渲染一个视图文件目录或别名下的视图文件。
yii\base\Controller::renderFile();

yii\base\Controller::renderContent();
```

### 小部件渲染视图的方法

```php
<?php

// 渲染一个视图名
yii\base\Widget::render()

// 渲染一个视图文件目录或别名下的视图文件
yii\base\Widget::renderFile()
```

### 视图中访问数据
```js
$this->render('视图名',['key'=>$value]);

echo $this->key;
```

### 视图间共享数据
yii\base\View 视图组件提供yii\base\View::params参数 属性来让不同视图共享数据。
```js
$this->params['breadcrumbs'][] = 'About Us';
```

### 布局
布局的渲染过程
```php
<?php
// 该方法应在布局的开始处调用， 它触发表明页面开始的 yii\base\View::EVENT_BEGIN_PAGE 事件。
yii\base\View::beginPage();

// 该方法应在布局的结尾处调用， 它触发表明页面结尾的 yii\base\View::EVENT_END_PAGE 时间。
yii\base\View::endPage();

// 该方法应在HTML页面的<head>标签中调用， 它生成一个占位符，在页面渲染结束时会被注册的头部HTML代码 （如，link标签, meta标签）替换。
yii\web\View::head();

// 该方法应在<body>标签的开始处调用， 它触发 yii\web\View::EVENT_BEGIN_BODY 事件并生成一个占位符， 会被注册的HTML代码（如JavaScript）在页面主体开始处替换。
yii\web\View::beginBody();

// 方法应在<body>标签的结尾处调用， 它触发 yii\web\View::EVENT_END_BODY 事件并生成一个占位符， 会被注册的HTML代码（如JavaScript）在页面主体结尾处替换。
yii\web\View::endBody()
```

布局中访问数据
在布局中可访问两个预定义变量：$this 和 $content， 前者对应和普通视图类似的 yii\base\View 视图组件 后者包含调用 yii\base\Controller::render() 方法渲染内容视图的结果。

使用布局

- yii\base\Application::layout 管理所有控制器的布局
- yii\base\Controller::layout 覆盖总布局来控制单个控制器布局

嵌套布局

布局数据块

### 使用视图组件

### 设置页面标题
```php
<?php
	$this->title = 'My page title';
?>

<title><?= Html::encode($this->title) ?></title>
```

### 注册 Meta 元标签
```php
<?php
	$this->registerMetaTag(['name' => 'keywords', 'content' => 'yii, framework, php']);
?>
```

### 注册链接标签
```php
<?php
	$this->registerLinkTag([
	    'title' => 'Live News for Yii',
	    'rel' => 'alternate',
	    'type' => 'application/rss+xml',
	    'href' => 'http://www.yiiframework.com/rss.xml/',
	]);
?>
```

### 视图事件
```php
<?php
// 在控制器渲染文件开始时触发，该事件可设置 yii\base\ViewEvent::isValid 为 false 取消视图渲染。
yii\base\View::EVENT_BEFORE_RENDER:

// 在布局中调用 yii\base\View::beginPage() 时触发，该事件可获取 yii\base\ViewEvent::output 的渲染结果，可修改该属性来修改渲染结果。
yii\base\View::EVENT_AFTER_RENDER:

// 在布局调用 yii\base\View::beginPage() 时触发
yii\base\View::EVENT_BEGIN_PAGE:

// 在布局调用 yii\base\View::endPage() 是触发
yii\base\View::EVENT_END_PAGE:

// 在布局调用 yii\web\View::beginBody() 时触发
yii\web\View::EVENT_BEGIN_BODY:

// 在布局调用 yii\web\View::endBody() 时触发
yii\web\View::EVENT_END_BODY:
```

## 过滤器（中间件）

过滤器是控制器动作执行之前或之后执行的对象。例如访问控制过滤器可在动作执行之前来控制特殊终端用户是否有权限执行动作， 内容压缩过滤器可在动作执行之后发给终端用户之前压缩响应内容。

继承 yii\base\ActionFilter 类并覆盖 yii\base\ActionFilter::beforeAction() 和/或 yii\base\ActionFilter::afterAction() 方法来创建动作的过滤器，前者在动作执行之前执行，后者在动作执行之后执行。 yii\base\ActionFilter::beforeAction() 返回值决定动作是否应该执行， 如果为 false，之后的过滤器和动作不会继续执行。

### 核心过滤器

- yii\filters\AccessControl
	- AccessControl提供基于yii\filters\AccessControl::rules规则的访问控制
- yii\filters\auth\HttpBasicAuth
	- 认证方法过滤器通过HTTP Basic Auth或OAuth 2 来认证一个用户， 认证方法过滤器类在 yii\filters\auth 命名空间下
- yii\filters\ContentNegotiator
	- ContentNegotiator支持响应内容格式处理和语言处理
- yii\filters\HttpCache
	- HttpCache利用Last-Modified 和 Etag HTTP头实现客户端缓存
- yii\filters\PageCache
	- PageCache实现服务器端整个页面的缓存
- yii\filters\RateLimiter
	- RateLimiter 根据 漏桶算法 来实现速率限制。 主要用在实现RESTful APIs， 更多关于该过滤器详情请参阅 Rate Limiting 一节
- yii\filters\VerbFilter
	- VerbFilter检查请求动作的HTTP请求方式是否允许执行， 如果不允许，会抛出HTTP 405异常
- yii\filters\Cors
	- 跨域资源共享 CORS 机制允许一个网页的许多资源（例如字体、JavaScript等） 这些资源可以通过其他域名访问获取


## 请求

### 请求基础用法
```php
<?php

$request = Yii::$app->request;

// 等价于: $get = $_GET;
$get = $request->get();

// 等价于: $id = isset($_GET['id']) ? $_GET['id'] : null;
$id = $request->get('id');

// 等价于: $id = isset($_GET['id']) ? $_GET['id'] : 1;
$id = $request->get('id', 1);

// 等价于: $post = $_POST;
$post = $request->post();

// 等价于: $name = isset($_POST['name']) ? $_POST['name'] : null;
$name = $request->post('name');

// 等价于: $name = isset($_POST['name']) ? $_POST['name'] : '';
$name = $request->post('name', '');

// 返回所有参数
$params = $request->bodyParams;

// 返回参数 "id"
$param = $request->getBodyParam('id');

/* 该请求是一个 AJAX 请求 */
if ($request->isAjax) {
}

/* 请求方法是 GET */
if ($request->isGet)  {
}

/* 请求方法是 POST */
if ($request->isPost) {

}

/* 请求方法是 PUT */
if ($request->isPut)  {

}
```

### 请求 URLs

- **yii\web\Request::url**
    - 返回 /admin/index.php/product?id=100, 此URL不包括host info部分。
- **yii\web\Request::absoluteUrl**
    - 返回 http://example.com/admin/index.php/product?id=100, 包含host infode的整个URL。
- **yii\web\Request::hostInfo**
    - 返回 http://example.com, 只有host info部分。
- **yii\web\Request::pathInfo**
    - 返回 /product， 这个是入口脚本之后，问号之前（查询字符串）的部分。
- **yii\web\Request::queryString**
    - 返回 id=100,问号之后的部分。
- **yii\web\Request::baseUrl**
    - 返回 /admin, host info之后， 入口脚本之前的部分。
- **yii\web\Request::scriptUrl**
    - 返回 /admin/index.php, 没有path info和查询字符串部分。
- **yii\web\Request::serverName**
    - 返回 example.com, URL中的host name。
- **yii\web\Request::serverPort**
    - 返回 80, 这是web服务中使用的端口。


### HTTP 头

可以通过 yii\web\Request::headers 属性返回的 yii\web\HeaderCollection 获取HTTP头信息
```php
<?php

// $headers 是一个 yii\web\HeaderCollection 对象
$headers = Yii::$app->request->headers;

// 返回 Accept header 值
$accept = $headers->get('Accept');

/* 这是一个 User-Agent 头 */
if ($headers->has('User-Agent')) {

}
```

### 客户端信息

可以通过 `yii\web\Request::userHost` 和 `yii\web\Request::userIP` 分别获取 host-name 和客户机的 IP 地址
```php
<?php

$userHost = Yii::$app->request->userHost;
$userIP = Yii::$app->request->userIP;
```

## 响应

### 状态码

构建响应时，最先应做的是标识请求是否成功处理的状态，可通过设置 yii\web\Response::statusCode 属性，该属性使用一个有效的 HTTP 状态码
```
Yii::$app->response->statusCode = 200;
```

当错误处理器 捕获到一个异常，会从异常中提取状态码并赋值到响应， 对于上述的 yii\web\NotFoundHttpException 对应HTTP 404状态码， 以下为Yii预定义的HTTP异常
```
throw new \yii\web\NotFoundHttpException;
```
以下为Yii预定义的HTTP异常：

- **yii\web\BadRequestHttpException**
    - status code 400.
- **yii\web\ConflictHttpException**
    - status code 409.
- **yii\web\ForbiddenHttpException**
    - status code 403.
- **yii\web\GoneHttpException**
    - status code 410.
- **yii\web\MethodNotAllowedHttpException**
    - status code 405.
- **yii\web\NotAcceptableHttpException**
    - status code 406.
- **yii\web\NotFoundHttpException**
    - status code 404.
- **yii\web\ServerErrorHttpException**
    - status code 500.
- **yii\web\TooManyRequestsHttpException**
    - status code 429.
- **yii\web\UnauthorizedHttpException**
    - status code 401.
- **yii\web\UnsupportedMediaTypeHttpException**
    - status code 415.

如果想抛出的异常不在如上列表中，可创建一个yii\web\HttpException异常， 带上状态码抛出，如下：
```php
throw new \yii\web\HttpException(402);
```

### HTTP 头部

```php
<?php

$headers = Yii::$app->response->headers;

// 增加一个 Pragma 头，已存在的Pragma 头不会被覆盖。
$headers->add('Pragma', 'no-cache');

// 设置一个Pragma 头. 任何已存在的Pragma 头都会被丢弃
$headers->set('Pragma', 'no-cache');

// 删除Pragma 头并返回删除的Pragma 头的值到数组
$values = $headers->remove('Pragma');
```

### 响应主体

如果在发送给终端用户之前需要格式化，应设置 yii\web\Response::format 和 yii\web\Response::data 属性，yii\web\Response::format 属性指定 yii\web\Response::data 中数据格式化后的样式，例如：
```php
<?php

$response = Yii::$app->response;
$response->format = \yii\web\Response::FORMAT_JSON;
$response->data = ['message' => 'hello world'];
```

Yii支持以下可直接使用的格式，每个实现了 yii\web\ResponseFormatterInterface 类， 可自定义这些格式器或通过配置 yii\web\Response::formatters 属性来增加格式器。

- **yii\web\Response::FORMAT_HTML**
- **yii\web\Response::FORMAT_XML**
- **yii\web\Response::FORMAT_JSON**
- **yii\web\Response::FORMAT_JSONP**

正规格式返回 Yii 的 HTTP 响应
```php
<?php

 return \Yii::createObject([
    'class' => 'yii\web\Response',
    'format' => \yii\web\Response::FORMAT_JSON,
    'data' => [
        'message' => 'hello world',
        'code' => 100,
    ],
]);
```

### 浏览器跳转
```php
<?php

public function actionOld()
{
    return $this->redirect('http://example.com/new', 301);

    \Yii::$app->response->redirect('http://example.com/new', 301)->send();
}
```

### 发送文件

```php
<?php

public function actionDownload()
{
    return \Yii::$app->response->sendFile('path/to/file.txt');

    \Yii::$app->response->sendFile('path/to/file.txt')->send();
}


// 发送一个已存在的文件到客户端
yii\web\Response::sendFile()

// 发送一个文本字符串作为文件到客户端
yii\web\Response::sendContentAsFile()

// 发送一个已存在的文件流作为文件到客户端
yii\web\Response::sendStreamAsFile()
```

## Session
```php
<?php

$session = Yii::$app->session;

// 检查session是否开启
if ($session->isActive) {

}

// 开启session
$session->open();

// 关闭session
$session->close();

// 销毁session中所有已注册的数据
$session->destroy();

// 获取session中的变量值，以下用法是相同的：
$language = $session->get('language');
$language = $session['language'];
$language = isset($_SESSION['language']) ? $_SESSION['language'] : null;

// 设置一个session变量，以下用法是相同的：
$session->set('language', 'en-US');
$session['language'] = 'en-US';
$_SESSION['language'] = 'en-US';

// 删除一个session变量，以下用法是相同的：
$session->remove('language');
unset($session['language']);
unset($_SESSION['language']);

// 检查session变量是否已存在，以下用法是相同的：
if ($session->has('language')) ...
if (isset($session['language'])) ...
if (isset($_SESSION['language'])) ...

// 遍历所有session变量，以下用法是相同的：
foreach ($session as $name => $value) ...
foreach ($_SESSION as $name => $value) ...

// 直接使用$_SESSION (确保Yii::$app->session->open() 已经调用)
$_SESSION['captcha']['number'] = 5;
$_SESSION['captcha']['lifetime'] = 3600;

// 先获取session数据到一个数组，修改数组的值，然后保存数组到session中
$captcha = $session['captcha'];
$captcha['number'] = 5;
$captcha['lifetime'] = 3600;
$session['captcha'] = $captcha;

// 使用ArrayObject 数组对象代替数组
$session['captcha'] = new \ArrayObject;
...
$session['captcha']['number'] = 5;
$session['captcha']['lifetime'] = 3600;

// 使用带通用前缀的键来存储数组
$session['captcha.number'] = 5;
$session['captcha.lifetime'] = 3600;
```

### 自定义 Session 存储

Yii提供以下session类实现不同的session存储方式：

- yii\web\DbSession
    - 存储session数据在数据表中
- yii\web\CacheSession
    - 存储session数据到缓存中，缓存和配置中的缓存组件相关
- yii\redis\Session
    - 存储session数据到以redis 作为存储媒介中
- yii\mongodb\Session
    - 存储session数据到MongoDB

### Flash 数据

Flash数据是一种特别的session数据，它一旦在某个请求中设置后， 只会在下次请求中有效，然后该数据就会自动被删除。 常用于实现只需显示给终端用户一次的信息， 如用户提交一个表单后显示确认信息。
```php
<?php

$session = Yii::$app->session;

// 请求 #1
// 设置一个名为"postDeleted" flash 信息
$session->setFlash('postDeleted', 'You have successfully deleted your post.');

// 请求 #2
// 显示名为"postDeleted" flash 信息
echo $session->getFlash('postDeleted');

// 请求 #3
// $result 为 false，因为flash信息已被自动删除
$result = $session->hasFlash('postDeleted');

// 请求 #4
// 在名称为"alerts"的flash信息增加数据
$session->addFlash('alerts', 'You have successfully deleted your post.');
$session->addFlash('alerts', 'You have successfully added a new friend.');
$session->addFlash('alerts', 'You are promoted.');

// 请求 #5
// $alerts 为名为'alerts'的flash信息，为数组格式
$alerts = $session->getFlash('alerts');
```


## Cookie

Yii使用 yii\web\Cookie对象来代表每个cookie，yii\web\Request 和 yii\web\Response 通过名为'cookies'的属性维护一个cookie集合， 前者的cookie 集合代表请求提交的cookies， 后者的cookie集合表示发送给用户的cookies。

### 读取 Cookies

当前请求的cookie信息可通过如下代码获取：
```php
<?php

// 从 "request"组件中获取cookie集合(yii\web\CookieCollection)
$cookies = Yii::$app->request->cookies;

// 获取名为 "language" cookie 的值，如果不存在，返回默认值"en"
$language = $cookies->getValue('language', 'en');

// 另一种方式获取名为 "language" cookie 的值
if (($cookie = $cookies->get('language')) !== null) {
    $language = $cookie->value;
}

// 可将 $cookies当作数组使用
if (isset($cookies['language'])) {
    $language = $cookies['language']->value;
}

// 判断是否存在名为"language" 的 cookie
if ($cookies->has('language')) ...
if (isset($cookies['language'])) ...
```

### 发送 Cookies

可使用如下代码发送 cookie 到终端用户
```php
<?php

// 从"response"组件中获取cookie 集合(yii\web\CookieCollection)
$cookies = Yii::$app->response->cookies;

// 在要发送的响应中添加一个新的cookie
$cookies->add(new \yii\web\Cookie([
    'name' => 'language',
    'value' => 'zh-CN',
]));

// 删除一个cookie
$cookies->remove('language');

// 等同于以下删除代码
unset($cookies['language']);
```


## 分页

```php
<?php
use yii\data\Pagination;

public function actionCountry()
{
    $query = Country::find();

    $pagination = new Pagination([
        'defaultPageSize' => 5,
        'totalCount'      => $query->count(),
    ]);

    $countries = $query->orderBy('name')
        ->offset($pagination->offset)
        ->limit($pagination->limit)
        ->all();

    return $this->render('country', compact('countries', 'pagination'));
}
```

## 应用主体和应用组件



## 应用属性

### 必要属性

- id：用来区分其他应用的唯一标识ID。
- basePath ：指定该应用的根目录。

### 重要属性

- aliases：该属性允许你用一个数组定义多个别名。 数组的 key 为别名名称，值为对应的路径。
- bootstrap ：这个属性很实用，它允许你用数组指定启动阶段 yii\base\Application::bootstrap() 需要运行的组件。
- catchAll：该属性仅网页应用支持。 它指定一个要处理所有用户请求的控制器方法。
- components：允许你注册多个在其他地方使用的应用组件。
- controllerMap：该属性允许你指定一个控制器 ID 到任意控制器类。
- controllerNamespace：该属性指定控制器类默认的命名空间，默认为 app\controllers。
- language：该属性指定应用展示给终端用户的语言， 默认为 en 标识英文
- modules ：该属性指定应用所包含的模块。

## 前端资源

### 资源

### 资源包

### 定义资源包

## 表单

ActiveForm::begin()，ActiveForm::end() 分别用来渲染表单的开始和关闭标签

## 查询构建器
