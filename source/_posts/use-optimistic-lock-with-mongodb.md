---
title: MongoDB中关于乐观锁的一些使用
date: 2015-12-27 20:05:28
tags:
- mongodb
---


> 很多时候我们对一些数据的操作总是涉及到原子性一致性问题，这类问题往往是系统功能中很重要的环节，需要正确有效的处理． 

我所使用的数据模型是存储在`MongoDB`中的, 当然`乐观锁`这种解决方案是与数据库无关的，重要的是思想本身．

<!-- more -->

## 场景

主要是客服功能，用户在连接客服时，会存在一种竞争资源，这个资源就是客服服务人数，一个客服能够服务的人数是有限的，那么当多个用户接入时，就需要保证个客服分配的客户不能超过其自身能够服务的最大人数．

初期我们为了实现功能，各种实现方式都是针对正常分配的场景，但是正常的使用情况却不可能如此简单，`QA`报出的`bug`说客服能够分配超出本身容量的客户，理所当然，我们并没有做任何关于这方面的处理．

## 解决方案

首先我们知道问题的所在，这就成功了一半，哈哈:smile: ．分析一下用户连接客服所在的一些工作：
1. 创建会话，将客服与客户建立联系
2. 更新缓存，将客服连接时间写入缓存
3. 更新客服服务人数
4. 通知客户连接成功分配客服

问题在于更新客服服务人数，如果现在客服最多能够接入１客户，但是有两个或更多客户连接，按照一开始的逻辑会都给予分配，这样就会超出容量．如果我们在更新的时候就能判断出能否分配，那么问题就迎刃而解了．
改进后的实现步骤:
1. 更新客服服务人数
2. 未能成功分配，将客户放入等待队列，通知客服等待客服
3. 成功分配，更新缓存
4. 建立客服与客户的连接会话
5. 通知客户连接成功分配客服

接下来就是如何解决在更新客服服务人数时是否更新成功
`MongoDB`的更新update的第一个参数是查询条件，第二个参数是更新操作，当查询语句不满足是便不会进行更新．
因此我们的更新语句是：

```javascript
db.getCollection('helpdesk').update(
    {
        '_id': helpdeskId, 
        'clientCount': {'$lt': maxClientLimit}
    },
    {
        '$set': {'$inc': {'clientCount': 1}}
    }
);
```

通过判断结果是否＞０来判断是否更新成功．
改进后的方案成功通过了_QA_妹子的测试，:+1: 哈哈

## 问题引申

这里可以引申出一个数据库设计方面的问题，这里我们能够很简单的通过更新语句来解决问题，是因为我们的collection在设计的时候存储了clientCount, 如果没有这个field那么如何去做这种判断呢

就好比做商品销量更新操作中，商品数量应该也是一个竞争资源，购买商品时需要实时判断商品数量是否超过最大数量．因此对于这种需求可以通过一个额外的field来处理：

### 方案１：avaliableCount

可用数量，一开始设置为商品的最大数量，每次购买操作对字段进行减法操作：
因此更新语句是：
假设购买数量为: **m**

```javascript
db.getCollection('product').update(
    {
        '_id': productId, 
        'avaliableCount': {'$gte': m}
    },
    {
        '$set': {'$inc': {'avaliableCount':-m}}
    }
);
```

### 方案２：saledCount

售出数量，一开始置为０，每次购买操作对字段进行加法操作：
因此更新语句是：
假设购买数量为: **m**, 商品的最大数量为：**maxCount**

```javascript
db.getCollection('product').update(
    {
        '_id': productId, 
        'saledCount': {'$lte': maxCount-m}
    },
    {
        '$set': {'$inc': {'saledCount':m}}
    }
);
```

这就要求我们在进行数据库设计时不仅要考虑到页面展示时的需要，还应该考虑各种业务场景时的一些功能需求，有时候一些额外的字段能够给予你极大的帮助．这需要你对系统足够了解，对实现成竹于胸．

`MongoDB`我还有很多需要进一步了解的地方，如回滚操作，不像一些关系型数据库（如`mysql`），有语法上的支持（commit, rollback）. 我所能想到的一些实现方式仅仅为保护现场，对每一步操作记录一个反向的操作，当失败了执行．很好奇数据库通过操作日志进行同步回滚，值得研究一下．

其实里面很多思想都是参考别人的代码得出的，这里需要感谢我的工作伙伴@rexchen给予的帮助
