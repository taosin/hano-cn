# CORS 中间件

有许多将 Cloudflare Workers 用作 Web API 并从外部前端应用程序调用它们的用例。
对于它们，我们必须实现 CORS，让我们也用中间件来做这个。

## 导入

```ts
import { Hono } from 'hono'
import { cors } from 'hono/cors'
```

## 用法

```ts
const app = new Hono()

// CORS 应该在路由之前调用
app.use('/api/*', cors())
app.use(
  '/api2/*',
  cors({
    origin: 'http://example.com',
    allowHeaders: ['X-Custom-Header', 'Upgrade-Insecure-Requests'],
    allowMethods: ['POST', 'GET', 'OPTIONS'],
    exposeHeaders: ['Content-Length', 'X-Kuma-Revision'],
    maxAge: 600,
    credentials: true,
  })
)

app.all('/api/abc', (c) => {
  return c.json({ success: true })
})
app.all('/api2/abc', (c) => {
  return c.json({ success: true })
})
```

多个来源：

```ts
app.use(
  '/api3/*',
  cors({
    origin: ['https://example.com', 'https://example.org'],
  })
)

// 或者你可以使用 "function"
app.use(
  '/api4/*',
  cors({
    // `c` 是 `Context` 对象
    origin: (origin, c) => {
      return origin.endsWith('.example.com')
        ? origin
        : 'http://example.com'
    },
  })
)
```

基于来源的动态允许方法：

```ts
app.use(
  '/api5/*',
  cors({
    origin: (origin) =>
      origin === 'https://example.com' ? origin : '*',
    // `c` 是 `Context` 对象
    allowMethods: (origin, c) =>
      origin === 'https://example.com'
        ? ['GET', 'HEAD', 'POST', 'PATCH', 'DELETE']
        : ['GET', 'HEAD'],
  })
)
```

## 选项

### <Badge type="info" text="optional" /> origin: `string` | `string[]` | `(origin:string, c:Context) => string`

"_Access-Control-Allow-Origin_" CORS header 的值。你也可以传递回调函数，如 `origin: (origin) => (origin.endsWith('.example.com') ? origin : 'http://example.com')`。默认值为 `*`。

### <Badge type="info" text="optional" /> allowMethods: `string[]` | `(origin:string, c:Context) => string[]`

"_Access-Control-Allow-Methods_" CORS header 的值。你也可以传递回调函数来根据来源动态确定允许的方法。默认值为 `['GET', 'HEAD', 'PUT', 'POST', 'DELETE', 'PATCH']`。

### <Badge type="info" text="optional" /> allowHeaders: `string[]`

"_Access-Control-Allow-Headers_" CORS header 的值。默认值为 `[]`。

### <Badge type="info" text="optional" /> maxAge: `number`

"_Access-Control-Max-Age_" CORS header 的值。

### <Badge type="info" text="optional" /> credentials: `boolean`

"_Access-Control-Allow-Credentials_" CORS header 的值。

### <Badge type="info" text="optional" /> exposeHeaders: `string[]`

"_Access-Control-Expose-Headers_" CORS header 的值。默认值为 `[]`。

## 依赖于环境的 CORS 配置

如果你想根据执行环境（如开发或生产）调整 CORS 配置，从环境变量注入值很方便，因为它消除了应用程序感知其自身执行环境的需要。请参阅下面的示例以澄清。

```ts
app.use('*', async (c, next) => {
  const corsMiddlewareHandler = cors({
    origin: c.env.CORS_ORIGIN,
  })
  return corsMiddlewareHandler(c, next)
})
```

## 与 Vite 一起使用

当将 Hono 与 Vite 一起使用时，你应该通过在 `vite.config.ts` 中将 `server.cors` 设置为 `false` 来禁用 Vite 的内置 CORS 功能。这可以防止与 Hono 的 CORS 中间件冲突。

```ts
// vite.config.ts
import { cloudflare } from '@cloudflare/vite-plugin'
import { defineConfig } from 'vite'

export default defineConfig({
  server: {
    cors: false, // 禁用 Vite 的内置 CORS 设置
  },
  plugins: [cloudflare()],
})
```
