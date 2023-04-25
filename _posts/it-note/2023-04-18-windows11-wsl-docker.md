---
layout: post
title: window 11 wsl Ubuntu 22.04.1 LTS  docker  
date: 2023-04-19 10:12:15
categories: 分布式
tags: docker 
excerpt: window11 下的子系统是 ubuntu22，安装docker 无法启动
---

在 shell 执行 docker version 

```sh
Client: Docker Engine - Community
 Version:           23.0.4
 API version:       1.42
 Go version:        go1.19.8
 Git commit:        f480fb1
 Built:             Fri Apr 14 10:32:03 2023
 OS/Arch:           linux/amd64
 Context:           default
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
```

从错误上看是 docker daemon 没有启动。 

```sh 
 systemctl start docker
 System has not been booted with systemd as init system (PID 1). Can't operate.
```
window 11 下的 wsl 并没有 systemctl。 改用 service 启动

```sh 
service docker start
* Starting Docker: docker                                                                                                                  [ OK ]
```

但其实还是不行，看了日志， `cat /var/log/docker.log`

```sh 
failed to start daemon: Error initializing network controller: error obtaining controller instance: unable to add return rule in DOCKER-ISOLATION-STAGE-1 chain:  (iptables failed: iptables --wait -A DOCKER-ISOLATION-STAGE-1 -j RETURN: iptables v1.8.7 (nf_tables):  RULE_APPEND failed (No such file or directory): rule in chain DOCKER-ISOLATION-STAGE-1
 (exit status 4))
```

看了网上的别人的解决的方法：

```sh 
sudo update-alternatives --set iptables /usr/sbin/iptables-legacy
sudo update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy

sudo service docker start

docker version

Client: Docker Engine - Community
 Version:           23.0.4
 API version:       1.42
 Go version:        go1.19.8
 Git commit:        f480fb1
 Built:             Fri Apr 14 10:32:03 2023
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          23.0.4
  API version:      1.42 (minimum version 1.12)
  Go version:       go1.19.8
  Git commit:       cbce331
  Built:            Fri Apr 14 10:32:03 2023
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.6.20
  GitCommit:        2806fc1057397dbaeefbea0e4e17bddfbd388f38
 runc:
  Version:          1.1.5
  GitCommit:        v1.1.5-0-gf19387a
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0

```
可以了。

文章见： [https://github.com/microsoft/WSL/issues/6655](https://github.com/microsoft/WSL/issues/6655)

我的一些版本信息 ：

Windows 11 家庭中文版  Windows Feature Experience Pack 1000.22640.1000.0

```sh
uname -a
Linux xxxxx 5.10.102.1-microsoft-standard-WSL2 #1 SMP Wed Mar 2 00:30:59 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux

cat /etc/issue
Ubuntu 22.04.1 LTS \n \l
----

