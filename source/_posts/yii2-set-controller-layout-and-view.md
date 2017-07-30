---
title: Yii2 controller中如何设置layout和view的路径
date: 2017-03-02 20:25:41
tags:
- Yii2
---

在`yii\base\View`中的 `findViewFile($view, $context = null)` 中 存在 **5**种`view`的处理方式:

<!-- more -->

```php
<?php

if (strncmp($view, '@', 1) === 0) {
    // e.g. "@app/views/main"
    $file = Yii::getAlias($view);
} elseif (strncmp($view, '//', 2) === 0) {
    // e.g. "//layouts/main"
    $file = Yii::$app->getViewPath() . DIRECTORY_SEPARATOR . ltrim($view, '/');
} elseif (strncmp($view, '/', 1) === 0) {
    // e.g. "/site/index"
    if (Yii::$app->controller !== null) {
        $file = Yii::$app->controller->module->getViewPath() . DIRECTORY_SEPARATOR . ltrim($view, '/');
    } else {
        throw new InvalidCallException("Unable to locate view file for view '$view': no active controller.");
    }
} elseif ($context instanceof ViewContextInterface) {
    $file = $context->getViewPath() . DIRECTORY_SEPARATOR . $view;
} elseif (($currentViewFile = $this->getViewFile()) !== false) {
    $file = dirname($currentViewFile) . DIRECTORY_SEPARATOR . $view;
} else {
    throw new InvalidCallException("Unable to resolve view file for view '$view': no active view context.");
}
```

在`yii\base\Controller`中的 `findLayoutFile($view)` 中 存在 **3**种`layout`的处理方式:

```php
<?php
if (strncmp($layout, '@', 1) === 0) {
    $file = Yii::getAlias($layout);
} elseif (strncmp($layout, '/', 1) === 0) {
    $file = Yii::$app->getLayoutPath() . DIRECTORY_SEPARATOR . substr($layout, 1);
} else {
    $file = $module->getLayoutPath() . DIRECTORY_SEPARATOR . $layout;
}
```
