---
title: 使用travis-ci自动构建你的Github主页
date: 2017-07-29 23:08:41
tags:
- travis-ci
- github pages
- hexo

---

> 之前使用 [**hexo**](https://hexo.io/) 搭建自己的Github pages, 配置了很久的 [**next**](http://theme-next.iissnan.com/) 主题, 写了一段时间的文章，感觉棒棒的。 但是后面换了电脑，忘记备份数据，所有的markdown文件也没有push到Github上，因为这些原因我就没有写过了。

> 最近打算重新拾起写文章记录的习惯，因为只有不断的总结才能更好的理解学习。考虑到之前遇到的问题，加上有过gitlab ci的使用经验，决定使用 [travis-ci](https://travis-ci.org) 自动构建github主页。

<!-- more -->

## travis-ci使用

### travis-ci设置

打开[travis-ci](https://travis-ci.org/)主页，选择右上角的 `Sign in with Github`, 使用你的 Github账号登录，进入你的accounts页面：
![accounts](http://otuvs4s36.bkt.clouddn.com/travis-ci-setting.png)

选择你要使用travis-ci构建的Github repository, 这里我开启的是： `wangming1993.github.com` 
![wangming1993.github.com](http://otuvs4s36.bkt.clouddn.com/wangming1993-github-com.png)

点击右下角的`More options` ![More options](http://otuvs4s36.bkt.clouddn.com/more-options.png)， 进入repository的setting页面： ![setting](http://otuvs4s36.bkt.clouddn.com/setting.png), 做一些设置，如：只有存在`.travis.yml`文件时才会触发自动build。

### travis-ci配置文件

travis-ci自动build依赖于`.travis.yml`文件，文件会配置你的语言环境，版本，branch信息，环境变量，以及`before_install`, `install`, `script`,`after_success` 之类的hook.

这里我先列出我的`travis.yml`文件内容，然后坐进一步介绍：

```yml
language: node_js
node_js: stable
branches:
  only:
  - develop
before_install:
- npm install -g hexo
- npm install -g hexo-cli
install:
- npm install
script:
- hexo clean
- hexo generate
after_success:
  - cd ./public
  - git init
  - git config --global user.name 'wangming1993'
  - git config --global user.email 'wangming19932008@163.com'
  - git add .
  - git commit -m "generate static resources, triggerd by travis ci"
  - git push --force "https://wangming1993:${REPO_TOKEN}@${GH_REF}" master:master
env:
  global:
    - GH_REF: github.com/wangming1993/wangming1993.github.com.git
```

- `language: node_js`
    - 因为使用的hexo是通过npm安装的，这就要求nodejs环境
- `node_js: stable`
    - 这个也很重要，不然低版本会报错
- `branches`
    - only develop, 因为master分支我push的是作为Github pages需要的静态文件的。
- `before_install`,`install`
    - 安装hexo以及运行hexo需要的一些node modules
- `script`
    - 这里指定构建时的shell命令， `hexo clean`清除缓存，`hexo generate`将markdown文件转换为html文件，并使用配置的主题。
- `after_success`
    - 可以用于指定构建步骤成功后的执行动作，如将生成的静态资源push到Github repository
- `env`
    - 用于定于一些环境变量，这些会通过`export`的方式供当前shell引用

这里需要着重介绍一下push文件到Github的过程，我使用:
```shell
git push --force "https://wangming1993:${REPO_TOKEN}@${GH_REF}"
```

- `wangming1993`
    - 这是我的Github ID
- `${REPO_TOKEN}` 
    - Github的Personal access tokens， 因为这个涉及到安全性，不能push到Github,所以这个环境变量我是在travis-ci里面配置的
- `{GH_REF}`
    - 指定你push的Github repository, 配置的`.travis.yml`文件的env中



**那么如何在travis-ci中配置环境变量呢？**

首先明确的是这个环境变量的作用域是在你的具体repository中的，如我的repository是:`wangming1993.github.com`, 那么就需要在这个repository的**setting**里面进行配置：

![env setting](http://otuvs4s36.bkt.clouddn.com/env-setting.png)

而关于Personal access tokens,你需要在你的Github中生成， 访问： https://github.com/settings/tokens/new ， 选择权限 ![scopes](http://otuvs4s36.bkt.clouddn.com/select-scopes.png) （PS: 我是勾选的全部权限). 然后将会生成一个token.
