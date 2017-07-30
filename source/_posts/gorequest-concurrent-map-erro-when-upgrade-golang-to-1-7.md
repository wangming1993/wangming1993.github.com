---
title: golang升级到1.7过程中使用gorequest碰到的问题
date: 2016-08-19 20:52:27
tags:
- golang
---

go在1.6的版本中加入了 `Detection of unsafe concurrent access to maps`, 
考虑下面的代码:

```golang
const workers = 100 // what if we have 1, 2, 25?

var wg sync.WaitGroup
wg.Add(workers)
m := map[int]int{}
for i := 1; i <= workers; i++ {
    go func(i int) {
        for j := 0; j < i; j++ {
            m[i]++
        }
        wg.Done()
    }(i)
}
wg.Wait()
```

<!-- more -->

在运行时会有如下的输出:

```shell
Outputs:
fatal error: concurrent map read and map write
fatal error: concurrent map writes
```

更多详情请参考:  https://talks.golang.org/2016/state-of-go.slide#30

而我们项目使用`gorequest`, 模拟一下我们的使用场景:

```golang
package extension

import (
    "github.com/parnurzeal/gorequest"
    "encoding/json"
    "net/http"
)

var Request *gorequest.SuperAgent = nil

func init() {
    Request = gorequest.New()
}

func (request *gorequest.SuperAgent) PostJson(url string, data interface{}) ([]byte, gorequest.Response) {
    jsonData, err := json.Marshal(data)
    if nil != err {
        panic(err)
    }
    Request.Post(url).Send(string(jsonData))
    resp, body, err := Request.End()

    if err != nil {
        panic(err)
    }

    bytes := ([]byte)(body)
    return bytes, resp
}

```

调用过程:

```golang
package main

import (
    "extension"
    "net/http"
)

func main () {
    body := map[string]string{
        "id":        "1",
        "status":    "success",
        "message":   "Send msg",
    }
    go send("http://www.site1.com", body)
    go send("http://www.site2.com", body)
    go send("http://www.site3.com", body)
}

func send(url string, body interface{}) {
    _, resp := extension.Request.PostJson(url, body)
    if resp == nil || resp.StatusCode != http.StatusOK {
        fmt.Println("Error response code")
    }
}
```

在1.5版本中，有时候会出现 `panic: runtime error: invalid memory address or nil pointer dereference`, 因为在跑test case过程中,有时候能够happy pass, 而且服务一直也没有出什么问题，便一直没有解决.

今天试图将go升级到1.7, 发现了 `fatal error: concurrent map read and map write`， 而且频率很高，便想着找到原因，到底是在哪一步出错的。

通过调试，最终定位定位到了gorequest的`SendString`方法中.

首先从SuperAgent的结构上来分析:

```golang
// A SuperAgent is a object storing all request data for client.
type SuperAgent struct {
    Url        string
    Method     string
    Header     map[string]string
    TargetType string
    ForceType  string
    Data       map[string]interface{}
    FormData   url.Values
    QueryData  url.Values
    Client     *http.Client
    Transport  *http.Transport
    Cookies    []*http.Cookie
    Errors     []error
    BasicAuth  struct{ Username, Password string }
    Debug      bool
    logger     *log.Logger
}
```

注意到**Data**这个属性， 是一个 map, 

```golang
func (s *SuperAgent) SendString(content string) *SuperAgent {
    var val map[string]interface{}
    // check if it is json format
    d := json.NewDecoder(strings.NewReader(content))
    d.UseNumber()
    if err := d.Decode(&val); err == nil {
        for k, v := range val {
            s.Data[k] = v
        }
    } else {
        ...
    }
}
```

当我们使用goroutine去调用时，由于是同一个SuperAgent的实例，在给Data赋值的过程中`s.Data[k] = v`, 就会引起`concurrent map read and map write`的问题。

官方给出的解决方案是在赋值时加锁:

```golang
func count(n int) {
    var wg sync.WaitGroup
    wg.Add(n)
    m := map[int]int{}
    var mu sync.Mutex
    for i := 1; i <= n; i++ {
        go func(i int) {
            for j := 0; j < i; j++ {
                mu.Lock()
                m[i]++
                mu.Unlock()
            }
            wg.Done()
        }(i)
    }
    wg.Wait()
}
```

而且从官方给出的Benchmark results来看, 并不会出现很大的性能损耗。
详情请参考: https://talks.golang.org/2016/state-of-go.slide#31

我们出现concurrent write的原因在于用于同一个SuperAgent实例, 如果我们不将Request作为SuperAgent的单例， 而是每次去new一个，那么就不会存在这个问题.

```golang
//增加一个获取*gorequest.SuperAgent实例的方法，而不让它变成一个单例
func GetRequest () *gorequest.SuperAgent {
    return gorequest.New()
}

//在调用时不再通过extension.Request， 而是:
extension.GetRequest().PostJson(url, body)
```
