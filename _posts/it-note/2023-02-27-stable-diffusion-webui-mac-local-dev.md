---
layout: post
title: stable diffusion webui 的本地部署 mac 环境
date: 2023-02-27 10:12:15
categories: 学习备忘  
tags: stable-diffusion 人工智能
excerpt: Mac 本地环境部署 stable diffusion webui 使用体验并不是很好
---

`stable diffusion webui` 的 github 地址：[https://github.com/AUTOMATIC1111/stable-diffusion-webui.git](https://github.com/AUTOMATIC1111/stable-diffusion-webui.git)。我没有去深究这个地址是不是官方的。 

安装 `stable diffusion webui` 的 方法并不难，只是在 Mac 环境上可能有一些坑。

按上面的 `README.md` 的 **Installation and Running** 要求的环境有：

- [python 3.10]([https://www.python.org/ftp/python/3.10.10/python-3.10.10-macos11.pkg](https://www.python.org/ftp/python/3.10.10/python-3.10.10-macos11.pkg)) 最好就用这个版本，其他版本估计要报很多错误了。 
- git. 下载 github 用的。 `git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui.git`.
- [Anaconda3-5.3.1-MacOSX-x86_64.sh](https://repo.anaconda.com/archive/Anaconda3-5.3.1-MacOSX-x86_64.sh) 我 mac 本地有 python 2.7 的版本还有，所以安装了  Anaconda。如果不用就去把 py2.7 升级到上面的 py3.10

# Anaconda

进入Anaconda 环境的方法是： 

```sh 
. ~/anaconda3/bin/activate 
```

注意前面有一点 `.`。进入之后，前面有 `(base)`  如图：

![](/assets/stable-diffusion/stable-diffusion-webui-2023-02-27-23-48-14.png)

# stable diffusion webui 

进入 `stable-diffusion-webui` 目录之后，可以执行 `webui.sh`。

脚本会安装一些必要的依赖。 可能有一点慢。最好是有“梯子”，不然可能会有一些依赖包没有办法下载点。 

目录中有一个脚本 **webui-macos-env.sh**  运行的过程中会出现一些错误是由于 Mac 环境引起的。 要在这里配置参数。我添加后参数之后的样子：

```sh 
export COMMANDLINE_ARGS="--skip-torch-cuda-test --upcast-sampling --no-half-vae --use-cpu interrogate --precision full --no-half --disable-nan-check"
```

执行后 `webui.sh` 之后出现一个 URL: `http://127.0.0.1:7860`  如下图：

![](/assets/stable-diffusion/stable-diffusion-webui-2023-02-27-23-56-56.png)

在浏览器访问它即可。 

另： 上面出现的一些错误应该下载一些模型失败的。可以暂时忽略掉。 

![](/assets/stable-diffusion/stable-diffusion-webui-2023-02-28-00-00-17.png)

# 最后

Mac 环境是安装好，但是在这样的环境下进行 AI 绘图有一些难受。生成一张图大于1个小时左右，训练模型就久了吧。 

最好还是要在台式机上，要有好的 GPU 才行。 其实有一些遗忘的，以前遇到了再补充进来。



