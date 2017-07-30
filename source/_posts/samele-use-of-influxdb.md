---
title: influxdb简单使用
date: 2016-10-11 20:59:13
tags:
- influxdb
---

> - github: https://github.com/influxdata/influxdb

## Deploying InfluxDB using Docker

> - docker influxdb:      http://www.tomdee.co.uk/2015/10/10/deploying-influxdb-using-docker/
- docker pull influxdb
- docker run -d --volume=/var/influxdb:/data --restart=always -e PRE_CREATE_DB="tomdee" -p 8083:8083 -p 8086:8086 influxdb:lastest

<!-- more -->

# Influxdb HTTP API

> https://docs.influxdata.com/influxdb/v1.0/guides/

Influxdb的http api的请求端口为: `8086`

## Writing Data

> https://docs.influxdata.com/influxdb/v1.0/guides/writing_data/

### create database

- `curl -i -XPOST http://localhost:8086/query --data-urlencode "q=CREATE DATABASE mydb"`
  - 使用POST方法
  - url为 `/query`
  - `q` 用于设置URL参数
  - `CREATE DATABASE mydb`为influxdb的语法

### writing data

- `curl -i -XPOST 'http://localhost:8086/write?db=mydb' --data-binary 'cpu_load_short,host=server01,region=us-west value=0.64 1434055562000000000'`
  - 使用POST方法
  - url为 `/write`
  - `db=mydb`用于指定db名称
  - cpu_load_short 为 `measurement`， 类似于table, collection的概念
  - host,region为tag key
  - server01, us-west为tag value
  - `tag key`和`tag value`都会被索引, 类型为`string`
  - value为`field key`, 不会被索引
  - 0.64为`field value`
  - 1434055562000000000 为 `timestamp`

**`/write` api支持一次写入多条数据或者从文件中读取， 但每条数据必须被组织在单行中**

## Schema Design

Influxdb是schemaless database(无模式)， 但是对于同一个`measurement`, `tag field`对应的`tag value`的**数据类型必须一致**

`curl -i -XPOST 'http://localhost:8086/write?db=mydb' --data-binary 'tobeornottobe booleanonly=true'`

```
HTTP/1.1 204 No Content
Content-Type: application/json
Request-Id: 44f427bc-8f57-11e6-8021-000000000000
X-Influxdb-Version: 1.0.1
Date: Tue, 11 Oct 2016 02:06:06 GMT
```

`curl -i -XPOST 'http://localhost:8086/write?db=mydb' --data-binary 'tobeornottobe booleanonly=5'`

```
HTTP/1.1 400 Bad Request
Content-Type: application/json
Request-Id: 4cbc4d09-8f57-11e6-8022-000000000000
X-Influxdb-Version: 1.0.1
Date: Tue, 11 Oct 2016 02:06:19 GMT
Content-Length: 142

{"error":"field type conflict: input field \"booleanonly\" on measurement \"tobeornottobe\" is type float64, already exists as type boolean"}

```

## Querying Data

> https://docs.influxdata.com/influxdb/v1.0/guides/querying_data/

- `curl -GET 'http://localhost:8086/query?pretty=true' --data-urlencode "db=mydb" --data-urlencode "q=SELECT \"value\" FROM \"cpu_load_short\" WHERE \"region\"='us-west'"`
  - 使用GET方法
  - url为 `/query`
  - `q` 用于设置URL参数
  - `CREATE DATABASE mydb`为influxdb的语法
  - 需要指定db名称
  - 查询语法中`tag key, field key`需要使用`""`引起来, `tag value`需要使用`''`引起来
  - `pretty=true`获取JSON格式的输出
