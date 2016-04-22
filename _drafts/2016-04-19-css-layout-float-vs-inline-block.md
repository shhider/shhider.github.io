---
layout: post
title: CSS布局的与时俱进——从float到inline-block
category: code
---

> 「小孩子才分对错，成年人只看利弊」

## 引言

（我自己怎么从float到inline-block）

从目前来看，公司的首页（www.163.com）、易信官网（yixin.im）、考拉（www.kaola.com）满满的都是``float:left``

呐，再看一眼开头的话~这些产品的业务中，肯定有些因素使开发者们选择了``float:left``来进行横向布局。


## 使用float进行布局的典型场景

使用float布局最典型的场景就是，一个元素下需要包含一个居左、一个居右的元素，如常见的页面头部。

![网易蜂巢首页](/public/img/20160419-1-01.png)

（能不能不用这种float来实现这种局部呢？）

float布局最初是为了实现文字环绕图片的排版方式。

** 坏处 **

- 高度塌陷；
- 需要清除浮动；
- 双刃剑——环绕布局。当前流行的卡片式布局可能会出现问题；



## inline-block的出现

** 好处 **

- 语义上更符合；
- 可以应用``text-align``与``vertical-align``、``white-space``等属性；

** 副作用 **

- 最大的问题就是空白符

### inline-block的应用举例

- 横向导航栏；
- 自适应的卡片排布；
- text-align: justify；
- 内联块元素：文本与按钮并排



## 其他布局方案

其实以上的float和inline-block更偏向于“排版”，而不是“布局”。

### 绝对定位position: absolute

### Flex布局

### Grid布局

## 小结

> 「小孩子才分对错，成年人只看利弊」

本文主要比较了用``float``和``inline-block``来布局的优势和劣势，但并不是说哪种方式更好，我们在开发时，要根据实际需求和场景来选择实现方案。



## Reference

- [CSS 中，position：absolute、float、display：inline-block 都能实现相同效果，区别是什么？ - 知乎](https://www.zhihu.com/question/20821569)
- [inline-block 前世今生-层叠之美-inline-block 空隙, inline-block 间隙, inline-block 间距, inline-blcok 兼容, IE6 inline-blcok, 跨浏览器inline-block,inline-block bug 云路科技](http://www.iyunlu.com/view/css-xhtml/64.html)
- [Float vs. Inline-Block](http://www.ternstyle.us/blog/float-vs-inline-block)
- [拜拜了,浮动布局-基于display:inline-block的列表布局 « 张鑫旭-鑫空间-鑫生活](http://www.zhangxinxu.com/wordpress/2010/11/%E6%8B%9C%E6%8B%9C%E4%BA%86%E6%B5%AE%E5%8A%A8%E5%B8%83%E5%B1%80-%E5%9F%BA%E4%BA%8Edisplayinline-block%E7%9A%84%E5%88%97%E8%A1%A8%E5%B8%83%E5%B1%80/)
