---
layout: post
category: code
title: 「NEJ源码系列」列表缓存器详解
---

## 引言

在单页应用中，几乎所有的业务数据都是通过异步请求接口获取的，随着项目的发展，页面中的请求会越来越多，如果不加以管理就会出现大量雷同的请求逻辑，遇到接口调整、需求变更时也会容易出错。

NEJ框架针对这样的情况，把异步请求中常见的「增删改查」操作进行抽象封装，并提供数据缓存特性，得到了「列表缓存器」控件。

NEJ列表缓存器对应用开发最显著的两点优势是：

- 集中管理应用中的异步请求，相同的请求逻辑只需写一次，具体页面中就只需要关心业务逻辑；
- 快速实现对数据的缓存，减少无谓的网络请求，提升页面性能和用户体验；

本文将从源码出发，结合我在实际应用中体会，介绍和分析NEJ列表缓存器的实现思路与使用方法。

## 使用方法

NEJ列表缓存器对象名为``_$$CacheList``，实现文件位于``/src/util/cache/list.js``。它继承的父类是``_$$CacheAbstract``，位于同目录的``cache.js``。基本的使用方法大致为以下几个步骤：

1. 针对业务需求添加一个列表缓存器子类，如``_$$CacheUser``，继承自``_$$CacheList``；
2. 在子类中实现``_$doLoadList``、``__doLoadItem``等接口方法，或添加自定义的业务方法；
2. 在相关业务脚本中引入子类文件，实例化列表缓存器，调用方法、处理事件回调。

### 实例化参数

在实例化一个列表缓存器时，同样要传入参数对象。除事件绑定外，其它参数字段如下：

```javascript
/*
 * @param    {Object}   options - 可选配置参数
 * @property {String}   id      - 缓存标识
 * @property {String}   key     - 指定数据项主键，默认为'id'
 * @property {Boolean}  autogc  - 标识是否自动进行垃圾回收
 * @property {Variable} data    - 供自定义的数据
 */
```

以上参数中：

- id是可选参数，不指定也不会有影响。后面所说的「缓存标识」就是指该参数，代码中用``cacheId``表示；
- 当autogc设置为true时，在``_$clearListInCache``方法执行后会接着自动执行垃圾回收方法``__doGCAction``；

### 获取数据列表

调用``_$getList``方法。参数传入列表标识。后面所说的「列表标识」就是指该参数，代码中用``listKey``表示

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

#### __doLoadList

```javascript
/**
 * 从服务器载入列表
 * 
 * @event    module:util/cache/list._$$CacheList#doloadlist
 * @param    {Object}   event  - 可选配置参数
 * @property {String}   key    - 列表标识
 * @property {Variable} ext    - 回调回传数据
 * @property {Number}   data   - 需要提交到服务器的其他信息
 * @property {Number}   offset - 偏移量
 * @property {Number}   limit  - 数量
 * @property {Function} onload - 请求回调
 */
```


#### __doFormatItem

该方法用于格式化数据项，即把后端传的数据，转换到前端需要的格式（完全另返回一个对象也是可以的）。基础列表缓存器对象中是一个空方法，我们使用时覆盖即可。

### 获取数据项

## 原理解析

接下来讲解列表缓存器以下四个方面的原理：

- 获取列表数据；
- 数据项的增删改查；
- 前向刷新列表；
- 清除缓存与垃圾回收。

在一一讲解之前，需要先介绍一下列表缓存器的缓存结构。

### 缓存结构

NEJ列表缓存器在其构造函数中声明了一个属性（不需要关心属性名），用于「存放」缓存结构，每个列表缓存器实例化时，``this.__cache``属性会指向该缓存结构。

```javascript
// _$$CacheAbstract#__init
this.__cache = this.constructor[_ckey];
```

通过这种形式，所以的列表缓存器实例会在同一个缓存结构中读写数据，即使某个列表缓存器实例被回收，（未显式clear的）数据依然存在，下次再需要相同标识的列表数据时，仍然可以直接从该缓存结构中获取。这也就达到了页面级的数据缓存。

缓存结构的构成大致如下，可以分为3部分讲解：

```javascript
// this.__cache
{
	// 1
	'cacheId_1': cacheUnit_1,
    'cacheId_2': cacheUnit_2,
    ...
    // 2
    'hash': {},
    'listKey_1': [],
    'listKey_2': [],
    ...
    // 3
    'ckey-l': requestList
}
```

第1部分就是「缓存标识 => 缓存单元(cacheUnit)」。每个列表缓存器实例化时，通过缓存标识取得对应的缓存单元（由``this.__lspl``保存引用），相同缓存标识的列表缓存器会把数据放在同一个缓存单元中。一个缓存单元的结构如下：

```javascript
{
	'hash': {
    	'itemId_1': item_1,
        'itemId_2': item_2,
        ...
    },
    'listKey_1': [item_1, item_2, ...],
    'listKey_2': [item_x, item_y, ...],
    ...
}
```

缓存的数据项（包括数据列表中的项）都会以「数据项ID => 数据项」保存在``hash``中，数据列表还会按列表标识再保存一份。数据列表和``hash``中相同的项，引用的是同一个数据，也就是：``_$getList``缓存的数据列表的项会有两个引用，``_getItem``缓存的数据项只有一个引用。

下面来看第2部分……等等，第2部分不是和缓存单元的结构一样么？没错，这就是一个缓存单元。当列表缓存器实例没有指定缓存标识时，就直接把缓存结构作为一个公共缓存单元。基于这一点，建议**实例化列表缓存器时都指定缓存标识**，使缓存结构更简单，避免潜在的数据混乱风险。当然，相同数据的缓存标识不能一次一个样，不然缓存相当于无效了。可以查看``_$$CacheList#__doSwapCache``源码帮助理解。

第3部分就是一个请求列表，保存当前等待响应中请求，无需关心属性名。当某个请求发起时，就把该请求保存到该列表中，之后相同请求不再重复发起。


### 获取列表

列表缓存器获取某列表



#### __doSaveItemToCache

列表缓存器在获取、更新数据列表或数据项时，都会通过该方法保存到``this.__lspl``缓存结构中。内部执行逻辑是：

1. 调用``__doFormatItem``方法格式化数据项；
2. 以「数据项主键值 => 数据项」的形式保存到``this.__lspl.hash``中；
3. 然后返回数据项的引用。

**注意**，第二步中，如果``this.__lspl.hash``中已经存在同主键的数据，则会用``_u._$merge(_oldItem, _newItem)``进行合并。用该方法合并会出现的情况是，oldItem中存在、而newItem中不存在的字段，合并结果中会被保留。

因此，建议使用列表缓存器时，子类中都要实现``__doFormatItem``方法来处理数据，保证同类型的数据项字段一致、简洁。

#### _$getListInCache

参数是``_$getList``使用的列表标识。方法中从``this.__lspl``中拿数据。

### 请求数据与垃圾回收

#### _$clearListInCache

本方法清除(splice)``this.__lspl``中列表的数据项（引用）。实际上数据还存在与``this.__lspl.hash``中。

如果``this.__auto``为true，则在之后调用``__doGCAction``方法。

#### __doGCAction

本方法用于从``this.__lspl.hash``结构中彻底删除数据（即去除数据的引用，真正的回收是由浏览器进行的）。``this.__lspl.list``中依然存在的数据将不会被删除。

该方法在列表缓存器实例被回收(``__destroy``)时调用；若实例化时指定``autogc: true``，则在每次``_$clearListInCache``之后也会调用。

为更清晰地理解，上源码：

```javascript
_pro.__doGCAction = function(){
    this.__timer = window.clearTimeout(this.__timer);
	// dump id map for used items
    var _map = {};
    _u._$loop(
        this.__lspl,function(_list,_key){
            if (_key=='hash') return;
            if (!_u._$isArray(_list)) return;
            // 把缓存结构的列表中，依然存在的数据项记录下来
            _u._$forEach(_list,function(_item){
                if (!_item) return;
                _map[_item[this.__key]] = !0;
            },this);
        },this
    );
    // check used in hash
    _u._$loop(
        this.__getHash(),
        function(_item,_id,_hash){
        	// 没有标识的数据项，delete删除
            if (!_map[_id]){
                delete _hash[_id];
            }
        }
    );
};
```

因此，对于不再需要了的数据列表，请在列表缓存器回收之前及时使用``_$clearListInCache``方法，或者在实例化时指定``autogc: true``。另外，数据项(item)不需要关心，本身就只在``this.__lspl.hash``里，回收时直接就删除了。

## 小结和建议

从源码层面介绍了NEJ列表缓存器后，最后小结几点NEJ列表缓存器的使用注意点和建议：

- 即使加载错误，也要调用onload等方法，数据传入null即可，列表缓存器会处理这些异常情况。
- 其他相关异步请求也可以写在列表缓存器子类中，集中管理这些请求将使代码更清晰。

「TODO」

## 其他记录

NEJ基础控件的生命周期中

实例化方法``_$allocate``会调用``__reset``方法；

``_$recycle``方法会调用``__destroy``方法；

``__reset``方法会把实例化传入的参数中，属性值为函数的属性都认为是事件进行注册

``_$klass``方法返回的是一个函数，这个函数的原型包含了``__init``方法和``_$extend``方法。在调用``_$allocate``实例化时执行new。

