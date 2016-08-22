---
layout: post
category: code
title: 「NEJ源码系列」列表缓存管理器(2)——原理解析
---

原理解析部分将介绍3个方面：

- 缓存结构，数据的缓存是如何实现的；
- 各操作的逻辑细节，使用中要注意哪些点；
- 数据回收，如何保证数据正确更新以及回收；

## 缓存结构

直接上图：

（曾经我以为我能通过文字说明清楚，但最后还是删掉了大段文字、画了下面这张图……）

![列表缓存器的缓存机制和缓存结构](../public/img/cache-list-1.png)

列表缓存管理器在其构造函数中声明了一个属性（不需要关心属性名），指向一个缓存结构——用于「存放」缓存数据。

```javascript
// @method module:util/cache/cache._$$CacheAbstract#__init
_pro.__init = function(){
    // ...
    this.__cache = this.constructor[_ckey];
    // ...
};
```

每个列表缓存管理器实例化时，``this.__cache``属性会指向该缓存结构。

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

## 各操作逻辑详解

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

### 获取数据列表

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

### 前向刷新列表

「前向刷新列表」，单从这个名称上看，我一时也不清楚具体它是做什么用的，也暂时没有在实际项目中使用过。在阅读其源码后，我理解其功能是类似微博的下拉刷新，调用后将取得新数据插入到数据列表最前面。

它不关注分页等逻辑，因此执行逻辑也要简单：调用后直接抛出``dopullrefresh``事件、执行数据加载。回调方法``__pullRefresh``调用后，将取得的数据插入到对应数据列表的最前面(``__doUnshiftToList``)，随后触发``onpullrefresh``事件、抛出请求信息对象。

### 获取数据项

获取数据项使用的方法为``_$getItem``，被调用时执行以下逻辑：

1. 构造请求信息；
2. 根据传入的项标识值，从``this.__lspl.hash``中取数据项。若数据项存在**且有效**，则直接触发``onitemload``事件，方法执行结束；
3. 缓存中没有有效的数据，则指定请求回调方法后触发``doloadlist``事件。

注意第2点中的「有效」，列表缓存管理器给每个数据项添加了一个``__dirty``属性，标识当前数据项是否已过期，具体用处在后文垃圾回收部分介绍。

获取数据项的请求回调方法``__getItem``的逻辑要简单一些：调用``__doSaveItemToCache``方法将得到的数据项缓存后，触发``onitemload``事件、抛出请求信息对象。

### 添加数据项

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



### 删除数据项

``_$deleteItem``方法接受的配置参数接受4个字段：

删除数据项操作同样把主要的逻辑放在了请求回调方法``__deleteItem``中，``_$deleteItem``方法就是直接触发``dodeleteitem``方法。

请求回调方法要求onload被调用时，传入是否删除成功的Boolean值。若删除成功，回调方法还需要同步内存、将该数据项从指定的列表内删除（splice方法移除列表内该数据项的引用）。

### 更新数据项

### _$getListInCache

参数是``_$getList``使用的列表标识。方法中从``this.__lspl``中拿数据。

## 请求数据与垃圾回收

### _$clearListInCache

本方法清除(splice)``this.__lspl``中列表的数据项（引用）。实际上数据还存在与``this.__lspl.hash``中。

如果``this.__auto``为true，则在之后调用``__doGCAction``方法。

### __doGCAction

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

