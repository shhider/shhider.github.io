---
layout: post
title: 沸点分享 - 编码规范与最佳实践
---

为什么要规范编码习惯：

- 利于团队开发，“用户体验”；
- 避免不必要的错误和异常；

## 通用

1、代码格式：

- 空格
- 换行
- 缩进
- 注释，两头空格
- 括号：大括号、小括号；
- 保存文件时删除行尾的空格；
- 文件结尾添加一个空白行；

```javascript
/* 注释 */
function func(){
	// TODO
}

function func()
{
	// TODO
}
```

2、变量。

- 先声明再使用。
- 统一命名风格：
	- 普通变量、函数用驼峰法（第一个字母小写，如varName）；
	- 类名也使用驼峰法（第一个字母大写，如ClassName）；
	- 常量所有字母大写；

3、清晰的项目目录结构；

4、随手查阅文档；

以上。可以将这些配置文件添加到项目的.editorconfig文件中。目前大多数编辑器、IDE已经支持。

## 前端部分

### HTML

1. 文档类型声明、编码声明、标题；
2. 标签、属性全部小写，属性值要用双引号；
3. 标签一定要闭合；
4. 结构、样式、行为的分离。避免在标签里的属性样式和事件触发；
5. 合理的结构、布局。不要使用table进行布局，同时避免过多的div。
6. Web语义化。利于开发，利于SEO；li、form、h1-h6、p、table、thead、tbody、th、label；
7. 特殊符号用HTML实体：
	- 空格：&amp;nbsp;
	- &：&amp;amp;
	- &lt;：&amp;lt;
	- &gt;：&amp;gt;
	- &copy;：&amp;copy;

```html
<!DOCTYPE html>
<html>
<head>
	<meta charset="utf-8">
	<title>标题</title>
	<link rel="stylesheet" type="text/css" href="/path/to/css">
</head>
<body>
	<div id="abc" class="xyz">
		<!-- 注释 -->
		<img src="/path/to/img" width="200" height="200" onclick="doSomething()">
	</div>
<!-- 空一些行 -->
<script>
(function(){
	/* 注释 */
	console.log('some debug info');
})();
</script>
</body>
</html>
```

### CSS

1. 选择器的名称、属性和值都使用小写；
2. 缩进风格：
	- 每个选择器一行写完；
	- 每个属性一行；
	- 折中。一类属性写一行（Position、BoxModel、Font、more）；
3. 值为0时省略单位；可缩写的值尽量缩写；颜色不要用命名色值；属性值为0-1的小数时，省略0；
4. 如果CSS可以做到，就不要使用JS。渐变、圆角、动画；
5. Reset。不要用 ``*{...}``；不要使用 !important 声明；
6. 尽量不用ID选择器；选择器的嵌套层级尽量不要超过3级；
7. 熟悉CSS盒子模型；
8. 浏览器兼容.开发前确认需要兼容到IEｘ；CSS3属性要根据规范添加浏览器前缀；
9. 开发前根据设计稿划分模块。g-abc；f-ijk；m-xyz；
10. css文件可以分为：
	- 基础样式：主要是样式重置、功能型样式；
	- 主体样式：针对当前项目的样式内容；
	- 其他，皮肤样式等； 

```css
@import url("normalize.css");
/* bad reset */
*{
	margin: 0; padding: 0;
}
.nav li{ float: left; margin-right: 5px; border: none; opacity: .7;}
.nav li:hover{
	border: 1px solid #cccccc;
	background: #ffffff;
	-moz-border-radius: 5px;
	-webkit-border-radius: 5px;
	border-radius: 5px;
}
```

### JavaScript

1. 分号；
2. 避免使用eval、document.write()等函数
3. 变量：
	- 用前声明，注意作用域，避免污染全局；
	- 不要用on开头；
4. 合适的循环；

```javascript
/* bad loop */
for(var i=0; i<arr.length; i++){
	// TODO
}
/* good loop */
var i, len = arr.length;
for(i=0; i<len; i++){
	// TODO
}
```

### 性能优化

1. 减少文件数量：CSS/JS文件合并、压缩；CSS Sprite；
2. 样式在上，脚本在下；
3. CSS、JS连续请求；
4. 图片格式：jpg、png8、png24、gif；避免没必要的图片；
5. CDN。浏览器对一个域下的连接数有限制；
6. 测试工具：Chrome DevTool；Firefox firebug；YSlow；IETester；其他各种测试网站；
7. 避免浏览器重绘；


## PHP

1. <?php ... ?>标签（纯php文件可以省略?>）
2. 单引号、双引号；
3. 统一编码：数据库、php文件、编码声明
4. 永远不要相信来自用户的数据；
5. PHP代码与HTML代码分离；

```php
<?php
header('Content-Type: text/html; charset=utf-8');
echo '$something';
echo "$something";
// ===
$arr[key];
$arr['key'];
```


## Reference

1. [How to lose weight](http://browserdiet.com/zh/)
2. [WEB开发基础 最佳实践手册](http://wf.uisdc.com/cn/performance/)
3. [NEC - Netease Easy CSS](http://nec.netease.com/)
4. [Caniuse](http://caiuse.com)
5. [Bootstrap：编写灵活、稳定、高质量的HTML和CSS代码的规范](http://codeguide.bootcss.com/)