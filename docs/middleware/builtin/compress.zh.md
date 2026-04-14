# Compress 中间件

此中间件根据 `Accept-Encoding` 请求 header 压缩响应体。

::: info
**注意**：在 Cloudflare Workers 和 Deno Deploy 上，响应体将自动压缩，因此无需使用此中间件。
:::

## 导入

```ts
import { Hono } from 'hono'
import { compress } from 'hono/compress'
```

## 用法

```ts
const app = new Hono()

app.use(compress())
```

## 选项

### <Badge type="info" text="optional" /> encoding: `'gzip'` | `'deflate'`

允许响应压缩的压缩方案。`gzip` 或 `deflate`。如果未定义，则两者都允许，并将基于 `Accept-Encoding` header 使用。如果未提供此选项且客户端在 `Accept-Encoding` header 中同时提供两者，则优先使用 `gzip`。

### <Badge type="info" text="optional" /> threshold: `number`

要压缩的最小大小（以字节为单位）。默认为 1024 字节。
