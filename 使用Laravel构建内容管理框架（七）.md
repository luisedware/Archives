![Happy Coding](http://upload-images.jianshu.io/upload_images/1277229-a09205aecddd5bf8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
# 本文目标

***

新增角色管理模块，管理角色的增删查改。

# 新增请求

***

```
php artisan make:request Form/RoleForm
```

文件`RoleForm`代码如下：
```
<?php

namespace App\Http\Requests;

class RoleForm extends Request
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
            'name.required'         => '角色标识不能为空',
            'display_name.required' => '角色名称不能为空',
            'description.required'  => '角色描述不能为空'
        ];
    }
}
```

# 新增控制器

***

```
php artisan make:controller Backend/RoleController
```

文件`RoleController.php`代码如下：
```
<?php

namespace App\Http\Controllers\Backend;

use App\Models\Role;
use App\Http\Requests;
use App\Http\Requests\Form\RoleForm;
use App\Http\Controllers\Controller;

class RoleController extends Controller
{
    /**
     * Display a listing of the resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function index()
    {
        $roles = Role::paginate(25);

        return view('backend.role.index', compact('roles'));
    }

    /**
     * Show the form for creating a new resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function create()
    {
        return view('backend.role.create');
    }

    /**
     * Store a newly created resource in storage.
     *
     * @param  \Illuminate\Http\RoleForm $request
     *
     * @return \Illuminate\Http\Response
     */
    public function store(RoleForm $request)
    {
        try {
            if (Role::create($request->all())) {
                return redirect()->route('role.index')->withSuccess('新增角色成功');
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
        return view('backend.role.show', compact('id'));
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
        $role = Role::find($id);

        return view('backend.role.edit', compact('role'));
    }

    /**
     * Update the specified resource in storage.
     *
     * @param  \Illuminate\Http\Request $request
     * @param  int                      $id
     *
     * @return \Illuminate\Http\Response
     */
    public function update(RoleForm $request, $id)
    {
        $data = $request->all();
        unset($data['_token']);
        unset($data['_method']);
        try {
            if (Role::where('id', $id)->update($data)) {
                return redirect()->back()->withSuccess('编辑角色成功');
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
            if (Role::destroy($id)) {
                return redirect()->back()->withSuccess('删除角色成功');
            }
        } catch (\Exception $e) {
            return redirect()->back()->withErrors(array('error' => $e->getMessage()));
        }
    }
}
```

# 新增视图

***

## index.blade.php
```
@extends('backend.layout.main')

@section('content')
    <div class="row">
        <div class="col-xs-1">
            <div class="small-box">
                <a href="{{URL::to('role/create')}}" class="btn btn-success">新增角色</a>
            </div>
        </div>
    </div>
    <div class="row">
        <div class="col-md-12">
            <div class="box">

                <div class="box-header with-border">
                    <h3 class="box-title">角色列表</h3>

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
                            <th>角色编号</th>
                            <th>角色标识</th>
                            <th>角色名称</th>
                            <th>角色描述</th>
                            <th>管理操作</th>
                        </tr>
                        @forelse($roles as $role)
                            <tr>
                                <td>{{$role->id}}</td>
                                <td>{{$role->name}}</td>
                                <td>{{$role->display_name}}</td>
                                <td>{{$role->description}}</td>
                                <td>
                                    <a class="btn btn-info" href="{{URL::to('role/'.$role->id.'/edit')}}">
                                        编辑
                                    </a>
                                    <a class="btn btn-primary" href="{{URL::to('role/'.$role->id)}}">
                                        赋权
                                    </a>
                                    <a class="btn btn-danger" data-toggle="modal" data-target="#defalutModal" data-url="{{URL::to('role/'.$role->id)}}">
                                        删除
                                    </a>
                                </td>
                            </tr>
                        @empty
                            <tr>
                                <td colspan="4" class="text-center">暂无数据</td>
                            </tr>
                        @endforelse
                    </table>
                </div>

                @if($roles->render() !== "")
                    <div class="box-footer">
                        {!! $roles->render() !!}
                    </div>
                @endif
            </div>
        </div>
    </div>
    @include('backend.layout.model.default',['model_title'=>'操作提示','model_content'=>'你确定要删除这名角色吗?'])
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

## show.blade.php
```
@extends('backend.layout.main')

@section('content')

@stop
@section('script')

@stop
```

## edit.blade.php
```
@extends('backend.layout.main')
@section('content')
    <div class="row">
        <div class="col-md-6">
            <div class="box box-info">
                <form class="form-horizontal" action="{{URL::to('role/'.$role->id)}}" method="post" enctype="multipart/form-data">
                    <div class="box-header with-border">
                        <h3 class="box-title">{{$page_title or "Page Title"}}</h3>
                        <input type="hidden" name="_token" value="{{csrf_token()}}">
                        <input type="hidden" name="_method" value="put">
                    </div>
                    <div class="box-body">
                        <div class="form-group">
                            <label for="name" class="col-sm-3 control-label">角色标识</label>
                            <div class="col-sm-9">
                                <input type="text" class="form-control" id="name" name="name" placeholder="角色标识" value="{{$role->name}}">
                                @include('backend.layout.message.tips',['field'=>'name'])
                            </div>
                        </div>
                        <div class="form-group">
                            <label for="display_name" class="col-sm-3 control-label">角色名称</label>
                            <div class="col-sm-9">
                                <input type="text" class="form-control" id="display_name" name="display_name" placeholder="角色名称" value="{{$role->display_name}}">
                                @include('backend.layout.message.tips',['field'=>'display_name'])
                            </div>
                        </div>
                        <div class="form-group">
                            <label for="description" class="col-sm-3 control-label">角色描述</label>
                            <div class="col-sm-9">
                                <input type="text" class="form-control" id="description" name="description" placeholder="角色描述" value="{{$role->description}}">
                                @include('backend.layout.message.tips',['field'=>'description'])
                            </div>
                        </div>
                    </div>
                    <div class="box-footer">
                        <button type="button" class="btn btn-default" onclick="javascript:history.back('{{route('role.index')}}');return false;">
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
                <form class="form-horizontal" action="{{URL::to('role')}}" method="post" enctype="multipart/form-data">
                    <div class="box-header with-border">
                        <h3 class="box-title">{{$page_title or "Page Title"}}</h3>
                        <input type="hidden" name="_token" value="{{csrf_token()}}">
                    </div>
                    <div class="box-body">
                        <div class="form-group">
                            <label for="name" class="col-sm-3 control-label">角色标识</label>
                            <div class="col-sm-9">
                                <input type="text" class="form-control" id="name" name="name" placeholder="角色标识" value="{{old('name')}}">
                                @include('backend.layout.message.tips',['field'=>'name'])
                            </div>
                        </div>
                        <div class="form-group">
                            <label for="display_name" class="col-sm-3 control-label">角色名称</label>
                            <div class="col-sm-9">
                                <input type="text" class="form-control" id="display_name" name="display_name" placeholder="角色名称" value="{{old('display_name')}}">
                                @include('backend.layout.message.tips',['field'=>'display_name'])
                            </div>
                        </div>
                        <div class="form-group">
                            <label for="description" class="col-sm-3 control-label">角色描述</label>
                            <div class="col-sm-9">
                                <input type="text" class="form-control" id="description" name="description" placeholder="角色描述" value="{{old('description')}}">
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
```
接着在终端执行以下命令回滚并再次执行迁移，填充数据
```
php artisan migrate:refresh --seed
```

# 效果预览

***

![屏幕快照 2016-02-19 17.19.36.png](http://upload-images.jianshu.io/upload_images/1277229-33f4df310c1ecac8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
