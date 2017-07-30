---
title: Yii2 中关于cookie使用遇到的问题
date: 2017-07-27 22:51:57
tags:
- PHP
- Yii2
- Cookie

categories:
- PHP

---

> 关于 [**Yii2**](https://github.com/yiisoft/yii2)这里我就不介绍，主要讲的是[**cookie**](https://en.wikipedia.org/wiki/HTTP_cookie)的使用及遇到的一些问题

客户端与服务器通信，这是一个`request`与`response`的过程。在`Yii2`中分别有`yii\web\Request.php` 和 `yii\web\Response.php` 来处理。

其中**request**处理`cookie`的逻辑是:

<!-- more -->

```php
<?php
protected function loadCookies()
{
    $cookies = [];
    if ($this->enableCookieValidation) {
        if ($this->cookieValidationKey == '') {
            throw new InvalidConfigException(get_class($this) . '::cookieValidationKey must be configured with a secret key.');
        }
        foreach ($_COOKIE as $name => $value) {
            if (is_string($value) && ($value = Yii::$app->getSecurity()->validateData($value, $this->cookieValidationKey)) !== false) {
                $cookies[$name] = new Cookie([
                    'name' => $name,
                    'value' => @unserialize($value),
                    'expire'=> null
                ]);
            }
        }
    } else {
        foreach ($_COOKIE as $name => $value) {
            $cookies[$name] = new Cookie([
                'name' => $name,
                'value' => $value,
                'expire'=> null
            ]);
        }
    }

    return $cookies;
}
```

这里**request**从`PHP`的全局变量`$_COOKIE`中获取客户端传的`cookie`, 构造`yii\web\Cookie`对象，而**这个对象只有name和value属性赋值了**

**response**中处理`cookie`的逻辑是:

首先通过`Yii::$app->response->cookies` 调用`yii\web\CookieCollection.php`:

```php
<?php

public function getCookies()
{
    if ($this->_cookies === null) {
        $this->_cookies = new CookieCollection;
    }
    return $this->_cookies;
}
```

不论是新增,还是删除，修改`cookie`，都是通过将`cookie`赋值给`Yii::$app->response->cookies`

最后**response**通过`sendCookies()`返回给客户端:

```php
<?php
protected function sendCookies()
{
    if ($this->_cookies === null) {
        return;
    }
    $request = Yii::$app->getRequest();
    if ($request->enableCookieValidation) {
        if ($request->cookieValidationKey == '') {
            throw new InvalidConfigException(get_class($request) . '::cookieValidationKey must be configured with a secret key.');
        }
        $validationKey = $request->cookieValidationKey;
    }
    foreach ($this->getCookies() as $cookie) {
        $value = $cookie->value;
        if ($cookie->expire != 1  && isset($validationKey)) {
            $value = Yii::$app->getSecurity()->hashData(serialize($value), $validationKey);
        }
        setcookie($cookie->name, $value, $cookie->expire, $cookie->path, $cookie->domain, $cookie->secure, $cookie->httpOnly);
    }
    $this->getCookies()->removeAll();
}
```

**默认的我们的`cookie`是写到当前域名下面的**,  对于一般场景这是没问题的。当时对于前后端分离的应用，尤其是前后端域名不一致的场景，就会出现问题。

假定现在有一个应用采用前后端分离的架构，前端域名为: `mike.com`, 后端域名为: `api.mike.com`. 后端使用`cookie` 存储用户名:

```json
{
    "name": "user_name",
    "value": "mike",
    "domain": ".mike.com",
    "path": "/"
}
```

> http://php.net/manual/zh/function.setcookie.php


前台登录，调用API `http://api.mike.com/site/login`, 后台代码:

```php
<?php

public function actionLogin()
{
      // validate username and passwd
      // set cookie

      $cookie = new yii\web\Cookie();
      $cookie->name = 'user_name';
      $cookie->value = 'mike';
      $cookie->domain = 'mike.com';
      // 注意这里的domain不能不填，否则会使用api.mike.com, 这样mike.com就不会拿到cookie
      Yii::$app->response->cookies->add($cookie);
}
```

当前台退出登录，调用API `http://api.mike.com/site/logout` 清除cookie, 后台代码如下:

```php
<?php

public function actionLogout()
{
    Yii::$app->response->cookies->remove('user_name');
}
```

**你以为你清除了cookie user_name, 然而并不是这样**， 你可以通过**chrome**浏览器`F12` 在`Application`中查看当前域名下面存在的`cookie`

 为什么会不会删除呢？ 让我们先看看`reomve`的逻辑:

```php
<?php

public function remove($cookie, $removeFromBrowser = true)
{
    if ($this->readOnly) {
        throw new InvalidCallException('The cookie collection is read only.');
    }
    if ($cookie instanceof Cookie) {
        $cookie->expire = 1;
        $cookie->value = '';
    } else {
        $cookie = new Cookie([
            'name' => $cookie,
            'expire' => 1,
        ]);
    }
    if ($removeFromBrowser) {
        $this->_cookies[$cookie->name] = $cookie;
    } else {
        unset($this->_cookies[$cookie->name]);
    }
}
```

如果传的不是一个`Cookie`对象，那么会构造一个`Cookie`对象，设置`expire`, 添加到response中，服务器收到一个`expire = 1`的`cookie`, 就会从本地删除，之后的请求就不会带这个name的`cookie`了。问题在于**重新构造的cookie没有指定domain, 那么就会用当前域名api.mike.com； 可是login写的cookie domain为mike.com, 虽然name相同，都是user_name, 但是domain不同，浏览器不认为是一个cookie, 所以login写入的cookie不会被删除**,

改进后的代码:

```php
<?php

public function actionLogout()
{
    $cookie = new yii\web\Cookie();
    $cookie->domain = 'mike.com';
    $cookie->name = 'user_name';
    Yii::$app->response->cookies->remove($cookie);
}
```

其实不仅`remove`, 去更新cookie的时候也是会存在这个问题的。

```php
<?php

//错误做法
$cookie = Yii::$app->request->cookies->get('user_name');
$cookie->expire = 60 * 60 * 24;     // 更新cookie过期时间
Yii::$app->response->cookies->add($cookie);

// 这种做法也会存在两个同名的cookie
// 一个domain为: api.mike.com; 一个为: mike.com

//正确做法
$cookie = Yii::$app->request->cookies->get('user_name');
$cookie->expire = 60 * 60 * 24;     // 更新cookie过期时间
$cookie->domain = 'mike.com';
Yii::$app->response->cookies->add($cookie);
```

**需要注意的是Yii2从request里面获取的cookie对象只有name,value属性值，如果domain不是当前域，那么需要重新指定**
