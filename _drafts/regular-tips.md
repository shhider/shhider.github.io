---
layout: post
category: code
title: regularjs源码系列——regular开发Tips
---


## $watch

```javascript
$watch: function (expr, fn, options) {
    // ...
},
```

1、``$watch``方法接受3个参数。前面两个参数已经很熟悉了，分别是要监听的语句，和监听的语句的运算值发生变化时的回调。那第三个参数是一个配置对象，从代码看，可以接受以下参数：

```javascript
/**
 * Boolean      deep        是否进行深度检查
 * Boolean      force       //
 * Boolean      diffArray   若设置为true，当监听语句为数组时，会使用“莱文斯坦距离（Levenshtein distance）”算法计算数组差异。
 * Boolean      sync        //
 * Boolean      last        //
 * Boolean      init        若设置为true，则添加后立即检查一次
 */
```

2、当监听语句为数组时，回调方法被调用时，只传入一个参数。balabala

## $update
