---
layout: post
category: code
title: 从蜂巢开发中想到的几个前端开发建议——样式篇
---

## float vs inline-block

在横向布局场景下，考虑下是不是用``inline-block``会更好？

### 控制换行

``float``进行横向布局，无法控制特殊情况下的换行。而使用``inline-block``，加上``white-space: nowrap``，就可以严格控制不换行，使布局更健壮。

典型场景：

![某浏览器宽度下的蜂巢首页](/public/img/20160704-1-01.png)
![某浏览器宽度下的表单](/public/img/20160704-1-02.png)

而且还不会有float这样那样的问题，不用清除浮动，没有冗余的节点，简直良心。

![float导致的错位](/public/img/20160704-1-03.png)

### 垂直对齐

``vertical-align: top | middle | bottom | ...``，各种对齐随心所欲。

### 水平对齐

``text-align: center``是最基本的用法了。通过``text-align: justify``，``inline-block``元素还可以实现两端对齐、均匀排列，还是自适应的哟~

两端对齐会碰到的问题是，如果只有一行、或者最后一行，无法两端对齐。这其实不能说是问题，因为它就是这样的特性，想一下文本的对齐情况就能理解了。解决方案是祭出``:after``大法，在父节点最后添加一个``width: 100%; height:0``的一个节点就OK。

### 问题

inline-block带来的最明显的问题是：元素之间的空白间隙。

网上有很多方法来解决这个问题，我一般使用「font-size: 0」，感觉最方便、最干净。虽然有点hack的味道、而且子元素还要再声明font-size，但我觉得总体是利大于弊的。

> ``inline-block``是一项被忽视了的「黑科技」。    ———— 尼古拉斯·浩宏
