---
title: 我的第一个PHP扩展
date: 2016-07-23 20:48:23
tags:
- PHP
---

> 写php有一年了，说实话这门语言入门实在是太简单了，以至于我都不想说我会php(这年头谁学这个不是分分钟的事). 但是任何一门语言，都有着其独特的魅力，如果你还没有发现，只能说你还只是停留在这门语言浅显的使用上(不服不行)。

我觉得PHP的一个魅力之处便在于它的扩展性，如果你是`c`大牛, php中不存在想要的功能，完全可以基于c写一个php的扩展，引入到你的项目中，这感觉不要太好(^_^)。当然现在php的性能问题在php 7 中得到了极大的改善(我也只是道听途说的，😄)。

下面我将会一步步地讲解一个简单的php extension. 希望能起到抛砖引玉的作用(我其实就是怕自己下次忘记)。本文是基于[PHP扩展开发与内核应用](http://www.walu.cc/phpbook/5.1.md)，感谢这些乐于分享的人。
> 我觉得程序世界伟大的地方在于有那么多伟大的人，是他们的开源精神，才造就了如今朝气蓬勃的代码世界。

<!-- more -->

首先介绍一下这个扩展的目录结构：

+ ext_mike
    - config.m4
    - php_mike.c
    - php_mike.h

`config.m4`是作为编译文件，在执行`phpize` 时会通过config.m4中的内容生成一系统编译配置文件，如configure.

```c
PHP_ARG_ENABLE(mike, Whether to enable Mike extension, [ --enable-mike Enable Mike Extension])

if test "$PHP_MIKE" != "no"; then
    PHP_SUBST(MIKE_SHARED_LIBADD)
    PHP_NEW_EXTENSION(mike, php_mike.c, $ext_shared)
fi
```

上面`PHP_ARG_ENABLE`函数有三个参数，第一个参数是我们的扩展名`mike`(注意不用加引号)，第二个参数是当我们运行./configure脚本时显示的内容，最后一个参数则是我们在调用./configure --help时显示的帮助信息。

`PHP_NEW_EXTENSION`函数声明了这个扩展的名称、需要的源文件名、此扩展的编译形式。如果我们的扩展使用了多个文件，便可以将这多个文件名罗列在函数的参数里。最后的$ext_shared参数用来声明这个扩展不是一个静态模块，而是在php运行时动态加载的。

`php_mike.h`  作为标准的c语言写法，定义一些宏，函数

```c
#define PHP_MIKE_EXTENSION "mike"
#define PHP_MIKE_VERSION "1.0"

PHP_FUNCTION(mike);
```

`php_mike.c` 扩展模块的引入，与函数的声明

 ```c
#include <php.h>
#include "php_mike.h"

zend_function_entry mike_functions[] = {
    PHP_FE(mike, NULL)
    {NULL, NULL, NULL}
};

zend_module_entry mike_module_entry = {
    STANDARD_MODULE_HEADER,
    PHP_MIKE_EXTENSION,  //扩展名称
    mike_functions,
    NULL,
    NULL,
    NULL,
    NULL,
    NULL,
    PHP_MIKE_VERSION,
    STANDARD_MODULE_PROPERTIES
};

ZEND_GET_MODULE(mike)

PHP_FUNCTION(mike) {
    php_printf("This is mike's first PHP extension! \n");
}
```

关于其中的一些参数，我还没有去搞清楚，  因为我总是run it first。(这将是我接下来的任务)

## 如何去编译扩展

其实如果你安装过其他的php扩展，如：redis, mongo,那么安装自定义的php 扩展对你来说也是很简单的事。

按照下面的步骤执行即可：

```shell
phpize
./configure
make
make test
sudo make install
```

接下来你重启你的`php-fpm`, 使用 `phpinfo()`就可以看到新加入的扩展`mike`了.
![](http://upload-images.jianshu.io/upload_images/1718261-c82ef691a99401cc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
如果没有找到扩展，可能需要手动添加，在`php.ini`中添加

```
extension="mike.so"
```

接下来你就可以在你的php中使用`mike` extension中的函数了，当然现在还只有一个`mike()`
![](http://upload-images.jianshu.io/upload_images/1718261-1a9c22b9ac6af182.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到输出：

![](http://upload-images.jianshu.io/upload_images/1718261-aa36bd9bcf2dcf89.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

至此一个简单的php extension 完成， 当然这个extension并不具备任何实用价值, 但这确是你走向远方的关键。

> 附加一个有用的shell命令：
rm -rf `ls -al | grep -v -E "(.c$|.h$|config.m4|reco.sh)"`
删除非指定的文件(夹) 
-E 使用正则表达式
-v 反向输出，及输出不匹配的结果
