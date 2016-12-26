---
layout: post
title: TensorFlow学习笔记(1):搭建环境
category: programming
tags: [tensorflow]
---

出于对新技术的好奇以及一部分的工作需要，最近决定踏足之前从未涉及过的机器学习领域。毕竟不是从头开始做理论研究，所以还是想先从实践中直观感受一下。做了简单地了解后，决定就先从谷人希的TensorFlow系统开始尝试吧~（机器学习、深度学习的大神们请轻拍）。

TensorFlow是Google在2015年11月开源的一个人工智能系统，是Google Brain在之前所开发的深度学习基础架构DistBelief的改进版本，该系统可以被用于语音识别、图片识别等多个领域。而[官网](https://www.tensorflow.org)上对其的定义是：

{:.quote}
> TensorFlow™ is an open source software library for numerical computation using data flow graphs.

数据流图中的节点，代表着某种数值运算；节点节点之间的边，代表多维数据(tensors)之间的某种联系。你可以在多种设备（含有CPU或GPU）上通过简单的API调用来使用该系统的功能。

我们先来搭建环境，我是在 Ubuntu16.04（64位 桌面版）的虚拟机上搭的。

安装Ubuntu系统的过程不在此赘述，装好后例行的换源并更新软件包。搜了一下发现阿里云的源似乎不错，将以下内容复制到 `/etc/apt/source.list` 文件中：

    deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted universe multiverse
    deb http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted universe multiverse
    deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted universe multiverse
    deb http://mirrors.aliyun.com/ubuntu/ xenial-proposed main restricted universe multiverse
    deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse
    deb-src http://mirrors.aliyun.com/ubuntu/ xenial main restricted universe multiverse
    deb-src http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted universe multiverse
    deb-src http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted universe multiverse
    deb-src http://mirrors.aliyun.com/ubuntu/ xenial-proposed main restricted universe multiverse
    deb-src http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse

保存，然后在终端里执行 `sudo apt-get update` 和 `sudo apt-get upgrade` 即可。

根据官网的说明，有五种安装tensorflow的方式（懒得翻译了。。）。

- **Pip install**: Install TensorFlow on your machine, possibly upgrading previously installed Python packages. May impact existing Python programs on your machine.
- **Virtualenv install**: Install TensorFlow in its own directory, not impacting any existing Python programs on your machine.
- **Anaconda install**: Install TensorFlow in its own environment for those running the Anaconda Python distribution. Does not impact existing Python programs on your machine.
- **Docker install**: Run TensorFlow in a Docker container isolated from all other programs on your machine.
- **Installing from sources**: Install TensorFlow by building a pip wheel that you then install using pip.

我选择了最简单直接的pip安装方式。首先是安装pip和python的开发包。

    $ sudo apt-get install python-pip python-dev

接下来在用pip安装tensorflow之前，可以先更换一下pip的源。和apt-get的源类似，原始的源网站

`pypi.python.org` 在某些国内网络环境下访问很不稳定，下载安装包时常断掉，速度也奇慢。所幸豆瓣提供了一个该站得镜像源 `pypi.douban.com` 软件更新很及时，速度也很快。换源的方式也很简单，在你的home目录下新建文件 `~/.pip/pip.conf` 并填入以下内容即可。

    [global]
    index-url = https://pypi.douban.com/simple

接下来再执行如下命令进行安装就可以了~pip会将所依赖的安装包全部自动装上，很方便。

    $ sudo pip install --upgrade https://storage.googleapis.com/tensorflow/linux/cpu/tensorflow-0.12.0-cp27-none-linux_x86_64.whl` 

全部安装好后用官网所给例子验证一下。

{:.post-image}
![image]({{BASE_PATH}}/assets/posts/images/2016-12-26-tensorflow-test.png)

至此，tensorflow的开发环境搭好了。

<p style="margin-top:60px;">参考资料：</p>

【1】 [TensorFlow: Download and Setup](https://www.tensorflow.org/get_started/os_setup)

【2】 [ubuntu 16.04阿里源](http://www.victorup.com/2016/05/ubuntu-16-04阿里源/)

【3】 [pip 更换软件镜像源](http://www.jianshu.com/p/785bb1f4700d)
