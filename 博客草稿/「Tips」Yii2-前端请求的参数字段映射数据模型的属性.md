---
title: 「Tips」Yii2 前端请求的参数字段映射数据模型的属性
date: 2017-03-15 17:18:34
tags: PHP
category: PHP
---

使用 Yii 2 构建 RESTFul API 项目时，需要实现一个功能。这个功能可以将 HTTP 请求的参数和后端数据模型相映射，让前端的请求参数命名和后端数据的模型属性不需要保持一致，并在指定场景中实现数据验证。实现代码如下：
```php
<?php
/*
 * 前端传递 JSON 数据进来，格式是{"name":"admin","pwd":"123456"}
 * 需要对数据在指定场景下验证，并对应到模型的 username 属性和 password_hash 属性
 */

// 模型代码
class SignInForm
{
	public $name;
	public $pwd;

	public function rules()
    {
        $rules = [
            [
                [
                    'name',
                    'pwd',
                ],
                'required',
                'on' => 'signIn',
            ],
            [
                ['name'],
                'default',
                'value' => $this->username,
                'on' => 'signIn',
            ],
            [
                ['pwd'],
                'default',
                'value' => $this->password_hash,
                'on' => 'signIn',
            ],
        ];

        return $rules;
    }
}

// 控制器代码
<?php

use Yii;
use workerbee\api\controllers\BaseController;

class UserController extends BaseController
{
public function actionSignIn()
    {
        $data['SignInForm'] = Yii::$app->request->post();
        $signInModel = new SignInForm(['scenario' => 'signIn']);

        if ($signInModel->load($data) && $signInModel->validate()) {
            $user = SignInForm::findByUsername($signInModel->username);
            $validator = Yii::$app->getSecurity()->validatePassword($signInModel->pwd, $user->password_hash);
            if ($user !== null && $validator) {
                $user->access_token = $signInModel->getNewAccessToken();
                if ($user->save()) {
                    return $user;
                }

                throw new ServerErrorHttpException('登陆失败.');
            }

            throw new BadRequestHttpException('账号名或密码错误.');
        }

        throw new UnprocessableEntityHttpException('数据验证失败.');
    }
}
```
