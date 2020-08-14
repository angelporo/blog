---
layout: post
title: javascript中对象的引用
date: 2020-02-23 11:34:11
tags:
    - javascript
---

# 温习一下js中的对象引用与基本数据赋值的不同

## 基本数据类型
```
• 空值(null)
• 未定义(undefined)
• 布尔值( boolean)
• 数字(number)
• 字符串(string)
• 符号(symbol，ES6 中新增)
```

对于基本类型，赋值（=）是值的拷贝，比较（===）的是实际的值，而对于引用类型（Array也是一种Object），赋值（=）是引用地址的拷贝，比较（===）的是引用地址：


```javascript

const a = '哈哈'
const b = '哈哈'
console.log(a === b) // true 基本数据类型 栈中存放的就是这个变量本身

const c = {}  // 对象
const d = {}  // 对象
console.log(c === d) // false 由于栈中使对象的引用, 其实就是一个 二进制 转 十六进制 的堆内存地址 , c和d分别指向不同的堆地址所以不同(就是一个不同的指针) 其中在 golang 中又指针的概念理解这个还是很容易的 .
```

备注:
js 中引用对象分别有:`object、Array、RegExp、Date、Function`

- 上面所有对象 内存分配都是在堆中, 栈中只是存放了指向堆的指针
- 因为内存分配有 指针 和 直接存放 区别所有 对象拷贝的时候会有`深拷贝`和`浅拷贝`之分 ,  想要了解[javascript mdn 深浅拷贝之分](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)

[更多案例](https://segmentfault.com/a/1190000014724227)
