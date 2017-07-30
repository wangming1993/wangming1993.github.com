---
title: Yii2 请求工作流
date: 2016-02-26 20:23:50
tags:
- Yii2
---

# Yii2 请求处理工作流

1. 用户的请求发送到入口脚本:
   `web/index.php`
2. 入口脚本加载配置,创建应用实例
3. `request`应用组件分析处理路由
4. 创建`controller`实例
5. 创建`action`实例 
6. 过滤器验证
7. `action` 加载数据 `model`
8. `action` 渲染 `view`
9. `response`应用组件负责将渲染的结果发送到用户浏览器

<!-- more -->

具体的工作流(workflow)如下:
![](http://www.yiiframework.com/doc-2.0/images/request-lifecycle.png)
