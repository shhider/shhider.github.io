---
layout: post
title: 【CSS标准系列】Visual Formatting Model —— 文档元素在页面中如何布局
category: code
---

标准文档地址：[Visual formatting model](https://www.w3.org/TR/CSS22/visuren.html#floats)

首先我们再回顾下Float元素的[定位机制](https://www.w3.org/TR/CSS22/visuren.html#positioning-scheme)。

> Floats. 
> In the float model, a box is first laid out according to the normal flow, then taken out of the flow and shifted to the left or right as far as possible. Content may flow along the side of a float.

根据W3C标准文档所述，Float元素首先按照普通流进行定位，然后将其脱离出普通流，并向左或向右移动尽可能远。

这里的「尽可能远」是指，移动Float元素，直到该元素的外边缘碰到父元素的边缘，或者另一个Float元素的边缘。

> A floated box is shifted to the left or right until its outer edge touches the containing block edge or the outer edge of another float.


## 9.4 普通流

处在普通流内的框属于一个格式化上下文。在CSS2.2中的格式化上下文，可能指表格、块或者行内（？）。在未来的CSS版本中，将介绍其他类型的格式化上下文。

块级框包含在


Block Formatting Context(翻译这个名次灰常蛋疼，后文直接称BFC吧)


## Reference

- [CSS深入理解流体特性和BFC特性下多栏自适应布局 « 张鑫旭-鑫空间-鑫生活](http://www.zhangxinxu.com/wordpress/2015/02/css-deep-understand-flow-bfc-column-two-auto-layout/)
