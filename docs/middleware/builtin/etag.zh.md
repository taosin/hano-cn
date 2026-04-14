# ETag 中间件

使用此中间件，你可以轻松添加 ETag headers。

## 导入

```ts
import { Hono } from 'hono'
import { etag } from 'hono/etag'
```

## 用法

```ts
const app = new Hono()

app.use('/etag/*', etag())
app.get('/etag/abc', (c) => {
  return c.text('Hono is cool')
})
```

## 保留的 headers

304 响应必须包括在等效 200 OK 响应中发送的 headers。默认的 headers 是 Cache-Control、Content-Location、Date、ETag、Expires 和 Vary。

如果你想添加发送的 header，你可以使用 `retainedHeaders` 选项和包含默认 headers 的 `RETAINED_304_HEADERS` 字符串数组变量：

```ts
import { etag, RETAINED_304_HEADERS } from 'hono/etag'

// ...

app.use(
  '/etag/*',
  etag({
    retainedHeaders: ['x-message', ...RETAINED_304_HEADERS],
  })
)
```

## 选项

### <Badge type="info" text="optional" /> weak: `boolean`

定义是否使用 [弱验证](https://developer.mozilla.org/en-US/docs/Web/HTTP/Conditional_requests#weak_validation)。如果设置为 `true`，则在值的前缀添加 `w/`。默认值为 `false`。

### <Badge type="info" text="optional" /> retainedHeaders: `string[]`

你希望在 304 响应中保留的 headers。

### <Badge type="info" text="optional" /> generateDigest: `(body: Uint8Array) => ArrayBuffer | Promise<ArrayBuffer>`

自定义摘要生成函数。默认情况下，它使用 `SHA-1`。此函数使用响应体作为 `Uint8Array` 调用，并应返回哈希作为 `ArrayBuffer` 或其 Promise。
