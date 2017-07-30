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
