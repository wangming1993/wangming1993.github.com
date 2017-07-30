---
title: PHP 静态方法中获取调用者的类名
date: 2017-07-26 18:31:28
tags:
- PHP
---

## 需求

在写一个简单的orm框架时, 需要在父类的静态方法中获取到实际调用类的信息，
而通过　`__class__` ,  `get_class()` 等只能获取到当前类的类名

## 解决方案

使用　`get_called_class()` 即可获取调用类的类名

<!-- more -->

## 实际案例

``` php
<?php

class Model {
    public static function say() {
        echo __CLASS__;
        echo get_called_class();
    }
}

```

``` php
<?php
require 'model.php';

class Kid extends Model {

}

Kid::say();
```
