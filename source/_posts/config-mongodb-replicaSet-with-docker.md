---
title: 在docker容器内使用容器外的mongodb的复制集
date: 2017-06-16 21:22:06
tags:
- mongodb
- 复制集
---

复制集配置:

```shell
config = {
    _id: "mike", 
    members: [ 
        {_id:1, host:"127.0.0.1:30001"}, 
        {_id:2, host:"127.0.0.1:30002"}, 
        {_id:3, host:"127.0.0.1:30003",arbiterOnly:true} 
    ]
}
rs.initiate(config);
```

<!-- more --> 

使用`PHP`测试mongodb复制集和读写分离，以下是测试的代码:

```php
<?php

$dsn = 'mongodb://192.168.222.107:30001/mike';
$options = [
    'replicaSet' => 'mike',
    'readPreference' => 'primary',
    'w' => 1,
    'wtimeout' => 2000,
    'connectTimeoutMS' => 5000,                                                                                            
];

$manager = new MongoDB\Driver\Manager($dsn, $options);
$readPreference = $manager->getReadPreference();
$server = $manager->selectServer($readPreference)
```

使用docker环境下的PHP中执行,出现错误:

```shell
No suitable servers found (`serverselectiontryonce` set): [connection timeout calling ismaster on '127.0.0.1:3306']
```

为了确定数据库运行正常，不使用复制集配置，去掉了`options`中的`replicaSet`配置，发现正常连接，基本确认数据库运行正常. 那么问题就出现在复制集的使用上。

分析错误，问题出在确认`master`时**connection refused**. 于是我单独使用`mongo shell`发送命令确认:

```shell
mongo 192.168.222.107:30001 --eval 'printjson(db.runCommand({"isMaster": 1}))'
```
也显示连接正常.

分析场景，PHP代码的执行是在docker容器内，`mongodb`实例是运行是在宿主机，虽然PHP代码中的连接配置是`192.168.222.107:30001`,宿主机的IP地址，但是发送`isMaster`命令的时候却是使用的**127.0.0.1:30001**,这样是连接的docker容器内的，因为容器内没有mongodb实例，所有以肯定**connection refused**.  

**为什么发送isMaster的时候使用的是127.0.0.1:30001呢**

因为复制集的config中指定的host就是`127.0.0.1`

更改复制集配置:

```shell
config = {
            _id: "mike", 
    members: [ 
        {_id:1, host:"192.168.222.107:30001"}, 
        {_id:2, host:"192.168.222.107:30002"}, 
        {_id:3, host:"192.168.222.107:30003",arbiterOnly:true} 
    ]
}
rs.reconfig(config);
```
重新运行代码，一切OK. oops!!!
