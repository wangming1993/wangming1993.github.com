---
title: endless是如何实现graceful restart
date: 2017-02-14 21:12:45
tags:
- Go
---

> - https://grisha.org/blog/2014/06/03/graceful-restart-in-golang/
> - https://github.com/fvbock/endless
> - https://github.com/fvbock/endless/tree/master/examples

## 什么是graceful restart
能够优雅的重启，可以处理未处理完的request, 保证服务不中断

## graceful restart需要做的事
- 重启服务期间可以确保待处理的request能够得到处理（完成或超时）
- 不关闭已经打开的socket连接(重用socket)

<!-- more -->

## endless的实现思路

![image](https://cloud.githubusercontent.com/assets/5611286/22914327/e0cf3e9a-f2aa-11e6-86c7-d8379e425827.png)


- 通过`endlessListener`重载`http.Server.Accept()`, 每接收一下request, `sync.WaitGroup.Add(1)`
- 通过`endlessConn`重载`http.conn.Close()`, 当一个request被处理, `sync.WaitGroup.Done()`
- `endlessServer.Serve()` 通过`sync.WaitGroup.Wait()`等待所有已接收的request处理完毕
- 当接收到`SINGHUP`信号时，`fork`一个子进程，将当前运行的程序重新执行，并通过环境变量`ENDLESS_CONTINUE=1`来告知这是子进程。当检测到是子进程时，通过`syscall.Kill(syscall.Getppid(), syscall.SIGTERM)`来关闭父进程
- 协程`handleSignals`通过`for`循环监听到`SIGINT`或`SIGTERM`信号，shutdown endlessServer, close endLessListener, 此时退出`http.Server.Serve()`的`for`循环，执行`srv.wg.Wait()`
- 等待父进程处理完所有request,或者通过`endlessServer.hammerTime()`, sleep指定时间后，`for`循环调用`WaitGroup.Done()`,强制结束


## 测试

[fvbock/endless](https://github.com/fvbock/endless)提供了[example](https://github.com/fvbock/endless/tree/master/examples), 按照文档所说去一步步执行，可以观察到预料的实验现象。　注意`kill -1 $pid` 是指发送`HUP`信号到`$pid`进程。

kill时的数字对应信号信息如下:
![image](https://cloud.githubusercontent.com/assets/5611286/22914948/b8f28784-f2ae-11e6-9f5c-a760110f0e78.png)

所以很容易理解`kill -9 $pid`强制杀死`$pid`进程其实就是发送`KILL`信号给`$pid`进程。
