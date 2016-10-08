---
title: 使用 Laravel 构建内容管理框架（四）
date: 2016-04-04 20:33:17
tags: Laravel
category: PHP
---

新增用户管理模块功能

# 新增验证请求
***
在终端执行以下命令，在文件夹`app/Http/Requests/Form`下新增表单验证`UserForm`
```
php artisan make:request Form/UserForm
```

修改文件`UserForm`代码如下：
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
            'name'                  => 'required',
            'email'                 => 'required',
            'password'              => 'required|confirmed',
            'password_confirmation' => 'required'
        ];
    }

    public function messages()
    {
        return [
            'name.required'                  => '用户名称不能为空',
            'email.required'                 => '用户邮箱不能为空',
            'password.required'              => '用户密码不能为空',
            'password.confirmed'             => '确认密码不一致',
            'password_confirmation.required' => '确认密码不能为空'
        ];
    }
}
```

# 新增控制器UserController

***

在终端执行以下命令，在文件夹`app/Http/Controllers/Backend`新增文件`UserController.php`
```
php artisan make:controller Backend/UserController
```

修改文件`UserController.php`代码如下：
```
<?php

namespace App\Http\Controllers\Backend;

use App\Models\User;
use App\Http\Requests\Form\UserForm;
use App\Http\Requests;
use App\Http\Controllers\Controller;

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
        return view('backend.user.create');
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
            if (User::create($data)) {
                return redirect()->back()->withSuccess('新增用户成功');
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

        return view('backend.user.edit', compact('user'));
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
        $data = [
            'name'     => $request['name'],
            'email'    => $request['email'],
            'password' => bcrypt($request['password']),
        ];

        try {
            if (User::where('id', $id)->update($data)) {
                return redirect()->back()->withSuccess('编辑用户成功');
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

# 新增路由

***

在文件`app\Http\routes.php`新增代码如下：
```
Route::group(['namespace' => 'Backend', 'middleware' => ['auth']], function () {
    Route::get('/', 'IndexController@index');
    Route::resource('user', 'UserController');
});
```

# 新增视图

***

在文件夹`resources/views/backend/`新建文件夹`user`，并新建以下模板文件：

- index.blade.php
- create.blade.php
- edit.blade.php

### index.blade.php
```
@extends('backend.layout.main')

@section('content')
    <div class="row">
        <div class="col-md-1">
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
                                <td></td>
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
                                <select class="form-control select2" name="role_id">
                                    <option value="0">/</option>
                                    {{--@foreach($roles as $role)--}}
                                        {{--<option value="{{$role->id}}">{{$role->display_name}}</option>--}}
                                    {{--@endforeach--}}
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
                <form class="form-horizontal" action="{{URL::to('user/'.$user->id)}}" method="post" enctype="multipart/form-data">
                    <div class="box-header with-border">
                        <h3 class="box-title">{{$page_title or "Page_title"}}</h3>
                        <input type="hidden" name="_token" value="{{csrf_token()}}">
                        <input type="hidden" name="_method" value="put">
                    </div>
                    <div class="box-body">
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
@section('script')
    <script type="text/javascript">
        $(function () {
            $(".select2").select2();
        });
    </script>
@stop

```

## 新增视图组件

***

在文件夹`resources/views/layout/`新建文件夹`model`，并新建模板文件`default.blade.php`，代码如下：
```
<div class="modal fade" id="defalutModal" tabindex="-1" role="dialog" aria-labelledby="defaultModalLabel">
    <form class="form-horizontal" method="post" enctype="multipart/form-data">
        <div class="modal-dialog" role="document">
            <div class="modal-content">
                <div class="modal-header">
                    <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                        <span aria-hidden="true">×</span></button>
                    <h4 class="modal-title" id="defaultModalLabel">{{$model_title}}</h4>
                </div>
                <div class="modal-body">
                    {{$model_content}}
                    <input type="hidden" name="_token" value="{{csrf_token()}}">
                    <input type="hidden" name="_method" value="delete">
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-default" data-dismiss="modal">关闭</button>
                    <button type="submit" class="btn btn-primary">确认</button>
                </div>
            </div>
        </div>
    </form>
</div>
```
