---
title: 使用 Laravel 构建内容管理框架（五）
date: 2016-04-04 09:33:22
tags: Laravel
category: PHP
---

新增菜单管理模块功能，完成侧边栏菜单视图显示

# 新增迁移文件与数据填充

***

在终端执行以下命令，在文件夹`database/migrations/`新增迁移文件
```
php artisan make:migration --table=menus create_menus_table
```

在文件夹`database/migrations/`下打开迁移文件`create_menus_table.php`，修改代码如下：
```
<?php

use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateMenusTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('menus', function (Blueprint $table) {
            $table->engine = 'InnoDB';
            $table->increments('id');
            $table->smallInteger('parent_id')->default(0);
            $table->string('ico', 50);
            $table->string('url', 50);
            $table->string('name', 50);
            $table->string('description', 50);
            $table->tinyInteger('sort')->default(0);
            $table->tinyInteger('is_hide')->default(0);
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::drop('menus');
    }
}
```

打开文件`database/seeds/DatabaseSeeder.php`，修改代码如下：
```
<?php

use Illuminate\Database\Seeder;
use Illuminate\Database\Eloquent\Model;
use App\Models\User;
use App\Models\Menu;

class DatabaseSeeder extends Seeder
{
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        Model::unguard();
        $this->call(UserTableSeeder::class);
        $this->call(MenuTableSeeder::class);
        Model::reguard();
    }

}

class UserTableSeeder extends Seeder
{
    public function run()
    {
        DB::table('users')->delete();
        User::create(['name' => 'Ann', 'email' => 'ann@qq.com', 'password' => bcrypt(123456)]);
        User::create(['name' => 'Luis', 'email' => 'luis@qq.com', 'password' => bcrypt(123456)]);
        User::create(['name' => 'admin', 'email' => 'admin@qq.com', 'password' => bcrypt(123456)]);
    }
}

class MenuTableSeeder extends Seeder
{
    public function run()
    {
        DB::table('menus')->delete();
        Menu::create(["parent_id" => "0", "name" => "首页管理", "url" => "index.index", 'description' => '展示系统的各项基础数据']);
        Menu::create(["parent_id" => "0", "name" => "菜单管理", "url" => "menu.index", 'description' => '管理菜单的新增、编辑、删除']);
        Menu::create(["parent_id" => "2", "name" => "菜单列表", "url" => "menu.index", 'description' => '管理菜单的新增、编辑、删除']);
        Menu::create(["parent_id" => "2", "name" => "新增菜单", "url" => "menu.create", 'description' => '新增菜单的页面']);
        Menu::create(["parent_id" => "2", "name" => "编辑菜单", "url" => "menu.edit", 'description' => '编辑菜单的页面', 'is_hide' => 1]);
        Menu::create(["parent_id" => "0", "name" => "角色管理", "url" => "role.index", 'description' => '管理角色的新增、编辑、删除']);
        Menu::create(["parent_id" => "6", "name" => "角色列表", "url" => "role.index", 'description' => '管理角色的新增、编辑、删除']);
        Menu::create(["parent_id" => "6", "name" => "新增角色", "url" => "role.create", 'description' => '新增角色的页面']);
        Menu::create(["parent_id" => "6", "name" => "编辑角色", "url" => "role.edit", 'description' => '编辑角色的页面', 'is_hide' => 1]);
        Menu::create(["parent_id" => "6", "name" => "角色赋权", "url" => "role.show", 'description' => '编辑角色的页面', 'is_hide' => 1]);
        Menu::create(["parent_id" => "0", "name" => "权限管理", "url" => "permission.index", 'description' => '管理权限的新增、编辑、删除']);
        Menu::create(["parent_id" => "11", "name" => "权限列表", "url" => "permission.index", 'description' => '管理权限的新增、编辑、删除']);
        Menu::create(["parent_id" => "11", "name" => "新增权限", "url" => "permission.create", 'description' => '新增权限的页面']);
        Menu::create(["parent_id" => "11", "name" => "编辑权限", "url" => "permission.edit", 'description' => '编辑权限的页面', 'is_hide' => 1]);
        Menu::create(["parent_id" => "0", "name" => "用户管理", "url" => "user.index", 'description' => '管理用户的新增、编辑、删除']);
        Menu::create(["parent_id" => "15", "name" => "用户列表", "url" => "user.index", 'description' => '管理用户的新增、编辑、删除']);
        Menu::create(["parent_id" => "15", "name" => "新增用户", "url" => "user.create", 'description' => '新增用户的页面']);
        Menu::create(["parent_id" => "15", "name" => "编辑用户", "url" => "user.edit", 'description' => '编辑用户的页面', 'is_hide' => 1]);
    }
}
```

在终端运行命名如下：
```
// 生成数据表
php artisan migrate

// 填充数据表
php artisan db:seed
```

进行上述操作之后，会在数据库生成menus数据表并往menus数据表里面填充数据

# 新增模型

***

在终端执行以下命令，在文件夹`app/Models/`新建文件`Menu.php`、`Tree.php`
```
php artisan make:model Models/Menu
php artisan make:model Models/Tree
```

打开文件`Menu.php`，修改代码如下：
```
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Menu extends Model
{
    protected $guarded = [];


    /**
     * 获取侧边栏菜单
     * @return string
     */
    public static function getSidebar()
    {
        $tree = Tree::createNodeTree(Menu::all());
        $sidebar = self::setSidebar($tree);

        return $sidebar;
    }


    /**
     * 设置侧边栏菜单
     * @param $tree
     *
     * @return string
     */
    public static function setSidebar($tree)
    {
        $html = "";
        foreach ($tree as $menu) {
            if ($menu->is_hide == 0) {
                if ($menu->child) {
                    $html .= '<li class="treeview">'
                        . '<a><i class="fa fa-bars"></i> <span>' . $menu->name . '</span><i class="fa fa-angle-left pull-right"></i></a>'
                        . '<ul class="treeview-menu">'
                        . self::setSidebar($menu->child)
                        . '</ul>'
                        . '</li>';
                } else {
                    $html .= '<li><a href="' . route($menu->url) . '"><i class="fa fa-bars"></i><span> ' . $menu->name . '</span></a></li>';
                }
            }
        }

        return $html;
    }
}
```

打开文件`Tree.php`，修改代码如下：
```
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

/**
 * Class Tree
 * 树的创建与操作
 *
 * @package App\Models
 */
class Tree extends Model
{
    /**
     * 生成无序层级树
     *
     * @param        $models
     * @param int    $parent_id
     * @param int    $level
     * @param string $html
     *
     * @return array
     */
    public static function createLevelTree($models, $parent_id = 0, $level = 0, $html = "-")
    {
        $tree = [];
        foreach ($models as $model) {
            if ($model->parent_id == $parent_id) {

                if ($level != 0) {
                    $model->html = str_repeat("    ", $level);
                    $model->html .= '|';
                }
                $model->html .= str_repeat($html, $level);
                $tree[] = $model;
                $tree = array_merge($tree, self::createLevelTree($models, $model->id, $level + 1));
            }
        }

        return $tree;
    }

    /**
     * 生成无序节点树
     *
     * @param        $models
     * @param int    $parent_id
     * @param string $node
     *
     * @return array
     */
    public static function createNodeTree($models, $parent_id = 0, $node = 'child')
    {
        $tree = [];

        foreach ($models as $model) {
            if ($model->parent_id == $parent_id) {
                $model->$node = self::createNodeTree($models, $model->id);
                $tree[] = $model;
            }
        }

        return $tree;
    }
}
```


# 新增验证请求

***

在终端执行以下命令，在文件夹`app/Http/Requests/Form`下新增表单验证`MenuForm`
```
php artisan make:request Form/MenuForm
```

修改文件`MenuForm`代码如下：
```
<?php

namespace App\Http\Requests\Form;

use App\Http\Requests\Request;

class MenuForm extends Request
{
    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        return true;
    }

    /**
     * Get the validation rules that apply to the request.
     *
     * @return array
     */
    public function rules()
    {
        return [
            'url'       => 'required',
            'name'      => 'required',
            'parent_id' => 'required'
        ];
    }

    public function messages()
    {
        return [
            'url.required'       => '菜单地址不能为空',
            'name.required'      => '菜单名称不能为空',
            'parent_id.required' => '父级分类不能为空'
        ];
    }
}
```

# 新增控制器MenuController

***

在终端执行以下命令，在文件夹`app/Http/Controllers/Backend`新增文件`MenuController.php`
```
php artisan make:controller Backend/MenuController
```

修改文件`MenuController.php`代码如下：
```
<?php

namespace App\Http\Controllers\Backend;

use App\Models\Menu;
use App\Models\Tree;
use App\Http\Requests;
use App\Http\Controllers\Controller;
use App\Http\Requests\Form\MenuForm;

class MenuController extends Controller
{
    /**
     * Display a listing of the resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function index()
    {
        $menus = Menu::paginate(25);

        return view('backend.menu.index', compact('menus'));
    }

    /**
     * Show the form for creating a new resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function create()
    {
        $tree = Tree::createLevelTree(Menu::all());

        return view('backend.menu.create', compact('tree'));
    }

    /**
     * Store a newly created resource in storage.
     *
     * @param  \Illuminate\Http\Request $request
     *
     * @return \Illuminate\Http\Response
     */
    public function store(MenuForm $request)
    {
        try {
            if (Menu::create($request->all())) {
                return redirect()->back()->withSuccess("新增菜单成功");
            }
        } catch (\Exception $e) {
            return redirect()->back()->withErrors(array('error' => $e->getMessage()))->withInput();
        }
    }

    /**
     * Display the specified resource.
     *
     * @param  int $id
     *
     * @return \Illuminate\Http\Response
     */
    public function show($id)
    {
        //
    }

    /**
     * Show the form for editing the specified resource.
     *
     * @param  int $id
     *
     * @return \Illuminate\Http\Response
     */
    public function edit($id)
    {
        $menu = Menu::find($id);
        $tree = Tree::createLevelTree(Menu::all());

        return view('backend.menu.edit', compact('menu', 'tree'));
    }

    /**
     * Update the specified resource in storage.
     *
     * @param  \Illuminate\Http\Request $request
     * @param  int                      $id
     *
     * @return \Illuminate\Http\Response
     */
    public function update(MenuForm $request, $id)
    {
        $data = $request->all();
        unset($data['_method']);
        unset($data['_token']);

        try {
            if (Menu::where('id', $id)->update($data)) {
                return redirect()->back()->withSuccess('编辑菜单成功');
            }
        } catch (\Exception $e) {
            return redirect()->back()->withErrors(array('error' => $e->getMessage()))->withInput();
        }
    }

    /**
     * Remove the specified resource from storage.
     *
     * @param  int $id
     *
     * @return \Illuminate\Http\Response
     */
    public function destroy($id)
    {
        $childMenus = Menu::where('parent_id', '=', $id)->get()->toArray();

        if ( ! empty($childMenus)) {
            return redirect()->back()->withErrors("请先删除其下级分类");
        }

        try {
            if (Menu::destroy($id)) {
                return redirect()->back()->withSuccess('删除菜单成功');
            }
        } catch (\Exception $e) {
            return redirect()->back()->withErrors(array('error' => $e->getMessage()));
        }
    }
}

```

# 新增路由

***

在文件`app\Http\routes.php`修改代码如下：
```
Route::group(['namespace' => 'Backend', 'middleware' => ['auth']], function () {
    Route::get('/', ['as' => 'index.index', 'uses' => 'IndexController@index']);
    Route::resource('user', 'UserController');
    Route::resource('menu', 'MenuController');
});

Route::group(['namespace' => 'Auth'], function () {
    Route::get('auth/login', 'AuthController@getLogin');
    Route::post('auth/login', 'AuthController@postLogin');
    Route::get('auth/logout', 'AuthController@getLogout');

});
```

# 新增视图

***

在文件夹`resources/views/backend/`新建文件夹`menu`，并新建以下模板文件：

- edit.blade.php
- index.blade.php
- create.blade.php

### index.blade.php
```
@extends('backend.layout.main')

@section('content')
    <div class="row">
        <div class="col-xs-1">
            <div class="small-box">
                <a href="{{URL::to('menu/create')}}" class="btn btn-success">新增菜单</a>
            </div>
        </div>
    </div>
    <div class="row">
        <div class="col-md-12">
            <div class="box box-primary">

                <div class="box-header with-border">
                    <h3 class="box-title">菜单列表</h3>

                    <div class="box-tools pull-right">
                        <div class="input-group input-group-sm" style="width: 150px;">
                            <input type="text" name="table_search" class="form-control pull-right" placeholder="快速查询">
                            <div class="input-group-btn">
                                <button type="button" class="btn btn-default disabled">
                                    <i class="fa fa-search"></i>
                                </button>
                            </div>
                        </div>
                    </div>
                </div>

                <div class="box-body table-responsive no-padding">
                    <table class="table table-hover">
                        <tr>
                            <th>菜单编号</th>
                            <th>菜单名称</th>
                            <th>菜单地址</th>
                            <th>操作</th>
                        </tr>
                        @forelse($menus as $menu)
                            <tr>
                                <td>{{$menu->id}}</td>
                                <td>{{$menu->name}}</td>
                                <td>{{$menu->url}}</td>
                                <td>
                                    <a class="btn btn-info" href="{{URL::to('menu/'.$menu->id.'/edit')}}">
                                        编辑
                                    </a>
                                    <button class="btn btn-danger" data-toggle="modal" data-target="#defalutModal" data-url="{{URL::to('menu/'.$menu->id)}}">
                                        删除
                                    </button>
                                </td>
                            </tr>
                        @empty
                            <tr>
                                <td colspan="4" class="text-center">暂无数据</td>
                            </tr>
                        @endforelse
                    </table>
                </div>

                @if($menus->render() !== "")
                    <div class="box-footer">
                        {!! $menus->render() !!}
                    </div>
                @endif
            </div>
        </div>
    </div>
    @include('backend.layout.model.default',['model_title'=>'操作提示','model_content'=>'你确定要删除这条菜单吗?'])
@stop
@section('script')
    <script type="text/javascript">
        $('#defalutModal').on('show.bs.modal', function (event) {
            var button = $(event.relatedTarget);
            var url = button.data('url');
            var modal = $(this);

            modal.find('form').attr('action', url);
        })
    </script>
@stop
```

### create.blade.php
```
@extends('backend.layout.main')
@section('content')
    <div class="row">
        <div class="col-md-6">
            <div class="box box-primary">
                <form class="form-horizontal" action="{{URL::to('menu')}}" method="post" enctype="multipart/form-data">
                    <div class="box-header with-border">
                        <h3 class="box-title">{{$page_title or "Page Title"}}</h3>
                        <input type="hidden" name="_token" value="{{csrf_token()}}">
                    </div>
                    <div class="box-body">
                        <div class="form-group">
                            <label class="col-sm-3 control-label">父级菜单</label>
                            <div class="col-sm-9">
                                <select class="form-control select2" name="parent_id">
                                    <option value="0">/</option>
                                    @foreach($tree as $menu)
                                    <option value="{{$menu->id}}" @if(old('parent_id') == $menu->id) selected @endif >{{$menu->html}}{{$menu->name}}</option>
                                    @endforeach
                                </select>
                                @include('backend.layout.message.tips',['field'=>'parent_id'])
                            </div>
                        </div>
                        <div class="form-group">
                            <label for="name" class="col-sm-3 control-label">菜单名称</label>
                            <div class="col-sm-9">
                                <input type="text" class="form-control" id="name" name="name" placeholder="菜单名称" value="{{old('name')}}">
                                @include('backend.layout.message.tips',['field'=>'name'])
                            </div>
                        </div>
                        <div class="form-group">
                            <label for="url" class="col-sm-3 control-label">菜单地址</label>
                            <div class="col-sm-9">
                                <input type="text" class="form-control" id="url" name="url" placeholder="Controller.method" value="{{old('url')}}">
                                @include('backend.layout.message.tips',['field'=>'url'])
                            </div>
                        </div>
                    </div>
                    <div class="box-footer">
                        <button type="button" class="btn btn-default" onclick="javascript:history.back(-1);return false;">
                            返回
                        </button>
                        <button type="submit" class="btn btn-danger pull-right">确 定</button>
                    </div>
                </form>
            </div>
        </div>
    </div>
@stop
```

### edit.blade.php
```
@extends('backend.layout.main')
@section('content')
    <div class="row">
        <div class="col-md-6">
            <div class="box box-info">
                <form class="form-horizontal" action="{{URL::to('menu/'.$menu->id)}}" method="post" enctype="multipart/form-data">
                    <div class="box-header with-border">
                        <h3 class="box-title">编辑菜单</h3>
                        <input type="hidden" name="_token" value="{{csrf_token()}}">
                        <input type="hidden" name="_method" value="put">
                    </div>
                    <div class="box-body">
                        <div class="form-group">
                            <label class="col-sm-3 control-label">父级菜单</label>
                            <div class="col-sm-9">
                                <select class="form-control select2" name="parent_id">
                                    <option value="0">/</option>
                                    @foreach($tree as $data)
                                        <option value="{{$data->id}}" @if($menu->parent_id == $data->id) selected @endif >{{$data->html}}{{$data->name}}</option>
                                    @endforeach
                                </select>
                                @include('backend.layout.message.tips',['field'=>'parent_id'])
                            </div>
                        </div>
                        <div class="form-group">
                            <label for="name" class="col-sm-3 control-label">菜单名称</label>
                            <div class="col-sm-9">
                                <input type="text" class="form-control" id="name" name="name" placeholder="菜单名称" value="{{$menu->name}}">
                                @include('backend.layout.message.tips',['field'=>'name'])
                            </div>
                        </div>
                        <div class="form-group">
                            <label for="url" class="col-sm-3 control-label">菜单地址</label>
                            <div class="col-sm-9">
                                <input type="text" class="form-control" id="url" name="url" placeholder="Controller/method" value="{{$menu->url}}">
                                @include('backend.layout.message.tips',['field'=>'url'])
                            </div>
                        </div>
                    </div>
                    <div class="box-footer">
                        <button type="button" class="btn btn-default" onclick="javascript:history.back(-1);return false;">
                            返回
                        </button>
                        <button type="submit" class="btn btn-danger pull-right">确 定</button>
                    </div>
                </form>
            </div>
        </div>
    </div>
@stop
@section('script')
    <script type="text/javascript">
        $(function(){
            $(".select2").select2();
        });
    </script>
@stop

```

# 视图共享数据

***

打开文件`resources/views/backend/layout/sidebar.blade.php`，修改代码如下：
```
<aside class="main-sidebar">
    <section class="sidebar">
        <div class="user-panel">
            <div class="pull-left image">
                <img src="{{ asset("/assets/img/user2-160x160.jpg") }}" class="img-circle" alt="User Image"/>
            </div>
            <div class="pull-left info">
                <p>Alexander Pierce</p>
                <a href="#"><i class="fa fa-circle text-success"></i> Online</a>
            </div>
        </div>
        <form action="#" method="get" class="sidebar-form">
            <div class="input-group">
                <input type="text" name="q" class="form-control" placeholder="Search..."/>
                <span class="input-group-btn">
                  <button type='submit' name='search' id='search-btn' class="btn btn-flat">
                      <i class="fa fa-search"></i>
                  </button>
                </span>
            </div>
        </form>
        <ul class="sidebar-menu">
            @inject('menu','App\Models\Menu')
            {!! $menu::getSidebar() !!}
        </ul>
    </section>
</aside>
```

完成上述操作后，便可以在每个视图都能看到共享的侧边栏菜单数据


