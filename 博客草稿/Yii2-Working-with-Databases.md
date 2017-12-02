---
title: Yii2 Working with Databases
date: 2016-12-02 11:09:02
tags: Yii
category: PHP
---

## Create
```php
<?php
 // 单条数据插入
Yii::$app->db->createCommand()->insert('shop_user', [
    'userName' => 'HelloWorld',
    'userPass' => md5(123456),
    'userEmail' => 'hello@world.com',
    'createdAt' => time(),
])->execute();

$customer = new Customer();
$customer->name = 'Ann';
$customer->email = 'AnnEason@qq.com';
$customer->save();

// 批量数据插入
Yii::$app->db->createCommand()->batchInsert('shop_user',
    ['userName', 'userPass', 'userEmail', 'createdAt'],
    [
        ['Ann', md5(123456), 'Ann@qq.com', time()],
        ['Leon', md5(123456), 'Leon@qq.com', time()],
        ['bitch', md5(123456), 'bitch@qq.com', time()],
        ['Linda', md5(123456), 'Linda@qq.com', time()],
    ]
)->execute();
```

## Update
```php
<?php
// 数据更新方法(表名, 字段与值, 条件)
Yii::$app->db->createCommand()->update('shop_user', ['userName' => 'hello'], 'userId = 6')->execute();

$customer = Customer::findOne(123);
$customer->email = 'AnnEason@qq.com';
$customer->save();

$values = [
	'name' => 'Ann',
	'email' => 'Ann@qq.com'
];

$customer = new Customer();
$customer->attributes = $values;
$customer->save();

Customer::updateAll(['status' => Customer::STATUS_ACTIVE],['like','email','@example.com']);
```

## Retrieve
```php
<?php
// 存在数据返回二维数组,没有数据返回空数组
$orders = Yii::$app->db->createCommand("select * from z_order order by id desc limit 100 ;")->queryAll();
$rows = (new \yii\db\Query())->select(['id','email'])->from('user') ->all();
$models = Model::find()->where(['status' => Model::STATUS])->all();

// 存在数据返回一维数组,没有数据返回 false
$order = Yii::$app->db->createCommand("select * from z_order where id = 802478;")->queryOne();
$row = (new \yii\db\Query())->from('user')->where(['like','username','test'])->one();
$model = Model::find()->where(['id'=>1])->one();

// 存在数据返回一维数组,没有数据返回空数组
$orderNumbers = Yii::$app->db->createCommand("select orderno from z_order order by id desc limit 100")
->queryColumn();
$columns = (new \yii\db\Query())->select('id')->from('user')->column()

// 存在数据返回标量数值,没有数据返回 false
$orderCount = Yii::$app->db->createCommand("select count(orderno) from z_order")->queryScalar();
$count = (new \yii\db\Query())->from('user')->where(['like','username','test'])->count();
$modelCount = Model::find()->where(['status'=>1])->count();
```

## Delete
```php
<?php
// 数据删除方法(表名，条件)
Yii::$app->db->createCommand()->delete('shop_user', 'userName = "HelloWorld"')->execute();

$customer = Customer::findOne(123);
$customer->delete();

Customer::deleteAll(['status' => Customer::STATUS_INACTIVE]);
```

## Where
```php
<?php
// String Format
$query->where('status = 1');
$query->where('status = :status', [':status' => $status]);
$query->where('status = :status')->addParams([':status' => $status]);

// Hash Format ...WHERE (`status` = 10) AND (`type` IS NULL) AND (`id` IN (4, 8, 15))
$query->where([
    'status' => 10,
    'type' => null,
    'id' => [4, 8, 15],
]);

// Operator Format [operator, operand1, operand2, ...]
$query->where(['and', 'id=1', 'id=2']);
$query->where(['and', 'type=1', ['or', 'id=1', 'id=2']]);

$query->where(['between', 'id', 1, 10]);
$query->where(['not between', 'id', 1, 10]);

$query->where(['in', 'id', [1, 2, 3, 4, 5]]);
$query->where(['not in', 'id', [1, 2, 3, 4, 5]]);

$query->where(['like', 'name', 'hello']);
$query->where(['like', 'name', ['hello', 'world']]);

$query->where(['>', 'age', 10])
$query->andWhere(['like', 'title', 'hello-world']);
$query->orWhere(['like','name','LuisEdware']);

$query->filterWhere(['name' => $name, 'email' => $email]);
$query->andFilterWhere(['!=', 'id', 1]);
$query->orFilterWhere(['status' => 2]);

$query->andFilterCompare('name', 'John Doe');
```

## orderBy
```php
<?php
$query->orderBy([
	'id' => SORT_ASC,
	'name' => SORT_DESC,
]);

$query->orderBy('id ASC, name DESC');

$query->orderBy('id ASC')->addOrderBy('name DESC');
```

## groupBy
```php
<?php

$query->groupBy(['id','status']);

$query->groupBy('id,status');

$query->groupBy(['id','status'])->addGroupBy('age');
```

## having
```php
<?php

$query->having(['status'=>1]);

$query->having(['status'=>1])->andHaving(['>','age',30]);
```

## limit and offset
```php
<?php
$query->limit(10)->offset(20);
```

## join
```php
<?php
// ... LEFT JOIN `post` ON `post`.`user_id` = `user`.`id`
$query->join('LEFT JOIN','post','post.user_id = user.id');
```

## union
```php
<?php
$query1 = (new \yii\db\Query())
	->select('id,category_id AS type, name')
	->from('post')
	->limit(10);

$query2 = (new \yii\db\Query())
	->select('id,type,name')
	->from('user')
	->limit(10);

$query1->union($query2);
```

## find
```php
<?php
// SELECT * FROM customer WHERE id = 123;
$customer = Customer::findOne(123);

// SELECT * FROM customer WHERE id IN (1,2,3,4,5)
$customers = Customer::findOne([1,2,3,4,5]);

$customer = Customer::findOne(['id'=>123, 'status'=> Customer::STATUS]);
$customers = Customer::findAll(['id'=>123, 'status'=> Customer::STATUS_ACTIVE]);
```

## Relations
```php
<?php
class Order extends ActiveRecord
{
    public function getOrderItems()
    {
        return $this->hasMany(OrderItem::className(), ['order_id' => 'id']);
    }

    public function getItems()
    {
        return $this->hasMany(Item::className(), ['id' => 'item_id'])
            ->via('orderItems');
    }
}
$items = $order->items;


$customers = Customer::find()->with('orders.items')->all();
// no SQL executed
$items = $customers[0]->orders[0]->items;
```


