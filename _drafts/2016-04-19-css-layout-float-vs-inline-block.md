---
layout: post
title: inline-block元素在页面布局中的应用
category: code
---

元素横向排列，是前端页面中常见的布局方式。而目前实现横向排列，最普遍的方法就是使用CSS的浮动——Float属性。而本文要抛弃Float属性，介绍``inline-block``在页面布局中的应用，以及相比Float布局的优劣之处。

> ``inline-block``是一项被忽视了的「黑科技」。    ———— 尼古拉斯·浩宏

## inline-block

css属性为inline-block的元素，称为行内块级元素。表现方式是元素内部表现为块级框，自身表现为行内级框。在CSS的定位机制中，属于普通流（normal flow）。

相比之下，应用了Float属性的元素，即属于浮动（Float）定位，不仅本身元素布局更复杂之外，还会影响前后各级元素。你需要使用额外的CSS属性或节点来解决随之带来的问题，高度塌陷等。

inline-block无论从表现、层叠、影响等各方面，都要更「单纯」。

下面，将细数inline-block实现横向排列的优势以及相应的实例。

### 最基本的应用

下面以常见的导航栏为例：



### 通过vertical-align属性轻松控制垂直对齐方式

balabala

通过这一属性，还可以轻松实现元素垂直居中，如modal。

### 具备了inline元素的某些特性，使用高效

``white-space``

可以控制``inline-block``元素列表不换行，这一点在布局上比较可靠，不会出现意外情况而导致的布局混乱；而float属性，会在当前行空间不足时自动换行，容易在内容超长、页面缩放等情况下布局混乱。

``text-align``

通过left、right、center属性值，我们可以实现inline-block元素居左、居右以及居中显示；另外，我们还可以使用justify值实现横向布局下，元素自适应的两端对齐。

### 元素之间的空格问题

inline-block当然也不是完美的解决方案，其带给开发者们最头疼的问题就是inline-block元素之间的空隙。

默认情况下，inline-block元素之间，会出现一个4px或6px大小的空白。

造成inline-block元素空隙的本质是HTML中存在的空白符。（为啥这里的空白符就回表现出来呢？？）

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

- [Visual formatting model](https://www.w3.org/TR/CSS22/visuren.html#floats)

- [CSS 中，position：absolute、float、display：inline-block 都能实现相同效果，区别是什么？ - 知乎](https://www.zhihu.com/question/20821569)
- [inline-block 前世今生-层叠之美-inline-block 空隙, inline-block 间隙, inline-block 间距, inline-blcok 兼容, IE6 inline-blcok, 跨浏览器inline-block,inline-block bug 云路科技](http://www.iyunlu.com/view/css-xhtml/64.html)
- [Float vs. Inline-Block](http://www.ternstyle.us/blog/float-vs-inline-block)
- [拜拜了,浮动布局-基于display:inline-block的列表布局 « 张鑫旭-鑫空间-鑫生活](http://www.zhangxinxu.com/wordpress/2010/11/%E6%8B%9C%E6%8B%9C%E4%BA%86%E6%B5%AE%E5%8A%A8%E5%B8%83%E5%B1%80-%E5%9F%BA%E4%BA%8Edisplayinline-block%E7%9A%84%E5%88%97%E8%A1%A8%E5%B8%83%E5%B1%80/)
- [应不应该使用inline-block代替float_inline-block, float 教程_w3cplus](http://www.w3cplus.com/css/inline-blocks.html)
- [回归CSS标准之Float \| EFE Tech](http://efe.baidu.com/blog/float/)
- [Visual formatting model](https://www.w3.org/TR/CSS22/visuren.html)
- [那些年我们一起清除过的浮动-层叠之美-清除浮动,清除浮动方法,闭合浮动,hasLayout,Block formatting contexts,块级格式化上下文,伪元素清除浮动 云路科技](http://www.iyunlu.com/view/css-xhtml/55.html)
- [inline-block 前世今生 - 前端技术 \| TaoBaoUED](http://ued.taobao.org/blog/2012/08/inline-block/)
- [inline-block 未来-层叠之美- 云路科技](http://www.iyunlu.com/view/css-xhtml/79.html)
- [有哪些好方法能处理 display: inline-block 元素之间出现的空格？ - 一丝的回答 - 知乎](https://www.zhihu.com/question/21468450/answer/18342657)
