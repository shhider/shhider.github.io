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

HtmlFormElement对象中的表单项是只读的！[http://www.w3.org/TR/DOM-Level-2-HTML/html.html#ID-40002357](http://www.w3.org/TR/DOM-Level-2-HTML/html.html#ID-40002357)

键盘事件，按下Enter键有以下几种情况：单独按下 | Ctrl+Enter | Shif+Enter。除IE8及以下未测试外，Firefox是：13 | 13 |13，其他（Chrome、IE9+）均为：13 | 10 | 13。

在项目中，有大量的Ajax操作，后来发现一些页面请求接口后，数据并不是最新的。原因是浏览器的缓存机制，特别是IE中，以及Firefox。解决方案是，在需要获取最新数据的URL中加上一个随机字符串，我使用的是时间戳+随机数。

ES5中String对象已经支持trim方法，即去除字符串首尾的空白字符。IE8及以下不支持，实现的代码可以看以下。NEJ中已经对String添加该方法。

```javascript
function trim(str){ //删除左右两端的空格
    return str.replace(/(^\s*)|(\s*$)/g, "");
}
function ltrim(str){ //删除左边的空格
    return str.replace(/(^\s*)/g,"");
}
function rtrim(str){ //删除右边的空格
    return str.replace(/(\s*$)/g,"");
}
```

在很多情况下，我们经常通过多个网址提供相同的内容。通过多个网址访问的内容（或等同内容）定义一个规范网址，我们可以避免一些seo上的问题。

```html
<link rel="canonical" href="https://example.com/more/better/url" />
```

- Google文档：[https://support.google.com/webmasters/answer/139066?hl=zh-Hans](https://support.google.com/webmasters/answer/139066?hl=zh-Hans)；
- 也可以参考我网易考拉的做法：[www.kaola.com](www.kaola.com)，整体页面超级规范的。

