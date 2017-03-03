---
layout: post
category: code
title: 怎么理解“表现与数据分离”？
---

今天看到一家公司的前端招聘要求里，有一项说“对表现与数据分离有深刻理解”。

我念叨了几遍，感觉把表现与数据分离好像不太对啊？我们现在用[Regular.js](http://regularjs.github.io/)、Vue.js这些框架，双向数据绑定、根据数据更新视图用起来无比欢乐啊，那都分离了怎么来更新数据呢？

所以是我理解得不对，还是这个概念就有问题？我找了些资料，最后结论是，我的理解不准确、同时这句话也略显过时了。

首先搜索结果中的各种文章，大多数是在13年、14年间的文章，被转载最多的是这篇：[http://www.cnblogs.com/yexiaochai/p/3167465.html](http://www.cnblogs.com/yexiaochai/p/3167465.html)

最后文章的重点是，表现与数据分离，就是通过``MVC``的方式进行开发。通过JS实现 Model、View、Controller 封装，只在 View 中进行 DOM 操作，Model 只进行数据读写，Controller 作为中间桥梁。当用户操作页面：

- View 响应事件、调用 Controller 方法：“某某数据要做修改啦”；
- Controller 得知后，调用 Model 方法：“你得把某某值设置为xxx”；
- 最后 Model 根据 Controller 要求修改相应的值。

反过来，当数据发生变化时，Model 可以直接通知 View 进行视图更新，也可以通知 Controller、由其通知 View 进行视图更新。

如此看来，处理表现的逻辑与处理数据的逻辑的确适合分开了，之后出现需求变动时，在 View 层只进行 DOM 操作，在 Model 层只进行数据操作，的确对可维护性是有好处的。

而现在更流行、更好用的 MVVM 类型的框架，其实就可以理解为 ViewModel 帮我们做了 Controller 的事情，只要我们通过模版来指定数据与视图的对应关系。

以上是“纯前端”的“表现与数据分离”的理解，另外，“前后端分离”也可以认为是此概念的一部分。

以上~
