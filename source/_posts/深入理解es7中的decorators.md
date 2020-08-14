---
layout: post
title: 深入理解es7中的decorators
date: 2017-08-11 16:57:37
tags:
    - es7
    - 装饰者
categories: "decorator"
---

## 背后原理
ES7中的`Decorators`让我们能够在设计是对类, 属性等进行标注和修改成为了可能, `Decorators`是利用了es5中的
```javascript
Object.defineProperty(target, name, descriptor);
```
如果对`Object.defineProperty`不够了解, 请查阅[Mdn文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)首先我们来考虑一个es6中的类:
```javascript
class Person {
    name() {
        return `${ this.first } ${ this.last }`
    }
}
```
运行上面class, 给`Person.prototype`注册一个name属性, 粗略的和下面代码相似:
```javascirpt
Object.defineProperty(Person.prototype, 'name', {
  value: specifiedFunction,
  enumerable: false,
  configurable: true,
  writable: true
})
```
如果利用装饰器来标注一个属性呢?
```javascript
class Person{
  $readonly
  name() {
    return `${this.first} ${this.last}`;
  }
}
// 其实就是一种语法糖
```
上面代码展开类似下面代码
```javascript
let descriptor = {
  value: specifiedFunction,
  enumerable: false,
  configurable: true,
  writable: true
};

descriptor = readonly( Person.prototype, 'name', descriptor ) || descriptor;
Object.defineProperty(Person.prototype, 'name', descriptor);
```
上面的代码中, 我们能看出, 装饰器只是在Object.defineProperty为Person.prototype注册属性之前, 执行一个装饰函数, 其属于一类对Object.defineProperty的拦截.所以它和Object.defineProperty具有一致的方法签名, 他们的三个参数分别为:
    - obj: 需要被操作的对象
    - prop: 被操作对象定义或修改的属性的名称
    - descriptor: 针对该属性的描述符

对象里目前存在的属性描述符有两种主要形式：数据描述符和存取描述符。数据描述符是一个拥有可写或不可写值的属性。存取描述符是由一对 getter-setter 函数功能来描述的属性。描述符必须是两种形式之一；不能同时是两者。

> 先学到这里, 因为自己用的也不是很熟悉. 后续...
