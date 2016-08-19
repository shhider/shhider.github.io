---
layout: post
category: code
title: 从蜂巢开发中想到的几点前端开发建议
---

> 持续更新中
> - 20160704 添加``inline-block``

## 流程化

1. 提交代码前自主进行code review，避免手误造成的错误提交到仓库。建议安装git的；
2.

## NEJ的几种循环方法

### _$loop

### _$reverseEach

### _$forEach

### _$forIn

## NEJ各生命周期各司其职

模块的模版DOM事件，在``__init``方法中使用``_$addEvent``方法进行绑定；全局DOM事件(window/document/location等)应在``__onShow``或``__onRefresh``方法中进行绑定。

## 事件绑定

## JS中取DOM节点

```javascript
// Bad
this.__tipTag = _e._$getByClassName(this.__body, 'u-invalid')[0];           // 1
this.__base_desc = _e._$getByClassName(this.__body, 'u-invalid')[1];        // 2
this.__detail_desc = _e._$getByClassName(this.__body, 'u-invalid')[2];      // 3
this.__userName = _e._$getByClassName(this.__body, 'j-userNameText')[0];    // 4
this.__openLevel = _e._$getByClassName(this.__body, 'openLevel');           // 5

// Good
this.__tips = _e._$getByClassName(this.__body, 'j-tip');
this.__tipTag = this.__tips[0];
this.__tipBaseDesc = this.__tips[1];
this.__tipDetailDesc = this.__tips[2];
```

### 在class中添加js标识

看到1、2、3行，很明显css与js发生了耦合。若下次调整样式，class名称需要改动，岂不是还要修改js？因此使用``j-xxx``的形式，用于js中取得该节点。

### ``_$getByClassName``获取节点

``_$getByClassName``方法怎么说也要消耗一些性能，即使IE9+的浏览器有原生支持。像1、2、3行中，“大费周折”取得了一串节点、取得其中一个，然后后面扔掉了已经取得的节点，又重新去取这些节点。


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
