---
layout: post
title: 常用命令 curl 
date: 2024-04-01 19:21:12
categories: 工具
tags: 工作提效
excerpt: Frequently Used Command Reference Curl 
---

# curl  常见的技巧

github 地址：[https://github.com/curl/curl](https://github.com/curl/curl)

**参数：**

```bash 
-o <file>    # --output: 写入文件
-u user:pass # --user: 验证
-v           # --verbose: 在操作期间使 curl 冗长
-vv          # 更冗长
-s           # --silent: 不显示进度表或错误
-S           # --show-error: 与 --silent (-sS) 一起使用时，显示错误但没有进度表
-i           # --include: 在输出中包含 HTTP 标头
-I           # --head: 仅标头
```

**数据：**

```bash 
# --data: HTTP post 数据
# URL 编码(例如，status="Hello")
-d 'data'

# --data 通过文件
-d @file

# --get: 通过 get 发送 -d 数据
-G
```

**头信息 Headers：**

```bash
-A <str>     # --user-agent
-b name=val  # --cookie

# 从 URL 的指定文件加载 cookie
-b, --cookie FILE
# 将 cookie 从 URL 保存到指定文件
-c, --cookie-jar FILE

-b FILE          # --cookie
-H "X-Foo: y"    # --header
--compressed     # 使用 deflate/gzip 

```

**ssl :** 

```bash
    --cacert <file>
    --capath <dir>
-E, --cert <cert>     # --cert: 客户端证书文件
    --cert-type       # der/pem/eng
-k, --insecure        # 对于自签名证书

```

**示例**

1、GET 请求

```bash 
curl -I https://www.baidu.com                                          # curl 发请求
curl -v -I https://www.baidu.com                                       # 带有详细信息的 curl 发请求
curl -X GET https://www.baidu.com                                      # 使用显式 http 方法进行 curl
curl --noproxy 127.0.0.1 http://www.stackoverflow.com                  # 没有 http 代理的 curl
curl --connect-timeout 10 -I -k https://www.baidu.com                  # curl 默认没有超时
curl --verbose --header "Host: www.mytest.com:8182" www.baidu.com      # curl 得到额外的标题
curl -k -v https://www.google.com                                      # curl 获取带有标题的响应

```

2、POST 请求

```bash
url -d "name=username&password=123456" <URL>    # curl 发请求
curl <URL> -H "content-type: application/json" -d "{ \"woof\": \"bark\"}"    # curl 发送 json
```

3、高级用法

```bash 
curl -L -s http://ipecho.net/plain, curl -L -s http://whatismijnip.nl            # 获取我的公共 IP撒··
curl -u $username:$password http://repo.dennyzhang.com/README.txt                # 带凭证的 curl
curl -v -F key1=value1 -F upload=@localfilename <URL>                            # curl 上传
curl -k -v --http2 https://www.google.com/                                       # 使用 http2 curl
curl -T cryptopp552.zip -u test:test ftp://10.32.99.187/                         # url ftp 上传
curl -u test:test ftp://10.32.99.187/cryptopp552.zip -o cryptopp552.zip    curl  # ftp 下载
curl -v -u admin:admin123 --upload-file package1.zip http://mysever:8081/dir/package1.zip    # 使用凭证 curl 上传
```

4、查询当前机器的出口IP

```bash 
 (base) C:\tools> curl cip.cc
IP      : 121.12.81.78
地址    : 中国  广东  江门
运营商  : 电信

数据二  : 广东省深圳市 | 电信

数据三  : 中国广东省深圳市 | 电信
URL     : http://www.cip.cc/121.12.81.78

```



