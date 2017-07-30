---
title: 识别并替换一段文件中的url地址
date: 2015-10-27 08:36:43
tags:
- coffeescript
---

## 需求

需要在textarea中将一段文字中的url形式的地址以超链接的形式展现, 　类似于微信, QQ 中的自动链接识别

## 解决方案

```cofferscript
  chat.wrapLink = (body) ->
    replacedBody = body
    url = /(ftp|http|https):\/\/([\w-]+\.)+(\w+)(:[0-9]+)?(\/(\w)*)*(\/|([\w#!:.?+=&%@!\-\/]+)?|\/([\w#!:.? +=&%@!\-\/]+))?/
    result = body.match url
    @log 'result', result
    if result
      match = result[0]
      replacedBody = '<a target="_BLANK" href="' + match + '">' + match + '</a>'
      replacedBody = body.replace url, replacedBody
    replacedBody
```

<!-- more -->

主要是 `url` 的正则定义:

1. (ftp|http|https) 表示这三种协议中的一种
2. :\/\/  表示的是 ://
3.  ([\w-]+.)++(\w+) 表示的是host
4. (:[0-9]+)? 表示端口
5. (\/(\w)_)_ 表示请求的路径
6. (\/|([\w#!:.?+=&%@!-\/]+)?|\/([\w#!:.? +=&%@!-\/]+))? 表示的是请求参数，段地址(#)
