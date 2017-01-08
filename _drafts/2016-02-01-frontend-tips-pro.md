---
layout: post
title: FrontEnd Tips Pro
category: code
---

### Mixin的概念

> (React中的概念)
> 虽然组件的原则就是模块化，彼此之间相互独立，但是有时候不同的组件之间可能会共用一些功能，共享一部分代码。所以 React 提供了 mixins 这种方式来处理这种问题。
> Mixin 就是用来定义一些方法，使用这个 mixin 的组件能够自由的使用这些方法（就像在组件中定义的一样），所以 mixin 相当于组件的一个扩展，在 mixin 中也能定义“生命周期”方法。

另外，从Vue对Mixin的实现，我的理解就是，Mixin是一个方法、属性集，对象在使用Mixin时，把这些方法和属性赋值到自身。

```javascript
/**
* Apply a global mixin by merging it into the default
* options.
*/

Vue.mixin = function (mixin) {
    Vue.options = mergeOptions(Vue.options, mixin)
}
```

#### Reference

- [Mixin | React 入门教程](https://hulufei.gitbooks.io/react-tutorial/content/mixin.html)


### JavaScript的编程范式

前端圈子里常见的编程范式主要有：

- 命令式（Imperative）
- 声明式（Declarative）
- 函数式（Functional）

#### Reference

- [What is functional, declarative and imperative programming?](http://stackoverflow.com/questions/602444/what-is-functional-declarative-and-imperative-programming)
- [编程范型详解| 四火的唠叨](http://www.raychase.net/2265)
