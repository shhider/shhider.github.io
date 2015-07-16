---
layout: post
title: NEJ学习、实践笔记
category: code
---

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

### 疑问：如何在URL中携带数据？

背景：手头项目中，需要实现一个产品详情页，希望模块路径是``index.html#/m/product/productName``，或者类似的、可以把产品名保存在URL中的形式，利于用户通过地址直接打开指定产品的详情页。

而就我目前了解的NEJ的模块调度，hash部分都将被解析为模块路径。因此这个问题希望求解。

## 模版系统

## 模块测试
