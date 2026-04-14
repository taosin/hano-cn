# HonoRequest

`HonoRequest` 是一个可以从 `c.req` 获取的对象，它包装了 [Request](https://developer.mozilla.org/en-US/docs/Web/API/Request) 对象。

## param()

获取路径参数的值。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
// 捕获的参数
app.get('/entry/:id', async (c) => {
  const id = c.req.param('id')
  //    ^?
  // ...
})

// 一次性获取所有参数
app.get('/entry/:id/comment/:commentId', async (c) => {
  const { id, commentId } = c.req.param()
  //      ^?
})
```

## query()

获取查询字符串参数。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
// 查询参数
app.get('/search', async (c) => {
  const query = c.req.query('q')
  //     ^?
})

// 一次性获取所有参数
app.get('/search', async (c) => {
  const { q, limit, offset } = c.req.query()
  //      ^?
})
```

## queries()

获取多个查询字符串参数值，例如 `/search?tags=A&tags=B`

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/search', async (c) => {
  // tags 将是 string[]
  const tags = c.req.queries('tags')
  //     ^?
  // ...
})
```

## header()

获取请求头部值。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/', (c) => {
  const userAgent = c.req.header('User-Agent')
  //      ^?
  return c.text(`Your user agent is ${userAgent}`)
})
```

::: warning
当 `c.req.header()` 不带参数调用时，返回记录中的所有键都是**小写**的。

如果你想获取大写名称头部的值，
使用 `c.req.header("X-Foo")`。

```ts
// ❌ 不会工作
const headerRecord = c.req.header()
const foo = headerRecord['X-Foo']

// ✅ 会工作
const foo = c.req.header('X-Foo')
```

:::

## parseBody()

解析 `multipart/form-data` 或 `application/x-www-form-urlencoded` 类型的请求体。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.post('/entry', async (c) => {
  const body = await c.req.parseBody()
  // ...
})
```

`parseBody()` 支持以下行为。

**单个文件**

```ts twoslash
import { Context } from 'hono'
declare const c: Context
// ---cut---
const body = await c.req.parseBody()
const data = body['foo']
//    ^?
```

`body['foo']` 是 `(string | File)`。

如果上传多个文件，将使用最后一个。

### 多个文件

```ts twoslash
import { Context } from 'hono'
declare const c: Context
// ---cut---
const body = await c.req.parseBody()
body['foo[]']
```

`body['foo[]']` 始终是 `(string | File)[]`。

需要 `[]` 后缀。

### 多个文件或同名字段

如果你有允许多个 `<input type="file" multiple />` 的输入字段或多个同名的复选框 `<input type="checkbox" name="favorites" value="Hono"/>`。

```ts twoslash
import { Context } from 'hono'
declare const c: Context
// ---cut---
const body = await c.req.parseBody({ all: true })
body['foo']
```

`all` 选项默认禁用。

- 如果 `body['foo']` 是多个文件，将解析为 `(string | File)[]`。
- 如果 `body['foo']` 是单个文件，将解析为 `(string | File)`。

### 点符号

如果你将 `dot` 选项设置为 `true`，返回值将根据点符号结构化。

想象接收以下数据：

```ts twoslash
const data = new FormData()
data.append('obj.key1', 'value1')
data.append('obj.key2', 'value2')
```

你可以通过将 `dot` 选项设置为 `true` 来获取结构化的值：

```ts twoslash
import { Context } from 'hono'
declare const c: Context
// ---cut---
const body = await c.req.parseBody({ dot: true })
// body 是 `{ obj: { key1: 'value1', key2: 'value2' } }`
```

## json()

解析 `application/json` 类型的请求体。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.post('/entry', async (c) => {
  const body = await c.req.json()
  // ...
})
```

## text()

解析 `text/plain` 类型的请求体。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.post('/entry', async (c) => {
  const body = await c.req.text()
  // ...
})
```

## arrayBuffer()

将请求体解析为 `ArrayBuffer`。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.post('/entry', async (c) => {
  const body = await c.req.arrayBuffer()
  // ...
})
```

## blob()

将请求体解析为 `Blob`。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.post('/entry', async (c) => {
  const body = await c.req.blob()
  // ...
})
```

## formData()

将请求体解析为 `FormData`。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.post('/entry', async (c) => {
  const body = await c.req.formData()
  // ...
})
```

## valid()

获取验证后的数据。

```ts
app.post('/posts', async (c) => {
  const { title, body } = c.req.valid('form')
  // ...
})
```

可用目标如下。

- `form`
- `json`
- `query`
- `header`
- `cookie`
- `param`

使用示例请参阅 [验证部分](/docs/guides/validation)。

## routePath

::: warning
**v4.8.0 中已弃用**：此属性已弃用。改用 [路由辅助函数](/docs/helpers/route) 中的 `routePath()`。
:::

你可以像这样在处理程序中获取注册的路径：

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/posts/:id', (c) => {
  return c.json({ path: c.req.routePath })
})
```

如果你访问 `/posts/123`，它将返回 `/posts/:id`：

```json
{ "path": "/posts/:id" }
```

## matchedRoutes

::: warning
**v4.8.0 中已弃用**：此属性已弃用。改用 [路由辅助函数](/docs/helpers/route) 中的 `matchedRoutes()`。
:::

它在处理程序中返回匹配的路由，这对于调试很有用。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.use(async function logger(c, next) {
  await next()
  c.req.matchedRoutes.forEach(({ handler, method, path }, i) => {
    const name =
      handler.name ||
      (handler.length < 2 ? '[handler]' : '[middleware]')
    console.log(
      method,
      ' ',
      path,
      ' '.repeat(Math.max(10 - path.length, 0)),
      name,
      i === c.req.routeIndex ? '<- respond from here' : ''
    )
  })
})
```

## path

请求路径名。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/about/me', async (c) => {
  const pathname = c.req.path // `/about/me`
  // ...
})
```

## url

请求 URL 字符串。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/about/me', async (c) => {
  const url = c.req.url // `http://localhost:8787/about/me`
  // ...
})
```

## method

请求的方法名。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/about/me', async (c) => {
  const method = c.req.method // `GET`
  // ...
})
```

## raw

原始 [`Request`](https://developer.mozilla.org/en-US/docs/Web/API/Request) 对象。

```ts
// 对于 Cloudflare Workers
app.post('/', async (c) => {
  const metadata = c.req.raw.cf?.hostMetadata?
  // ...
})
```

## cloneRawRequest()

从 HonoRequest 克隆原始 Request 对象。即使在请求体已被验证器或 HonoRequest 方法消耗后也能工作。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()

import { cloneRawRequest } from 'hono/request'
import { validator } from 'hono/validator'

app.post(
  '/forward',
  validator('json', (data) => data),
  async (c) => {
    // 验证后克隆
    const clonedReq = await cloneRawRequest(c.req)
    // 不会抛出错误
    await clonedReq.json()
    // ...
  }
)
```
