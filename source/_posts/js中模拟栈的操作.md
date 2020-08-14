---
layout: post
title: js实现多种括号匹配的解题思路
date: 2017-08-17 17:28:17
tags:
    - 正则
    - 模拟栈
categories: "字符串操作"
---
## 前言:
>这几天工作中项目不是很忙,  也真是这样, 我们组中不知道怎么就开始在`codewars`上面刷题了.我也不例外, 就和他们一起来玩耍这个游戏.
>然而在昨天的时候遇到一个题:

## 解决问题
要求要求匹配`()[]{}`着三种括号的匹配,函数返回一个匹配的`boole`值

### 方法1: 模拟出栈入栈来实现
我在网上查的是通过模拟栈的方式来解决, 因为每次`()`**左括号**要匹配成功, 必须有一个相对应的**右括号**这样才能功能匹配,
如果没有相对于的**有括号**就匹配失败,利用**出栈, 入栈**的理解刚好可以和这个想吻合, 遇到**左括号**入栈遇到**有括号**出栈, 随后检测栈中有没有元素.
通过上面思路我们来写代码
```javascript
function groupCheck (s)  {
  if (s.length % 2 !== 0) return false;
  let len = s.length,
      res = [];
  for(let i = 0; i < len; i++) {
    switch(s[i]){
    case '(':res.push(1);break; // 由于括号有3中, 所以我们给他们每一种打上标签 , 出栈的时候对应标签选择出栈;
    case '[':res.push(2);break;
    case '{':res.push(3);break;
    case ')':if(res[res.length - 1] !== 1) return false;res.pop();break;
    case ']':if(res[res.length - 1] !== 2) return false;res.pop();break;
    case '}':if(res[res.length - 1] !== 3) return false;res.pop();break;
    }
  }
  return res.length == 0;
}
groupCheck('()[{}()[{}]]');
// return 结果为true;
```
这种方法代码看起来不是很简便, 是因为我当时写完之后, 看到'最佳实践'只有4行代码,  才恍然大悟,

### 方法2: 正则匹配然后替换

这个方法总体思路也是使用模拟出入栈的方式来检测栈中内容, 而这个方式看起来更简单, 更高效, 直接操作的字符串, 不是数组,  所以实际意义上, 并不是模拟出入栈, 只是借鉴了这种想法.
```javascript
function groupCheck(s){
   var r = /\{\}|\[\]|\(\)/; // 使用正则来匹配是否一对括号.
   while(r.test(s)) // 知道没有匹配括号为止
     s = s.replace(r, ''); // 把匹配中的括号替换为空字符串;
   return !s.length;
 }
groupCheck('()[{}()[{}]]');
// return 结果为true;
```
