---
title: 一次 Dead Lock 事件的处理
date: 2019-02-25 11:29:51
tags:
- MySQL
---

## 表结构

```mysql
CREATE TABLE `group_works` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `group` varchar(50) NOT NULL,
  `work` varchar(50) NOT NULL,
  `createdAt` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE KEY `idx_group_work` (`group`,`work`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

## 代码逻辑

```go
tran := extension.MySQL.Begin()

err := tran.Exec("DELETE FROM group_works WHERE group= ?", group).Error
    if err != nil {
        tran.Rollback()
        return err
    }
}

for _, work := range works {
    err := tran.Create(&GroupWork{
        Group:      group,
        Work:       work,
        CreatedAt:  time.Now(),
    }).Error
    if err != nil {
        tran.Rollback()
        return err
    }
}

return tran.Commit().Error
```

## 死锁步骤

打开两个 `mysql cli` ， 分别开启事务：

transation A:

```mysql
begin;    # step 1

DELETE FROM group_works WHERE `group` = '1111'; # step 3

INSERT INTO `group_works` (`group`,`work`,`createdAt`) VALUES ('1111','test','2019-02-27 16:31:13.991669');  # step 5

```

transation B:

```mysql
begin; # step 2

DELETE FROM group_works WHERE `group` = '1111';  # step 4

INSERT INTO `group_works` (`group`,`work`,`createdAt`) VALUES ('1111','test','2019-02-27 16:31:13.991669');   # step 6
```

![image](https://user-images.githubusercontent.com/5611286/53490203-14e90400-3ace-11e9-8744-c8abde1c9999.png)

测试发现当 没有 `group` = '1111' 的记录时 去做 `DELETE`, 会发生死锁， 如果存在记录，反而不会死锁。

![image](https://user-images.githubusercontent.com/5611286/53490399-93de3c80-3ace-11e9-9567-99f1e156394f.png)

## 解决方案

在做 `DELETE` 之前先判断是否有记录，当存在记录的时候才执行 `DELETE`

```go
tran := extension.MySQL.Begin()

var total int
tran.Table("group_works").Where("`group` = ?", group).Count(&total)
if total > 0 { // to avoid dead lock when delete no record found, it will add gap lock
    err := tran.Exec("DELETE FROM group_works WHERE `group` = ?", group).Error
    if err != nil {
        tran.Rollback()
      return err
    }
}

for _, work := range works {
    err := tran.Create(&GroupWork{
        Group:      group,
        Work:       work,
        CreatedAt:  time.Now(),
    }).Error
    if err != nil {
        tran.Rollback()
        return err
    }
}

return tran.Commit().Error
```

## 参考

> [何登成的技术博客](http://hedengcheng.com/?p=771)