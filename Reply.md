# Reply


答复是从创建响应的中间件返回的方法。这些是``server.reply``的可用方法及其参数：

|reply name	|example	|final|
|-------|-------|----------|-----|
|cookie(name, value, opts)	|cookie('name', 'Francisco')	|false
|download(path[, filename])	|download('resume.pdf')	|true
|header(field[, value])	|header('ETag': '12345')	|false
|json([data])	|json({ hello: 'world' })	|true
|jsonp([data])	|jsonp({ hello: 'world' })	|true
|redirect([status,] path)	|redirect(302, '/')	|true
|render(view[, locals])	|render('index.hbs')	|true
|send([body])	|send('Hello there')	|true
|status(code)	|status(200)	|mixed
|type(type)	|type('html')	|false


示例：

```
const { get, post } = require('server/router');
const { render, redirect, file } = require('server/reply');

module.exports = [
  get('/', ctx => render('index.hbs')),
  post('/', processRequest, ctx => redirect('/'))
];
```

确保返回您要使用的答复。否则将无法正常工作。


在中间件的上下文中解释了ctx参数。回复方法可以通过几种方式导入：

```
// For whenever you have previously defined `server`
const { send, json } = server.reply;

// For standalone files:
const { send, json } = require('server/reply');
```
导入回复方法的方法有很多种，但以上是推荐的方法。


### Chainable

虽然大部分答复是最终的，而且只能援引一次，但也有一些可以链接的。这些增加了正在进行的回应：

- cookie(): 添加cookie标头
- header(): 添加你想要的任何标题
- status(): 设置响应的状态	
- type(): 添加头“Content-Type”



你可以通过最终的方法把它们和其中的任何一个链接起来。如果在任何地方都没有调用最终方法，那么请求将以404响应结束。

``status（）``回复可以被用作最终的或者作为可链接的，如果其他东西被添加的话。


### Return value

无论是在同步模式还是异步模式，您都可以返回一个字符串来创建响应：


```
// Send a string
const middle = ctx => 'Hello 世界';

// Test it
const res = await run(middle).get('/');
expect(res.body).toBe('Hello 世界');
```

返回一个数组或者一个对象将会把它们串成JSON：

```
server(ctx => ['life', 42]);
// Note: extra parenthesis needed by the arrow function to return an object
server(ctx => ({ life: 42 }));
```

一个单一的号码将被解释为一个状态码，并将返回该状态的相应主体：

```
server(get('/nonexisting', => 404));
```

你也可以抛出任何东西来触发一个错误：


```
const middle = ({ req }) => {
  if (!req.body) {
    throw new Error('No body provided');
  }
}

const handler = error(ctx => ctx.error.message);

// Test it
const res = await run(middle, handler).get('/nonexisting');
expect(res.body).toBe('No body provided');
```

### Multiple replies

另一个重要的是，第一个使用的答复是将要使用的答复。但是，您应该尽量避免这一点，我们可能会在未来更严格：

```
// I hope you speak Spanish
server([
  ctx => 'Hola mundo',
  ctx => 'Hello world',
  ctx => 'こんにちは、世界'
]);
```

为了避免这种情况，只需在路由器中为每个请求指定url即可：


```
// I hope you speak Spanish
server([
  get('/es', ctx => 'Hola mundo'),
  get('/en', ctx => 'Hello world'),
  get('/jp', ctx => 'こんにちは、世界')
]);
```

然后，每个网址都将使用不同的语言。


### cookie()


在浏览器上设置一个cookie。它会发送Set-Cookie头文件：

```
const { cookie } = server.reply;
const setCookie = ctx => cookie('foo', 'bar').send();

// Test
run(setCookie).get('/').then(res => {
  expect(res.headers['Set-Cookie:']).toMatch(/foo\=bar/);
});
```

|Key	|Default	|Type
|-------|-----------|
|domain	|Current domain|	String
|encode	|encodeURIComponent	|Function
|expires	|undefined (session)|	Date
|httpOnly	|false	|Boolean
|maxAge|	undefined (session) |	Number
|path	|"/"	|String
|secure	|false	|Boolean
|signed	|false	|Boolean
|sameSite	|false	|Boolean or String


请在 express的文档 中查看更好的解释。

### download()

采用本地路径和可选文件名的异步函数。它将返回带有文件名称的本地文件，供浏览器下载。

```
server(ctx => download('user-file-5674354.pdf'));
server(ctx => download('user-file-5674354.pdf', 'report.pdf'));
```

您可以在下游处理此方法的错误：

```
server([
  ctx => download('user-file-5674354.pdf'),
  error(ctx => { console.log(ctx.error); })
]);
```

### header()

设置与响应一起发送的标题。它接受两个字符串作为键和值或一个对象来设置多个头：

```
const mid = ctx => header('Content-Type', 'text/plain');
const mid2 = ctx => header('Content-Length', '123');

// Same as above
const mid = ctx => header({
  'Content-Type': 'text/plain',
  'Content-Length': '123'
});
```

### json()


发送一个JSON响应。它接受一个普通的对象或者一个将被``JSON.stringify``字符串化的数组。设置正确的``Content-Type``标题：


```
const mid = ctx => json({ foo: 'bar' });

// Test it
run(mid).get('/').then(res => {
  expect(res.body).toEqual(`{"foo":"bar"}`);
});
```

### jsonp()

与json（）相同，但是用回调包装。阅读更多关于JSONP：


```
const mid = ctx => json({ foo: 'bar' });

// Test it
run(mid).get('/').then(res => {
  expect(res.body).toEqual(`callback({foo:"bar"})`);
});
```

加载数据跨域是非常有用的。向请求中添加一个 ``?callback = foo``查询来更改回调名称：


```
const mid = ctx => json({ foo: 'bar' });

// Test it
run(mid).get('/?callback=foo').then(res => {
  expect(res.body).toEqual(`foo({foo:"bar"})`);
});
```

### redirect()

重定向到指定的网址。它可以是内部（只是一个路径）或外部URL：

```
const mid1 = ctx => redirect('/foo');
const mid2 = ctx => redirect('../user');
const mid3 = ctx => redirect('https://google.com');
const mid4 = ctx => redirect(301, 'https://google.com');
```


### render()

这是最复杂的方法，也是最有用的方法。它需要一个文件名和一些数据，并呈现它：


```
const mid1 = ctx => render('index.hbs');
const mid2 = ctx => render('index.hbs', { user: 'Francisco' });
```

文件名是相对于views选项（默认为``'views'``）：

```
// Renders PROJECT/somefolder/index.hbs
server({ views: 'somefolder' }, ctx => render('index.hbs'));
```

这个文件名的扩展名是可选的。它接受默认的``.hbs``，``.pug``和``.html``，并可以接受更多的类型安装其他引擎：


```
const mid1 = ctx => render('index.pug');
const mid2 = ctx => render('index.hbs');
const mid3 = ctx => render('index.html');
```

数据将被传递给模板引擎。请注意，一些插件也可能传递额外的数据。
	

### send()


将数据发送到前端。这是默认使用原始回调的方法：

```
const mid1 = ctx => send('Hello 世界');
const mid2 = ctx => 'Hello 世界';
```

但是它支持更多的数据类型：字符串，对象，数组或缓冲区：

```
const mid1 = ctx => send('Hello 世界');
const mid2 = ctx => send('<p>Hello 世界</p>');
const mid4 = ctx => send({ foo: 'bar' });
const mid3 = ctx => send(new Buffer('whatever'));
```

它的优点是可以链接，不像返回字符串：

```
const mid1 = ctx => status(201).send({ resource: 'foobar' });
const mid2 = ctx => status(404).send('Not found');
const mid3 = ctx => status(500).send({ error: 'our fault' });
```

### status()

设置响应的状态。如果没有回复完成，它将变成最终的，并将该回复消息作为正文发送：


```
const mid1 = ctx => status(404);  // The same as:
const mid2 = ctx => status(404).send('Not found');
```


### type()

设置响应的``Content-Type``标头。它可以是一个明确的MIME类型，如下所示：

```
const mid1 = ctx => type('text/html').send('<p>Hello</p>');
const mid2 = ctx => type('application/json').send(JSON.stringify({ foo: 'bar' }));
const mid3 = ctx => type('image/png').send(...);
```
或者你也可以写出他们更友好的名字，以获得相同的结果：

```
const mid1 = ctx => type('.html');
const mid2 = ctx => type('html');
const mid3 = ctx => type('json');
const mid4 = ctx => type('application/json');
const mid5 = ctx => type('png');
```




### 继续阅读

所有主题的列表：

[Options](https://serverjs.io/documentation/options) 
[Context](https://serverjs.io/documentation/context)
[Router](https://serverjs.io/documentation/router) 
[Reply](https://serverjs.io/documentation/reply)
[Error](https://serverjs.io/documentation/errors)
[Testing](https://serverjs.io/documentation/testing)





