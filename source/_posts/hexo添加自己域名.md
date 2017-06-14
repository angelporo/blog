---
title: hexo添加自己域名
date: 2017-06-12 10:09:35
categories: "hexo"
tags:
    - hexo
    - next主题
comments: false
---

### 折腾由来:
> 前几天搭建了自己的git pages 博客,  虽然没有什么技术含量, 但是源于人的折腾精神, 不由的就在里面写东西.
> 进去blog必须要通过github进入, 太麻烦, 就在万网上面找好点的域名, blog当然是.me结尾的合适一点, 然后选中了[angely.me](angely.me)恩, 这样对我自身形象很贴(zi)切(lian), 想也没有想直接拿下, 首年13RMB

<!-- more -->

## 然后解析域名, 地址`userName.github.io`, 通过CNAME方式解析, 看起来是这样的
{% asset_img admin.png 解析之后图解 %}

## 自己域名有了, 开始配置

直接修改config

```coffeescript
# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://angely.me/
root: /
permalink: :year/:month/:day/:title/
permalink_defaults:
```
root字段配置不正确会导致blog静态文件download 404



> 给项目 github pages 添加CNAME文件, 内容就是自己域名,`angely.me`
> 因为hexo内容都会把source文件编译到public, so 直接给source文件夹添加CNAME文件然后`hexo d -g`到github项目内

```base
➜ touch CNAME
➜ vim CNAME
➜ ga .
➜ gcm '提交commint 备注'
➜ git push origin master
```

ga -> git add
gcm -> git commint -m
