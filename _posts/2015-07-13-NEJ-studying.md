---
layout: post
title: NEJ学习、实践笔记
category: code
---

= = = = = = = = Update(2015-08-15) = = = = = = = =

## ajax提交

我们在用NEJ进行ajax提交时，一般会用``util/ajax/rest``和``util/ajax/xdr``来实现。

当使用前者``util/ajax/rest``时，所发出的请求的``Content-Type``字段值为``application-json``，数据部分格式也直接是json字符串格式。一切感觉很和谐，唯独遗憾的是，现在一些后台对``application-json``形式的请求支持度不高，需要开发者一番特殊处理才能拿到数据。

而后者``util/ajax/xdr``主要是基本的一些请求方式，各种后台的支持情况应该是都很好。但是在使用中却发现，进行POST请求时，后台还是无法拿到数据。这是怎么回事呢？

这一节就是要来解决这个问题。打开F12来查看一下请求，我们看到：``Content-Type``是正常的``application/x-www-form-urlencoded``，但请求的数据却是``[object object]``。一般，我们是这样调用``_$request``的：

```javascript
_j._$request('/api/test', {
  method: 'POST',
  type: 'JSON',
  data: {
    username: name,
    password: pwd
  },
  onload: function(res){}
  /* more */
});
```

请求数据保存在data属性中，一般就是简单对象类型。所以一定在哪里被toString了。

经过控制台的一路跟踪，一直到发出请求的最后一步，data属性都没有被处理过。来看看最后一步是这样的：

```javascript
/* util/ajax/proxy/xhr.js, line: 130 */
this.__xhr.send(_request.data);
```

那么问题应该就出在这，对于xhr的send方法，查阅资料得到：

```javascript
void send([data = null]);
/**
 * data参数可选，支持以下类型：
 * - ArrayBuffer
 * - Blob
 * - Document
 * - DomString(string)*
 * - FormData
 * 
 * 对以上这些类型的解释，可以移步张鑫旭的博客：
 * http://www.zhangxinxu.com/wordpress/2013/10/understand-domstring-document-formdata-blob-file-arraybuffer/
 */
```

可以看到，简单对象类型在send方法并不被接受。简单对象类型data在这里被强制toString，变成了[object object]。

所以我们在这之前要对data进行转换成上面的其中一种类型。

一般来说，ajax请求的``Content-Type``都是``application/x-www-form-urlencoded``（还有比较新的``application/json``）。大部分后台程序面对``application/x-www-form-urlencoded``请求，会把请求数据部分按照``aa=xx&&bb=yy&&cc=zz``这样的格式来解析。

所以了，我们就直接把data转换成上面说的需要的格式。先来测试一下，把send的参数转换成字符串：

```javascript
this.__xdr.send(_u._$object2string(_request.data, '&'));
```

经过测试，果然发出了正确的请求，并获得正确的响应。

不过，直接在这一句转换类型是不合理的，因为有时候可能会有其他Content-Type的请求，最后都会走这一句。所以针对我们针对``application/x-www-form-urlencoded``来进行单独处理。本来我是想模仿NEJ处理ajx上传文件时的做法，使用FormData对象来转换成可用的类型：

```javascript
/* util/ajax/proxy/xhr.js, line: 92 */
if (_headers[_g._$HEAD_CT]===_g._$HEAD_CT_FILE){
    // ...
    _request.data = new FormData(_request.data);
    // ...
}
```

但是根据资料：[FormData - Web API 接口| MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/FormData)，IE9及以下版本IE不支持FormData对象。并且，我查找了NEJ代码，没有相关的兼容性处理，所以我在这里还是选择以字符串形式处理，以保证基本的兼容性，如下：

在send之前检查Content-Type类型，如果是``application/x-www-form-urlencoded``，那么就把参数转换成所需格式的字符串(string)。

```javascript
if(_headers[_g._$HEAD_CT]===_g._$HEAD_CT_FORM){
    _request.data = _u._$object2string(_request.data, '&');
}
```

另外有一点就是，相关方法在处理请求参数时，如果没有设置Content-Type，就会默认设置为``application/x-www-form-urlencoded``（util/ajax/proxy/proxy.js, line: 88）。当使用``util/ajax/rest``时，NEJ会自动设置Content-Type为``application/json``；而如果在设置请求参数时设置Content-Type为``application/x-www-form-urlencoded``，那么当前处理方法也并不能正确地发出请求。所以默认情况下，笔者所做的改动并不会影响到rest请求。

= = = = = = = = Update(2015-07-22) = = = = = = = =

NEJ，Nice Easy JavaScript，出自网易前端大神——genify之手，是网易产品线中使用最广泛的前端框架。

- NEJ官网：[http://nej.netease.com/](http://nej.netease.com/)
- Github仓库：[https://github.com/genify/nej](https://github.com/genify/nej)
- 开发文档：[nej/doc Github](https://github.com/genify/nej/tree/master/doc);
- API手册：[http://nej.netease.com/help/index.html](http://nej.netease.com/help/index.html)

此处省略1w字的对NEJ强大之处的描述，以及对飞哥的敬仰~

进入正题，从代码和API手册上，可以看到NEJ的功能点非常多。而也因为这么多的功能，我不知道我要实现的某个功能是否在框架中已经实现；当我使用某个功能时，我不清楚需要注意点什么，即使有在看文档。

下面，笔者记录一些比较关键、常用的使用点。由于笔者对NEJ的使用现在还不多，以下的内容都是探索中的记录，因此不免有错误、误解之处，望各位大神批评指正。

## 模块调度系统

NEJ提供了一套模块调度系统用于支持单页富应用的系统架构、模块拆分和重组、模块调度管理等功能。

请先阅读[这篇文档](https://github.com/genify/nej/blob/master/doc/DISPATCHER.md)，详细地讲解了模块调度系统。

模块调度系统的实现文件路径是``util/dispatcher``，目录中包括``dispatcher``和``module``。前者是模块调度器的实现文件，后者是模块基类实现文件。

### 配置项中的“/”符号

在开始写代码前，需要对项目进行分析、划分模块，并确定各模块的依赖关系树。在调度器的实例化代码中，配置这些模块。

从上面说的文档的一些模块配置的例子中，笔者发现一些配置项最后带了“/”，而有些没有。

根据试验，配置项最后不带“/”符号，则表示当前模块是一个父模块；反之，最后是“/”符号，则该模块是一个“叶子”模块，没有子模块了。

这里看一个例子。

```javascript
  modules:{
      '/?/head/':'module/head/index.html',
      '/?/blog/tab/':'module/blog/tab/index.html',
      '/m':{
          module:'module/layout/index.html',
          composite:{
              head:'/?/head/'
          }
      },
      '/m/blog':{
          module:'module/layout/blog/index.html',
          composite:{
              tab:'/?/blog/tab/'
          }
      },
      '/m/blog/tag/':'module/blog/tag/index.html',
      '/m/blog/list/':'module/blog/list/index.html',
  }
```

当访问``/index.html#/m/blog/list/``页面时，调度系统解析Hash：``/m/blog/list/``，制定了解析顺序：m > blog > list。前项即为后项的父模块，后项为前项的子模块。

根据配置，父模块``/m``指定了模块页面，因此系统会加载`module/layout/index.html`中的内容。

```javascript
_pro.__doBuild = function(){
    this.__body = _e._$html2node(
        _e._$getTextTemplate('module-layout')
    );
    var _list = _e._$getByClassName(this.__body, 'j-flag');
    this.__export = {
        head: _list[0],
        parent: _list[1]
    };
};
```

根据该父模块的``__export``属性，和配置中的``composite``属性，系统将加载``/?/head/``->``module/head/index.html``中的内容到指定的节点中；同时，接下来的子模块``/m/blog``会加载到``parent``指定的节点中。``parent``是默认的，无需在配置中声明。

同理，``/m/blog``将进行加载，接着是``/m/blog/list/``，这是一个“叶子”模块，系统会忽略``composite``和``__export``的配置，不再往下加载。

所以，在配置模块调度的时候，要特别注意最后的这个“/”符号。

### 模块实现

一个模块包含3大元素：

- 样式：定义模块的效果
- 结构：定义模块的结构
- 逻辑：实现模块的功能

简单地说，还是CSS、HTML、JavaScript这三个部分。

模块的逻辑部分从模块基类``dispatcher/module``扩展得来，需要实现以下几个关键接口：

- \_\_doBuild：首先执行，构建模块结构，缓存模块需要使用的节点，初始化组合控件的配置参数；
- \_\_onShow：其次执行，将模块放置到指定的容器中，分配组合控件，添加事件，调用``__onRefresh``；
- \_\_onRefresh：根据外界输入的参数信息获取数据并展示；
- \_\_onHide：模块放至内存中，回收在``__onShow``中分配的组合控件和添加的事件，回收``__onRefresh``中产生的视图（这里尽量保证执行完成后恢复到``__doBuild``后的状态）

这里主要记录一下这几个接口的执行顺序。当模块载入时，首先执行``__doBuild``；在显示时（前、后取决于``__super``方法在何时执行）执行``__onShow``方法，过程中会调用``__onRefresh``方法；当页面转向其他模块时，会最后执行``__onHide``方法，尽可能使状态回复到``__doBuild``之后的状态；当再次显示时，再执行``__onShow``和``__onRefresh``方法。

### 关于模块调度系统的其他

1、实例化模块调度器后，NEJ会将该调度器添加到页面全局。后续可以直接调用。

```javascript
// util/dispatcher/dispatcher, line: 1231
window.dispatcher =
    _p._$$Dispatcher.
    _$getInstance(_options);
```

### 在URL中携带查询数据

背景：手头项目中，需要实现一个产品详情页，希望模块路径是``index.html#/m/product/productName``，或者类似的、可以把产品名保存在URL中的形式，利于用户通过地址直接打开指定产品的详情页。

~~ 更新 ~~

之前在文档和代码中没有看到有关查询数据（?aa=11&bb=22）的说明，所以以为NEJ并没有实现查询数据相关。所以当时原本打算自己从location中提取query数据。

然而，在尝试的过程中，我发现hash部分和query部分的顺序前后会出现不同的问题。

如``index.html#/m/app/?aa=11``和``index.html?aa=11#/m/app/``，两者查看location时：

```javascript
// 前者
location.hash = '#/m/app/?aa=11'
location.search = ''
// 后者
location.hash = '#/m/app/'
location.search = '?aa=11'
```

前者没发现什么副作用，只是不知道怎么取到query值；后者则在模块跳转后，query仍在url中。

今天又碰到了这个障碍，所以又看了看相关代码，终于发现，其实NEJ已经帮我们做了这块工作，刚才的前后两种情况都会把查询数据解析出来，只是最后没有保存到location对象中。不过在onbeforechange事件传入的参数中，还带着query数据，只要保存到location对象中，后续就可以取用了。

## 模版系统

## 模块测试


## 其他

1、像模块、控件等模块化开发中一个概念就是“生命周期”，如``__init``到``__show``再到``__onRefrsh``等等这样的过程。引入生命周期除了带来很好的扩展性外，更重要的是理顺了组件的各个阶段，有助于更好的理解和运用。

控件的生命周期：\_\_init（调用了\_\_initXGui和\_\_initNode）（第一次实例化时执行） > \_\_reset（每次使用都执行） >  \_\_destroy（每次移除时执行）

2、在查看控件的回收（\_\_recyle）实现时，发现一些操作其实是绑定在构造函数上，然后通过原型进行调用。所以之后在查看某方法的实现时，尽量避免迷惑；

在继续对recycle方法追究的过程中。还学习到几点：

- 在定义一个子类型时，某方法是其原型链中已经存在的方法，那么重写该方法就会屏蔽原来的那个方法。NEJ则添加了\_\_super方法，去调用执行上一个同名方法；
- UI控件在\_\_recyle方法调用后，节点也会从页面中去除。我一开始想，肯定是有个过程中执行了removeChild方法，而在追寻调用链的过程中，却没有看到这个方法。经过定位，最后发现节点是在原生方法appendChild执行后被去除的（base/element#_$removeByEC）。查看文档——[Node.appendChild() - Web API Interfaces | MDN](https://developer.mozilla.org/en-US/docs/Web/API/Node/appendChild)，可以看到这样一段话：

> The Node.appendChild() method adds a node to the end of the list of children of a specified parent node. **If the given child is a reference to an existing node in the document, appendChild() moves it from its current position to the new position** (i.e. there is no requirement to remove the node from its parent node before appending it to some other node).

这其实是一个很常用的方法，但是平时我却没有注意到这一点。

- NEJ的\_\_super方法实现中，使用了arguments.callee.caller，而该属性在ES6中已经被移除，所以NEJ一定程度上不兼容ES6；
