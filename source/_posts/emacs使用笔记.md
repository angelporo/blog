---
title: emacs使用笔记
date: 2017-07-04 09:48:00
tags:
    - emacs
    - emacs API
---

> 记录一些emacs 常用的使用技巧,  老年痴呆的脑子记不住很多东西

- M-x eval-last-sexp 使当前 elisp 配置中光标前的那一条语句立刻生效；
- M-x eval-region 使当前 elisp 配置中选中的 region 中的语句立刻生效；
- M-x eval-buffer 使当前的 buffer 中的设置语句立刻生效；
- M-x load-file ~/.emacs 载入 .emacs 文件，从而使其中的设置生效，要生效其它 elisp 文件只需要把 .emacs 文件名换成其它的即可。

### emacs 编辑时好用的会快捷键

- `C-t` 命令用来交换两个字母的位置
- `C-x C-t`这个命令进行交换2行。
- 使单词首字母大写 `M-c` 来将光标所指向的字母首字母大写
- 使用命令`M-u` 将光标所指向字母及其到单词结尾的字母均至为大写字母。(平时写常量的时候, 直接快捷键让其大写就ok了)

