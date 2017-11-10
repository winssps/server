# Context 

上下文是中间件接收的唯一参数，包含请求中此时可用的所有信息：

|name	|example	|type|
|-------|----------|-----|
|.options	|{ port: 3000, public: 'public' }|	Object|
|.data	|{ firstName: 'Francisco '}	 |Object|
|.params	|{ id: 42 }	|Object|
|.query	|{ search: '42' }	|Object|
|.session	|{ user: { firstName: 'Francisco' } } | Object|
|.headers	|{ 'Content-Type': 'application/json'  | Object|
|.cookie	|{ acceptCookieLaw: true }	|Object|
|.files	|{ profilepic: { ... } }	|Object|
|.ip	|'192.168.1.1'|	String|
|.url	|'/cats/?type=cute'|	String|
|.method	|'GET'	|String|
|.path	|'/cats/'	|String|
|.secure	|true	|Boolean|
|.xhr	|false	|Boolean|

它可以出现在几个点上，但最重要的是作为一个中间件参数：



```
// Load the server from the dependencies
const server = require('server');

// Display "Hello 世界" for any request
const middleware = ctx => {
  // ... (ctx is available here)
  return 'Hello 世界';
};

// Launch the server with a single middleware
server(middleware);
```

### .options

包含server.js使用的所有解析选项的对象。它结合了环境变量和显式选项``（{a：'b'}）``;


```
const mid = ctx => {
  expect(ctx.options.port).toBe(3012);
};

/* test */
const res = await run({ port: 3012 }, mid, () => 200).get('/');
expect(res.status).toBe(200);
```

如果我们在``.env``或者其他一些环境变量中设置了一个变量，那么它将使用它来代替环境选项。


```
 # .env
PORT=80

```
```
const mid = ctx => {
  expect(ctx.options.port).toBe(7693);
};

/* test */
const res = await run({ port: 7693 }, mid, () => 200).get('/');
expect(res.status).toBe(200);
```


### .data


这个别名 ``body`` 像其他库一样。这是与请求一起发送的数据。它可以是POST或PUT请求的一部分，但也可以由其他人设置，例如websocket：

```
const middle = ctx => {
  expect(ctx.data).toBe('Hello 世界');
};

// Test it (csrf set to false for testing purposes)
run(noCsrf, middle).post('/', { body: 'Hello 世界' });
run(middle).emit('message', 'Hello 世界');
```

处理正常发送的表单：

```
//- index.pug
form(method="POST" action="/contact")
  input(name="email")
  input(name="_csrf" value=csrf type="hidden")
  input(type="submit" value="Subscribe")
```

然后解析来自后端的数据：

```
const server = require('server');
const { get, post } = server.router;
const { render, redirect } = server.reply;

server([
  get(ctx => render('index.pug')),
  post(ctx => {
    console.log(ctx.data);  // Logs the email
    return redirect('/');
  })
]);
```

### .params

路由中指定的URL的参数：

```
const mid = get('/:type/:id', ctx => {
  expect(ctx.params.type).toBe('dog');
  expect(ctx.params.id).toBe('42');
});

// Test it
run(mid).get('/dog/42');
```

他们来自解析ctx.path路径包 path-to-regexp。去那里看看更多的信息。

```
const mid = del('/user/:id', ctx => {
  console.log('Delete user:', ctx.params.id);
});
```

### .query



发出请求时查询中的参数。这些来自url的片段 ``?answer= 42＆...``

```
const mid = ctx => {
  expect(ctx.query.answer).toBe('42');
  expect(ctx.query.name).toBe('Francisco');
};

// Test it
run(mid).get('/question?answer=42&name=Francisco');
```


### .session



在完成生产教程中的会话之后，会议应该准备好进行。这是用户刷新页面和导航之间持续的一个对象：


```
// Count how many pages the visitor sees
const mid = ctx => {
  ctx.session.counter = (ctx.session.counter || 0) + 1;
  return ctx.session.counter;
};

// Test that it works
run(ctx).alive(async ctx => {
  await api.get('/');
  await api.get('/');
  const res = await api.get('/');
  expect(res.body).toBe('3');
});
```

### .headers

获取与请求一起发送的标题：


```
const mid = ctx => {
  expect(ctx.headers.answer).toBe(42);
};

// Test it
run(mid).get('/', { headers: { answer: 42 } });
```

### .cookie


持有客户端发送的Cookie的对象：


```
const mid = ctx => {
  console.log(ctx.cookies);
};

run(mid).get('/');
```

### .files



包含请求发送的所有文件。它通常会通过一个``<input type =“file”>``字段或通过前端FormData的表单发送javascript：

```
<form method="POST" action="/profilepic" enctype="multipart/form-data">
  <input name="profilepic" type="input">
  <input type="hidden" name="_csrf" value="{{_csrf}}">
  <input type="submit" value="Send picture">
</form>
```


注意csrf标记和``enctype =“multipart / form-data”``，他们都需要。然后用Node.js来处理它：


```
const mid = post('/profilepic', ctx => {
  // This comes from the "name" in the input field
  console.log(ctx.files.profilepic);
  return redirect('/profile');
});
```


### .ip


远程客户端的IP。如果代理显示其代理条件，则``ips``字段也将填充相应的ips：

```
const mid = ctx => {
  console.log(ctx.ip, '|', ctx.ips);
};

run(mid).get('/');
```

### .url


完整的URL：

```
const mid = ctx => {
  expect(ctx.url).toBe('/hello?answer=42');
};

run(mid).get('/hello?answer=42');
```


### .method

请求方法，它可以是``GET``，``POST``，``PUT``，``DELETE``：


```
const mid = ctx => {
  expect(ctx.method).toBe('GET');
};

// Test it
run(mid).get('/');
```

或者其他方法：

```
const mid = ctx => {
  expect(ctx.method).toBe('POST');
};

// Test it
run(noCsrf, mid).post('/');
```

### .path


只有URL中的路径部分。这是查询以外的完整网址：

```
const mid = ctx => {
  expect(ctx.path).toBe('/question');
};

// Test it
run(mid).get('/question?answer=42');
```


### .secure

如果请求是通过HTTPS进行的，则返回true。考虑到，如果您在Cloudflare或类似的后面，即使您的客户端看到``https``，也可能会报告为false：


```
const mid = ctx => {
  expect(ctx.secure).toBe(false);
};

// Test it
run(mid).get('/');
```


### .xhr


如果请求是通过AJAX完成的，则布尔值设置为true。具体来说，如果``X-Requested-With``是``“XMLHttpRequest”``：

```
const mid = ctx => {
  expect(mid.xhr).toBe(false);
};

run(mid).get('/');
```

### 继续阅读

所有主题的列表：

[Options](https://serverjs.io/documentation/options) 
[Context](https://serverjs.io/documentation/context)
[Router](https://serverjs.io/documentation/router) 
[Reply](https://serverjs.io/documentation/reply)
[Error](https://serverjs.io/documentation/errors)
[Testing](https://serverjs.io/documentation/testing)

