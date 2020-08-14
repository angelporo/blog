---
title: 使用autojump一键到达你要得路径
date: 2017-07-29 10:49:39
tags:
    - autojump
    - mac
    - linux
categories: "mac工具"
---

### 安装

安装 `brew install autojump`

因为使用的是`zsh`和`term2` 所以需要在`~/.zshrc`内添加`autojump` 作为插件来使用.

看起来是这样子:

```base
# Which plugins would you like to load? (plugins can be found in ~/.oh-my-zsh/plugins/*)
# Custom plugins may be added to ~/.oh-my-zsh/custom/plugins/
# Example format: plugins=(rails git textmate ruby lighthouse)
# Add wisely, as too many plugins slow down shell startup.
plugins=(git bundler osx)
plugins=(autojump)

```
还需要添加`[[ -s ~/.autojump/etc/profile.d/autojump.zsh ]] && . ~/.autojump/etc/profile.d/autojump.zsh`同样到`~/.zshrc`

## 使用

`j` 是 `autojump` 封装好的命令,  在你 `cd` 之后它就会在它的数据库内写入地址链接,  然后在使用`jc`的时候检索数据库,  到你指定的目录. ( 模糊搜索 )
