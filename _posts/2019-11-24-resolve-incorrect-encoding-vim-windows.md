---
title: 解决Windows下Vim中文乱码
date: 2019-11-24 15:53:52 +0800
categories: [Vim]
tags: [vim, encoding]
---
在Windows下使用Vim打开文件时中文字符会显示为乱码

![中文乱码](/assets/images/resolve-incorrect-encoding-vim-windows/中文乱码.png)

解决方法： 在配置文件开始处加入以下三行

```
set encoding=utf-8
set fileencodings=utf-8,ucs-bom,cp936,big5
set fileencoding=utf-8
```

Vim配置文件位置：{Vim安装目录}\\_vimrc

![配置文件位置](/assets/images/resolve-incorrect-encoding-vim-windows/配置文件位置.png)

或在gVim中选择菜单“编辑”-“启动设定”

![配置文件](/assets/images/resolve-incorrect-encoding-vim-windows/配置文件.png)

重新启动Vim，中文字符即可正确显示

![正确显示中文](/assets/images/resolve-incorrect-encoding-vim-windows/正确显示中文.png)
