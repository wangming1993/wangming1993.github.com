---
title: go有用的使用方式
date: 2016-09-08 20:56:49
tags:
- golang
---

> - http://lib.csdn.net/base/go/structure   go知识总结
> - http://www.hellogcc.org/effective_go.html

- **查看函数的方法名**

``` go
import (
    "reflect"
    "runtime"
)
runtime.FuncForPC(reflect.ValueOf(md.Handler).Pointer()).Name()
```

<!-- more -->

- **查看接口的具体实现**

``` go
import (
    "reflect"
)
reflect.TypeOf(i)
```

- **关于defer**
  
  defer采用的是**LIFO**,即栈的实现方式

- **数组，切片, Map的区别**
  - 数组是值,值传递而不是引用传递
  - 切片持有对底层数组的引用,引用传递
  - map持有对底层数据库的引用,引用传递
  - map的key可能会按照任意顺序被输出
- **方法调用**
  - 值方法可以在指针和值上进行调用
  - 指针方法只能在指针上调用
  
## 获取goroot, gopath, version

``` go
import (
      "os"
      "runtime"
)
fmt.Println(os.Getenv("GOPATH"))
fmt.Println(os.Environ())
fmt.Println(runtime.GOARCH, runtime.GOOS, runtime.GOROOT(), runtime.Version())

```

