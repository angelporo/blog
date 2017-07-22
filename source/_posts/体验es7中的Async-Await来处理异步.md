---
title: 体验es7中的Async / Await来处理异步
date: 2017-06-14 19:21:56
tags:
    - es7
    - javascript
categories: 'javascript'
comments: false
---


## 体验下es6对异步的最终解决方案
> 阅读之前, 希望对promise和es6(ECMA2015)了解, 会更容易理解

### 直接上例子

Async/Await 应该是目前最简单的异步方案
下面这个例子就是要实现一个暂停功能, 输入N毫秒, 则停顿N毫秒后才继续执行的例子

<!-- more -->

```javascript
let sleep = function (time) {
    return new Promise(function (resolve, reject) {
        setTimeout(function () {
            resolve()
        }, time);
    })
}

let start = async function () {
    //这里使用起来像同步代码那样直观
    console.log('start');
    await sleep(3000);
    console.log('end')
};

start();
```

控制台先输出 `start`, 等候3秒后, 输出了 `end` .

## 基本规则

1. async 表示 异步函数,  await只能用在异步函数里面.

2. `await` 表示在这里 `等待promise返回结果` , 在继续执行下面的.

3. `await` 后面跟着的 `应该是一个promise对象` (当然, 反悔其他的也没有关系, 只是会立即执行, 不过那样子就没有异步的意义了)

## 获得返回值

`await`等待的虽然是promise对象, 但不必写 `.then(function() {...})`, 直接就可以得到返回值.

```javascript

let sleep = function (time) {
    return new Promise(function (resolve, reject) {
        setTimeout(function () {
            //  return 'ok'
            resolve('ok');
        }, time);
    })
}

let start = async function () {
    let result = await sleep(3000);
    console.log(result); // 'ok'
}
```

上面是一些正确的处理逻辑
下面接着看怎么捕捉错误

## 捕捉错误

既然 `.then(function () {...})`不写了,  那么 `.catch(function () {...})`也需要写,
可以直接使用`try catch`语法来捕捉错误

```javascript
let sleep = function (time) {
    return new Promise(function (resolve, reject) {
        setTimeout(function () {
            // 模拟出错了，返回 ‘error’
            reject('error');
        }, time);
    })
};

let start = async function () {
    try {
        console.log('start');
        await sleep(3000); //这里得到一个返回错误
        //所以一下代码不会执行
        console.log('end');
    } catch( e ){
        console.log(e); //这里捕捉错误 'e'
    }
}
```

## 循环多个`await`

await 写起来有种写同步代码, 所以可以理所当然的卸载`for`循环里面, 不必当心以往需要`闭包`才能解决的问题

```javascript
...省略以上代码

let start = async function () {
    for(let i = 1; i <= 10; i++ ){
        console.log(`当前是第${i}次等待...`);
        await sleep(1000;)
    }
};
```

指的注意的是,  `await`必须在`async函数的上下文中, 也就是await需要在async函数环境中`.

```javascript
...省略以上代码

let oneToTen = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];


// 错误示范
oneToTen.forEach(function (item) {
    console.log(`当前是第${item}次等待...`);
    await sleep(1000); // error!! await只能在async函数中运行
})

// 正确示范
for(let item of oneToten) {
    console.log(`当前是第${item}次等待...`);
    await sleep(1000); //正确,  for循环的上下文还在async函数中
}
```
