---
title: Mac OS gem install 问题
date: 2017-07-28 10:15:57
tags:
- MacOS

categories:
- MacOS

---

在使用`gem install travis`时出错：

```shell
ERROR:  You must add /O=Digital Signature Trust Co./CN=DST Root CA X3 to your local trusted store
```

<!-- more -->

解决方案:

- 首先要安装Homebrew终端输入这条命令即可 
```shell
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
- 安装rvm 
```shell
curl -L get.rvm.io | bash -s stable
source ~/.rvm/scripts/rvm
```
- 安装2.3.0版本ruby
```shell
rvm install 2.3.0
rvm use 2.3.0 --default
```

**注意:**

国内使用`gem install`时网络不好，需要更换`gem`的镜像, 原来的[淘宝镜像](https://ruby.taobao.org/)不在维护, 使用[RubyGems 镜像- Ruby China](https://gems.ruby-china.org/)

```shell
> gem sources --add https://gems.ruby-china.org/ --remove https://rubygems.org/
> gem sources -l
https://gems.ruby-china.org
# 确保只有 gems.ruby-china.org
```
