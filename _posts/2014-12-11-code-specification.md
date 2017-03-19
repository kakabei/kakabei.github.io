---
layout: post
title: 后台编码规
date: 2014-12-11 21:12:15
categories: 编程语言
tags: 系统编程
excerpt: c/c++ 语言编程行为规范，对自己而言，对他人无效。
---


#### 1.文件编码

源文件(.h/.c文件)编码一律使用UTF-8 (不是UTF-8 +BOM)。

可以使用EditPlus来修改文件编码。

#### 2.tab和空格

代码中不允许使用tab，只有空格，可以把编辑器中的tab设置为自动转换为4个空格。

#### 3.变量命名

a)局部变量，采用骆驼式命名: yourVarriableName, 如：

```c
UserInfo  userInfo;
UserInfo* userInfo;
```

b)全局变量，骆驼式命名前加 g_, 如：

```c
UserInfo  g_userInfo;
UserInfo* g_userInfo;
```

变量的前缀，不能使用代表数据类型的标记，如下命名都是不允许的：

```c
char strUserName[64];        // 前缀为str
UserInfo arryUserList[100];  // 前缀为arry
int iIndex;                  // 前缀为i
```

#### 4.函数命名

文件名_xxx_yyy, 且xxx和yyy皆为小写, 比如user.h/c文件中的函数:

```c
int user_login();
int user_logout();
```

#### 5.枚举命名

枚举名使用typedef定义别名，格式为骆驼式，且首字母大写。
枚举定义，全部使用大写字母，单词之间用下划线连接。

```c
typdef enum
{
    YOUR_ENM_ITEM,
    YOUR_ENM_ITEM,
}
YourEnmName;
```

如:

```c
typedef enum
{
    USER_STATUS_NA,
    USER_STATUS_LOGING,
    USER_STATTUS_LOGOUT,
}
UserStatus;
```

#### 6.结构命名

结构名使用typedef定义别名，格式为骆驼式，且首字母大写。

结构中的字段名命名，使用骆驼式，首字母小写。

结构中的字段，要对齐，并且在字段后加注释, 且前后注释要对齐。

```c
typedef struct
{
    Type  fieldName;
    Type  fieldName;
}
YourStructName;
```

如：

```c
/////
typedef struct
{
    /////
    ListNode        onlineNode;    // 在线列表节点
    ListNode        chatNode;      // 聊天队列节点
    
    /////
    int             sock;          // websock fd
    time_t          lastTime       // time
    
    /////
    UserInfo       info;          //  用户数据
    
}
User;

```
#### 7.宏命名

宏命名，全部使用大写，单词之间用下划线连接，如：

```c
#define  USER_NUM_MAX
```

#### 8.宏表达式使用小括号原则

1)宏定义的是单个数值，不用加小括号，如：

```c
#define USER_NUM_MAX  100
```

2)宏定义是个表达式，一定要在整个表达式加小括号:

```c
#define USER_NUM_MAX  (100+20)
#define USER_LIST_MAX  (USER_NUM_MAX * 100)
```

#### 9.注释方式

1)表明同一个意图的几行代码之间要加/////(5个反斜杠)，且/////之前起码要有一个空行：

```c
    // 初始化libwebsocket的日志接口
    log_init();

    ///////
    ConnCtx* connCtx = conn_ctx_get();

    ///////
    if (NULL == connCtx->conf) {
        fprintf(stderr, "conf is NULL\n");
        return -1;
    }

    ///////
    if (NULL == connCtx->conf->log) {
        fprintf(stderr, "log conf is NULL\n");
        return -1;
    }

    ///////
    init_log(
        connCtx->conf->log->fileName,
        connCtx->conf->log->fileDir,
        connCtx->conf->log->fileNum,
        connCtx->conf->log->fileSize
    );
```
2)如果需要注释，则在//(两个反斜杠),  +  (1个空格)  + 文字描述:

```c
    // zmq: data
    fd_len = sizeof(fd_data);
    rc = zmq_getsockopt(connCtx->zmq.socks.data.fd, ZMQ_FD, &fd_data, &fd_len);

    if (-1 == rc) {
        fprintf(stderr, "fatal: zmq_data socket not ready: %s\n", strerror(errno));
        return -1;
    }

    // evloop是在conn_websock_init()中初始化的
    struct ev_loop *evloop = libwebsocket_libev_loop(webctx);

```

3)除了文件头部可以使用多行注释` /* */`，其他地方禁用`/* */`, 只能使用单行注释

```c
/*
 * desc: 客户端weboskc连接处理
 * author: youngs
 */

#ifndef CONN_CONN_H
#define CONN_CONN_H

#include "websock.h"
#include "conn_ctx.h"
```

#### 10.换行方式

1) if格式

```c
if  (condition) 
{
    int var1 = 0;
    int var2 = 0;
}
```

格式: if + 空格 + () + 空格 + {

2) for格式

```c
for ()
{
   int var1 = 0;
   int var2 = 0;
}
```

格式: for + 空格 + ()

3) while格式

```c
while
{
    int var1 = 0;
    int var2 = 0;
}
```

4) do{}while格式

```c
do
{
    int var1 = 0;
    int var2 = 0;
}
while();
```

5) 如果for 里面的代码只有一行，允许{跟在后面，如

```c
for () {
int var = 0;
}
```
#### 11.接口最小原则

只在当前.c文件中使用的函数，都要定义为static函数，不能把函数原型放入.h中。

#### 12.禁止在头文件中定义变量

头文件中只能放以下内容:

1)结构声明

2)函数原型

3)枚举定义

4)宏定义



