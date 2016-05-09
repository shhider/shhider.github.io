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

可以说，``inline-block``是一项被忽视了的「黑科技」。


## 使用float进行布局的典型场景

使用float布局最典型的场景就是，一个元素下需要包含一个居左、一个居右的元素，如常见的页面头部。

![网易蜂巢首页](/public/img/20160419-1-01.png)

（能不能不用这种float来实现这种局部呢？）


### 实现文字环绕图片排版

### 在页面布局中的应用

#### 天然的顶部对齐


### 脱离了文档流（待考证），但依然占据位置


### 高度塌陷


### 需要清除浮动；


总结来说，浮动布局是一把双刃剑，在帮助我们快速实现布局的同时，还需要额外的节点和css属性来处理带来的问题。



## inline-block的出现

### 语义更准确、行为更简单

#### 未脱离文档流

#### 层叠关系清晰

![HTML元素层叠关系](/public/img/20160419-1-02.png)

### 通过vertical-align属性轻松控制垂直对齐方式

### 具备了inline元素的某些特性，使用高效

``white-space``

可以控制``inline-block``元素列表不换行，这一点在布局上比较可靠，不会出现意外情况而导致的布局混乱；

``text-align``

通过left、right、center属性值，我们可以实现inline-block元素居左、居右以及居中显示；另外，我们还可以使用justify值实现横向布局下，元素自适应的两端对齐。

### 元素之间的空格问题

默认情况下，inline-block元素之间，会出现一个4px或6px大小的空白。

#### 解决方法

1、``font-size: 0``；


### inline-block的应用举例

- 横向导航栏；
- 自适应的卡片排布；
- text-align: justify；
- 内联块元素：文本与按钮并排
- 栅格系统

可以说，除了「文字环绕」只能用float实现，其他场景下都可以用``inline-block``方式代替，各方面表现更加协调。

## 其他布局方案

其实以上的float和inline-block更偏向于“排版”，而不是“布局”。

### 绝对定位position: absolute

### Flex布局

### Grid布局

## 小结

> 「小孩子才分对错，成年人只看利弊」

本文主要比较了用``float``和``inline-block``来布局的优势和劣势，但并不是说哪种方式更好。还是那句话——具体情况具体分析，在进行页面开发时根据实际需求和场景来选择实现方案。



## Reference

- [CSS 中，position：absolute、float、display：inline-block 都能实现相同效果，区别是什么？ - 知乎](https://www.zhihu.com/question/20821569)
- [inline-block 前世今生-层叠之美-inline-block 空隙, inline-block 间隙, inline-block 间距, inline-blcok 兼容, IE6 inline-blcok, 跨浏览器inline-block,inline-block bug 云路科技](http://www.iyunlu.com/view/css-xhtml/64.html)
- [Float vs. Inline-Block](http://www.ternstyle.us/blog/float-vs-inline-block)
- [拜拜了,浮动布局-基于display:inline-block的列表布局 « 张鑫旭-鑫空间-鑫生活](http://www.zhangxinxu.com/wordpress/2010/11/%E6%8B%9C%E6%8B%9C%E4%BA%86%E6%B5%AE%E5%8A%A8%E5%B8%83%E5%B1%80-%E5%9F%BA%E4%BA%8Edisplayinline-block%E7%9A%84%E5%88%97%E8%A1%A8%E5%B8%83%E5%B1%80/)
- [应不应该使用inline-block代替float_inline-block, float 教程_w3cplus](http://www.w3cplus.com/css/inline-blocks.html)
- [回归CSS标准之Float | EFE Tech](http://efe.baidu.com/blog/float/)
