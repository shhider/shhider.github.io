---
layout: post
title: 配置Redhat系统的yum源
---

首先一个字：坑。

之前一直在用Ubuntu，也没接触过其他发行版本的Linux系统。这次这项目的服务器结果给我的是redhat的系统，我连怎么安装软件都不知道……

## 配置yum

当然现在已经知道Ubuntu里的``apt-get``在centos里对应的是``yum``命令，但是一开始，完全用不了……

- yum update不是更新源，而是更新本机已安装的包；
- yum list all发现没有几个包可用……
- yum install httpd发现无法下载安装……

各种绕之后，我打算下载源码，自己编译安装。结果没有任何编译器，然后陷入“鸡生蛋，蛋生鸡”的纠结之中……

再各种绕之后，终于发现，原来centos在国内就是这逼样，最好的方法就是更换国内的yum源。

#### 删除原有的yum

```
rpm -aq|grep yum|xargs rpm -e --nodeps
```

#### 下载yum安装文件

根据你的系统版本，在镜像站中选择相应的安装包。我大网易的镜像站：[http://mirrors.163.com/centos/6/os/x86_64/](http://mirrors.163.com/centos/6/os/x86_64/)

```
wget http://mirrors.163.com/centos/6/os/x86_64/Packages/yum-3.2.27-14.el6.centos.noarch.rpm
wget http://mirrors.163.com/centos/6/os/x86_64/Packages/yum-metadata-parser-1.1.2-14.1.el6.x86_64.rpm
wget http://mirrors.163.com/centos/6/os/x86_64/Packages/yum-plugin-fastestmirror-1.1.26-11.el6.noarch.rpm
wget http://mirrors.163.com/centos/6/os/x86_64/Packages/python-iniparse-0.3.1-2.1.el6.noarch.rpm
```

(查看你的发行版本：``lsb_release -a``)

#### 安装yum

```
rpm -ivh python-iniparse-0.3.1-2.1.el6.noarch.rpm
rpm -ivh yum-metadata-parser-1.1.2-14.1.el6.x86_64.rpm
rpm -ivh yum-3.2.27-14.el6.centos.noarch.rpm um-plugin-fastestmirror-1.1.26-11.el6.noarch.rpm
```

说是最后两个包得同时安装，否则会相互依赖。

#### 更新repo文件

先备份原来的repo文件，防止出错。

```
mv /etc/yum.repos.d/rhel-debuginfo.repo /etc/yum.repos.d/rhel-debuginfo.repo.bak
```

在网易的[镜像使用帮助](http://mirrors.163.com/.help/centos.html)中下载一份repo文件，在相同的位置，文件名也修改为``rhel-debuginfo.repo``（根据你的情况定）。

这里的一个坑是：repo里会用到两个变量，$releasever和$basearch，而前者不知为何取不到值（后者根据系统，是i386或x86_64），导致URl错误，仍然无法使用yum。

因此还要编辑这个文件，把里面所有的``$releasever``修改成你的系统版本，我是全部改成``6``。

#### 更新源

```
yum clean all
yum makecache
```

Done!!
