---
layout: post
title: FrontEnd Tips
category: code
---

getElement(s)ByXXX方法返回的都是HTMLCollection，有length等属性，但不是Array；

Console对象有Time、TimeEnd等方法，用来测试运行时间蛮方便；

需要在监听Radio的选中状态时，一直没有理想的方法，后来发现RadioStateChange这么一个事件，经过查询，这并不是标准的W3C事件，而是Mozilla的XUL的事件。

在使用表单控件（input/select/textarea）的时候，常常会使用label来指定说明文字和控件之间的关联，以方便用户点击、输入、选择，并提升页面的语义化。今天为了监听radio控件的选中状态，考虑到点击的是label，因此把点击事件监听到了label上，结果发现在点击label时，事件会触发两次。经过在IE1(9-11)、Firefox、Chrome中试验发现，在label上点击后，浏览器会“帮忙”在对应的控件上执行一次点击。

text-align有一个属性——justify，表示两端对齐。在实现表单时，我希望label的文字能够两端对齐，但是添加text-align: justify属性却没有效果，表示很不解。查找资料后得知：text:align: justify属性是不会处理块内的最后一行文本，所以当块内只有一行文字时，不会有两端对齐的效果。在部分浏览器（(caniuse text-align-last)[http://caniuse.com/#search=text-align-last]）中，支持text-align-last属性，当指定text-align-last: justify时，块内最后一行文本也会进行两端对齐处理，不过这一属性支持度并不高。有一种hack方式就是：给单行文本后面再加一行（:after）。

```css
div{
    /* xxx */
    text-align: justify;
}
div:after{
    display:inline-block;
    content:'';
    overflow:hidden;
    width:100%;
    height:0;
}
```
