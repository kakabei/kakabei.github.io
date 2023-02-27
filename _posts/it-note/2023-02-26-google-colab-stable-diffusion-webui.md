---
layout: post
title: google colab stable diffusion webui
date: 2023-02-26 10:12:15
categories: 学习备忘  
tags: stable-diffusion 人工智能
excerpt: 在 Google colab 中使用 stable diffusion 的方式
---
在群里看到有人分享 AI 绘图。 问了一下，是 stable diffusion 。了解了一下，发现很牛。就学习了一下怎么画。 

stable diffusion 给图很吃GPU。 发现Google 有提供免费的 GUP 服务可以用。 当然是有使用限制的。 

# google colab 

网址： [https://colab.research.google.com/](https://colab.research.google.com/)

当然，你要有梯子可以科学上网，要有google 的帐号。 找开网址进行的界面如下：

![](/assets/stable-diffusion/colab-2023-02-27-22-27-47.png)

看到可以免费使用 GPU 就感觉很爽。 

这么有关于 Colab 的介绍。 可以在浏览器中写和执行 python 。可以被用于 AI 的相关开发。 

#  stable diffusion

如何在  colab 中使用 stable diffusion webui 呢？ 

在文件中打开一个笔记本。方式： 文件-> 打开笔记本 -> 选择GItHub。

在输入框中输入 ： [https://github.com/kakabei/stable_diffusion_chilloutmix_ipynb](https://github.com/kakabei/stable_diffusion_chilloutmix_ipynb)  点击搜索会展示出下面的三个项目，可以选择 `stable_diffusion_1_5_webui.ipynb`，  点击进去。

如下图：

![](/assets/stable-diffusion/colab-2023-02-27-22-36-26.png)

 加载后之后出现的主界面如下： 
![](/assets/stable-diffusion/colab-2023-02-27-22-42-56.png)

可以看左边的菜单，分为几个主要部分： 

- 选择模型
- 检查 GPU & 检查环境
- 第一次使用 - 安装依赖并启动
- 重启 - 重新启动 Stable Diffusion WebUI

可以在右边的主界面依次执行。

右上角的 **连接** 就是连接 google 的服务器的。 

我的 GPU 已经使用量已经被限制了。 

![](/assets/stable-diffusion/colab-2023-02-27-22-47-30.png)

google 分配的资源还可以：
![](/assets/stable-diffusion/colab-2023-02-27-22-49-36.png)

依次点击主界面的左边的执行按钮，可以执行 python 代码。 如图：
![](/assets/stable-diffusion/colab-2023-02-27-22-51-00.png)

执行到第三步“3. 第一次使用 - 安装依赖并启动” 完成后，会出现 URL 。点击进去就是 stable diffcusion webui  的界面了。 

![](/assets/stable-diffusion/colab-2023-02-27-22-55-41.png)

 stable diffcusion webui  的界面如下图： 
![](/assets/stable-diffusion/colab-2023-02-27-22-57-44.png)

脚本中安装了很多的插件，还是很不错的。 

# Colab Pro

google 会对免费的用户进行 GPU 的限制。花钱的话，它会更提供更多GPU 计算单位给你。 

![](/assets/stable-diffusion/colab-2023-02-27-23-01-46.png)

