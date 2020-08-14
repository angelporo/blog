---
layout: post
title: javascript中的new操作符
date: 2019-9-24 09:33:32
tags:
    - new
---

# 温故而知新: `new`一个对象到底发生了什么!

*《JavaScript高级程序设计（第三版）》* P145是这么写的：
> 要创建 `Person` 的新实例，必须使用`new`操作符。以这种方式调用构造函数实际上会经历以下4个步骤：
 （1）创建一个新对象；
 （2）将构造函数的作用域赋给新对象（因此this就指向了这个对象）；
 （3）执行构造函数中的代码（为这个新对象添加属性）；
 （4）返回新对象。


## 按照上面的说法, 我们来*创建*一个对象

构造函数:
```javascript
function Person (name) {
    this.name = name;
}
```


接下来我们创建一个`Person`:
```javascript

function newPerson (name)  {
    let O = {}; // 创建一个空对象, 用来存放
    Person.call(O, name) // 使用 `Function` 原型下的 `call` 方法来绑定 `this`
    return O;  //方法返回`O`
}

const person1 = newPerson('小李');
const person2 = new Person('小李');

person1 instanceof Person // false
```

*`person1 instanceof Person`* 这个尽然是 `false`, 仔细观察. 原来我们这个`call`方法只是指定了`this` 并没有 绑定原型, 那好办, 
我们把`O.__proto__ = Person.prototype` 手动赋值的形式来绑定`prototype`


然后我们的`new`方法就成了下面的样子:
```javascript
function newPerson (name)  {
    let O = {}; // 创建一个空对象, 用来存放
    // let O = { __proto__: P.prototype }; 或者一次操作
    O.__proto__ = Person.prototype; // 手动绑定原型链
    Person.call(O, name) // 使用 `Function` 原型下的 `call` 方法来绑定 `this`
    return O;  //方法返回`O`
}
```


## FBI WARNING ⚠️
{% asset_img 1.png 关于直接设置`__proto__`属性的⚠️ %}
> 有点牛头不对马尾, 还是提醒下把

## 总结:
> 由此可见, `person1` 和 `person2` 的`prototype` 在同一条原型链上, 所以他们两个使用同一个构造函数构造出来.
> 也证实了书中自有黄金屋

