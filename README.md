# server.js for Node.js
功能强大的Node.js服务器，可以让你专注于你的真棒项目

```
// Include it and extract some methods for convenience
const server = require('server');
const { get, post } = server.router;

// Launch server with options and a couple of routes
server({ port: 8080 }, [
  get('/', ctx => 'Hello world'),
  post('/', ctx => console.log(ctx.data))
]);
```
>
Simplicity is a great virtue but it requires hard work to achieve it and education to appreciate it. And to make matters worse: complexity sells better.
― Edsger W. Dijkstra
>

## 入门

初学者入门有一个[完整的教程](https://serverjs.io/tutorials/getting-started/)，但是快速的版本是首先安装服务器作为依赖：

```
npm install server
```

>
``server``需要Node.js 7.6.0或更高版本。 Node.js 8.9.x LTS推荐用于Node.js的长期支持。
>

然后你可以用下面的代码创建一个名为``index.js``的文件：

```
// Include the server in your file
const server = require('server');
const { get, post } = server.router;

// Handle requests to the url "/" ( http://localhost:3000/ )
server([
  get('/', ctx => 'Hello world!')
]);
```
在终端执行此操作以启动服务器：

```
node .
```

最后，在``[localhost：3000](http://localhost:3000/)``上打开你的浏览器，你应该看到'Hello world！'在您的浏览器上。

## 文档

serverjs 库文档:
[中文完整文档](https://github.com/winssps/server/blob/master/%E4%BB%8B%E7%BB%8D.md)
[英语完整的文档](https://serverjs.io/documentation/)

[订阅此处](http://eepurl.com/cGRggH)接收发布时的教程。一旦你了解了基础知识，这些教程对于学习是有好处的，而这些文档是很好的参考/快速使用。

您也可以下载资源库并通过浏览它们和节点来尝试实例。在他们每个``/examples``里面。

## 用例

包服务在很多情况下都很棒。我们来看看其中的一些：

### 小到中等的项目

一切正常，您可以获得对大多数功能的支持，您可以轻松使用Express的中间件生态系统。还有什么理由不喜欢？

一些导入的功能：body和file parsers，cookies，session，websockets，Redis，gzip，favicon，csrf，SSL等。他们这样工作，所以你会节省一个或者两个烦恼，并且专注于您的实际项目。获取一个简单的形式：

```
const server = require('server');
const { get, post } = server.router;
const { render, redirect } = server.reply;

server(
  get('/', () => render('index.pug')),
  post('/', ctx => {
    console.log(ctx.data);
    return redirect('/');
  })
);
```

## API设计

从包的灵活性和表现性来看，设计API是一件轻而易举的事情：

```
// books/router.js
const { get, post, put, del } = require('server/router');
const ctrl = require('./controller');

module.exports = [
  get('/book', ctrl.list),
  get('/book/:id', ctrl.item),
  post('/book', ctrl.create),
  put('/book/:id', ctrl.update),
  del('/book/:id', ctrl.delete)
];
```

##  Real time

Websockets从来没有这么容易使用！使用前端的socket.io，你可以简单地在后端执行这些事件来处理这些事件：

```
// chat/router.js
const { socket } = require('server/router');
const ctrl = require('./controller');

module.exports = [
  socket('connect', ctrl.join),
  socket('message', ctrl.message),
  socket('disconnect', ctrl.leave)
];
```



