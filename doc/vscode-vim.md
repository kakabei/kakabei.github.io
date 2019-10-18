---
layout: tools
title: vscode 用法总结
date: 2019-10-18 21:00:30
categories: tools
tags: tools
excerpt: 
---

### vscode + vim

1. vscode上安装了vim插件（vscodevim），有一些快键冲突。 可以更改一下配置setting.json。 如下： 

```sh 
    "vim.useSystemClipboard": true,
    "vim.handleKeys": {
        "<C-a>": false,
        "<C-f>": false,
        "<C-x>": false,
        "<C-c>": false,
        "<C-h>": false
    }
```

`useSystemClipboard`  使用系统的粘贴板

`<C-a>` 屏蔽 Ctrl+A
