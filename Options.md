# 选项 #
.env中的可用选项，默认值，类型和名称：

| name        | default     |  .env  |   type   |
| ------------- |:------------:|-------:|---------:|
| port        | 3000  |  PORT=3000 | Number |
| secret| 'secret-XXXX'|   SECRET=secret-XXXX |  String |
| public | 'public'      |    PUBLIC=public | String     |
|views   |'views' |VIEWS=views|String|
|engiine|  'pug'|ENFINE=pug|String|
|env     |  'development'|NODE_ENV=development|String|
|favicon  |  false|FACICON=public/logo.png|String|
|log      |  'info'|LOG=info|String|
|session  |  [info]|[info]|Object|
|security  |   [info]|[info]|Object|


你可以通过``server（）``函数中的第一个参数来设置它们：

```
// Import the main library
const server = require('server');

// Launch the server with the options
server({
  port: 3000,
  public: 'public',
});
```

选项偏好顺序是这样的，从更重要到更少：

1. ``.env``:环境中的变量。
2. ``server({OPTION: 3000})``:启动服务器时将该变量设置为参数。
3. 默认值：将使用默认值，如下所示

它们可以通过``ctx.options``访问您的开发需求（在[上下文选项中阅读更多](https://serverjs.io/documentation/context/#options)）：

```
server(ctx => console.log(ctx.options));
// { port: 3000, public: './public', ... }
```

### Environment

环境变量不在您的版本控制中提交，而是由机器或Node.js进程提供。通过这种方式，这些选项在您的机器和测试，生产或其他类型的服务器上可能会有所不同。

它们是大写字母，可以通过根文件夹中的字面上的.env文件来设置：

```
PORT=3000
PUBLIC=public
SECRET=secret-XXXX
ENGINE=pug
NODE_ENV=development
```

>
Remember to add ``.env`` to your ``.gitignore``.
>

要在远程服务器中设置它们，将取决于您使用的主机（请参阅[Heroku示例](https://devcenter.heroku.com/articles/config-vars)）。

### 实参 Argument

在调用``server（）``时，环境变量的替代方法是将它们作为第一个参数传递。每个选项都是对象中键/值的组合，并且都是小写字母。用默认值查看一些选项：


```
const server = require('server');

server({
  port: 3000,
  public: 'public',
  secret: 'secret-XXXX',
  engine: 'pug',
  env: 'development'   // Remember this is "env" and not "node_env" here
});
```

### 特殊情况

作为一般规则，作为对象的选项将成为``.env``文件的大写``_``分隔字符串。例如，对于SSL，我们必须传递一个对象，如：

```
server({
  port: 3000,
  ssl: {
    key: './ssl.pem',
    cert: './ssl.cert'
  }
});
```

所以如果我们想把它放在环境变量中，我们将它设置为：

```
PORT=3000
SSL_KEY=test/fixtures/keys/agent2-key.pem
SSL_CERT=test/fixtures/keys/agent2-cert.cert
```

相反是不正确的; ``.env``中的一个``_``分隔的字符串不一定会成为一个对象作为参数。你将不得不阅读每个选项和插件的具体细节的文档。


### Port

要启动服务器的端口。默认为3000，这是唯一的选项，可以指定为一个单一的选项：

```
server();        // 使用默认端口 3000
server(3000);    // 指定端口
server({ port: 3000 });  // 和前面一样指定端口
```
一些主机如Heroku将定义一个名为PORT的环境变量，所以它在那里将会顺利运行。你也可以在你的``.env``中设置它，如果你喜欢的话：

```
PORT=3000
```

例如：将端口设置为其他数值。对于1-1024数值，您需要管理员权限，所以我们使用更高的端口进行测试：

```
const options = {
  port: 5001
};

/* test */
const same = ctx => ({ port: ctx.options.port });
const res = await run(options, same).get('/');
expect(res.body.port).toBe(5001);
```

### Secret

[强烈建议](https://serverjs.io/tutorials/sessions-production/)您在开始编码之前将其设置在开发和生产的环境变量中。
它应该是一个随机的，长的字符串。它可以被中间件用于存储秘密和保持``cookie /session``：

```
SECRET=your-random-string-here
```

每次启动服务器时提供的默认值都会有所不同。

这不适合于生产，因为即使服务器重新启动也需要持久会话。查看[生产教程中的session](https://serverjs.io/tutorials/sessions-production/)以正确设置它（包括一些额外的例如Redis session）。

请不要将其设置为变量 ``server({ secret: 'whatever' });``

### Public

静态资产所在的文件夹，默认为`` public``。这包括图像，样式，浏览器的JavaScript等。您想要通过浏览器直接访问的任何文件（如``example.com/myfile.pdf``）都应位于此文件夹中。

要在环境中设置公用文件夹，请将其添加到``.env``中：

```
PUBLIC=public
```

通过初始化参数:

```
server({ public: 'public' });
```

要设置根文件夹，请将其指定为`` `./` ``：

```
server({ public: './' });
```

如果您不希望公开访问任何文件，则可以通过虚假值或空值取消它：

```
server({ public: false });
server({ public: '' });	
```

### Views

视图和模板所在的文件夹，默认为视图。这些是``render（）``[方法](https://serverjs.io/documentation/reply/#render-)使用的文件。您可以将其设置为您的项目中的任何文件夹。

要在环境中设置视图文件夹，请将其添加到``.env``中：

```
VIEWS=views
```

通过初始化参数：

```
server({ views: 'views' });
```

要设置根文件夹，将其指定为`` `./` ``：

```
server({ views: './' });
```

如果您没有任何视图文件，则不必创建文件夹。视图内的文件应该都有一个扩展名，例如``.hbs``，``.pug``等。要了解如何安装和使用这些继续阅读。


### Engine

>
注意：这个选项可以忽略，``server.js``可以同时使用``.pug``和``.hbs``（Handlebars）文件类型。
>

您想用来呈现您的模板的视图引擎。查看[所有可用的引擎](https://github.com/expressjs/express/wiki#template-engines)。要使用引擎，通常必须首先安装，除了预先安装的[pug](https://pugjs.org/)和[Handlebars](	http://handlebarsjs.com/)：

```
npm install [ejs|nunjucks|emblem] --save
```
然后使用这个引擎，你只需要将扩展​​添加到``render（）``[方法](https://serverjs.io/documentation/reply/#render-)中：

```
// No need to specify the engine if you are using the extension
server(ctx => render('index.pug'));
server(ctx => render('index.hbs'));
// ...
```

但是，如果你想使用它，而没有扩展名，你可以通过在``.env``中指定引擎来实现：

```
ENGINE=pug
```
或者通过javascript中的相应选项：

```
server({ engine: 'pug' }, ctx => render('index'));
```

### Env

定义服务器运行的上下文。最常见和被接受的案例是``'development'``，``'测试'``和``'production'``。某些功能可能因环境而异，例如实时/热重新加载，缓存等。

>
注意：环境变量被称为``NODE_ENV``，而作为参数的选项是``env``。
>

这个变量作为main函数的参数是没有意义的，所以我们通常在我们的``.env``文件中使用它。在这里看到它与默认的``development``：

```
NODE_ENV=development
```

然后在您的主机环境中，您将其设置为生产（像Heroku这样的主机自动执行）：

```
NODE_ENV=production
```

这些是NODE_ENV最常见和推荐的类型：

```
development
test
production
```

你可以检查你的代码中的那些：

```
server(ctx => {
  console.log(ctx.options.env);
});
```

### Favicon
要包含一个收藏夹图标，请使用收藏夹``favicon``指定其路径：

```
const server = require('server');

server({ favicon: 'public/favicon.png' },
  ctx => 'Hello world'
);
```

路径可以是绝对的或相对于您的项目的根。

### Log

显示一些可能对开发者有价值的数据。这包括从一些信息到真正重要的错误和错误通知。

您可以设置[多个日志级别](https://www.npmjs.com/package/log#log-levels)，默认为'info'：

- ``emergency`` : 系统不可用
- ``alert``: 必须立即采取行动
- ``critical``:该系统处于危险状态
- ``error``:错误状态
- ``warning``:警告状态
- ``notice``:一个正常但重要的情况
- ``info``:一个纯粹的信息消息
- ``debug``:消息来调试应用程序

在你的.env里做：

```
LOG=info
```
或者作为主要功能的参数：

```
server(ctx => {
  ctx.log.info('Simple info message');
  ctx.log.error('Shown on the console');
});
```

如果我们想修改关卡并只显示警告或更重要的日志：

```
server({ log: 'warning' }, ctx => {
  ctx.log.info('Not shown anymore');
  ctx.log.error('Shown on the console');
});
```

#### 高级日志记录

你也可以传递一个``report``变量，在这种情况下，该级别应该被指定为``level``：


```
server({
  log: {
    level: 'info',
    report: (content, type) => {
      console.log(content);
    }
  }
});
```

这允许你以不同的方式处理一些特定的错误。

### Session

它接受这些选项作为一个对象：

```
server({ session: {
  resave: false,
  saveUninitialized: true,
  cookie: {},
  secret: 'INHERITED',
  store: undefined,
  redis: undefined
}});
```
您可以[在Express包装文档中阅读更多关于这些选项的信息](https://github.com/expressjs/session)。

所有这些都是可选的。如果没有明确设定，实参就会从全局实参中继承实参。


如果使用```Redis URL```设置了redis或REDIS_URL，则将启动Redis存储以实现会话中的持久性。在[Sessions in production](http://serverjs.io/tutorials/sessions-production/)中阅读更多关于这个的内容。

### Security

它结合了[Csurf](https://github.com/expressjs/csurf)和[Helmet](https://github.com/helmetjs/helmet)提供额外的安全性：

```
server({
  security: {
    csrf: {
      ignoreMethods: ['GET', 'HEAD', 'OPTIONS'],
      value: req => req.body.csnowflakerf
    }
  }
});
```

我们正在使用[Helmet](https://helmetjs.github.io/)来实现很好的安全性。要传递任何[Helmet Option](https://github.com/helmetjs/helmet)，只需将其作为安全性的另一个选项传递即可：

```
server({
  security: {
    frameguard: {
      action: 'deny'
    }
  }
});
```

对于快速测试/原型，整个安全插件可以被禁用（不推荐）：

```
server({ security: false });
```

个别部分也可以像这样被禁用：

```
server({
  security: {
    csrf: false
  }
});
```

``.env中``的名字是：

```
SECURITY_CSRF
SECURITY_CONTENTSECURITYPOLICY
SECURITY_EXPECTCT
SECURITY_DNSPREFETCHCONTROL
SECURITY_FRAMEGUARD
SECURITY_HIDEPOWEREDBY
SECURITY_HPKP
SECURITY_HSTS
SECURITY_IENOOPEN
SECURITY_NOCACHE
SECURITY_NOSNIFF
SECURITY_REFERRERPOLICY
SECURITY_XSSFILTER
```

### 继续阅读

所有主题的列表：

[Options](https://serverjs.io/documentation/options) 
[Context](https://serverjs.io/documentation/context)
[Router](https://serverjs.io/documentation/router) 
[Reply](https://serverjs.io/documentation/reply)
[Error](https://serverjs.io/documentation/errors)
[Testing](https://serverjs.io/documentation/testing)