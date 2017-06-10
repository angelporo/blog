---
title: react-native构建踩坑
date: 2017-06-09 14:10:59
tags:
  - react-native
categories: react-native
---

## react-native run-ios 构建失败

> 记得昨天项目好好的,  今天run-ios的时候就不行了,  然后重新init之后还是不行.
> 在网上各种查找相似的问题,  最终无果, 自己猜想估计是版本问题

<!-- more -->

构建错误信息

```base
** BUILD FAILED **


The following commands produced analyzer issues:

    Analyze /Users/lcz/workspace/APP/temp/node_modules/react-native/ReactCommon/yoga/yoga/YGNodeList.c
    Analyze /Users/lcz/workspace/APP/temp/node_modules/react-native/ReactCommon/yoga/yoga/Yoga.c
(2 commands with analyzer issues)

The following build commands failed:
    PhaseScriptExecution Install\ Third\ Party /Users/lcz/workspace/APP/temp/ios/build/Build/Intermediates/React.build/Debug-iphonesimulator/double-conversion.build/Script-190EE32F1E6A43DE00A8543A.sh
(1 failure)

Installing build/Build/Products/Debug-iphonesimulator/temp.app
An error was encountered processing the command (domain=NSPOSIXErrorDomain, code=2):
Failed to install the requested application
An application bundle was not found at the provided path.
Provide a valid path to the desired application bundle.
Print: Entry, ":CFBundleIdentifier", Does Not Exist

Command failed: /usr/libexec/PlistBuddy -c Print:CFBundleIdentifier build/Build/Products/Debug-iphonesimulator/temp.app/Info.plist
Print: Entry, ":CFBundleIdentifier", Does Not Exist
```

当前版本
> 查看package.jsonn

```json
"react": "16.0.0-alpha.12",
"react-native": "0.45.0"
```

> 把node-modules rm之后 把package.json改成下面版本就可以了

```json
"react": "16.0.0-alpha.6",
"react-native": "0.44.3"
```

之后再`yarn install`
