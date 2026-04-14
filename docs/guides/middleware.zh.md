# 中间件

中间件在端点 `Handler` 之前/之后工作。我们可以在调度之前获取 `Request` 或在调度之后操作 `Response`。

## 中间件的定义

- Handler - 应该返回 `Response` 对象。只有一个处理程序会被调用。
- Middleware - 应该 `await next()` 并返回 nothing 以调用下一个中间件，**或** 返回 `Response` 以提前退出。

用户可以使用 `app.use` 或使用 `app.HTTP_METHOD` 注册中间件，就像处理程序一样。这个功能很容易指定路径和方法。

```ts
// 匹配任何方法，所有路由
app.use(logger())

// 指定路径
app.use('/posts/*', cors())

// 指定方法和路径
app.post('/posts/*', basicAuth())
```

如果处理程序返回 `Response`，它将用于最终用户并停止处理。

```ts
app.post('/posts', (c) => c.text('Created!', 201))
```

在这种情况下，四个中间件在调度之前按如下方式处理：

```ts
logger() -> cors() -> basicAuth() -> *handler*
```

## 执行顺序

中间件的执行顺序由注册顺序决定。
第一个注册的中间件的 `next` 之前的处理首先执行，
`next` 之后的处理最后执行。
看下面。

```ts
app.use(async (_, next) => {
  console.log('middleware 1 start')
  await next()
  console.log('middleware 1 end')
})
app.use(async (_, next) => {
  console.log('middleware 2 start')
  await next()
  console.log('middleware 2 end')
})
app.use(async (_, next) => {
  console.log('middleware 3 start')
  await next()
  console.log('middleware 3 end')
})

app.get('/', (c) => {
  console.log('handler')
  return c.text('Hello!')
})
```

结果如下。

```
middleware 1 start
  middleware 2 start
    middleware 3 start
      handler
    middleware 3 end
  middleware 2 end
middleware 1 end
```

注意，如果处理程序或任何中间件抛出，hono 将捕获它并将其传递给 [你的 app.onError() 回调](/docs/api/hono#error-handling) 或在返回到中间件链之前自动将其转换为 500 响应。这意味着 next() 永远不会抛出，所以不需要用 try/catch/finally 包装它。

## 内置中间件

Hono 有内置中间件。

```ts
import { Hono } from 'hono'
import { poweredBy } from 'hono/powered-by'
import { logger } from 'hono/logger'
import { basicAuth } from 'hono/basic-auth'

const app = new Hono()

app.use(poweredBy())
app.use(logger())

app.use(
  '/auth/*',
  basicAuth({
    username: 'hono',
    password: 'acoolproject',
  })
)
```

::: warning
在 Deno 中，可以使用与 Hono 版本不同的中间件版本，但这可能导致错误。
例如，这段代码不工作，因为版本不同。

```ts
import { Hono } from 'jsr:@hono/hono@4.4.0'
import { upgradeWebSocket } from 'jsr:@hono/hono@4.4.5/deno'

const app = new Hono()

app.get(
  '/ws',
  upgradeWebSocket(() => ({
    // ...
  }))
)
```

:::

## 自定义中间件

你可以直接在 `app.use()` 中编写自己的中间件：

```ts
// 自定义日志记录器
app.use(async (c, next) => {
  console.log(`[${c.req.method}] ${c.req.url}`)
  await next()
})

// 添加自定义头部
app.use('/message/*', async (c, next) => {
  await next()
  c.header('x-message', 'This is middleware!')
})

app.get('/message/hello', (c) => c.text('Hello Middleware!'))
```

但是，直接将中间件嵌入 `app.use()` 会限制其可重用性。因此，我们可以将中间件分离到不同的文件中。

为了确保我们不会丢失 `context` 和 `next` 的类型定义，当分离中间件时，我们可以使用 Hono 工厂中的 [`createMiddleware()`](/docs/helpers/factory#createmiddleware)。这也允许我们类型安全地 [访问我们在 `Context` 中 `set` 的数据](https://hono.dev/docs/api/context#set-get) 从下游处理程序。

```ts
import { createMiddleware } from 'hono/factory'

const logger = createMiddleware(async (c, next) => {
  console.log(`[${c.req.method}] ${c.req.url}`)
  await next()
})
```

:::info
类型泛型可以与 `createMiddleware` 一起使用：

```ts
createMiddleware<{Bindings: Bindings}>(async (c, next) =>
```

:::

### 在 Next 之后修改响应

此外，中间件可以设计为在必要时修改响应：

```ts
const stripRes = createMiddleware(async (c, next) => {
  await next()
  c.res = undefined
  c.res = new Response('New Response')
})
```

## 在中间件参数中访问 Context

要在中间件参数内部访问 context，直接使用 `app.use` 提供的 context 参数。看下面的示例以澄清。

```ts
import { cors } from 'hono/cors'

app.use('*', async (c, next) => {
  const middleware = cors({
    origin: c.env.CORS_ORIGIN,
  })
  return middleware(c, next)
})
```

### 在中间件中扩展 Context

要在中间件内部扩展 context，使用 `c.set`。你可以通过向 `createMiddleware` 函数传递 `{ Variables: { yourVariable: YourVariableType } }` 泛型参数来使其类型安全。

```ts
import { createMiddleware } from 'hono/factory'

const echoMiddleware = createMiddleware<{
  Variables: {
    echo: (str: string) => string
  }
}>(async (c, next) => {
  c.set('echo', (str) => str)
  await next()
})

app.get('/echo', echoMiddleware, (c) => {
  return c.text(c.var.echo('Hello!'))
})
```

### 跨链式中间件的类型推断

当你使用 `.use()` 链接多个中间件时，Hono 自动累积 `Variables` 类型。跟随中间件链的路由处理程序可以类型安全地访问所有前面中间件的所有变量：

```ts
import { createMiddleware } from 'hono/factory'

const authMiddleware = createMiddleware<{
  Variables: { user: { id: string; name: string } }
}>(async (c, next) => {
  c.set('user', { id: '123', name: 'Alice' })
  await next()
})

const dbMiddleware = createMiddleware<{
  Variables: { db: { query: (sql: string) => Promise<unknown> } }
}>(async (c, next) => {
  c.set('db', {
    query: async (sql) => {
      /* ... */
    },
  })
  await next()
})

const app = new Hono()
  .use(authMiddleware)
  .use(dbMiddleware)
  .get('/', (c) => {
    // `user` 和 `db` 都可用且类型安全
    const user = c.var.user // { id: string; name: string }
    const db = c.var.db // { query: (sql: string) => Promise<unknown> }
    return c.json({ user })
  })
```

这是因为每个 `.use()` 调用返回一个新的 Hono 实例，具有合并的类型，所以类型随着中间件链接而增长。这消除了大多数用例需要手动声明组合的 `Env` 类型的需要。

## 第三方中间件

内置中间件不依赖外部模块，但第三方中间件可以依赖第三方库。因此，使用它们，我们可以制作更复杂的应用程序。

我们可以探索各种 [第三方中间件](https://hono.dev/docs/middleware/third-party)。
例如，我们有 GraphQL 服务器中间件、Sentry 中间件、Firebase 认证中间件等。
