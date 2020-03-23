---
layout: post
title: js使用jsonp方式解决跨域问题
date: 2018-07-04 16:35:37
tags:
    - jsonp
    - 跨域
categories: "算法"
---

跨域的安全限制都是对浏览器端来说的，服务器端是不存在跨域安全限制的。

浏览器的同源策略限制从一个源加载的文档或脚本与来自另一个源的资源进行交互。

如果协议，端口和主机对于两个页面是相同的，则两个页面具有相同的源，否则就是不同源的。

如果要在js里发起跨域请求，则要进行一些特殊处理了。或者，你可以把请求发到自己的服务端，再通过后台代码发起请求，再将数据返回前端。

这里讲下使用jquery的jsonp如何发起跨域请求及其原理。

 

先看下准备环境：两个端口不一样，构成跨域请求的条件。

获取数据：获取数据的端口为9090

{% asset_img 1.png 跨域1 %}

请求数据：请求数据的端口为8080

{% asset_img 2.png 跨域1 %}

1. **先看下直接发起ajax请求会怎么样**

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width">
  <title>JS Bin</title>
</head>
<body>
<script src="https://code.jquery.com/jquery-2.2.4.js"></script>

<input id="btn" type="button" value="跨域获取数据" />
<textarea id="text" style="width: 400px; height: 100px;"></textarea>
  <script>
  $(document).ready(function () {
            $("#btn").click(function () {
                 $.ajax({
                    url: 'http://www.quanqiuwa.com:3001',
                     type: 'GET',
                     success: function (data) {
                         $(text).val(data);
                     },
                   error: function (xhr) {
                     console.log(xhr)
                   }
                });
            });
         });
  </script>
</body>
</html>
```

请求的结果如下图：可以看到跨域请求因为浏览器的同源策略被拦截了。

{% asset_img 2.png 跨域1 %}

2.**接下来看如何发起跨域请求。解决跨域请求的方式有很多，这里只说一下jquery的jsop方式及其原理。**

> 首先我们需要明白，在页面上直接发起一个跨域的ajax请求是不可以的，但是，在页面上引入不同域上的js脚本却是可以的，就像你可以在自己的页面上使用<img src=""> 标签来随意显示某个域上的图片一样。

比如我在8080端口的页面上请求一个9090端口的图片：可以看到直接通过src跨域请求是可以的。
下面是可以请求回来图片内容的.
{% asset_img 5.png 跨域1 %}

3.**那么看下如何使用`<script src="">`来完成一个跨域请求：**

　　当点击"跨域获取数据"的按钮时，添加一个`<script>`标签，用于发起跨域请求；注意看请求地址后面带了一个`callback=showData`的参数；

　　`showData`即是回调函数名称，传到后台，用于包裹数据。数据返回到前端后，就是`showData(result)`的形式，因为是script脚本，所以自动调用showData函数，而`result`就是`showData`的参数。

　　至此，我们算是跨域把数据请求回来了，但是比较麻烦，需要自己写脚本发起请求，然后写个回调函数处理数据，不是很方便。

```js
function showData (result) {
             var data = JSON.stringify(result); //json对象转成字符串
             $("#text").val(data);
         }
 
         $(document).ready(function () {
    
            $("#btn").click(function () {
                 //向头部输入一个脚本，该脚本发起一个跨域请求
                 $("head").append("<script src='http://www.quanqiuwa.com:3001/update'><\/script>");
             });
 
         });
```

{% asset_img 6.png 跨域1 %}


4.**再来看jquery的jsonp方式跨域请求：**
服务端代码不变，js代码如下：最简单的方式，只需配置一个`dataType:'jsonp'`，就可以发起一个跨域请求。jsonp指定服务器返回的数据类型为jsonp格式，可以看发起的请求路径，自动带了一个`callback=xxx`，`xxx`是`jquery`随机生成的一个回调函数名称。

这里的`success`就跟上面的`showData`一样，如果有`success`函数则默认`success()`作为回调函数。

```
$.ajax({
                     url: "http://localhost:9090/student",
                     type: "GET",
                     dataType: "jsonp", //指定服务器返回的数据类型
                     success: function (data) {
                         var result = JSON.stringify(data); //json对象转成字符串
                         $("#text").val(result);
                     }
                 });
```

懒得copy了,
> [原文](https://www.cnblogs.com/chiangchou/p/jsonp.html)
