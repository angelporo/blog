---
layout: post
title: iphone和safari浏览器中关于Date对象的兼容
date: 2017-11-09 17:42:18
tags:
    - Date
    - 时间转换
categories: "算法"
---

今天公司遇到一个很奇怪的问题,  是微信端的商城,  促销板块需要来显示出`倒计时`, 后端传过来一个`2032-04-23 12:12:32`格式的数据,  然后前端来显示其倒计时

因为用到了`new Date(stringDate)`结果iphone就不显示了,  找了一下午,  在加上调试不方便,  真的很烦人,

原来是苹果不支持`stringDate`这种解析,  还需要吧`stringDate`转换成数组然后逐个传入

```javascript
function getDateForStringDate(strDate){
      //切割年月日与时分秒称为数组
      var s = strDate.split(" ");
      var s1 = s[0].split("-");
      var s2 = s[1].split(":");
      if(s2.length==2){
        s2.push("00");
      }
      return new Date(s1[0],s1[1]-1,s1[2],s2[0],s2[1],s2[2]);
    }
function clock(t){
    var n = new Date().getTime(),//取得当前毫秒数
        c = t - n;//得到时间差
    if(c<=0){
        document.getElementById("jstimerBox").innerHTML ='活动已经结束';
        // clearInterval(window['timer']);//清除计时器
        return;
    }
    var ds = 60*60*24*1000,
        d = parseInt(c/ds),
        h = parseInt((c-d*ds)/(3600*1000)),
        m = parseInt((c - d*ds - h*3600*1000)/(60*1000)),
        s = parseInt((c-d*ds-h*3600*1000-m*60*1000)/1000);
        document.getElementById("jstimerBox").innerHTML = d + '天' + h + '时' + m + '分' + s + '秒';
}

clock(getDateForStringDate("2013-11-22 00:12:32")) // safari是可以兼容的
```

