---
title: MongoDB 索引小结
date: 2016-01-27 20:12:55
tags:
- mongodb
- 索引
---

# MongoDB Indexes

- 默认 _id 会作为collection的索引
- 对于单个字段的索引, 排序方向并不影响索引的使用
- 嵌入文档的索引,必须要完全匹配
- 对于复合索引,可以匹配任意带有前缀的查询

<!-- more -->

```json
{ "item": 1, "location": 1, "stock": 1 }
```

 支持:

```json
{ item: 1 }
{ item: 1, location: 1 }
```
- 排序

```js
db.events.createIndex( { "username" : 1, "date" : -1 } )
```

支持sort:

```js
db.events.find().sort( { username: -1, date: 1 } )
db.events.find().sort( { username: 1, date: -1 } )
```

不支持:

```js
db.events.find().sort( { username: 1, date: 1 } )
```

## 实验

> MongoDB 版本

```shell
~ » mongo --version                                     
MongoDB shell version: 3.0.8
```

### 单个索引

> 创建订单表, 插入10000条数据

```shell
~ » mongo
for(var i = 0; i < 10000; i++) { 
    db.order.insert({"sku": i+1, "cid": 10000 - i});
}
```

> 根据sku查询, 使用**explain()**进行分析

```js
 db.order.find({"sku":100}).explain()
```

```json
{
    "queryPlanner" : {
        "plannerVersion" : 1,
        "namespace" : "test.order",
        "indexFilterSet" : false,
        "parsedQuery" : {
            "sku" : {
                "$eq" : 100
            }
        },
        "winningPlan" : {
            "stage" : "COLLSCAN",
            "filter" : {
                "sku" : {
                    "$eq" : 100
                }
            },
            "direction" : "forward"
        },
        "rejectedPlans" : [ ]
    },
    "ok" : 1
}
```

观察到 winningPlan 的 stage 为 **COLLSCAN** 为 **全表扫描**

> 添加索引 

```js
db.order.ensureIndex({"sku":1})
```

> 再次根据sku查询

```js
> db.order.find({"sku":100}).explain()
{
    "queryPlanner" : {
        "plannerVersion" : 1,
        "namespace" : "test.order",
        "indexFilterSet" : false,
        "parsedQuery" : {
            "sku" : {
                "$eq" : 100
            }
        },
        "winningPlan" : {
            "stage" : "FETCH",
            "inputStage" : {
                "stage" : "IXSCAN",
                "keyPattern" : {
                    "sku" : 1
                },
                "indexName" : "sku_1",
                "isMultiKey" : false,
                "direction" : "forward",
                "indexBounds" : {
                    "sku" : [
                        "[100.0, 100.0]"
                    ]
                }
            }
        },
        "rejectedPlans" : [ ]
    }
}
```

发现现在 stage 为 **IXSCAN**, 意为 index scan, 索引扫描, 此时利用了索引

### 复合索引

> 创建collection, 添加数据

```js
for(var i = 0; i < 10000; i++) { 
    db.order_2.insert({"sku":i+1, "cid":9999-i});
}
```

> 添加复合索引 `sku_1_cid_1`

```js
db.order_2.ensureIndex({"sku":1, "cid":1});
```

> 观察现象

| 查询条件 | 查找方式 | 是否使用索引 |
| :-: | :-: | :-: |
| sku : 100 | IXSCAN | 是 |
| cid : 100 | COLLSCAN | 否 |
| sku:2, cid:9998 | IXSCAN | 是 |
| cid:2, sku:9998 | IXSCAN | 是 |

> 4个字段组成的复合索引支持的查询类型

```js
for(var i = 0; i < 10000; i++) { 
    db.order_4.insert({"sku":i+1, "cid":9999-i, "status": i * 100, "order" : i});
}
```

```js
db.order_4.ensureIndex({"sku":1,"cid":1,"status":1,"order":1})
```

索引 `sku_1_cid_1_status_1_order_1` 

| 查询条件 | 查找方式 | 是否使用索引 |
| :-- | :-: | :-: |
| sku : 100 | IXSCAN | 是 |
| cid : 100 | IXSCAN | 是 |
| status : 100 | IXSCAN | 是 |
| order : 100 | IXSCAN | 是 |
|  |  |  |
| sku:2, cid:9998 | IXSCAN | 是 |
| sku:9998, status: 100 | IXSCAN | 是 |
| sku:9998, order: 100 | IXSCAN | 是 |
| cid:1, status:100 | COLLSCAN | 否 |
| cid:1, order: 0 | COLLSCAN | 否 |
| status:100, order:1 | COLLSCAN | 否 |
|  |  |  |
| sku:1,cid:100,status:1 | IXSCAN | 是 |
| sku:1,cid:100,order:1 | IXSCAN | 是 |
| sku:1,status:100,order:0 | IXSCAN | 是 |
| cid:1,status:100,order:0 | COLLSCAN | 否 |
|  |  |  |
| sku:1,cid:100,status:1,order:1 | IXSCAN | 是 |

综上所述:
- 是否使用索引与查询条件的顺序无关
- 创建索引时指定的字段顺序很重要
- 只要查询条件满足索引的前缀就会使用索引
- 确保你创建的复合索引在大部分的查询语句中使用到前缀(包含最前面的字段)
