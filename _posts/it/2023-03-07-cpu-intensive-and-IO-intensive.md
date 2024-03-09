---
layout: post
title: CPU 密集型 和I/O 密集型
date: 2023-03-06 10:12:15
categories: Linux
tags: 工作经验 性能分析
excerpt: CPU 密集型和 I/O 密集型是两种不同类型的系统负载
---

CPU 密集型和 I/O 密集型是两种不同类型的系统负载。

CPU 密集型的系统负载主要是指系统需要大量的 CPU 处理能力，例如复杂的计算、加密、解压等操作，此时系统的CPU资源是系统性能的瓶颈。

而 I/O 密集型的系统负载则主要是指系统需要大量的I/O（输入输出）操作，例如大量的文件读写、网络传输、数据库查询等操作，此时系统的瓶颈在于I/O资源的限制。

要判断一个系统是 CPU 密集型还是IO密集型，可以考虑以下几点：

- CPU 密集型系统负载会占用大量的 CPU 资源，导致 CPU 利用率接近或达到 100%。而 I/O 密集型系统负载则不一定会占用大量的 CPU 资源，CPU 利用率可能不会很高。

- CPU 密集型系统负载的主要瓶颈在于 CPU 资源，因此增加 CPU 的数量或提高 CPU 的性能可以提升系统性能。而 I/O 密集型系统负载的主要瓶颈在于 I/O 资源，因此增加磁盘、网络等 I/O 设备的数量或提高 I/O 设备的性能可以提升系统性能。

- CPU 密集型系统负载通常可以通过并行化处理来提高系统性能。例如将一个任务拆分成多个子任务，分别由多个 CPU 核心同时处理。而 I/O 密集型系统负载则通常无法通过并行化处理来提高性能，因为 I/O 操作通常是有序的，无法并行处理。

在实际应用中，可以通过监控系统资源的使用情况来判断系统是 CPU 密集型还是 I/O 密集型。例如可以通过查看 CPU 利用率、磁盘 I/O 速率、网络吞吐量等指标来判断系统负载类型。同时也可以通过分析系统日志、调查应用程序的工作方式来确定系统负载类型。

在 Linux 系统中，通过什么指标来判断程序是 CPU 密集型和 I/O 密集型？

- CPU 使用率：如果一个程序在运行过程中占用了大量的 CPU 时间，那么它很可能是 CPU 密集型的。可以使用 top、htop 或 pidstat 等命令来查看进程的 CPU 使用率。

- I/O 操作次数和速率：如果一个程序在运行过程中需要频繁进行 I/O 操作，那么它很可能是 I/O 密集型的。可以使用 `iostat` 或 `dstat` 命令来查看进程的 I/O 操作次数和速率。

- 上下文切换次数：上下文切换是 CPU 在不同进程或线程之间切换的过程，如果一个程序需要频繁地进行上下文切换，那么它很可能是 CPU 密集型的。可以使用 `vmstat` 或 `pidstat` 命令来查看进程的上下文切换次数。

内存使用情况：如果一个程序需要大量的内存来存储数据，那么它很可能是 CPU 密集型的。可以使用 top 或 ps 命令来查看进程的内存使用情况。

用 iostat 查看系统信息，性能分析

```sh
[root@VM-0-15-centos ~]# iostat
Linux 3.10.0-1062.18.1.el7.x86_64 (VM-0-15-centos) 	2023年03月23日 	_x86_64_	(1 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.73    0.00    0.60    0.16    0.00   98.51

Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
vda               5.07        49.60        88.86 3946267318 7070489480
scd0              0.00         0.00         0.00        314          0
```
 **I/O瓶颈**

如果 `%iowait` 的值过高，表示硬盘存在 I/O 瓶颈

**内存不足**

`%idle` 值高，表示 CPU 较空闲，如果`%idle` 值高但系统响应慢时，有可能是 CPU 在等待分配内存，此时应加大内存容量

**CPU资源不足**

如果 %idle 值持续低于 10，那么系统的 PU 处理能力相对较低，表明系统中最需要解决的资源是 CPU.
