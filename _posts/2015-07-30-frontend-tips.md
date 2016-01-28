---
layout: post
title: FrontEnd Tips
category: code
---

getElement(s)ByXXX方法返回的都是HTMLCollection，有length等属性，但不是Array；

---

Console对象有Time、TimeEnd等方法，用来测试运行时间蛮方便；

---

需要在监听Radio的选中状态时，一直没有理想的方法，后来发现RadioStateChange这么一个事件，经过查询，这并不是标准的W3C事件，而是Mozilla的XUL的事件。

---

在使用表单控件（input/select/textarea）的时候，常常会使用label来指定说明文字和控件之间的关联，以方便用户点击、输入、选择，并提升页面的语义化。今天为了监听radio控件的选中状态，考虑到点击的是label，因此把点击事件监听到了label上，结果发现在点击label时，事件会触发两次。经过在IE1(9-11)、Firefox、Chrome中试验发现，在label上点击后，浏览器会“帮忙”在对应的控件上执行一次点击。

---

为什么进行Web页面设计时分辨率往往设置为72？因为1px = 1pt * 图像分辨率/72，方便计算。

---

避免过深的层级的同时，不过分苛求浅层级，按照页面模块划分、何时即可。

---

clip: rect(0px,60px,200px,0px); 可以对元素进行裁剪，浏览器支持良好除了IE不支持“inherit”值。该属性只对absolute、fixed绝对定位元素生效。四个值分别是top/right/bottom/left，要注意的是值是相对左上角(0, 0)来取的。

---

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

---

HtmlFormElement对象中的表单项是只读的！[http://www.w3.org/TR/DOM-Level-2-HTML/html.html#ID-40002357](http://www.w3.org/TR/DOM-Level-2-HTML/html.html#ID-40002357)

---

键盘事件，按下Enter键有以下几种情况：单独按下 | Ctrl+Enter | Shif+Enter。除IE8及以下未测试外，Firefox是：13 | 13 |13，其他（Chrome、IE9+）均为：13 | 10 | 13。

---

在项目中，有大量的Ajax操作，后来发现一些页面请求接口后，数据并不是最新的。原因是浏览器的缓存机制，特别是IE中，以及Firefox。解决方案是，在需要获取最新数据的URL中加上一个随机字符串，我使用的是时间戳+随机数。

---

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

---

在很多情况下，我们经常通过多个网址提供相同的内容。通过多个网址访问的内容（或等同内容）定义一个规范网址，我们可以避免一些seo上的问题。

```html
<link rel="canonical" href="https://example.com/more/better/url" />
```

- Google文档：[https://support.google.com/webmasters/answer/139066?hl=zh-Hans](https://support.google.com/webmasters/answer/139066?hl=zh-Hans)；
- 也可以参考我网易考拉的做法：[www.kaola.com](www.kaola.com)，整体页面超级规范的。

---

Date对象可识别的ISO日期格式。ISO 8601是国际标准化组织制定的日期时间表示规范，全称是《数据存储和交换形式·信息交换·日期和时间的表示方法》。

起因是需要获得当天0点时的Unix时间戳，用 ``now - now%(24*60*60*1000)``取得的是UTC时区的0点——东八区的8点……所以最后使用`2015-11-02T01:20:00.456+0800`这样的格式来获得正确的0点时间戳。

补充：标准的格式应该是`2015-11-02T01:20:00.456+08:00`，IE对格式要求最严格，只有这样才可以识别。

Reference:

- [一起Polyfill系列：让Date识别ISO 8601日期时间格式](http://www.cnblogs.com/fsjohnhuang/p/3731251.html)
- [ISO 8601 - 维基百科，自由的百科全书](https://zh.wikipedia.org/zh-cn/ISO_8601)

---

**input事件**，当可输入控件的内容发生变化时触发，包括键盘事件、粘贴、undo等。

- 属于HTML5特性，IE9及以上和其他现代浏览器支持良好；
- select、checkbox、radio、file等控件慎用，各浏览器支持程度不一。

---

对勾√的HTML实体字符：&radic;

---

**innerHTML**

From [MSDN](https://msdn.microsoft.com/en-us/library/ms533897\(v=vs.85\).aspx)：

> The innerHTML property is read-only on the **col**, **colGroup**, **frameSet**, **html**, **head**, **style**, **table**, **tBody**, **tFoot**, **tHead**, **title**, and **tr** objects.
> To change the contents of the **table**, **tFoot**, **tHead**, and **tr** elements, use the table object model described in [Building Tables Dynamically](https://msdn.microsoft.com/en-us/library/ms532998\(v=vs.85\).aspx). However, to change the content of a particular cell, you can use innerHTML.

不过从测试（测试页面：[w3school.com.cn](http://www.w3school.com.cn/tiy/loadtext.asp?f=hdom_tablerow_innerhtml)）的情况来看，**tr**还是具有innerHTML属性的，至少目前可读。

[2015-11-27]经过一系列的测试（测试浏览器：Chrome46、Firefox42、Edge、IE8-11），table、thead、tbody、tr、td都存在innerHTML属性，在IE8-9中，前四者只读，其他情况都是可读写。

Reference:

- [BX9046: 各浏览器对 HTML 对象的 innerHTML 属性的读写支持存在差异](http://w3help.org/zh-cn/causes/BX9046)
- [HTML DOM innerHTML 属性](http://www.w3school.com.cn/jsref/prop_tablerow_innerhtml.asp)


---

HTML原生就带着一个`checkValidity`方法，以检验填写项是否正确，可用性待测试。

---

HTML的内容模型

在HTML5前，元素的类型分为：Block Level Elements和Text Level Elements，中文通常称为块级元素和行内元素。

> Most elements that can appear in the document body fall into one of two groups: block level elements which cause paragraph breaks, and text level elements which don't. Common block level elements include H1 to H6 (headers), P (paragraphs) LI (list items), and HR (horizontal rules). Common text level elements include EM, I, B and FONT (character emphasis), A (hypertext links), IMG and APPLET (embedded objects) and BR (line breaks). Note that **block elements generally act as containers for text level and other block level elements (excluding headings and address elements)**, while **text level elements can only contain other text level elements**. The exact model depends on the element.
> —— form [http://www.w3.org/TR/REC-html32#level](http://www.w3.org/TR/REC-html32#level)

而到了HTML5中，元素的内容模型得到了更细致的分类，共有以下7种：

- Metadata content
- Flow content
- Sectioning content
- Heading content
- Phrasing content
- Embedded content
- Interactive content

更详细的说明请前往：[W3C](http://www.w3.org/TR/2011/WD-html5-20110525/content-models.html#kinds-of-content)。同样，也可以阅读[WHATWG](https://developers.whatwg.org/content-models.html#content-models)

---

阅读文章：

- [http://www.cnblogs.com/damonlan/archive/2012/07/01/2553425.html](http://www.cnblogs.com/damonlan/archive/2012/07/01/2553425.html)
- [http://lixiaoshenxian.com/javascript/JavaScript%20变量声明.html](http://lixiaoshenxian.com/javascript/JavaScript%20%E5%A3%B0%E6%98%8E%E6%8F%90%E5%8D%87.html)

关键词：

- 变量提升（Hoisting）；
- 作用域（Scoping）：
    + 函数作用域；
    + 块级作用域；
- 函数提升：
    + 函数声明(Function declarations)。整个函数会放到最前面进行初始化；
    + 函数表达式(Function expressions)。仅提升变量；

关于作用域，对比下面两端代码，并查看打印的内容：

```javascript
var x = 1;
console.log(x);
if (true) {
  var x = 2;
  console.log(x);
}
console.log(x);
```

```javascript
'use strict';
var x = 1;
console.log(x);
if (true) {
  let x = 2;        // 注意let关键字
  console.log(x);
}
console.log(x);
```

---

call & apply

- 在 **ES5的严格模式** 中，两者的第一个参数都会变成this的值，哪怕传入的参数是原始值甚至是``null``或``undefined``；
- 在ES3和非严格模式中，传入的``null``和``undefined``会被全局对象代替；
- call方法接受多个参数；apply接受两个参数。牢记；

--- 

![20151223思维导图](/public/img/20150730-1-01.png)

---

URL Hash在3xx跳转时何去何从~

首先，**HTTP请求不会包含URL的Hash部分**。

当你请求``http://www.example.com/#/m/xxx/``时，HTTP的请求是这样的：

```
GET / HTTP/1.1
Host: www.example.com
...
```

Hash部分并不会包含在请求中。因此，后端程序也是无法取得设置的hash的。

现在，有这么一个情况：``http://www.example.com/detail/#id=123``需要登录后访问；未登录时，会302跳转到登录页``http://www.example.com/login/``。

那么，302跳转之后，hash部分还保留着吗？

根据资料(look at Reference)和自己本机的测试(Chrome/FireFox/Edge/IE8+)，hash部分是能够保留的。但是safari浏览器不能保留。

目前，也没有找到官方性质的文档，说明3xx跳转时是否应该保留hash。

### Reference

- [Hash URIs | W3C Blog](https://www.w3.org/blog/2011/05/hash-uris/)
- [URL Fragments and Redirects - IEInternals - Site Home - MSDN Blogs](http://blogs.msdn.com/b/ieinternals/archive/2011/05/17/url-fragments-and-redirects-anchor-hash-missing.aspx)
- [Bug 24175 – URL Redirect Loses Fragment](https://bugs.webkit.org/show_bug.cgi?id=24175)
