---
layout: post
title: 优化动画性能——requestAnimationFrame方法
---

偷懒了几天，还是得继续写代码。今天开始着手学习HTML5中的动画。

首先看到的就是``requestAnimationFrame``方法，该方法与``setTimeout``、``setInterval``类似，不同之处是，浏览器能根据当前的页面状态、性能来调节循环间隔、暂停等，主要用于优化Canvas动画的性能。

它在1秒内执行的次数是不一定的，而是根据当前状态，**尽可能快**的执行传入的函数。

现在还存在的疑问是，如以下的代码：

```javascript
var globalID;
function repeatOften() {
    $("<div />").appendTo("body");
    globalID = requestAnimationFrame(repeatOften);
}
$("#start").on("click", function() {
    globalID = requestAnimationFrame(repeatOften);
});
$("#stop").on("click", function() {
    cancelAnimationFrame(globalID);
});
```

在repeatOften函数中，递归调用自身，会一直生成requestAnimationFrame的句柄。那么当一次调用过后，这次requestAnimationFrame产生的句柄是不是自动撤销了。而在点击stop时，cancel的是当前正在执行的这次动，即不接着往下执行了。

根据[司徒正美的博客](http://www.cnblogs.com/rubylouvre/archive/2011/08/22/2148797.html)中所说，requestAnimationFrame的优势还有：

- 对于一个侦中对DOM的所有操作，只进行一次Layout和Paint。
- 如果发生动画的元素被隐藏了，那么就不再去Paint。



### others

关于设置

	git config --global user.name 'your name'
	git config --global user.email 'your email'
	git config --global --list

关于提交

	git commig -m 'msg1' -m 'msg2'    // 可以提交多段信息

关于log

	git log -x    // 指定显示几条log信息

关于查看差异

	git diff    // 默认比较 工作目录-暂存区 之间的差异
	git diff --cache    // 比较 暂存区-版本库 之间的差异
	git diff HEAD    // 比较 所有改动与版本库 之间的差异

关于忽略设置

	可以把要忽略的文件添加到 .git/info/exclude 文件中，本地将忽略指定的文件，而不会传播出去

***

### 练手题

前端：

- 使用 Backbone、AngularJS 或者其他任意一款 MVC 框架实现 计时器 | 计算器 | todos
- 使用 HTML/CSS/CSS3 实现一个 水杯|手机|电脑|梅花，造型自选
- 使用 HTML5 实现一个 拍照|语音写作 功能
- IE的滚动条太丑，使用前端技术实现一个自定义的滚动条
- 在canvas上绘制一个可拖动矩形色块，不使用封装canvas的类库

Node：

- 实现一个可用的hook模块。
- 实现一个rss爬虫，任选三个站点抓取24小时内容并保存为文件，格式为 "标题+\n+内容+\n\n"。
- 使用基础 http/https 模块实现一个反向代理 server。
- 实现一套基础任务队列系统（协议与中间件任选）。需要具备分布式，可横向扩展，错误警告与重试，数据持久化特性，能通过 c10k 测试。