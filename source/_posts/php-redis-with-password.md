---
title: Redis开启密码, 配置Yii2 redis密码访问
date: 2015-12-18 20:01:54
tags:
- PHP
- Yii2
- redis
---


```shell
sudo vi /etc/redis/redis.conf

#添加密码
requirepass abc123_

#重启redis-server
sudo service redis-server restart

#或者
sudo redis-server /etc/redis/redis.conf
```

<!-- more --> 

# 配置Yii2 redis 支持密码访问

```php
#修改common/config/main.php
'redis' => [
       'class' => 'yii\redis\Connection',
       'hostname' => 'localhost',
       'port' => 6379,
       'database' => 2,
       'password' => 'abc123_'
 ],
```

# PHP 如何连接Redis并支持auth

## 需要php-redis扩展

> https://github.com/phpredis/

## 具体连接实例

```php
//连接本地的 Redis 服务
$redis = new \Redis();
$redis->connect('127.0.0.1', 6379);
//验证服务
$redis->auth('abc123_');
//选择连接库
$redis->select(1);
//查看服务是否运行
$keysList = $redis->keys("*");
```
