# Context

`Context` 对象为每个请求实例化并保留到响应返回。你可以在其中放置值，设置想要返回的头部和状态码，并访问 HonoRequest 和 Response 对象。

## req

`req` 是 HonoRequest 的实例。更多详情，请参阅 [HonoRequest](/docs/api/request)。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/hello', (c) => {
  const userAgent = c.req.header('User-Agent')
  // ...
  // ---cut-start---
  return c.text(`Hello, ${userAgent}`)
  // ---cut-end---
})
```

## status()

你可以使用 `c.status()` 设置 HTTP 状态码。默认是 `200`。如果代码是 `200`，你不需要使用 `c.status()`。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.post('/posts', (c) => {
  // 设置 HTTP 状态码
  c.status(201)
  return c.text('Your post is created!')
})
```

## header()

你可以为响应设置 HTTP 头部。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/', (c) => {
  // 设置头部
  c.header('X-Message', 'My custom message')
  return c.text('Hello!')
})
```

## body()

返回 HTTP 响应。

::: info
**注意**：返回文本或 HTML 时，推荐使用 `c.text()` 或 `c.html()`。
:::

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/welcome', (c) => {
  c.header('Content-Type', 'text/plain')
  // 返回响应体
  return c.body('Thank you for coming')
})
```

你也可以这样写：

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/welcome', (c) => {
  return c.body('Thank you for coming', 201, {
    'X-Message': 'Hello!',
    'Content-Type': 'text/plain',
  })
})
```

响应与以下 `Response` 对象相同。

```ts twoslash
new Response('Thank you for coming', {
  status: 201,
  headers: {
    'X-Message': 'Hello!',
    'Content-Type': 'text/plain',
  },
})
```

## text()

将文本渲染为 `Content-Type: text/plain`。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/say', (c) => {
  return c.text('Hello!')
})
```

## json()

将 JSON 渲染为 `Content-Type: application/json`。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/api', (c) => {
  return c.json({ message: 'Hello!' })
})
```

## html()

将 HTML 渲染为 `Content-Type: text/html`。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/', (c) => {
  return c.html('<h1>Hello! Hono!</h1>')
})
```

## notFound()

返回 `Not Found` 响应。你可以使用 [`app.notFound()`](/docs/api/hono#not-found) 自定义它。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/notfound', (c) => {
  return c.notFound()
})
```

## redirect()

重定向，默认状态码是 `302`。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/redirect', (c) => {
  return c.redirect('/')
})
app.get('/redirect-permanently', (c) => {
  return c.redirect('/', 301)
})
```

## res

你可以访问将要返回的 [Response] 对象。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
// Response 对象
app.use('/', async (c, next) => {
  await next()
  c.res.headers.append('X-Debug', 'Debug message')
})
```

[Response]: https://developer.mozilla.org/en-US/docs/Web/API/Response

## set() / get()

获取和设置任意键值对，生命周期为当前请求。这允许在中间件之间或从中间件到路由处理程序传递特定值。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono<{ Variables: { message: string } }>()
// ---cut---
app.use(async (c, next) => {
  c.set('message', 'Hono is cool!!')
  await next()
})

app.get('/', (c) => {
  const message = c.get('message')
  return c.text(`The message is "${message}"`)
})
```

将 `Variables` 作为泛型传递给 `Hono` 的构造函数以使其类型安全。

```ts twoslash
import { Hono } from 'hono'
// ---cut---
type Variables = {
  message: string
}

const app = new Hono<{ Variables: Variables }>()
```

`c.set` / `c.get` 的值仅在同一请求内保留。它们不能跨不同请求共享或持久化。

## var

你也可以使用 `c.var` 访问变量的值。

```ts twoslash
import type { Context } from 'hono'
declare const c: Context
// ---cut---
const result = c.var.client.oneMethod()
```

如果你想创建提供自定义方法的中间件，
像下面这样写：

```ts twoslash
import { Hono } from 'hono'
import { createMiddleware } from 'hono/factory'
// ---cut---
type Env = {
  Variables: {
    echo: (str: string) => string
  }
}

const app = new Hono()

const echoMiddleware = createMiddleware<Env>(async (c, next) => {
  c.set('echo', (str) => str)
  await next()
})

app.get('/echo', echoMiddleware, (c) => {
  return c.text(c.var.echo('Hello!'))
})
```

如果你想在多个处理程序中使用中间件，可以使用 `app.use()`。
然后，你必须将 `Env` 作为泛型传递给 `Hono` 的构造函数以使其类型安全。

```ts twoslash
import { Hono } from 'hono'
import type { MiddlewareHandler } from 'hono/types'
declare const echoMiddleware: MiddlewareHandler
type Env = {
  Variables: {
    echo: (str: string) => string
  }
}
// ---cut---
const app = new Hono<Env>()

app.use(echoMiddleware)

app.get('/echo', (c) => {
  return c.text(c.var.echo('Hello!'))
})
```

## render() / setRenderer()

你可以使用 `c.setRenderer()` 在自定义中间件中设置布局。

```tsx twoslash
/** @jsx jsx */
/** @jsxImportSource hono/jsx */
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.use(async (c, next) => {
  c.setRenderer((content) => {
    return c.html(
      <html>
        <body>
          <p>{content}</p>
        </body>
      </html>
    )
  })
  await next()
})
```

然后，你可以利用 `c.render()` 在此布局中创建响应。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/', (c) => {
  return c.render('Hello!')
})
```

输出将是：

```html
<html>
  <body>
    <p>Hello!</p>
  </body>
</html>
```

此外，此功能提供了自定义参数的灵活性。
为确保类型安全，可以定义类型为：

```ts
declare module 'hono' {
  interface ContextRenderer {
    (
      content: string | Promise<string>,
      head: { title: string }
    ): Response | Promise<Response>
  }
}
```

以下是如何使用此功能的示例：

```ts
app.use('/pages/*', async (c, next) => {
  c.setRenderer((content, head) => {
    return c.html(
      <html>
        <head>
          <title>{head.title}</title>
        </head>
        <body>
          <header>{head.title}</header>
          <p>{content}</p>
        </body>
      </html>
    )
  })
  await next()
})

app.get('/pages/my-favorite', (c) => {
  return c.render(<p>Ramen and Sushi</p>, {
    title: 'My favorite',
  })
})

app.get('/pages/my-hobbies', (c) => {
  return c.render(<p>Watching baseball</p>, {
    title: 'My hobbies',
  })
})
```

## executionCtx

你可以访问 Cloudflare Workers 特定的 [ExecutionContext](https://developers.cloudflare.com/workers/runtime-apis/context/)。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono<{
  Bindings: {
    KV: any
  }
}>()
declare const key: string
declare const data: string
// ---cut---
// ExecutionContext 对象
app.get('/foo', async (c) => {
  c.executionCtx.waitUntil(c.env.KV.put(key, data))
  // ...
})
```

`ExecutionContext` 还有一个 [`exports`](https://developers.cloudflare.com/workers/runtime-apis/context/#exports) 字段。要使用 Wrangler 生成的类型获得自动补全，你可以使用模块增强：

```ts
import 'hono'

declare module 'hono' {
  interface ExecutionContext {
    readonly exports: Cloudflare.Exports
  }
}
```

## event

你可以访问 Cloudflare Workers 特定的 `FetchEvent`。这在 "Service Worker" 语法中使用。但现在不推荐。

```ts twoslash
import { Hono } from 'hono'
declare const key: string
declare const data: string
type KVNamespace = any
// ---cut---
// 类型定义以进行类型推断
type Bindings = {
  MY_KV: KVNamespace
}

const app = new Hono<{ Bindings: Bindings }>()

// FetchEvent 对象（仅在使用 Service Worker 语法时设置）
app.get('/foo', async (c) => {
  c.event.waitUntil(c.env.MY_KV.put(key, data))
  // ...
})
```

## env

在 Cloudflare Workers 中，绑定到 worker 的环境变量、密钥、KV 命名空间、D1 数据库、R2 存储桶等称为绑定。
无论类型如何，绑定始终可作为全局变量使用，并可以通过上下文 `c.env.BINDING_KEY` 访问。

```ts twoslash
import { Hono } from 'hono'
type KVNamespace = any
// ---cut---
// 类型定义以进行类型推断
type Bindings = {
  MY_KV: KVNamespace
}

const app = new Hono<{ Bindings: Bindings }>()

// Cloudflare Workers 的环境对象
app.get('/', async (c) => {
  c.env.MY_KV.get('my-key')
  // ...
})
```

## error

如果处理程序抛出错误，错误对象放在 `c.error` 中。
你可以在中间件中访问它。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.use(async (c, next) => {
  await next()
  if (c.error) {
    // 做一些事情...
  }
})
```

## ContextVariableMap

例如，如果你想在特定中间件使用时向变量添加类型定义，你可以扩展 `ContextVariableMap`。例如：

```ts
declare module 'hono' {
  interface ContextVariableMap {
    result: string
  }
}
```

然后你可以在中间件中使用它：

```ts twoslash
import { createMiddleware } from 'hono/factory'
// ---cut---
const mw = createMiddleware(async (c, next) => {
  c.set('result', 'some values') // result 是 string
  await next()
})
```

在处理程序中，变量被推断为适当的类型：

```ts twoslash
import { Hono } from 'hono'
const app = new Hono<{ Variables: { result: string } }>()
// ---cut---
app.get('/', (c) => {
  const val = c.get('result') // val 是 string
  // ...
  return c.json({ result: val })
})
```
