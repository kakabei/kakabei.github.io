---
layout: post
title: linux 进程间通信-有名管道（FIFO）
date: 2014-10-11 20:20:33
categories: Linux
tags: 系统编程
excerpt: linux process communication about FIFO
---

# 有名管道（FIFO）

命名管道也被称为 `FIFO` 文件，是一种特殊的文件。由于 linux 所有的事物都可以被视为文件，所以对命名管道的使用也就变得与文件操作非常统一。

# 创建命名管道

 用如下两个函数中的其中一个，可以创建命名管道。

 ```c
#include <sys/types.h>
#include <sys/stat.h>
int mkfifo(const char *filename, mode_t mode);
int mknod(const char *filename, mode_t mode | S_IFIFO, (dev_t)0);
```

 `filname` 是指文件名，而 `mode` 是指定文件的读写权限。`mknod` 是比较老的函数，而使用 `mkfifo` 函数更加简单和规范，所以建议用 `mkfifo`。

# 打开命名管道

和打开其他文件一样，可以用 open 来打开。通常有四种方法：

```c
open(const char *path, O_RDONLY);//1
open(const char *path, O_RDONLY | O_NONBLOCK);//2
open(const char *path, O_WRONLY);//3
open(const char *path, O_WRONLY | O_NONBLOCK);//4
```

**有两点要注意:**

a、程序不能以 `O_RDWR` (读写)模式打开 `FIFO` 文件进行读写操作，因其行为也未明确定义，因为如一个管道以读/写方式打开，进程就会读回自己的输出，同时我们通常使用 FIFO 只是为了单向的数据传递。

b、传递给 `open` 调用的是 `FIFO` 的路径，而不是正常的文件。如：`const char *fifo_name = "/tmp/my_fifo"; `

c、第二个参数中的选项 `O_NONBLOCK`，选项 `O_NONBLOCK` 表示非阻塞，加上这个选项后，表示 `open` 调用是非阻塞的，如果没有这个选项，则表示 `open` 调用是阻塞的。


# 阻塞问题

对于以只读方式 `O_RDONLY` 打开的 `FIFO` 文件，如果 `open` 调用是阻塞的（即第二个参数为 `O_RDONLY`），除非有一个进程以写方式打开同一个 `FIFO`，否则它不会返回；

如果 `open` 调用是非阻塞的的（即第二个参数为 `O_RDONLY | O_NONBLOCK` ），则即使没有其他进程以写方式打开同一个 FIFO 文件，open调用将成功并立即返回。

对于以只写方式（`O_WRONLY`）打开的 `FIFO` 文件，如果 `open` 调用是阻塞的（即第二个参数为 `O_WRONLY` ），open 调用将被阻塞，直到有一个进程以只读方式打开同一个 `FIFO` 文件为止；

如果 open 调用是非阻塞的（即第二个参数为 `O_WRONLY | O_NONBLOCK` ）总会立即返回，但如果没有其他进程以只读方式打开同一个 `FIFO` 文件，`open` 调用将返回- 1，并且 `FIFO` 也不会被打开。

# 使用 FIFO 实现进程间的通信

管道的写入端从一个文件读出数据，然后写入写管道。管道的读取端从管道读出后写到文件中。

```c
//写入端代码：fifowrite.c
#include <unistd.h>
#include <stdlib.h>
#include <fcntl.h>
#include <limits.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <stdio.h>
#include <string.h>
 
int main()
{
    const char *fifo_name = "/tmp/my_fifo";
    int pipe_fd = -1;
    int data_fd = -1;
    int res = 0;
    const int open_mode = O_WRONLY;
    int bytes_sent = 0;
    char buffer[PIPE_BUF + 1];
    int bytes_read = 0;
    if(access(fifo_name, F_OK) == -1)
    {
        printf ("Create the fifo pipe.\n");
        res = mkfifo(fifo_name, 0777);
        if(res != 0)
        {
            fprintf(stderr, "Could not create fifo %s\n", fifo_name);
            exit(EXIT_FAILURE);
        }
    }
    printf("Process %d opening FIFO O_WRONLY\n", getpid());
    pipe_fd = open(fifo_name, open_mode);
    printf("Process %d result %d\n", getpid(), pipe_fd);
 
    if(pipe_fd != -1)
    {
        bytes_read = 0;
        data_fd = open("Data.txt", O_RDONLY);
        if (data_fd == -1)
        {
            close(pipe_fd);
            fprintf (stderr, "Open file[Data.txt] failed\n");
            return -1;
        }
        bytes_read = read(data_fd, buffer, PIPE_BUF);
        buffer[bytes_read] = '\0';
        while(bytes_read > 0)
        {
            res = write(pipe_fd, buffer, bytes_read);
            if(res == -1)
            {
                fprintf(stderr, "Write error on pipe\n");
                exit(EXIT_FAILURE);
            }
            bytes_sent += res;
            bytes_read = read(data_fd, buffer, PIPE_BUF);
            buffer[bytes_read] = '\0';
        }
        close(pipe_fd);
        close(data_fd);
    }
    else
        exit(EXIT_FAILURE);
    printf("Process %d finished\n", getpid());
    exit(EXIT_SUCCESS);
}


//管道读取端 fiforead.c
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <fcntl.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <limits.h>
#include <string.h>
 
int main()
{
    const char *fifo_name = "/tmp/my_fifo";
    int pipe_fd = -1;
    int data_fd = -1;
    int res = 0;
    int open_mode = O_RDONLY;
    char buffer[PIPE_BUF + 1];
    int bytes_read = 0;
    int bytes_write = 0;
 
    memset(buffer, '\0', sizeof(buffer));
 
    printf("Process %d opening FIFO O_RDONLY\n", getpid());
    pipe_fd = open(fifo_name, open_mode);
    data_fd = open("DataFormFIFO.txt", O_WRONLY|O_CREAT, 0644);
    if (data_fd == -1)
    {
        fprintf(stderr, "Open file[DataFormFIFO.txt] failed\n");
        close(pipe_fd);
        return -1;
    }
    printf("Process %d result %d\n",getpid(), pipe_fd);
    if(pipe_fd != -1)
    {
        do
        {
            res = read(pipe_fd, buffer, PIPE_BUF);
            bytes_write = write(data_fd, buffer, res);
            bytes_read += res;
        }while(res > 0);
        close(pipe_fd);
        close(data_fd);
    }
    else
        exit(EXIT_FAILURE);
     printf("Process %d finished, %d bytes read\n", getpid(), bytes_read);
     exit(EXIT_SUCCESS);
}
```

# 命名管道的安全问题 

有一种情况是：一个 `FIFO` 文件，有多个进程同时向同一个 `FIFO` 文件写数据，而只有一个读 `FIFO` 进程在同一个 `FIFO` 文件中读取数据时，会发生数据块的相互交错。不同进程向一个 FI`FO 读进程发送数据是很普通的情况。这个问题的解决方法，就是让写操作的原子化。系统规定：在一个以 `O_WRONLY`（即阻塞方式）打开的 `FIFO` 中， 如果写入的数据长度小于等待 `PIPE_BUF` ，那么或者写入全部字节，或者一个字节都不写入。如果所有的写请求都是发往一个阻塞的 `FIFO` 的，并且每个写记请求的数据长度小于等于 `PIPE_BUF` 字节，系统就可以确保数据决不会交错在一起。








