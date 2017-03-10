---
layout: post
category: code
title: webpack2的ES6 Modules支持
---

Features:

- ES6 Modules
- tree shaking

不过webpack2直接打包es6代码的话，并不会编译成es5，所以web应用依然需要用babel进行编译。为了不浪费webpack的module特性，可以配置babel的``modules: false``.

> Webpack 1 wasn’t able to parse ES2015 modules, so Babel would convert them into CommonJS. Now Webpack 2 can parse ES2015 modules, and is able to eliminate dead code based on which modules are used. With ``[ "es2015", { "modules": false } ]`` you tell Babel to stop compiling ES2015 modules.

```javascript
{
  "presets": [
    "react",
    [ "es2015", { "modules": false } ]
  ]
}
```

```
Module build failed: ReferenceError: [BABEL] /Users/shihaohong/Workspaces/test/es6-module/app.js: Using removed Babel 5 option: foreign.modules - Use the corresponding module transform plugin in the `plugins` option. Check out http://babeljs.io/docs/plugins/#modules
```


## Reference

- [如何评价 Webpack 2 新引入的 Tree-shaking 代码优化技术？ - 知乎](https://www.zhihu.com/question/41922432/answer/93346223)
