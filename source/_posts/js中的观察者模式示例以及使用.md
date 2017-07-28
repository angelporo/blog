---
title: js中的观察者模式示例以及使用
date: 2017-07-26 15:08:51
tags:
    - javascript
    - 观察者模式
    - es6
---

### 介绍

观察者模式，也叫订阅-发布模式。
顾名思义, 就是订阅一些功能,  然后在合适的实际发布出来,
这个模式听名字感觉非常抽象、难以理解，而且网上有很多混淆视听的解释，会把人搞的晕头转向不知所措。
<!-- more -->
今天用代码来和直白的文字来说一下这个非常抽象,  难以理解, 而且在网上很多混淆视听的理解,

订阅, 就是吧几个函数推入数组中代用,  听明白了,  是数组,
发布, 就是把缓存在数组中的那写函数列表执行.
具体看代码

```javascript
/*
最终, 我们需要的操作的数据应该是:
{
    eventType: [fn, fn, fn...]
}
*/
class Event {
  constructor () {
    this.eventList = {};
  }

  listen (key, fn) {
    if (!this.eventList[key]) {
      // 对象中eventList
      this.eventList[key] = [];
    }
    this.eventList[key].push(fn);
    console.log(this.eventList);
  }

  trigger (key) {
    let fns = this.eventList[key]; //获取到一组数组
    if (!fns || fns.length === 0 ) {
      // 对应key的函数,  反悔
      return false;
    }
    fns.map(n => fns[i]());
  }
}

let test = new EVent();
test.listen('loginSouccess', () => console.log('登出成功, 拿到用户数据'));
test.listen('loginSouccess', () => console.log('再次登录'));
test.trigger('loginSouccess');// 订阅需要触发的事件
```

在listen()之后打印出来的event
```base
[object Object] {
  loginSouccess: [function () {
return window.runnerWindow.proxyConsole.log('登出成功, 拿到用户数据');
}]
}
```
我来简单试一下上面这段代码.
listen方法就是订阅(缓存), tergger方法就是发布(执行)
这就是最简单的一个订阅-发布者模式.

明白这些之后估计你就会想的, 这种模式哪里使用呢? 举个栗子

现在前端领域, 基本全都是单页面, 使用的都是ajax异步请求.
那么 ajax请求有一个很闹心的问题,  就是层级回调, 一个页面, 需要调用三个数据接口

1. login登录接口.
2. 根据登录接口返回的id, 调取头像接口.
3. 更具登录接口返回的id, 调取消息列表接口.

一般情况下:
```javascript
$.ajax({
  url:'http://ajax.login.com',
  dataType:'json',
  success:function(data){
    getAvatar(data.id);
    getMsg(data.id);
    ....
  }
});
```
这样写是没有问题, 就是那天需求改了,  需要添加接口, 需要success会掉中添加一个函数

添加函数能解决还是好的, 有的人直接在success回调里面继续请求ajax这样的代码,

这样, 代码就变成一大坨油煎粑粑, 这种写法叫造粪模式，百分百的造出垃圾来。因为耦合性太大，接口调用都成了拴在一条绳子上的蚂蚱，一扯就是一坨。

如何解耦 ? 就是利用订阅发布模式.
我们可以在getAvatar方法中, 订阅 (listen) login接口,  而一旦login接口走到success回调，我们就发布（trigger）一下即可。

具体代码实现在文章最后, 如果看不懂,  可以回到这里

应为使用的es6语法,  所以直接, 哪里需要哪里`import`就可以愉快的使用了,

### 发布-订阅没问题了,  现在看看取消订阅

直接上代码,  相信感兴趣的朋友, 代码看起来更容易一点,

```javascript
  remove ( key, fn) {
    let fns = this.eventList[key];
    if (!fns) {
      return false;
    }
    if (!fn) {
      // 如果没有回调, 表示取消此key下所有方法
      fns && (fns.length = 0); // 快速清空数组
    }else{
      //取消指定的方法
      // 此时fns只是this.eventList[key]的引用,
      // 实际要删除this.eventList[key]下的方法
      fns.forEach( (n, i) => {
        if (fns[i] == fn) {
          fns.splice(i, 1);
        }
      });
    }
  }
```

我们要做的就是把remove方法放到Event对象中
```javascript
class Login {
  constructor () {
    this.eventList = {};
  }

  listen (key, fn) {
    if (!this.eventList[key]) {
      // 对象中eventList
      this.eventList[key] = [];
    }
    this.eventList[key].push(fn);
  }

  remove ( key, fn) {
    let fns = this.eventList[key];
    if (!fns) {
      return false;
    }
    if (!fn) {
      // 如果没有回调, 表示取消此key下所有方法
      fns && (fns.length = 0); // 快速清空数组
    }else{
      //取消指定的方法
      // 此时fns只是this.eventList[key]的引用,
      // 实际要删除this.eventList[key]下的方法
      console.log('0',this.eventList);
      fns.forEach( (n, i) => {
        if (fns[i] == fn) {
          fns.splice(i, 1);
        }
      });
    }
  }
  trigger (key) {
    console.log(this.eventList);
    let fns = this.eventList[key]; //获取到一组数组
    if (!fns || fns.length === 0 ) {
      // 对应key的函数,  反悔
      return false;
    }
    fns.forEach((n,i) => fns[i]());
  }
}

function loginSouccess () {console.log('登录成功')}
function loginSouccess2 () {console.log('再次登录')}
let test = new Login();
test.listen('loginSouccess', loginSouccess);
test.listen('loginSouccess', loginSouccess2);
test.remove('loginSouccess',loginSouccess2);
test.trigger('loginSouccess');
```

[这里是发布订阅模式的多种是现](https://msdn.microsoft.com/en-us/magazine/hh201955.aspx)

### 代码总汇

<a class="jsbin-embed" href="http://jsbin.com/hoqijiwula/embed?js,console">JS Bin on jsbin.com</a><script src="http://static.jsbin.com/js/embed.min.js?4.0.4"></script>

### 深入理解

这样子看下里观察者模式是不是也没有那么难,  设计模式用的时间就了,  在重构代码的时候脑子里面有这个东西,  自然而然你就写出了容易维护的代码...

还有一种更普遍的订阅发布模式, 就是javascirpt中的事件,  浏览器中的事件系统, 用户在触发事件的时候就就发生了发布然后触发,

需要明确的一点就是: 订阅发布不等于监听, 很多人聊起这个模式，都会不经意的蹦出监听这个字眼，因为观察者么，观察和监听概念上差不多，这对初学者很容易造成理解上的混乱。

监听是啥？那你知道盯梢吧？电视上大侦探破案，都会穿个大衣戴个礼帽，然后紧紧跟在嫌疑人屁股后面，无论嫌疑人吃饭逛街还是上厕所，他都形影不离的跟着他，一直盯着他，直到盯出破绽为止。(这个栗子很形象吧)

用监听的方式实现我们的ajax回调问题，就是用while循环或setInterval函数。

while循环就是设定一个初始状态位，然后通过不断的循环来“监听”login接口的数据到了没有？到了的话，我才跳出循环，这才是监听，具体应该叫“阻塞监听”，因为会让浏览器阻塞在while这里无法动弹。

而setInterval的方式，是每隔一段时间过来瞅两眼，看看犯罪嫌疑人还在不在？这种方式不算实时监听，如果时间间隔太长，犯罪嫌疑人从眼皮溜走的概率非常大。这种方式，我们姑且叫它“时段监听”吧。

监听的方式非常消耗资源，虽然也可以达到目的，但资源开销实在太大,很容易搞死浏览器，所以一般不推荐使用。

监听的实用价值，其实是socket，这个也给大家科普一下。在前端领域属于html5的新式武器，叫websocket。

socket的方式，就是专门开辟一条双向通道(客户端和服务端)来实时获取内容。这才是真实意义上的实时监听。

比如我们做一个页面，内嵌了股票趋势图。这个毫不夸张的说，真的是每秒钟几十万上下啊！setInterval的方式会把你的客户全坑惨的；而while循环的方式也不行，因为页面除了股票图，还有其他内容，你阻塞在股票图上不往下显示了么？

socket的方式还是非常好用的，感兴趣的童鞋可以回去查查使用方法，非常简单。不过因为浏览器兼容问题，支持的还不是很多，故使用场景较低，但这个解决问题的思路大家还是要掌握的。

> [设计模式总的几种简单示例](https://zhuanlan.zhihu.com/p/24980136)
> [原文地址](https://zhuanlan.zhihu.com/p/24453252)
