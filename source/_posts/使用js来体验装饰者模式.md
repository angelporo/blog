---
layout: post
title: 使用js来体验装饰者模式
date: 2017-08-11 14:22:31
tags:
    - 设计模式
    - es7
    - decorator
categories: "设计模式"
---

## 装饰者模式:

在不必改变原始**类**文件和使用继承的情况下, 动态地扩展一个对象的功能, 通过一个创建一个包装对象, 也就是装饰者来包裹这个真实的对象, 这样我们就可以为对象添加一个方法或者一些行为, 但是保持变量名不变。 将方法调用传递给原始对象; 达到不改变原始对象的情况下, 扩展原始对象的目的.

{% asset_img 理解装饰者模式.jpg 理解装饰者模式 %}

其实这个模式也可以使用的**函数**上面,  看起来像是下面这样子的

```javascript
// 原有函数
let getData = () => {
  return (new Date()).toString();
};

// 包装函数,  写过函数式的应该很容易就可以看明白
let upperCaseDacorator = fn => {
  return (...arg) => {
    return fn.apply(this, arg).toUpperCase();
  }
};

// 调用装饰折后的函数方法
let test = upperCaseDacorator(getData)()
console.log(test);
```

个人经常使用上面这种形式的方法,  平时写函数比较多, 看起来也更简单.

## 装饰者模式和`es7`中的`decorator`

[不知道的可以先看看这里](http://www.cnblogs.com/whitewolf/p/details-of-ES7-JavaScript-Decorators.html)
`es7`中增加了一个`decorator`属性, 借鉴自`python`,

### 下面我们以 钢铁侠 为例讲解如何使用 ES7 的 decorator。

以钢铁侠为例，钢铁侠本质是一个人，只是“装饰”了很多武器方才变得那么 NB，不过再怎么装饰他还是一个人。

{% asset_img 钢铁侠.jpg 理解装饰者模式 %}

我们的示例场景是这样的

- 创建一个普通的Man类，它的抵御值 2，攻击力为 3，血量为 3；
- 我们让其带上钢铁侠的盔甲，这样他的抵御力增加 100，变成 102；
- 让其带上光束手套，攻击力增加 50，变成 53；
- 最后让他增加“飞行”能力

好的, 现在我们来创建普通`Man`类:

```javascript
class Man{
  constructor(def = 2, atk = 3, hp = 3) {
    this.init(def, atk, hp);
  }
  init(def, atk, hp) {
    this.def = def; // 防御值
    this.atk = atk; // 攻击力
    this.hp = hp; // 血量
  }
  toString() {
    return `防御力:${this.def},攻击力:${this.atk},血量:${this.hp}`;
  }
}

let tony = new Man();

console.log(`当前状态 ===> ${tony}`);
// 输出：当前状态 ===> 防御力:2,攻击力:3,血量:3
```
穿件`decorateArmour`头盔方法, 为钢铁侠装配盔甲 . `decorateArmour`是装饰在方法`init`上的

```javascript
function decorateArmour(target, key, descriptor) {
  const method = descriptor.value;
  let moreDef = 100;
  let ret;
  descriptor.value = (...args) => {
    args[0] += moreDef;
    ret = method.apply(target, args);
    return ret;
  };
  return descriptor;
}

class Man{
  constructor(def = 2, atk = 3, hp = 3) {
    this.init(def, atk, hp);
  }

  @decorateArmour
  init(def, atk, hp) {
    this.def = def; // 防御值
    this.atk = atk; // 攻击力
    this.hp = hp; // 血量
  }
  toString() {
    return `防御力:${this.def},攻击力:${this.atk},血量:${this.hp}`;
  }
}

// 输出：当前状态 ===> 防御力:102,攻击力:3,血量:3
```
我们看到输出结果, `输出：当前状态 ===> 防御力:102,攻击力:3,血量:3`, 看起来盔甲起作用了.

初学者这里会有两个疑问

- `decorateArmour`方法的参数为啥是这三个? 可以更换么?
- `decorateArmour`方法为什么放回的是`descriptor`

这里给出个人的解答作为参考:
- **Decorators**的本质是利用了es5中的`Object.defineProperty`属性, 这三个参数其实是和`Object.defineProperty`参数一样的, 因此不能改变, [详细分析参考](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)
- 可以看看**bable转码后**的代码, 其中有一句是`descriptor = descriptor(target, key, descriptor) || descriptor;`想要具体理解还需要看代码中的上下文.

**demo2 装饰器叠加: 增加光束手套**
在上面的示例中, 我们成功为普通人 增加'盔甲'这个装饰;现在我们继续添加光束手套, 来达到增加50点防御值.
我们还是在**`decorateArmour`**方法做修改, 改名为`decorateLight`, 同事修改防御值的属性:
```javascript
function decorateLight(target, key, descriptor) {
  const method = descriptor.value;
  let moreAtk = 50;
  let ret;
  descriptor.value = (...args) => {
    args[1] += moreAtk;
    ret = method.apply(target, args);
    return ret;
  }
  return descriptor;
}
```
Step2: 直接在**`init`**方法上添加装饰语法:
```javascript
@decorateArmour
  @decorateLight
  init(def,atk,hp){
    this.def = def; // 防御值
    this.atk = atk;  // 攻击力
    this.hp = hp;  // 血量
  }
```

最后, 我们的代码就成了下面的样子
```javascript
function decorateLight(target, key, descriptor) {
  const method = descriptor.value;
  let moreAtk = 50;
  let ret;
  descriptor.value = (...args)=>{
    args[1] += moreAtk;
    ret = method.apply(target, args);
    return ret;
  }
  return descriptor;
}

class Man{
  constructor(def = 2,atk = 3,hp = 3){
    this.init(def,atk,hp);
  }

  @decorateArmour
  @decorateLight
  init(def,atk,hp){
    this.def = def; // 防御值
    this.atk = atk;  // 攻击力
    this.hp = hp;  // 血量
  }
}
var tony = new Man();
console.log(`当前状态 ===> ${tony}`);
//输出：当前状态 ===> 防御力:102,攻击力:53,血量:3
```

我们这个总结一下上面装饰模式的优势, 它可以对方法进行叠加使用, 对原始类的侵入行非常小, 只是增加一行
`@decorateLight`, 可以方便的修改(同事还可以复用)

装饰模式有两种: **纯粹的装饰模式**和**半透明的装饰模式**.
上面的demo中所使用的恩是 **纯粹的装饰模式**, 它并不增加对原始累的接口;
下面我们使用**半透明的装饰模式**来增加一个飞行的能力.看样子像适配器模式(TODO: 适配器模式)的样子

Step1: 增加一个方法
```javascript
function addFly(canFly) {
  return target => {
    target.canFly = canFly;
    const extra = canFly ? '(添加飞行能力)' : '';
    const method = target.prototype.toString;
    target.prototype.toString = (...args) => {
      return method.apply(target.prototype.args) + extra;
    }
    return target;
  }
}
```

Step2: 这个方法将直接去装饰原始lei:
```javaScript
function addFly(canFly){
  return function(target){
    target.canFly = canFly;
    let extra = canFly ? '(技能加成:飞行能力)' : '';
    let method = target.prototype.toString;
    target.prototype.toString = (...args)=>{
      return method.apply(target.prototype,args) + extra;
    }
    return target;
  }
}

@addFly(true)
class Man{
  constructor(def = 2,atk = 3,hp = 3){
    this.init(def,atk,hp);
  }

  @decorateArmour
  @decorateLight
  init(def,atk,hp){
    this.def = def; // 防御值
    this.atk = atk;  // 攻击力
    this.hp = hp;  // 血量
  }
}

console.log(`当前状态 ===> ${tony}`);
// 输出：当前状态 ===> 防御力:102,攻击力:53,血量:3(技能加成:飞行能力)
```
作用在方法上的 `decorator` 接收的第一个参数（ target ）是类的 prototype ；如果把一个 `decorator` 作用到类上，则它的第一个参数 target 是 类本身 。

## 总结
看到这里估计还有些人会迷糊着, 说到底,  装饰模式中的`@`其实就是一个**高阶函数**, 下面是语法糖给你传入的参数;
这里就那`redux`中的`connect`函数来举例子,

`connect(mapStoreToProps, mapDispatchToProps)(Component)`这里是`connect`方法的具体调用方法, 其中使用装饰者模式就是下面的样子
```javascript

@connect(mapStoreToProps, mapDispatchToProps)
export class App extends Component {
    render () {
        return <div id="">组件</div>
    }
}
// 这里可以看出装饰者模式在es7中只是一种语法糖!!!
```

> [原文地址](http://www.tuicool.com/articles/2yeiEr)

## 使用原生**js**实现装饰器模式

> [javascript设计模式: 装饰者模式](http://www.codingserf.com/index.php/2015/05/javascript-design-patterns-decorator/)
