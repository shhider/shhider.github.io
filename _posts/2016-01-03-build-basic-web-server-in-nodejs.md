---
layout: post
title: Node.js学习：搭建基本功能的Web服务器
category: code
---

> 这是2014年底在公司实习时在内部社区发的几篇文章之一。

虽然接触过两个Node.js的项目，但都是直接上express框架，至今还没有好好学习过基础的内容。由于之后会参与到一个node.js相关的项目中，所以现在有必要学习一下基础知识，避免之后碰到太多的坑。

现在node.js主要的应用还是Web，所以尝试写一个有基本功能的Server，我想也能学习到它的一些知识。

功能：

- 简化server的创建；
- 基本的路由；
- 简化GET/POST数据的获取；

后面就直接上代码了。其实主要要了解的是有哪些模块、有想些对象、有哪些属性，以及执行的原理和过程。另外发现还会回顾到一些网络方面的内容，比如HTTP协议的报文结构什么的。

之后会继续尝试把文件上传等其他基本功能也实现一下。

```javascript
/* server.js */
var http = require('http'),
    url = require('url'),
    qs = require('querystring');

/**
 @param port 监听端口
 @param route 路由对象
 @param callback 创建成功后执行的回调函数
 */
var start = function(port, route, callback){
    if('number' != typeof port){
        // 必须指定监听端口
        console.warn('Please specify the port.');
        return;
    }

    http.createServer(function(request, response){
        var objUrl = url.parse(request.url),    // 分析request的URL
            pathname = objUrl.pathname;         // 取到请求路径

        // 如果没有指定某请求的处理方法，返回404
        if(!(pathname in route)){
            response.writeHead(404, {'Content-Type': 'text/plain'});
            response.end();
            return;
        }

        var postData = '';
        var _Request = {
                _GET: {},   // 对象形式存放GET数据
                _POST: {}   // 对象形式存放POST数据
            };
        _Request._GET = qs.parse(objUrl.query);     // GET的数据('?'后面的键值对)保存在了query属性

        if('POST' == request.method){
            // 处理post请求。
            // 一般来说，post传输的数据量较大，因此会进行分包传输。
            // 分包传输可能是一个相对长期的过程，因此需要异步的形式。
            // 每到达一个包就触发获取。
            request.setEncoding("utf8");
            request.addListener('data', function(postDataChunk){
                postData += postDataChunk;
                // Too much POST data, kill the connection!
                if (postData.length > 1e6)
                    request.connection.destroy();
            });
            request.addListener('end', function(){
                _Request._POST = qs.parse(postData);
                console.log('Received postData finish: ' + postData);
                // 在之前的尝试中，我把下面调用对应方法的语句放在了判断之后，
                // 没有考虑*异步*的特性，
                // 造成POST的数据始终是空的。
                (route[pathname])(response, _Request);
                response.end();
            });
        }else{  // GET
            // 不得不再写一次
            (route[pathname])(response, _Request);
            response.end();
        }

    }).listen(port);

    callback && callback();
}

exports.start = start;

```

启动就非常简单了

```javascript
/* app.js */
var server = require('./server'),
    requestHandler = require('./requestHandler');

var route = {
    '/': requestHandler.index,
    '/index': requestHandler.index,
    '/home': requestHandler.home
};

server.start(80, route, function(){
    console.log('The server already start...');
});
```

对应的操作

```javascript
exports.index = function(response, _Request){
    console.log('\nrequest /index');

    var retStr = '';
    retStr += '\
<!DOCTYPE html>\
// 省略了...
</form>';

    var getData = _Request._GET;
    for(var i in getData){
        retStr += '<p>' + i + ': ' + getData[i] + '</p>';
    }

    retStr += '</body></html>';

    response.writeHead(200, {'Content-Type': 'text/html'});
    response.write(retStr);
};

exports.home = function(response, _Request){
    console.log('\nrequest /home');

    var retStr = '';
    retStr += '\
<!DOCTYPE html>\
// 省略了...
<body>';

    var postData = _Request._POST;
    for(var i in postData){
        retStr += '<p>' + i + ': ' + postData[i] + '</p>';
    }

    retStr += '</body></html>';
    response.writeHead(200, {'Content-Type': 'text/html'});
    response.write(retStr);
};
```

over

发现一工具挺好的：supervisor。在开发时使用它启动应用(supervisor app.js)，可以监视文件的修改，自动重启，避免了每次 Ctrl+C 去重启应用。
