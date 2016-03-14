![Happy Coding](http://upload-images.jianshu.io/upload_images/1277229-a09205aecddd5bf8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 本文目标

*** 

完成登录管理页面与登录验证功能

# 管理用户模型

***

在文件夹`app`下新建文件夹`Models`，将文件夹`app`下的`User.php`移动到文件夹`app/Models`下
修改文件`User.php`的命名空间
```
namespace App\Models;
```

打开文件`config/auth.php`，修改代码如下
```
'model' => App\Models\User::class,
```

# 新增路由

***
打开文件`app/Http/routes.php`，文件代码如下：
```
<?php
Route::group(['namespace' => 'Backend', 'middleware' => ['auth']], function () {
    Route::get('/', 'IndexController@index');
});

Route::group(['namespace' => 'Auth'], function () {
    Route::get('auth/login', 'AuthController@getLogin');
    Route::post('auth/login', 'AuthController@postLogin');
    Route::get('auth/logout', 'AuthController@getLogout');
});
```

# 新增登录视图

***

在文件夹`resources/views/`新建文件夹`auth`，然后新建视图文件`login.blade.php`
在文件夹`resources/views/backend/layout`新建文件夹`message`，然后新建视图文件如下：

- tips.blade.php
- error.blade.php
- success.blade.php

## login.blade.php
```
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title>AdminLTE 2 | Log in</title>
    <meta content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no" name="viewport">
    <link href="{{ asset("/assets/css/app.css") }}" rel="stylesheet" type="text/css"/>
</head>
<body class="hold-transition login-page">
<div class="login-box">
    <div class="login-logo">
        <b>TianMao</b>CMF
    </div>
    <div class="login-box-body">
        <p class="login-box-msg">Happy Coding</p>

        <form action="{{URL::to('/auth/login')}}" method="post" enctype="multipart/form-data">
            <input type="hidden" name="_token" value="{{ csrf_token() }}">
            <div class="row form-group has-feedback">
                <input type="text" class="form-control" placeholder="账号" name="email">
                <span class="fa fa-user form-control-feedback"></span>
                @include('backend.layout.message.tips',['field'=>'email'])
            </div>
            <div class="row form-group has-feedback">
                <input type="password" class="form-control" placeholder="密码" name="password">
                <span class="fa fa-lock form-control-feedback"></span>
                @include('backend.layout.message.tips',['field'=>'password'])
            </div>
            <div class="row form-group has-feedback">
                <input type="text" class="form-control" placeholder="验证码" name="captcha">
                <span class="fa fa-image form-control-feedback"></span>
                @include('backend.layout.message.tips',['field'=>'captcha'])
            </div>
            <div class="row form-group has-feedback">
                <img src="{{$captcha}}" alt="图片验证码" style="width: 100%;">
            </div>
            <div class="row">
                <div class="col-xs-8">
                    <div class="checkbox icheck">
                        <label>
                            <input type="checkbox" name="remember" value="1"> 保持登录
                        </label>
                    </div>
                </div>
                <div class="col-xs-4">
                    <button type="submit" class="btn btn-primary btn-block btn-flat">登 录</button>
                </div>
            </div>
        </form>
    </div>
</div>
<script src="{{ asset ("/assets/js/app.js") }}"></script>
<script>
    $(function () {
        $('input').iCheck({
            checkboxClass: 'icheckbox_square-blue',
            radioClass: 'iradio_square-blue',
            increaseArea: '20%'
        });
    });
</script>
</body>
</html>
```

## tips.blade.php
```
@if($errors->has($field))
	@foreach ($errors->get($field) as $error)
		<button class="btn btn-danger col-sm-12 btn-flat" style="width: 100%;">
			<i class="fa fa-times-circle-o"></i> {{$error}}
		</button>
	@endforeach
@endif
```

## error.blade.php
```
@if(Session::has('errors'))
	<div id="errors-message" class="alert alert-danger alert-dismissible">
		<button type="button" class="close" data-dismiss="alert" aria-hidden="true">×</button>
		<h4><i class="icon fa fa-ban"></i> 错误提示!</h4>
		@foreach($errors->all() as $error)
			{{$error}}
		@endforeach
	</div>
@endif
```

## success.blade.php
```
@if(Session::has('success'))
	<div id="success-message" class="alert alert-success alert-dismissible">
		<button type="button" class="close" data-dismiss="alert" aria-hidden="true">×</button>
		<h4><i class="icon fa fa-check"></i> 成功提示!</h4>
		{{Session::get('success')}}
	</div>
@endif
```

# 新增验证码Package

***

打开`composer.json`文件,修改代码如下:
```
 "require": {
    "php": ">=5.5.9",
    "laravel/framework": "5.1.*",
    "gregwar/captcha": "1.*"
},
```

`gregwar/captcha`是验证码Package，用来生成验证码，在终端执行以下命令进行安装。
```
sudo composer install
```

# 管理登录认证控制器

打开文件`app/Http/Auth/AuthController.php`，新增代码如下:
```
use Illuminate\Support\Facades\Session;

public function getLogin()
{
    if (view()->exists('auth.authenticate')) {
        return view('auth.authenticate');
    }

    $builder = new CaptchaBuilder();
    $builder->build();
    Session::put('phrase', $builder->getPhrase());

    return view('auth.login')->with('captcha', $builder->inline());
}

public function postLogin(Request $request)
{
    $this->validate($request, [
        $this->loginUsername() => 'required',
        'password'             => 'required',
        'captcha'              => 'required'
    ], [
        $this->loginUsername() . '.required' => '请输入邮箱或用户名称',
        'password.required'                  => '请输入用户密码',
        'captcha.required'                   => '请输入验证码'
    ]);

    if (Session::get('phrase') !== $request->get('captcha')) {
        return redirect()->back()->withErrors(array('captcha' => '验证码不正确'))->withInput();
    }
}
```

# 使用迁移文件生成数据表

***

接下来打开终端，在项目下运行以下命令，生成用户表并创建用户
```
// 生成数据表
php artisan migrate

// 生成用户数据
php artisan tinker
$user = new App\Models\User;
$user->name = "admin";
$user->email = "admin@qq.com";
$user->password = bcrypt(123456);
$user->save();
exit
```

# 管理登录认证操作跳转页面

打开文件`app\Http\Controller\Auth\AuthController`，新增以下成员属性:
```
// 设置成功登录后转向的页面：
protected $redirectPath = '/';

// 设置登录失败后转向的页面：
protected $loginPath = '/auth/login';

// 设置退出登录后转向的页面：
protected $redirectAfterLogout = '/';
```
打开中间件`app\Http\Middleware\RedirectIfAuthenticated.php`，找到相应的地方，修改代码如下：

```
 public function handle($request, Closure $next)
{
    // 设置登录之后，输入/auth/login的跳转地址
    if ($this->auth->check()) {
        return redirect('/');
    }

    return $next($request);
}
```

# 效果预览
完成上述步骤后，在浏览器输入项目域名/auth/login，可以看到效果如下：
![屏幕快照 2016-02-18 11.38.16.png](http://upload-images.jianshu.io/upload_images/1277229-7733789649e84078.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
