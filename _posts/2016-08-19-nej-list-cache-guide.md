---
layout: post
category: code
title: 「NEJ源码系列」列表缓存管理器(1)——使用指南
---


## 引言

在单页应用中，几乎所有的业务数据都是通过异步请求接口获取的，随着项目的发展，页面中的请求会越来越多，如果不加以管理就会出现大量雷同的请求逻辑，遇到接口调整、需求变更时也会容易出错。

NEJ框架针对这样的情况，把异步请求中常见的「增删改查」操作进行抽象封装，并提供数据缓存特性，得到了「列表缓存管理器」控件。

NEJ列表缓存管理器对应用开发最显著的两点优势是：

- 集中管理应用中的异步请求，相同的请求逻辑只需写一次，具体页面中就只需要关心业务逻辑；
- 快速实现对数据的缓存，减少无谓的网络请求，提升页面性能和用户体验；

本文将从源码出发，结合我的一些使用体会，介绍NEJ列表缓存管理器的使用方法和实现原理。

## 列表缓存管理器的继承关系

查看NEJ目录``/src/util/cache/``目录下的文件，关于列表缓存、存在继承关系的的类有3个：

- **缓存管理基类**：[util/cache/cache._$$CacheAbstract](https://github.com/genify/nej/blob/master/src/util/cache/cache.js)，继承NEJ控件基类。它提供了基于内存/本地存储的缓存管理、请求管理等实现；
- **列表缓存管理器**：[util/cache/list._$$CacheList](https://github.com/genify/nej/blob/master/src/util/cache/list.js)，继承``_$$CacheAbstract``。它提供了更具体的缓存结构、事件、数据列表管理（包括分页）、数据项管理（增删改查）、垃圾回收等实现；
- **列表缓存管理基类**：[util/cache/abstract._$$CacheListAbstract](https://github.com/genify/nej/blob/master/src/util/cache/abstract.js)，继承``_$$CacheList``。它指定了几个关键事件的回调接口方法；对请求逻辑再封装，支持在项目中统一管理接口地址、请求错误处理等。

本文介绍的列表缓存管理器主要是``_$$CacheList``的部分，它实现了列表缓存的关键逻辑。缓存管理基类部分只关注基于内存的缓存结构和请求管理部分；列表缓存管理基类相对比较偏向上层的实现，因具体场景而异，比如在蜂巢开发中仅沿用了其声明的事件回调方法、请求逻辑则根据蜂巢需求重新进行了封装，所以这部分将不做详细介绍。

## 最佳实践

一般来说，列表缓存管理器的使用分为以下2步：

1. 根据业务需求实现一个列表缓存管理器子类，比如用户数据相关的``_$$CacheUser``，从``_$$CacheList``继承。在子类中实现加载数据列表、数据项增删改查等事件回调方法，或添加自定义的业务方法；
2. 在相关业务脚本中引入子类文件，实例化列表缓存管理器、调用方法、响应事件进行业务处理。

### 实现列表缓存管理器子类

列表缓存管理器设计了一系列事件，用于推进逻辑执行。这些事件可以分为两种类型：

- 数据加载型事件
- 数据就绪型事件

（原谅我实在想不到更好的命名方式了……）

当列表缓存管理器被调用获取某数据，而缓存结构中没有所需的数据时，就会触发「数据加载型事件」：“我现在需要某某数据，但是我在缓存中没有找到，谁负责向服务端加载一下，加载完后调用给你的onload方法就行，我已经准备好接收了”。

当负责加载数据的事件回调执行、并取得数据后，调用所指定的onload方法，列表缓存管理器将数据进行缓存、并触发「数据就绪型事件」：“所需的数据已经在缓存结构中，大家可以直接从缓存取数据了”；另外执行操作时，若已经有正确的缓存数据，也会直接触发就绪事件。

列表缓存管理器目前提供了以下事件：

```javascript
// 数据加载型事件
doloadlist      // 向服务端请求数据列表
dopullrefresh   // 向服务端前向刷新数据列表
doloaditem      // 向服务端请求数据项
doadditem       // 向服务端添加数据项
dodeleteitem    // 向服务端删除数据项
doupdateitem    // 向服务端更新数据项

// 数据就绪型事件
onlistload      // 数据列表 载入成功
onpullrefresh   // 数据列表 前向载入成功
onitemload      // 数据项 载入成功
onitemadd       // 数据项 添加成功
onitemdelete    // 数据项 删除成功
onitemupdate    // 数据项 更新成功
```

大概了解列表缓存管理器的事件设计后可以发现，数据加载型事件是一个关键的部分，如果没有指定方法去响应对应的事件，列表缓存管理器的逻辑就是断开的、无法达到所需的功能。因此，列表缓存管理器子类中，所要做的就是指定和实现数据加载型事件响应方法。

以常见的用户数据为例，用户数据缓存管理器子类``_$$CacheUser``大致是这样的：

```javascript
NEJ.define([
    'base/klass',
    'util/cache/list'
], function(_k, _t, _p, _o, _f, _r){
    var _pro;

    _p._$$CacheUser = _k._$klass();
    // 继承列表缓存管理器基类
    _pro = _p._$$CacheUser._$extend(_t._$$CacheList);

    /**
     * 控件重置方法中绑定事件
     */
    _pro.__reset = function(_options){
        this.__super(_options);
        this._$batEvent({
            doloadlist:    this.__doLoadList._$bind(this),
            dopullrefresh: this.__doPullRefresh._$bind(this),
            doloaditem:    this.__doLoadItem._$bind(this),
            doadditem:     this.__doAddItem._$bind(this),
            dodeleteitem:  this.__doDeleteItem._$bind(this),
            doupdateitem:  this.__doUpdateItem._$bind(this)
        });
        // 这里用了和_$$CacheListAbstract同名的回调方法
        // 你也可以直接继承_$$CacheListAbstract，就不用重写__reset方法了
    };

    _pro.__doLoadList = function(_options){
        // TODO
    };

    _pro.__doPullRefresh = function(_options){ /* TODO */ };
    _pro.__doLoadItem    = function(_options){ /* TODO */ };
    _pro.__doAddItem     = function(_options){ /* TODO */ };
    _pro.__doDeleteItem  = function(_options){ /* TODO */ };
    _pro.__doUpdateItem  = function(_options){ /* TODO */ };
    // 后文中，我将称这些响应数据加载事件的方法为「数据加载方法」
    return _p;
});
```

#### 载入数据列表

上述代码中，各数据加载方法都会传入一个Object类型的参数，这里将其称为「请求信息」。当调用执行某操作时，列表缓存管理器会根据各参数，组织出请求信息对象，然后跟随事件传递给对应的数据加载方法。请求信息中包含了与服务端进行数据交互所需的参数，数据加载方法根据请求信息发起请求。

以载入数据列表为例，它的请求信息中包含了以下字段：

```javascript
/**
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

所以，载入数据列表方法``__doLoadList``中大致是这样的：

```javascript
_pro.__doLoadList = function(_options){
    var _url = '...';

    // ...

    // 使用util/ajax/xdr._$request等NEJ提供的方法
    // 或者自己封装的请求方法发起请求
    this.__doSendRequest(_url, {
        method: 'GET',
        data: _options.data,
        // ...
        onload: function(_res){
            // ...
            _options.onload(_result);
        },
        onerror: function(_err){
            // 错误处理逻辑
            // ...
        }
    });
}
```

在数据加载成功后，需要调用请求信息中的``onload``回调方法，把获取的数据交给“准备好接受”的方法处理（传入的格式要求见后文）。

其他数据加载方法的逻辑，与``__doLoadList``基本类似。大家可以从``util/cache/list._$$CacheList``源码中查阅到各自的请求信息结构，实现请求逻辑。

#### 格式化数据项

在从服务端获取数据后，列表缓存管理器都会把所有数据项，经过``__doFormatItem``方法格式化后再存入缓存。该方法在列表缓存管理器基类中是一个空函数，需要在子类中根据具体业务场景实现具体逻辑。上源码：

```javascript
/**
 * 格式化数据项，子类实现具体业务逻辑
 *
 * @protected
 * @method module:util/cache/list._$$CacheList#__doFormatItem
 * @param  {Object} arg0 - 数据项
 * @param  {String} arg1 - 列表标识
 * @return {Object}        格式化后的数据项
 */
_pro.__doFormatItem = _f;

/**
 * 缓存数据项
 * @method module:util/cache/list._$$CacheList#__doSaveItemToCache
 */
_pro.__doSaveItemToCache = function(_item,_lkey){
    // ...
    _item = this.__doFormatItem(_item,_lkey) || _item;
    // ...
};
```

``__doFormatItem``方法其实非常实用，不过略显低调，没有看到有文档提到过它。使用这个方法，就可以集中进行格式化、避免零散的格式化逻辑；可以统一数据字段结构、避免数据在不同状态下的字段差异带来逻辑隐患；可以过滤掉无用的字段，减小缓存占用等。建议在所有的列表缓存管理器子类中都实现该方法。

```javascript
_pro.__doFormatItem = function(_item, _lkey){
    // Do It.
};
```

#### 自定义方法

除了NEJ提供的这些操作外，我们还可以在子类中实现自定义方法，以进行其他特殊的数据交互。这里要提的一点建议是，尽量统一风格、用事件的方式来推进逻辑的执行。

```javascript
// Bad
_pro._$doSomething = function(_options, _callback){
    // ...
    _callback(...);
};

// Good
_pro._$doSomething = function(_options){
    // ...
    this._$dispatchEvent('onsomethingsuccess', _obj);
};
```

以上，一个列表缓存管理器子类就基本上实现了。另外，你还可以根据项目全局需要，再抽象一层进行封装，如全局统一处理请求错误等。

### 使用列表缓存管理器

上一节中说了这么多操作的事件和响应，那么要怎么去调用这些操作呢？

列表缓存管理器对外提供了以下的公共方法，对应的事件也一并列出来：

| 公共操作方法    | 说明 | 数据加载事件    | 数据就绪事件    |
| ---           | --- | ---             | ---             |
| _$getList     | （分页）获取数据列表 | doloadlist      | onlistload      |
| _$pullRefresh | 前向更新数据列表 | dopullrefresh   | onpullrefresh   |
| _$getItem     | 获取数据项 | doloaditem      | onitemload      |
| _$addItem     | 添加数据项 | doadditem       | onitemadd       |
| _$deleteItem  | 删除数据项 | dodeleteitem    | onitemdelete    |
| _$updateItem  | 更新数据项 | doupdateitem    | onitemupdate    |

还是以用户数据为例，现在来实现一个``_$$ModuleUserList``模块，展示用户列表。

#### 实例化

页面模块中引入缓存模块后，首先要实例化一个列表缓存管理器。

实例化参数的说明，可以在``util/cache/list``的110行左右找到。

```javascript
/*
 * @param    {Object}   options - 实例化配置参数
 * @property {String}   id      - 缓存标识，默认为空
 * @property {String}   key     - 指定数据项主键，默认为'id'
 * @property {Boolean}  autogc  - 标识是否自动进行数据清理，默认为false
 * @property {Variable} data    - 供自定义的数据
 */
```

关于实例化参数的各属性，要再细说的是：

- 缓存标识``id``，是一个比较重要的概念，但它是个可选属性、不指定也可正常使用，但建议每次都指定它。本文原理解析部分将对其进行介绍；
- 参数``key``用于指定数据项的主键，管理器要根据主键区分数据项，可以参考数据库中主键的概念；
- 参数``autogc``则用于决定数据回收的逻辑。默认``autogc``为false时，``_$clearListInCache``方法只是从列表中去除数据项的引用，数据项依然在缓存结构中，下次获取数据列表时再将新得到的数据项合并；而将``autogc``设置为true，在``_$clearListInCache``方法执行后就会紧跟着完全清除缓存结构中孤立的数据项引用，下次获取数据列表或数据项时将是新的引用；
- 除以上4个参数外，其他Function类型的属性都会被作为事件进行绑定。

来看用户列表页的实例化代码，我们把实例化逻辑放在页面模块显示时(``__onShow``or``__onRefresh``)进行：

```javascript
NEJ.define([
    'base/klass',
    'util/dispatcher/module',
    'pro/path/to/cache/user'
], function(_k, _dm, _cache, _p){
    var _pro;

    _p._$$ModuleUserList = _k._$klass();
    _pro = _p._$$ModuleUserList._$extend(_dm, _$$ModuleAbstract);

    // ...

    _pro.__onShow = function(_options){
        this.__super(_options);
        // ...
        this.__cacheUser = _cache._$$CacheUser._$allocate({
            id: 'user',   // 同一个管理器的实例，缓存标识尽量都一致
            key: 'userId',
            autogc: true,
            data: {...},        // 不需要自定义的数据，就不需要指定

            onlistload: this.__showUserList._$bind(this),
            // 其他事件回调...
        });
    };

    _pro.__showUserList = function(_event){
        // TODO
    }
});
```

#### 加载数据列表

用户数据缓存管理器实例化后，就可以调用``_$getList``方法开始加载用户列表了。

在写代码前先看一下``_$getList``方法的参数中需要哪些属性：

```javascript
/**
 * @method   module:util/cache/list._$$CacheList#_$getList
 * @param    {Object} arg0   - 可选配置参数
 * @property {String} key    - 列表标识
 * @property {Number} data   - 请求数据信息
 * @property {Number} offset - 偏移量
 * @property {Number} limit  - 数量
 * @property {Object} ext    - 回传数据
 * @return   {Void}
*/
```

以上参数的属性中：

- 属性key为「列表标识」，虽说都是可选属性，但还是建议每次都指定（0.2.9之前的版本不指定key的话可能会出现函数调用死循环的错误）。在管理器的源码中，列表标识常常以``lkey``出现；
- 属性data需要传入请求的数据，像``addItem``等方法时data中就是要提交的数据；
- 属性offset、limit是分页相关参数，会在之后的「NEJ分页列表模块」文章中再详细介绍。

所以，调用的代码如下：

```javascript
_pro.__onShow = function(_options){
    this.__super(_options);
    // ...
    this.__cacheUser = _cache._$$CacheUser._$allocate({
        // ...
    });
    this.__cacheUser._$getList({
        key: 'user-list',
        // 其他需要的参数
    });
};
```

以上，页面模块显示之后，用户数据缓存管理器就会开始向服务端请求用户数据列表。若已经有缓存的数据列表，将直接触发``onlistload``事件；否则会先向服务端请求数据，缓存就绪后再触发事件。根据在实例化时指定的事件响应，``__showUserList``方法将被调用执行。

``__showUserList``方法中，通过缓存管理器的``_$getListInCache``方法获取列表：

```javascript
/**
 * 展示用户列表
 * 列表缓存就绪后触发
 * @param  {Object} _event 载入列表操作的「请求信息」对象
 * @return {Void}
 */
_pro.__showUserList = function(_event){
    var _userList = this.__cacheUser._$getListInCache('user-list');
    // 方法返回的必定是一个数组，不用进行类型的检查
    // ...
}
```

列表缓存管理器提供了2个取数据的方法：

- ``_$getListInCache``，从缓存中取数据列表，参数即列表标识；
- ``_$getItemInCache``，从缓存中取数据项，参数即数据项主键的值。

通过列表缓存管理器加载数据列表到这就完成了，其他增删改查等操作的逻辑也基本类似，本文就不再细述了，不过**注意**在使用前，请先看一看源码中各操作方法的参数格式。

#### 清除缓存数据

有了获取数据的方法，当然也有清除数据的方法，以满足随时获取最新数据的需求。

列表缓存管理器提供了2个清除数据的方法：``_$clearListInCache``和``_$clearItemInCache``。

```javascript
/**
 * 清除缓存列表
 *
 * @method module:util/cache/list._$$CacheList#_$clearListInCache
 * @param  {String} _lkey - 列表标识
 * @return {Void}
 */
_pro._$clearListInCache = function(_key){
    // ...
};

/**
 * 清除缓存项
 *
 * @method module:util/cache/list._$$CacheList#_$clearItemInCache
 * @param  {String} _id - 项标识
 * @return {Void}
 */
_pro._$clearItemInCache = function(_id){
    // ...
};
```

在调用清除数据方法后，若再获取数据列表或数据项时，将再次发起请求、从服务端加载最新数据。一般来说，我会在离开页面模块、管理器实例回收时调用它们：

```javascript
_pro.__onHide = function(){
    this.__cacheUser._$clearListInCache('user-list');
    // or
    this.__cacheUser._$clearItemInCache(2333);

    this.__cacheUser._$recycle();
    this.__super();
};
```

当调用``_$clearListInCache``时没有指定列表标识时，将会清除当前缓存结构中所有的数据列表缓存。

另外不得不提醒，在前面实例化部分也已经提到过，默认情况下调用``_$clearListInCache``，只是从列表中删除了数据项的引用；``_$clearItemInCache``也只是以标志位标识数据项过期，并不是真正从缓存结构中删除数据。之后有新的数据时，是以数据项为单元，与旧数据项进行**合并**，这一特性是为了保证数据项的引用地址保持不变。基于这样的情况，如果要真正的删除数据列表和数据项，排除旧数据的影响，就要靠``autogc=true``加上``_$clearListInCache``。

> 以上，列表缓存管理器的使用部分就介绍得差不多了，敬请期待原理解析部分~
