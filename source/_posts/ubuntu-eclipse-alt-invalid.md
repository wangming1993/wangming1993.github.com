---
title: ubuntu 下 eclipse 的代码提示失效
date: 2015-10-25 14:28:10
tags:
- maven
---

### 前提条件:

1.  x86_64 GNU/Linux
2. JDK 1.8
3. Eclipse luna

### 出现问题

使用　`ALT+/` 不出现代码提示

### 解决方案
1. _（eclipse）window --> preferences --> General --> keys或者直接在preferences中输入keys，把“word completion”所对应的快捷解（alt + /）去掉（选择需要改变的快捷键行，在binding中用backspace删除）。_
2. _找到"content Assist"，在binding中按住alt，再按/（alt + /）就可以了。_

<!-- more -->
