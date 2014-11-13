---
layout: post
title: 使用源码包安装MySQL
category: programming
tags: [linux, mysql]
---
在 Windows 和 Linux 平台下多次使用过 MySQL 来做事情。不过一般用的都是集成好了 AMP (Apache+MySQL+PHP) 环境的工具，或是直接用 apt-get 、yum 之类的包管理工具来安装的，因此没有深入研究过 MySQL 这个软件本身，例如它的软件结构、组成部分、在 Linux 下所用动态库的路径、数据库文件的路径、各种管理工具是如何实现的，等等。这次的一个项目需要我把一个 C 代码连接使用的 MySQL 数据库移植到一个 X86 的嵌入式设备上，设备系统是一个经过剪裁的 2.6 内核的 Linux 。因此想以我用源码包编译、安装 MySQL 的过程为线索，试着梳理一下我自己对 MySQL 的认识，顺带弄清楚上述疑问。

<!-- excerpt -->

