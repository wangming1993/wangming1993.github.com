---
title: 使用maven开发你的java 项目
date: 2015-10-25 14:30:10
tags:
- maven
---

# java 开发配置maven作为项目管理

## maven 下载

1. 下载 [**_maven**_](http://mirrors.hust.edu.cn/apache/maven/maven-3/)
2. 解压`maven` 到任意目录
3. 将`maven`配置到　`PATH` 中

``` shell
sudo vi /etc/profile
export MAVEN_HOME=/home/user/maven3
export PATH=$MAVEN_HOME/bin:$PATH
```

<!-- more -->

1. 使修改生效

```
source /etc/profile
```

1. 验证`maven`是否安装成功

```
 mvn --help
```

## `eclipse` 配置`maven`

### 安装

1.    `help`   ---->  `Eclipse Marketplace`  
2.   在　`Find` 中输入　_maven_
3.   选择 `Maven Intergation for Eclipse(Luna)`
4.   重启eclipse

### 配置

1. `Preference`  ----> `Maven` 
2. 选择 `Installations`  ---->  `Add` maven unpacked in your path 
3. 选择 `User  Settings` ----> `Browers` the maven settings.xml in your maven path
