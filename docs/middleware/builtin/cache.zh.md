# Cache 中间件

Cache 中间件使用 Web 标准的 [Cache API](https://developer.mozilla.org/en-US/docs/Web/API/Cache)。

Cache 中间件目前支持使用自定义域的 Cloudflare Workers 项目和使用 [Deno 1.26+](https://github.com/denoland/deno/releases/tag/v1.26.0) 的 Deno 项目。也可用于 Deno Deploy。

Cloudflare Workers 尊重 `Cache-Control` header 并返回缓存的响应。有关详细信息，请参阅 [Cloudflare 文档上的 Cache](https://developers.cloudflare.com/workers/runtime-apis/cache/)。Deno 不尊重 headers，因此如果你需要更新缓存，你需要实现自己的机制。

有关每个平台的说明，请参阅下面的 [用法](#usage)。

## 导入

```ts
import { Hono } from 'hono'
import { cache } from 'hono/cache'
```

## 用法

::: code-group

```ts [Cloudflare Workers]
app.get(
  '*',
  cache({
    cacheName: 'my-app',
    cacheControl: 'max-age=3600',
  })
)
```

```ts [Deno]
// 必须为 Deno 运行时使用 `wait: true`
app.get(
  '*',
  cache({
    cacheName: 'my-app',
    cacheControl: 'max-age=3600',
    wait: true,
  })
)
```

:::

## 选项

### <Badge type="danger" text="required" /> cacheName: `string` | `(c: Context) => string` | `Promise<string>`

缓存的名称。可用于存储具有不同标识符的多个缓存。

### <Badge type="info" text="optional" /> wait: `boolean`

布尔值，指示 Hono 是否应在继续请求之前等待 `cache.put` 函数的 Promise 解析。_对于 Deno 环境必须为 true_。默认值为 `false`。

### <Badge type="info" text="optional" /> cacheControl: `string`

`Cache-Control` header 的指令字符串。有关更多信息，请参阅 [MDN 文档](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control)。当未提供此选项时，不会向请求添加 `Cache-Control` header。

### <Badge type="info" text="optional" /> vary: `string` | `string[]`

在响应中设置 `Vary` header。如果原始响应 header 已经包含 `Vary` header，则合并值，删除任何重复项。将其设置为 `*` 将导致错误。有关 Vary header 及其对缓存策略的影响的更多详细信息，请参阅 [MDN 文档](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Vary)。

### <Badge type="info" text="optional" /> keyGenerator: `(c: Context) => string | Promise<string>`

为 `cacheName` 存储中的每个请求生成 keys。这可用于根据请求参数或上下文参数缓存数据。默认值为 `c.req.url`。

### <Badge type="info" text="optional" /> cacheableStatusCodes: `number[]`

应缓存的状态码数组。默认值为 `[200]`。使用此选项缓存具有特定状态码的响应。

```ts
app.get(
  '*',
  cache({
    cacheName: 'my-app',
    cacheControl: 'max-age=3600',
    cacheableStatusCodes: [200, 404, 412],
  })
)
```
