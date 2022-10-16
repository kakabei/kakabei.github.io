---
layout: post
title: unix 文件IO
date: 2014-10-13 20:08:12
categories: Linux
tags: 系统编程
excerpt: unix file I/O
---

### 一 不带缓冲的I/O
1、大多数UNIX文件I/O只需用到5个函数：open、read、write、lseek以及close。然后说明不同缓存器长度对read和write函数的影响。
2、这些函数经常被称为不带缓冲的I/O。不带缓冲的指的是每个read和write都调用内核中的一个系统调用。

### 二 文件描述符
1、对于内核而言，所有打开文件都由文件描述符引用。文件描述符是一个非负整数。
2、当打开一个现存文件或创建一个新文件时，内核向进程返回一个文件描述符。当读、写一个文件时，用open或creat返回的文件描述符标识该文件，将其作为参数传送给 read或write。
3、按照惯例，UNIX shell使文件描述符 0与进程的标准输入相结合，文件描述符 1与标准输出相结合，文件描述符 2与标准出错输出相结合。
4、在POSIX.1应用程序中，幻数0、1、2应被代换成符号常数STDIN_FILENO、STDOUT_FILENO和STDERR_FILENO。这些常数都定义在头文件<unistd.h>中。文件描述符的范围是0~OPEN_MAX（64个）。现在很多系统则将其增加至 63。

### 三 常用的函数

5个函数：open、read、write、lseek以及close

1、open函数

open函数打开或创建一个文件

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
int open(const char * pathname, int oflag,.../*, mode_t  mode */ ) ;
```

返回：若成功为文件描述符，若出错为- 1

pathname是要打开或创建的文件的名字。

oflag参数可用来说明此函数的多个选择项。用下
列一个或多个常数进行或运算构成 oflag参数(这些常数定义在<fcntl.h>头文件中)：
* O_RDONLY  只读打开。
* O_WRONLY  只写打开。
* O_RDWR  读、写打开。
在这三个常数中应当只指定一个。

下列常数则是可选择的：
* O_APPEND         每次写时都加到文件的尾端。
* O_CREAT            若此文件不存在则创建它。使用此选择项时，需同时说明第三个参数mode，用其说明该新文件的存取许可权位。
* O_EXCL              如果同时指定了O_CREAT，而文件已经存在，则出错。这可测试一个文件是否存在，如果不存在则创建此文件成为一个原子操作。
* O_TRUNC          如果此文件存在，而且为只读或只写成功打开，则将其长度截短为 0。
* O_NOCTTY        如果pathname指的是终端设备，则不将此设备分配作为此进程的控制终端。
* O_NONBLOCK  如果pathname指的是一个FIFO、一个块特殊文件或一个字符特殊文件，则此选择项为此文件的本次打开操作和后续的 I/O操作设置非阻塞方式。 
* O_SYNC            使每次write都等到物理I/O操作完成。
  
2、creat函数

用creat函数创建一个新文件

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
int creat(const char * pathname, mode_t  mode) ;
```

返回：若成功为只写打开的文件描述符，若出错为- 1

这个函数等于：

```c
open (pathname, O_WRONLY ｜O_CREAT｜O_TRUNC,  mode) ;
```

不知道这个函数现在存在的意义是什么？

3、close函数

close函数关闭一个打开文件

```c
#include <unistd.h>
int close (int  filedes)；
```

返回：若成功为 0，若出错为- 1
关闭一个文件时也释放该进程加在该文件上的所有记录锁。
当一个进程终止时，它所有的打开文件都由内核自动关闭。很多程序都使用这一功能而不显式地用close关闭打开的文件。（这个方式不是一个好习惯，建议不要用。）
 
4、lseek 函数

每个打开文件都有一个与其相关联的“当前文件位移量“。它是一个非负整数，用以度量从文件开始处计算的字节数。按系统默认，当打开一个文件时，除非指定 O_APPEND选择项，否则该位移量被设置为0。

```c
#include <sys/types.h>
#include <unistd.h>
off_t lseek(int  filedes, off_t  offset, int whence) ;
```

返回：若成功为新的文件位移，若出错为 	-1
对参数offset  的解释与参数whence的值有关。
* 若whence是SEEK_SET，则将该文件的位移量设置为距文件开始处 offset 个字节。
* 若whence是SEEK_CUR，则将该文件的位移量设置为其当前值加offset, offset可为正或负。
* 若whence是SEEK_END，则将该文件的位移量设置为文件长度加offset, offset可为正或负。

若lseek成功执行，则返回新的文件位移量。
判断文件是否可能设置偏移量的方法：

```c
off_t currpos;
currpos = lseek(fd, 0, SEEK_CUR);
```

这种方法也可用来确定所涉及的文件是否可以设置位移量。如果文件描述符引用的是一个管道或FIFO，则lseek返回-1，并将errno设置为EPIPE。
文件位移量可以大于文件的当前长度，在这种情况下，对该文件的下一次写将延长该文件，并在文件中构成一个空调，这一点是允许的。位于文件中但没有写过的字节都被读为 0。

5、read函数

用read 函数从打开文件中读数据。

```c
#include <unistd.h>
ssize_t read(int  filedes, void * buff, size_t  nbytes) ;
```

返回：读到的字节数，若已到文件尾为 0，若出错为- 1
如read成功，则返回读到的字节数。如已到达文件的尾端，则返回 0。
有多种情况可使实际读到的字节数少于要求读字节数：
* 读普通文件时，在读到要求字节数之前已到达了文件尾端。例如，若在到达文件尾端之前还有30个字节，而要求读100个字节，则read返回30，下一次再调用read时，它将返回 0 (文件尾端)。
* 当从终端设备读时，通常一次最多读一行。
* 当从网络读时，网络中的缓冲机构可能造成返回值小于所要求读的字节数。
* 某些面向记录的设备，例如磁带，一次最多返回一个记录。
读操作从文件的当前位移量处开始，在成功返回之前，该位移量增加实际读得的字节数。
  
6、write函数

用write函数向打开文件写数据。

```c
#include <unistd.h>
ssize_t write(int  filedes, const void * buff, size_t  nbytes) ;
```

返回：若成功为已写的字节数，若出错为- 1
其返回值通常与参数 nbytes的值不同，否则表示出错。 write出错的一个常见原因是：磁盘已写满，或者超过了对一个给定进程的文件长度限制 。
对于普通文件，写操作从文件的当前位移量处开始。如果在打开该文件时，指定了O_APPEND选择项，则在每次写操作之前，将文件位移量设置在文件的当前结尾处。在一次成功写之后，该文件位移量增加实际写的字节数。

 7、文件共享

UNIX支持在不同进程间共享打开文件。
内核使用了三种数据结构，它们之间的关系决定了在文件共享方面一个进程对另一个进程
可能产生的影响。

(1)每个进程在进程表中都有一个记录项，每个记录项中有一张打开文件描述符表，可将其视为一个矢量，每个描述符占用一项。与每个文件描述符相关联的是:
  - 文件描述符标志。
  - 指向一个文件表项的指针.

(2)内核为所有打开文件维持一张文件表。每个文件表项包含：
  - 文件状态标志(读、写、增写、同步、非阻塞等 )。
  - 当前文件位移量。
  - 指向该文件v节点表项的指针。

(3)每个打开文件（或设备）都有一个v节点结构。 
  - v节点包含了文件类型和对此文件进行各种操作的函数的指针信息。    
  - 对于大多数文件，v 节点还包含了该文件的i节点（索引节点）。
  - 这些信息是在打开文件时从盘上读入内存的，所以所有关于文件的信息都是快速可供使用的。例如， i 节点包含了文件的所有者、文件长度、文件所在的设备、指向文件在盘上所使用的实际数据块的指针等等。
  - Linux 没有使用v节点，而是使用了通用i节点结构。虽然两种实现有所不同，但在概念上，v节点与i节点是一样的。两者都指向文件系统特有的i节点结构。

三个数据结构表的关系如下图：

![](/assets/linux/linux-io-file-1.png) 

8、dup和dup2函数

这两个函数都可用来复制一个现存的文件描述符。

```c
#include <unistd.h>
int dup(int  filedes) ;
int dup2(int  filedes, int filedes2) ;
```

两函数的返回：若成功为新的文件描述符，若出错为- 1
(1)由dup返回的新文件描述符一定是当前可用文件描述符中的最小数值。
(2)用 dup2则可以用filedes2参数指定新描述符的数值。如果 filedes2已经打开，则先将其关闭。如若 filedes等于filedes2，则dup2返回filedes2，而不关闭它。
(3)这些函数返回的新文件描述符与参数 filedes共享同一个文件表项。
(4)newfd = dup(1);当此函数开始执行时，假定下一个可用的描述符是 3 (这是非常有可能的，因为 0，1和2由shell打开)。因为两个描述符指向同一文件表项，所以它们共享同一文件状态标志 (读、写、添写等 )以及同一当前文件位移量。
(5)复制一个描述符的另一种方法是使用 fcntl 函数，下一节将对该函数进行说明。实际上：
调用：

```c
dup ( filedes ) ; 
```

等效于：

```c
fcntl (filedes, F_DUPFD, 0);
```

而调用：

```c
dup2(filedes, filedes2) ；
```

等效于：
```c
close ( filedes2 ) ;
fcntl(filedes, F_DUPFD, filedes2);
```

在最后一种情况下，dup2并不完全等同于close加上fcntl。它们之间的区别是：
(1)dup2 是一个原子操作，而 close及fcntl则包括两个函数调用。有可能在 close和fcntl之间插入执行信号捕获函数，它可能修改文件描述符。 
(2)在dup2和fcntl之间有某些不同的errno。

9、fcntl 函数

fcntl函数可以改变已经打开文件的性质

```c
#include <sys/types.h>
#include <unistd.h>
#include <fcntl.h>
int fcntl(int  filedes, int cmd,.../* int  arg * / ) ;
```

返回：若成功则依赖于 cmd(见下)，若出错为- 1
(1)第三个参数总是一个整数，与上面所示函数原型中的注释部分相对应。但记录锁时，第三个参数则是指向一个结构的指针。
(2)fcnt l函数有五种功能:
- 复制一个现存的描述符（cmd＝F_DUPFD）。
- 获得/设置文件描述符标记（cmd=F_GETFD或F_SETFD）。
- 获得/设置文件状态标志（cmd = F_GETFL或F_SETFL）。
- 获得/设置异步I/O有权（cmd = F_GETOWN或F_SETOWN）。
- 获得/设置记录锁（cmd = F_GETLK , F_ SETLK或F_ SETLKW）。

(3)前七种我们将涉及与进程表项中各文件描述符相关联的文件描述符标志，以及每个文件表项中文件状态标志。
-  F_DUPFD  复制文件描述符filedes，新文件描述符作为函数值返回。它是尚未打开的各描述符中大于或等于第三个参数值（取为整型值）中各值的最小值。新描述符与 filedes共享同一文件表项 。但是，新描述符有它自己的一套文件描述符标志，其 FD_CLOEXEC文件描述符标志则被清除
-  F_GETFD  对应于filedes的文件描述符标志作为函数值返回。当前只定义了一个文件描述符标志FD_CLOEXEC。
- F_SETFD  对于filedes设置文件描述符标志。新标志值按第三个参数 (取为整型值)设置。应当了解很多现存的涉及文件描述符标志的程序并不使用常数FD_ CLOEXEC，而是将此标志设置为0 (系统默认，在exec时不关闭)或1 (在exec时关闭)。
- F_GETFL  对应于filedes的文件状态标志作为函数值返回。在说明 open函数时，已说明了文件状态标志。
- F_SETFL  将文件状态标志设置为第三个参数的值 (取为整型值)。 可以更改的几个标志是：O_APPEND，O_NONBLOCK，O_SYNC和O_ ASYNC。
- F_GETOWN  取当前接收SIGIO和SIGURG信号的进程ID或进程组ID。
- F_SETOWN  设置接收SIGIO和SIGURG信号的进程ID或进程组ID。正的arg指定一个进程D，负的arg表示等于arg绝对值的一个进程组ID。

```c
void set_fl(int fd, int flags) /*flags are file status flag */
{
    int val;
    if ((val = fcnt(fd, F_GETFL, 0)) < 0)
    {
        printf ("fctl F_GETFL error");
    }
    val |= flags; /*turn on fags */
    if (fcntl(fd, F_SETFL, val))< 0)
    {
        printf ("fcntl F_SETFL error");
    }
}
```

10、/dev/fd

(1)比较新的系统都提供名为/dev/fd 的目录，其目录项是名为 0 、1、2 等的文件。打开文件/dev/fd/n等效于复制描述符 n (假定描述符n是打开的)在函数中调用：

```c
fd = open("/dev/fd/0", mode);
```

大多数系统忽略所指定的mode，而另外一些则要求mode是所涉及的文件 ( 在这里则是标准输入)原先打开时所使用的mode的子集。因为上面的打开等效于：

```c
fd = dup(0);
```

描述符0和fd共享同一文件表项 。

(2)某些系统提供路径名/dev /stdin , /dev/stdout 和/dev/stderr 。这些等效于/dev/fd/0 , /dev/fd/1和/dev/fd/2。


