---
title: 禁止Vim生成~和un~后缀的文件
date: 2019-11-24 15:56:21 +0800
categories: [Vim]
tags: [vim]
---
在Windows下使用Vim编辑文件后默认会生成\~和un\~后缀的文件：

![自动生成文件](/assets/images/disable-vim-backup-undo-file/自动生成文件.png)

\~后缀的文件是备份；un\~后缀的文件是操作记录，用于下次打开文件时还能进行撤销操作

禁止生成这两个文件的方法： 在配置文件的`source $VIMRUNTIME/vimrc_example.vim`这一行之后加入以下两行

```
set noundofile
set nobackup
```

![配置文件](/assets/images/disable-vim-backup-undo-file/配置文件.png)

重新启动Vim，再编辑文件后就不会生成这两个文件
