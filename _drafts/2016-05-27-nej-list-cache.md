---
layout: post
category: code
title: 「NEJ源码系列」列表缓存器详解
---

列表缓存器``_$$CacheList``的实现文件路径是``/src/util/cache/list.js``，继承的父类``_$$CacheAbstract``位于同目录的``cache.js``文件。

## 使用方法

### 参数说明

在实例化一个列表缓存器时，同样要传入参数对象，参数对象可传入以下字段：

- id, String: 用于后面的缓存标识；
- key, String: 数据项的主键，实例化时赋值给``this.__key``；
- autogc, Boolean: 该字段标识是否自动回收无用数据。当该字段为true时，``_$clearListInCache``方法执行后会接着执行垃圾回收。实例化时赋值给``this.__auto``；

## 关键属性说明

### this.__cache

### this.__lspl

这个参数在实例化时调用的``__doSwapCache``方法中初始化，根据传入的id，从``this.__cache``中取出，若为空，则初始化为对象，添加hash属性，以后会包含以下三 _类_ 字段：

```javascript
// hash    [Object] - item map by id
// list    [Array]  - default list
// x-list  [Array]  - list with key 'x'
```

其中的list只是示意，后面会用``_$getList``时传入的列表标识为属性名。

而hash属性保存数据项键值对，数据项的主键值，指向数据项；


## 生命周期各阶段做的事


## 常用方法讲解

### _$getList

请求参数列表。

```javascript
/**
 * @method   module:util/cache/list._$$CacheList#_$getList
 * @param    {Object} arg0   - 可选配置参数
 * @property {String} key    - 列表标识
 * @property {Number} data   - 其他数据信息
 * @property {Number} offset - 偏移量
 * @property {Number} limit  - 数量
 * @property {Object} ext    - 回传数据
 * @return   {Void}
*/
```

对数据进行一些预处理后，调用写好的``__doLoadList``方法。

### __getHash方法

从``this.__lspl``属性中取，如果没有这个字段，则置为一个空的对象。

### _$getListInCache

参数是``_$getList``使用的列表标识。方法中从``this.__lspl``中拿数据。

### __doSaveItemToCache

首先调用``__doFormatItem``方法格式化数据项，接着以「数据项主键值 => 数据项」的形式保存到``this.__lspl.hash``中，然后返回数据项的引用。**注意，如果hash中已经存在同主键的数据，则会用``_u._$merge(_olgItem, _newItem)``进行合并。**

### __doFormatItem

该方法用于格式化数据项，即把后端传的数据，转换到前端需要的格式。基础列表缓存器对象中是一个空方法，我们使用时覆盖即可。

### _$clearListInCache

本方法清除(splice)``this.__lspl``中列表的数据项（引用）。实际上数据还存在与``this.__lspl.hash``中。

如果``this.__auto``为true，则在之后调用``__doGCAction``方法。

### __doGCAction

本方法清除(delete)``this.__lspl.hash``中的数据项。注意，这里清除的数据项是之前依然存在于``this.__lspl``中列表的项。

所以搭配``_$clearListInCache``和``__doGCAction``，当前列表缓存器的所有数据项都会被去掉 _一次_ 引用。

但是但是但是！！！比较奇怪的是，根据``this.__lspl``的结构，每个数据项应该有两个引用；按照现在列表缓存器的回收方式，应该还不会被JS引擎回收掉啊。

## 一些使用方法

一个列表缓存器可以缓存多个列表，但是最好只用于一个。

## 其他记录

NEJ基础控件的生命周期中

实例化方法``_$allocate``会调用``__reset``方法；

``_$recycle``方法会调用``__destroy``方法；