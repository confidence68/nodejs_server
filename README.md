## 前言

我的博客是用nodejs搭建的，但是博客中的文章关于nodejs的，却不多，今天主要讲讲[nodejs][1]入门教程及案例应用！

## nodejs开启Http服务，运行hello word

nodejs开启一个服务是很简单的，我们只要安装了nodejs,随便找个目录，创建一个js文件，把下面代码复制进去，然后运行一下，就可以开启hello word 最简单的http服务了！

    const http = require('http')
    
    const hostname = '127.0.0.1'
    const port = 3000
    

<!--more-->


    const server = http.createServer((req, res) => {
      res.statusCode = 200
      res.setHeader('Content-Type', 'text/plain')
      res.end('Hello World\n')
    })
    
    server.listen(port, hostname, () => {
      console.log(`Server running at http://${hostname}:${port}/`)
    })

大家看到，上面代码是运用了ES6的写法，nodejs已经大部分支持ES6了！

## nodejs 重要模块的讲解及应用

    //http协议模块
    const http = require('http');
    //url解析模块
    const url = require('url');
    //文件系统模块
    const fs = require("fs");
    //路径解析模块
    const path = require("path");

关于以上基础模块的常用操作命令，大家可以查看nodejs官网API，假如你觉得英文比较难懂，可以看下下面几篇文章：

<a href="http://www.jianshu.com/p/5683c8a93511" target="_blank" rel="nofollow">nodejs之fs模块</a>

<a href="http://www.jianshu.com/p/fe41ee02efc8" target="_blank" rel="nofollow">nodejs之Path模块</a>

<a href="http://www.jianshu.com/p/fe41ee02efc8" target="_blank" rel="nofollow">node.js之Url & QueryString模块</a>

当然，其他的我就不写了！我们用上面几个模块来创建一个简单的nodejs服务。

### path 模块应用及301重定向

    //如果路径中没有扩展名
    if(path.extname(pathName) === ''){
        //如果不是以/结尾的，加/并作301重定向
        if (pathName.charAt(pathName.length-1) != "/"){
            pathName += "/";
            var redirect = "http://"+request.headers.host + pathName;
            response.writeHead(301, {
                location:redirect
            });
            response.end();
        }
        //添加默认的访问页面,但这个页面不一定存在,后面会处理
        pathName += "index.html";
        hasExt = false; //标记默认页面是程序自动添加的
    }

### fs读取文件列表及404页面

    fs.exists(filePath,function(exists){
        if(exists){
            response.writeHead(200, {"content-type":contentType});
            var stream = fs.createReadStream(filePath,{flags:"r",encoding:null});
            stream.on("error", function() {
                response.writeHead(500,{"content-type": "text/html"});
                response.end("<h1>500 Server Error</h1>");
            });
            //返回文件内容
            stream.pipe(response);
        }else { //文件名不存在的情况
            if(hasExt){
                //如果这个文件不是程序自动添加的，直接返回404
                response.writeHead(404, {"content-type": "text/html"});
                response.end("<h1>404 Not Found</h1>");
            }else {
                //如果文件是程序自动添加的且不存在，则表示用户希望访问的是该目录下的文件列表
                var html = "<head><meta charset='utf-8'></head>";
    
                try{
                    //用户访问目录
                    var filedir = filePath.substring(0,filePath.lastIndexOf('\\'));
                    //获取用户访问路径下的文件列表
                    var files = fs.readdirSync(filedir);
                    //将访问路径下的所以文件一一列举出来，并添加超链接，以便用户进一步访问
                    for(var i in files){
                        var filename = files[i];
                        html += "<div><a  href='"+filename+"'>"+filename+"</a></div>";
                    }
                }catch (e){
                    html += "<h1>您访问的目录不存在</h1>"
                }
                response.writeHead(200, {"content-type": "text/html"});
                response.end(html);
            }
        }
    });

## console.time和console.timeEnd

我们可以运用如下，输出执行时间！

    console.time('time');
    console.timeEnd("time");

两个配合使用。

## nodejs中exports与module.exports的区别

nodejs我们经常用exports来创建模块，但是exports.XXX和module.exports的具体区别是什么？如何应用的呢？

我们来看下exports.XXX的应用：

    exports.name = function() {
        console.log('My name is Haorooms');
    };

在另一个文件中你这样引用

    var rocker = require('./rocker.js');
    rocker.name(); // "My name is Haorooms"

再来看下如下案例

    module.exports = 'ROCK IT!';
    exports.name = function() {
        console.log('My name is Haorooms');
    };

 再次引用执行rocker.js

    var rocker = require('./rocker.js');
    rocker.name(); // TypeError: Object ROCK IT! has no method 'name'

发现报错：对象“ROCK IT!”没有name方法

rocker模块忽略了exports收集的name方法，返回了一个字符串“ROCK IT!”。由此可知，你的模块并不一定非得返回“实例化对象”。你的模块可以是任何合法的javascript对象--boolean, number, date, JSON, string, function, array等等。

你的模块可以是任何你设置给它的东西。如果你没有显式的给Module.exports设置任何属性和方法，那么你的模块就是exports设置给Module.exports的属性。也就是说Module.exports才是真正的接口，exports只不过是它的一个辅助工具。　最终返回给调用的是Module.exports而不是exports。

假如你的模块是一个构造函数，如下：

    module.exports = function(name, age) {
        this.name = name;
        this.age = age;
        this.about = function() {
            console.log(this.name +' is '+ this.age +' years old');
        };
    };

可以这样应用它：

    var Rocker = require('./rocker.js');
    var r = new Rocker('Ozzy', 62);
    r.about(); // Ozzy is 62 years old

假如你的模块是一个数组：

    module.exports = ['Lemmy Kilmister', 'Ozzy Osbourne', 'Ronnie James Dio', 'Steven Tyler', 'Mick Jagger'];

可以这样应用它：

    var rocker = require('./rocker.js');
    console.log('Rockin in heaven: ' + rocker[2]); //Rockin in heaven: Ronnie James Dio

现在你明白了，如果你想你的模块是一个特定的类型就用Module.exports。如果你想的模块是一个典型的“实例化对象”就用exports。


## github源代码案例

上面nodejs开启Http服务的案例我已经放到github了，大家可以看下！

    http://localhost:3000/  //默认读取index.html中的内容
    
    http://localhost:3000/werwer/  //显示目录不存在
    
    http://localhost:3000/images/  //显示文件列表

github地址：https://github.com/confidence68/nodejs_server

### 案例二：nodejs+socket创建简单的聊天室

引用socket.io和mime，创建简单的聊天室，很简单，大家可以看下！

github地址：https://github.com/confidence68/node-socket







  [1]: http://www.haorooms.com/tag/nodejs