---
layout: post
category: code
title: NEJ生命周期介绍，以及模块开发建议
---


NEJ的模块调度系统中，模块的声明周期有以下几个步骤：

\_\_init: 该方法会调用\_\_doBuild方法，所以你在子类的\_\_init方法中\_\_super方法的位置，决定了那个方法先执行。



\_\_onShow是在模块显示时触发；

\_\_onRefresh是在每次URL变化时触发，另外在\_\_onShow方法中也会调用一次；

所以在开发过程中，\_\_super方法的执行位置不同会有不同的影响。

建议根据以上生命周期的介绍，合理的控制\_\_super的执行顺序。


建议在\_\_doBuild方法中初始化节点变量；

在\_\_init方法中初始化需要的数据变量；



``__onShow``和``__onRefresh``的参数中：

```javascript
href   : "/m/repository/version/?repoId=201"
referer: "/m/repository/?repoId=201"
source : "/m/repository/version/"
target : "/m/repository/version/"
umi    : "/m/repository/version/"
```
