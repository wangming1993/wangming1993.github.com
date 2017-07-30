---
title: go code review comments 【译文
date: 2016-12-05 21:07:14
tags:
- golang
- coding style
---

> 翻译自：https://github.com/golang/go/wiki/CodeReviewComments

## 注释

- 注释应该是一段完整的语句
- 注释应该以所描述内容的名字开头，并且以句号结尾

## 声明空的切片

应该使用： `var t []string`,  而不是： `t := []string{}`
前者会避免内存分配，除非使用了`append()`

## 不要使用`panic`
对于普通的错误处理，不要使用`panic`,使用error和多返回值， 
- 参考：https://golang.org/doc/effective_go.html#errors

<!-- more -->

## 错误字符串

- 错误字符串不应该大写(除非是专有名词或者缩写)
- 不要以符号结尾

## 错误处理

不要使用`_`去丢弃error.  当一个函数返回error,去检查并处理error,或者在真正的异常情形下,`panic`

## `import`

将多个`import`以组划分，用空行来区分组，标准包放在最上面
> 可以使用[goimports](https://godoc.org/golang.org/x/tools/cmd/goimports)来格式化

## import dot

除非在test 文件中有循环依赖而去使用`import .` 这种形式，否则不要在你的程序中去使用，
它使你的程序难以阅读，因为你很难清楚的知道它所处的层级关系。

## 缩进错误

尽量保持正常代码最小的缩进，缩进错误处理代码并且优先处理。
**尽量采取**：

```go
if err != nil {
    // error handling
    return // or continue, etc.
}
// normal code
```

**不要**：

```go
if err != nil {
    // error handling
} else {
    // normal code
}
```

## 缩写

缩略应该保持一致，例如： `url`/`URL`, 而不是`Url`, 这个规则同样适用于当`ID`作为一个标识的时候，使用`appID`而不是`appId`

## 包名

所有对包内的引用都应该使用包名去访问，因此包内的名称引用可以去掉包名这个标识。
例如：包`chubby`, 不需要使用`ChubbyFile`, 使用者调用方式为:`chubby.ChubbyFil`,
**而是**使用`File`，使用者调用形式为:`chubby.File`

## 接收者类型

当不知如何抉择值接收还是指针接收时，使用指针接收。但有时值接收是有意义的，尤其是效率因素，对于不常变的小的结构体，基础类型的值。
下面是一些有用的指导：

- 如果receiver是`map`,`func`,`chan`，不使用指针
- 如果receiver是`slice`,当方法不会重组或重新分配切片，不使用指针
- 如果方法需要改变receiver,必须使用指针
- 当receiver是包含锁或同步字段时，必须使用指针以避免复制
- 对于大的结构体或数组，指针更加的高效
- 当外面的改动必须影响到原始的receiver时，必须使用指针
- 最后，如果怀疑，那么请使用指针

## 变量名称

在`go`中变量名应该尽可能的短，尤其是有作用域的局部变量。
**基本原则**:
- 变量越是远离声明，名称越要具有描述性
- 全局变量或不常见的应该使用描述性的名称
- 对于方法的接收者，一两个字母就足够了
- 常见的变量可以使用单个字符，如循环次数i
