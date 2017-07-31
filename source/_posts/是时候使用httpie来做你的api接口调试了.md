---
title: 是时候使用httpie来做你的api接口调试了
date: 2017-07-26 10:43:54
tag:
    - api
    - http
    - python
---

{% asset_img httpie.png curl和httpie的代码高亮 %}

<!-- more -->

> [首先附上官方文档](https://github.com/jkbrzt/httpie)

## `httpie` 是有`python`写的,  所以,  你电脑里面必须有`python`

请提前get http协议相关知识

至于它是做什么的,  这个不难知道,  百度下就可以了,  我常用他来做api接口的调试

这里安装使用python的包管理器

`pip install httpie`

我电脑是mac,  所以我就使用了`brew install httpie` 最终效果一样

## 这里说一下基本的操作

```
模拟提交表单 -f
http -f POST serverPath.php username=name

显示详细的请求
http -v yhz.me

只显示Header
http -h yhz.me

只显示Body
http -b yhz.me

下载文件
http -d yhz.me

请求删除的方法
http DELETE yhz.me

传递JSON数据请求(默认就是JSON数据请求)
http PUT yhz.me name=nate password=nate_password

如果JSON数据存在不是字符串则用:=分隔，例如
http PUT yhz.me name=nate password=nate_password age:=28 a:=true streets:='["a", "b"]'

模拟Form的Post请求, Content-Type: application/x-www-form-urlencoded; charset=utf-8
http --form POST yhz.me name='nate'
模拟Form的上传, Content-Type: multipart/form-data
http -f POST example.com/jobs name='John Smith' file@~/test.pdf

修改请求头, 使用:分隔
http yhz.me  User-Agent:Yhz/1.0  'Cookie:a=b;b=c'  Referer:http://yhz.me/

认证
http -a username:password yhz.me
http --auth-type=digest -a username:password yhz.me

使用http代理
http --proxy=http:http://192.168.1.100:8060 yhz.me
http --proxy=http:http://user:pass@192.168.1.100:8060 yhz.me
```

基本上可以应付你工作中所有的http请求了,
它比curl返回的数据更有好, 代码高亮很喜欢, 使用起来也简单很多

> [写的更详细的在这里](https://xin053.github.io/2016/08/15/httpie%E4%BA%BA%E6%80%A7%E5%8C%96curl%E5%B7%A5%E5%85%B7%E4%BD%BF%E7%94%A8%E8%AF%A6%E8%A7%A3/)
