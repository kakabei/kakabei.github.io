---
layout: tools
title: vscode 用法总结
date: 2019-10-18 21:00:30
categories: tools
tags: tools
excerpt: 
---

### vscode + vim

1. vscode上安装了vim插件（vscodevim），有一些快键冲突。 可以更改一下配置setting.json。（用起来好别扭） 如下： 

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

vscode setting.json 的一些配置

```json
{
    "editor.minimap.enabled": false,
    "C_Cpp.updateChannel": "Insiders",
    "[cpp]": {
        "editor.quickSuggestions": true
    },
    "[c]": {
        "editor.quickSuggestions": true
    },
    
    "gnuGlobal.globalExecutable": "d:\\tools\\glo582wb\\bin\\global.exe",
    "gnuGlobal.gtagsExecutable": "d:\\tools\\glo582wb\\bin\\gtags.exe",
    "gnuGlobal.encoding": "Big5",
    "workbench.activityBar.visible": true, 
    "workbench.colorCustomizations": {
        //设置用户选中代码段的颜色 
        //"editor.selectionBackground": "#2f00ff",
        //搜索匹配的背景色
        "editor.findMatchBackground": "#ff0000",
        "editor.findMatchHighlightBackground": "#EE6A50",
        "editor.findRangeHighlightBackground": "#ff9900"

    },

    "php.validate.enable": true,
    "php.validate.executablePath": "d:\\tools\\php-7.3.7-Win32-VC15-x64\\php.exe",
    "php.executablePath": "d:\\tools\\php-7.3.7-Win32-VC15-x64\\php.exe",
    "php.validate.run": "onType",
    "window.zoomLevel": 0,
    

    "python.linting.flake8Enabled": true,
    "python.formatting.provider": "yapf",
    "python.linting.flake8Args": ["--max-line-length=248"],
    "python.linting.pylintEnabled": false,

    "vim.useSystemClipboard": true,
    "vim.handleKeys": {
        "<C-a>": false,
        "<C-f>": false,
        "<C-x>": false,
        "<C-c>": false,
        "<C-h>": false
    }

    
}

```

### Better Comments 

让代码的注释更加人性化。

* ! 红色
* ? 蓝色
* TODO 橙色
   
![](/assets/vscode/better-comments.png)  

### vscode ssh 连接linux

vscode　有一些remote-ssh的插件可以用，但其实感觉最方便的是自己装一个ssh客户端，然后在Terminal用ssh连接就可以了。 如果想方便管理ip地址，可自己写cmd脚本。

### vscode 的一些快捷键的运用

