# Server.js 中文文档
## 文档



概念上，服务器是一个接受选项和其他功能的函数。繁重的工作已经实现，所以你可以专注于你的项目：




```
// Import the variable into the file``
	const server = require('server');
	
// All of the arguments are optional
	server(options, fn1, fn2, fn3, ...);
```

您也可以按照 [教程](https://serverjs.io/tutorials) 学习Node.js开发。


### 入门开始
有一个[入门教程初学者](https://serverjs.io/tutorials/getting-started/)。如果你知道你的方式：

``npm install server``	

然后在你的index.js中创建一些演示代码：

```
// Import the libraryconst  
server = require('server');
// Answers to any request
server(ctx => 'Hello world');
```

从终端运行它：

``node . ``

在[localhost：3000](http://localhost:3000/)上打开你的浏览器，看看它在运行。

### 基本用法

一些组件是本身，[router](https://serverjs.io/documentation/router/)和[reply](https://serverjs.io/documentation/reply/)的主要功能。主函数首先接受[options](https://serverjs.io/documentation/options/)的可选对象，然后接受任意数量的[中间件](https://serverjs.io/documentation/#middleware)或中间件数组


```
const server = require('server');

server({ port: 3000 }, ctx => 'Hello 世界');
```

要使用路由器并回复，根据需要提取他们的方法：

```
const server = require('server');
const { get, post } = server.router;
const { render, json } = server.reply;

server([
  get('/', ctx => render('index.hbs')),
  post('/', ctx => json(ctx.data)),
  get(ctx => status(404))
]);
```
然后，当您将文件拆分成不同的部分，并且无法访问全局服务器时，只能导入相应的部分：

```
const { get, post } = require('server/router');
const { render, json } = require('server/reply');
```


### 中间件
中间件是每个请求都会调用的普通函数。它接收[一个上下文对象](https://serverjs.io/documentation/context)并[返回一个回复](https://serverjs.io/documentation/reply/)，一个[基本类型](https://serverjs.io/documentation/reply/#return-value)或者什么也没有。几个例子：

```
const setname = ctx => { ctx.user = 'Francisco'; };
const sendname = ctx => send(ctx.user);
server(setname, sendname);
```

它们可以作为``server（）``参数放置，组合成一个数组或从其他文件导入/导出：


```
server(
  ctx => send(ctx.user),
  [ ctx => console.log(ctx.data) ],
  require('./comments/router.js')
);
```
在./comments/router.js中：

```
const { get, post, put, del } = require('server/router');
const { json } = require('server/reply');

module.exports = [
  get('/',    ctx => { /* ... */ }),
  post('/',   ctx => { /* ... */ }),
  put('/:id', ctx => { /* ... */ }),
  del('/:id', ctx => { /* ... */ }),
];
```

同步和异步函数的主要区别在于你使用async关键字，然后能够使用关键字在函数内等待，避免[回调地狱](http://callbackhell.com/)。中间件的一些例子：

```
// Some simple logging
const mid = () => {
  console.log('Hello 世界');
};

// Asynchronous, find user with Mongoose (MongoDB)
const mid = async ctx => {
  ctx.user = await User.find({ name: 'Francisco' }).exec();
  console.log(ctx.user);
};

// Make sure that there is a user
const mid = ctx => {
  if (!ctx.user) {
    throw new Error('No user detected!');
  }
};

// Send some info to the browser
const mid = ctx => {
  return `Some info for ${ctx.user.name}`;
};
```

这样你可以``await``你的函数内部。在继续下一步之前，Server.js也将等待到您的中间件：

```
server(async ctx => {
  await someAsyncOperation();
  console.log('I am first');
}, ctx => {
  console.log('I am second');
});
```

如果在异步函数中发现错误，则可以抛出它。它将被捕获，将向用户显示一个500错误，并记录错误：


```
const middle = async ctx => {
  if (!ctx.user) {
    throw new Error('No user :(');
  }
};
```

``
避免基于回调的函数：错误传播是有问题的，他们必须转换为承诺。强烈倾向于异步/等待工作流程。
``

### Express中间件

Server.js使用express作为底层库(we <3 express!) 您可以导入``modern``设计的中间件：

```
const server = require('server');

// Require it and initialize it with some options
const legacy = require('helmet')({ ... });

// Convert it to server.js middleware
const mid = server.utils.modern(legacy);

// Add it as you'd add a normal middleware
server(mid, ...);
```
>
注意：{...}表示中间件的选项，因为许多 [express 库](https://expressjs.com/en/guide/writing-middleware.html) 都遵循[工厂模式](https://github.com/expressjs/express/issues/3150)。
>

为了简化它，我们也可以内联执行这个操作：

```
const server = require('server');
const { modern } = server.utils;

server(
  modern(require('express-mid-1')({ ... })),
  modern(require('express-mid-2')({ ... })),
  // ...
);const server = require('server');
const { modern } = server.utils;

server(
  modern(require('express-mid-1')({ ... })),
  modern(require('express-mid-2')({ ... })),
  // ...
);
```

或者将整个中间件保存在一个单独的文件/文件夹中：

```
// index.js
const server = require('server');
const middleware = require('./middleware');
const routes = require('./routes');

server(middleware, routes);
```

然后在我们的``middleware.js``中：

```
// middleware.js
const server = require('server');
const { modern } = server.utils;

module.exports = [
  modern(require('express-mid-1')({ /* ... */ })),
  modern(require('express-mid-2')({ /* ... */ }))
];
```

### 路由

这是将每个请求重定向到我们的服务器到正确的地方的概念。
例如，如果用户请求我们的主页``/``我们想要呈现主页
但如果他们要求一个图片库``/gallery/67546``我们要呈现``gallery 67846``

为此，我们将使用服务器的路由器创建路由。我们可以像这样导入它：

```
const server = require('server');
const { get, post } = server.router;

// OR

const { get, post } = require('server/router');
```

还有其他一些方法，但这些是推荐的方法。然后我们说我们要听的方法和一个中间件的请求路径：

```
const getHome = get('/', () => render('index.pug'));
const getGallery = get('/gallery/:id', async ctx => {
  const images = await db.find({ id: ctx.params.id }).exec();
  return render('gallery.pug', { images });
});
```

让我们把它们放在一起，看看它们是如何工作的：

```
const server = require('server');
const { get, post } = server.router;

const getHome = get('/', () => render('index.pug'));
const getGallery = get('/gallery/:id', async ctx => {
  const images = await db.find({ id: ctx.params.id }).exec();
  return render('gallery.pug', { images });
});

server(getHome, getGallery);
```

我们也可以通过路由器接收``post，del，error，socket``等请求类型。要查看全部内容，请访问Router文档：

[Router文档](https://serverjs.io/documentation/router)

### 高级主题

直到我们到达这里，还有很多基本的到难度的文档。只是一个快速的说明，

主函数返回一个承诺，当服务器正在运行并且可以被访问时，这个承诺就会被执行。它会收到一个更原始的上下文。所以这是完全有效的：

```
server(ctx => 'Hello world').then(ctx => {
  console.log(`Server launched on http://localhost:${ctx.options.port}/`);
});
```
### 继续阅读

所有主题的列表：

[Options](https://serverjs.io/documentation/options) 
[Context](https://serverjs.io/documentation/context)
[Router](https://serverjs.io/documentation/router) 
[Reply](https://serverjs.io/documentation/reply)
[Error](https://serverjs.io/documentation/errors)
[Testing](https://serverjs.io/documentation/testing)
