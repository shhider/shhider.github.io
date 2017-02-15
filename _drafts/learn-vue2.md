---
layout: post
category: code
description: 熟悉熟悉vue
title: 学习Vue的基本概念
---

## 模版语法


### 插值

- 文本，双大括号，同样支持js表达式；只绑定一次时，添加v-once指令属性；
- 插入html，和regular类似，v-html指令属性；
- 绑定属性，v-bind:id="";
- 模版中的表达式处在一个定制的沙盒中，根据白名单访问全局变量，如``Math``和``Date``；

### 指令

- 标签的指令属性，都是直接 **双引号** ，这点与regular细微差别；
- 指令属性``v-if``其实相当于``r-show``，但不满足时是移除元素；
- 指令属性冒号后是参数，如``<a v-bind:href="url"></a>``就是将属性href与url表达式的值绑定；
- 这还有个修饰符的概念，暂时还不清楚其具体意义；
- ``v-bind:href``可以缩写为``:href``，``v-on:click``可以缩写为``@click``；

### 过滤器

这点的概念和regular的filter一样的。使用上不同之处，首先是限制了使用的地方：

> Vue 2.x 中，过滤器只能在 mustache 绑定和 v-bind 表达式（从 2.1.0 开始支持）中使用，因为过滤器设计 **目的就是用于文本转换**。为了在其他指令中实现更复杂的数据变换，你应该使用计算属性。

然后传参是直接加在括号内：

```html
{{ message | filterA('arg1', arg2) }}
```

## 事件：

绑定事件：

```html
<component v-on:click="handle"></component>
```
