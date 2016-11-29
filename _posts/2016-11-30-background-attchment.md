---
layout: post
category: code
title: CSS背景属性background-attchment的取值对比
---

当元素设置了``background-image``属性时，``background-attachment``属性则决定背景图片的滚动表现——跟随、或不跟随元素滚动而滚动。

``background-attachment``支持3种取值：``fixed``、``scroll``、``local``，以及继承``inherit``：

#### fixed

设置该值，表示背景图片相对于视口固定。直白来讲，就是背景图片的左上角始终与文档显示区域的左上角对齐，不管各种父元素、子元素如何滚动，背景图片位置都不变。

#### scroll

属性的默认值，表示背景图片相对于本元素固定，即背景图片的左上角与本元素的左上角对齐。

#### local

设置该值，表示背景图片根据本元素中的内容定位，即当元素中内容超过本元素高度、出现滚动条时，背景图片跟随内容的滚动而滚动。

该值是在CSS的“背景与边框（第三版）”中新添加的值，IE9及以上版本浏览器支持该属性值。

点击[查看demo](http://codepen.io/shhider/pen/gLGBWg) 帮助理解。


## Refrence

- [background-attachment - CSS | MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/background-attachment)
