---
title: 'emacs使用company-mode,ycmd,hippie,来完善你的补全功能,以及遇到的坑'
date: 2017-06-16 10:57:50
tags:
    - emacs
    - emacs自动补全
    - ycmd
    - company-mode
    - hippie
categories: "emacs使用记"
comments: false
---

{% asset_img emacs-ycmd.png my emacs image %}

<!-- more -->
> 先说一下使用emacs的历程

之前使用vim, 配置github上面有很多, 可以自己配置,  无非就是一些插件, 然后改自己习惯的按键.
但是个人对vim还是不习惯,  别以为用了一年的时间,  也许还是因为自己没有真的用好它,  不过在工作中已经很方便了,  唯一不习惯的是模式之间的切换
毕竟是一个程序员, 不是做的运维, 因为要不停的写写写,  但是vim好多功能是在normal模式下,  所以需要不停的切换, 很不方便, 于是, 在一个夜黑风高的晚上, 卸载vim安装emacs, 因为用的mac折腾起来还是不费力的, 送上现在emacs的配置[fork的purcell大神的配置(其中添加了自己的layer和配置, 在lisp/init-local.el)](https://github.com/angelporo/emacs.d)


之前使用vim的时候会自己去写配置, 然后添加必要的插件,  去写一些适合自己的按键, 折腾一段时间后,
你会发现, 自己写的完全没有必要, 因为`github`上面有开源的, 写好的配置, 而且还是好几百人或是几千人维护的配置文件,你敢说你自己配置的能和大神配置的好么? 所以, 把那些折腾的时间用来专心写的你的项目或是研究自己喜欢的东西, 这个只是工具. 当然, 现在这么说只是因为爱过.

对于emacs补全, 我用的ycmd和hippie配合company-mode的方式来, ycmd后端用的tern

## 框框框的安装
前提自行安装git, python和以来包 , mac `birew install git build-essential cmake python-dev`


`npm install tern -g`

### 安装ycmd server

 `git clone https://github.com/Valloric/ycmd`
 
 下载完整的submodule(时间比较久)
 `git submodule update --init --recursive`
 
 下载完后进入`ycmd`文件夹编译
 
 ```base
 $ cd ~/ycmd
 $ ./build.py --tern-completer
 ```
 
### 2. 安装 ycmd client，还有必要的工具

打开Emacs , 命令`M-x list-packages`分别安装 `ycmd, company-ycmd, flycheck-ycmd company-mode`


### 配置

```elisp
;;设置ycmd补全插件
(set-variable 'ycmd-server-command '("python" "/Users/angel/ycmd/ycmd"))
(setq company-tooltip-limit 8)
(setq company-idle-delay 0.2)
(setq company-echo-delay 0)
(setq company-begin-commands '(self-insert-command))
(setq company-require-match nil)
(company-ycmd-setup)
(add-hook 'after-init-hook 'global-company-mode)
```

讲上面代码添加到配置文件中...

	注意上面路径需要绝对路径不能使用`~/ycmd/ycmd`

### 添加`hippie-extend`

hippie是一个提示先后顺序设置的插件,所以, 需要你自己的需求来配置

安装: `package-install hippie-extend`

```elisp
(global-set-key (kbd "M-/") 'hippie-expand)

(setq hippie-expand-try-functions-list
      '(try-expand-debbrev
        try-expand-debbrev-all-buffers
        try-expand-debbrev-from-kill
        try-complete-file-name-partially
        try-complete-file-name
        try-expand-all-abbrevs
        try-expand-list
        try-expand-line
        try-complete-lisp-symbol-partially
        try-complete-lisp-symbol))
```
上面是我的配置,如果不习惯, 可以自己配置

更换上面`hippie-expand-try-functions-list` 参数就可以

```elisp
try-expand-dabbrev                 ; 搜索当前 buffer
try-expand-dabbrev-visible         ; 搜索当前可见窗口
try-expand-dabbrev-all-buffers     ; 搜索所有 buffer
try-expand-dabbrev-from-kill       ; 从 kill-ring 中搜索
try-complete-file-name-partially   ; 文件名部分匹配
try-complete-file-name             ; 文件名匹配
try-expand-all-abbrevs             ; 匹配所有缩写词
try-expand-list                    ; 补全一个列表
try-expand-line                    ; 补全当前行
try-complete-lisp-symbol-partially ; 部分补全 elisp symbol
try-complete-lisp-symbol           ; 补全 lisp symbol
```


## 遇到的问题处理对策
> tern是一个补全的后端, 是需要配合emacs的前端来实现的. tern安装之后自己会启动, 如不能自动, 请手动启动

- tern 启动之后会在项目目录下创建一个 `.tern-port`文件, 里面的端口和emacs中
`M-x describe-variabble tern-known-port` 看到的一致.

- `tern-mode`是否启动?
  `M-x describe-variabel tern-mode`结果应该是t. (这一步我的meacs中tern启动后 没有出任何结果, 好奇怪)

```base
$ cd /path/to/project
$ tern
Listening on port 63935
```

记住这个端口号，回到 Emacs，手动 `M-x tern-use-server RET 63953` 第二个server可以不填 默认是`127.0.0.1`
当然最终还是要解决配置问题, 想tern自动启动.

把下面代码放到配置文件中就可以自启动了.

```elisp
(add-hook 'js-mode-hook
          '(lambda ()
             (company-mode 1)
             (tern-mode 1)
             (setq company-tooltip-align-annotations t)
             (add-to-list 'company-backends 'company-tern)))
```

如果不出现js补全dom的提示 可能是因为跟目录没有 `.tern-project`文件
然后添加下面代码到`~/.tern-project`

```json
{
  "libs": [
    "browser",
    "jquery"
  ],
  "plugins": {
    "node": {}
  }
}
```

所以即使项目目录没有任何配置,  根目录`.tern-project`也会起作用!
