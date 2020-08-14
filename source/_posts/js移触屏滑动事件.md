---
title: js移触屏滑动事件
date: 2017-06-07 09:22:00
categories: web前端
tags:
  - h5
type: "categories"
---

主要说一下移动端中算是经常用到的几个事件, 手机端写了好多, 一直都懒得去写滑动的一些功能组件, 今天有时间研究一下touch事件

## 介绍

- touchstart: 手指放到屏幕上时触发
- touchmove: 手指在屏幕上滑动式触发
- touchend: 手指离开屏幕时触发
- touchcancel: 系统取消touch事件的时候粗发

如果有偏差请到 这里查看详细[javascritp mdn](https://developer.mozilla.org/zh-CN/docs/Web/API/TouchEvent/touches)

<!-- more -->

```javascript
someElement.addEventListener('touchstart', e => {
  // Invoke the appropriate handler depending on thew
  // number of touch points.
  switch (e.touches.length) {
    case 1: handle_one_touch(e); break;
    case 2: handle_two_touches(e); break;
    case 3: handle_three_touches(e); break;
    default: console.log("Not supported"); break;
  }
},
false);
```

### 具体说一下触发后生成的event事件 通过回调传递过去

```javascript
someElement.addEventListener('touchstart', e => {
  // e.touches 当前屏幕上所有手指的列表
  // e.targetTouches 当前dom元素上手指的列表，尽量使用这个代替touches
  // e.changedTouches 涉及当前事件的手指的列表，尽量使用这个代替touches
  // 这些列表里的每次触摸由touch对象组成，touch对象里包含着触摸信息，主要属性如下：
  // clientX / clientY:      触摸点相对浏览器窗口的位置
  // pageX / pageY:       触摸点相对于页面的位置
  // screenX  /  screenY:    触摸点相对于屏幕的位置
  // identifier:        touch对象的ID
}
 }, false);
```

具体事件看来下, 接下来就做一个小小的demo来加深下理解

### html结构

```html
<div id="common_wrap" class="common-wrap">
    <h4 class="common-kit__h4">在区域内向左右滑动</h4>
        <!-- 这里让li左浮动, 然后ul定位设置为absolute 这样就可以改变ul中的left值来移动ul的位置 -->
        <ul class="common-kit__list" id="mask" style="left:0px">
            <li><a href="javascript:;">111</a></li>
            <li><a href="javascript:;">222</a></li>
            <li><a href="javascript:;">333</a></li>
            <li><a href="javascript:;">444</a></li>
            <li><a href="javascript:;">555</a></li>
            <li><a href="javascript:;">666</a></li>
        </ul>
</div>
```

### 具体样式实现代码

```css
.common-wrap{
    width: 100%;
    height: 105px;
    border-bottom: 8px solid #eee;
}
.common-kit__h4{
    font-size: 14px;
    margin-top: 17px;
    margin-left: 8px;
    letter-spacing: 0.2px;
}
.common-kit{
    width: 100%;
    position: relative;
}
.common-kit__list{
    width: 558px;
    position: absolute;
    margin-top: 10px;
    height: 80px;
}
.common-kit__list li{
    position: relative;
    list-style: none;
    width: 80px;
    height: 80px;
    background-color: #eee;
    float: left;
    margin-left: 13px;
}
.common-kit__list li a{
    text-decoration: none;
    font-size: 12px;
    position: absolute;
    top:50%;
    transform:translateY(-50%);
    text-align: center;
    padding: 0px 12px;
}
.common-kit__list li:first-child{
    margin-left: 8px;
}
```

### js代码实现以及注释原理

```javascript
function slidecommonkit(){

    const mask = document.getElementById('mask');
    const common_kit__list=document.querySelector('.common-kit__list');
    const startPosition, endPosition, deltaX, deltaY, moveLength;
    let commonkitLeft;

    /*手指按下瞬间触发touchstart事件*/
    mask.addEventListener('touchstart', e => {
        commonkitLeft=parseInt(common_kit__list.style.left);
        const touch = e.targetTouches[0];  //targetTouches位于当前DOM元素上的手指动作的列表
        startPosition = {   //取屏幕上第一个手指相对于页面的坐标
            x: touch.pageX,
            y: touch.pageY
        }
    });

    /*手指移动触发touchmove事件*/
    mask.addEventListener('touchmove', function (e) {
        var touch = e.targetTouches[0];
        endPosition = {
            x: touch.pageX,
            y: touch.pageY
        }

        deltaX = endPosition.x - startPosition.x;   //移动到最后的坐标x - 开始时的坐标x
        moveLength = Math.abs(deltaX);   //获得移动的x方向的距离

        /*向左移动的函数*/
        var swipeLeft=function(){
            if( deltaX<(-30) ){   //这里以30作为判断是否触发、如果deltaX小于-30，说明向左移动

                if(Math.abs(commonkitLeft)+moveLength > ( common_kit__list.offsetWidth-window.innerWidth ) ){   //判断临界值
                    common_kit__list.style.left=window.innerWidth-common_kit__list.offsetWidth+'px';
                }else{
                    common_kit__list.style.left=commonkitLeft-moveLength+'px';  //上一次的left值-移动的距离（由于距离是正数，而向左移动left值是负数，所以用-）
                }
            }
        }
        swipeLeft();   //执行该函数

        /*向右移动的函数*/
        var swipeRight=function(){
            if( deltaX>30 ){
                /*主要是逻辑*/
                if( commonkitLeft + moveLength > 0  ){
                    common_kit__list.style.left=0+'px';
                }else{
                    common_kit__list.style.left=commonkitLeft+moveLength+'px';
                }
            }
        }
        swipeRight();

    });

};

slidecommonkit();
```
