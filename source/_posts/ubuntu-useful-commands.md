---
title: ubuntu 上的一些常见操作命令
date: 2015-10-27 08:33:26
tags:
- ubuntu
- shell
---

## 远程文件拷贝

### 拷贝远程文件到本地

```
scp remote_user@remote_host:remote_file_path  local_file_path
```

### 拷贝本地文件到远程

```
scp local_file remote_user@remote_host:remote_file
```

<!-- more -->

## 解决 Grunt watch error - Waiting…Fatal error: watch ENOSPC

```
fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf && sudo sysctl -p
```

## 解决git权限问题

```
sudo chown -R $USER:$USER "$(git rev-parse --show-toplevel)/.git"
```

## 文件字符统计

```
cat Member.php  | tr -s '!/ $;:,*.[]{}()=-' '\n' | uniq -c | sort -n
cat Member.php  | sed -e 's/\'/\n/ | uniq -c | sort -n
cat Member.php  | tr -s [:punct:] '\n' | tr -s ' ' '\n' |
cat Member.php  | tr -s [=\'=] '\n' | tr -s ' ;,[](){}$=!*/>.&:\\@?-' '\n' | uniq -c
cat Member.php  | tr -s [:punct:] '\n' | tr -s ' ' '\n' | awk '{for(w=1;w<=NF;w++) print $w}' | sort | uniq -c | sort -nr
cat Member.php  | tr -s [:punct:] '\n' | tr -s ' ：' '\n' | sort | uniq -c | sort -n
```

## 查看端口占用

```shell
lsof -i tcp:26017
```

## 关闭端口

```shell
sudo fuser -k 80/tcp
```

## 用到的docker命令

```shell
# 删除为none的docker iamge
docker rmi `docker images | grep none | sed "s/\(\s\+\)/:/g" | cut -f 3 -d ":"`

# 停止所有的docker容器
docker stop `docker ps -aq`
```

- `shift` 移除第一个元素
- 字符串截取:  ${cmd:0:1}  从第一个字符开始，截取一位 

``` shell
#!/bin/bash

cmd=$1
#Get first char of the first arg
cmd=${cmd:0:1}

#Remove first arg from args
shift

if [ $cmd == 'l' ];then
    local $@
else
    remote $@
fi
```
