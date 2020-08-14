---
layout: post
title: 使用jsdoc通过注释来自动生成js文档
date: 2018-02-27 14:21:54
tags:
    - jsdoc
    - js文档
categories: "api生成器"
---

如果要维护一个项目,  文档是必不可少的, 如果没有文档, 你同事看起来就会减少效率.

好处不觉得没有必要写了, 好多公司都需要一个文档来注释项目的api

这里`js`我们选着[jsdoc](https://github.com/jsdoc3/jsdoc)来生成

安装

`npm install -g jsdoc`

命令行中使用
`jsdoc youJavascriptFile.js`

项目中安装
`npm install --save-dev jsdoc`

安装在项目中我们必须要调用`./node_modules/.bin/jsdoc youJavascriptFile.js`

一般生成好的文件在`out`目录下, 

## Examples

这里具体要说的是几个注释模板, 让你生成后的代码api文档更美观

```javascript
/** Class representing a point. */
class Point {
    /**
     * Create a point.
     * @param {number} x - The x value.
     * @param {number} y - The y value.
     */
    constructor(x, y) {
        // ...
    }

    /**
     * Get the x value.
     * @return {number} The x value.
     */
    getX() {
        // ...
    }

    /**
     * Get the y value.
     * @return {number} The y value.
     */
    getY() {
        // ...
    }

    /**
     * Convert a string containing two comma-separated numbers into a point.
     * @param {string} str - The string containing two comma-separated numbers.
     * @return {Point} A Point object.
     */
    static fromString(str) {
        // ...
    }
}

/** @module color/mixer */

/** The name of the module. */
export const name = 'mixer';

/** The most recent blended color. */
export var lastColor = null;

/**
 * Blend two colors together.
 * @param {string} color1 - The first color, in hexadecimal format.
 * @param {string} color2 - The second color, in hexadecimal format.
 * @return {string} The blended color.
 */
export function blend(color1, color2) {}

// convert color to array of RGB values (0-255)
function rgbify(color) {}

export {
    /**
     * Get the red, green, and blue values of a color.
     * @function
     * @param {string} color - A color, in hexadecimal format.
     * @returns {Array.<number>} An array of the red, green, and blue values,
     * each ranging from 0 to 255.
     */
    rgbify as toRgb
}

```

## Tags

具体可以看官方文档, [jsdoc](http://usejsdoc.org/)

妈妈再也不用担心文档需要自己手写了!!!
