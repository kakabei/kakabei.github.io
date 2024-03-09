---
layout: post
title: go-zero httpc.Do post 数据因 struct 继承导致的失败
date: 2024-03-09 9:08:12
categories: 工作日志
tags: 工作日志 
excerpt: golang 结构体的"继承"导致 go-zero  的 httpc.Do()发送失败的问题
---


工作时遇到的一个问题。

背景大概可以简化为：请求不同服务过来的数据后做聚合，然后转发另一个服务。 

对于数据的处理习惯性就是：

```go 

// data from server A 
type BaseB struct {
	Offset int64 `json:"offset"`
	Limit  int64 `json:"limit"`
}

// data from server B 
type BaseB struct {
	Id    int64 `json:"id"`
    Name string `json:"name"`
	  
}

// send to server C
type BaseC  struct {
	BaseA
	BaseB
    Addr  string `json:"addr"`
}

baseC := new(BaseC)
baseC.Offset = 199
baseC.Id = 1
baseC.Name = "kane"
baseC.Addr = "guangdong"
fmt.Printf("baseC ----- > %+v\n", baseC)

```

这里对 BaseC 成员变量的操作和它继承的BaseB、BaseB 的成员一样，都按 BaseC的成员一样处理。 

但是，在 fmt.Printf baseC 时，baseC 的结构却有点和想像中不一样。而是：

```sh 
 baseC ----- > &{BaseA:{Offset:199 Limit:0} BaseB:{Id:1 Name:kane} Addr:guangdong}
```
中包含了 "BaseA" "BaseB"。 

多想一步，把这个结构体转 json 输出，如下：

```sh
baseCByte ----- > {"offset":199,"limit":0,"id":1,"name":"kane","addr":"guangdong"}
```

想像上面 baseC 数据的结构应该是这样的。但事实却不一样。 

自己 golang 的底层知识不够导致的。 

go-zore 的  `httpc.Do()`在 post 数据带上设置头信息时，[https://go-zero.dev/docs/tutorials/http/client/index](https://go-zero.dev/docs/tutorials/http/client/index)

```go
func main() {
    flag.Parse()

    req := Request{
        Node:   "foo",
        Header: "foo-header",
        Foo: "foo",
        Bar: "bar",
    }
    resp, err := httpc.Do(context.Background(), http.MethodPost, *domain+"/nodes/:node", req)
    if err != nil {
        fmt.Println(err)
        return
    }

    io.Copy(os.Stdout, resp.Body)
}
```
当  `Request` 用了继承的方式：
如：

```go
type Body struct {
    Foo    string `json:"foo"`
    Bar    string `json:"bar"`
}
    
type Request struct {
    Body
    Header string `header:"X-Header"`
}
```

发送过去之后， Body 成了空的。估计就是继承无法正确解析的原因。 