---
layout: post
title: FrontEnd Tips
category: code
---

getElement(s)ByXXX方法返回的都是HTMLCollection，有length等属性，但不是Array；

Console对象有Time、TimeEnd等方法，用来测试运行时间蛮方便；

需要在监听Radio的选中状态时，一直没有理想的方法，后来发现RadioStateChange这么一个事件，经过查询，这并不是标准的W3C事件，而是Mozilla的XUL的事件。

在使用表单控件（input/select/textarea）的时候，常常会使用label来指定说明文字和控件之间的关联，以方便用户点击、输入、选择，并提升页面的语义化。今天为了监听radio控件的选中状态，考虑到点击的是label，因此把点击事件监听到了label上，结果发现在点击label时，事件会触发两次。经过在IE1(9-11)、Firefox、Chrome中试验发现，在label上点击后，浏览器会“帮忙”在对应的控件上执行一次点击。

为什么进行Web页面设计时分辨率往往设置为72？因为1px = 1pt * 图像分辨率/72，方便计算。

避免过深的层级的同时，不过分苛求浅层级，按照页面模块划分、何时即可。

clip: rect(0px,60px,200px,0px); 可以对元素进行裁剪，浏览器支持良好除了IE不支持“inherit”值。该属性只对absolute、fixed绝对定位元素生效。四个值分别是top/right/bottom/left，要注意的是值是相对左上角(0, 0)来取的。
