---
title: react-native中android环境问题
date: 2017-06-13 10:37:51
tags:
  - react-native
  - RN-android
categories: "react-native踩坑记"
comments: false
---

### react-native 中android环境总是报错

记得之前使用linux的时候搭建android环境就不太好弄, PATH里面总是会有版本差别问题, 这次使用mac也没有避免, 只是这次问题貌似比上次复制

<!-- more -->

具体问题之后解释

```base
* What went wrong:
A problem occurred evaluating project ':app'.
> java.lang.UnsupportedClassVersionError: com/android/build/gradle/AppPlugin : Unsupported major.minor version 52.0

* Try:
Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output.

BUILD FAILED

Total time: 11.564 secs
Could not install the app on the device, read the error above for details.
Make sure you have an Android emulator running or a device connected and have
set up your Android development environment:
https://facebook.github.io/react-native/docs/android-setup.html
```

### 解决办法

> 项目主要针对自己环境解决对应bug, 如果不知道, 请run `react-native run-android debug`来查看具体出错原因
> 由于环境中使用的java8 , 可能在build的时候buildToolsVersion版本使用之前的23不行, 所以修改成下面的样子就可以运行了


```javascritp
android: {
    compileSdkVersion 23
    buildToolsVersion "25.0.0"
    ...
}
```
