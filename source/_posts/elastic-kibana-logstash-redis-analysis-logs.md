---
title: Elastic + kibana + logstash + redis 对mongodb, nginx日志进行分析
date: 2015-10-28 19:47:43
tags:
- logstash
- kibana
---

## 项目分析

mongodb运行时没有将日志文件进行切割, 　随着运行时间的增加, mongod.log越来越大, 已经无法进行有效的数据分析了, 因此需要搭建一个日志分析平台, 可以索引每一条记录, 并能够提供方便快速准确的查询接口
网站的每一个访问都会在nginx的日志文件中产生一条记录, 通过kabana可以很好的展现中。

<!-- more -->

## 软件需求

### 1. [**elasticsearch**](https://www.elastic.co/downloads/elasticsearch) 版本1.7.3   [_(tar download)_](https://download.elastic.co/elasticsearch/elasticsearch/elasticsearch-1.7.3.tar.gz)
### 2. [**kibana**](https://www.elastic.co/downloads/kibana) 版本 Kibana 4.1.2     [_(64-bit tar download)_](https://download.elastic.co/kibana/kibana/kibana-4.1.2-linux-x64.tar.gz)　
### 3. [**logstash**](https://www.elastic.co/products/logstash) 版本 Logstash 1.5.4   [_(tar download)_](https://download.elastic.co/logstash/logstash/logstash-1.5.4.tar.gz)
### 4. [**redis**](http://redis.io/download) 版本 2.8.4
### 5. [**mongodb**](www.mongodb.org/) 版本 3.0.6

## 软件安装

### 安装 elasicsearch

``` shell
tar -zxvf elasticsearch-1.7.3.tar.gz  /home/user/elasticsearch-1.7.3
cd /home/user/elasticsearch-1.7.3/bin
./elasticsearch &
```

访问　http://127.0.0.1:9200 
![elasticsearch_start](https://cloud.githubusercontent.com/assets/5611286/10785282/f39afb46-7d9d-11e5-9c8f-bd8a00e9db48.png)

### 安装 kibana

``` shell
tar -zxvf kibana-4.1.2-linux-x64.tar.gz  /home/user/kibana-4.1.2-linux-x64
cd /home/user/kibana-4.1.2-linux-x64/bin
./kibana &
```

访问　http://127.0.0.1:5601/

### 安装redis

``` shell
sudo apt-get install redis-server
sudo apt-get install php5-redis
```

使用 `redis-cli -v` 查看安装是否成功

### 安装mongodb

[_安装教程_](http://docs.mongodb.org/manual/tutorial/install-mongodb-on-ubuntu/)

## 系统架构

![struct](https://cloud.githubusercontent.com/assets/5611286/10785251/b804181a-7d9d-11e5-8ad0-0e3e7f418882.png)

## 配置管理

### 开启logstash agent

``` shell
input {
        file {
                type => "mongo access log"
                path => ["/var/log/mongodb/mongod.log"]
        }
　　file {
                type => "nginx access log"
                path => ["/var/log/nginx/*.log"]
        }　
}
output {
        redis {
                host => "127.0.0.1" #redis server
                data_type => "list"
                key => "logstash:redis"
        }
}
```

可以这么理解: logstash 将 `/var/log/mongodb/mongod.log`　作为输入, 每一次mongod.log文件有改动
都会将新添加的内容输出到 output中（这里我们配置的是redis中, 存储的类型为list）

### 开启logstash indexer

``` shell
input {
        redis {
                host => "127.0.01"
                data_type => "list"
                key => "logstash:redis"
                type => "redis-input"
        }
}
output {
        elasticsearch {
                embedded => false
                protocol => "http"
                host => "localhost"
                port => "9200"
        }
}
```

将`redis`中的内容输出到`elasticsearch`

### 开启 mongodb的日志追加模式

开启　`verbose = true`

操作 `mongodb`　使产生新的日志，　打开kabana的接口 `http://127.0.0.1:5601` 即可观察到：

![kibana](https://cloud.githubusercontent.com/assets/5611286/10782359/a2bbab2a-7d8a-11e5-98de-92ead6f47169.png)
