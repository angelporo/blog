---
title: js中的mixin
date: 2017-08-07 16:43:23
tags:
    - es6
    - mixin
---

## es5 中的`mixin`模式

熟悉 JavaScript 的同学应该对 mixin 模式并不陌生。我们说 JavaScript / ES5 的继承模型是基于单一原型链的继承模型，通常情况下，在 JavaScript 实践中完全用原型链来实现继承式的代码复用，是远远不能满足需求的。因此实战中，我们的代码抽象基本上都是采用混合的模式, 基友原型继承, 也有ixin组合.

那什么是 mixin 呢？ 最基本的 mixin 其实就是简单地**将一个对象的属性复制给另一个对象**：

```javascript
function mixin(targetObject, method) {
  for(let key in method) {
    targetObject[key] = method[key];
  }
}

let person = {name: 'liyuan', age: 26};
let student = {grade: 1};
mixin(student, person);
```
上面的代码做的事情很简单，就是枚举出一个对象的所有属性，然后将这些属性添加到另一个对象上去。

有时候，我们需要将多个对象的属性 mixin 到一个对象上去：

```javascript
let dest = { name: 'liyuan'};
[methed1, methed2].forEach( methed => {
  mixin(dest, methed);
})
```
每次都用 `forEach` 操作显然很繁琐，因此通常情况下， mixin 考虑支持操作多个对象：

```javascript
function mixin(...objs) {
  return objs.reduce( (dest, src) => {
    for(let key in src) {
      dest[key] = src[key];
    }
    return dest;
  });
}

let dest = mixin({name: 'liyaun'}, {age: 26});
```
在许多框架中，都有 `mixin` 的类似实现， jQuery 的 `extend`、YUI 有好几个类似 mixin 的 API，lodash 中有 _.mixin 方法，npm 中的 [mixin](https://www.npmjs.com/package/mixin) 模块每个月有上千的下载。

恩, 关于es5的mixin就说这么多吧,  因为我现在基本只写es6

总结:

从一定程度上，`mixin` 弥补了 `JavaScript` 单一原型链的缺陷，可以实现类似于多重继承的效果。在上面的例子里，我们让 `Employee` “继承” `Person`，同时也“继承” Serializable。有趣的是我们通过 mixin Serializable 让 Employee 拥有了 stringify 和 parse 两个方法，同时我们改写了 Employee 实例的 `toString` 方法。

```javascript
var employee = new Employee("jane",25,"f",1,1000);
var employee2 = Employee.parse( employee +""); //通过序列化反序列化复制对象

console.log(employee2,
    employee2 instanceof Employee,    //true
    employee2 instanceof Person,    //true
    employee == employee2);        //false
```

```javascript
function mixin(...objs){
    return objs.reduce((dest, src) => {
        for (var key in src) {
            dest[key] = src[key]
        }
        return dest;
    });
}

function createWithPrototype(Cls){
    var P = function(){};
    P.prototype = Cls.prototype;
    return new P();
}

function Person(name, age, gender){
    this.name = name;
    this.age = age;
    this.gender = gender;
}

function Employee(name, age, gender, level, salary){
    Person.call(this, name, age, gender);
    this.level = level;
    this.salary = salary;
}

Employee.prototype = createWithPrototype(Person);

mixin(Employee.prototype, {
    getSalary: function(){
        return this.salary;
    }
});

function Serializable(Cls, serializer){
    mixin(Cls, serializer);
    this.toString = function(){
        return Cls.stringify(this);
    }
}

mixin(Employee.prototype, new Serializable(Employee, {
        parse: function(str){
            var data = JSON.parse(str);
            return new Employee(
                data.name,
                data.age,
                data.gender,
                data.level,
                data.salary
            );
        },
        stringify: function(employee){
            return JSON.stringify({
                name: employee.name,
                age: employee.age,
                gender: employee.gender,
                level: employee.level,
                salary: employee.salary
            });
        }
    })
);
```
> [好的文章是直的保留下来的](https://www.h5jun.com/post/mixin-in-es6.html)
