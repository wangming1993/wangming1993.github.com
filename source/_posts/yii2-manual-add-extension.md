---
title: Yii2 手动添加扩展模块 mongodb
date: 2015-11-06 19:54:55
tags:
- Yii2
---

## Yii2手动添加扩展模块 mongodb, 非composer方式

> 因为某些原因, 国内composer方式难以成功, 你懂的 , 而Yii2 的官方推荐就是使用composer的方式安装第三法扩展

<!-- more -->

## 需求

安装　[**yii2-mongodb**](https://github.com/yiisoft/yii2-mongodb)

## 实现步骤

1. 下载yii2-mongodb的源码，　拷贝到 yii2 的　vendor/yiisoft/　目录下
   ![vendor](https://cloud.githubusercontent.com/assets/5611286/10987013/33eff32c-846a-11e5-8826-0ddbfde4bde3.png)
2. 添加扩展配置

[[yii\base\Application::extensions|extensions]]

该属性用数组列表指定应用安装和使用的 扩展，默认使用@vendor/yiisoft/extensions.php文件返回的数组。 当你使用 Composer 安装扩展，extensions.php 会被自动生成和维护更新。 所以大多数情况下，不需要配置该属性。

特殊情况下你想自己手动维护扩展，可以参照如下配置该属性：

``` php
[
    'extensions' => [
        [
            'name' => 'extension name',
            'version' => 'version number',
            'bootstrap' => 'BootstrapClassName',  // 可选配，可为配置数组
            'alias' => [  // 可选配
                '@alias1' => 'to/path1',
                '@alias2' => 'to/path2',
            ],
        ],

        // ... 更多像上面的扩展 ...

    ],
]
```

如上所示，该属性包含一个扩展定义数组，每个扩展为一个包含 name 和 version 项的数组。 如果扩展要在 引导启动 阶段运行，需要配置 bootstrap以及对应的引导启动类名或 configuration 数组。 扩展也可以定义 别名

在　config/web.php 添加如下配置：

``` php
'extensions' => [
        [
            'name' => 'yiisoft/yii2-mongodb',
            'version' => '9999999-dev',
            'alias' => [  // 可选配
                '@yii/mongodb' => __DIR__ . '/../vendor/yiisoft/yii2-mongodb',
            ],
        ],
        // ... 更多像上面的扩展 ...
    ],
```

## 测试

### 添加一个　model

``` php
<?php

namespace app\models;

use Yii;
use yii\mongodb\ActiveRecord;

class Mongo extends ActiveRecord
{
    public static function collectionName()
    {
        return 'mongo';
    }

    public function attributes()
    {
        return ['_id', 'name', 'address', 'status'];
    }

    public function fields()
    {
        return ['name'];
    }

}
```

### 添加action

``` php

namespace app\controllers;

use Yii;
use yii\web\Controller;

use app\models\Mongo;

class SiteController extends Controller
{
    public function actionMongo()
    {
        $mongo = new Mongo();
        $mongo->name = 'aug';
        $mongo->save();
        $response = Yii::$app->response;
        $response->format = \yii\web\Response::FORMAT_JSON;
        return Mongo::findAll(['name' => 'aug']);
    }
}

```

### 访问

![access](https://cloud.githubusercontent.com/assets/5611286/10987143/76d82f50-846b-11e5-9573-2d57374795b9.png)

ＯＫ, 成功添加mongodb扩展.
