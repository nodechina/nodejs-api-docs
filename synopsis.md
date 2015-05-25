# 概述

第一个 [服务器](http.html) 的例子就从 “Hello World” 开始：  

    var http = require('http');

    http.createServer(function (request, response) {
      response.writeHead(200, {'Content-Type': 'text/plain'});
      response.end('Hello World\n');
    }).listen(8124);

    console.log('Server running at http://127.0.0.1:8124/');

把代码拷贝到`example.js`文件里，用 node 程序执行

    > node example.js
    Server running at http://127.0.0.1:8124/
  
文档中所有的例子都可以这么执行。   



