---
layout: post
title: 非阻塞加载JavaScript脚本文件以提升性能
category: code
---

> 这是2014年底在公司实习时在内部社区发的几篇文章之一。

今天在博客园一连看到几篇关于“通过非阻塞的方式加载JavaScript以提高性能”。

简单来说，阻塞是指浏览器遇到`<script>`标签使整个页面因脚本解析、运行而出现等待。不论实际的 JavaScript 代码是内联的还是包含在一个不相干的外部文件中,页面下载和解析过程必须停下,等待脚本 完成这些处理,然后才能继续。原因就是刚刚下载的脚本很有可能会修改页面内容，浏览器不得不让它先执行完后，接着渲染后面的内容。

所以，往往大家谈到“提升页面性能”的时候都会说“尽量把脚本放到页面底部”，就是为了在下载和执行脚本这段时间前，已经有内容显示，让页面显得更快。

既然知道JavaScript的阻塞会使页面加载变慢，那么就要想办法通过“非阻塞”的方式去加载JavaScript。目前有2种方法：

### 使用defer属性

HTML4.0.1为`<script>`标签添加了defer（延迟）属性，表明该标签引入的脚本在执行时不会影响页面内容。支持defer属性的浏览器遇到该属性后，会接着下载脚本文件，但是延迟执行，也就达到了非阻塞加载JavaScript的目的。（onload事件会在非阻塞脚本也执行完后触发）

但是在这个属性中还有几点需要注意：

首先，现代浏览器（HTML5）中明确规定，defer属性只适用于外部脚本文件。所以给嵌入脚本标签添加defer属性会被忽略（除了早期的 IE 4~7）；

在第二点之前，先来了解一下`document.write()`方法，在MDN中，对`document.write()`的概述是“向一个被 `document.open()` 打开的文档流中写入一串文本”，以及：

> Writing to a document that has already loaded without calling document.open() will automatically perform a document.open call.
> 在一个已经加载完成的页面，没有先调用document.call()方法就执行document.write()方法，会自动先执行document.open。

`document.open`方法将打开当前窗口的文档流并清空，就像一些语言中，使用'w'模式打开文件会同时清空该文件。（先给window对象设置一个属性，在执行`document.open`后该属性不变。）

再说回defer，浏览器以阻塞的方式加载脚本就是怕脚本会修改文档流，所以defer属性就要先告诉浏览器“别怕，我不会修改文档流的，你就把我放最后执行吧。”，浏览器才会放心的把这些脚本放到文档流完成之后再执行。但是，如果最后浏览器关闭文档流，开始执行非阻塞脚本的时候，又遇到了`document.write`方法，就想“丫的你不是说不会修改文档的嘛！？”，很生气，就直接忽略掉了`document.wirte()`方法……（chrome和FireFox Dev会报warning：来自一个外部异步载入的脚本的 `document.write()` 调用被忽略）

不过经过测试，IE9及以下版本居然不会忽略`document.write`……另外，对`document.open`方法，chrome还是会去执行它；IE会报致命错误（“没有权限”？？）；firefox等也会忽略。

如果你还是要修改文档内容，不是还有`appedChild`、`insertBefore`等DOM操作嘛。

另外，HTML5为`<script>`属性定义了async（异步）属性，大体上是和defer类似的，不同之处是，标记为async的脚本之间无法保证按照执行顺序。

### 动态脚本元素

还有实现非阻塞加载脚本的办法是动态脚本元素。引入脚本也是通过一个`<script>`标签实现的，所以我们也可以通过动态插入`<script>`标签的方式，根据页面完成后，再需要引入脚本，也就实现非阻塞地加载了脚本。

一般的套路如下：

```javascript
var script = document.createElement ("script");
script.type = "text/javascript";
script.src = "file1.js";
document.body.appendChild(script);
```

当然也可以写一个函数封装一下。优点是：跨浏览器、跨域，简单易用，也可以进行事件监听。目前很多模块化的JS框架都是以这种方式，按需载入模块文件。

另外，我们还可以直接通过XMLHttpRequest对象下载脚本代码后，直接注入到`<script>`标签中。这样的好处是下载和执行可以分离，但是会存在跨域的问题，脚本不得不与页面在同一个域中，无法使用CDN。

### Summary

以上，针对JavaScript脚本加载的优化，除了：

- 尽量把`<script>`放在页面底部，优先渲染页面；
- 根据实际情况，分组合并脚本文件。

之外，我们还可以使用上述的两类方法。
