---
layout: post
title: httpie嵌套json和发送数组
date: 2017-11-21 17:57:06
tags:
    - httpie
    - 接口测试json
categories: "httpie"
---


今天遇到一个需求是这样子的,

{% asset_img httpie.png httpie结构需求 %}

由于不好查,  所以这里贴出来.
官网结果:

```base
➜ imClientServer git:(login) http POST http://localhost:8080/sendmsg target_type=users target:='["u1", "u2", "u3"]' from=admin msg:='{"type":"txt","msg":"这才是消息内容"}'
```


后台接受到的结果映射json
```base
"content": {
    "from": "admin",
    "msg": {
        "msg": "这才是消息内容",
        "type": "txt"
    },
    "target": [
        "u1",
        "u2",
        "u3"
    ],
    "target_type": "users"
}
```
