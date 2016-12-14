---
layout: post
category: code
title: regularjs源码阅读系列——Event Loop
---

> 因为这两天要做一些regularjs相关的一些工作，所以终于开始有目标性地看一看regular的代码了。那从regular的代码里，能够发散出一些知识点，就打算记录在这个系列里，希望能够坚持记录下去。

```javascript
// src/util.js
_.nextTick = typeof setImmediate === 'function'?
  setImmediate.bind(win) :
  function(callback) {
    setTimeout(callback, 0)
  }
```

## setImmediate 与 setTimeout

setImmediate其实是一个新的定时器方法，（截至本文）只有在IE11/Edge与Nodejs中支持该方法。从regular的代码里可以得知，其实它的行为，和``setTimeout(0)``是一样的，都是将某任务立即添加到任务队列中——稍后执行。

两者之间实际上是有微小的差异的。HTML5规范中规定，setTimeout方法的timeout最小值是4ms，若小于4，则会自动增加为4ms；而setImmediate是没有类似的限制的。因此，setImmediate指定的回调通常比setTimeout早一些执行（具体在各环境的执行顺序未试验）。

> If the currently running task is a task that was created by the setTimeout() method, and timeout is less than 4, then increase timeout to 4.

## process.nextTick

在Nodejs中，nextTick其实是一个已有的方法，属于process的一个方法，同样是为了实现稍后执行的效果，但执行时机不一样，相对其他定时器方法要更早执行。

> On the next loop around the event loop call this callback. This is not a simple alias to setTimeout(fn, 0), it's much more efficient.

没有试验过，避免误人子弟，其他细节等我了解后再写。

## Event Loop

那前面的内容，都涉及到一个很重要的概念——事件循环(Event Loop)。先上图（图片来源：[Philip Roberts: Help, I’m stuck in an event-loop. on Vimeo](https://vimeo.com/96425312)）：

![Event Loop](/public/img/20161213-1-1.png)

（浏览器环境中）JavaScript的运行环境包含一个执行栈（Stack），当前任务中的函数将在栈内一步步地执行。执行过程中，除了ECMAScript各内置方法外，还会调用Web API，进行DOM、BOM等操作，这其中就会涉及到异步事件和定时任务。当异步事件或定时任务被触发，指定的回调函数执行任务就会被加入到任务队列（Callback Queue）中，等待执行。

当一个任务执行结束，任务栈为空时，JS引擎就会查询任务队列，若队列中有待执行的任务，就读取并放入执行栈开始执行；若为空，则继续查询，直到队列中出现新的任务。「事件循环」就是指JS引擎重复从任务队列中取任务、执行任务的过程。

基于这样的「事件循环」机制，JavaScript可以在单线程中实现「绝不阻塞」的并发执行。

（其实这里的「单线程」是指负责解释和执行JavaScript代码的主线程，其它像定时器、文件读写等也都会有各自单独的线程。）

了解了事件循环之后，setTimeout等定时任务的时间「误差」也就可以理解了，因为加入任务队列时，前面还有好多任务排着队呢。同时，``process.nextTick``的「强制性」，就是因为它相当于把任务「插队」到了任务队列的队首。

## Web Worker

！！TODO！！

## Reference

- [并发模型与Event Loop - JavaScript - MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/EventLoop)
- [6.3 Timers — HTML5](https://dev.w3.org/html5/spec-preview/timers.html)
- [JavaScript定时器与执行机制解析 - Web前端 腾讯AlloyTeam Blog - 愿景: 成为地球卓越的Web团队！](http://www.alloyteam.com/2016/05/javascript-timer/)
- [process Node.js v0.10.29 Manual & Documentation](https://nodejs.org/docs/v0.10.29/api/process.html#process_process_nexttick_callback)
- [Event Loop执行模拟](http://latentflip.com/loupe/?code=JC5vbignYnV0dG9uJywgJ2NsaWNrJywgZnVuY3Rpb24gb25DbGljaygpIHsKICAgIHNldFRpbWVvdXQoZnVuY3Rpb24gdGltZXIoKSB7CiAgICAgICAgY29uc29sZS5sb2coJ1lvdSBjbGlja2VkIHRoZSBidXR0b24hJyk7ICAgIAogICAgfSwgMjAwMCk7Cn0pOwoKY29uc29sZS5sb2coIkhpISIpOwoKc2V0VGltZW91dChmdW5jdGlvbiB0aW1lb3V0KCkgewogICAgY29uc29sZS5sb2coIkNsaWNrIHRoZSBidXR0b24hIik7Cn0sIDUwMDApOwoKY29uc29sZS5sb2coIldlbGNvbWUgdG8gbG91cGUuIik7!!!PGJ1dHRvbj5DbGljayBtZSE8L2J1dHRvbj4%3D)
- [JavaScript 运行机制详解：再谈Event Loop - 阮一峰的网络日志](http://www.ruanyifeng.com/blog/2014/10/event-loop.html)（有争议的一片文章，取其精华）
