# CSRF 保护

此中间件通过检查 `Origin` header 和 `Sec-Fetch-Site` header 来防止 CSRF 攻击。如果任一验证通过，则允许请求。

中间件仅验证以下请求：

- 使用不安全的 HTTP 方法（不是 GET、HEAD 或 OPTIONS）
- 具有可以由 HTML 表单发送的内容类型（`application/x-www-form-urlencoded`、`multipart/form-data` 或 `text/plain`）

不发送 `Origin` headers 的旧浏览器，或使用反向代理删除这些 headers 的环境，可能无法正常工作。在此类环境中，请使用其他 CSRF 令牌方法。

## 导入

```ts
import { Hono } from 'hono'
import { csrf } from 'hono/csrf'
```

## 用法

```ts
const app = new Hono()

// 默认：origin 和 sec-fetch-site 验证
app.use(csrf())

// 允许特定来源
app.use(csrf({ origin: 'https://myapp.example.com' }))

// 允许多个来源
app.use(
  csrf({
    origin: [
      'https://myapp.example.com',
      'https://development.myapp.example.com',
    ],
  })
)

// 允许特定 sec-fetch-site 值
app.use(csrf({ secFetchSite: 'same-origin' }))
app.use(csrf({ secFetchSite: ['same-origin', 'none'] }))

// 动态 origin 验证
强烈建议验证协议以确保匹配 `$`。
你绝不应该进行正向匹配。
app.use(
  '*',
  csrf({
    origin: (origin) =>
      /https:\/\/(\w+\.)?myapp\.example\.com$/.test(origin),
  })
)

// 动态 sec-fetch-site 验证
app.use(
  csrf({
    secFetchSite: (secFetchSite, c) => {
      // 始终允许 same-origin
      if (secFetchSite === 'same-origin') return true
      // 为 webhook 端点允许 cross-site
      if (
        secFetchSite === 'cross-site' &&
        c.req.path.startsWith('/webhook/')
      ) {
        return true
      }
      return false
    },
  })
)
```

## 选项

### <Badge type="info" text="optional" /> origin: `string` | `string[]` | `Function`

指定 CSRF 保护的允许来源。

- **`string`**：单个允许的来源（例如 `'https://example.com'`）
- **`string[]`**：允许的来源数组
- **`Function`**：自定义处理器 `(origin: string, context: Context) => boolean` 用于灵活的来源验证和绕过逻辑

**默认**：仅与请求 URL 相同的来源

函数处理器接收请求的 `Origin` header 值和请求上下文，允许基于请求属性（如路径、headers 或其他上下文数据）进行动态验证。

### <Badge type="info" text="optional" /> secFetchSite: `string` | `string[]` | `Function`

使用 [Fetch Metadata](https://web.dev/articles/fetch-metadata) 指定 CSRF 保护的允许 Sec-Fetch-Site header 值。

- **`string`**：单个允许的值（例如 `'same-origin'`）
- **`string[]`**：允许的值数组（例如 `['same-origin', 'none']`）
- **`Function`**：自定义处理器 `(secFetchSite: string, context: Context) => boolean` 用于灵活验证

**默认**：仅允许 `'same-origin'`

标准 Sec-Fetch-Site 值：

- `same-origin`：来自同一来源的请求
- `same-site`：来自同一站点（不同子域）的请求
- `cross-site`：来自不同站点的请求
- `none`：不是来自网页的请求（例如，浏览器地址栏、书签）

函数处理器接收请求的 `Sec-Fetch-Site` header 值和请求上下文，支持基于请求属性的动态验证。
