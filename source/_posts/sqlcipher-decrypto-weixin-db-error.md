---
title: sqlcipher 解密微信 db 错误
date: 2018-12-13 11:08:02
tags:
- sqlcipher
---

在使用  `sqlcipher` 解密微信 db 时， 出错

```shell
sqlcipher error file is not a database
```

原因是  `sqlcipher` 的版本和 加密的微信 db 版本不兼容

因为 我通过 `brew install sqlcipher` 安装的是 最新的  `v4.0.0`
解决方案是 安装 `v3.x.x` 版本的， 于是通过源代码的方式生成

<!-- more -->

步骤如下：

```shell
git clone https://github.com/sqlcipher/sqlcipher

cd sqlcipher

# v3.4.2 是一个 tag, 表示一个版本
git checkout v3.4.2


## 这里的步骤主要是为了解决 编译的时候openssl 的问题
## sqlite3.c:18280:10: fatal error: 'openssl/rand.h' file not found
brew uninstall --ignore-dependencies openssl
brew uninstall --force openssl
brew cleanup  -s openssl
brew prune
brew install openssl


./configure --enable-tempstore=yes CFLAGS="-DSQLITE_HAS_CODEC" LDFLAGS="-lcrypto" CPPFLAGS="-I/usr/local/opt/openssl/include"

make
```

然后设置 `sqlcipher` alias
因为我使用的是 `zsh`,
所以执行如下步骤：

```shell
# 在 ~/.zshrc 中添加如下内容，其中 /Users/wangming/Desktop/sqlcipher 为 git clone 之后的路径
# 注意一旦 make 之后，移动了 sqlcipher 的路径需要重新编译(make)
alias sqlcipher="/Users/wangming/Desktop/sqlcipher/sqlcipher"

source ~/.zshrc
```
