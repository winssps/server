# Router

server.router的可用方法及其参数：

| name	| example |
|-------|-------- |
| get(PATH, FN1, FN2, ...)	| get('/', ctx => { ... })  |
|post(PATH, FN1, FN2, ...)	| post('/', ctx => { ... }) |
|put(PATH, FN1, FN2, ...)	| put('/', ctx => { ... })|
|del(PATH, FN1, FN2, ...)	| del('/', ctx => { ... })  |
|error(NAME, FN1, FN2, ...)	 | error('user', ctx => { ... }) |
|sub(SUBDOMAIN, FN1, FN2, ...)	| sub('es', ctx => { ... })  |
|socket(NAME, FN1, FN2, ...) |	socket('/', ctx => { ... })  |


路由器是一个告诉服务器如何处理每个请求的函数。它们是一种特定类型的中间件，它包装了你的逻辑并作为一个网关：

```
// Import methods 'get' and 'post' from the router
const { get, post } = require('server/router');

server([
  get('/', ctx => { /* ... */ }),      // Render homepage
  get('/users', ctx => { /* ... */ }), // GET requests to /users
  post('/users', ctx => { /* ... */ }) // POST requests to /users
]);
```

在中间件的上下文中解释了ctx参数。路由器方法可以通过几种方式导入：

```
// For whenever you have previously defined `server`
const { get, post } = server.router;

// For standalone files:
const { get, post } = require('server/router');
```

导入路由器方法的方法有很多种，但以上是推荐的方法。


### Complex routers

如果你要有很多routes，我们建议把它们分成不同的文件，或者在项目的根目录中作为``routes.js``或者在不同的地方：

```
// app.js
const server = require('server');
const routes = require('./routes');

server(routes);
// routes.js
const { get, post } = require('server/router');
const ctrl = require('auto-load')('controllers');

// You can simply export an array of routes
module.exports = [
  get('/', ctrl.home.index),
  get('/users', ctrl.users.index),
  post('/users', ctrl.users.add),
  get('/photos', ctrl.photos.index),
  post('/photos', ctrl.photos.add),
  ...
];
```

``ctx``变量是上下文（文档在这里）。路由和中间件之间的一个重要区别是所有的路由都是最终的。这意味着每个请求最多只使用一条路由。


所有的路由器都驻留在``server.router``中，并遵循以下结构：

```
const server = require('server');
const { TYPE } = server.router;
const doSomething = TYPE(ID, fn1, [fn2], [fn3]);
server(doSomething);
```

### CSRF token

对于POST，PUT和DELETE请求，还必须发送字段名为``_csrf``的有效CSRF token。局部变量由``server.js``设置，所以你可以像这样包含它：

```
<form action="/" method="POST">
 <input name="firstname">
 <input type="submit" value="Contact us">
 <input type="hidden" name="_csrf" value="{{csrf}}">
</form>
```

如果您使用Javascript中的API，比如新的``fetch（）``，则可以通过以下方式处理：

```
<!-- within your main template -->
<script>
  window.csrf = '{{csrf}}';
</script>
```

```
// Within your javascript.js/bundle.js/app.js
fetch('/', {
  method: 'POST',
  body: 'hello world',
  credentials: 'include',  // Important! to maintain the session
  headers: { 'csrf-token': csrf }  // From 'window'
}).then(...);
```

或者，如果你知道自己在做什么，也可以禁用它：

```
server({ security: { csrf: false } }, ...);
```

### get()

处理``GET``类型的请求（加载网页）：

```
// Create a single route for GET /
const route = get('/', ctx => 'Hello 世界');

// Testing that it actually works
run(route).get('/').then(res => {
  expect(res.body).toBe('Hello 世界');
});
```

>
注意：阅读代码示例中的更多测试，或者忽略它们
>

你可以指定一个查询和参数设置：


```
const route = get('/:page', ctx => {
  console.log(ctx.params.page);  // hello
  console.log(ctx.query.name);   // Francisco
  return { page: ctx.params.page, name: ctx.query.name };
});

// Test it
run(route).get('/hello?name=Francisco').then(res => {
  expect(res.body).toEqual({ page: 'hello', name: 'Francisco' });
});
```

### post()

处理``POST``类型的请求。它需要提供一个csrf标记：


```
// Create a single route for POST /
const route = post('/', ctx => {
  console.log(ctx.data);
});

// Test our route. Note: csrf disabled for testing purposes
run(noCsrf, route).post('/', { body: 'Hello 世界' });
```

``data``属性可以是字符串或``{name：value}``对的简单对象。

示例:

```
// index.js
const server = require('server');
const { get, post } = server.router;
const { file, redirect } = server.reply;

server(
  get('/', ctx => file('index.hbs')),
  post('/', ctx => {
    // Show the submitted data on the console:
    console.log(ctx.data);
    return redirect('/');
  })
);
```


```

<form method="POST" action="/">
  <h2>Contact us</h1>
  <label><p>Name:</p> <input type="text" name="fullname"></label>
  <label><p>Message:</p> <textarea name="message"></textarea></label>

  <input type="hidden" name="_csrf" value="{{csrf}}">
  <input type="submit" name="Contact us">
</form>

```

示例2：JSON API。要使用JSON进行POST，您可以遵循以下步骤


```
fetch('/42', {
  method: 'PUT',
  body: JSON.stringify({ a: 'b', c: 'd' }),
  credentials: 'include', // !important for the CSRF
  headers: {
    'csrf-token': csrf,
    'Content-Type': 'application/json'
  }
}).then(res => res.json()).then(item => {
  console.log(item);
});
```
