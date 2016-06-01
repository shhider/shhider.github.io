---
layout: post
category: code
title: 「NEJ源码系列」列表缓存器详解
---

NEJ列表缓存器``_$$CacheList``，实现文件路径：``/src/util/cache/list.js``。
继承自父类``_$$CacheAbstract``，位于同目录的``cache.js``。

## 参数说明

在实例化一个列表缓存器时，同样要传入参数对象，参数对象可传入以下字段：

```javascript
{
    id: 'list-id',      // String, 缓存标识
    key: 'item-id',     // String, 指定数据项的主键，默认为'id'
    autogc: true,       // Boolean，标识是否自动进行垃圾回收
}
```

以上参数中：

- 当autogc设置为true时，在``_$clearListInCache``方法执行后会调用垃圾回收方法``__doGCAction``。

## 关键属性说明

### this.__cache

该属性指向的是一个全局的缓存结构，每个页面只有一个。根据``_$$CacheAbstract``中的源码：

```javascript
this.__cache = this.constructor[_ckey];
```

该缓存结构「保存」在构造函数的属性中（不需要关心属性名），因此，即使某个具体的列表缓存器已经回收，没有显式调用clear方法的列表数据依然存在，下次再需要相同标识的列表数据时，仍然可以从该缓存结构中获取，达到页面层级的缓存。

该缓存结构的形式是「缓存标识 => 缓存对象」，结合接下来介绍的``this.__lspl``理解具体结构。另外它有一个特殊的属性，用于保存请求列表，可以参见``__doQueueRequest``方法的源码进行理解。结构示意如下：

```javascript
// this.__cache
{
	'cache_id_1': lspl_1,
    'cache_id_2': lspl_2,
    ...
    'ckey-l': {...}
}
```

### this.__lspl

这个属性在实例化时调用的``__doSwapCache``方法中初始化，根据传入的缓存标识(id)，从``this.__cache``中取出，若为空，则初始化为对象，添加hash属性，以后会包含以下三 _类_ 属性：

```javascript
// hash    [Object] - item map by id
// list    [Array]  - default list
// x-list  [Array]  - list with key 'x'
this.__lspl = this.__cache[_id];
```

以上属性中：

- hash，保存了数据项的键值对；
- list和x-list是列表标识，此处（以及在下文）只是示意，具体的属性名由调用``_$getList``时指定。

举例数据：

```javascript
{
	hash: {
    	10001: {
        	id: 10001,
            ...
        },
        10002: {
        	id: 10002,
            ...
        },
        ...
    },
    // 调用_$getList时指定了列表标识为'product-list'
    'product-list': [{
    	id: 10001,
        ...
    },{
    	id: 10002,
        ...
    }],
    // 其他列表
}
```


## 常用方法讲解

### _$getList

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

### __doLoadList

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

### __getList

「TODO」

### __doFormatItem

该方法用于格式化数据项，即把后端传的数据，转换到前端需要的格式。基础列表缓存器对象中是一个空方法，我们使用时覆盖即可。

### __doSaveItemToCache

首先调用``__doFormatItem``方法格式化数据项，接着以「数据项主键值 => 数据项」的形式保存到``this.__lspl.hash``中，然后返回数据项的引用。**注意，如果hash中已经存在同主键的数据，则会用``_u._$merge(_olgItem, _newItem)``进行合并。**

### _$getListInCache

参数是``_$getList``使用的列表标识。方法中从``this.__lspl``中拿数据。

### _$clearListInCache

本方法清除(splice)``this.__lspl``中列表的数据项（引用）。实际上数据还存在与``this.__lspl.hash``中。

如果``this.__auto``为true，则在之后调用``__doGCAction``方法。

### __doGCAction

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

「TODO」

## 其他记录

NEJ基础控件的生命周期中

实例化方法``_$allocate``会调用``__reset``方法；

``_$recycle``方法会调用``__destroy``方法；


``_$klass``方法返回的是一个函数，这个函数的原型包含了``__init``方法和``_$extend``方法。在调用``_$allocate``实例化时执行new。

