---
layout: js中的source
title: JavaScript Source Map详解
date: 2018-05-04 13:02:26
tags:
    - js
---

## 前言

接触source map是在使用`autoprefixer`来自动给css添加前缀的时候发现底部有段代码
```css
body{
    border-radius: 10px;
    display: flex;
}

/*# sourceMappingURL=all.css.map */
```
上面是css中的

js中的`//@ sourceMappingURL=jquery.min.map``jquery`中有使用

上面指定的map是一个独立的map文件, 与源码在同一个目录下, 上面css中的文件是这样子的:
```json
{"version":3,"sources":["test.css"],"names":[],"mappings":"AAAA;IACI,oBAAoB;IACpB,cAAc;CACjB","file":"all.css","sourcesContent":["body{\n    border-radius: 10px;\n    display: flex;\n}\n"]}
```


## 从源码转换讲起

JavaScript脚本正变得越来越复杂。大部分源码（尤其是各种函数库和框架）都要经过转换，才能投入生产环境。

其中postcss也是需要转换才能投入生产

常见的源码转换，主要是以下三种情况：

- 压缩，减小体积。比如jQuery 1.9的源码，压缩前是252KB，压缩后是32KB。
- 多个文件合并，减少HTTP请求数
- 其他语言编译成JavaScript。最常见的例子就是CoffeeScript。

这种情况就导致源代码和生产代码有很大的区别, `debug`就变得很复杂

js debug 通常，JavaScript的解释器会告诉你，第几行第几列代码出错。但是，这对于转换后的代码毫无用处。举例来说，jQuery 1.9压缩后只有3行，每行3万个字符，所有内部变量都改了名字。你看着报错信息，感到毫无头绪，根本不知道它所对应的原始位置。

有了source map就可以知道编译后的代码对应的源代码字符和jquery中变量, 这只是我的猜测, 并没有做实际的测试, 继续往下看就知道了其中的道理.

这就是Source map想要解决的问题。

## 什么是Source map

简单说，Source map就是一个信息文件，里面储存着位置信息。也就是说，转换后的代码的每一个位置，所对应的转换前的位置。

有了它，出错的时候，除错工具将直接显示原始代码，而不是转换后的代码。这无疑给开发者带来了很大方便。


目前，暂时只有Chrome浏览器支持这个功能。在Developer Tools的Setting设置中，确认选中"Enable source maps"。

## 如何启动Source map详解

正如前文所提到的，只要在转换后的代码尾部，加上一行就可以了。
js:
```js
//@ sourceMappingURL=/path/to/file.js.map
```

css:
```css
/*# sourceMappingURL=all.css.map */
```

map文件可以放在网络上，也可以放在本地文件系统。

## 如何生产Source map

常用方式使用Goolge的[Closure Compiler](https://developers.google.com/closure/compiler/)

生成命令的格式如下：
```base
java -jar compiler.jar \ 
　　　　--js script.js \
　　　　--create_source_map ./script-min.js.map \
　　　　--source_map_format=V3 \
　　　　--js_output_file script-min.js
```

各个参数的意义如下：
```
js： 转换前的代码文件
　　- create_source_map： 生成的source map文件
　　- source_map_format：source map的版本，目前一律采用V3。
　　- js_output_file： 转换后的代码文件。
```

## Source map的格式

```json
{
　　　　version : 3,
　　　　file: "out.js",
　　　　sourceRoot : "",
　　　　sources: ["foo.js", "bar.js"],
　　　　names: ["src", "maps", "are", "fun"],
　　　　mappings: "AAgBC,SAAQ,CAAEA"
　　}
```
整个文件就是一个JavaScript对象，可以被解释器读取。它主要有以下几个属性：
``` base
    - version：Source map的版本，目前为3。

　　- file：转换后的文件名。

　　- sourceRoot：转换前的文件所在的目录。如果与转换前的文件在同一目录，该项为空。

　　- sources：转换前的文件。该项是一个数组，表示可能存在多个文件合并。

　　- names：转换前的所有变量名和属性名。

　　- mappings：记录位置信息的字符串，下文详细介绍。
```


> 转自[阮一峰大大的blog](http://www.ruanyifeng.com/blog/2013/01/javascript_source_map.html)
