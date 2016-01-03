---
layout: post
title: 关于事件对象中的几种坐标值
category: code
---

> 这是2014年底在公司实习时在内部社区发的几篇文章之一。

在chrome 控制台中打印 click event 信息，发现除了之前已经知道的screenX/screenY、clientX/clientY、pageX/pageY，还有offsetX/offsetY、layerX/layerY、x/y。

各个属性的取值主要是：

- screenX/screenY：以屏幕显示区域为基准，没有异议；
- clientX/clientY：普遍说法是以客户端（浏览器）显示页面的区域为基准（不包括菜单栏、地址栏等），但是测试iframe的情况后，我认为**正确**的说法应该是：以页面(document)被显示的区域为基准；
- pageX/pageY：以整个页面(document)为准，所以当页面没有发生滚动时，clientY == screenY；IE8 及以下不支持；
- offsetX/offsetY：firefox中不支持该属性。在chrome和IE中，以触发事件的元素event.target的区域为基准。所以在使用该值时要**注意**子元素；
- layerX/layerY：从event.target开始，找到设置了position属性的元素，以该元素区域为基准；找不到则以document为基准（等同于pageX/pageY）；IE8及以下不支持；
- x/y：firefox 中不支持该属性；在chrome中等同于clientX/clientY；在IE中，从event.target开始，找到设置了position属性的元素，以该元素区域为基准，找不到则等同于clientX/clientY。以上，事件该属性目前基本没有意义；

另外，后三者的值在IE9+中**有时**会取到了小数点后多位精度。

一般我们常常要用到前3个属性，所以需要对pageX/pageY的值进行兼容。

jQuery中的PageX代码：

```javascript
var body, eventDoc, doc,
// Calculate pageX/Y if missing and clientX/Y available
if ( event.pageX == null && original.clientX != null ) {    // original为原event对象
    eventDoc = event.target.ownerDocument || document;
    doc = eventDoc.documentElement;
    body = eventDoc.body;
    event.pageX = original.clientX + ( doc && doc.scrollLeft || body && body.scrollLeft || 0 ) - ( doc && doc.clientLeft || body && body.clientLeft || 0 );
    event.pageY = original.clientY + ( doc && doc.scrollTop  || body && body.scrollTop  || 0 ) - ( doc && doc.clientTop  || body && body.clientTop  || 0 );
}
```

即 与显示区域的距离 + 滚动的距离 + 文档与显示区域的距离，我画的图太丑，大家自行脑补一下吧……
