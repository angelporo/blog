---
layout: post
title: 简单聊聊web响应式布局
date: 2018-04-25 09:53:45
tags:
    - web前端
    - css
categories: "布局"
---

前几天在'全球蛙'面试的时候面试官问到了兼容分辨率的知识

个人认为回答的不是很好, 虽然拿到了offer(暗喜)
{% asset_img hei.gif %}
平时写js的比较多, 布局相对来说不是很多.倒是没有时间总结这些, 导致谈论起来还需要想想, 于是就有了这篇短文

## 理解响应式设计

响应式网页设计就是一个网站能够兼容多个终端-而不是为每个终端做一个特定的版本
直白的说就是同一个网页在平板, 手机, pc上面显示的效果不同, 但是还同样美观, 让人看着舒服, 用户体验更好

看起来是这样的:
{% asset_img 2222.jpg %}
{% asset_img 333.jpg %}
{% asset_img 444.jpg %}

## 响应式设计的步骤
> 了解了什么是响应式，那么接下来我们就要说说响应式设计的步骤，有人这时候会说“博主，你傻啊，响应式设计的步骤不就是1.编写非响应式代码、2.加工成响应式代码、3.响应式细节处理、4.完成响应式开发吗？”博主菊花一震 原来高手在民间啊，微微一硬表示敬重，我去 ，说错了 是微微一笑，大家不要误会啊。

因为都是文档流的渲染方式, 所以高度不需要管.

1. 前端程序员当拿到设计图之后,切记不能框框框的就写,  首先要打量下相关元素的布局,

2. 对布局有具体方案之后接下来就是写非响应式的部分, 按照常理应该是写pc的,这里就需要考虑使用html中`meta`标签来告诉浏览器一些具体设置, 比如ios手机上面可以放大缩小的功能, 如果不需要就把他禁止:
```
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">
<meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
<meta name="HandheldFriendly" content="true">
```
user-scalable属性能够解决ipad切换横屏之后触摸才能回到具体尺寸的问题。

3. 使用`媒体查询`来设置不同浏览器页面大小的样式, 这样html结构不变得情况就可以达到页面不同的style 

假如浏览器客户端分辨率小于980px
写法:
```css
@media screen and (max-width:980px){
     #head { … }
     #content { … }
     #footer { … }
} /*这里面的样式会覆盖掉之前所定义的样式。*/
```

4. 其他浏览器宽度规定
```
/**ipad**/
@media only screen and (min-width:768px)and(max-width:1024px){}
/**iphone**/
 @media only screen and (width:320px)and (width:768px){}
```

5. 上面只是块的宽度设置,  页面中很重要的字体大小也很关键.
   - rem
   - em
很好理解,  rem相对html的字体大小设置, em相对父元素字体大小设置, 这样就脱离了px这个绝对单位. 这两个属性都是css3才有的
```
html{font-size:16px;}
完成后，你可以定义响应式字体：
@media (min-width:640px){body{font-size:1rem;}}
@media (min-width:960px){body{font-size:1.2rem;}}
@media (min-width:1200px){body{font-size:1.5rem;}}
```



## 图片万能处理方法

想法来源于css3的flex布局,  俗称流式布局, 个人理解, 流及是液态的, 液态可以充满整个容器, 不管容器款到
```css
.container img{
    max-width: 100%;
    height: auto;
}
```
上面代码可以看出图片会根据`.container`的容器的改变来达到填充满`container`这个容器, 高度自动, 可以保证图片比例不扭曲

### 图片响应式属性
`img`标签`srcset`: 是根据屏幕密度实现对应尺寸图片
还有`img`标签中的`sizes`属性
```
<img src="mm-width-128px.jpg" srcset="mm-width-128px.jpg 1x, mm-width-256px.jpg 2x">
```

简写:`<img src="mm-width-128px.jpg" srcset="mm-width-256px.jpg 2x">`

具体详情情况[张鑫旭大声写的日志](http://www.zhangxinxu.com/wordpress/2014/10/responsive-images-srcset-size-w-descriptor/)

背景图:
```css
#log a{
    display:block;
    width:100%;
    height:40px;
    text-indent:55rem;
    background-img:url(logo.png);
    background-repeat:no-repeat;
    background-size:100% 100%;
}
```
background-size是css3的新属性，用于设置背景图片的大小，有两个可选值，第一个值用于指定背景图的width,第2个值用于指定背景图的height,如果只指定一个值，那么另一个值默认为auto。
background-size:cover; 等比扩展图片来填满元素
background-size:contain; 等比缩小图片来适应元素的尺寸

## 手机端布局(只考虑手机端布局, 那就用手指头想也是用flex简单好用)
兼容问题没有的,  手机全部支持css3, 因为它出来的晚. 顺便吐槽下ie浏览器, 还好不是前6年的前端程序员
{% asset_img 111.png %}
