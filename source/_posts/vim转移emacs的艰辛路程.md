---
title: vim转移emacs的艰辛路程
date: 2017-06-04 17:54:47
tags:
    - vim
    - emacs
categories: emacs
---
# 工欲善其事, 必先利其器

> 大道理不用多讲
<!-- more -->

## vim和emacs相比ide优点缺点(想对而论)

> 使用之前建议把`Caps Lock`和`control`替换

- 100%linux装机率,没有GUI的服务器编辑器首选
- 学习曲线高
- 速度快
- 效率高
- vimscript太蹩脚,
- 使用python写插件,
- 可以帮助乌干达贫困儿童
- 长时间编写, 来回模式切换(个人不喜欢)

而emacs是用LISP和c混合编程的，c编写内核(30%)，LISP接口(70%)

因为vim底层使用c写的, vimscript占到了30%, 所以发挥的作用也只有30%

## vim简单介绍

> Vim是一个类似于Vi的文本编辑器，不过在Vi的基础上增加了很多新的特性，Vim普遍被推崇为类Vi编辑器中最好的一个，事实上真正的劲敌来自Emacs的不同变体。1999年Emacs被选为Linuxworld文本编辑分类的优胜者，Vim屈居第二。但在2000年2月Vim赢得了Slashdot Beanie的最佳开放源代码文本编辑器大奖，又将Emacs推至二线， 总的来看， Vim和Emacs同样都是非常优秀的文本编辑器。Vim(和Vi)一个最大的优势在于它最常用的命令都是简单的字符，这比起使用复杂的控制组合键要快得多，而且也解放了手指的大量工作，学习使用这些命令的时间很快。

## 简单使用

[推荐配置](https://github.com/SpaceVim/SpaceVim)

vim有3中模式,

- `normal`所有快捷键发出指令需要在normal模式下, 除非自己添加`insert`模式配置
- `inster`插入模式, 编写代码
- `visual` 模式, 文本的选择

[懒人,别人写的快捷键博客](http://www.cnblogs.com/Zjmainstay/articles/vim_quickkey.html);
{% asset_img vim_key_help.png vim键位查询 %}

### vim使用`python`来写你的vim插件

> 注意!!! python电脑版本一定要和vim支持的版本相同`:h python` 查看vim支持版本 `:!command` 用来执行base命令

vim执行python命令直接`:python print "这里是输入内容"`

如果在`vimrc`里面使用python来扩展功能.下面是格式

```
py[thon] << {endmarker}
{script}
{endmarker}
```

demo:

```python
function! Foo()
python << EOF
class Foo_demo:
    def __init__(self):
        print 'Foo_demo init'
Foo_demo()
EOF
endfunction
```

想要在vim内部使用上面的函数, 需要在在vim内把需要执行的文件导入, 然后才能使用`:source path_to_script/script_demo.vim` 然后`:call Foo()`来执行

[python扩展vim插件原文](https://segmentfault.com/a/1190000000756107)

## emacs简单介绍

> Emacs，著名的集成开发环境和文本编辑器。Emacs被公认为是最受专业程序员喜爱的代码编辑器之一，另外一个vim。
EMACS，即Editor MACroS（编辑器宏）的缩写，最初由Richard Stallman(理查德·马修·斯托曼)于1975年在MIT协同Guy Steele共同完成。这一创意的灵感来源于TECMAC和TMACS，它们是由Guy Steele、Dave Moon、Richard Greenblatt、Charles Frankston等人编写的宏文本编辑器。
自诞生以来，Emacs演化出了众多分支，其中使用最广泛的两种是：1984年由Richard Stallman发起并由他维护至今的GNU Emacs，以及1991年发起的XEmacs。XEmacs是GNU Emacs的分支，至今仍保持着相当的兼容性。

Emac使用Emacs Lisp，这种有着极强扩展性的编程语言，从而实现了包括编程、编译乃至网络浏览等等功能的扩展

## emacs 使用

### 优缺点

- 效率高
- 社区牛人多
- 速度快
- 学习曲线高
- lisp强大的语言支持
- 不能资助乌干达贫困儿童
- 相比vim不需要切换模式
- 每个buffer可以选用单个mode, 互相不冲突

[推荐使用配置(容易上手些)](https://github.com/syl20bnr/spacemacs)

[个人使用配置](https://github.com/angelporo/emacs.d)

emacs使用介绍就比vim简单多了,
如果你折腾过vim, 使用meacs来说前期会很好受, 但是, emacs的学习曲线越来越难,

emacs只有一种inster模式,相比vim来说, 所以, emacs所有的快捷键只能在插入下来完成, 那么, 快捷键数量就多了要比vim多一点,

具体使用查看手册`C-h k f b`

[常用快捷键,](http://blog.csdn.net/CherylNatsu/article/details/6536959)

内置帮助手册

`Ctrl-h i m emacs`就可以调出详细的Emacs使用手册;
`Ctrl-h i m emacs lisp intro` 可以调出Emacs Lisp入门教程；
`Ctrl-h i m elisp` 可以调出完整的elisp编程手册。

## vim和emacs使用上感受以及不同

在使用vim和emacs之前一直用的`sublime text` 这里说下感受,
相率确实高了不少, 因为sublime底层使用python写的,当时对python也不是很了解,更没有时间来学习另一种语言 而且对自己`sublime`配置不满意, 自己设置了一些快捷键还是没有达到预期的效果, 之后一气之下就卸载了.

最终逼着自己使用vim,最终自己写过一套配置, 可是在看了`github`大神开源的配置后, 想也没想`fork`过来做二次修改, 果断把自己配置删除掉, (看了大神的配置自己的就是一坨`油煎粑粑`)

前车之鉴: 最好使用大神开源配置, 少走弯路

> vim更适合修改网管, emacs才是更适合长时间的编码

所有操作使用快捷键, 可以更好的使自己专注一件事情,而不会被一些繁琐的操作分心
