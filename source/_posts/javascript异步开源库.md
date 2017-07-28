---
title: javascript中异步执行的方法以及异步的开源库介绍
date: 2017-07-28 22:33:59
categories: "javascript"
tags:
    - js异步
    - javascript
---
## 异步执行方法
写前端这么久,  也没有好好的梳理一下异步的一些东西, 今天就蹭着说异步开源库 也顺便简单说一下`javascript`中的异步

```base
A: 嘿，哥们儿，快点！
B: 我要三分钟，你先等着，完了叫你~
A: 好的，记得叫我啊~ 你（C）也等着吧，完了叫你~
C: 嗯！
...
```

由于 `javascript` 的执行环境是单线程,  所以只能一次执行一个task, 所以后面的任务只能干等着.
好处是实现起来想对比较简单, 执行环境想对单纯; 坏处就是如果任务繁重, 那么后面会等候很长时间, 整个程序执行时间就会想对长 这样就会导致浏览器处于假死状态, 为了解决这个问题,  `javascirpt`执行模式分为两种: 同步 or 异步

所以在js中异步执行是非常重要的,  应该在写的时候, 耗时的任务都应该异步执行, 这样可以避免浏览器假死, 和用户体验.

比如我们经常用到的`ajax`

    接下来就说一下js中经常使用异步的几种方法

### 1. 回调函数

这是个最基本的方法就是吧方法通过回调函数来传入函数, 

假设这个有两个方法
```
f1();
f2();
```
f1()是一个比较耗时的任务, 可以重构f1, 吧`f2`当做参数传入`f1`

```javascript
function f1(callback){
　　　　setTimeout(function () {
　　　　　　// f1的任务代码
　　　　　　callback();
　　　　}, 1000);
　　}

f1(f2); //执行代码

```
上面这种方式就是吧同步转变成异步的执行,这种方法比较简单,  也更容易理解,  在开发过程中如果回调函数使用遍地开花, 就会引起回调地狱,  如果不对代码做很好的注释,  后期根本看不懂.

### 2. 事件监听

这种是采用事件驱动, 任务的执行不取决于代码的顺序, 而是取决于事件的触发. 

```javascript
// 这个jQuery的事件写法, 也是浏览器中的事件写法
f1.on('down', f2); // 这里是f1绑定了事件, 回调是f2;
// 就是当f1发生`down`时间之后就执行f2方法
```
接着看监听时间
```javascript
let f1 = () => {
    setTimeout( () => {
        f1.trigger('down'); // 粗发`down`事件, 从而开始执行f2.
    }, 1000);
}
```
这种方法的优点是可以绑定多个事件 , 也比较容易理解,  也有利于实现[模块化](http://www.ruanyifeng.com/blog/2012/10/javascript_module.html)
缺点是整个程序都要变成事件驱动型，运行流程会变得很不清晰。

### 3. 发布 / 订阅
请移驾到这篇, 专门介绍观察者模式 [这里是发布订阅模式, 和事件驱动很相似, 应该说基本一样](http://angely.me/2017/07/26/js%E4%B8%AD%E7%9A%84%E8%A7%82%E5%AF%9F%E8%80%85%E6%A8%A1%E5%BC%8F%E7%A4%BA%E4%BE%8B%E4%BB%A5%E5%8F%8A%E4%BD%BF%E7%94%A8/)(只不过粗发的一个是用户操作自动, 一个是使用代码手动触发)

这个不多说,  放一个`jQuery`一个插件的栗子

```
    jQuery.subscribe("down", f2); // 订阅down事件
```

```
　　function f1(){
　　　　setTimeout(function () {
　　　　　　// f1的任务代码
　　　　　　jQuery.publish("done"); // 发布down事件
　　　　}, 1000);
　　}
```

### 4. Promise 对象

这个比较复杂[先付上mdn地址](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)

{% asset_img promise.png 官方的这个图我认识很容易看懂 %}

想了好久也不知道怎么描述它.(简单点说就是处理异步请求。。我们经常会做些承诺，如果我赢了你就嫁给我，如果输了我就嫁给你之类的诺言。这就是promise的中文含义。一个诺言，一个成功，一个失败。)

不难看出, `promise` 类似一个协议.通过这个协议来操作你的业务逻辑代码,
promise 本质上是分离了异步数据获取和业务逻辑，从而让开发人员能专注于一个事物，而不必同时考虑业务和数据
它的链式调用可以让你的代码更容易阅读,

```javascript
f1().then(f2)
    .catch(f3)
// 这样写异步代码是不是觉得很优雅, 同样也很容易修改业务逻辑
```
修改f1代码
```javascript
　　let f1 = () => {
　　　　var dfd = $.Deferred();
　　　　setTimeout(function () {
　　　　　　// f1的任务代码
　　　　　　dfd.resolve();
　　　　}, 500);
　　　　return dfd.promise;
　　}
```
最终在`javascript`中异步函数使用`Async/Await` [请移驾到这篇](http://angely.me/2017/06/14/%E4%BD%93%E9%AA%8Ces7%E4%B8%AD%E7%9A%84Async-Await%E6%9D%A5%E5%A4%84%E7%90%86%E5%BC%82%E6%AD%A5/)


## 今天介绍3种`javascript`异步开源库

- jQuery Deferred
- Q.js
- Koajs

> 先占坑, 有时间完善文章
