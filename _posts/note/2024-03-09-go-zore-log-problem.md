---
layout: post
title: go-zero 日志输出的问题
date: 2024-03-09 9:08:12
categories: 工作日志
tags: goolang 
excerpt: 用 json 格式输出时有字段显示不正常，用 plain 格式时正常
---


go-zore 日志输出方式体验让我有一些不适。 

今天还遇到一个现象，日志输出如下：

```sh
{"@timestamp":"2024-03-09T15:48:08.026+08","caller":"server/Helper.go:164","content":"httpc.Do err userId[512] host[http://127.0.0.1:8888/xxxxx/xxxxxxList?offset=0\u0026limit=10\u0026order=desc\u0026sortby=create_time\u0026state=-1\u0026states=0,1,2,3,4\u0026biz_id=0\u0026agent_id=512]","level":"error","span":"66a2ba1882458680","trace":"c94c9ac8179690d592fc348b40666e16"}

```

出一些  `\u0026` 之类的。 但如果把 Encoding 设置为 plain，则正常。 

```sh 
2024-03-09T15:47:08.006+08       error  httpc.Do err userId[512] host[http://127.0.0.1:8888/xxxxx/xxxxxxList?offset=0&limit=10&order=desc&sortby=create_time&state=-1&states=0,1,2,3,4&biz_id=0&agent_id=512]      caller=server/Helper.go:164        trace=ca5e1ad81e7d0528f38b361c63a75f35  span=48b14f5d8bce91e9
```


