# Bearer Auth 中间件

Bearer Auth 中间件通过验证 Request header 中的 API 令牌提供认证。
访问端点的 HTTP 客户端将添加带有 `Bearer {token}` 作为 header 值的 `Authorization` header。

从终端使用 `curl`，它将如下所示：

```sh
curl -H 'Authorization: Bearer honoiscool' http://localhost:8787/auth/page
```

## 导入

```ts
import { Hono } from 'hono'
import { bearerAuth } from 'hono/bearer-auth'
```

## 用法

> [!NOTE]
> 你的 `token` 必须匹配正则表达式 `/[A-Za-z0-9._~+/-]+=*/`，否则将返回 400 错误。值得注意的是，此正则表达式同时适用于 URL-safe Base64 和标准 Base64 编码的 JWT。此中间件不要求 bearer 令牌是 JWT，只需匹配上述正则表达式。

```ts
const app = new Hono()

const token = 'honoiscool'

app.use('/api/*', bearerAuth({ token }))

app.get('/api/page', (c) => {
  return c.json({ message: 'You are authorized' })
})
```

要限制为特定路由 + 方法：

```ts
const app = new Hono()

const token = 'honoiscool'

app.get('/api/page', (c) => {
  return c.json({ message: 'Read posts' })
})

app.post('/api/page', bearerAuth({ token }), (c) => {
  return c.json({ message: 'Created post!' }, 201)
})
```

要实现多个令牌（例如，任何有效令牌都可以读取，但创建/更新/删除仅限于特权令牌）：

```ts
const app = new Hono()

const readToken = 'read'
const privilegedToken = 'read+write'
const privilegedMethods = ['POST', 'PUT', 'PATCH', 'DELETE']

app.on('GET', '/api/page/*', async (c, next) => {
  // 有效令牌列表
  const bearer = bearerAuth({ token: [readToken, privilegedToken] })
  return bearer(c, next)
})
app.on(privilegedMethods, '/api/page/*', async (c, next) => {
  // 单个有效特权令牌
  const bearer = bearerAuth({ token: privilegedToken })
  return bearer(c, next)
})

// 定义 GET、POST 等的 handlers
```

如果你想自己验证令牌的值，请指定 `verifyToken` 选项；返回 `true` 表示接受。

```ts
const app = new Hono()

app.use(
  '/auth-verify-token/*',
  bearerAuth({
    verifyToken: async (token, c) => {
      return token === 'dynamic-token'
    },
  })
)
```

## 选项

### <Badge type="danger" text="required" /> token: `string` | `string[]`

用于验证传入 bearer 令牌的字符串。

### <Badge type="info" text="optional" /> realm: `string`

领域的域名，作为返回的 WWW-Authenticate 挑战 header 的一部分。默认值为 `""`。
请参阅更多：https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/WWW-Authenticate#directives

### <Badge type="info" text="optional" /> prefix: `string`

Authorization header 值的前缀（或称为 `schema`）。默认值为 `"Bearer"`。

### <Badge type="info" text="optional" /> headerName: `string`

header 名称。默认值为 `Authorization`。

### <Badge type="info" text="optional" /> hashFunction: `Function`

用于处理认证令牌安全比较的散列函数。

### <Badge type="info" text="optional" /> verifyToken: `(token: string, c: Context) => boolean | Promise<boolean>`

用于验证令牌的函数。

### <Badge type="info" text="optional" /> noAuthenticationHeader: `object`

自定义请求没有认证 header 时的错误响应。

- `wwwAuthenticateHeader`: `string | object | MessageFunction` - 自定义 WWW-Authenticate header 值。
- `message`: `string | object | MessageFunction` - 响应体的自定义消息。

`MessageFunction` 是 `(c: Context) => string | object | Promise<string | object>`。

### <Badge type="info" text="optional" /> invalidAuthenticationHeader: `object`

自定义认证 header 格式无效时的错误响应。

- `wwwAuthenticateHeader`: `string | object | MessageFunction` - 自定义 WWW-Authenticate header 值。
- `message`: `string | object | MessageFunction` - 响应体的自定义消息。

### <Badge type="info" text="optional" /> invalidToken: `object`

自定义令牌无效时的错误响应。

- `wwwAuthenticateHeader`: `string | object | MessageFunction` - 自定义 WWW-Authenticate header 值。
- `message`: `string | object | MessageFunction` - 响应体的自定义消息。
