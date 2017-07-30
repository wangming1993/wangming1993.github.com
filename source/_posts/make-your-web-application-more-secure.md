---
title: 让你的web应用更安全
date: 2016-03-09 20:39:22
tags:
- cookie
- Yii2
---

# 让你的web应用更安全

- **X-Frame-Options**
- **Cookie of secure and httpOnly**

## 设置X-Frame-Options

> https://developer.mozilla.org/en-US/docs/Web/HTTP/X-Frame-Options

**X-Frame-Options** 主要是为了防止`点击劫持`[(clickjacking)](https://en.wikipedia.org/wiki/Clickjacking)
**点击劫持(clickjacking)**是一种在网页中将恶意代码等隐藏在看似无害的内容（如按钮）之下，并诱使用户点击的手段。
**X-Frame-Options** HTTP 头部字段用来指示传输的资源是否可以被包含在 `<frame>` 或 `<iframe>` 中，服务端可以声明这个策略来确保自己的网页内容不会被嵌入到其他的页面中．
**X-Frame-Options**　具有３个具体的值：
- **DENY**
  表明网页内容不可以被嵌入到任何`frame`中
- **SAMEORIGIN**
  同源策略，声明网页内容可以被同一域下面的`frame`嵌入，但不能被不在同一域下面的`frame`嵌入．
  **同源**： 协议，域名和端口全部相同，即使是ip与域名对应，也认为是不同域下，
  下面说明了具体的情况，与：`http://www.example.com/dir/page.html`对比：

<!-- more -->

| Compared URL | Outcome | Reason |
| :-: | :-: | :-: |
| `http://www.example.com/dir/page2.html` | 同源 | 协议主机端口相同 |
| `http://www.example.com/dir2/other.html` | 同源 | 协议主机端口相同 |
| `http://username:password@www.example.com/dir2/other.html` | 同源 | 协议主机端口相同 |
| `http://www.example.com:81/dir/other.html` | 不同源 | 端口不同 |
| `https://www.example.com/dir/other.html` | 不同源 | 协议不同 |
| `http://en.example.com/dir/other.html` | 不同源 | 主机名不同 |
| `http://example.com/dir/other.html` | 不同源 | 主机不同，必须完全匹配 |
| `http://v2.www.example.com/dir/other.html` | 不同源 | 主机不同，必须完全匹配 |
| `http://www.example.com:80/dir/other.html` | Depends | Port explicit. Depends on implementation in browser |

- **ALLOW-FROM** 
  指定具体的来源

## 服务器配置

### Apache

添加站点配置：

```
Header always append X-Frame-Options SAMEORIGIN
```

### Nginx

添加 `http` , `server` 或者　`location` 配置

```
add_header X-Frame-Options SAMEORIGIN;
```

### IIS

添加到站点的 Web.config

```xml
<system.webServer>
  ...

  <httpProtocol>
    <customHeaders>
      <add name="X-Frame-Options" value="SAMEORIGIN" />
    </customHeaders>
  </httpProtocol>

  ...
</system.webServer>
```

### HAProxy

添加到 `frontend`, `listen`, 或者 `backend` 配置中:

```
rspadd X-Frame-Options:\ SAMEORIGIN
```

## 更安全的Cookie

`Cookie` 是浏览器储存在客户端的小的文本文件，用于客户端与服务器之间互传．
Web服务器通过指定 `http header` 的 `Set-Cookie`, 其结构如下：

```
Set-Cookie: name=value[; expires=date][; domain=domain][; path=path][; secure][; httpOnly]
```

可以看到，`Cookie` 主要包含一下几个字段：
- **name**
- **value**
- **expire**
- **domain**
- **path**
- **secure**
- **httpOnly**

这里主要讲`secure` 和 `httpOnly`

### httpOnly标识

用来告诉浏览器不能通过`JavaScript`的 `document.cookie` 来访问 cookie, 目的是避免 **跨站脚本攻击 (XSS)**

### secure标识

强制应用通过`Https`来传输 Cookie

## PHP Yii2中设置Cookie的 httpOnly 和 secure

### 设置 _csrf

Yii2中的 Cookie  `yii\web\Cookie`默认是 httpOnly的, 

```php
<?php

class Cookie extends \yii\base\Object
{ 
    public $name;
    public $value = '';
    public $domain = '';
    public $expire = 0;
    public $path = '/';
    public $secure = false;
    public $httpOnly = true;
}
```

使用 `<?= Html::csrfMetaTags() ?>` 就会生成一个 `name=_csrf` 的 Cookie,  默认是 httpOnly, 通过注入`request`来改变:

在 `config/main.php` 中

```php
    ...
    'components' => [
        'request' => [
            'csrfCookie' => [
                'httpOnly' => true,
                'secure'   => SECURE_COOKIE,
            ],
        ],
    ...
    ]
    ...
```

注入 `csrfCookie` 的 httpOnly 和 secure 属性值

### 生成secure的 Cookie

```php
$cookies = Yii::$app->response->cookies;
$cookies->add(new Cookie([
    'name' => 'accesstoken', 
    'value' => $accessToken, 
    'expire' => time() + Token::EXPIRE_TIME, 
    'secure' => SECURE_COOKIE
    ])
);
```

注意更新 Cookie 的时候也需要更新 secure 属性

```php
$cookies = Yii::$app->request->cookies;
if (($cookie = $cookies->get('accesstoken')) !== null) {
    $cookie->secure = SECURE_COOKIE;
    // 这里很重要, 不然就会丢失
    $cookie->expire = time() + Token::EXPIRE_TIME;
    Yii::$app->response->cookies->add($cookie);
}
```
