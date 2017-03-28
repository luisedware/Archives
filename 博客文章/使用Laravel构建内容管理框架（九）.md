---
title: 使用 Laravel 构建内容管理框架（九）
date: 2016-04-04 21:33:49
tags: Laravel
category: PHP
---

利用 `zizaco/entrust` Package 进行权限管理

# 修改用户表单请求

***

打开文件`app/Http/Request/Form/UserForm.php`，修改代码如下：
```
<?php

namespace App\Http\Requests\Form;

use App\Http\Requests\Request;

class UserForm extends Request
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
            'name'                  => 'required|unique:users',
            'email'                 => 'required|unique:users',
            'role_id'               => 'required',
            'password'              => 'required|confirmed',
            'password_confirmation' => 'required',
        ];
    }

    public function messages()
    {
        return [
            'name.required'                  => '用户名称不能为空',
            'name.unique'                    => '用户名称已存在',
            'email.required'                 => '用户邮箱不能为空',
            'email.unique'                   => '用户邮箱已存在',
            'role_id.required'               => '用户角色不能为空',
            'password.required'              => '用户密码不能为空',
            'password.confirmed'             => '确认密码不一致',
            'password_confirmation.required' => '确认密码不能为空'
        ];
    }
}
```
修改用户表单请求的验证规则，确保用户名称、用户邮箱唯一。

# 修改用户管理控制器

***

打开文件`app/Http/Controllers/Backend/UserController/php`，修改代码如下：
```
<?php

namespace App\Http\Controllers\Backend;

use App\Models\Role;
use App\Models\User;
use App\Http\Requests;
use App\Http\Controllers\Controller;
use App\Http\Requests\Form\UserForm;

class UserController extends Controller
{
    /**
     * Display a listing of the resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function index()
    {
        $users = User::paginate(25);

        return view('backend.user.index', compact('users'));
    }

    /**
     * Show the form for creating a new resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function create()
    {
        $roles = Role::all();

        return view('backend.user.create', compact('roles'));
    }

    /**
     * Store a newly created resource in storage.
     *
     * @param  \Illuminate\Http\Request $request
     *
     * @return \Illuminate\Http\Response
     */
    public function store(UserForm $request)
    {
        $data = [
            'name'     => $request['name'],
            'email'    => $request['email'],
            'password' => bcrypt($request['password']),
        ];

        try {
            $roles = Role::whereIn('id', $request->get('role_id'))->get();
            if (empty($roles->toArray())) {

                return redirect()->back()->withErrors("用户角色不存在,请刷新页面并选择其他用户角色")->withInput();
            }

            $user = User::create($data);
            if ($user) {

                foreach ($roles as $role) {
                    $user->attachRole($role);
                }

                return redirect()->route('user.index')->withSuccess('新增用户成功');
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
        $user = User::find($id);
        $roles = Role::all();
        $userRoles = $user->roles->toArray();
        $displayNames = array_map(function ($value) {
            return $value['display_name'];
        }, $userRoles);

        return view('backend.user.edit', compact('user', 'roles', 'displayNames'));
    }

    /**
     * Update the specified resource in storage.
     *
     * @param  \Illuminate\Http\Request $request
     * @param  int                      $id
     *
     * @return \Illuminate\Http\Response
     */
    public function update(UserForm $request, $id)
    {
        $user = User::find($id);
        $user->name = $request['name'];
        $user->email = $request['email'];
        $user->password = bcrypt($request['password']);

        try {
            $roles = Role::whereIn('id', $request->get('role_id'))->get();
            if (empty($roles->toArray())) {

                return redirect()->back()->withErrors("用户角色不存在,请刷新页面并选择其他用户角色")->withInput();
            } else {
                if ($user->save()) {
                    foreach ($roles as $role) {
                        $user->attachRole($role);
                    }

                    return redirect()->route('user.index')->withSuccess('编辑用户成功');
                }
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
        try {
            if (User::destroy($id)) {
                return redirect()->back()->withSuccess('删除用户成功');
            }
        } catch (\Exception $e) {
            return redirect()->back()->withErrors(array('error' => $e->getMessage()));
        }
    }
}
```
修改新增用户、编辑用户的业务流程，新增用户、编辑用户的时候，必须为用户指定一名角色。

# 修改视图

***

打开文件夹`resources/views/backend/user/`下的

- `index.blade.php`
- `create.blade.php`
- `edit.blade.php`

修改代码如下：

### index.blade.php
```
@extends('backend.layout.main')

@section('content')
    <div class="row">
        <div class="col-xs-1">
            <div class="small-box">
                <a href="{{URL::to('user/create')}}" class="btn btn-success">新增用户</a>
            </div>
        </div>
    </div>
    <div class="row">
        <div class="col-md-12">
            <div class="box">

                <div class="box-header with-border">
                    <h3 class="box-title">用户列表</h3>

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
                            <th>用户编号</th>
                            <th>用户名称</th>
                            <th>用户邮箱</th>
                            <th>所属角色</th>
                            <th>管理操作</th>
                        </tr>
                        @forelse($users as $user)
                            <tr>
                                <td>{{$user->id}}</td>
                                <td>{{$user->name}}</td>
                                <td>{{$user->email}}</td>
                                <td>
                                    @foreach($user->roles as $role)
                                        {{$role->display_name}}
                                    @endforeach
                                </td>
                                <td>
                                    <a class="btn btn-info" href="{{URL::to('user/'.$user->id.'/edit')}}">
                                        编辑
                                    </a>
                                    <button class="btn btn-danger" data-toggle="modal" data-target="#defalutModal" data-url="{{URL::to('user/'.$user->id)}}">
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

                @if($users->render() !== "")
                    <div class="box-footer">
                        {!! $users->render() !!}
                    </div>
                @endif
            </div>
        </div>
    </div>
    @include('backend.layout.model.default',['model_title'=>'操作提示','model_content'=>'你确定要删除这名用户吗?'])
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
            <div class="box box-info">
                <form class="form-horizontal" action="{{URL::to('user')}}" method="post" enctype="multipart/form-data">
                    <div class="box-header with-border">
                        <h3 class="box-title">{{$page_title or "page_title"}}</h3>
                        <input type="hidden" name="_token" value="{{csrf_token()}}">
                    </div>
                    <div class="box-body">
                        <div class="form-group">
                            <label class="col-sm-3 control-label">用户角色</label>
                            <div class="col-sm-9">
                                <select class="form-control select2" multiple="multiple" name="role_id[]">
                                    @foreach($roles as $role)
                                        <option value="{{$role->id}}">{{$role->display_name}}</option>
                                    @endforeach
                                </select>
                                @include('backend.layout.message.tips',['field'=>'role_id'])
                            </div>
                        </div>
                        <div class="form-group">
                            <label for="name" class="col-sm-3 control-label">用户名称</label>
                            <div class="col-sm-9">
                                <input type="text" class="form-control" id="name" name="name" placeholder="用户名称" value="{{old('name')}}">
                                @include('backend.layout.message.tips',['field'=>'name'])
                            </div>
                        </div>
                        <div class="form-group">
                            <label for="email" class="col-sm-3 control-label">用户邮箱</label>
                            <div class="col-sm-9">
                                <input type="text" class="form-control" id="email" name="email" placeholder="用户邮箱" value="{{old('email')}}">
                                @include('backend.layout.message.tips',['field'=>'email'])
                            </div>
                        </div>
                        <div class="form-group">
                            <label for="password" class="col-sm-3 control-label">用户密码</label>
                            <div class="col-sm-9">
                                <input type="text" class="form-control" id="password" name="password" placeholder="用户密码" value="{{old('password')}}">
                                @include('backend.layout.message.tips',['field'=>'password'])
                            </div>
                        </div>
                        <div class="form-group">
                            <label for="password_confirmation" class="col-sm-3 control-label">确认密码</label>
                            <div class="col-sm-9">
                                <input type="text" class="form-control" id="password_confirmation" name="password_confirmation" placeholder="确认密码" value="{{old('password_confirmation')}}">
                                @include('backend.layout.message.tips',['field'=>'password_confirmation'])
                            </div>
                        </div>
                    </div>
                    <div class="box-footer">
                        <a class="btn btn-default" href="{{route('user.index')}}">返回</a>
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
                <form class="form-horizontal" action="{{URL::to('user/'.$user->id)}}" method="post" enctype="multipart/form-data">
                    <div class="box-header with-border">
                        <h3 class="box-title">{{$page_title or "Page_title"}}</h3>
                        <input type="hidden" name="_token" value="{{csrf_token()}}">
                        <input type="hidden" name="_method" value="put">
                    </div>
                    <div class="box-body">
                        <div class="form-group">
                            <label class="col-sm-3 control-label">用户角色</label>
                            <div class="col-sm-9">
                                <select class="form-control select2" multiple="multiple" name="role_id[]" style="width: 100%">
                                    @foreach($roles as $role)
                                        <option value="{{$role->id}}" @if(in_array($role->display_name,$displayNames)) selected @endif>{{$role->display_name}}</option>
                                    @endforeach
                                </select>
                                @include('backend.layout.message.tips',['field'=>'role_id'])
                            </div>
                        </div>
                        <div class="form-group">
                            <label for="name" class="col-sm-3 control-label">用户名称</label>
                            <div class="col-sm-9">
                                <input type="text" class="form-control" id="name" name="name" placeholder="用户名称" value="{{$user->name}}">
                                @include('backend.layout.message.tips',['field'=>'name'])
                            </div>
                        </div>
                        <div class="form-group">
                            <label for="email" class="col-sm-3 control-label">用户邮箱</label>
                            <div class="col-sm-9">
                                <input type="text" class="form-control" id="email" name="email" placeholder="用户邮箱" value="{{$user->email}}">
                                @include('backend.layout.message.tips',['field'=>'email'])
                            </div>
                        </div>
                        <div class="form-group">
                            <label for="password" class="col-sm-3 control-label">用户密码</label>
                            <div class="col-sm-9">
                                <input type="text" class="form-control" id="password" name="password" placeholder="用户密码">
                                @include('backend.layout.message.tips',['field'=>'password'])
                            </div>
                        </div>
                        <div class="form-group">
                            <label for="password_confirmation" class="col-sm-3 control-label">确认密码</label>
                            <div class="col-sm-9">
                                <input type="text" class="form-control" id="password_confirmation" name="password_confirmation" placeholder="确认密码" value="{{$user->password_confirmation}}">
                                @include('backend.layout.message.tips',['field'=>'password_confirmation'])
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

# 新增模型

***

在终端运行以下命令，新增数据模型

```
php artisan make:model Models/RoleUser
php artisan make:model Models/PermissionRole
```

分别打开文件`RoleUser.php`，`PermissionRole.php`，修改代码如下：

### RoleUser.php
```
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class RoleUser extends Model
{
    protected $fillable = ['user_id', 'role_id'];

    protected $table = "role_user";

    public $timestamps = false;
}
```

### PermissionRole.php
```
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class PermissionRole extends Model
{
    protected $fillable = ['permission_id', 'role_id'];

    protected $table = "permission_role";

    public $timestamps = false;
}
```

# 新增数据填充

***

打开文件`database/seeds/DatabaseSeeder.php`，修改文件代码如下：
```
<?php

use Illuminate\Database\Seeder;
use App\Models\Menu;
use App\Models\Role;
use App\Models\User;
use App\Models\RoleUser;
use App\Models\Permission;
use App\Models\PermissionRole;

class DatabaseSeeder extends Seeder
{
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        $this->call("MenusTableSeeder");
        $this->call("UsersTableSeeder");
        $this->call("RolesTableSeeder");
        $this->call("RoleUserTableSeeder");
        $this->call("PermissionTableSeeder");
        $this->call("PermissionRoleTableSeeder");
    }
}

class PermissionRoleTableSeeder extends Seeder
{
    public function run()
    {
        DB::table('permission_role')->delete();
        for ($i = 1; $i < 3; $i++) {
            for ($j = 1; $j < 15; $j++) {
                PermissionRole::create(['permission_id' => $j, 'role_id' => $i]);
            }
        }

    }
}


class UsersTableSeeder extends Seeder
{
    public function run()
    {
        DB::table('users')->delete();
        User::create(['name' => 'Ann', 'email' => 'ann@qq.com', 'password' => bcrypt(123456)]);
        User::create(['name' => 'Luis', 'email' => 'luis@qq.com', 'password' => bcrypt(123456)]);
        User::create(['name' => 'admin', 'email' => 'admin@qq.com', 'password' => bcrypt(123456)]);
    }
}


class RolesTableSeeder extends Seeder
{
    public function run()
    {
        DB::table('roles')->delete();
        Role::create(['name' => 'admin', 'display_name' => 'User Administrator', 'description' => 'User is allowed to manage and edit other users']);
        Role::create(['name' => 'owner', 'display_name' => 'Project Owner', 'description' => 'User is the owner of a given project']);
    }
}

class RoleUserTableSeeder extends Seeder
{
    public function run()
    {
        DB::table('role_user')->delete();
        RoleUser::create(['user_id' => 3, 'role_id' => 1]);
        RoleUser::create(['user_id' => 2, 'role_id' => 2]);
        RoleUser::create(['user_id' => 1, 'role_id' => 2]);
    }
}

class PermissionTableSeeder extends Seeder
{
    public function run()
    {
        DB::table('permissions')->delete();
        Permission::create(["display_name" => "首页管理", "name" => "index.index", 'description' => '展示系统的各项基础数据']);
        Permission::create(["display_name" => "菜单列表", "name" => "menu.index", 'description' => '管理菜单的新增、编辑、删除']);
        Permission::create(["display_name" => "新增菜单", "name" => "menu.create", 'description' => '新增菜单的页面']);
        Permission::create(["display_name" => "编辑菜单", "name" => "menu.edit", 'description' => '编辑菜单的页面']);
        Permission::create(["display_name" => "角色列表", "name" => "role.index", 'description' => '管理角色的新增、编辑、删除']);
        Permission::create(["display_name" => "新增角色", "name" => "role.create", 'description' => '新增角色的页面']);
        Permission::create(["display_name" => "编辑角色", "name" => "role.edit", 'description' => '编辑角色的页面']);
        Permission::create(["display_name" => "角色赋权", "name" => "role.show", 'description' => '编辑角色的页面']);
        Permission::create(["display_name" => "权限列表", "name" => "permission.index", 'description' => '管理权限的新增、编辑、删除']);
        Permission::create(["display_name" => "新增权限", "name" => "permission.create", 'description' => '新增权限的页面']);
        Permission::create(["display_name" => "编辑权限", "name" => "permission.edit", 'description' => '编辑权限的页面']);
        Permission::create(["display_name" => "用户列表", "name" => "user.index", 'description' => '管理用户的新增、编辑、删除']);
        Permission::create(["display_name" => "新增用户", "name" => "user.create", 'description' => '新增用户的页面']);
        Permission::create(["display_name" => "编辑用户", "name" => "user.edit", 'description' => '编辑用户的页面']);
    }
}

class MenusTableSeeder extends Seeder
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

接着在终端执行以下命令，执行数据回滚与填充
```
php artisan migrate:refresh --seed
```

# 新建中间件

***

在终端执行以下命令新增一个中间件
```
php artisan make:middleware Entrust
```

打开文件`app/Http/Middleware/Entrust`，修改文件代码如下：
```
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Route;

class Entrust
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request $request
     * @param  \Closure                 $next
     *
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        if ( ! Auth::user()->hasPermission(Route::currentRouteName())) {
            return redirect()->back()->withErrors("没有操作权限");
        }

        return $next($request);
    }
}
```

# 注册中间件

***

打开文件`app/Http/Kernel.php`，在数组`$routeMiddleware`添加以下代码：
```
'Entrust'    => \App\Http\Middleware\Entrust::class
```

# 路由绑定中间件

***

打开文件`app/Http/routes.php`，修改文件代码如下：
```
Route::group(['namespace' => 'Backend', 'middleware' => ['auth','Entrust']], function () {
    Route::get('/', ['as' => 'index.index', 'uses' => 'IndexController@index']);
    Route::resource('user', 'UserController');
    Route::resource('menu', 'MenuController');
    Route::resource('role', 'RoleController');
    Route::resource('permission', 'PermissionController');
});

Route::group(['namespace' => 'Auth'], function () {

    Route::get('auth/login', 'AuthController@getLogin');
    Route::post('auth/login', 'AuthController@postLogin');
    Route::get('auth/logout', 'AuthController@getLogout');

});
```

凡是进行后台访问、操作的路由，都必须经过`Entrust`中间件进行权限验证。当前登录用户对应的角色没有权限，则无法查看页面或进行数据操作

