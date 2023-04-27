---
layout: post
title: docker data volume 
date: 2023-04-19 10:12:15
categories: 分布式
tags: docker 
excerpt: docker date volume 
---

容器用的分层存储技术，运行时，以镜像为基础层，在其上创建一个当前容器的存储层，叫“容器存储层”。容器消亡时，这个容器存储层也同样会消亡。

所以运行的数据如果保存在这一层，则会丢失。

docker 数据卷，就是绕过 union fs 的分层，而是直接读写在宿主的目录上。 有点类似于，把一个硬盘挂载到系统上一样。

它的特点：

- 可以在多个容器上共享
- 即时生效
- 不会影响镜像
- 不会随着容器消亡而消失 

# 创建数据卷

```sh 
# 创建一个 nginx-log 的卷
 docker  volume create inspect nginx-log 
 nginx-log

 # 查看卷信息
 docker  volume  inspect nginx-log
[
    {
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/nginx-log/_data",
        "Name": "nginx-log",
        "Options": {},
        "Scope": "local"
    }
]
```

# 启动容器时带上数据卷

```sh 
 docker run -d -it --name=edc-nginx -p 8800:80 -v nginx-log:/usr/share/nginx/html nginx

Unable to find image 'nginx:latest' locally
Trying to pull repository docker.io/library/nginx ...
latest: Pulling from docker.io/library/nginx
26c5c85e47da: Pull complete
4f3256bdf66b: Pull complete
2019c71d5655: Pull complete
8c767bdbc9ae: Pull complete
78e14bb05fd3: Pull complete
75576236abf5: Pull complete
Digest: sha256:63b44e8ddb83d5dd8020327c1f40436e37a6fffd3ef2498a6204df23be6e7e94
Status: Downloaded newer image for docker.io/nginx:latest
a90f37d097e5e35b645b530a4dd20726a52819c688373c0d683a9244f411b570
```
-v 代表挂载数据卷，并且将数据卷挂载到 宿主的目录 /usr/share/nginx/html 下

# 查看数据 

```sh
docker inspect nginx-log
[
    {
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/nginx-log/_data",
        "Name": "nginx-log",
        "Options": {},
        "Scope": "local"
    }
]
```
# 删除数据卷

```sh 
docker stop edc-nginx
docker rm edc-nginx
docker inspect nginx-log
[
    {
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/nginx-log/_data",
        "Name": "nginx-log",
        "Options": {},
        "Scope": "local"
    }
]
```

停止并删除容器之后， 数据卷是依旧存在的。要自己手动清除

```sh
docker volume rm nginx-log
nginx-log

docker inspect nginx-log
[]
Error: No such object: nginx-log
```

