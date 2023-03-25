---
layout: post
title: golang 笔记 网络编程
date: 2018-03-04 07:21:12
categories:  go 
tags: go
excerpt: go 标准库里提供的 net 包，支持基于 IP 层、TCP/UDP 等网络操作
---

Go 语言标准库里提供的 net 包，支持基于 IP 层、TCP/UDP 层及更高层面(如 HTTP、FTP、SMTP )的网络操作，其中用于 IP 层的称为 Raw Socket。

# 一、socket 编程

Go 语言标准库对此过程进行了抽象和封装。无论我们期望使用什么协议建立什么形式的连接，都只需要调用`net.Dial()`即可。

## 1.1 Dial()函数

`Dial()` 函数的原型如下:

```go 
func Dial(net, addr string) (Conn, error)
```

其中 net 参数是网络协议的名字，addr 参数是 IP 地址或域名，而端口号以“:”的形式跟随在地址 或域名的后面，端口号可选。如果连接成功，返回连接对象，否则返回 error。

```go
// TCP链接:
conn, err := net.Dial("tcp", "192.168.0.10:2100")

// UDP链接:
conn, err := net.Dial("udp", "192.168.0.12:975")

// ICMP 链接(使用协议名称):
conn, err := net.Dial("ip4:icmp", "www.baidu.com")

// ICMP链接(使用协议编号):
conn, err := net.Dial("ip4:1", "10.0.0.3")
```

目前，`Dial()` 函数支持如下几种网络协议: "tcp"、"tcp4"(仅限IPv4)、"tcp6"(仅限 IPv6)、"udp"、"udp4"(仅限IPv4)、"udp6"(仅限IPv6)、"ip"、"ip4"(仅限IPv4)和"ip6"(仅限IPv6)。

在成功建立连接后，我们就可以进行数据的发送和接收。发送数据时，使用 conn 的 `Write()` 成员方法，接收数据时使用 Read()方法。

## 1.2 TCP示例程序

```go 
package main
import ("net"
        "os"
        "bytes"
        "fmt"
)

func main() {

    if len(os.Args) != 2 {
        fmt.Fprintf(os.Stderr, "Usage: %s host:port", os.Args[0])
        os.Exit(1) 
    }

    service := os.Args[1]
    conn, err := net.Dial("tcp", service)
    checkError(err)

    _, err = conn.Write([]byte("HEAD / HTTP/1.0\r\n\r\n")) 
    checkError(err)

    result, err := readFully(conn)
    checkError(err)
    
    fmt.Println(string(result)) 
    os.Exit(0)
}

func checkError(err error) {

    if err != nil {
      fmt.Fprintf(os.Stderr, "Fatal error: %s", err.Error())
      os.Exit(1)
    } 
}

func readFully(conn net.Conn) ([]byte, error) {
     defer conn.Close()
    result := bytes.NewBuffer(nil) 
    var buf [512]byte
    for {
        n, err := conn.Read(buf[0:]) 
        result.Write(buf[0:n])
        if err != nil {
            if err == io.EOF { 
                break
            }
            return nil, err 
        }
    }

    return result.Bytes(), nil 
}
```
## 1.3 更丰富的网络通信

`Dial()` 函数是对 `DialTCP()`、`DialUDP()`、`DialIP()` 和 `DialUnix()` 的封装。我们也可以直接调用这些函数，它们的功能是一致的。这些函数的原型如下:

```go 
func DialTCP(net string, laddr, raddr *TCPAddr) (c *TCPConn, err error) 
func DialUDP(net string, laddr, raddr *UDPAddr) (c *UDPConn, err error) 
func DialIP(netProto string, laddr, raddr *IPAddr) (*IPConn, error)
func DialUnix(net string, laddr, raddr *UnixAddr) (c *UnixConn, err error)
```

验证IP地址有效性；

```go 
func net.ParseIP()
```

根据域名查找IP的代码如下:

```go
func ResolveIPAddr(net, addr string) (*IPAddr, error)
func LookupHost(name string) (cname string, addrs []string, err error);
```

# 二、HTTP 编程

Go 语言标准库内建提供了net/http包，涵盖了HTTP客户端和服务端的具体实现。使用 `net/http` 包，我们可以很方便地编写 HTTP 客户端或服务端的程序。

## 2.1 HTTP 客户端

Go 内置的 `net/http` 包提供了最简洁的 HTTP 客户端实现，我们无需借助第三方网络通信库 (比如 libcurl )就可以直接使用 HTTP 中用得最多的 GET 和 POST 方式请求数据。

### 基本方法

`net/http` 包的 Client 类型提供了如下几个方法，让我们可以用最简洁的方式实现 HTTP 请求:

```go 
func (c *Client) Get(url string) (r *Response, err error)
func (c *Client) Post(url string, bodyType string, body io.Reader) (r *Response, err error)
func (c *Client) PostForm(url string, data url.Values) (r *Response, err error) 
func (c *Client) Head(url string) (r *Response, err error)
func (c *Client) Do(req *Request) (resp *Response, err error)
```

**http.Get()**

要请求一个资源，只需调用 `http.Get()` 方法(等价于 `http.DefaultClient.Get()` )即可。

```go
resp, err := http.Get("http://example.com/") 
if err != nil {
    // 处理错误 ...
    return
}

defer resp.Body.close()
io.Copy(os.Stdout, resp.Body)
```

**http.Post()**

调用`http.Post()`方法并依次传递下面的3个参数即可

- 请求的目标 URL
- 将要 POST 数据的资源类型( MIMEType )
- 数据的比特流(`[]byte` 形式)

```go
resp, err := http.Post("http://example.com/upload", "image/jpeg", &imageDataBuf) 
if err != nil {
    // 处理错误
    return
}
if resp.StatusCode != http.StatusOK { // 处理错误
    return
}
// ...
```

**http.PostForm()** 

实现了标准编码格式为`application/x-www-form-urlencoded`的表单提交

```go
resp, err := http.PostForm("http://example.com/posts", 
                            url.Values{"title": {"article title"}, "content": {"article body"}})
if err != nil { // 处理错误
    return
}
// ...
```

**http.Head()**

HTTP 中的 Head 请求方式表明只请求目标 URL 的头部信息，即 HTTP Header 而不返回 HTTP Body。

```go
resp, err := http.Head("http://example.com/")
```

**(*http.Client).Do()**

在多数情况下，`http.Get()` 和 `http.PostForm()` 就可以满足需求，但是如果我们发起的 HTTP 请求需要更多的定制信息，我们希望设定一些自定义的 Http Header 字段。

- 设定自定义的"User-Agent"。
- 传递 Cookie

```go
req, err := http.NewRequest("GET", "http://example.com", nil) // ...
req.Header.Add("User-Agent", "Gobook Custom User-Agent")

// ...
client := &http.Client{ 
//... 
}

resp, err := client.Do(req)
// ...
```

## 2.2 HTTP 服务端

### 处理 HTTP 请求

使用 `net/http` 包提供的 `http.ListenAndServe()` 方法，可以在指定的地址进行监听， 开启一个 HTTP，服务端该方法的原型如下:

```go
func ListenAndServe(addr string, handler Handler) error
```
该方法用于在指定的 TCP 网络地址 addr 进行监听，然后调用服务端处理程序来处理传入的连接请求。该方法有两个参数: 

1、 addr 即监听地址;

2、表示服务端处理程序，通常为空，这意味着服务端调用 `http.DefaultServeMux` 进行处理，而服务端编写的业务逻 辑处理程序 `http.Handle()` 或 `http.HandleFunc()` 默认注入 `http.DefaultServeMux` 中。

```go
http.Handle("/foo", fooHandler)
http.HandleFunc("/bar", func(w http.ResponseWriter, r *http.Request) {
                        fmt.Fprintf(w, "Hello, %q", html.EscapeString(r.URL.Path))
                        }) 
log.Fatal(http.ListenAndServe(":8080", nil))
```
如果想更多地控制服务端的行为，可以自定义 `http.Server`

```go
s := &http.Server{
    Addr:               ":8080",
    Handler:            myHandler,
    ReadTimeout:        10*time.Second,
    WriteTimeout:       10*time.Second,
    MaxHeaderBytes: 1 << 20,
}

log.Fatal(s.ListenAndServe())
```
### 处理 HTTPS 请求

net/http 包还提供  `http.ListenAndServeTLS()` 方法，用于处理 HTTPS 连接请求:

```go
func ListenAndServeTLS(addr string, certFile string, keyFile string, handler Handler) error
```
certFile 对应 SSL 证书文件存放路径，keyFile 对应证书私钥文件路径。如果证书是由证书颁发机构签署的，certFile 参数指定的路径必须是存放在服务器上的经由 CA 认证过的 SSL 证书。

```go
http.Handle("/foo", fooHandler)
http.HandleFunc("/bar", func(w http.ResponseWriter, r *http.Request) {
                                fmt.Fprintf(w, "Hello, %q", html.EscapeString(r.URL.Path)) 
                            })

log.Fatal(http.ListenAndServeTLS(":10443", "cert.pem", "key.pem", nil))
```


# 三、Json 处理

Go 语言内建对 JSON 的 支持。使用 Go 语言内置的 `encoding/json` 标准库。

## 3.1 编码为JSON格式

使用 `json.Marshal()` 函数可以对一组数据进行JSON格式的编码。`json.Marshal()` 函数的声明如下:

```go 
func Marshal(v interface{}) ([]byte, error)
```

```go 
type Book struct { 
    Title string
    Authors []string 
    Publisher string 
    IsPublished bool Price float
}

gobook := Book{ "Go语言编程",
                ["XuShiwei", "HughLv", "Pandaman", "GuaguaSong", "HanTuo", "BertYuan", "XuDaoli"],
                "ituring.com.cn", true,
                9.99
            }

b, err := json.Marshal(gobook) // 将 gobook 实例生成一段 JSON 格式的文本
```

```go
b == []byte(`{
"Title": "Go语言编程",
"Authors": ["XuShiwei", "HughLv", "Pandaman", "GuaguaSong", "HanTuo", "BertYuan",
             "XuDaoli"],
         "Publisher": "ituring.com.cn",
         "IsPublished": true,
         "Price": 9.99
}`)
```

Go 语言的大多数数据类型都可以转化为有效的 JSON 文本，但 channel、complex 和函数这几种类型除外。

如果转化前的数据结构中出现指针，那么将会转化指针所指向的值，如果指针指向的是零值， 那么 null 将作为转化后的结果输出。

在 Go 中，JSON 转化前后的数据类型映射如下：

- 布尔值转化为 JSON 后还是布尔类型。
- 浮点数和整型会被转化为 JSON 里边的常规数字。
- 字符串将以 UTF-8 编码转化输出为 Unicode 字符集的字符串，特殊字符比如 < 将会被转义为 \u003c。
- 数组和切片会转化为 JSON 里边的数组，但 `[]byte` 类型的值将会被转化为 Base64 编码后的字符串，slice 类型的零值会被转化为 null。
- 结构体会转化为 JSON 对象，并且只有结构体里边以大写字母开头的可被导出的字段才会被转化输出，而这些可导出的字段会作为JSON对象的字符串索引。
- 转化一个map类型的数据结构时，该数据的类型必须是 `map[string]T` (T 可以是 `encoding/json` 包支持的任意数据类型)。

## 3.2 解码 JSON 数据

可以使用 `json.Unmarshal()` 函数将JSON格式的文本解码为Go里边预期的数据结构。 `json.Unmarshal()` 函数的原型如下:

```go 
func Unmarshal(data []byte, v interface{}) error
```
如：

```go 
var book Book

err := json.Unmarshal(b, &book)

book := Book{ "Go语言编程",
                ["XuShiwei", "HughLv", "Pandaman", "GuaguaSong", "HanTuo", "BertYuan", "XuDaoli"],
                "ituring.com.cn", 
                true,
                9.99
            }
```

## 4.3 解码未知结构的 JSON 数据

要解码一段未知结构的 JSON，只需将这段 JSON 数据解码输出到一个空接口即可。

Go 的标准库 `encoding/json` 包中，允许使用 `map[string]interface{}` 和`[]interface{}` 类型的值来分别存放未知结构的 JSON 对象或数组。

```go 
b := []byte(`{
            "Title": "Go语言编程",
            "Authors": ["XuShiwei", "HughLv", "Pandaman", "GuaguaSong", "HanTuo", "BertYuan","XuDaoli"],
            "Publisher": "ituring.com.cn",
            "IsPublished": true,
            "Price": 9.99,
            "Sales": 1000000
}`)

var r interface{}
err := json.Unmarshal(b, &r)
```
最终 r 将会是一个键值对的 `map[string]interface{}` 结构:

```go
map[string]interface{}{
    "Title": "Go语言编程",
    "Authors": ["XuShiwei", "HughLv", "Pandaman", "GuaguaSong", "HanTuo", "BertYuan", "XuDaoli"],
    "Publisher": "ituring.com.cn",
    "IsPublished": true,
    "Price": 9.99,
    "Sales": 1000000
}
```

## 3.4 JSON 的流式读写

Go 内建的 `encoding/json` 包还提供 Decoder 和 Encoder 两个类型，用于支持JSON数据的 流式读写，并提供 `NewDecoder()`和`NewEncoder()`两个函数来便于具体实现:

```go
func NewDecoder(r io.Reader) *Decoder 
func NewEncoder(w io.Writer) *Encoder
```
如：

```go
package main import (
        "encoding/json"
        "log"
        "os"
)

func main() {

    dec := json.NewDecoder(os.Stdin) 
    enc := json.NewEncoder(os.Stdout) 
    for {
        var v map[string]interface{}
        if err := dec.Decode(&v); err != nil {
            log.Println(err)
            return
        }

        for k := range v {
            if k != "Title" { 
                v[k] = nil, false
            } 
        }

        if err := enc.Encode(&v); err != nil { 
            log.Println(err)
        }
    }
}
```
