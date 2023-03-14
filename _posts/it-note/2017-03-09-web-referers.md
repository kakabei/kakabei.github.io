---
layout: post
title: 防盗链技术
date: 2017-03-09 21:08:12
categories: network
tags: network 系统编程
excerpt: 防止别人盗用你服务器的上资源
---


在《增长黑客》这个本书上，看到一个例子。作者用curl 在网上抓取了一个网站上用户的头像。这个网站对于图片的静态链接没有做防盗链。
以前有过这种开发。也疑惑过这种问题。 

## 防盗链的定义

此内容不在自己服务器上，而通过技术手段，绕过别人放广告有利益的最终页，直接在自己的有广告有利益的页面上向最终用户提供此内容。 常常是一些名不见经传的 小网站来盗取一些有实力的大网站的地址（比如一些音乐、图片、软件的下载地址）然后放置在自己的网站中，通过这种方法盗取大网站的空间和流量。

通俗的讲：就是你提供给自己的网站的静态链接。别人一样可能无条件用。图片，音频，文件，别人可以下载得到。

这个技术的本质就在于 http 上。表头字体有一个叫 **referer** 的。这个字段的作用是：当浏览器访问web服务器时，一般会带上`referer`，告诉服务器我是从哪个页面链接过来的。所以，我们可用检测这个链接是不是我们自己的服务器上的，如果不是，那可能就是别人来盗链接的了，就应该阻止掉。

## nginx 防盗链的配置

1、nginx针对文件类型的防盗链配置方法：
```sh
　　location ~* \.(gif|jpg|png|swf|flv|bmp)$ {
　　valid_referers none blocked *.php100.com php100.com;
　　if ($invalid_referer) {
　　     #rewrite ^/ http://www.php100.com/403.html;
　　     return 403;
　　     }     
　　}
```

这种方法是在 server 或者 location 段中加入：**valid_referers none blocked**，其中 none 表示空的来路，也就是直接访问，比如直接在浏览器打开一个文件，blocked 表示被防火墙标记过的来路，*.php100.com表示所有子域名。

2、nginx针对文件目录的防盗链配置方法：

```sh
　　location /img/ {
　　     root /data/img/;
　　     valid_referers none blocked *.php100.com php100.com;

　　     if ($invalid_referer) {
　　          rewrite ^/ http://www.php100.com/error.gif;
　　          #return 403;
　　     }     
　　}
```

如果不是用 nginx，就要自己去解析表头然后做出判断。

## 最后

用 `referer` 进行拦截只是比较简单的一种，还是可以被伪造。对于不是很重要的资源，用 `referer` 的方式倒是没有太大的问题。

但如果是比较重要的资源，那么最好要做登录鉴权系统管理。





