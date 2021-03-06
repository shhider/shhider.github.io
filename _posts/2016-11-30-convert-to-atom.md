---
layout: post
category: code
title: Atom编辑器折腾记
---


【置顶】
Recommend Packages：
- emmet-atom, 前端必备；
- highlight-selected, 选中某词时，将当前文件中其它位置相同的词高亮；
- minimap, 用过 Sublime 的都懂；
- docblockr, 注释利器，快速编辑各种格式的注释；
- activate-power-mode, 装逼必备；
- atom-editorconfig, 识别``.editorconfig``文件，并加以应用；
- atom-project-viewer, Atom 自身的 project 机制务必蛋疼，使用它方便地管理 Project；
- auto-detect-indentation, 目前没有发现打开文件自动识别缩进的包，暂时用它每次手动切换缩进形式；
- file-icon, 使 Tree View 的图标更好看、更直观；

昨天晚上手贱，把sublime编辑器的一些插件删掉了、又改了些配置，结果今天各种高亮不对、各种缩进不对，弄了好久也没有恢复到之前的手感。一怒之下，干脆下载了Atom，看看能不能用Atom来替换Sublime。

首先，Atom的下载包约90+MB，相对来讲不小，原因我想是Chromium包的关系。下载之后，点击运行，不能选择安装的位置，等待片刻之后，直接就已经启动了。

第一眼呢，UI非常漂亮，暗色系的配色很符合我口味，我也就懒得去折腾其他的主题了。

接下来就是打开设置页，将可配置的项设置成我习惯的样子，比如缩进风格、行号显示等。可视化的设置页，这一点相比于sublime，还是更人性化的；而且除此之外，同样可以使用配置文件的方式进行配置。两者之间是同步的。

基本可以用了之后，我就准备开始写代码了，也就开始了折腾之路。

## 安装Packages

作为前端，首先就是要安装HTML必备——Emmet。但是在设置页的install页面进行搜索，加载非常的慢，我想这应该是得益于伟大的GFW吧。

还好，经过查找，还有一种更方便、更geek的方法，就是直接从Github上clone下对应的代码库，保存到``[用户目录]/.atom/packages/``中，就“安装”成功了~不过有些package需要安装一些依赖包，那么打开终端、进入对应的package目录，执行``npm install``，执行完成即可。

你有没有发现这就是一个个的``node module``~没错，前端大法好~~

通过这种方式，我目前安装了下面的这些package，大家可以参考~

- Emmet，前端必备，提高效率；
- Minimap，从Sublime转过来的都懂，很方便；
- Project Viewer。Atom的project机制用起来比较蛋疼，添加的project目录可能在下次打开就没有了；有时候单纯打开一个文件，又会将其所在目录加进来……添加了project Viewer之后，它会在窗口右侧添加一个列表，你可以导入project，只要点击一个project，Atom的文档目录树就打开对应的目录，非常方便；

另外……Activate Power Mode，装逼必备……

## 设置指定文件类型的语法高亮

场景是这样的，我们项目中，部分HTML文件后缀是``.ftl``，以及使用了**MCSS**预编译器、后缀为``.mcss``。那要设置指定文件后缀的语法高亮，在Sublime里就很容易了，而在Atom……打开文件后，可以通过``shift+ctrl+l``打开语法选择，找到对应的语法后就能高亮显示，但再次打开同类型文件，依然是一片灰白……

经过了一大~~圈的搜索，最后使用的办法是——自己写一个语法Package。以``.ftl``文件为例，在``[用户目录]/.atom/packages/``中新建目录``language-ftl``（目录名称自定义即可），在该目录中新建目录``grammars``，新建文件``ftl.cson``：

```text
/* 名称，不要重复 */
'scopeName': 'source.ftl'

/* 后缀，可以再添加其他的后缀类型 */
'fileTypes': [
    'ftl'
]
/* 语法的名称，会显示在语法选择器里 */
'name': 'FTL'

/* 规则，这里从Atom自带的html语法继承 */
'patterns': [
    {
        'include': 'text.html.basic'
    }
]
```

接着保存、reload之后，再打开``.ftl``文件就可以显示html的高亮了~

要再提的是，继承的html语法、其名称找了我半天，从网上看到的，通常说是``source.html``、``source.css``等，然后我填写之后就是不生效哇~后来我灵光一闪，直接从Atom的Github仓库找到了HTML语法的源文件，从而得知其名称为``text.html.basic``。其他Atom自带的语法，你也可以从其仓库找到；参考这些仓库，你也可以为自己的语言语法，做一个语法高亮库。

## Reload Config

在修改了``config.cson``文件、或者自定义的package修改后，Atom不会自动载入新的配置，此时需要``shift+ctrl+p``，查找``reload``命令并执行，Atom就回重新载入。相对重启会快速一些，但也是要等待一小段空白时间。
