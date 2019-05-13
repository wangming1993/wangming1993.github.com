---
title: Go MySQL SELECT @@max_allowed_packet
date: 2019-02-25 11:15:00
tags:
- Go
- MySQL
---

## SELECT @@max_allowed_packet

我们使用的是 阿里云的 RDS, 它提供了很棒的数据库优化服务，可以定期观察，清理一些慢查询。

在 `慢SQL明细` 里面， 发现最近一周有不少的 SQL :

```mysql
SELECT @@max_allowed_packet
```

<!-- more -->

我们使用的是 `Golang` 去连接 `MySQL`,  通过全局搜索 `max_allowed_packet`, 发现在 [github.com/go-sql-driver/mysql](https://github.com/go-sql-driver/mysql/blob/master/driver.go#L146) 中存在如下逻辑：

```go
if mc.cfg.MaxAllowedPacket > 0 {
    mc.maxAllowedPacket = mc.cfg.MaxAllowedPacket
} else {
    // Get max allowed packet size
    maxap, err := mc.getSystemVar("max_allowed_packet")
    if err != nil {
        mc.Close()
        return nil, err
    }
    mc.maxAllowedPacket = stringToInt(maxap) - 1
}
```

```mysql
// Gets the value of the given MySQL System Variable
// The returned byte slice is only valid until the next read
func (mc *mysqlConn) getSystemVar(name string) ([]byte, error) {
    // Send command
    if err := mc.writeCommandPacketStr(comQuery, "SELECT @@"+name); err != nil {
        return nil, err
    }

    // ignored
}
```

根据上面代码发现如果没有配置 `MaxAllowedPacket`, 那么就会向 `MySQL` 发送 查询 `max_allowed_packet`  的命令， 所以只要配置了`MaxAllowedPacket`  就可以避免发送命令.

在 `dsn.go`  的 `parseDSNParams` 方法中：

```go
case "maxAllowedPacket":
    cfg.MaxAllowedPacket, err = strconv.Atoi(value)
    if err != nil {
        return
    }
```

发现只要在 `MySQL` 的 dsn 中配置 `maxAllowedPacket` 这个参数即可：

```go
maxPacketSize := 1<<30 - 1 // 1G, to avoid send query: SELECT @@max_allowed_packet
dsn := fmt.Sprintf("%s:%s@tcp(%s)/%s?parseTime=true&loc=Local&charset=utf8mb4,utf8&timeout=3s&readTimeout=30s&writeTimeout=30s&maxAllowedPacket=%d", user, password, host, db, maxPacketSize)

```