---
layout: post
title: 刮刮卡效果的实现和优化
category: code
---

> 这是2014年底在公司实习时在内部社区发的几篇文章之一。

js也学了有段时间了，但是一直没有学习使用canvas。所以今天就来上手实践一下。

- 参照目标：[http://www.lightapp.cn/auto/index/4085](http://www.lightapp.cn/auto/index/4085) （建议用手机打开或Chrome DevTool模拟手机）
- 我目前实现的效果：[http://1.shhider.sinaapp.com/gagale.html](http://1.shhider.sinaapp.com/gagale.html)

   
## 页面结构

结构非常简单：

```html
<body>
    <div id="bd">
        <div class="pic"></div>
        <!-- canvas -->
    </div>
</body>
```

- bd节点是主体区域，它的宽高会根据窗口大小进行适应：PC等大屏设备上设置宽度并居中；手机等设备则满屏；
- pic节点用于放置要显示的图片，绝对定位，初始会被canvas层遮盖住；
- canvas层会在页面加载完成后，由js动态插入，以适应主体区域。绝对定位，z-index属性值要大于pic节点；

```css
#bd .pic {
    position: absolute;
    top: 0; left: 0; z-index: 1;
    width: 100%; height: 100%;
    background: url(./static/img/porsche.jpg) no-repeat center;
    background-size: cover;
}
#bd canvas {
    position: absolute;
    top: 0; left: 0; z-index: 2;
    cursor: pointer;
}
```
   

## 实现涂抹效果（canvas基本操作）
首先取得canvas标签的节点，之前我们可以先判断浏览器是否支持，以及设置canvas的宽高。

```javascript
var cvs = document.getElementById('canvas');
```

接着取得canvas元素的2D上下文，可以理解为“控件句柄”，之后通过它绘制内容：

```javascript
var ctx = cvs.getContext('2d');
```

然后给canvas绘制一层半透明的“雾”：

```javascript
ctx.fillStyle = 'rgba(0,0,0,.5)';  // 决定之后填充的颜色，使用rgb或者16进制颜色均可，我们使用rgba以实现透明效果
ctx.fillRect(0, 0, cvs.width, cvs.height)  // 该方法绘制矩形并填充，参数依次是起始位置x坐标、y坐标，矩形的宽、高
```

先再了解一下绘制圆形的方法。触摸和点击返回的坐标只是一个点，我们要以这个点为圆心绘制一个圆，达到所需的效果。

canvas中除了矩形之外没有提供绘其他图形的方法，通常通过绘制路径+填充颜色来实现。canvas中绘制路径的方法包括：

- arc(x, y, radius, startAngle, endAngle, direction)：以（x, y）为圆心，radius为半径绘制一条弧线路径，startAngle、endAngle分别是起始角度和结束角度，最后是绘制方向，false表示顺时针方向；
- arcTo(x1, y1, x2, y2, radius)：从上一点开始绘制一条半径为radius的曲线路径，到(x2, y2)为之，图中穿过(x1, y1)；
- moveTo(x, y)：将绘图游标移动到(x, y)；
- lineTo(x, y)：从游标位置移动到(x, y)，中间绘制一条直线路径；
- rect(x, y, width, height)：与之前的 fillRect() 类似，这里只绘制一条路径；
- 其他还有 bezierCurveTo() 绘制贝塞尔曲线、quadraticCurveTo 绘制一条二次曲线；

绘制之前记得先调用 beginPath() 方法，路径绘制完成后，接着还可以：

- 调用 closePath() 方法，绘制当前点到起始点的路径，形成封闭图形；
- 调用 fill() 方法，用设置的 fillStyle 填充路径形成的封闭区域；
- 调用 stroke() 方法，用设置的 strokeStyle 对路径描边；
- 调用 clip() 方法，在路径上创建一个剪切区域；

OK，根据以上，我们封装一个绘制圆形的方法：

```javascript
ctx.fillCircle = function(x, y, radius, fillColor){
    this.fillStyle = fillColor;
    this.beginPath();
    this.arc(x, y, radius, 0, Math.PI*2);
    this.closePath();
    this.fill();
};
```

要注意的是，当触摸或点击某点时，我们是要把这一点变成透明，以露出底下的图片。默认情况下，新的形状会*叠加*在现有之上，所以“填充rgba颜色”是不可行的。

我们的解决方案是：设置 context 的 globalCompositeOperation 属性，大家自行感受一下下面两张图：

![gco-value.png](/public/img/20160103-1-01.png) 
![gco-value-demo.png](/public/img/20160103-1-02.png) 

各个属性值形成的效果在不同的浏览器里可能有一些差异，还好我们需要的 destination-out 基本表现一致。在初始的灰色透明绘制完后，设置该属性，之后的触摸就有的“抹去”的效果。

## 触摸和鼠标事件

在手机等设备上，手指的涂抹动作通过绑定 touch 事件来进行。为了在PC等设备上也能进行体验，相应的要替换成鼠标事件。所以，我们要先判断打开页面的设备，来绑定相应的时间。我们不用非常精确，下面的代码就够用了：

```javascript
var isMobile = (function(){
        var ua = navigator.userAgent;
        var reg = /iPod|iPad|iPhone|Android|Windows Phone|Symbian/i;
        return reg.test(ua);
    })();
```

所以，绑定事件应该是这样：

```javascript
cvs.addEventListener(isMobile ? 'touchmove' : 'mousemove', function(e){
    var x = isMobile ? e.changedTouches[0].pageX : e.pageX,
           y = isMobile ? e.changedTouches[0].pageY : e.pageY,
           r = 20;
    x -= bd.offsetLeft;
    y -= bd.offsetTop;
    ctx.fillCircle(x, y, 20, '#000');    // 前面封装的方法
}, false);
```

手指（鼠标）在移动过程中会持续触发touchmove事件，每次触发时在该点绘制一个透明圆，连续的过程就达到了涂抹的效果。其中，该效果是建立在canvas的基础上，IE8及以下是不支持的，因此也无需针对IE8及以下的事件绑定attachEvent()方法进行兼容。

另外，与鼠标事件不同的是，触摸事件的坐标等属性是位于Touch对象中。触摸事件的属性中除了常见的DOM属性，还有touches、targetTouches、changedTouches三个属性跟踪触摸情况，这三个都是Touch对象的数组。

我们需要取到触摸的坐标，以绘制透明圆。事件对象中包含多个坐标属性，应该取哪个值才能获得正确的位置？可以先参考[
关于事件对象中的几种坐标值](http://ks.netease.com/blog?id=945)。这里取 pageX/pageY - offsetLeft/offsetTop 的值。

## 优化
现在，基本的效果已经达成了。不过在实际体验中会发现，涂抹出来的阴影不是连续的，边缘不够自然，这是因为touchmove事件之间有一定的时间间隔。

![QQ拼音截.png](/public/img/20160103-1-03.png) 

采取的解决方法是touchmove过程中不绘制圆，而是与上一个触发点之间绘制一个矩形（touchstart时还是要先绘制一个圆）。通过前后两个点和r（touchstart时绘制的圆的半径），计算得到矩形四个顶点的坐标，绘制出路径后填充。

![aaa.png](/public/img/20160103-1-04.png) 

如此，涂抹效果就显得比较自然了。

![QQ拼音命名.png](/public/img/20160103-1-05.png) 

访问文首贴出的demo链接，查看源代码可以看到这100行代码。
