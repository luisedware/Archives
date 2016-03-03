![Happy Coding](http://upload-images.jianshu.io/upload_images/1277229-a09205aecddd5bf8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
# 本文目标

***

新增权限管理模块，管理权限的增删查改。

# 新增请求

***

```
php artisan make:request Form/PermissionForm
```

文件`PermissionForm`代码如下：
```
<?php

namespace App\Http\Requests\Form;

use App\Http\Requests\Request;

class PermissionForm extends Request
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
            'name'         => 'required',
            'display_name' => 'required',
            'description'  => 'required'
        ];
    }

    public function messages()
    {
        return [
            'name.required'         => '权限标识不能为空',
            'display_name.required' => '权限名称不能为空',
            'description.required'  => '权限描述不能为空'
        ];
    }
}

```

# 新增控制器

***

```
php artisan make:controller Backend/PermissionController
```

文件`PermissionController.php`代码如下：
```
<?php

namespace App\Http\Controllers\Backend;

use App\Models\Permission;
use App\Http\Requests;
use App\Http\Controllers\Controller;
use App\Http\Requests\Form\PermissionForm;

class PermissionController extends Controller
{
    /**
     * Display a listing of the resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function index()
    {
        $permissions = Permission::paginate(25);

        return view('backend.permission.index', compact('permissions'));
    }

    /**
     * Show the form for creating a new resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function create()
    {
        return view('backend.permission.create');
    }

    /**
     * Store a newly created resource in storage.
     *
     * @param  \Illuminate\Http\Request $request
     *
     * @return \Illuminate\Http\Response
     */
    public function store(PermissionForm $request)
    {
        try {
            if (Permission::create($request->all())) {
                return redirect()->route('permission.index')->withSuccess('新增权限成功');
            }
        } catch (\Exception $e) {
            return redirect()->back()->withErrors(['error' => $e->getMessage()])->withInput();
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
        $permission = Permission::find($id);

        return view('backend.permission.edit', compact('permission'));
    }

    /**
     * Update the specified resource in storage.
     *
     * @param  \Illuminate\Http\Request $request
     * @param  int                      $id
     *
     * @return \Illuminate\Http\Response
     */
    public function update(PermissionForm $request, $id)
    {
        $data = $request->all();

        unset($data['_token']);
        unset($data['_method']);

        try {
            if (Permission::where('id', '=', $id)->update($data)) {
                return redirect()->back()->withSuccess("新增权限成功");
            }
        } catch (\Exception $e) {
            return redirect()->back()->withErrors(['error' => $e->getMessage()])->withInput();
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
            if (Permission::destroy($id)) {
                return redirect()->back()->withSuccess("删除菜单成功");
            }
        } catch (\Exception $e) {
            return redirect()->back()->withErrors(['error' => $e->getMessage()])->withInput();
        }
    }
}

```

# 新增视图

***

在文件夹`resources/views/permission/`下新增文件

- index.blade.php
- create.blade.php
- edit.blade.php

## index.blade.php
```
@extends('backend.layout.main')

@section('content')
    <div class="row">
        <div class="col-xs-1">
            <div class="small-box">
                <a href="{{URL::to('permission/create')}}" class="btn btn-success">新增权限</a>
            </div>
        </div>
    </div>
    <div class="row">
        <div class="col-md-12">
            <div class="box">

                <div class="box-header with-border">
                    <h3 class="box-title">权限列表</h3>

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
                            <th>权限编号</th>
                            <th>权限标识</th>
                            <th>权限名称</th>
                            <th>权限描述</th>
                            <th>管理操作</th>
                        </tr>
                        @forelse($permissions as $permission)
                            <tr>
                                <td>{{$permission->id}}</td>
                                <td>{{$permission->name}}</td>
                                <td>{{$permission->display_name}}</td>
                                <td>{{$permission->description}}</td>
                                <td>
                                    <a class="btn btn-info" href="{{URL::to('permission/'.$permission->id.'/edit')}}">
                                        编辑
                                    </a>
                                    <a class="btn btn-danger" data-toggle="modal" data-target="#defalutModal" data-url="{{URL::to('permission/'.$permission->id)}}">
                                        删除
                                    </a>
                                </td>
                            </tr>
                        @empty
                            <tr>
                                <td colspan="5" class="text-center">暂无数据</td>
                            </tr>
                        @endforelse
                    </table>
                </div>

                @if($permissions->render() !== "")
                    <div class="box-footer">
                        {!! $permissions->render() !!}
                    </div>
                @endif
            </div>
        </div>
    </div>
    @include('backend.layout.model.default',['model_title'=>'操作提示','model_content'=>'你确定要删除这条权限吗?'])
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

## edit.blade.php

```
@extends('backend.layout.main')
@section('content')
    <div class="row">
        <div class="col-md-6">
            <div class="box box-info">
                <form class="form-horizontal" action="{{URL::to('permission/'.$permission->id)}}" method="post" enctype="multipart/form-data">
                    <div class="box-header with-border">
                        <h3 class="box-title">编辑菜单</h3>
                        <input type="hidden" name="_token" value="{{csrf_token()}}">
                        <input type="hidden" name="_method" value="put">
                    </div>
                    <div class="box-body">
                        <div class="form-group">
                            <label for="name" class="col-sm-3 control-label">权限名称</label>
                            <div class="col-sm-9">
                                <input type="text" class="form-control" id="name" name="name" placeholder="权限名称" value="{{$permission->name}}">
                                @include('backend.layout.message.tips',['field'=>'name'])
                            </div>
                        </div>
                        <div class="form-group">
                            <label for="display_name" class="col-sm-3 control-label">权限标识</label>
                            <div class="col-sm-9">
                                <input type="text" class="form-control" id="display_name" name="display_name" placeholder="权限标识" value="{{$permission->display_name}}">
                                @include('backend.layout.message.tips',['field'=>'display_name'])
                            </div>
                        </div>
                        <div class="form-group">
                            <label for="description" class="col-sm-3 control-label">菜单地址</label>
                            <div class="col-sm-9">
                                <input type="text" class="form-control" id="description" name="description" placeholder="权限描述" value="{{$permission->description}}">
                                @include('backend.layout.message.tips',['field'=>'description'])
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

## create.blade.php
```
@extends('backend.layout.main')
@section('content')
    <div class="row">
        <div class="col-md-6">
            <div class="box box-info">
                <form class="form-horizontal" action="{{URL::to('permission')}}" method="post" enctype="multipart/form-data">
                    <div class="box-header with-border">
                        <h3 class="box-title">{{$page_title or "Page Title"}}</h3>
                        <input type="hidden" name="_token" value="{{csrf_token()}}">
                    </div>
                    <div class="box-body">
                        <div class="form-group">
                            <label for="name" class="col-sm-3 control-label">权限标识</label>
                            <div class="col-sm-9">
                                <input type="text" class="form-control" id="name" name="name" placeholder="权限标识" value="{{old('name')}}">
                                @include('backend.layout.message.tips',['field'=>'name'])
                            </div>
                        </div>
                        <div class="form-group">
                            <label for="display_name" class="col-sm-3 control-label">权限名称</label>
                            <div class="col-sm-9">
                                <input type="text" class="form-control" id="display_name" name="display_name" placeholder="权限名称" value="{{old('display_name')}}">
                                @include('backend.layout.message.tips',['field'=>'display_name'])
                            </div>
                        </div>
                        <div class="form-group">
                            <label for="description" class="col-sm-3 control-label">权限描述</label>
                            <div class="col-sm-9">
                                <input type="text" class="form-control" id="description" name="description" placeholder="权限名称" value="{{old('description')}}">
                                @include('backend.layout.message.tips',['field'=>'description'])
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

# 填充数据

***

打开文件`database/seeds/DatabaseSeeder.php`，修改代码如下：
```
<?php

use Illuminate\Database\Seeder;
use Illuminate\Database\Eloquent\Model;
use App\Models\User;
use App\Models\Menu;
use App\Models\Role;

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
        $this->call(RoleTableSeeder::class);
        $this->call(PermissionTableSeeder::class);
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

class RoleTableSeeder extends Seeder{
    public function run()
    {
        DB::table('roles')->delete();
        Role::create(['name' => 'admin', 'display_name' => 'User Administrator', 'description' => 'User is allowed to manage and edit other users']);
        Role::create(['name' => 'owner', 'display_name' => 'Project Owner', 'description' => 'User is the owner of a given project']);
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
```
接着在终端执行以下命令回滚并再次执行迁移，填充数据
```
php artisan migrate:refresh --seed
```

# 效果预览

***

![屏幕快照 2016-02-22 09.50.45.png](http://upload-images.jianshu.io/upload_images/1277229-465b839dd3946f01.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
