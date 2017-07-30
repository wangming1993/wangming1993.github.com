---
title: Sublime Text 3自定义插件
date: 2016-03-06 20:35:15
tags:
- sublime text 3
---

> http://docs.sublimetext.info/en/latest/extensibility/plugins.html
> https://clarknikdelpowell.com/blog/creating-sublime-text-3-plugins-part-1/

Sublime Text 3 (**ST3**)　作为我工作之后的唯一编辑器，　已经彻底征服了我的心，它非凡的性能变现以及众多插件的扩展性，已经让我欲罢不能了．每一个独特功能的插件都加速了我的日常工作，这使我萌生了自定义插件的想法．
<!-- more --> 

## 关于插件

**ST3**的插件以文件的形式组织，它会在3个目录搜索插件:
- Installed Packages
- Packages
- Packages/pkg_name/

但是如果`Packages`下文件嵌套太深，插件将不会被load
另外，**ST3** 的插件是已`Python`编写的，这要求你需要知道如何使用`Python3`

## 自定义插件

### 环境

- x86_64 GNU/Linux
- Sublime Text 3
- Python 3+

### 添加插件

首先新建一个插件
1. 点击　`Perferences` -> `Browse Packages...`, 在打开的文件夹下面新建一个目录：`Mike`;
2. 点击　`Tools` -> `New Plugin...`，将文件保存到插件目录 `Mike` 下面，取名为: `mike.py`, 修改文件内容如下：

```python
import sublime, sublime_plugin

class MikeCommand(sublime_plugin.TextCommand):
    def run(self, edit):
        self.view.insert(edit, 0, "Hello, World!")
```

1. 使用　**Ctrl + `** 打开 **ST3** 的命令行，输入：

```python
view.run_command("mike")
```

如果成功在当前文件下看到　`Hello, World!` 表示插件已经添加成功

插件命令的解析规则很简单，　命令名称 + `Command` 构成_class_, 当然这里的命令名称是首字母大写的，但是执行的时候会转化为小写的，例如：
`class MikeWangCommand` 所对应的命令是: `mike_wang`

### 为插件添加快捷键

在插件的目录`Mike`下面新建文件:  `Default (Linux).sublime-keymap`,
表示在Linux下面的快捷键设置，当然也可以为不同平台指定快捷键，具体支持如下:
- Default (Windows).sublime-keymap
- Default (OSX).sublime-keymap
- Default (Linux).sublime-keymap

其内容如下：

```json
[
    {
        "keys": [
            "ctrl+alt+k"
        ],
        "command": "mike"
    }
]
```

文件内容必须是一个json格式的数组，　`keys`表示快捷键的组合方式，　`command`则表示所要执行的命令．

### 为插件添加菜单

**ST3**支持３种类型的菜单：
- Main.sublime-menu
  既所谓的顶部菜单，位于**ST3**菜单栏具体菜单下
- Side Bar.sublime-menu
  侧边栏菜单，当在侧边栏右键时弹出
- Context.sublime-menu
  当在文字区域内右键是弹出的上下文菜单

这里我定义了３中菜单：
**Main.sublime-menu**, 内容如下：

```json
[
  {
    "id": "tools",
    "children": [
      {
        "caption": "Mike Wang",
        "id": "mike",
        "command": "mike"
      }
    ]
  }
]
```

效果为：在菜单栏`Tools`下面有一个 `Mike Wang`的菜单项

**Side Bar.sublime-menu**, 内容如下：

```json
[
  {
    "caption": "Mike Wang",
    "command": "mike"
  }
]
```

效果为：在侧边栏右键是可以看到一个 `Mike Wang`的菜单项

**Context.sublime-menu**, 内容如下：

```json
[
  {
    "id": "mike",
    "command": "mike",
    "caption": "Mike Wang"
  }
]
```

效果为：在文字区域右键有一个 `Mike Wang`的菜单项

很明显，这些文件都是以 json 数组的形式组织的，其内容表现为：
- id 
  每一个菜单项的唯一标识
- caption
  相当于菜单的一个展示名称，所看到的效果
- command
  大小写敏感，所要执行的命令
- children
  用于定义子菜单
## 总结

到这里，一个简单的 **ST3** 插件就定义完毕，这里只是表现了一个插件的组织结构，并不具备什么有用的功能，因为这里并没有结合 **ST3** 的 API . 但千里之行始于足下，具备了完整的结构才可以继续开展壮大起来．

更多技术细节：

> http://www.sublimetext.com/docs/3/api_reference.html
> http://www.sublimetext.com/docs/api-reference
> http://docs.sublimetext.info/en/latest/reference/reference.html
