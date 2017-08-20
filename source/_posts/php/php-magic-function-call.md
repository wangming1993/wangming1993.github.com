---
title: PHP 魔法函数 __call
date: 2015-12-17 19:58:19
tags:
- PHP
---

1. 调用一个类中的方法，　如果方法名不存在，就会抛异常，可以使用 
   __call($method, $args) 来同意处理这种调用
2. 另一种case就是如果一个类需要对外提供很多方法，而这些方法的实际
   处理中具有很强的一致性，那么没必要将这些方法全部写出，可以使用
   一种统一的逻辑来处理．

<!-- more -->

先贴上一段代码：

``` php
<?php
namespace ns;

class People
{
    public function __construct()
    {
        echo "Construct People \n";
    }

    public function __call($method, $args)
    {
        echo "Not exist $method \n";
    }

    public static function __callStatic($method, $args)
    {
        echo "Not exist static $method \n";
    }

    public function go()
    {
        echo "Call class method go \n";
    }

    public static function run()
    {
        echo "Call static method run \n";
    }
}

$p = new People;
$p->exit('exit');
People::done('done');
```

`People`实例对象调用`exit`方法，　但`People`类中并没有`exit`方法，　
此时就会调用 `__call`方法，　并将方法名和参数传递给`__call`方法

`__callStatic` 则是用于调用_静态方法_时的处理

> 很神奇的地方, PHP中貌似实例对象也可以调用静态方法
