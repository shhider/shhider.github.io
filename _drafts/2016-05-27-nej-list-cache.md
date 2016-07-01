---
layout: post
category: code
title: 「NEJ源码系列」列表缓存管理器详解
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

本文介绍的列表缓存管理器主要是``_$$CacheList``的部分，它实现了列表缓存的关键逻辑。缓存管理基类部分只关注基于内存的缓存结构和请求管理部分；列表缓存管理基类相对比较偏向上层的实现，具体的开发场景并不都适用，比如在蜂巢开发中仅沿用了其声明的事件回调方法、请求逻辑则根据蜂巢需求重新进行了封装，所以这部分将不做详细介绍。

## 最佳实践

一般来说，列表缓存管理器的使用分为以下2步：

1. 根据业务需求实现一个列表缓存管理器子类，比如用户数据相关的``_$$CacheUser``，继承自``_$$CacheList``。在子类中实现加载数据列表、数据项增删改查等事件回调方法，或添加自定义的业务方法；
2. 在相关业务脚本中引入子类文件，实例化列表缓存管理器、调用方法、响应事件进行业务处理。

### 实现列表缓存管理器子类

列表缓存管理器设计了一系列事件，用于推进逻辑执行。这些事件可以分为两种类型：

- 数据加载型事件
- 数据就绪型事件

（原谅我实在想不到更好的命名方式了……）

当列表缓存管理器被调用执行某操作，并且没有正确的缓存数据时，就会触发「数据加载型事件」：“我现在需要最新的数据，谁负责向服务端加载一下，加载完后调用给你的onload方法就行，我已经准备好接收了”。

当负责加载数据的方法取得数据后，调用所给的onload方法，列表缓存管理器将数据缓存成功后触发「数据就绪型事件」：“缓存数据已经更新成功了，大家可以从缓存取数据了”；另外执行操作时，若已经有正确的缓存数据，也会直接触发就绪事件。

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

了解了列表缓存管理器的事件设计后，我们可以发现，数据加载型事件是一个关键的部分，如果没有方法去响应这类事件，列表缓存管理器的逻辑就是断开的、无法达到所需的功能。因此，列表缓存管理器子类中，所要做的就是（按需）指定和实现数据加载型事件响应方法。

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

    return _p;
});
```

#### 载入数据列表

上述代码中，各数据加载方法都会传入一个Object类型的参数，这里将其称为「请求信息」。当调用执行某操作时，列表缓存管理器会根据各参数，组织出请求信息对象，然后跟随对应的数据请求型事件传递给响应方法。请求信息中包含了与服务端进行数据交互所需的参数，数据加载方法根据请求信息发起请求。

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

    // 使用自己封装的请求方法发起请求
    // 或者util/ajax/xdr._$request等NEJ提供的方法
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

在数据加载成功后，记得调用请求信息中的``onload``回调方法，把获取的数据传入（传入的格式要求见后文）。

其他数据加载方法的逻辑，与```__doLoadList``基本类似。大家可以从``util/cache/list._$$CacheList``源码中查阅到各自的请求信息结构，实现请求逻辑。

#### 格式化数据项

在从服务端获取数据后，列表缓存管理器都会把所有数据项，经过一个方法格式化后再存入缓存。其中的格式化方法就是``__doFormatItem``，它是一个接口方法，需要在子类中实现具体的业务逻辑。如果子类中没有实现，数据项就按原格式存入缓存。

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

``__doFormatItem``方法其实非常实用，不过略显低调，没有看到有文档提到过它。使用这个方法，就可以集中进行格式化、避免零散的格式化逻辑；可以统一数据字段结构、避免数据不同状态下字段差异带来的逻辑隐患；可以过滤掉无用的字段，减小缓存占用等。建议在所有的列表缓存管理器子类中都实现该方法。

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

列表缓存管理器对外提供了以下的公开操作方法，对应的事件也一并列出来：

| 操作接口      | 数据加载事件    | 数据就绪事件    |
| ---           | ---             | ---             |
| _$getList     | doloadlist      | onlistload      |
| _$pullRefresh | dopullrefresh   | onpullrefresh   |
| _$getItem     | doloaditem      | onitemload      |
| _$addItem     | doadditem       | onitemadd       |
| _$deleteItem  | dodeleteitem    | onitemdelete    |
| _$updateItem  | doupdateitem    | onitemupdate    |

还是以用户数据为例，现在来实现一个``_$$ModuleUserList``模块，展示用户列表。

#### 实例化

页面模块中引入缓存模块后，首先要实例化一个列表缓存管理器实例出来，后续才可以进行调用。

实例化参数的说明，可以在``util/cache/list``的110行左右找到。

```javascript
/*
 * @param    {Object}   options - 实例化可选配置参数
 * @property {String}   id      - 缓存标识
 * @property {String}   key     - 指定数据项主键，默认为'id'
 * @property {Boolean}  autogc  - 标识是否自动进行数据清理
 * @property {Variable} data    - 供自定义的数据
 */
```

关于实例化参数的各属性，要再细说的是：

- 缓存标识``id``，是一个比较重要的概念，但它是个可选属性、不指定也可正常使用，但建议每次都指定它。后文的原理解析中对其进行介绍；
- 参数``key``用于指定数据项的主键，管理器要根据主键区分数据项，可以参考数据库中主键的概念；
- 参数``autogc``则用于决定数据回收的逻辑。``autogc = true``，则在``_$clearListInCache``方法被调用后会立即进行数据清理；``autogc=false``，则在管理器回收时才进行数据清理。比如，用户列表要提供刷新列表功能时，就需要设置``autogc = true``；
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
            id: 'cache-user',   // 同一个管理器的实例，缓存标识尽量都一致
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

- 属性key为「列表标识」，虽说都是可选属性，但还是建议每次都指定。在其他方法的代码中，列表标识常常以``lkey``出现；
- 属性data需要传入请求的数据。一般请求列表不需要额外的属性，另外调用像``addItem``等方法时data中就是要提交的数据；
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

以上，页面模块显示之后，用户数据缓存管理器就会开始向服务端请求用户数据列表。请求完成、数据缓存就绪后，管理器就会抛出``onlistload``事件。根据在实例化时指定了事件响应，``__showUserList``方法将被调用执行。

``__showUserList``方法中，调用缓存管理器的``_$getListInCache``方法获取数据：

```javascript
/**
 * 展示用户列表
 * 列表缓存就绪后触发
 * @param  {Object} _event 载入列表操作的「请求信息」对象
 * @return {Void}
 */
_pro.__showUserList = function(_event){
    var _userList = this.__cacheUser._$getListInCache('user-list');
    // ...
}
```

列表缓存管理器提供了2个取数据的方法：

- ``_$getListInCache``，从缓存中取数据列表，参数即列表标识；
- ``_$getItemInCache``，从缓存中取数据项，参数即数据项主键的值。

另外，各数据就绪事件抛出的事件对象，即对应操作的请求信息对象。

以上，使用列表缓存管理器加载数据列表的目标就达成了。而其他各操作的逻辑也基本类似，本文就不再细述了，不过**注意**在使用前，请先看一看源码中各操作方法的参数格式。

#### 回收管理器实例

///////////////////////////////

## 原理解析

原理解析部分将介绍3个方面：

- 缓存结构，数据的缓存是如何实现的；
- 各操作的逻辑细节，使用中要注意哪些点；
- 数据回收，如何保证数据正确更新以及回收；

### 缓存结构

列表缓存管理器在其构造函数中声明了一个属性（不需要关心属性名），用于「存放」缓存结构，每个列表缓存管理器实例化时，``this.__cache``属性会指向该缓存结构。

```javascript
// _$$CacheAbstract#__init
this.__cache = this.constructor[_ckey];
```

通过这种形式，所以的列表缓存管理器实例会在同一个缓存结构中读写数据，即使某个列表缓存管理器实例被回收，（未显式clear的）数据依然存在，下次再需要相同标识的列表数据时，仍然可以直接从该缓存结构中获取。这也就达到了页面级的数据缓存。

缓存结构的构成大致如下，可以分为3部分讲解：

```javascript
// this.__cache
{
    // 1
    'cacheId_1': cacheUnit_1,
    'cacheId_2': cacheUnit_2,
    ...
    // 2
    'hash': {...},
    'listKey_1': [...],
    'listKey_2': [...],
    ...
    // 3
    'ckey-l': requestList
}
```

第1部分就是「缓存标识 => 缓存单元(cacheUnit)」。每个列表缓存管理器实例化时，通过缓存标识取得对应的缓存单元（由``this.__lspl``保存引用），相同缓存标识的列表缓存管理器会把数据放在同一个缓存单元中。一个缓存单元的结构如下：

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

缓存的数据项（包括数据列表中的项）都会以「项标识 => 数据项」保存在``hash``中，数据列表还会按列表标识再保存一份。数据列表和``hash``中相同的项，引用的是同一个数据，也就是：``_$getList``缓存的数据列表的项会有两个引用，``_getItem``缓存的数据项只有一个引用。

下面来看第2部分……等等，第2部分不是和缓存单元的结构一样么？没错，这就是一个缓存单元。若管理器实例化时没有指定缓存标识，就会直接把缓存结构作为一个公共缓存单元。基于这一点，建议**实例化列表缓存管理器时都指定缓存标识**，使缓存结构更简单，避免潜在的数据混乱风险。当然，如果单页应用中，某类数据的缓存管理器在各模块单独实例化时，记得指定一样的缓存标识。可以查看``_$$CacheList#__doSwapCache``源码帮助理解。

第3部分就是一个请求列表，保存当前等待响应中请求，无需关心属性名。当某个请求发起时，就把该请求保存到该列表中，之后相同请求不再重复发起。

### 各操作逻辑详解

在分别介绍各操作逻辑前，先看下面的表格。NEJ列表缓存管理器提供的6类操作，都有对应的「请求回调方法」，即各onload调用后所执行的方法。请求回调方法即负责接收数据、按各自策略进行缓存，其中的一些逻辑细节如果不了解，则可能会掉进隐藏的坑里、或者无法更好地利用列表缓存器。

| 操作接口方法  | 操作请求回调    |
| ---           | ---             |
| _$getList     | __getList       |
| _$pullRefresh | __pullRefresh   |
| _$getItem     | __getItem       |
| _$addItem     | __addItem       |
| _$deleteItem  | __deleteItem    |
| _$updateItem  | __updateItem    |

下面详细讲解各操作的逻辑。

#### 获取数据列表

载入数据列表方法``_$getList``被调用后，方法内将执行以下逻辑：

1. 构造「请求信息(ropt)」；
2. 根据列表标识，从缓存中取列表。若列表存在，则直接触发``onlistload``事件，事件参数传入请求信息，方法结束执行；
3. 缓存中没有列表数据，则指定请求回调后触发``doloadlist``事件，抛出请求信息。

```javascript
/**
 * 取列表
 * @method   module:util/cache/list._$$CacheList#_$getList
 */
_pro._$getList = function(){
    // ...
    // _ropt即请求信息
    _ropt.onload = this.__getList._$bind(this,_ropt);
    // ...
};

/**
 * 取列表回调
 * @protected
 * @method module:util/cache/list._$$CacheList#__getList
 * @param  {Object}       _options - 请求信息
 * @param  {Array|Object} _result  - 数据列表，或者带总数信息列表
 * @return {Void}
 */
_pro.__getList = function(_options, _result){
    // ...
    var _list = _result||[];
    if (!_u._$isArray(_list)){
        _list = _result.result||_result.list||[];
        // ...
    }
    // ...
};
```

当onload方法被调用，即开始执行``__getList``回调方法：

1. 检查传入的数据列表，若无数据，则直接到第3步；
2. 有数据列表时，循环列表、**按项**存入缓存、加入缓存列表，同时处理分页相关信息；
3. 触发``onlistload``事件，抛出请求信息对象。

源码中可以看到，``__getList``方法接受2个参数，第一个参数是请求信息，已经在``_$getList``中指定；第二个参数即请求得到的列表，在onload方法调用时传入，格式需要是数据列表(Array)，或者如下的对象：

```javascript
{
    list: [],
    total: 12
}
// 或者
{
    result: [],
    total: 13
}
// 这也许是为了兼容历史遗留问题
```

继续介绍细节，上面第2步中把数据项存入缓存的方法是``__doSaveItemToCache``，它也支持批量缓存数据项。

```javascript
/**
 * 缓存列表项
 *
 * @protected
 * @method module:util/cache/list._$$CacheList#__doSaveItemToCache
 * @param  {Object|Array} arg0 - 列表项或列表
 * @param  {String}       arg1 - 列表标识
 * @return {Object|Array}        缓存中列表项或列表
 */
_pro.__doSaveItemToCache = function(_item,_lkey){
    // ...
    _item = _u._$merge(_oldItem, _newItem);
    // ...
}
```

其执行逻辑如下：

1. 调用``__doFormatItem``方法格式化数据项；
2. 根据列表缓存管理器实例化时指定的项标识，获得该数据项的标识值，然后尝试从``this.__lspl.hash``中取同标识的数据项。
3. 若没有同标识的数据项，则直接存入；若有同标识的数据项，则**合并**数据项内容，然后存入；
4. 返回数据项引用。

**注意**第3步中的「合并」，当已经有同标识数据项存在时，NEJ不是直接覆盖已有数据，而是用``_$merge``方法进行合并。采用这种方式是为了保证已有列表的数据项的正确引用。

我就在这里栽过跟头。在蜂巢开发中，当容器使用了公网时，容器信息会有``pubNet``字段，不实用公网时，接口返回的容器信息直接没有``pubNet``字段，这就造成容器从使用公网设置为不使用公网后，查看容器信息，发现公网设置依然存在。因为``_$merge``方法的特性如下：

```javascript
var obj_1 = {
    'a': 1,
    'b': 2
};
var obj_2 = {
    'b': -2,
    'c': 3
};
// _u是NEJ通用接口实现对象，base/util
var obj_3 = _u._$merge(obj_1, obj_2);

// 执行后
// obj_3 === obj_1
{
    'a': 1,
    'b': -2,
    'c': 3
}
// obj_2 不变
{
    'b': -2,
    'c': 3
}
```

当然一方面这是接口实现不够合理，可以要求后端修改接口，保证数据项的字段一直。不过在此之外，前端方面也最好使用``__doFormatItem``方法对数据项进行格式化，保证所有场景下数据项的字段都是一致的，保证程序的健壮性。

#### 前向刷新列表

「前向刷新列表」，单从这个名称上看，我一时也不清楚具体它是做什么用的，也暂时没有在实际项目中使用过。在阅读其源码后，我理解其功能是类似微博的下拉刷新，调用后将取得新数据插入到数据列表最前面。

它不关注分页等逻辑，因此执行逻辑也要简单：调用后直接抛出``dopullrefresh``事件、执行数据加载。回调方法``__pullRefresh``调用后，将取得的数据插入到对应数据列表的最前面(``__doUnshiftToList``)，随后触发``onpullrefresh``事件、抛出请求信息对象。

#### 获取数据项

获取数据项使用的方法为``_$getItem``，被调用时执行以下逻辑：

1. 构造请求信息；
2. 根据传入的项标识值，从``this.__lspl.hash``中取数据项。若数据项存在**且有效**，则直接触发``onitemload``事件，方法执行结束；
3. 缓存中没有有效的数据，则指定请求回调方法后触发``doloadlist``事件。

注意第2点中的「有效」，列表缓存管理器给每个数据项添加了一个``__dirty``属性，标识当前数据项是否已过期，具体用处在后文垃圾回收部分介绍。

获取数据项的请求回调方法``__getItem``的逻辑要简单一些：调用``__doSaveItemToCache``方法将得到的数据项缓存后，触发``onitemload``事件、抛出请求信息对象。

#### 添加数据项

与前面的操作相比，添加数据项的逻辑会简单一些、并有几点差异。

``_$addItem``不构造请求信息对象，触发``doadditem``事件时抛出的就是其接受的参数对象；回调方法``__addItem``中，触发``onitemadd``事件时传递的也是新生成的一个对象。

```javascript
_pro.__addItem = function(_options,_item){
    // ...
    var _event = {
        key:_key,
        flag:_flag,
        data:_item,
        action:'add',
        ext:_options.ext
    };
    this._$dispatchEvent('onitemadd',_event);
    // ...
};
```



#### 删除数据项

``_$deleteItem``方法接受的配置参数接受4个字段：

删除数据项操作同样把主要的逻辑放在了请求回调方法``__deleteItem``中，``_$deleteItem``方法就是直接触发``dodeleteitem``方法。

请求回调方法要求onload被调用时，传入是否删除成功的Boolean值。若删除成功，回调方法还需要同步内存、将该数据项从指定的列表内删除（splice方法移除列表内该数据项的引用）。

#### 更新数据项

#### _$getListInCache

参数是``_$getList``使用的列表标识。方法中从``this.__lspl``中拿数据。

### 请求数据与垃圾回收

#### _$clearListInCache

本方法清除(splice)``this.__lspl``中列表的数据项（引用）。实际上数据还存在与``this.__lspl.hash``中。

如果``this.__auto``为true，则在之后调用``__doGCAction``方法。

#### __doGCAction

本方法用于从``this.__lspl.hash``结构中彻底删除数据（即去除数据的引用，真正的回收是由浏览器进行的）。``this.__lspl.list``中依然存在的数据将不会被删除。

该方法在列表缓存管理器实例被回收(``__destroy``)时调用；若实例化时指定``autogc: true``，则在每次``_$clearListInCache``之后也会调用。

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

因此，对于不再需要了的数据列表，请在列表缓存管理器回收之前及时使用``_$clearListInCache``方法，或者在实例化时指定``autogc: true``。另外，数据项(item)不需要关心，本身就只在``this.__lspl.hash``里，回收时直接就删除了。

数据列表，没有显式调用``_$clearListInCache``的数据项不会被清楚。

## 小结和建议

从源码层面介绍了NEJ列表缓存管理器后，最后小结几点NEJ列表缓存管理器的使用注意点和建议：

- 即使加载错误，也要调用onload等方法，数据传入null即可，列表缓存管理器会处理这些异常情况。
- 其他相关异步请求也可以写在列表缓存管理器子类中，集中管理这些请求将使代码更清晰。

在NEJ列表缓存管理器的应用中，最大的痛点是请求错误的处理不够灵活。这一点将在后续整理出一套比较适合的解决方案。

「TODO」

因为NEJ列表缓存管理器的细节非常之多，文章中无法一一说明，也难免出现错误。若大家有疑问或发现错误，非常欢迎一起交流。

代码中一系列的考验对Js引用的理解

## 其他记录

NEJ基础控件的生命周期中

实例化方法``_$allocate``会调用``__reset``方法；

``_$recycle``方法会调用``__destroy``方法；

``__reset``方法会把实例化传入的参数中，属性值为函数的属性都认为是事件进行注册

``_$klass``方法返回的是一个函数，这个函数的原型包含了``__init``方法和``_$extend``方法。在调用``_$allocate``实例化时执行new。

