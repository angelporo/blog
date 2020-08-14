---
layout: post
title: 使用Uglify2js来压缩你的js
date: 2018-02-01 10:43:10
tags:
    - js压缩
    - uglify-js
categories: "压缩"
---

例行惯例`npm install uglify-js -g` 全局安装`uglifyjs`

> [官网](https://github.com/mishoo/UglifyJS2)

安装好之后就可以使用了,  包括命令行使用和`nodejs`中`api`的使用

## 命令行

详情使用查看`uglifyjs -h`

下面是翻译后的`options`

- –source-map [string]，生成source map文件。
- –source-map-root [string], 指定生成source map的源文件位置。
- –source-map-url [string], 指定source map的网站访问地址。
- –source-map-include-sources，设置源文件被包含到source map中。
- –in-source-map，自定义source map，用于其他工具生成的source map。
- –screw-ie8, 用于生成完全兼容IE6-8的代码。
- –expr, 解析一个表达式或JSON。
- -p, –prefix [string], 跳过原始文件名的前缀部分，用于指定源文件、source map和输出文件的相对路径。
- -o, –output [string], 输出到文件。
- -b, –beautify [string], 输出带格式化的文件。
- -m, –mangle [string], 输出变量名替换后的文件。
- -r, –reserved [string], 保留变量名，排除mangle过程。
- -c, –compress [string], 输出压缩后的文件。
- -d, –define [string], 全局定义。
- -e, –enclose [string], 把所有代码合并到一个函数中，并提供一个可配置的参数列表。
- –comments [string], 增加注释参数，如@license、@preserve。
- –preamble [string], 增加注释描述。
- –stats, 显示运行状态。
- –acorn, 用Acorn做解析。
- –spidermonkey, 解析SpiderMonkey格式的文件，如JSON。
- –self, 把UglifyJS2做为依赖库一起打包。
- –wrap, 把所有代码合并到一个函数中。
- –export-all, 和–wrap一起使用，自动输出到全局环境。
- –lint, 显示环境的异常信息。
- -v, –verbose, 打印运行日志详细。
- -V, –version, 打印版本号。
- –noerr, 忽略错误命令行参数。

```base
➜ uglifyjs index.js -c -m > index.min.js
```

## api

官网查看`nodeapi`
