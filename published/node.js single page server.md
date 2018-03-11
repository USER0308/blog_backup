# 使用 node.js 快速搭建一个单页服务器

写好静态网页之后需要配置服务器才能被访问, apache,nginx 等对于单张网页进行处理的话显得有些笨重, 然后需要服务端脚本语言快速启动一个本地服务器, python 或者 nodejs 自然是最好的选择.

代码如下:

```
var http=require("http");
var fs=require("fs");
var server =http.createServer(function(req,res){
    console.log("server is working");
    res.setHeader("Content-Type","text/html;charset='utf-8'");
     fs.readFile("./index.html","utf-8",function(err,data){
         if(err) {
           console.log("index.html loading is failed :"+err);
         }
         else{
             res.end(data);
         }

     });
}).listen(8858);
```
