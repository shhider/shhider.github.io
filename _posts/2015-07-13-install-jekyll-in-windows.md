---
layout: post
title: 在Windows中安装、配置Jekyll
category: code
---

完整详细版请参考：[http://www.cnblogs.com/yevon/p/3308158.html](http://www.cnblogs.com/yevon/p/3308158.html)。

根据以上，笔者会列一下基本的步骤：

1. 下载rubyinstall（[官网](http://rubyinstaller.org/)）。这里要注意一点，Ruby2.x的Jekyll貌似存在很多莫名的Error，所以建议还是安装Ruby1.9.x；
2. 在上面这个页面往下翻，下载相应版本的DevKit，必需；
3. 安装Ruby，记得勾选添加到环境变量Path中。笔者在自己的机器上安装时很受伤，Ruby安装完后，命令窗口还是提示“没有可运行的程序”。重新安装好几次，结果发现系统要重启一下，如果大家也碰到这样的问题，可以试着重启，节省点时间……
4. 安装Devkit，其实就是解压。解压后，要在解压出的目录里运行

```
ruby dk.rb init
ruby dk.rb install
```

第一条命令会提示查看config.yml文件；如果第二条命令出现错误，记得查看一下这个文件，根据注释，添加Ruby的安装路径；

5. 这样，一般就可以安装Jekyll了。不过，国内的网络环境你懂的，可能安装会比较蛋疼。这里给出了淘宝提供的镜像：[http://ruby.taobao.org/](http://ruby.taobao.org/)，按照提示，更改一下源，安装会顺畅很多；
6. 运行``gem install jekyll``就开始安装了；
7. 最后，就可以在你的项目目录执行``jekyll server --watch``，看到你的站点了。

以上。

### 更新

现在在Ruby1.9.3版本下，第6步安装jekyll时，最后会提示：

```
Error installing jekyll:
jekyll requires Ruby version >= 2.0.0.
```

根据资料和亲测，以下资料有效：

> sudo gem install jekyll -v 2.5.3
> The default command apparently tries to install Jekyll 3, which was released on October 27th. The 2.5.3 version still works with Ruby 1.9.3, and its quite allright for me, probably for you too.

问题地址：[StackOverflow](http://stackoverflow.com/questions/33503796/error-installing-jekyll-requires-ruby-2-0-0)
