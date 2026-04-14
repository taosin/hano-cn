# Body Limit 中间件

Body Limit 中间件可以限制请求体的文件大小。

此中间件首先使用请求中 `Content-Length` header 的值（如果存在）。
如果未设置，它将在流中读取 body，如果大于指定的文件大小，则执行错误处理器。

## 导入

```ts
import { Hono } from 'hono'
import { bodyLimit } from 'hono/body-limit'
```

## 用法

```ts
const app = new Hono()

app.post(
  '/upload',
  bodyLimit({
    maxSize: 50 * 1024, // 50kb
    onError: (c) => {
      return c.text('overflow :(', 413)
    },
  }),
  async (c) => {
    const body = await c.req.parseBody()
    if (body['file'] instanceof File) {
      console.log(`Got file sized: ${body['file'].size}`)
    }
    return c.text('pass :)')
  }
)
```

## 选项

### <Badge type="danger" text="required" /> maxSize: `number`

你想要限制的最大文件大小。默认值为 `100 * 1024` - `100kb`。

### <Badge type="info" text="optional" /> onError: `OnError`

如果超过指定的文件大小，将调用此错误处理器。

## 与 Bun 一起使用以处理大请求

如果显式使用 Body Limit 中间件来允许大于默认值的请求体，则可能需要相应地更改 `Bun.serve` 配置。[在撰写本文时](https://github.com/oven-sh/bun/blob/f2cfa15e4ef9d730fc6842ad8b79fb7ab4c71cb9/packages/bun-types/bun.d.ts#L2191)，`Bun.serve` 的默认请求体限制为 128MiB。如果你将 Hono 的 Body Limit 中间件设置为大于该值，你的请求仍将失败，此外，中间件中指定的 `onError` 处理器将不会被调用。这是因为 `Bun.serve()` 将在将请求传递给 Hono 之前将状态码设置为 `413` 并终止连接。

如果你想使用 Hono 和 Bun 接受大于 128MiB 的请求，你也需要为 Bun 设置限制：

```ts
export default {
  port: process.env['PORT'] || 3000,
  fetch: app.fetch,
  maxRequestBodySize: 1024 * 1024 * 200, // 你的值
}
```

或者，根据你的设置：

```ts
Bun.serve({
  fetch(req, server) {
    return app.fetch(req, { ip: server.requestIP(req) })
  },
  maxRequestBodySize: 1024 * 1024 * 200, // 你的值
})
```
