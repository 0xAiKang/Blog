---
title: Socket.io 快速上手
date: 2020-07-20 13:02:05
tags: ["Socket.io"]
categories: ["Node", "Socket.io"]
---

最近使用`socket.io` 和 `redis` 完成了一些小功能，觉得很实用，所以整理一下`socket.io`相关的知识。

<!-- more -->

# `socket.io` 是什么
它是一个服务端与客户端之间建立通讯的工具。

服务端创建好服务之后，客户端通过主机与之建立连接。然后就可以进行通讯了。

想要使用好`socket.io`，一定要理解通讯的概念。通讯一定是双向的，如果客户端能够收到消息，那么在某个地方就一定存在服务端向客户端推送消息。

## 快速上手
要开始使用`socket.io`进行开发，需要先安装Node和npm。

创建一个名为`app.js`的文件，并添加以下代码。
```
var app = require('express')();
var http = require('http').Server(app);
// 创建一个附加到http服务器的新socket.io实例
var io = require('socket.io')(http);

app.get('/', function(req, res){
  res.sendFile(__dirname + '/index.html');
});

io.on('connection', function(socket){
  console.log('a user connected');
});

http.listen(3000, function(){
  console.log('listening on *:3000');
});
```
这样就完成了一个最简单的`socket`服务端。

创建`index.html` 文件来作为客户端提供服务。
```
<!DOCTYPE html>
<html>
   <head>
      <title>Hello world</title>
   </head>
   <body>Hello world</body>
</html>
```

启动服务
```
node app.js
```
创建的服务运行在本地的 `3000` 端口上，打开浏览器，输入`http://localhost:3000`进行访问。 

## 使用事件

`socket.io` 的核心理念就是允许发送、接收任意事件和任意数据。任意能被编码为 JSON 的对象都可以用于传输。二进制数据 也是支持的。

在上面的代码中，我们已经创建了一个服务端的`socket.io`对象，如果想要能正常通讯，还需要在客户端同样也创建一个`socket.io`对象。这个脚本由服务端的`/socket.io/socket.io.js` 提供。

```
<!DOCTYPE html>
<html>
   <head>
      <title>Hello world</title>
   </head>
   <script src = "/socket.io/socket.io.js"></script>
   
   <script>
      var socket = io();
   </script>
   <body>Hello world</body>
</html>
```

在客户端中建立 `socket.io` 连接。

在服务端中添加以下代码：
```
  ...

// 只有有客户端连接，就会触发这个事件
io.on('connection', function(socket) {
   console.log('A user connected');

   // 只有有客户端断开连接，就会触发这个事件
   socket.on('disconnect', function () {
      console.log('A user disconnected');
   });
});

  ...
```
现在再次访问`http://localhost:3000`，不仅可以在浏览器中看见`hello world`，如果刷新浏览器，还能在控制台中看见以下内容：

```
A user connected
A user disconnected
A user connected
```

在上面的案例中，我们使用了`socket.io`的`connection`和`disconnect`事件，`socket.io`还有很多其中事件。 

### 事件处理
在服务端中有以下是保留字：
* Connect
* Message
* Disconnect
* Reconnect
* Ping
* Join and
* Leave

在客户端中以下是保留字：
* Connect
* Connect_error
* Connect_timeout
* Reconnect, etc

### 常用API

客户端 提供的一些用于处理错误/异常的API。
```
Connect − When the client successfully connects.

Connecting − When the client is in the process of connecting.

Disconnect − When the client is disconnected.

Connect_failed − When the connection to the server fails.

Error − An error event is sent from the server.

Message − When the server sends a message using the send function.

Reconnect − When reconnection to the server is successful.

Reconnecting − When the client is in the process of connecting.

Reconnect_failed − When the reconnection attempt fails.
```

### 广播
广播意味着向所有连接的客户端发送消息。

要向所有客户端广播事件，我们可以使用`io.sockets.emit`方法。
```
  ...

var clients = 0;
io.on('connection', function(socket) {
   clients++;
   io.sockets.emit('broadcast',{ description: clients + ' clients connected!'});
   socket.on('disconnect', function () {
      clients--;
      io.sockets.emit('broadcast',{ description: clients + ' clients connected!'});
   });
});
  
  ...
```

广播在`socket.io`中应用的非常多，有广播就意味着有接收。需要在客户端中处理广播事件：
```
<!DOCTYPE html>
<html>
   <head>
      <title>Hello world</title>
   </head>
   <script src = "/socket.io/socket.io.js"></script>
   <script>
      var socket = io();
      socket.on('broadcast',function(data) {
         document.body.innerHTML = '';
         document.write(data.description);
      });
   </script>
   <body>Hello world</body>
</html>
```

可以尝试打开多个浏览器，输入`http://localhost:3000`，可能会得到以下结果：

![](https://www.tutorialspoint.com/socket.io/images/broadcast_to_all.jpg)

### 参考链接
* [Socket.io Tutorial](https://www.tutorialspoint.com/socket.io/index.htm)