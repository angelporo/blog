---
layout: post
title: js中迭代器和生成器
date: 2018-02-02 17:06:36
tags:
    - yield
    - es6
    - 迭代器
    - 可迭代对象
    - 生成器
categories: "算法"
---
# 迭代器和可迭代对象

请先看看[可迭代协议和迭代器协议](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Iteration_protocols#iterable)

## 定义

迭代器`iterator`是一个`Object`, 这个`Object`有一个`next`函数, 该函数返回一个`value`和`done`属性额`Object`其中`value`指向迭代序列中下一个值 . 这样看来迭代器中定义简直简单的感人......

栗子:
``` javascript
/**
* 制作一个迭代器
**/
function makeIterator(array) {
    console.log("Enter this function");
    var nextIndex = 0; // 设置局部count
    return {
        next: function () {
            return nextIndex < array.length ?
                {value: array[nextIndex++], done: false } :
                { done: true }
        }
    };
}
 var iteratorT = makeIterator([1, 2, 3]);
     console.log(iteratorT) // 打印出{next}函数
     console.log(iteratorT.next().value); // 1
     console.log(iteratorT.next().value); // 2
     console.log(iteratorT.next().value); // 3
     console.log(iteratorT.next().done);  false
```

## 可迭代协议

这是一个协议, 一旦支持可迭代协议, 就意味着该`Object`可以用`for-of`来遍历, 可以用来定义或者定制js对象的迭代行为. 常见的内建类型比如`Array & Map` 都是支持可迭代协议的.

### 实现该协议

对象必须实现@@iterator方法,  意味着对象必须有一个带有@@iterator key的可以通过常量`Symbol.iterator`访问到的属性.
`Symbol.iterator` 属性 返回一个对象的无参数函数, 被返回对象符合迭代协议

## 迭代器协议

上面的例子就是一个迭代器协议的实现 `iterator`协议定义了产生`value`序列的一种标准方法, 只要实现符合要求的next函数, 改对象就是一个迭代器. 这个概念是不是和`golang`中的`interface`相似呢.

`String`内置的可迭代对象
```javascript
var someString = "hi";
typeof someString[Symbol.iterator];          // "function"
```

可见`String`内部已经实现了可迭代协议

String 的默认迭代器会一个接一个返回该字符串的字符

```javascript
var iterator = someString[Symbol.iterator]();
iterator + "";                               // "[object String Iterator]"

iterator.next();                             // { value: "h", done: false }
iterator.next();                             // { value: "i", done: false }
iterator.next();                             // { value: undefined, done: true }
```

我们可以通过自己的 @@iterator 方法重新定义迭代行为：
```javascript
var someString = new String("hi");          // need to construct a String object explicitly to avoid auto-boxing

someString[Symbol.iterator] = function() {
  return { // this is the iterator object, returning a single element, the string "bye"
    next: function() {
      if (this._first) {
        this._first = false;
        return { value: "bye", done: false };
      } else {
        return { done: true };
      }
    },
    _first: true
  };
};
```

`String`, `Array`, `TypedArray`, `Map and Set` 是所有内置可迭代对象， 因为它们的原型对象都有一个 `@@iterator` 方法.


### 自定义可迭代对象

```javascript
var myIterator = {};

myIterator[Symbol.iterator] = function* () {
    yield 1;
    yield 2;
    yield 3;
}

[ ...myIterator ] //[1, 2, 3]
```


如何还有不理解的可以自己去mdn上面仔细看看.
