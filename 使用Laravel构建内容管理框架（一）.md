# 安装Laravel

***

在终端使用Composer安装Laravel
```
sudo composer create-project laravel/laravel backend --prefer-dist 5.1.x
```
执行后，会在当前目录下新建一个文件夹，里面就是新建的Laravel项目

# 安装gulp、bower

***

在终端使用npm安装gulp
```
sudo npm install -g gulp  #全局安装npm
sudo npm install -g bower #全局安装bower

cd backend
sudo npm install gulp   #在项目本地安装gulp
sudo npm install bower  #在项目本地安装bower
sudp npm install        #安装项目Node依赖、Laravel Elixir

```

运行`gulp tdd`之后，会自动监控测试单元



# 使用bower集成前端依赖库

***

在项目根目录新增文件`.bowerrc`
```
{
    "directory": "vendor/bower"
}

```

上述配置告诉bower在`vendor/bower`存放下载文件
执行命令`bower init`创建文件`bower.json`


依赖安装所需前端资源
```
sudo bower install admin-lte
sudo bower install fontawesome
sudo bower install ionicons
```

打开文件`gulpfile.js`，编辑如下：

```
var gulp = require('gulp');
var elixir = require('laravel-elixir');

gulp.task('copy', function () {
    // jQuery
    gulp.src("vendor/bower/AdminLTE/plugins/jQuery/jQuery-2.1.4.min.js")
        .pipe(gulp.dest("resources/assets/js/"));

    // bootstarp
    gulp.src("vendor/bower/AdminLTE/bootstrap/css/bootstrap.min.css")
        .pipe(gulp.dest("resources/assets/css/"));
    gulp.src("vendor/bower/AdminLTE/bootstrap/js/bootstrap.min.js")
        .pipe(gulp.dest("resources/assets/js/"));

    // AdminLTE
    gulp.src("vendor/bower/AdminLTE/dist/css/AdminLTE.min.css")
        .pipe(gulp.dest("resources/assets/css/"));
    gulp.src("vendor/bower/AdminLTE/dist/css/skins/skin-blue.min.css")
        .pipe(gulp.dest("resources/assets/css/"));
    gulp.src("vendor/bower/AdminLTE/dist/js/app.min.js")
        .pipe(gulp.dest("resources/assets/js/"));
    gulp.src("vendor/bower/AdminLTE/dist/img/*")
        .pipe(gulp.dest("public/assets/img/"));

    // Fontawesome
    gulp.src("vendor/bower/font-awesome/css/font-awesome.min.css")
        .pipe(gulp.dest("resources/assets/css/"));
    gulp.src("vendor/bower/font-awesome/fonts/*")
        .pipe(gulp.dest("public/assets/fonts/"));

    // Ionicons
    gulp.src("vendor/bower/Ionicons/css/ionicons.min.css")
        .pipe(gulp.dest("resources/assets/css/"));
    gulp.src("vendor/bower/Ionicons/fonts/*")
        .pipe(gulp.dest("public/assets/fonts/"));

    // slimScroll
    gulp.src("vendor/bower/AdminLTE/plugins/slimScroll/jquery.slimscroll.min.js")
        .pipe(gulp.dest("resources/assets/js/"));

    // iCheck
    gulp.src("vendor/bower/AdminLTE/plugins/iCheck/icheck.min.js")
        .pipe(gulp.dest("resources/assets/js/"));
    gulp.src("vendor/bower/AdminLTE/plugins/iCheck/square/blue.css")
        .pipe(gulp.dest("resources/assets/css/"));
    gulp.src("vendor/bower/AdminLTE/plugins/iCheck/square/blue.png")
        .pipe(gulp.dest("public/assets/css/"));
    gulp.src("vendor/bower/AdminLTE/plugins/iCheck/square/blue@2x.png")
        .pipe(gulp.dest("public/assets/css/"));

    // select2
    gulp.src("vendor/bower/AdminLTE/plugins/select2/select2.full.min.js")
        .pipe(gulp.dest("resources/assets/js/"));
    gulp.src("vendor/bower/AdminLTE/plugins/select2/select2.min.js")
        .pipe(gulp.dest("resources/assets/js/"));
    gulp.src("vendor/bower/AdminLTE/plugins/select2/select2.min.css")
        .pipe(gulp.dest("resources/assets/css/"));

    // pace
    gulp.src("vendor/bower/AdminLTE/plugins/pace/pace.min.css")
        .pipe(gulp.dest("resources/assets/css/"));
    gulp.src("vendor/bower/AdminLTE/plugins/pace/pace.min.js")
        .pipe(gulp.dest("resources/assets/js/"));
});

elixir(function (mix) {
    // 合并javascript脚本
    mix.scripts(
        [
            'jQuery-2.1.4.min.js',
            'bootstrap.min.js',
            'app.min.js',
            'pace.min.js',
            'jquery.slimscroll.min.js',
            'icheck.min.js',
            'select2.full.min.js',
            'select2.min.js'
        ],
        'public/assets/js/app.js',
        'resources/assets/js/'
    );

    // 合并css脚本
    mix.styles(
        [
            'bootstrap.min.css',
            'pace.min.css',
            'select2.min.css',
            'AdminLTE.min.css',
            'skin-blue.min.css',
            'font-awesome.min.css',
            'ionicons.min.css',
            'blue.css'
        ],
        'public/assets/css/app.css',
        'resources/assets/css/'
    );

    // 运行单元测试
    mix.phpUnit();
});

```


在终端执行命令如下：
```
gulp copy
gulp
```

上述操作将会在根目录文件夹`public/assets/`生成所需的前端资源，接下来就是在模板中使用。
