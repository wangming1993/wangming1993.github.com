---
title: Yii2编码规范
date: 2015-10-28 08:38:18
tags:
- PHP
- coding style
- Yii2
---

**遵循PSR-2编码规范**
## 概览
1. 文件必须使用　`<?php` 或者　`<?=`　标签
2. 文件的结尾必须是一个空行
3. 文件编码格式必须是**不带BOM的UTF-8格式**
4. 必须使用4个空格作为缩进, 不使用`tab`键
5. 类命名规则采用**驼峰命名法**　（Pascal 命名法，　首字母大写）
6. 常量命名全部字母大写，单词间以**下划线**作为分割
7. 方法名采用**小驼峰式命名法**, 除第一个单词首字母小写, 其余单词首字母大写
8. 属性名也采用**小驼峰式命名法**
9. 私用属性以**下划线**开头
10. 总是使用`elseif`而不是`else if`

<!-- more -->

## 文件

### PHP标签

- 文件必须使用　`<?php` 或者　`<?=`　标签
- 对于纯php文件, 可以省略结束标签`?>`
- 结束行不要添加空格
- 任何包含php代码的文件应该使用`.php`作为扩展名

### 字符编码

- 文件编码格式必须是**不带BOM的UTF-8格式**

### 类名

- 类命名规则采用**驼峰命名法**　（Pascal 命名法，　首字母大写）

### 类(包含接口)

- 类命名规则采用**驼峰命名法**　（Pascal 命名法，　首字母大写）
- `{` 应该写在类名下面
- 每一个类必须有一个符合`PHPDoc`的文档块
- 类中的所有代码必须缩进一个制表符
- 一个PHP文件中应该只有一个类
- 所有的类应该注明命名空间
- 类名应该匹配文件名，类的命名空间应该匹配目录结构

#### 常量

- 常量命名全部字母大写，单词间以**下划线**作为分割

#### 属性

- 公有的类成员需要使用`public`指明
- `public`或者`protected` 的变量应该在方法之前声明
- 类中变量的声明顺序应该按照　`public` -> `protected` -> `private`
- 属性声明间不要空行, 属性和方法之间以两个空行分隔
- 属性名也采用**小驼峰式命名法**
- 私用属性以**下划线**开头
- 使用有描述性的名称

#### 方法

- 方法名采用**小驼峰式命名法**, 除第一个单词首字母小写, 其余单词首字母大写
- 方法名应该能够描述方法的意图作用
- 方法声明应该使用`public`,`protected`,`private`修饰符
- `{`应该在方法声明行之下

#### PHP Doc

- 声明类型应该为: `@param` `@var` `@property` `@return` `boolean` `integer` `string` `array` `null`

#### 构造器

- 应该使用 `__construct`

## PHP

### 类型

- 所有的`php`类型都应该使用小写

### 字符串

- 不包含单引号或变量的字符串应该使用**单引号**
- **双引号**作为变量替换时引用
- 使用**点**来进行字符串的拼接
- 长串应该分多行格式化

### 数组

使用 PHP5.4的简短数组语法

#### 数字索引

- 不要使用负数作为索引

#### 关联索引

- 采取每行一个键值对的形式

### 控制语句

- 控制语句的条件的括号前后需要有一个空格

``` php
if ($event === null) {
    return new Event();
}
```

- 避免在`return` 后面使用 `else`

``` php
#不推荐
double getPayAmount() {
  double result;
  if (_isDead) result = deadAmount();
  else {
    if (_isSeparated) result = separatedAmount();
    else {
      if (_isRetired) result = retiredAmount();
      else result = normalPayAmount();
    };
  }
  return result;
};  
```

``` php
#推荐
double getPayAmount() {
  if (_isDead) return deadAmount();
  if (_isSeparated) return separatedAmount();
  if (_isRetired) return retiredAmount();
  return normalPayAmount();
};  
```

### 匿名函数声明

- `function/use` 之间需要空格
