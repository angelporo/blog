---
layout: post
title: node使用node-smushit命令行来压缩你的图片
date: 2018-03-12 10:44:34
tags:
    - node
    - 压缩图片
    - smushit
categories: "算法"
---

个人不习惯在线对图片进行压缩, 所以就用命令行来解决这个问题, 这里选择node, 当然还有很多语言来做这件事.


> [github项目地址](https://github.com/colorhook/node-smushit)

安装:

`npm install -g node-smushit`

安装后就可以使用`smushit -h`来查看相关参数


example:
压缩images目录的图片, 如果需要遍历子目录请加`- R`参数
`smushit images`


压缩单个文件,`user-default.png`

`smushit user-default.png`
