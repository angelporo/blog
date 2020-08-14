---
layout: post
title: 关于react-native中react-native-vector-icons包错误
date: 2020-04-27 21:14:17
tags:
    - react-native
---


{% asset_img 1.png 鸡尾酒排序图解 %}

前几天 ssr 被和谐, 然后换了一个配置后就不能跑了, 心中一万个mmp, 代码放着放着就坏了吗

出现的场景，使用SS设置了全局代理，然后在Android Studio中设置了代理，然后将Android Studio代理关闭之后，还是发现上不了网。出现了这个错误。

解决：

找到.gradle文件夹下的gradle.properties 文件，将末尾的代理取消掉。

注：gradle.properties并非是项目中的文件，而是本地gradle中的文件。比如mac电脑是用户目录下.gradle文件夹中。

mac下该文件为隐藏文件，大家自行改为可见文件。


注释点就可以
```
systemProp.https.proxyPort=1080
systemProp.http.proxyHost=127.0.0.1
systemProp.https.proxyHost=127.0.0.1
systemProp.http.proxyPort=1080
```
