---
layout: post
title: javascript二叉树
date: 2019-08-05 15:01:38
tags:
    - 二叉树创建
    - 二叉树排序
---
## 引言:

> * 二叉树是非常基础又非常重要的数据结构，在一些场合有着非常重要的作用。掌握二叉树对编写高质量代码、减少代码量有很大的帮助！
> * 二叉树是一种特殊的树， 非常适合计算机处理数据， 所以对于程序员来说掌握二叉树是非常有必要的。
> * 二叉树的前序遍历可以用来显示目录结构等；中序遍历可以实现表达式树，在编译器底层很有用；后序遍历可以用来实现计算目录内的文件及其信息等。二叉树是非常重要的数据结构， 其中二叉树的遍历要使用到栈和队列还有递归等，很多其它数据结构也都是基于二叉树的基础演变而来的。熟练使用二叉树在很多时候可以提升程序的运行效率，减少代码量，使程序更易读。



## javascript 实现二叉树
上图:
{% asset_img 1.png 二叉树 %}

## 创建

```
var tree = {
    value: "-",
    left: {
        value: '+',
        left: {
            value: 'a',
        },
        right: {
            value: '*',
            left: {
                value: 'b',
            },
            right: {
                value: 'c',
            }
        }
    },
    right: {
        value: '/',
        left: {
            value: 'd',
        },
        right: {
            value: 'e',
        }
    }
}
```


## 遍历
> 二叉树有深度遍历和广度遍历， 深度遍历有前序、 中序和后序三种遍历方法。 广度遍历就是层次遍历。 因为树的定义本身就是递归定义， 因此采用递归的方法实现树的三种遍历容易理解而且代码比较简洁。
> 有时对一段代码来说， 可读性有时比代码本身的效率要重要的多。

四种遍历的主要思想：
>   * 前序遍历：访问根–>遍历左子树–>遍历右子树;
>   * 中序遍历：遍历左子树 –> 访问根 –> 遍历右子树;
>   * 后序遍历：遍历左子树 –> 遍历右子树 –> 访问根;
>   * 广度遍历：按照层次一层层遍历;


1. **`前中后`** 序 **递归** 遍历
```javascript
// 前序递归
function recursionTree(tree) {
    if(tree) {
        // 前序递归
        console.log('当前根节点', tree.value) // 访问前序
        recursionTree(tree.left) // 递归左子树
        recursionTree(tree.right) // 递归右子树
    }
}
// 前序输出 - + a * b c / d e


// 中序递归排序
// 前序和中序递归只是换了下位置
function recursionTree(tree) {
    if(tree) {
        // 中序递归
        recursionTree(tree.left) // 递归左子树
        console.log('当前根节点', tree.value) // 访问中序
        recursionTree(tree.right) // 递归右子树
    }
}
// 递归顺序: "a", "+", "b", "*", "c",(这个位置左子数已经遍历完成) "-"(获取根节点),(开始遍历右子数) "d", "/", "e"


// 后序递归
function recursionTree(tree) {
    if(tree) {
        // 后序递归
        recursionTree(tree.left) // 递归左子树
        recursionTree(tree.right) // 递归右子树
        console.log('当前根节点', tree.value) // 访问根节点
    }
}
// 递归顺序: "a", "b", "c", "*", "+",(左子树遍历完成) "d", "e", "/",(右子树遍历完成) "-"(最后获取根节点)
```

## 递归总结

---
{% asset_img 2.png 二叉树遍历路径 %}

通过上面遍历可以看出, 遍历路径是相同的  唯一不同的是访问的顺序 , ** 按照路径来, 每个节点 , 都有三次相遇的机会, 第一次就是前序, 第二次就是中序, 第三次就是后序 **

前序是我们第一次遇到节点的时候, 中序是第二次遇到节点的时候 , 第一次我们压栈, 第二次我们出栈

## 利用中序递归 推到出 堆栈非递归遍历

上面明白了 **中序** 递归访问节点路线后, 我们就可以看出来, 在递归完左子树后后回头访问中间的节点, 然后访问右子树, 依次类推, 遍历路线就成了 `左 -> 中 -> 右`

### **总结**:
------
* 遇到一个`节点`, 就把它`压栈`, 并去遍历他的`左子树`;
* 当左子树遍历结束后, 从栈顶弹出这个节点并访问它;
* 然后按照其右指针再去遍历该节点的`右子树`


### 代码实现

### 中序非递归排序
````
function inOrderTrversalTree(tree) {
  let dock = [], // 创建安全栈 用来入栈出栈
      T = tree,  // 创建树
      result = [];
  while(T || !!tree){
    while(T){   // 当前树
      dock.push(T)
      T = T.left // 修改循环条件 来 循环左子树
    }
    if(dock.length !== 0){ // 如果栈中右数据 这出栈, 并切换循环右子树
      T = dock.pop() // 出栈
      result.push(T.value) //
      T = T.right // 修改循环条件 来 循环右子树
    }
  }
  return result;
}
const result = inOrderTrversalTree(tree)
console.log(result) // "a", "+", "b", "*", "c", "-", "d", "/", "e" 和上面递归排序是一样的
```


### 非递归前序排序
---
通过上面得出结论我们修改一下中序中**访问顺序** 方可得到前序非递归排序

{% asset_img 3.png 二叉树遍历路径 %}


### 后序排序
```javascript
// 后序排序
function inOrderTrversalTree(tree){
  var arr=[],res=[];
  arr.push(tree);
  while(arr.length!=0){
    var p = arr.pop();
    res.push(p.val);
    if(p.left != null){
      arr.push(p.left);
    }
    if(p.right != null){
      arr.push(p.right);
    }
  }
  return res.reverse();
}
inOrderTrversalTree(root)
```

## 队列排序
---
遍历从根节点开始, 首先将根节点放入列, 然后开始执行循环, 每次从队列里面拿出一个元素, 访问该节点, 然后把该节点的左右儿子放入队列里面, 依次循环
```javascript
function levelOrderTraversal(tree) {
    let T = tree; // 默认
    let Q = []; // 创建队列
    let result = []; // 创建返回
    if(!tree) return ; // 空树这直接返回
    Q.push(T); // 默认入队树
    while(Q.length !== 0) { // 如果树存在则 循环
        T = Q.pop() // 抛出队列元素
        result.push(T.value);
        if(T.left) Q.push(T.left); // 将下一层节点放入队列
        if(T.right) Q.push(T.right)
    }
    return result
}
```

## 二叉树的深度
---


```javascript
function deep(root){
    if(!root){
        return 0
    }
    let left = deep(root.left)
    let right = deep(root.right)
    console.log('left',left)
    return left > right ? left + 1 : right + 1 // 返回当前深度, 依次左右子树 递归 比较后获取深度深的为最大深度
}

deep(tree)
```

## 二叉树的应用:

可以用来抽象语法树, 但是中序排序解析语法树会错误 , 这个和中序排序访问顺序有关, 所以, 前序后后序解析语法树都没有问题的.
---
