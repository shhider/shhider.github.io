---
layout: post
title: 给夫人看的Ubuntu系统安装方法
category: code
---

## 前言

## Wubi安装

Wubi（基于Windows的Ubuntu安装程序，Windows-based Ubuntu Installer）

### 功能和目标：

- 帮助不熟悉Linux的Windows用户在试用Ubuntu时，无需对硬盘进行格式化或重新分区；
- 也可以通过Wubi快速地卸载Ubuntu。

### 关键词：

- 非虚拟机。Wubi是在一个虚拟设备中创建一个独立的安装（我的理解就是与虚拟机的原理类似）；
- 从8.04起，Wubi的代码开始合并到Ubuntu中；
- Ubuntu 13.04停止支持Wubi（然后我在目录里还能看到Wubi？）；
- 另外还有Lubi、Mubi，分别基于Linux、Mac；

### 局限性：

- 不支持休眠[8]。
- 强制重启（关闭电源）时，Wubi的文件系统比普通的文件系统更脆弱[9]；
- 

### 安装方法

1. 首先，下载Ubuntu系统的ISO镜像文件。你可以从[Ubuntu官方](http://www.ubuntu.org.cn/desktop)找到各个版本的下载地址，或者可以从国内的镜像站找到，下载会快不少~

    - [网易的镜像站](http://mirrors.163.com/ubuntu-releases/)
    - [搜狐的镜像站](http://mirrors.sohu.com/ubuntu-releases/)
    - [浙江大学的镜像站](http://mirrors.zju.edu.cn/ubuntu-releases/)
    - 你还可以找到很多公司和高校提供的镜像站

2. 在下载Ubuntu的ISO文件时，你也可以看到`wubi.exe`文件，一起下载吧~建议下载到同个文件夹里；
3. Ubuntu的镜像和`Wubi.exe`下载完成后，双击`wubi.exe`，出现如图的窗口；
4. 就像安装一个Windows的应用程序一样，选择你喜欢的配置，点击安装。然后就等待它安装完成吧~

### Reference

- [Wubi - 维基百科](https://zh.wikipedia.org/wiki/Wubi)
- [五种ubuntu安装方法简述](http://forum.ubuntu.org.cn/viewtopic.php?t=268355)
